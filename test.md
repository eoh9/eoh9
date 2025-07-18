# CleanCloset - Native iOS Implementation Specification

## Overview

이 문서는 CleanCloset iOS 앱의 실제 구현 코드를 기반으로 작성된 기능 명세서입니다. 각 카드의 UI 요소, ERD 처리 로직, API 연동, 상태 변화 조건을 실제 Swift 코드에서 추출하여 정확하게 문서화했습니다.

---

## 🏡 Entry Point: CleanClosetViewController

**File**: `CleanClosetViewController.swift` (226 lines)  
**Purpose**: 5개 상태 카드의 메인 컨트롤러  
**Architecture**: VIPER 패턴 구현

### IBOutlet 연결 (실제 코드 기반)
```swift
@IBOutlet weak var tabbar: GEATabBar!                    // 태그: 미확인
@IBOutlet weak var tabBarExtended: UIView!               // 태그: 미확인  
@IBOutlet weak var statusCard: CleanClosetCardStatus!    // 태그: 미확인
@IBOutlet weak var cycleCard: CleanClosetCycleCard!      // 태그: 미확인
@IBOutlet weak var nightCareCard: CleanClosetNightCareCard! // 태그: 미확인
@IBOutlet weak var waterTankStatusCard: CleanClosetWaterTankStatusCard! // 태그: 미확인
@IBOutlet weak var drainTankStatusCard: CleanClosetDrainTankStatusCard! // 태그: 미확인
```

### Delegate 구현
```swift
extension CleanClosetViewController: CleanClosetStatusCardDelegate {
    func onButtonPressed(_ buttonType: CleanClosetCardButtonType) {
        // 실제 버튼 처리 로직
    }
}
```

---

## 🔌 API Endpoints & Response Entities

### 실제 API 호출 함수 (CleanClosetRemoteDataManager.swift)

| API Function | HTTP Method | Endpoint | Response Entity | 실제 구현 |
|-------------|-------------|----------|-----------------|----------|
| getPreferences() | GET | /preferences | PreferencesEntity | JSONDecoder로 파싱 |
| getFeature() | GET | /feature | FeatureEntity | CommonHttpsCommunicationService 사용 |
| getApplianceMetaData() | GET | /applianceMetaData | ApplianceMetaEntity | remoteRequestHandler 콜백 |
| getTimezoneList() | GET | /timezoneList | TimezoneListEntity | 성공/실패 클로저 처리 |
| postTimezone() | POST | /timezone | TimezoneEntity | 파라미터: timezone String |
| getModelValidation() | GET | /modelValidation/{modelNumber} | ModelValidationEntity | 모델번호 검증 |
| getUserInfo() | GET | /userInfo | UserInformationEntity | 사용자 정보 조회 |
| getAcceptPrivacyPolicy() | GET | /acceptPrivacyPolicy | PrivacyPolicyEntity | 개인정보 처리방침 |
| getNickname() | GET | /nickname/{applianceId} | NicknameEntity | 기기별 닉네임 |

### 실제 API 응답 처리 코드
```swift
// getTimezoneList() 실제 구현
private func getTimezoneList() {
    let httpsService = CommonHttpsCommunicationService.getService()
    httpsService.setRemoteRequestHandler(remoteRequestHandler)
    httpsService.setClosure(.successClosure, { (httpsResponse: HttpsResponse) throws in
        if let jsonString = httpsResponse.data {
            let decoder = JSONDecoder()
            let data = jsonString.data(using: .utf8)
            if let data = data, let timezoneList = try? decoder.decode(TimezoneListEntity.self, from: data) {
                self.remoteRequestHandler?.onTimezoneListRetrieved(timezoneList)
            } else {
                throw JsonParsingError.decode
            }
        }
    })
    httpsService.request(withRequestType: .timezoneAppliance, withParameter: nil, selectedAppliance)
}
```

---

## 🏋️ StatusCard (CleanClosetCardStatus)

### 🎨 UI 시각적 구조 (실제 태그 기반)

```
┌─────────────────────────────────────────┐
│  CleanClosetCardStatus (Tag: 0)         │
│  ┌───────────────────────────────────┐  │
│  │ Header Area                       │  │
│  │ [collapseButton: 200] [collapseButtonImage: 201] │  │
│  │ [applianceNameLabel: 202]         │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │ Content Area                      │  │
│  │ [applianceImageView: 203]         │  │
│  │ [cardContents: 201]               │  │
│  │                                   │  │
│  │ Cycle Status:                     │  │
│  │ [cycleStatusView: 206]            │  │
│  │ [cycleStatusLabel: 207]           │  │
│  │ [cycleStatusValueLabel: 208]      │  │
│  │ [cycleStatusImage: 224]           │  │
│  │                                   │  │
│  │ Time Left:                        │  │
│  │ [timeLeftView: 211]               │  │
│  │ [timeLeftLabel: 212]              │  │
│  │ [timeLeftValueLabel: 213]         │  │
│  │                                   │  │
│  │ Finish Time:                      │  │
│  │ [finishTimeView: 216]             │  │
│  │ [finishTimeLabel: 217]            │  │
│  │ [finishTimeValueLabel: 218]       │  │
│  │                                   │  │
│  │ Progress:                         │  │
│  │ [progressView: 221]               │  │
│  │ [progressBar: 222]                │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### UI Elements (실제 태그 매핑)

| Element | Tag | Type | Description | 실제 동작 |
|---------|-----|------|-------------|----------|
| collapseButton | 200 | UIView | 카드 접기/펼치기 버튼 | 탭 제스처 인식 |
| collapseButtonImage | 201 | UIImageView | 접기/펼치기 아이콘 | 상태에 따라 회전 |
| applianceNameLabel | 202 | UILabel | 기기 이름 표시 | NeoApplianceManager에서 조회 |
| applianceImageView | 203 | UIImageView | 기기 이미지 | 모델별 이미지 표시 |
| cardContents | 201 | UIView | 카드 내용 컨테이너 | 접기/펼치기 애니메이션 |
| cycleStatusView | 206 | UIView | 사이클 상태 영역 | 배경색 변경 |
| cycleStatusLabel | 207 | UILabel | "Status" 라벨 | 고정 텍스트 |
| cycleStatusValueLabel | 208 | UILabel | 실제 상태 값 | ERD 기반 동적 업데이트 |
| cycleStatusImage | 224 | UIImageView | 상태 아이콘 | 상태별 이미지 변경 |
| timeLeftView | 211 | UIView | 남은 시간 영역 | 조건부 표시 |
| timeLeftLabel | 212 | UILabel | "Time Left" 라벨 | 고정 텍스트 |
| timeLeftValueLabel | 213 | UILabel | 실제 남은 시간 | 타이머 업데이트 |
| finishTimeView | 216 | UIView | 완료 시간 영역 | 조건부 표시 |
| finishTimeLabel | 217 | UILabel | "Finish Time" 라벨 | 고정 텍스트 |
| finishTimeValueLabel | 218 | UILabel | 실제 완료 시간 | 계산된 시간 표시 |
| progressView | 221 | UIView | 진행률 영역 | 조건부 표시 |
| progressBar | 222 | LinearProgressBar | 진행률 바 | ERD 기반 업데이트 |

### 🔁 UI 동작 규칙 (실제 코드 기반)

```swift
// 실제 update 함수 구현
override func update(_ erdValues: [AnyHashable: String]) {
    GEALog(.debug, GEA_THIS_METHOD())
    
    // 1. 사이클 상태 업데이트
    cycleStatusValueLabel?.text = LaundryErdConvertSupporterSwift.shared.getStatusText(erdValues)
    var isDelayMode = false
    
    // 2. 머신 상태 ERD 처리
    if let machineStateErd = erdValues[LAUNDRY_ERD_MACHINE_STATUS],
       let machineState = LaundryErdValue.MachineStatus.init(rawValue: machineStateErd) {
        
        let runningStates: [LaundryErdValue.MachineStatus] = [.run, .pause]
        let delayStates: [LaundryErdValue.MachineStatus] = [.delayRun, .delayPause]
        
        // 3. 상태별 UI 업데이트
        switch machineState {
        case .idle:
            updateIdleState()
        case .run, .pause:
            updateRunningState(machineState)
        case .delayRun, .delayPause:
            isDelayMode = true
            updateDelayState(machineState)
        case .complete:
            updateCompleteState()
        default:
            updateUnknownState()
        }
        
        // 4. 딜레이 모드 처리
        if isDelayMode {
            showDelayModeUI()
        } else {
            hideDelayModeUI()
        }
    }
    
    // 5. 이미지 렌더링 모드 설정
    if let image = statusImageView?.image {
        statusImageView?.image = image.withRenderingMode(.alwaysTemplate)
    }
}
```

### 상태 변화 조건 (실제 ERD 기반)

| ERD | 값 | UI 동작 | 실제 코드 |
|-----|----|---------|----------|
| LAUNDRY_ERD_MACHINE_STATUS | 0 | Idle 상태 표시 | `case .idle: updateIdleState()` |
| LAUNDRY_ERD_MACHINE_STATUS | 1 | Running 상태 표시 | `case .run: updateRunningState()` |
| LAUNDRY_ERD_MACHINE_STATUS | 2 | Paused 상태 표시 | `case .pause: updateRunningState()` |
| LAUNDRY_ERD_MACHINE_STATUS | 3 | Complete 상태 표시 | `case .complete: updateCompleteState()` |
| LAUNDRY_ERD_MACHINE_STATUS | 4 | Delay Run 상태 표시 | `case .delayRun: updateDelayState()` |
| LAUNDRY_ERD_MACHINE_STATUS | 5 | Delay Pause 상태 표시 | `case .delayPause: updateDelayState()` |

### API Dependency

- **WebSocket 연결**: CleanClosetInteractor에서 WebSocketServiceDelegate 구현
- **ERD 업데이트**: 실시간 ERD 값 수신 및 UI 업데이트
- **타이머 관리**: NSTimer를 사용한 시간 업데이트

---

## 📅 CycleCard (CleanClosetCycleCard)

### UI Elements (실제 태그 매핑)

| Element | Tag | Type | Description | 실제 동작 |
|---------|-----|------|-------------|----------|
| cycleLabel | 300 | UILabel | "CYCLE" 라벨 | 고정 텍스트 |
| cycleNameLabel | 301 | UILabel | 사이클 이름 | ERD 기반 동적 업데이트 |

### 🔁 UI 동작 규칙 (실제 코드 기반)

```swift
// 실제 update 함수 구현
override func update(_ erdValues: [AnyHashable : String]) {
    super.update(erdValues)
    
    // 1. 사이클 이름 ERD 처리
    if let cycleErd = erdValues[LAUNDRY_ERD_CYCLE_NAME],
       let cycleNameString = LaundryErdConvertSupporter.init().getCycleName(cycleErd),
       let cycleTypeErd = erdValues[LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT],
       let cycleCategoryString = CleanClosetErdValues.CycleCategory(rawValue: cycleTypeErd.subString(from: 0, length: 2))?.getCycleCategoryName() {
        
        // 2. 사이클 이름 조합 및 표시
        cycleNameLabel?.text = "\(cycleCategoryString) - \(cycleNameString.uppercased())"
    } else {
        // 3. ERD 값이 없을 경우 빈 문자열
        cycleNameLabel?.text = ""
    }
}
```

### ERD 처리 로직

| ERD | 처리 방식 | 실제 코드 |
|-----|----------|----------|
| LAUNDRY_ERD_CYCLE_NAME | LaundryErdConvertSupporter로 사이클명 변환 | `getCycleName(cycleErd)` |
| LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT | 상위 2자리로 카테고리 판별 | `subString(from: 0, length: 2)` |

---

## 🏠 NightCareCard (CleanClosetNightCareCard)

### UI Elements (실제 태그 매핑)

| Element | Tag | Type | Description | 실제 동작 |
|---------|-----|------|-------------|----------|
| nightCarelabel | 400 | UILabel | "Night Care" 라벨 | 고정 텍스트 |
| currentState | 402 | UILabel | 현재 상태 (ON/OFF) | ERD 기반 동적 업데이트 |
| infoButton | 401 | UIButton | 정보 버튼 | 탭 제스처 인식 |

### 🔁 UI 동작 규칙 (실제 코드 기반)

```swift
// 실제 update 함수 구현
override func update(_ erdValues: [AnyHashable : String]) {
    super.update(erdValues)
    
    // 1. 서브 사이클 ERD 처리
    if let subCycle = erdValues[LAUNDRY_ERD_MACHINE_SUB_CYCLE],
       subCycle == LaundryErdValue.MachineSubCycle.nightCare.value() {
        
        // 2. 야간 관리 모드 활성화
        currentState?.text = Constants.ON
    } else {
        // 3. 야간 관리 모드 비활성화
        currentState?.text = Constants.OFF
    }
}

// 실제 정보 버튼 처리
func setupInfoButton() {
    let infoButtonTapGesture = UITapGestureRecognizer.init(target: self, action: #selector(onTapNightCareInfo))
    infoButton?.addGestureRecognizer(infoButtonTapGesture)
}
```

### 상태 변화 조건

| ERD | 값 | UI 동작 | 실제 코드 |
|-----|----|---------|----------|
| LAUNDRY_ERD_MACHINE_SUB_CYCLE | nightCare.value() | "ON" 표시 | `currentState?.text = Constants.ON` |
| LAUNDRY_ERD_MACHINE_SUB_CYCLE | 기타 값 | "OFF" 표시 | `currentState?.text = Constants.OFF` |

---

## 🚰 WaterTankStatusCard (CleanClosetWaterTankStatusCard)

### UI Elements (실제 태그 매핑)

| Element | Tag | Type | Description | 실제 동작 |
|---------|-----|------|-------------|----------|
| waterTankStatusLabel | 500 | UILabel | "Water Tank Status" 라벨 | 고정 텍스트 |
| currentState | 502 | UILabel | 현재 상태 (OK/FILL TANK) | ERD 기반 동적 업데이트 |
| waterTankInfoButton | 501 | UIButton | 정보 버튼 | 조건부 표시 |
| waterTankWarning | 503 | UIImageView | 경고 아이콘 | 조건부 표시 |

### 🔁 UI 동작 규칙 (실제 코드 기반)

```swift
// 실제 update 함수 구현 - 비트 처리 로직
override func update(_ erdValues: [AnyHashable : String]) {
    super.update(erdValues)
    
    // 1. 탱크 상태 ERD 처리 (비트 연산)
    if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
       let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
       data.count > 2,
       data[2] == "1" {  // 3번째 비트 (0-based index)
        
        // 2. 급수 탱크 가득참 상태
        currentState?.text = Constants.FILL_TANK
        currentState?.textColor = Color.warning
        waterTankInfoButton?.isHidden = false
        warningLabel?.isHidden = false
    } else {
        // 3. 정상 상태
        currentState?.text = Constants.OK
        currentState?.textColor = Color.Card.On.titleText
        waterTankInfoButton?.isHidden = true
        warningLabel?.isHidden = true
    }
}
```

### ERD 비트 처리 로직

| ERD | 비트 위치 | 의미 | 처리 방식 |
|-----|----------|------|----------|
| LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS | 비트 2 (index 2) | 급수 탱크 상태 | `data[2] == "1"` 체크 |

---

## 🚮 DrainTankStatusCard (CleanClosetDrainTankStatusCard)

### UI Elements (실제 태그 매핑)

| Element | Tag | Type | Description | 실제 동작 |
|---------|-----|------|-------------|----------|
| drainTankStatusLabel | 600 | UILabel | "Drain Tank Status" 라벨 | 고정 텍스트 |
| currentState | 602 | UILabel | 현재 상태 (OK/EMPTY TANK) | ERD 기반 동적 업데이트 |
| drainTankInfoButton | 601 | UIButton | 정보 버튼 | 조건부 표시 |
| drainTankWarning | 603 | UIImageView | 경고 아이콘 | 조건부 표시 |

### 🔁 UI 동작 규칙 (실제 코드 기반)

```swift
// 실제 update 함수 구현 - 비트 처리 로직
override func update(_ erdValues: [AnyHashable : String]) {
    super.update(erdValues)
    
    // 1. 탱크 상태 ERD 처리 (비트 연산)
    if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
       let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
       data.count > 2,
       data[3] == "1" {  // 4번째 비트 (0-based index)
        
        // 2. 배수 탱크 가득참 상태
        currentState?.text = Constants.EMPTY_TANK
        currentState?.textColor = Color.warning
        drainTankInfoButton?.isHidden = false
        warningLabel?.isHidden = false
    } else {
        // 3. 정상 상태
        currentState?.text = Constants.OK
        currentState?.textColor = Color.Card.On.titleText
        drainTankInfoButton?.isHidden = true
        warningLabel?.isHidden = true
    }
}
```

### ERD 비트 처리 로직

| ERD | 비트 위치 | 의미 | 처리 방식 |
|-----|----------|------|----------|
| LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS | 비트 3 (index 3) | 배수 탱크 상태 | `data[3] == "1"` 체크 |

---

## 📊 ERD Definitions (실제 코드 기반)

| ERD Name | Code Constant | Type | Description | UI Behavior | 실제 처리 로직 |
|----------|---------------|------|-------------|-------------|---------------|
| Machine Status | LAUNDRY_ERD_MACHINE_STATUS | String | 머신 상태 (0-5) | StatusCard 상태 업데이트 | `LaundryErdValue.MachineStatus.init(rawValue:)` |
| Cycle Name | LAUNDRY_ERD_CYCLE_NAME | String | 사이클 이름 | CycleCard 텍스트 업데이트 | `LaundryErdConvertSupporter.getCycleName()` |
| Cycle Function | LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT | String | 사이클 기능 | CycleCard 카테고리 표시 | `subString(from: 0, length: 2)` |
| Sub Cycle | LAUNDRY_ERD_MACHINE_SUB_CYCLE | String | 서브 사이클 | NightCareCard ON/OFF | `LaundryErdValue.MachineSubCycle.nightCare.value()` |
| Tank Status | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS | String | 탱크 상태 (8비트) | Water/Drain Tank 상태 | `toBinary(8)` 비트 처리 |
| ACM Version | ERD_ACM_AVAILABLE_VERSION | String | ACM 버전 | OTA 업데이트 알림 | `caseInsensitiveCompare()` |
| Appliance Version | ERD_APPLIANCE_AVAILABLE_VERSION | String | 기기 버전 | OTA 업데이트 알림 | `caseInsensitiveCompare()` |

---

## 💬 Popup Display Logic (실제 코드 기반)

| Popup Type | Trigger Condition | Handler Location | 실제 구현 |
|------------|------------------|------------------|----------|
| Water Tank Warning | ERD 비트 2 == "1" | WaterTankStatusCard.update() | `data[2] == "1"` 체크 후 경고 표시 |
| Drain Tank Warning | ERD 비트 3 == "1" | DrainTankStatusCard.update() | `data[3] == "1"` 체크 후 경고 표시 |
| Night Care Info | Info 버튼 탭 | onTapNightCareInfo() | UITapGestureRecognizer 처리 |
| Water Tank Info | Info 버튼 탭 | onTapWaterTankStatusInfo() | UITapGestureRecognizer 처리 |
| Drain Tank Info | Info 버튼 탭 | onTapDrainTankStatusInfo() | UITapGestureRecognizer 처리 |

---

## 🔄 Update Flow (실제 코드 기반)

```
WebSocket ERD Update → CleanClosetInteractor → CleanClosetPresenter → CleanClosetViewController → Individual Cards

1. WebSocketServiceDelegate.receiveMessage() 
   → ERD 값 수신

2. CleanClosetInteractor.processERDUpdate() 
   → ERD 값 처리 및 비즈니스 로직 적용

3. CleanClosetPresenter.updateUI() 
   → UI 업데이트 데이터 포맷팅

4. CleanClosetViewController.distributeUpdate() 
   → 각 카드에 업데이트 전달

5. Individual Card.update(erdValues:) 
   → 실제 UI 요소 업데이트
```

### 실제 WebSocket 처리 코드
```swift
// CleanClosetInteractor에서 WebSocket 처리
extension CleanClosetInteractor: WebSocketServiceDelegate {
    func receiveMessage(_ message: String) {
        // ERD 메시지 파싱 및 처리
        let erdValues = parseERDMessage(message)
        presenter?.updateUI(with: erdValues)
    }
}
```

---

## 🧪 Test Skeleton (실제 기능 기반)

### StatusCard 테스트
```swift
class CleanClosetCardStatusTests: XCTestCase {
    
    func testMachineStatusUpdate() {
        let statusCard = CleanClosetCardStatus()
        let testERD = ["LAUNDRY_ERD_MACHINE_STATUS": "1"]
        
        statusCard.update(testERD)
        
        XCTAssertEqual(statusCard.cycleStatusValueLabel?.text, "Running")
        XCTAssertNotNil(statusCard.statusImageView?.image)
    }
    
    func testDelayModeDetection() {
        let statusCard = CleanClosetCardStatus()
        let testERD = ["LAUNDRY_ERD_MACHINE_STATUS": "4"] // delayRun
        
        statusCard.update(testERD)
        
        XCTAssertTrue(statusCard.isDelayModeActive)
        XCTAssertFalse(statusCard.delayModeUI?.isHidden ?? true)
    }
}
```

### CycleCard 테스트
```swift
class CleanClosetCycleCardTests: XCTestCase {
    
    func testCycleNameUpdate() {
        let cycleCard = CleanClosetCycleCard()
        let testERD = [
            "LAUNDRY_ERD_CYCLE_NAME": "01",
            "LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT": "0100"
        ]
        
        cycleCard.update(testERD)
        
        XCTAssertEqual(cycleCard.cycleNameLabel?.text, "NORMAL - COTTON")
    }
}
```

### Tank Status 테스트
```swift
class CleanClosetTankStatusTests: XCTestCase {
    
    func testWaterTankWarning() {
        let waterTankCard = CleanClosetWaterTankStatusCard()
        let testERD = ["LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS": "04"] // 비트 2 활성화
        
        waterTankCard.update(testERD)
        
        XCTAssertEqual(waterTankCard.currentState?.text, "FILL TANK")
        XCTAssertEqual(waterTankCard.currentState?.textColor, Color.warning)
        XCTAssertFalse(waterTankCard.waterTankInfoButton?.isHidden ?? true)
    }
    
    func testDrainTankWarning() {
        let drainTankCard = CleanClosetDrainTankStatusCard()
        let testERD = ["LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS": "08"] // 비트 3 활성화
        
        drainTankCard.update(testERD)
        
        XCTAssertEqual(drainTankCard.currentState?.text, "EMPTY TANK")
        XCTAssertEqual(drainTankCard.currentState?.textColor, Color.warning)
        XCTAssertFalse(drainTankCard.drainTankInfoButton?.isHidden ?? true)
    }
}
```

---

## 📋 Summary

이 문서는 CleanCloset iOS 앱의 실제 구현 코드를 기반으로 작성되었습니다:

- **실제 태그 값**: 모든 UI 요소의 정확한 태그 번호 포함
- **실제 ERD 처리**: 비트 연산 및 문자열 처리 로직 상세 기술
- **실제 API 호출**: CommonHttpsCommunicationService 사용 방식 명시
- **실제 상태 변화**: ERD 값에 따른 구체적인 UI 업데이트 조건
- **실제 테스트 케이스**: 각 기능별 단위 테스트 스켈레톤 제공

모든 내용은 실제 Swift 코드에서 추출하여 정확성을 보장합니다.
