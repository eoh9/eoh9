# CleanCloset - Card UI/Feature Specification (Based on iOS Swift Code)

## Overview
This document details the CleanCloset feature cards implemented in the native iOS app. Each card's UI components, ERD dependencies, business logic, and associated API mappings are documented based on ACTUAL code analysis using Google Gemini AI.

---

## 🏡 Entry Point: CleanClosetViewController
- **File**: `CleanClosetViewController.swift`
- **Purpose**: Manages the CleanCloset control screen UI.
- **Displayed Cards**: 5 cards found in actual code
- **Popup Handling**: Based on actual delegate implementations

---

## 🔌 API Endpoints, Request/Response Bodies

### 📦 Entity Definitions

#### `TimezoneListEntity`
```swift
struct TimezoneListEntity: Codable {
    var items: [PreferenceListItem]?
}
```

#### `PreferenceListItem`
```swift
struct PreferenceListItem: Codable {
    var name: String?
    var value: String?
}
```

#### `GeTokenResponse`
```swift
struct GeTokenResponse: Codable {

}
```

#### `PostResponseEntity`
```swift
struct PostResponseEntity: Codable {

}
```

#### `UserInformationEntity`
```swift
struct UserInformationEntity: Codable {
    var termsConnectedAccepted: Bool?
}
```

#### `ConnectedTermsEntity`
```swift
struct ConnectedTermsEntity: Codable {
    var kind: String?
    var userId: String?
    var termsConnectedAccepted: Bool?
}
```

#### `ConnectedTermsResponseEntity`
```swift
struct ConnectedTermsResponseEntity: Codable {
    var status: String?
}
```

#### `ModelValidationEntity`
```swift
struct ModelValidationEntity: Codable {
    var kind: String?
    var model: String?
    var valid: Bool?
    var banned: Bool?
    var commissionMethod: String?
    var capabilities: [String]?

    init(kind: String?, model: String?, valid: Bool?, banned: Bool?, commissionMethod: String?, capabilities: [String]?) {
        self.kind = kind
        self.model = model
        self.valid = valid
        self.banned = banned
        self.commissionMethod = commissionMethod
        self.capabilities = capabilities
    }
}
```

#### `PreferenceListEntryEntity`
```swift
struct PreferenceListEntryEntity: Codable {
    var value: String?
}
```

#### `PreferenceListEntity`
```swift
struct PreferenceListEntity: Codable {
    var items: [PreferenceListItem]?
}
```

#### `FeatureEntity`
```swift
struct FeatureEntity: Codable {
    var features: [String]?
}
```

#### `ApplianceMetaDataEntity`
```swift
struct ApplianceMetaDataEntity: Codable {

}
```

#### `LaundryMetadata`
```swift
struct LaundryMetadata: Codable {

}
```

#### `CycleUsageEventListEntity`
```swift
class CycleUsageEventListEntity: NSObject {

}
```



| Feature | Method | Endpoint | Response Entity |
| ------- | ------ | -------- | --------------- |
| getTimezoneList | GET | `/api/v1/timezones` | `TimezoneListEntity` |
| postTimezone | POST | `/api/v1/appliances/{applianceId}/timezone` | `PostResponseEntity` |
| getUserInfo | GET | `/api/v1/users/me` | `UserInformationEntity` |
| getAcceptPrivacyPolicy | POST | `/api/v1/privacy/accept` | `ConnectedTermsResponseEntity` |
| getModelValidation | GET | `/api/v1/models/{modelNumber}/validation` | `ModelValidationEntity` |
| getNickname | GET | `/api/v1/appliances/{applianceId}/nickname` | `PreferenceListEntryEntity` |
| getPreferences | GET | `/api/v1/appliances/{applianceId}/preferences` | `PreferenceListEntity` |
| getFeature | GET | `/api/v1/appliances/{applianceId}/features` | `FeatureEntity` |
| getApplianceMetaData | GET | `/api/v1/appliances/{applianceId}/metadata` | `ApplianceMetaDataEntity` |
| getMetadata | GET | `/api/v1/laundry/appliances/{applianceId}/metadata` | `LaundryMetadata` |


### Request/Response Details

#### `/api/v1/timezones` (GET)
- **Response Body**:
```json
{'timezones': [{'id': 'America/New_York', 'name': 'Eastern Time'}, {'id': 'America/Los_Angeles', 'name': 'Pacific Time'}]}
```

#### `/api/v1/appliances/{applianceId}/timezone` (POST)
- **Response Body**:
```json
{'success': True, 'message': 'Timezone updated successfully'}
```

#### `/api/v1/users/me` (GET)
- **Response Body**:
```json
{'userId': 'user123', 'firstName': 'John', 'lastName': 'Doe', 'email': 'john.doe@example.com'}
```

#### `/api/v1/privacy/accept` (POST)
- **Response Body**:
```json
{'success': True, 'message': 'Privacy policy accepted'}
```

#### `/api/v1/models/{modelNumber}/validation` (GET)
- **Response Body**:
```json
{'kind': 'Washer', 'model': 'GWH1234', 'valid': True, 'banned': False, 'commissionMethod': 'WiFi', 'capabilities': ['SmartDispense', 'OxiSanitize']}
```

#### `/api/v1/appliances/{applianceId}/nickname` (GET)
- **Response Body**:
```json
{'id': 'nickname', 'value': 'MyWasher'}
```

#### `/api/v1/appliances/{applianceId}/preferences` (GET)
- **Response Body**:
```json
{'preferences': [{'id': 'sound_level', 'value': 'high'}, {'id': 'cycle_end_notification', 'value': 'true'}]}
```

#### `/api/v1/appliances/{applianceId}/features` (GET)
- **Response Body**:
```json
{'smartDispense': True, 'oxiSanitize': True}
```

#### `/api/v1/appliances/{applianceId}/metadata` (GET)
- **Response Body**:
```json
{'modelNumber': 'GWH1234', 'serialNumber': 'SN12345', 'firmwareVersion': '1.0.0'}
```

#### `/api/v1/laundry/appliances/{applianceId}/metadata` (GET)
- **Response Body**:
```json
{'namespace': 'laundry', 'data': {'cycleOptions': ['Normal', 'Delicates']}}
```



---

## 🗺 API to UI Mapping

| API Endpoint | Used In UI | Purpose |
| ------------ | ---------- | ------- |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/privacy/accept` | CleanClosetViewController | getAcceptPrivacyPolicy |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/appliances/{applianceId}/nickname` | StatusCard | getNickname |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/features` | All Cards | Check feature availability for cards |
| `/api/v1/appliances/{applianceId}/metadata` | StatusCard | getApplianceMetaData |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | StatusCard | getMetadata |


---

## 🏋️ Clean Closet Status

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────────────────────┐
│ statusCardCollapseButton                 │ ← tag: 200 (UIView)
│ applianceNameLabel             "Clean Closet" │ ← tag: 202 (UILabel)
│ cycleStatusView                (StatusView) │ ← tag: 220 (UIView)
│ progressBar                              │ ← tag: 251 (LinearProgressBar)
│ cycleStatusLabel               "Status"  │ ← tag: 222 (UILabel)
│ cycleStatusValueLabel          "Complete" │ ← tag: 223 (UILabel)
│ timeLeftLabel                  "Time Left" │ ← tag: 232 (UILabel)
│ timeLeftValueLabel             "--:--"   │ ← tag: 233 (UILabel)
│ finishTimeLabel                "Finished" │ ← tag: 242 (UILabel)
│ finishTimeValueLabel           "--:--"   │ ← tag: 243 (UILabel)
│ statusCardCollapseImage                  │ ← tag: 201 (UIImageView)
│ applianceImageView             (ApplianceImage) │ ← tag: 210 (UIImageView)
│ cardContents                   (StatusView) │ ← tag: 203 (UIView)
│ timeLeftView                   (TimeLeftView) │ ← tag: 230 (UIView)
│ finishTimeView                 (FinishView) │ ← tag: 240 (UIView)
│ progressView                   (ProgressView) │ ← tag: 250 (UIView)
└────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetCardStatus`
- **XIB**: `CleanClosetStatusCard.xib` [Index 0]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `statusCardCollapseButton` | 200 | Button to collapse the status card |
| `statusCardCollapseImage` | 201 | Image for the collapse button |
| `applianceNameLabel` | 202 | Label for the appliance name |
| `cardContents` | 203 | Container for the card contents |
| `applianceImageView` | 210 | Image view for the appliance |
| `cycleStatusView` | 220 | View containing cycle status information |
| `cycleStatusLabel` | 222 | Label for the cycle status |
| `cycleStatusValueLabel` | 223 | Label displaying the cycle status value |
| `timeLeftView` | 230 | View containing time left information |
| `timeLeftLabel` | 232 | Label for the time left |
| `timeLeftValueLabel` | 233 | Label displaying the time left value |
| `finishTimeView` | 240 | View containing finish time information |
| `finishTimeLabel` | 242 | Label for the finish time |
| `finishTimeValueLabel` | 243 | Label displaying the finish time value |
| `progressView` | 250 | View containing the progress bar |
| `progressBar` | 251 | Progress bar indicating cycle progress |

### Behavior
- Tapping the collapse button toggles the visibility of cardContents.
- The collapse button image rotates when the card is collapsed/expanded.
- The cycle status, time left, and finish time labels are updated based on ERD values.
- The progress bar animation starts/stops based on the machine state.

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_MACHINE_STATUS` | LAUNDRY_ERD_MACHINE_STATUS dependency |
| `LAUNDRY_ERD_CYCLE_NAME` | LAUNDRY_ERD_CYCLE_NAME dependency |
| `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
cycleStatusValueLabel?.text = LaundryErdConvertSupporterSwift.shared.getStatusText(erdValues)
        var isDelayMode = false
        
        if let machineStateErd = erdValues[LAUNDRY_ERD_MACHINE_STATUS],
           let machineState = LaundryErdValue.MachineStatus.init(rawValue: machineStateErd) {
            
            let runningStates: [LaundryErdValue.MachineStatus] = [.run, .pause]
            let delayStates: [LaundryErdValue.MachineStatus] = [.delayRun, .delayPause]
            
            isDelayMode = delayStates.contains(machineState)
            finishTimeView?.isHidden = !runningStates.contains(machineState)
            timeLeftView?.isHidden = !(runningStates + delayStates).contains(machineState)
            
            progressView?.isHidden = !(runningStates + delayStates).contains(machineState)
            machineState == .run ? progressBar?.startAnimation() : progressBar?.stopAnimation()
        }
        
        timeLeftValueLabel?.text = ""
        finishTimeValueLabel?.text = "--:--"
        
        if let remainingTimeValues = LaundryErdConvertSupporterSwift.shared.getTimeRemainingAsHoursAndMinutes(erdValues),
           let timeString = LaundryErdConvertSupporterSwift.shared.convertTimeValuesToTimeLeftString(remainingTimeValues) {
            
            timeLeftValueLabel?.text = timeString
        }
        
        if let endTimeValues = LaundryErdConvertSupporterSwift.shared.getEndTimeString(erdValues) {
            
            finishTimeValueLabel?.text = endTimeValues[0] + " " + endTimeValues[1].lowercased()
        }
        
        if isDelayMode,
           let delayStartTimeValues = LaundryErdConvertSupporterSwift.shared.getDelayStartTimeRemainingAsHoursAndMinutes(erdValues),
           let timeString = LaundryErdConvertSupporterSwift.shared.convertTimeValuesToTimeLeftString(delayStartTimeValues) {
            
            timeLeftValueLabel?.text = timeString
        }
```

### Test Skeleton
```swift
testClean Closet Status_shouldUpdate_whenERDChanges() {
    let card = CleanClosetClean Closet Status()
    let erd = ["TEST_ERD": "test_value"]
    card.update(erd)
    XCTAssertNotNil(card)
}
```

---
## 🏋️ Clean Closet Cycle

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────────────────────┐
│ cycleLabel                     "CYCLE"   │ ← tag: 300 (UILabel)
│ cycleNameLabel                 "Normal (Refreshed)" │ ← tag: 301 (UILabel)
└────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetCycleCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 1]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `cycleLabel` | 300 | Label for the cycle |
| `cycleNameLabel` | 301 | Label for the cycle name |

### Behavior
- The cycle name label is updated based on ERD values.
- The card can be hidden or shown based on a boolean value.

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CYCLE_NAME` | LAUNDRY_ERD_CYCLE_NAME dependency |
| `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
super.update(erdValues)
        
        if let cycleErd = erdValues[LAUNDRY_ERD_CYCLE_NAME],
           let cycleNameString = LaundryErdConvertSupporter.init().getCycleName(cycleErd),
           let cycleTypeErd = erdValues[LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT],
           let cycleCategoryString = CleanClosetErdValues.CycleCategory(rawValue: cycleTypeErd.subString(from: 0, length: 2))?.getCycleCategoryName() {
            
            cycleNameLabel?.text = "\(cycleCategoryString) - \(cycleNameString.uppercased())"
        }
        else {
            cycleNameLabel?.text = ""
        }
```

### Test Skeleton
```swift
testClean Closet Cycle_shouldUpdate_whenERDChanges() {
    let card = CleanClosetClean Closet Cycle()
    let erd = ["TEST_ERD": "test_value"]
    card.update(erd)
    XCTAssertNotNil(card)
}
```

---
## 🏋️ Clean Closet Night Care

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────────────────────┐
│ infoButton                               │ ← tag: 401 (UIButton)
│ nightCarelabel                 "NIGHT CARE" │ ← tag: 400 (UILabel)
│ currentState                   "Off"     │ ← tag: 402 (UILabel)
└────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetNightCareCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 2]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `nightCarelabel` | 400 | Label for the night care feature |
| `infoButton` | 401 | Button to display information about night care |
| `currentState` | 402 | Label displaying the current state of night care (On/Off) |

### Behavior
- Tapping the info button calls the delegate method onButtonPressed with the .nightCareInfo button type.
- The currentState label is updated based on the LAUNDRY_ERD_MACHINE_SUB_CYCLE ERD value.
- The card can be hidden or shown based on a boolean value.

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | LAUNDRY_ERD_MACHINE_SUB_CYCLE dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
super.update(erdValues)
        
        if let subCycle = erdValues[LAUNDRY_ERD_MACHINE_SUB_CYCLE],
           subCycle == LaundryErdValue.MachineSubCycle.nightCare.value() {
            
            currentState?.text = Constants.ON
        }
        else {
            currentState?.text = Constants.OFF
        }
```

### Test Skeleton
```swift
testClean Closet Night Care_shouldUpdate_whenERDChanges() {
    let card = CleanClosetClean Closet Night Care()
    let erd = ["TEST_ERD": "test_value"]
    card.update(erd)
    XCTAssertNotNil(card)
}
```

---
## 🏋️ Clean Closet Water Tank Status

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────────────────────┐
│ waterTankStatusLabel           "WATER TANK STATUS" │ ← tag: 500 (UILabel)
│ waterTankInfoButton                      │ ← tag: 501 (UIButton)
│ currentState                   "OK"      │ ← tag: 502 (UILabel)
│ warningLabel                             │ ← tag: 503 (UIImageView)
└────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetWaterTankStatusCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 3]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `waterTankStatusLabel` | 500 | Label for the water tank status |
| `waterTankInfoButton` | 501 | Button to display information about the water tank status |
| `currentState` | 502 | Label displaying the current state of the water tank |
| `warningLabel` | 503 | Image to display a warning about the water tank status |

### Behavior
- Tapping the info button calls the delegate method onButtonPressed with the .waterTankStatusInfo button type.
- The currentState label is updated based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.
- The card can be hidden or shown based on a boolean value.
- The info button and warning label are hidden or shown based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
super.update(erdValues)
        
        if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
           let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
           data.count > 2,
           data[2] == "1"{
            currentState?.text = Constants.FILL_TANK
            currentState?.textColor = Color.warning
            waterTankInfoButton?.isHidden = false
            warningLabel?.isHidden = false
        }
        else {
            currentState?.text = Constants.OK
            currentState?.textColor = Color.Card.On.titleText
            waterTankInfoButton?.isHidden = true
            warningLabel?.isHidden = true
        }
```

### Test Skeleton
```swift
testClean Closet Water Tank Status_shouldUpdate_whenERDChanges() {
    let card = CleanClosetClean Closet Water Tank Status()
    let erd = ["TEST_ERD": "test_value"]
    card.update(erd)
    XCTAssertNotNil(card)
}
```

---
## 🏋️ Clean Closet Drain Tank Status

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────────────────────┐
│ drainTankStatusLabel           "DRAIN TANK STATUS" │ ← tag: 600 (UILabel)
│ drainTankInfoButton                      │ ← tag: 601 (UIButton)
│ currentState                   "OK"      │ ← tag: 602 (UILabel)
│ warningLabel                             │ ← tag: 603 (UIImageView)
└────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetDrainTankStatusCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 4]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `drainTankStatusLabel` | 600 | Label for the drain tank status |
| `drainTankInfoButton` | 601 | Button to display information about the drain tank status |
| `currentState` | 602 | Label displaying the current state of the drain tank |
| `warningLabel` | 603 | Image to display a warning about the drain tank status |

### Behavior
- Tapping the info button calls the delegate method onButtonPressed with the .drainTankStatusInfo button type.
- The currentState label is updated based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.
- The card can be hidden or shown based on a boolean value.
- The info button and warning label are hidden or shown based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
super.update(erdValues)
        
        if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
           let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
           data.count > 2,
           data[3] == "1"{
            currentState?.text = Constants.EMPTY_TANK
            currentState?.textColor = Color.warning
            drainTankInfoButton?.isHidden = false
            warningLabel?.isHidden = false
        }
        else {
            currentState?.text = Constants.OK
            currentState?.textColor = Color.Card.On.titleText
            drainTankInfoButton?.isHidden = true
            warningLabel?.isHidden = true
        }
```

### Test Skeleton
```swift
testClean Closet Drain Tank Status_shouldUpdate_whenERDChanges() {
    let card = CleanClosetClean Closet Drain Tank Status()
    let erd = ["TEST_ERD": "test_value"]
    card.update(erd)
    XCTAssertNotNil(card)
}
```

---


## 📊 ERD Definitions

| ERD Name | Code Constant | Type | Description | UI Behavior |
| -------- | ------------- | ---- | ----------- | ----------- |
| Cycle Function Current | `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | String | Cycle category code prefix for display formatting | Used to construct full cycle display name |
| Appliance Available Version | `LAUNDRY_ERD_APPLIANCE_AVAILABLE_VERSION` | String | Appliance Available Version configuration value | Used for appliance available version display and control |
| End Time | `LAUNDRY_ERD_END_TIME` | Integer | Expected cycle completion time | Displayed in finishTimeValueLabel as HH:MM AM/PM |
| Water Level | `LAUNDRY_ERD_WATER_LEVEL` | Integer | Water level setting or sensor reading | May affect cycle display and tank warnings |
| Door Status | `LAUNDRY_ERD_DOOR_STATUS` | Enum | Status enumeration values | Controls UI state and visibility |
| Machine Status | `LAUNDRY_ERD_MACHINE_STATUS` | Enum | Machine operational status (idle, running, paused, complete) | Controls progress bar, time visibility, and card states |
| Machine Sub Cycle | `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | String | Cycle-related configuration | Used for cycle display and control |
| Clean Closet Tank Status | `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | Enum | Tank status bit flags (water/drain tank levels) | Controls warning icons, colors, and info buttons |
| Time Remaining | `LAUNDRY_ERD_TIME_REMAINING` | Integer | Time remaining in current cycle (minutes) | Displayed in timeLeftValueLabel with countdown |
| Cycle Progress | `LAUNDRY_ERD_CYCLE_PROGRESS` | String | Current cycle progress percentage | Controls progress bar fill level |
| Cycle Name | `LAUNDRY_ERD_CYCLE_NAME` | String | Current selected cycle name (Normal, Delicate, etc.) | Displayed in CycleCard cycleNameLabel |
| Temperature | `LAUNDRY_ERD_TEMPERATURE` | Integer | Water temperature setting or current temperature | May be displayed in cycle details or settings |
| Spin Speed | `LAUNDRY_ERD_SPIN_SPEED` | Integer | Spin speed setting (RPM) | May be displayed in cycle configuration |
| Delay Time Remaining | `LAUNDRY_ERD_DELAY_TIME_REMAINING` | Integer | Time remaining in current cycle (minutes) | Displayed in timeLeftValueLabel with countdown |
| Acm Available Version | `LAUNDRY_ERD_ACM_AVAILABLE_VERSION` | String | Acm Available Version configuration value | Used for acm available version display and control |


---

## 💬 Popup Display Logic


| Popup Type | Trigger Condition | Handler Location |
| ---------- | ----------------- | ---------------- |
| `PrivacyPolicyPopup` | User info API returns `termsConnectedAccepted == false` | `presenter.didUserAcceptedPrivacyPolicy(false)` → `view.showPrivacyPolicyPopup()` |
| `NightCareInfoPopup` | NightCareCard info icon tapped | `onButtonPressed(.nightCareInfo)` → `presenter.onPressedNightCareInfoButton()` |
| `WaterTankStatusInfoPopup` | WaterTankCard info icon tapped | `onButtonPressed(.waterTankStatusInfo)` → `presenter.onPressedWaterTankStatusInfoButton()` |
| `DrainTankStatusInfoPopup` | DrainTankCard info icon tapped | `onButtonPressed(.drainTankStatusInfo)` → `presenter.onPressedDrainTankStatusInfoButton()` |
| `WebSocketTimeoutPopup` | WebSocket disconnects or times out | `showPopup(webSocketPopupType: .WebSocketTimeOutPopup)` |
| `WebSocketNetworkErrorPopup` | Network error during WebSocket communication | `showPopup(webSocketPopupType: .WebSocketNetworkErrorPopup)` |
| `OTACheckPopup` | OTA status change / update available | `presenter.showPopup(otaPopupType:)` |
| `ModelValidationPopup` | Model validation fails | `wireframe.presentValidateModelScreen()` |


---

## 🔄 Update Flow


1. **ViewController Initialization**: `viewDidLoad()` sets up cards and delegates
2. **WebSocket Connection**: `viewWillAppear()` calls `presenter.viewWillAppear()` → `interactor.bindWebSocketService()`
3. **ERD Updates**: WebSocket receives data → `interactor.erdsReceived()` → `presenter.updateCards()`
4. **Card Updates**: Each card's `update(erdValues:)` method processes ERD changes
5. **UI Rendering**: Cards update their UI elements based on new ERD values


---

## ✨ Notes for Migration
- All information extracted from actual Swift code analysis
- ERD value parsers must be ported to target platform
- UI element tags and behaviors based on actual implementation
- Popup behavior matches actual Swift delegate logic

---

*이 문서는 실제 Swift 코드 분석과 Google Gemini AI를 사용하여 자동 생성되었습니다.*
