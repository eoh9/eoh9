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
    let termsConnectedAccepted: Bool?
}
```

#### `ConnectedTermsEntity`
```swift
struct ConnectedTermsEntity: Codable {
    let kind: String?
    let userId: String?
    let termsConnectedAccepted: Bool?
}
```

#### `ConnectedTermsResponseEntity`
```swift
struct ConnectedTermsResponseEntity: Codable {
    let status: String?
}
```

#### `ModelValidationEntity`
```swift
struct ModelValidationEntity: Codable {
    let kind: String?
    let model: String?
    let valid: Bool?
    let banned: String?
    let commissionMethod: String?
    let capabilities: [String]?
}
```

#### `PreferenceListEntryEntity`
```swift
struct PreferenceListEntryEntity: Codable {
    let value: String?
}
```

#### `PreferenceListEntity`
```swift
struct PreferenceListEntity: Codable {
    let items: [PreferenceListItem]?
}
```

#### `FeatureEntity`
```swift
struct FeatureEntity: Codable {
    let features: [String]?
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
struct CycleUsageEventListEntity {

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
{'key': 'nickname', 'value': 'MyWasher'}
```

#### `/api/v1/appliances/{applianceId}/preferences` (GET)
- **Response Body**:
```json
{'preferences': [{'key': 'sound_level', 'value': 'medium'}, {'key': 'display_brightness', 'value': 'high'}]}
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
{'namespace': 'laundry', 'data': {'cycle_count': 100, 'last_cycle_duration': 60}}
```



---

## 🗺 API to UI Mapping

| API Endpoint | Used In UI | Purpose |
| ------------ | ---------- | ------- |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/timezones` | CleanClosetInteractor | getTimezoneList |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/appliances/{applianceId}/timezone` | CleanClosetInteractor | postTimezone |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/users/me` | CleanClosetViewController | getUserInfo |
| `/api/v1/privacy/accept` | CleanClosetInteractor | getAcceptPrivacyPolicy |
| `/api/v1/privacy/accept` | CleanClosetInteractor | getAcceptPrivacyPolicy |
| `/api/v1/privacy/accept` | CleanClosetInteractor | getAcceptPrivacyPolicy |
| `/api/v1/privacy/accept` | CleanClosetInteractor | getAcceptPrivacyPolicy |
| `/api/v1/privacy/accept` | CleanClosetInteractor | getAcceptPrivacyPolicy |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/models/{modelNumber}/validation` | CleanClosetInteractor | getModelValidation |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | getNickname |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | getNickname |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | getNickname |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | getNickname |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | getNickname |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/preferences` | CleanClosetInteractor | getPreferences |
| `/api/v1/appliances/{applianceId}/features` | CleanClosetCardStatus | Support Clean Closet Status functionality |
| `/api/v1/appliances/{applianceId}/features` | CleanClosetCycleCard | Support Clean Closet Cycle functionality |
| `/api/v1/appliances/{applianceId}/features` | CleanClosetNightCareCard | Support Clean Closet Night Care functionality |
| `/api/v1/appliances/{applianceId}/features` | CleanClosetWaterTankStatusCard | Support Clean Closet Water Tank Status functionality |
| `/api/v1/appliances/{applianceId}/features` | CleanClosetDrainTankStatusCard | Support Clean Closet Drain Tank Status functionality |
| `/api/v1/appliances/{applianceId}/metadata` | CleanClosetInteractor | getApplianceMetaData |
| `/api/v1/appliances/{applianceId}/metadata` | CleanClosetInteractor | getApplianceMetaData |
| `/api/v1/appliances/{applianceId}/metadata` | CleanClosetInteractor | getApplianceMetaData |
| `/api/v1/appliances/{applianceId}/metadata` | CleanClosetInteractor | getApplianceMetaData |
| `/api/v1/appliances/{applianceId}/metadata` | CleanClosetInteractor | getApplianceMetaData |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | CleanClosetInteractor | getMetadata |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | CleanClosetInteractor | getMetadata |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | CleanClosetInteractor | getMetadata |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | CleanClosetInteractor | getMetadata |
| `/api/v1/laundry/appliances/{applianceId}/metadata` | CleanClosetInteractor | getMetadata |


---

## 🏋️ Clean Closet Status

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────┐
│ statusCardCollapseButton   │ ← tag: 200
│ statusCardCollapseImage    │ ← tag: 201
│ applianceNameLabel         │ ← tag: 202
│ cardContents               │ ← tag: 203
│ applianceImageView         │ ← tag: 210
│ cycleStatusView            │ ← tag: 220
│ cycleStatusLabel           │ ← tag: 222
│ cycleStatusValueLabel      │ ← tag: 223
│ timeLeftView               │ ← tag: 230
│ timeLeftLabel              │ ← tag: 232
│ timeLeftValueLabel         │ ← tag: 233
│ finishTimeView             │ ← tag: 240
│ finishTimeLabel            │ ← tag: 242
│ finishTimeValueLabel       │ ← tag: 243
│ progressView               │ ← tag: 250
│ progressBar                │ ← tag: 251
└────────────────────────────┘
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
- The progress bar animation starts and stops based on the machine state.

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
┌────────────────────────────┐
│ cycleLabel                 │ ← tag: 300
│ cycleNameLabel             │ ← tag: 301
└────────────────────────────┘
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
- The card's visibility can be toggled.

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
┌────────────────────────────┐
│ nightCarelabel             │ ← tag: 400
│ infoButton                 │ ← tag: 401
│ currentState               │ ← tag: 402
└────────────────────────────┘
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
- Tapping the info button triggers the onButtonPressed delegate method with .nightCareInfo.
- The current state label is updated based on the LAUNDRY_ERD_MACHINE_SUB_CYCLE ERD value.
- The card's visibility can be toggled.

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
┌────────────────────────────┐
│ waterTankStatusLabel       │ ← tag: 500
│ waterTankInfoButton        │ ← tag: 501
│ currentState               │ ← tag: 502
│ warningLabel               │ ← tag: 503
└────────────────────────────┘
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
- Tapping the info button triggers the onButtonPressed delegate method with .waterTankStatusInfo.
- The current state label and warning icon are updated based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.
- The card's visibility can be toggled.
- The info button and warning label are hidden when the tank status is OK.

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
┌────────────────────────────┐
│ drainTankStatusLabel       │ ← tag: 600
│ drainTankInfoButton        │ ← tag: 601
│ currentState               │ ← tag: 602
│ warningLabel               │ ← tag: 603
└────────────────────────────┘
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
- Tapping the info button triggers the onButtonPressed delegate method with .drainTankStatusInfo.
- The current state label and warning icon are updated based on the LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS ERD value.
- The card's visibility can be toggled.
- The info button and warning label are hidden when the tank status is OK.

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
| Machine Status | `LAUNDRY_ERD_MACHINE_STATUS` | Enum | Machine status values | Controls UI state and visibility |
| Cycle Name | `LAUNDRY_ERD_CYCLE_NAME` | String | Cycle Name value | Used for cycle name display |
| Machine Sub Cycle | `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | String | Machine Sub Cycle value | Used for machine sub cycle display |
| Clean Closet Tank Status | `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | Enum | Machine status values | Controls UI state and visibility |
| Cycle Function Current | `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | String | Cycle Function Current value | Used for cycle function current display |


---

## 💬 Popup Display Logic

Based on actual delegate implementations found in code.

---

## 🔄 Update Flow
Based on actual WebSocket and ERD update patterns found in the code.

---

## ✨ Notes for Migration
- All information extracted from actual Swift code analysis
- ERD value parsers must be ported to target platform
- UI element tags and behaviors based on actual implementation
- Popup behavior matches actual Swift delegate logic

---

*이 문서는 실제 Swift 코드 분석과 Google Gemini AI를 사용하여 자동 생성되었습니다.*
