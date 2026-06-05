# FRACAS Semantic Search — 기술 보고서

> 자연어로 과거 고장 사례를 검색해 **유사 문제 + 원인(Cause) + 해결책(Corrective Action)** 을 찾아주고,
> 애매한 경우에만 LLM이 종합 분석하는 2단계 시스템.
> 세탁기/건조기(Clothes Care) FRACAS 23,230행 → 정제 12,397건 기반.

---

## 0. 한 줄 요약

기존 FRACAS 데이터는 엑셀 23K행 텍스트라 "예전에 비슷한 고장 어떻게 고쳤지?"를 찾을 방법이 없었다.
이 시스템은 **도메인 파인튜닝한 bge-m3 임베딩 + FAISS 의미검색**으로 유사 사례를 즉시 찾고,
**해결책이 있는 케이스를 우선 노출**하며, 애매할 때만 **GPT가 종합 분석**한다. (로컬 검색은 무료, LLM은 최소 호출)

---

## 1. 문제 정의 & 동기

| 항목 | 내용 |
|------|------|
| 데이터 | GE Appliances 세탁/건조 FRACAS 보고서 (Failure Reporting, Analysis, and Corrective Action System) |
| 규모 | 원본 **23,230행** → 유효 **12,397건** |
| 기존 한계 | 엑셀 키워드(Ctrl+F) 검색만 가능 → 표현이 다르면("whining" vs "loud noise") 못 찾음. 원인/해결책이 흩어져 있어 재사용 불가 |
| 목표 | ① 자연어 문제 입력 → 유사 과거 사례 Top-K, ② 각 사례의 Cause/Corrective Action 함께 제공, ③ Subsystem/Category 자동 힌트, ④ 비용 최소화 |

---

## 2. 전체 아키텍처

```
                         사용자 문제 입력 (자연어, 영어 권장)
                                     │
                                     ▼
              ┌──────────────────────────────────────────────┐
              │  [1차] 로컬 의미검색 (무료)                      │
              │  • fine-tuned bge-m3 로 query 임베딩            │
              │  • FAISS IndexFlatIP 코사인 Top-K               │
              │  • 해결책 보유 케이스 우선 재랭킹                 │
              │  • LogReg 분류기로 Failure Category 확률 힌트     │
              └──────────────────────────────────────────────┘
                                     │
                          needs_llm 판정 (api_server)
                     ┌───────────────┴────────────────┐
              유사도 충분/카테고리 일관          유사도 낮음 / 카테고리 분산
                     │                                 │
                     ▼                                 ▼
        검색 결과만 포맷 반환 (LLM 0회)      [2차] GPT 종합 분석 (gpt-4o-mini)
                                          유사 사례를 근거로 원인·조치·카테고리 판단
```

핵심 설계 철학: **검색이 충분히 좋으면 LLM을 부르지 않는다.** 12,397건을 다 LLM에 넣을 수 없고,
대부분의 질의는 의미검색만으로 정답 사례가 1등에 잡히기 때문이다. LLM은 "애매할 때만" 비용을 지불한다.

---

## 3. 데이터 전처리 (`src/preprocess.py`) — 엑셀 기반 상세

### 3.0 원본 엑셀의 정체

`fracasData.xls`는 확장자만 `.xls`이고 **실제로는 HTML `<table>`** (GE 내부 FRACAS 웹 리포트를 "Excel로 내보내기" 한 형태)이다.
그래서 `pd.read_excel`이 아니라 **`pd.read_html(...)[0]`** 로 읽고, 첫 번째 행을 컬럼 헤더로 승격시킨다.

- 파일 크기: 약 32 MB
- 원본 행: **23,230행** (`<tr>` 기준, 헤더 포함)
- 원본 컬럼: **34개** (Report Number, Project Name, Subsystem, Problem Description, Investigation, Cause, Corrective Action, Failure Category, Failure Classification, Model Number, Incident Date, Test Hours/Cycles/Years 등)

### 3.1 단계별 클렌징 파이프라인

| 단계 | 처리 | 코드 함수 | 효과 |
|------|------|-----------|------|
| 1. 로드 | HTML 테이블 → DataFrame, 첫 행 → 헤더 | `load_raw()` | 23,230행 적재 |
| 2. 유효행 필터 | `Problem Description` 결측/`nan`/빈문자 행 제거 | `filter_valid()` | **23,230 → 12,397** (텍스트 없는 행은 검색 불가) |
| 3. 컬럼 제거 | 검색에 무의미한 **18개 컬럼** 드롭 | `DROP_COLS` | 승인자·승인일(6쌍), SCR Code, Part Number, SW Rev Found/Fixed, Supplier, BCR/BPCR #, Product Line, C/A Implemented Date, Incident Time, Test Request # |
| 4. Junk 값 → NaN | 실질 결측인데 문자로 들어온 값 치환 | `replace_junk()` | `-`, `--`, `n/a`, `N/A`, `None.`, `Unknown.`, `See Attachment`, `Code review`, `Project Canceled`, `정규식 -+`, `"Remarks: Record information..."` 등 |
| 5. 텍스트 정제 | 본문 4개 필드 정규화 | `clean_text()` | 번호 prefix(`"1. "`) 제거 → 연속 공백/개행 1칸 압축 → `**` 마크업 제거 → **5자 미만은 NaN** |
| 6. 메타 정제 | 식별자 더미값 제거 | `clean_model_number()`, `clean_station()` | Model Number/Station #/SN 의 `"0"`,`"1"`,`"GE"`,`"Field"`,`"Unknown"`,`"CAMCO"`,`1~2자리 숫자` → NaN |
| 7. 라벨 인코딩 | `PENDING` 카테고리 재분류 | `encode_failure_category()` | 아래 3.3 |
| 8. 파생 컬럼 | `combined_text`, `desc_length` 생성 | `build_combined_text()` | 임베딩/필터용 |

> 정제 대상 본문 4필드 = **Problem Description / Investigation / Cause / Corrective Action**.
> 이 4개가 "문제–조사–원인–해결"의 FRACAS 핵심 스토리라인이라 가장 공들여 정제한다.

### 3.2 정제 후 결측 현황 (12,397건 기준, 실측)

| 컬럼 | 유효 | 결측률 | 비고 |
|------|------|--------|------|
| Problem Description | 12,396 | 0.0% | 검색의 기준 텍스트 |
| Failure Category | 12,397 | 0.0% | (PENDING 재인코딩 완료) |
| Failure Classification | 12,397 | 0.0% | CHARGEABLE 8,667 / NON-CHARGEABLE 3,730 |
| Incident Date | 12,397 | 0.0% | |
| Subsystem | 12,075 | 2.6% | |
| Corrective Action | 8,808 | 29.0% | **해결책 본문** |
| Cause | 8,451 | 31.8% | **원인 본문** |
| Investigation | 9,391 | 24.2% | |
| Model Number | 10,586 | 14.6% | |
| Part Name | 5,490 | 55.7% | 절반 이상 비어 있음 → 검색 핵심에서 제외 |

### 3.3 PENDING 라벨 인코딩 (핵심 도메인 판단)

원본 `Failure Category`에는 `PENDING`(분석 진행중)이 대량으로 섞여 있다. 그냥 두면 라벨이 오염되므로 **텍스트 유무로 분기**한다:

```
Failure Category == "PENDING" 인 행에 대해:
   Cause 또는 Corrective Action 이 10자 이상 존재  →  "UNRESOLVED_WITH_TEXT"  (조사 흔적은 있으나 결론 미확정)
   둘 다 없음                                       →  "UNRESOLVED"           (정보 자체가 없음)
```

→ 결과적으로 `UNRESOLVED 2,208 + UNRESOLVED_WITH_TEXT 1,688 = 3,896건`, 전체의 **약 31%가 미해결 라벨**.
이는 데이터의 본질적 특성이며, **학습·평가·분류기에서 이 라벨들은 전부 제외**한다(노이즈 라벨).

### 3.4 해결정보(원인/해결책) 보유 현황 — 비즈니스적으로 가장 중요

검색의 목적이 "과거에 어떻게 고쳤나"이므로, 실제 해결정보가 얼마나 있는지가 핵심 지표다.

| 항목 | 건수 | 비율 |
|------|------|------|
| Corrective Action 보유 | 8,808 | 71.0% |
| Cause 보유 | 8,451 | 68.2% |
| **Cause + CA 둘 다 보유** | 7,784 | **62.8%** |
| Cause 또는 CA 보유 | 9,475 | 76.4% |
| 아무 해결정보 없음(조사도 없음) | 1,843 | 14.9% |

→ 약 **76%는 재사용 가능한 해결 단서**를 가진다. 이게 검색의 가치를 보장한다.
동시에 약 15%는 단서가 전혀 없어, "water leaking from bottom" 같은 일부 질의에서 미해결 사례가 노출되는 원인이 된다.

### 3.5 텍스트 통계 & 통합 필드

- Problem Description 길이: 평균 **189자**, 중앙값 114자, 최소 4 ~ 최대 3,930자 (길이 편차 큼)
- `combined_text` = `"Problem: ... | Cause: ... | Corrective Action: ... | Investigation: ..."` 결합 (있는 필드만)
- `desc_length` = Problem 길이(필터링용)

### 3.6 정제 전후 예시 (실측)

```
[원본 셀 예]  Cause 컬럼에 "-", "n/a", "See Attachment", "Code review" 같은 값
   → replace_junk()로 전부 NaN 처리

[정제 후 살아남은 실제 레코드 예시]
 - SYS_WA-0003 | PULSATOR | UNRESOLVED_WITH_TEXT
   Problem: Metal to metal hitting noise in agitate ...
   Cause  : Rubber isolator not fully pressed in place, causing armature not to seat squarely ...
   Action : CAR has been entered for Yatong and is being addressed by TJ Berhow ...
```

### 3.7 전처리 산출물

- `data/fracas_clean.parquet` — 정제 12,397건 × 34컬럼 (다운스트림 전 단계의 단일 진실원천)
- `data/meta.json` — 컬럼/카테고리/서브시스템/프로젝트 분포 스냅샷

**데이터 분포 요약**
- Subsystem Top 5: CONTROLS 4,647 · DRIVE 2,529 · STRUCTURE 1,362 · SUBWASHER 1,200 · FILL 387
- Failure Category Top 6(미해결 제외): REPEAT FAILURE 1,594 · PART DESIGN 1,514 · SYSTEM DESIGN 1,102 · PART WORKMANSHIP 886 · SOFTWARE ERROR 883 · NON-PRODUCTION PROCESS 302

---

## 4. 데이터마이닝 — 학습쌍 생성 (`src/make_train_pairs.py`)

라벨(정답 유사쌍)이 없는 검색 데이터에서 **"무엇이 서로 비슷한가"를 데이터로부터 정의**하는 단계.
이게 이 프로젝트의 데이터마이닝 핵심이다.

### 4.1 핵심 가정 (약지도 / weak supervision)

> **같은 `(Failure Category, Subsystem)` 그룹에 속한 Problem Description 두 개는 의미적으로 유사하다.**

즉 "DRIVE 서브시스템의 PART WORKMANSHIP 고장"끼리는 표현이 달라도 같은 부류라고 보고, 그 안에서 문장 쌍을 묶는다.
사람이 일일이 라벨링하지 않고, **기존 엔지니어들이 분류해 둔 두 메타 컬럼을 약한 정답 신호로 재활용**하는 방식이다.

`MultipleNegativesRankingLoss(MNR)`는 배치 내 나머지 샘플을 자동으로 negative로 사용하므로, **positive(유사) 쌍만** 만들면 된다.

### 4.2 데이터 정제 & 그룹화 규칙

1. **추가 노이즈 제거**: Cause/CA 중 `undetermined / unable to determine / unknown / none / n/a / tbd / pending / "-" / "see item..."` → NaN
2. **제외 카테고리**: `UNRESOLVED`, `UNRESOLVED_WITH_TEXT`, `NORMAL FUNCTION`, `UNABLE TO DUPLICATE` (라벨 불확실/실패 아님)
3. **최소 길이**: Problem Description 20자 미만 제외
4. **그룹화**: `groupby(Failure Category, Subsystem)`, 그룹 내 **고유 Problem Description** 추출
5. **쌍 생성**: 그룹당 샘플 3개 이상일 때 `combinations(descs, 2)` 로 모든 쌍 생성 후 셔플
6. **상한**: 그룹당 **최대 60쌍** (대형 그룹이 데이터를 독점하지 않도록 균형 — 예: CONTROLS 4,647건이 전부 폭주하는 것 방지)
7. **분할**: 95% train / 5% val

### 4.3 산출물 (실측)

| 파일 | 건수 |
|------|------|
| `data/train_pairs.jsonl` | **6,290 쌍** |
| `data/val_pairs.jsonl` | **332 쌍** |

줄 형식: `{"anchor": ..., "positive": ..., "category": ..., "subsystem": ...}`

**카테고리별 학습쌍 분포 (Top)**: SYSTEM DESIGN 825 · PART DESIGN 772 · REPEAT FAILURE 738 · PART WORKMANSHIP 690 · NON-PRODUCTION PROCESS 474 · TEST PROCEDURE ERROR 338 · SYSTEM WORKMANSHIP 309 · IMPROPER INSTALLATION 296 · END OF LIFE WEAROUT 284 · SOFTWARE ERROR 248

**상위 기여 그룹 (Category / Subsystem, 그룹당 상한 60에 도달)**:
REPEAT FAILURE/FILL · SYSTEM DESIGN/SUBWASHER · TEST OPERATOR ERROR/CONTROLS · PART WORKMANSHIP/DRIVE · REPEAT FAILURE/DRIVE · COMPONENT DESIGN/CONTROLS …
→ 60쌍 상한 덕분에 특정 대형 그룹이 아니라 **다양한 (카테고리×서브시스템) 조합이 고르게** 학습에 기여한다.

---

## 5. 모델 학습 ① — 임베딩 파인튜닝 (`src/finetune.py`)

### 5.1 무엇을 학습하나

베이스 `bge-m3`는 범용 다국어 임베딩이라 "세탁기 도메인 용어"의 미묘한 유사도를 잘 못 잡는다.
파인튜닝의 목표는 **같은 고장 유형(positive 쌍)의 문장은 임베딩 공간에서 가까워지고, 다른 유형은 멀어지도록** 모델을 당기는 것.

### 5.2 손실 함수 — MultipleNegativesRankingLoss (대조학습)

- 배치 안의 `(anchor, positive)` 쌍에서, anchor에 대해 **자기 positive는 가깝게, 같은 배치의 다른 positive들은 negative로 멀게** 만든다.
- 별도 negative 라벨링이 필요 없어(= in-batch negatives) 우리처럼 positive만 있는 데이터에 최적.
- **batch size가 곧 negative 개수**라 클수록 유리 → 메모리 한도 내에서 최대화하려 seq를 128로 줄임.

### 5.3 하이퍼파라미터

| 항목 | 값 | 이유 |
|------|-----|------|
| 베이스 모델 | `BAAI/bge-m3` (1024차원, 다국어) | 다국어라 한글 질의도 어느 정도 동작 |
| Loss | `MultipleNegativesRankingLoss` | positive-only 대조학습 |
| Batch size | 16 | negative 확보 (MPS 메모리 한도) |
| Epochs | 2 | 6,290쌍, 과적합 방지 |
| Max seq len | 128 | 메모리 확보 → batch 키우기 |
| Warmup | 전체 step의 10% | 초기 학습 안정화 |
| AMP | CUDA만 사용(MPS 미지원) | |
| Device | MPS / CUDA / CPU 자동 | Apple Silicon에서 학습 |
| 산출 | `models/bge-m3-fracas-v2/` (운영), `bge-m3-fracas`(v1) | safetensors |

> **MPS 호환 패치**: torch 2.2 + MPS에서 `torch.mps.device` 가 없어 캐시형 MNR이 깨진다.
> 이를 no-op 컨텍스트로 몽키패치(`finetune.py` 상단)하고, 일반 MNR + 작은 seq로 우회했다.
> 학습 중간 가중치는 `checkpoints/` (model, model_1 … model_4)에 저장된다.

### 5.4 검증

`val_pairs.jsonl`(332쌍)에 대해 `EmbeddingSimilarityEvaluator`로 positive 쌍 유사도를 추적,
`src/eval_compare.py`로 base vs ft의 평균 positive 유사도를 비교한다(ft가 더 높으면 도메인 학습 성공).

---

## 5b. 모델 학습 ② — Failure Category 분류기

(상세는 8장) 파인튜닝 임베딩(`index/embeddings.npy`) 위에 **LogisticRegression**을 얹어,
검색 유사도로는 약한 "원인 유형(Failure Category)"을 직접 분류하도록 별도 학습한 것. 두 번째 학습 산출물이다.

---

## 6. 인덱싱 (`src/build_index.py`)

```bash
python3 src/build_index.py --model models/bge-m3-fracas-v2
```

| 단계 | 내용 |
|------|------|
| 임베딩 입력 텍스트 | `Problem Description + "Cause: ..." + "Fix: ..."` 결합, 2048자 상한 (`get_embed_text`) |
| 인코딩 | `normalize_embeddings=True` → 코사인 유사도를 inner product로 계산 가능 |
| 인덱스 | `faiss.IndexFlatIP(1024)` — 정확 검색(Flat). 12K건은 Flat으로 충분 (IVF 불필요) |
| 산출물 | `index/fracas.index` (FAISS), `index/metadata.pkl` (원본 메타, 인덱스 순서 동일), `index/embeddings.npy` (분류기 재사용), `index/index_stats.json` |

`index_stats.json` 의 `model` 필드에 **인덱싱에 쓴 모델 경로**를 저장 → 검색 시 동일 모델로 query를 인코딩(불일치 방지).

**현재 인덱스 통계**: `total_vectors 12,397 · embed_dim 1024 · model models/bge-m3-fracas-v2`

---

## 7. 검색 로직 (`src/search.py`)

단순 Top-K가 아니라 **해결책 우선 재랭킹**이 핵심이다.

### 7.1 흐름

1. query를 fine-tuned 모델로 정규화 임베딩
2. FAISS에서 `top_k × 30` 후보를 넓게 가져옴 (재랭킹 여지 확보)
3. `min_score`(기본 0.3) 미만, 필터(subsystem/category/project) 미달 후보 제거
4. **resolution boost 재랭킹**: 유사도 × 가중치로 정렬

### 7.2 Resolution Boost (`_resolution_boost`)

| 케이스 상태 | 가중치 |
|-------------|--------|
| Cause + Corrective Action 둘 다 있음 | ×1.00 |
| 둘 중 하나 | ×0.97 |
| Investigation만 있음 | ×0.95 |
| 정보 없음 | ×0.90 |

→ 유사도가 비슷하면 **실제로 고친 기록이 있는 사례를 위로** 끌어올린다. (검색의 목적이 "해결책 찾기"이므로)

### 7.3 옵션

- `filter_subsystem / filter_failure_category / filter_project` — 정밀 검색
- `resolved_only` — 해결책 있는 케이스만
- `rerank_by_resolution` — 위 boost on/off

---

## 8. Failure Category 분류기 (`src/train_classifier.py`)

의미검색만으로는 Failure Category(원인 유형) 예측이 약하다. 그래서 **임베딩 위에 LogReg를 따로 학습**.

| 항목 | 값 |
|------|-----|
| 입력 | `index/embeddings.npy` (파인튜닝 임베딩, 메타와 동일 순서) |
| 모델 | `LogisticRegression(max_iter=2000, C=1.0, class_weight="balanced")` |
| 제외 라벨 | UNRESOLVED / UNRESOLVED_WITH_TEXT / PENDING / NORMAL FUNCTION / UNABLE TO DUPLICATE |
| 최소 클래스 크기 | 20건 |
| 클래스 수 | **18개** (REPEAT FAILURE, PART DESIGN, SYSTEM DESIGN, SOFTWARE ERROR, …) |
| 산출 | `models/category_clf.pkl` |

검색 응답에서는 **단정이 아니라 "후보 + 확률" 힌트**로만 노출한다(`predict_category`).
신뢰도 높은 클래스(`SOFTWARE ERROR`, `REPEAT FAILURE`)만 `trusted=true` 플래그.

---

## 9. 2단계 LLM 라우팅 (`src/api_server.py` + `pipelines/fracas_pipeline.py`)

### 9.1 LLM 호출 판정 (`should_call_llm`)

다음 중 하나면 `needs_llm = True`:
- 1등 유사도 < **0.75**
- 상위 3개 결과의 Failure Category가 **전부 다름** (= 진단이 분산되어 사람/LLM 판단 필요)

### 9.2 파이프라인 동작

- `needs_llm = False` → **검색 결과만** 마크다운 포맷 반환 (LLM 0회, 비용 0)
- `needs_llm = True` → 상위 5개 사례를 근거로 **gpt-4o-mini** 가 스트리밍 종합 분석
  - 출력: ① 가장 관련 높은 사례 2~3개 + 이유, ② 추정 근본원인, ③ 권장 시정조치, ④ Failure Category 판단
  - 끝에 참고 케이스 목록 첨부
- OPENAI_API_KEY 없으면 LLM 생략하고 검색 결과만 반환 (graceful degradation)

### 9.3 "1차 분류 → 2차 GPT"의 정확한 의미 (중요)

자주 오해하는 부분이라 명확히 한다.

1. **1차는 항상 실행된다.** GPT 발동 여부와 무관하게 ① 의미검색(유사 사례 Top-K)과 ② 분류기 카테고리 힌트는 **무조건 먼저 계산**된다.
2. **GPT 발동 트리거는 "분류기 확률"이 아니다.** `needs_llm`은 **검색된 상위 3개 사례의 `failure_category`가 전부 다른지**(분산) 또는 1등 유사도가 0.75 미만인지로 판정한다.
   - 화면의 `Category 힌트 18%·10%…` = **분류기** 출력 (참고용)
   - GPT를 켤지 말지 = **검색 결과의 카테고리 일관성** (별개 신호)
3. **최종 카테고리(예: #3 PART DESIGN, #4 SYSTEM DESIGN)는 분류기가 아니라 GPT가 정한 것.**
   프롬프트에서 분류기 힌트는 *"low-confidence, reference only"* 로 명시하고, *"힌트가 아니라 사례를 근거로 판단하라"* 고 지시한다.

```
질의
 └▶ [1차] 의미검색 Top-K + 분류기 힌트   ← 항상 실행
      └▶ needs_llm 판정 (검색 결과의 카테고리 분산 / 유사도)
           ├─ False → #1,#2 : 검색 결과만 반환 (GPT 미사용)
           └─ True  → #3,#4 : 1차 결과(사례5건+힌트)를 근거로 GPT가 종합 → 원인·조치·최종 카테고리
```

즉 **#3·#4는 "1차로 검색+힌트를 만들고, 그 결과를 통째로 GPT에 넘겨 2차 종합"** 한 케이스가 맞다.

---

## 10. 성능 평가 (`src/eval_retrieval.py`)

### 10.1 방법론 — Leave-One-Out

라벨된 케이스 1,500건을 샘플링 → 각 케이스를 쿼리로 사용하되 **자기 자신은 제외**하고,
Top-K 이웃이 같은 `Failure Category` / `Subsystem` 인지 측정. base vs ft 동일 코퍼스로 비교.

### 10.2 결과 (leave-one-out, 1,500 샘플)

| 지표 | base bge-m3 | **ft-v2 (운영)** |
|------|------|------|
| Subsystem Recall@1 | 0.618 | **0.710** |
| Subsystem Recall@3 | 0.807 | **0.853** |
| Category Recall@3 | 0.584 | 0.575 |

### 10.3 해석

- **Subsystem / 유사문제 검색은 강력** — 파인튜닝으로 Recall@1이 +9.2%p 상승(0.618→0.710).
- **Failure Category는 ML 한계(~0.37)** — 카테고리는 *사후 진단 라벨*(고장을 다 분석한 뒤 붙이는 결론)이라
  Problem Description 텍스트만으로는 본질적으로 예측이 어렵다. 그래서:
  - 검색에서는 카테고리를 "확정"하지 않고 **힌트**로만 제공
  - 애매한 경우 **2차 LLM이 사례를 근거로 판단**하도록 위임
- 무작위 추측 베이스라인 대비 분류기/검색 모두 유의미하게 높음(평가 스크립트가 prior도 함께 출력).

---

## 11. 서빙 스택 (`run_stack.sh`)

```
┌─────────────────┐   /search    ┌──────────────────┐   /chat/completions  ┌──────────────┐
│  FastAPI (8000)  │◄────────────►│  Pipelines (9099) │◄───────────────────►│ OpenWebUI 3000│
│  의미검색+분류기  │              │  fracas_pipeline  │                      │   (UI/채팅)   │
└─────────────────┘              └──────────────────┘                      └──────────────┘
   호스트 프로세스                   Docker 컨테이너                            Docker 컨테이너
```

| 서비스 | 포트 | 실행 형태 |
|--------|------|-----------|
| FRACAS 검색 API (FastAPI) | 8000 | 호스트 python (`src/api_server.py`) |
| Pipelines (OpenWebUI) | 9099 | Docker, `SEARCH_API_URL=http://host.docker.internal:8000` |
| OpenWebUI | 3000 | Docker |

**API 엔드포인트**: `GET/POST /search`, `GET /filters`, `GET /health`

**OpenWebUI 연결**: Admin > Settings > Connections > OpenAI API 추가
→ URL `http://host.docker.internal:9099`, Key `0p3n-w3bu!`
> ⚠️ 이때 **Prefix ID 칸은 반드시 비워둘 것**. 값을 넣으면 모델 ID가 변형되어 채팅이 404("Not Found")로 깨진다.

---

## 12. 데모 결과 검증 (실제 4개 질의)

| # | 질의 | 1등 유사도 | 동작 | 평가 |
|---|------|-----------|------|------|
| 1 | Loud whining noise … motor during spin | **0.857** | 검색만 | ⭐ 거의 동일 사례 적중 + 실제 해결책("bearing issue → Replaced motor") |
| 2 | Water leaking from bottom of washer | 0.807 | 검색만 | ✅ 검색 정확(전부 "bottom leak"), 단 해당 사례들이 모두 UNRESOLVED → **데이터에 해결책 없음** |
| 3 | Control board shows error code … won't start | 0.814 | **GPT 발동** | ✅ 카테고리 분산 → GPT가 종합, PART DESIGN 판단 |
| 4 | Vibrates excessively + banging + stops mid-cycle | (복합) | **GPT 발동** | ✅ leveling / unbalance detection 근거 제시, SYSTEM DESIGN 판단 |

### 결론: **결과는 정확하고 의도대로 동작한다.**

- 1번이 이 시스템의 가장 강력한 데모다 — 표현이 달라도("whining"↔"loud noise") 의미로 정확히 매칭하고 **실제 수리 기록**까지 같이 준다.
- 2번은 모델이 아니라 **데이터의 한계**다. "바닥 누수" 사례가 데이터에 다 미해결로 기록돼 있다 → 오히려 FRACAS 프로세스 개선 포인트(이 유형은 근본원인 미규명)를 드러내는 정직한 신호.
- 3·4번은 needs_llm 라우팅이 정확히 발동해 GPT가 합리적으로 종합했다.

---

## 13. 한계 & 향후 개선

| 한계 | 개선 방향 |
|------|-----------|
| Failure Category 예측 정확도 낮음(~0.37) | Cause/CA 텍스트까지 입력에 포함한 분류기, 또는 2차 LLM 라벨링 |
| 데이터의 ~31%가 UNRESOLVED | "해결책 있는 사례만(resolved_only)" 토글을 UI에 노출 |
| 영어 데이터 → 한글 질의는 정확도 하락 | 질의 자동 번역 전처리, 또는 다국어 쿼리 증강 학습 |
| FAISS Flat (전수검색) | 데이터 수십만 건으로 커지면 IVF/HNSW 전환 |
| 누수 등 일부 유형은 해결책 부재 | FRACAS 입력 단계에서 Cause/CA 작성 유도 (데이터 품질 루프) |

---
