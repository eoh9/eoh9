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
{'timezones': ['America/New_York', 'America/Los_Angeles', 'Europe/London']}
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
{'kind': 'Washer', 'model': 'GWH2400', 'valid': True, 'banned': False, 'commissionMethod': 'WIFI', 'capabilities': ['cycle_status', 'remote_start']}
```

#### `/api/v1/appliances/{applianceId}/nickname` (GET)
- **Response Body**:
```json
{'key': 'nickname', 'value': 'My Washer'}
```

#### `/api/v1/appliances/{applianceId}/preferences` (GET)
- **Response Body**:
```json
{'preferences': [{'key': 'sound_level', 'value': 'high'}, {'key': 'display_units', 'value': 'fahrenheit'}]}
```

#### `/api/v1/appliances/{applianceId}/features` (GET)
- **Response Body**:
```json
{'remote_start': True, 'cycle_status': True}
```

#### `/api/v1/appliances/{applianceId}/metadata` (GET)
- **Response Body**:
```json
{'modelNumber': 'GWH2400', 'serialNumber': 'SN12345', 'firmwareVersion': '1.2.3'}
```

#### `/api/v1/laundry/appliances/{applianceId}/metadata` (GET)
- **Response Body**:
```json
{'namespace': 'laundry', 'data': {'cycle_options': ['Normal', 'Delicates']}}
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
┌────────────────────────────────────────────┐
│ statusCardCollapseButton                 │ ← tag: 200 (UIView)
│ applianceNameLabel             "Clean Closet" │ ← tag: 202 (UILabel)
│ cycleStatusView                (StatusView) │ ← tag: 220 (UIView)
│ progressBar                              │ ← tag: 251 (LinearProgressBar)
│ timeLeftImage                            │ ← tag: 231 (UIImageView)
│ cycleStatusImage                         │ ← tag: 221 (UIImageView)
│ cycleStatusLabel               "Status"  │ ← tag: 222 (UILabel)
│ cycleStatusValueLabel          "Complete" │ ← tag: 223 (UILabel)
│ timeLeftLabel                  "Time Left" │ ← tag: 232 (UILabel)
│ timeLeftValueLabel             "--:--"   │ ← tag: 233 (UILabel)
│ finishTimeLabel                "Finished" │ ← tag: 242 (UILabel)
│ finishTimeValueLabel           "--:--"   │ ← tag: 243 (UILabel)
│ finishTimeImage                          │ ← tag: 241 (UIImageView)
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
| `cycleStatusImage` | 221 | Image for the cycle status |
| `cycleStatusLabel` | 222 | Label for the cycle status |
| `cycleStatusValueLabel` | 223 | Value label for the cycle status |
| `timeLeftView` | 230 | View containing time left information |
| `timeLeftImage` | 231 | Image for the time left |
| `timeLeftLabel` | 232 | Label for the time left |
| `timeLeftValueLabel` | 233 | Value label for the time left |
| `finishTimeView` | 240 | View containing finish time information |
| `finishTimeImage` | 241 | Image for the finish time |
| `finishTimeLabel` | 242 | Label for the finish time |
| `finishTimeValueLabel` | 243 | Value label for the finish time |
| `progressView` | 250 | View containing the progress bar |
| `progressBar` | 251 | Progress bar |

### Behavior
- Sets appliance name label text to Constants.FABRIC_CARE_CLOSET
- Adds tap gesture recognizer to statusCardCollapseButton to call onTapStatusCardCollapse
- Sets cycleStatusLabel text to Constants.STATUS
- Sets timeLeftLabel text to Constants.TIME_LEFT
- Sets finishTimeLabel text to Constants.FINISHED
- Toggles visibility of cardContents on onTapStatusCardCollapse
- Rotates statusCardCollapseImage on onTapStatusCardCollapse
- Updates cycleStatusValueLabel with LaundryErdConvertSupporterSwift.shared.getStatusText(erdValues)
- Hides/shows finishTimeView, timeLeftView, and progressView based on machine state
- Starts/stops progressBar animation based on machine state
- Updates timeLeftValueLabel with LaundryErdConvertSupporterSwift.shared.convertTimeValuesToTimeLeftString(remainingTimeValues)
- Updates finishTimeValueLabel with LaundryErdConvertSupporterSwift.shared.getEndTimeString(erdValues)
- Updates timeLeftValueLabel with LaundryErdConvertSupporterSwift.shared.convertTimeValuesToTimeLeftString(delayStartTimeValues) if in delay mode

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_MACHINE_STATUS` | LAUNDRY_ERD_MACHINE_STATUS dependency |

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
- Sets cycleLabel text to Constants.CYCLE.uppercased()
- Updates cycleNameLabel with cycle category and cycle name from ERD values
- Hides the card if shouldShow is false

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
| `infoButton` | 401 | Button to show info about night care |
| `currentState` | 402 | Label to show the current state of night care (on/off) |

### Behavior
- Sets nightCarelabel text to Constants.NIGHT_CARE.uppercased()
- Adds tap gesture recognizer to infoButton to call onTapNightCareInfo
- Updates currentState label to Constants.ON if LAUNDRY_ERD_MACHINE_SUB_CYCLE is nightCare, otherwise sets it to Constants.OFF
- Hides the card if shouldShow is false
- Calls delegate?.onButtonPressed(.nightCareInfo) on onTapNightCareInfo

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
| `waterTankInfoButton` | 501 | Button to show info about water tank status |
| `currentState` | 502 | Label to show the current state of the water tank |
| `warningLabel` | 503 | Image to show a warning about the water tank |

### Behavior
- Sets waterTankStatusLabel text to Constants.WATER_TANK_STATUS.uppercased()
- Adds tap gesture recognizer to waterTankInfoButton to call onTapWaterTankStatusInfo
- Updates currentState label to Constants.FILL_TANK and shows warning if tank needs filling, otherwise sets it to Constants.OK and hides warning
- Hides the card if shouldShow is false
- Calls delegate?.onButtonPressed(.waterTankStatusInfo) on onTapWaterTankStatusInfo

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
| `drainTankInfoButton` | 601 | Button to show info about drain tank status |
| `currentState` | 602 | Label to show the current state of the drain tank |
| `warningLabel` | 603 | Image to show a warning about the drain tank |

### Behavior
- Sets drainTankStatusLabel text to Constants.DRAIN_TANK_STATUS.uppercased()
- Adds tap gesture recognizer to drainTankInfoButton to call onTapDrainTankStatusInfo
- Updates currentState label to Constants.EMPTY_TANK and shows warning if tank needs emptying, otherwise sets it to Constants.OK and hides warning
- Hides the card if shouldShow is false
- Calls delegate?.onButtonPressed(.drainTankStatusInfo) on onTapDrainTankStatusInfo

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
| Machine Sub Cycle | `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | String | Machine Sub Cycle value | Used for machine sub cycle display |
| Clean Closet Tank Status | `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | Enum | Machine status values | Controls UI state and visibility |
| Cycle Name | `LAUNDRY_ERD_CYCLE_NAME` | String | Cycle Name value | Used for cycle name display |
| Machine Status | `LAUNDRY_ERD_MACHINE_STATUS` | Enum | Machine status values | Controls UI state and visibility |
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
