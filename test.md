# CleanCloset - Card UI/Feature Specification (Based on iOS Swift Code)

## Overview
This document details the CleanCloset feature cards implemented in the native iOS app. Each card's UI components, ERD dependencies, business logic, and associated API mappings are documented to support accurate migration to Flutter. This format follows product spec conventions similar to those used by Elaine Arens (GE Appliances).

---

## 🏡 Entry Point: CleanClosetViewController
- **File**: `CleanClosetViewController.swift`
- **Purpose**: Manages the CleanCloset control screen UI.
- **Displayed Cards**:
  - `StatusCard`
  - `CycleCard`
  - `NightCareCard`
  - `WaterTankStatusCard`
  - `DrainTankStatusCard`
- **Popup Handling**:
  - Shows popup on WebSocket error, OTA status, and NightCare info.
  - Accepts privacy policy via modal popup.

---

## 🔌 API Endpoints, Request/Response Bodies

### 📦 Entity Definitions



| Feature | Method | Endpoint | Response Entity |
| ------- | ------ | -------- | --------------- |
| Retrieve User Info | GET | `/api/v1/user/info` | `UserInformationEntity` |
| Retrieve Preferences | GET | `/api/v1/user/preferences` | `PreferenceListEntity` |
| Retrieve Appliance Metadata | GET | `/api/v1/appliances/{applianceId}/metadata` | `ApplianceMetaDataEntity` |
| Retrieve Appliance Features | GET | `/api/v1/appliances/{applianceId}/features` | `FeatureEntity` |
| Set Nickname | POST | `/api/v1/appliances/{applianceId}/nickname` | `PreferenceListEntryEntity` |
| Retrieve Privacy Terms | GET | `/api/v1/terms/connected` | `ConnectedTermsEntity` |
| Accept Privacy Terms | POST | `/api/v1/terms/connected/accept` | `ConnectedTermsResponseEntity` |
| Retrieve Timezones | GET | `/api/v1/timezones` | `TimezoneListEntity` |
| Set Timezone | POST | `/api/v1/user/timezone` | `PostResponseEntity` |
| Validate Model | POST | `/api/v1/ota/validate` | `ModelValidationEntity` |


### Request/Response Details

#### `/api/v1/user/info` (GET)
- **Response Body**:
```json
{"userId": "abc123", "termsConnectedAccepted": true, "timeZone": "America/New_York"}
```

#### `/api/v1/user/preferences` (GET)
- **Response Body**:
```json
{"items": [{"name": "appliance.a1b2c3.timezone", "value": "America/New_York"}]}
```

#### `/api/v1/appliances/{applianceId}/metadata` (GET)
- **Response Body**:
```json
{"applianceId": "a1b2c3", "modelNumber": "GFW850SPNRS", "serialNumber": "123456", "firmwareVersion": "2.1.5"}
```

#### `/api/v1/appliances/{applianceId}/features` (GET)
- **Response Body**:
```json
{"features": ["nightCare", "tankStatus", "cycleStatus"]}
```

#### `/api/v1/appliances/{applianceId}/nickname` (POST)
- **Request Body**:
```json
{"value": "Laundry Room Washer"}
```
- **Response Body**:
```json
{"value": "Laundry Room Washer"}
```

#### `/api/v1/terms/connected` (GET)
- **Response Body**:
```json
{"version": "1.2", "url": "https://geappliances.com/privacy"}
```

#### `/api/v1/terms/connected/accept` (POST)
- **Request Body**:
```json
{"kind": "terms", "userId": "abc123", "termsConnectedAccepted": true}
```
- **Response Body**:
```json
{"status": "success"}
```

#### `/api/v1/timezones` (GET)
- **Response Body**:
```json
{"items": ["America/New_York", "America/Chicago"]}
```

#### `/api/v1/user/timezone` (POST)
- **Request Body**:
```json
{"timezoneId": "America/New_York"}
```
- **Response Body**:
```json
{"status": "ok"}
```

#### `/api/v1/ota/validate` (POST)
- **Request Body**:
```json
{"modelNumber": "GFW850SPNRS"}
```
- **Response Body**:
```json
{"model": "GFW850SPNRS", "valid": true, "capabilities": ["drs", "ota"]}
```



---

## 🗺 API to UI Mapping

| API Endpoint | Used In UI | Purpose |
| ------------ | ---------- | ------- |
| `/api/v1/user/info` | CleanClosetViewController | Determine privacy policy acceptance |
| `/api/v1/user/preferences` | CleanClosetInteractor | Extract appliance timezone, preferences |
| `/api/v1/appliances/{applianceId}/metadata` | StatusCard | Display model number, appliance info |
| `/api/v1/appliances/{applianceId}/features` | All Cards | Check if card features (e.g., nightCare) are supported |
| `/api/v1/appliances/{applianceId}/nickname` | CleanClosetInteractor | Retrieve nickname for display |
| `/api/v1/terms/connected` | CleanClosetInteractor | Get privacy policy URL and version |
| `/api/v1/terms/connected/accept` | CleanClosetViewController | Submit user's privacy policy acceptance |
| `/api/v1/timezones` | CleanClosetInteractor | Get list of available timezones |
| `/api/v1/user/timezone` | CleanClosetInteractor | Save user's selected timezone |
| `/api/v1/ota/validate` | CleanClosetInteractor | Check if appliance is eligible for OTA update |


---

## 🏋️ StatusCard

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌──────────────────────────────────────────────┐
│ FABRIC CARE CLOSET                           │ ← applianceNameLabel (tag: 202)
├──────────────────────────────────────────────┤
│ Status:         RUNNING                      │ ← cycleStatusValueLabel (tag: 223)
│ Time Left:      00:42                        │ ← timeLeftValueLabel (tag: 233)
│ Finish Time:    03:15 PM                     │ ← finishTimeValueLabel (tag: 243)
│                                               │
│ ▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░ (animated)         │ ← progressBar (tag: 251)
└──────────────────────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetCardStatus`
- **XIB**: `CleanClosetStatusCard.xib` [Index 0]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `applianceNameLabel` | 202 | Shows static label FABRIC CARE CLOSET |
| `cycleStatusValueLabel` | 223 | Shows machine state (e.g., RUNNING) |
| `timeLeftValueLabel` | 233 | Displays formatted remaining time |
| `finishTimeValueLabel` | 243 | Displays estimated finish time |
| `progressBar` | 251 | Animated progress bar when running |

### Behavior
- Updates dynamically via update(erdValues:) method
- Animates progressBar if machineState == run
- finishTimeView hidden unless machine is running
- timeLeftView hidden unless machine is running or delayed

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_MACHINE_STATUS` | LAUNDRY_ERD_MACHINE_STATUS dependency |
| `LAUNDRY_ERD_TIME_REMAINING` | LAUNDRY_ERD_TIME_REMAINING dependency |
| `LAUNDRY_ERD_END_TIME` | LAUNDRY_ERD_END_TIME dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
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
```

### Test Skeleton
```swift
testStatusCard_shouldShowFinishTime_whenMachineRunning() {
    let card = CleanClosetCardStatus()
    let erd = ["0x0032": "0x01"] // run status
    card.update(erd)
    XCTAssertFalse(card.finishTimeView?.isHidden ?? true)
}
```

---
## 🏋️ CycleCard

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────┐
│ CYCLE                      │ ← cycleLabel (tag: 300)
│ ▸ NORMAL - BULKY ITEMS     │ ← cycleNameLabel (tag: 301)
└────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetCycleCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 1]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `cycleLabel` | 300 | Static label CYCLE |
| `cycleNameLabel` | 301 | Shows current cycle name + category |

### Behavior
- Updates when LAUNDRY_ERD_CYCLE_NAME or LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT changes
- Converts raw ERD data to localized names

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CYCLE_NAME` | LAUNDRY_ERD_CYCLE_NAME dependency |
| `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
if let cycleErd = erdValues[LAUNDRY_ERD_CYCLE_NAME],
   let cycleNameString = LaundryErdConvertSupporter().getCycleName(cycleErd),
   let cycleTypeErd = erdValues[LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT],
   let cycleCategoryString = CleanClosetErdValues.CycleCategory(rawValue: cycleTypeErd.subString(from: 0, length: 2))?.getCycleCategoryName() {
    
    cycleNameLabel?.text = "\(cycleCategoryString) - \(cycleNameString.uppercased())"
} else {
    cycleNameLabel?.text = ""
}
```

### Test Skeleton
```swift
testCycleCard_shouldUpdateCycleName_whenERDChanges() {
    let card = CleanClosetCycleCard()
    let erd = ["LAUNDRY_ERD_CYCLE_NAME": "normal", "LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT": "01"]
    card.update(erd)
    XCTAssertNotNil(card.cycleNameLabel?.text)
}
```

---
## 🏋️ NightCareCard

### 🎨 UI 시각적 구조 (코드 기반 도식)
```
┌────────────────────────────┐
│ NIGHT CARE       (ⓘ)       │ ← nightCarelabel (tag: 400), infoButton (tag: 401)
├────────────────────────────┤
│ Status:       ON / OFF     │ ← currentState (tag: 402)
└────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetNightCareCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 2]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `nightCarelabel` | 400 | Static label NIGHT CARE |
| `currentState` | 402 | Shows ON/OFF |
| `infoButton` | 401 | Triggers info popup |

### Behavior
- Shows ON if subCycle == nightCare, otherwise OFF
- Tapping info icon calls delegate?.onButtonPressed(.nightCareInfo)

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | LAUNDRY_ERD_MACHINE_SUB_CYCLE dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
if let subCycle = erdValues[LAUNDRY_ERD_MACHINE_SUB_CYCLE],
   subCycle == LaundryErdValue.MachineSubCycle.nightCare.value() {
    
    currentState?.text = Constants.ON
} else {
    currentState?.text = Constants.OFF
}

@objc func onTapNightCareInfo() {
    delegate?.onButtonPressed(.nightCareInfo)
}
```

### Test Skeleton
```swift
testNightCareCard_shouldShowON_whenNightCareActive() {
    let card = CleanClosetNightCareCard()
    let erd = ["LAUNDRY_ERD_MACHINE_SUB_CYCLE": "nightCare"]
    card.update(erd)
    XCTAssertEqual(card.currentState?.text, Constants.ON)
}
```

---
## 🏋️ WaterTankStatusCard

### 🎨 UI 시각적 구조 (코드 기반 도식)
📥 정상이면:
```
┌────────────────────────────┐
│ WATER TANK STATUS          │ ← waterTankStatusLabel (tag: 500)
├────────────────────────────┤
│ Status:        OK          │ ← currentState (tag: 502)
└────────────────────────────┘
```

⚠️ 물 보충 필요 시:
```
┌────────────────────────────┐
│ WATER TANK STATUS   (⚠ ⓘ)  │ ← warningLabel (tag: 503), infoButton (tag: 501)
├────────────────────────────┤
│ Status:     FILL TANK      │ ← currentState (tag: 502, red color)
└────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetWaterTankStatusCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 3]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `currentState` | 502 | OK / FILL TANK |
| `infoButton` | 501 | Shows info popup |
| `warningLabel` | 503 | Displays warning icon |

### Behavior
- If bit 2 of tankStatus == 1, show FILL TANK and warning

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
   let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
   data.count > 2,
   data[2] == "1" {
    currentState?.text = Constants.FILL_TANK
    currentState?.textColor = Color.warning
    waterTankInfoButton?.isHidden = false
    warningLabel?.isHidden = false
} else {
    currentState?.text = Constants.OK
    currentState?.textColor = Color.Card.On.titleText
    waterTankInfoButton?.isHidden = true
    warningLabel?.isHidden = true
}
```

### Test Skeleton
```swift
testWaterTankCard_shouldShowFillTank_whenBit2Set() {
    let card = CleanClosetWaterTankStatusCard()
    let erd = ["LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS": "04"] // bit 2 set
    card.update(erd)
    XCTAssertEqual(card.currentState?.text, Constants.FILL_TANK)
}
```

---
## 🏋️ DrainTankStatusCard

### 🎨 UI 시각적 구조 (코드 기반 도식)
📥 정상이면:
```
┌────────────────────────────┐
│ DRAIN TANK STATUS          │ ← drainTankStatusLabel (tag: 600)
├────────────────────────────┤
│ Status:        OK          │ ← currentState (tag: 602)
└────────────────────────────┘
```

⚠️ 배수 필요 시:
```
┌────────────────────────────┐
│ DRAIN TANK STATUS   (⚠ ⓘ)  │ ← warningLabel (tag: 603), infoButton (tag: 601)
├────────────────────────────┤
│ Status:     EMPTY TANK     │ ← currentState (tag: 602, red color)
└────────────────────────────┘
```

### ✨ Supported Features
- **Class**: `CleanClosetDrainTankStatusCard`
- **XIB**: `CleanClosetStatusCard.xib` [Index 4]

### UI Elements
| Element | Tag | Description |
| ------- | --- | ----------- |
| `currentState` | 602 | OK / EMPTY TANK |
| `infoButton` | 601 | Shows info popup |
| `warningLabel` | 603 | Displays warning icon |

### Behavior
- If bit 3 of tankStatus == 1, show EMPTY TANK and warning

### ERDs Used
| ERD | Description |
| --- | ----------- |
| `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS dependency |

### API Dependency
- WebSocket-based ERD updates via `requestCache()`.

### 🔁 UI 동작 규칙 (코드 기반 처리)
```swift
if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
   let data = notificationErd.subString(from: 0, length: 2).toBinary(8),
   data.count > 3,
   data[3] == "1" {
    currentState?.text = Constants.EMPTY_TANK
    currentState?.textColor = Color.warning
    drainTankInfoButton?.isHidden = false
    warningLabel?.isHidden = false
} else {
    currentState?.text = Constants.OK
    currentState?.textColor = Color.Card.On.titleText
    drainTankInfoButton?.isHidden = true
    warningLabel?.isHidden = true
}
```

### Test Skeleton
```swift
testDrainTankCard_shouldShowEmptyTank_whenBit3Set() {
    let card = CleanClosetDrainTankStatusCard()
    let erd = ["LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS": "08"] // bit 3 set
    card.update(erd)
    XCTAssertEqual(card.currentState?.text, Constants.EMPTY_TANK)
}
```

---


## 📊 ERD Definitions

| ERD Name | Code Constant | Type | Description | UI Behavior |
| -------- | ------------- | ---- | ----------- | ----------- |
| Machine Status | `LAUNDRY_ERD_MACHINE_STATUS` | Enum | run / pause / delay states | Controls progress bar, time visibility in StatusCard |
| Time Remaining | `LAUNDRY_ERD_TIME_REMAINING` | Integer | Remaining time in seconds or minutes | Displayed in timeLeftValueLabel |
| End Time | `LAUNDRY_ERD_END_TIME` | String (HH:MM) | Expected finish time | Displayed in finishTimeValueLabel |
| Cycle Name | `LAUNDRY_ERD_CYCLE_NAME` | String | Current selected cycle | Displayed in CycleCard |
| Cycle Function | `LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT` | String | Encoded category prefix | Used to construct full cycle name |
| Machine SubCycle | `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | String | e.g. nightCare | Affects NightCare ON/OFF display |
| Tank Status Flags | `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | Binary String | Bit 2: water tank low, Bit 3: drain tank full | Triggers warning icon, text color change |


| ERD | Affects Card |
| --- | ------------ |
| `LAUNDRY_ERD_MACHINE_STATUS` | StatusCard |
| `LAUNDRY_ERD_CYCLE_NAME` | CycleCard |
| `LAUNDRY_ERD_MACHINE_SUB_CYCLE` | NightCareCard |
| `LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS` | WaterTankStatusCard, DrainTankStatusCard |

---

## 💬 Popup Display Logic

| Popup Type | Trigger Condition | Handler Location |
| ---------- | ----------------- | ---------------- |
| `PrivacyPolicyPopup` | User info API returns `termsConnectedAccepted == false` | `presenter.didUserAcceptedPrivacyPolicy(false)` → `view.showPrivacyPolicyPopup()` |
| `NightCareInfoPopup` | NightCareCard info icon tapped | `onButtonPressed(.nightCareInfo)` → `presenter.onPressedNightCareInfoButton()` |
| `WaterTankStatusInfoPopup` | WaterTankCard info icon tapped | `onButtonPressed(.waterTankStatusInfo)` → `presenter.onPressedWaterTankStatusInfoButton()` |
| `DrainTankStatusInfoPopup` | DrainTankCard info icon tapped | `onButtonPressed(.drainTankStatusInfo)` → `presenter.onPressedDrainTankStatusInfoButton()` |
| `WebSocketTimeoutPopup` | WebSocket disconnects or times out | `showPopup(webSocketPopupType: .WebSocketTimeOutPopup)` |
| `OTACheckPopup` | OTA status change / update available | `presenter.showPopup(otaPopupType:)` |

---

## 🔄 Update Flow
1. ViewController binds cards and delegates.
2. WebSocket connects on `viewWillAppear`.
3. On receiving new ERDs, `updateCards(_:)` called.
4. Each card's `update(erdValues:)` is called.
5. Cards render UI based on updated ERD values.

---

## ✨ Notes for Migration
- WebSocket feed provides ERD updates as `[AnyHashable: String]`
- ERD value parsers (e.g., `LaundryErdConvertSupporter`) must be ported to Dart
- XIB tag mapping should guide widget hierarchy
- Popup behavior should match Swift logic (presented by delegate callbacks)

---

*이 문서는 Swift Syntax와 Google Gemini AI를 사용하여 자동 생성되었습니다.*
