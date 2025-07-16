## CleanCloset iOS 앱 - 네이티브 기능 명세서

**문서 목적**: 본 문서는 CleanCloset iOS 앱의 네이티브 Swift 코드 구현을 기반으로, 현재 시점의 화면 구성, 데이터 처리, API 통신, 사용자 인터랙션 등 핵심 기능을 상세히 기술합니다. 이 명세는 Flutter 전환 제안을 제외하고 오직 기존 네이티브 코드의 동작을 정확히 분석하고 문서화하는 데 중점을 둡니다.

---

### 1. 화면 구성 (Screen Composition)

CleanCloset의 메인 제어 화면은 `CleanClosetViewController`를 중심으로 여러 개의 정보 카드(Card) 컴포넌트가 조합된 형태로 구성됩니다.

#### 1.1. CleanClosetViewController (메인 뷰 컨트롤러)

- **역할**: 화면의 전체적인 레이아웃과 생명주기를 관리하며, 각 하위 카드 컴포넌트를 담는 컨테이너 역할을 합니다.
- **UI 요소**:
  - `statusCard`: 제품의 핵심 상태를 표시하는 메인 카드.
  - `cycleCard`: 현재 설정된 세탁/관리 사이클을 표시하는 카드.
  - `nightCareCard`: 야간 관리 모드의 상태를 표시하는 카드.
  - `waterTankStatusCard`: 급수통 상태를 표시하는 카드.
  - `drainTankStatusCard`: 배수통 상태를 표시하는 카드.
  - `tabbar` & `tabBarExtended`: 화면 하단의 탭 메뉴 UI.
- **네비게이션**:
  - **좌측 상단**: `createNavigationBackButton()`을 통해 '뒤로 가기' 버튼이 생성됩니다.
  - **우측 상단**: `createNavigationSettingsButton()`을 통해 '설정' 화면으로 이동하는 버튼이 생성됩니다.
  - **헤더 타이틀**: `Constants.CONTROL` 값 (예: "제어")으로 설정됩니다.

#### 1.2. 핵심 UI 컴포넌트 (Cards)

##### 1.2.1. StatusCard (상태 카드)

제품의 전반적인 운전 상태를 종합적으로 보여주는 가장 중요한 카드입니다.

- **구성 요소**:
  - **헤더**:
    - `collapseButton` (ID: 200): 카드 접기/펼치기 버튼.
    - `collapseButtonImage` (ID: 201): 버튼 내 이미지 (화살표 등).
    - `applianceNameLabel` (ID: 202): 제품 이름 레이블.
  - **콘텐츠**:
    - `cardContents` (ID: 203): 카드 콘텐츠 영역의 컨테이너 뷰.
    - `applianceImageView` (ID: 210): 제품 이미지.
  - **상태 정보**:
    - `cycleStatusView` (ID: 220): 사이클 상태(운전중, 일시정지 등) 표시 영역.
    - `cycleStatusLabel` (ID: 222): '상태' 텍스트 레이블.
    - `cycleStatusValue` (ID: 223): 실제 상태 값 (예: "동작중").
  - **시간 정보**:
    - `timeLeftView` (ID: 230): 남은 시간 표시 영역.
    - `timeLeftLabel` (ID: 232): '남은 시간' 텍스트 레이블.
    - `timeLeftValue` (ID: 233): 실제 남은 시간 값 (예: "1시간 20분").
  - **완료 시간**:
    - `finishTimeView` (ID: 240): 종료 예정 시간 표시 영역.
    - `finishTimeLabel` (ID: 242): '종료 예정' 텍스트 레이블.
    - `finishTimeValue` (ID: 243): 실제 종료 시간 값 (예: "오후 3:40").
  - **진행 상태**:
    - `progressView` (ID: 250): 진행률 바 컨테이너.
    - `progressBar` (ID: 251): 실제 진행 상태를 시각적으로 보여주는 바.
- **동작**:
  - `isStatusCollapsed` Boolean 상태 값을 통해 접힘/펼침 상태를 관리합니다.
  - 상태 변경 시 0.2초의 애니메이션 효과와 함께 부드럽게 크기가 조절됩니다.

##### 1.2.2. 기타 카드

- **CycleCard (사이클 카드)**:
  - `cycleLabel` (ID: 300): '사이클' 텍스트 레이블.
  - `cycleNameLabel` (ID: 301): 현재 선택된 사이클의 이름 (예: "스팀 살균").
- **NightCareCard (야간 관리 카드)**:
  - `nightCarelabel` (ID: 400): '야간 관리' 텍스트 레이블.
  - `infoButton` (ID: 401): 기능 설명 팝업을 띄우는 정보 버튼.
  - `currentState` (ID: 402): 현재 상태 ("ON" 또는 "OFF").
- **WaterTankCard (급수통 카드)**:
  - `waterTankStatusLabel` (ID: 500): '급수통 상태' 텍스트 레이블.
  - `waterTankInfoButton` (ID: 501): 기능 설명 팝업을 띄우는 정보 버튼.
  - `currentState` (ID: 502): 현재 상태 (예: "정상", "물 채움 필요").
  - `waterTankWarning` (ID: 503): 경고 아이콘.
- **DrainTankCard (배수통 카드)**:
  - `drainTankStatusLabel` (ID: 600): '배수통 상태' 텍스트 레이블.
  - `drainTankInfoButton` (ID: 601): 기능 설명 팝업을 띄우는 정보 버튼.
  - `currentState` (ID: 602): 현재 상태 (예: "정상", "물 비움 필요").
  - `drainTankWarning` (ID: 603): 경고 아이콘.

---

### 2. 데이터 처리 및 비즈니스 로직

서버(ERD)로부터 받은 데이터를 해석하여 UI에 동적으로 반영합니다.

#### 2.1. ERD (Event-Driven Data) 기반 UI 상태 제어

- **StatusCard 상태 표시 로직**:
  - 기기의 상태(`LAUNDRY_ERD_MACHINE_STATUS`) 값에 따라 UI 요소의 가시성이 결정됩니다.
  - **종료 예정 시간 (`finishTimeView`)**: `run`, `pause` 상태일 때만 표시됩니다.
  - **남은 시간 (`timeLeftView`) 및 진행률 (`progressView`)**: `run`, `pause`, `delayRun`, `delayPause` 상태일 때 표시됩니다.
  - 이 외의 상태(예: 전원 꺼짐, 대기)에서는 해당 UI 요소들이 숨겨집니다.

- **WaterTank 상태 표시 로직**:
  - 급수통 상태(`LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS`) ERD 값의 특정 비트(인덱스 2번 비트가 "1")를 분석합니다.
  - 조건이 참이면, "물 채움 필요"(`Constants.FILL_TANK`) 텍스트가 경고 색상(`Color.warning`)으로 표시되고, 정보 버튼과 경고 라벨이 함께 나타납니다.

- **NightCare 상태 표시 로직**:
  - 서브 사이클(`LAUNDRY_ERD_MACHINE_SUB_CYCLE`) ERD 값을 확인합니다.
  - 값이 `nightCare`에 해당하면 상태를 "ON"으로 표시하고, 그렇지 않으면 "OFF"로 표시합니다.

#### 2.2. 기능(Feature) 기반 카드 표시 조건

- **급수통/배수통 카드 가시성**:
  - `didGetFeatures` 함수는 서버로부터 제품이 지원하는 기능 목록을 받습니다.
  - 기능 목록에 `clothesCareWaterDrainTankStatusCapable`이 포함된 경우에만 **급수통 카드**와 **배수통 카드**가 화면에 표시됩니다.
  - 해당 기능이 없으면 두 카드는 모두 숨겨져 사용자에게 보이지 않습니다.

---

### 3. 데이터 통신 (Data Communication)

앱은 HTTP API와 WebSocket을 통해 서버와 통신합니다.

#### 3.1. API (HTTPS) 요청 및 응답 처리

`RemoteDataManager`는 서버 API 호출을 담당하며, 각 요청은 지정된 응답 처리 콜백 함수로 연결됩니다.

| `RemoteDataManager` 함수         | API 요청 타입                     | 응답 처리 콜백 함수            | 설명                                     |
| -------------------------------- | --------------------------------- | ------------------------------ | ---------------------------------------- |
| `retrieveFeature()`              | `.feature`                        | `onFeatureRetrieved()`         | 제품 지원 기능 목록 조회                 |
| `retrieveApplianceMetaData()`    | `.metaData`                       | `onApplianceMetaDataRetrieved()` | 제품 메타데이터 조회                     |
| `retrievePreferences()`          | `.preferences`                    | `onPreferencesRetrieved()`     | 사용자 설정 정보 조회                    |
| `retrieveUserInfo()`             | `.userInformation`                | `onUserInformationRetrieved()` | 사용자 정보 조회                         |
| `retrieveModelValidation()`      | `.modelValidation`                | `onModelValidationRetrieved()` | 모델 유효성 검증                         |
| `retrieveTimezoneList()`         | `.timezoneList`                   | `onTimezoneListRetrieved()`    | 타임존 목록 조회                         |
| `postTimezone()`                 | `.timezoneAppliance`              | - (별도 핸들링)                | 기기 타임존 설정                         |
| `retrieveMetadata()`             | `.laundryRetrieveMetadata`        | - (별도 핸들링)                | 세탁기 관련 메타데이터 조회              |

#### 3.2. 실시간 데이터 처리 (WebSocket)

- **역할**: 기기 상태(ERD)의 실시간 변경 사항을 수신하여 UI를 즉시 업데이트합니다.
- **구현**:
  - `updateErd(_:)`: 특정 ERD 값이 변경되었을 때 호출됩니다. `Presenter`의 `updateErdByWebSocket(erdNumber:)`를 호출하여 변경된 데이터를 처리하도록 위임합니다.
  - `updateCache()`: WebSocket을 통해 전체 데이터 캐시를 갱신하라는 신호를 받으면 호출됩니다. `Presenter`의 `updateCacheByWebSocket()`을 호출하여 관련 데이터를 새로고침합니다.

---

### 4. 사용자 인터랙션 및 이벤트 처리

사용자의 입력에 따라 정의된 동작을 수행합니다.

#### 4.1. 카드 내 버튼 동작 (Delegate Pattern)

`CleanClosetStatusCardDelegate` 프로토콜을 통해 카드 내부의 버튼 클릭 이벤트를 `ViewController`로 전달하고, `Presenter`가 최종적으로 처리합니다.

- `onButtonPressed(_:)` 함수가 호출되면, `buttonType`에 따라 분기됩니다.
  - `.nightCareInfo`: `presenter.onPressedNightCareInfoButton()` 호출 → 야간 관리 정보 팝업 요청.
  - `.waterTankStatusInfo`: `presenter.onPressedWaterTankStatusInfoButton()` 호출 → 급수통 정보 팝업 요청.
  - `.drainTankStatusInfo`: `presenter.onPressedDrainTankStatusInfoButton()` 호출 → 배수통 정보 팝업 요청.

#### 4.2. 팝업 관리

- `showPopup(viewPopupType:)` 함수는 `Presenter`로부터 전달받은 팝업 타입에 따라 적절한 팝업을 화면에 표시합니다.
- 실제 팝업 표시는 `SwiftPopupSupporter.shared`라는 싱글턴 객체를 통해 중앙에서 관리됩니다.
  - `.nightCareInfo`: `showNightCareInformation()` 호출.
  - `.waterTankStatusInfo`: `showWaterTankStatusInformation()` 호출.
  - `.drainTankStatusInfo`: `showDrainTankStatusInformation()` 호출.

---

### 5. 화면 생명주기 관리 (View Lifecycle Management)

화면의 상태(나타나기, 사라지기 등)에 따라 필수적인 초기화 및 정리 작업을 수행합니다.

- **`viewWillAppear`**:
  - WebSocket 및 OTA(Over-the-Air) 서비스에 대한 바인딩(연결)을 수행합니다.
  - 제품의 기능 지원 여부를 체크합니다.
- **`viewDidAppear`**:
  - WebSocket에 로그인합니다.
  - 제품 닉네임, 메타데이터, 관련 JSON 파일을 서버에 요청하여 최신 정보를 가져옵니다.
- **`viewWillDisappear`**:
  - WebSocket 및 OTA 서비스의 언바인딩(연결 해제)을 수행합니다.
- **`viewDidPause`** (앱이 백그라운드로 전환될 때):
  - OTA 관련 팝업을 숨깁니다.
  - WebSocket에서 로그아웃합니다.
- **`viewWillResume`** (앱이 포그라운드로 다시 전환될 때):
  - WebSocket에 다시 로그인합니다.
  - 개인정보 처리방침 관련 정보를 요청합니다.
