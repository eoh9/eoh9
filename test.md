# CleanCloset - Card UI/Feature Specification (Based on iOS Swift Code)

## Overview

This document details the CleanCloset feature cards implemented in the native iOS app. Each card's UI components, ERD dependencies, business logic, and associated API mappings are documented to support accurate migration to Flutter. This format follows product spec conventions similar to those used by Elaine Arens (GE Appliances).

---

## 🏡 Entry Point: CleanClosetViewController

- **File**: `CleanClosetViewController.swift` (226 lines)
- **Purpose**: Main controller managing 5 different status cards for laundry appliance monitoring
- **Displayed Cards**: StatusCard, CycleCard, NightCareCard, WaterTankStatusCard, DrainTankStatusCard
- **Popup Handling**: CleanClosetStatusCardDelegate implementation

---

## 🔌 API Endpoints, Request/Response Bodies

### 📦 Entity Definitions

Based on CleanClosetRemoteDataManager (619 lines) and CleanClosetLocalDataManager (75 lines)

| Feature                     | Method | Endpoint                                    | Response Entity                |
| --------------------------- | ------ | ------------------------------------------- | ------------------------------ |
| Device Preferences          | GET    | /preferences                                | PreferencesEntity              |
| Feature Configuration       | GET    | /feature                                    | FeatureEntity                  |
| Appliance Metadata          | GET    | /applianceMetaData                          | ApplianceMetaEntity            |
| Timezone Management         | GET    | /timezoneList                               | TimezoneListEntity             |
| Timezone Update             | POST   | /timezone                                   | TimezoneEntity                 |
| Model Validation            | GET    | /modelValidation/{modelNumber}              | ModelValidationEntity          |
| User Information            | GET    | /userInfo                                   | UserInfoEntity                 |
| Privacy Policy              | GET    | /acceptPrivacyPolicy                        | PrivacyPolicyEntity            |
| Device Nickname             | GET    | /nickname/{applianceId}                     | NicknameEntity                 |

### Request/Response Details

API calls managed through BaseRemoteDataManager with WebSocket integration for real-time ERD updates.

---

## 🗺 API to UI Mapping

| API Endpoint                                | Used In UI                | Purpose                                                |
| ------------------------------------------- | ------------------------- | ------------------------------------------------------ |
| /preferences                                | All Status Cards          | Load user preferences for card display settings       |
| /feature                                    | CleanClosetViewController | Enable/disable specific card features                 |
| WebSocket ERD Updates                       | All Status Cards          | Real-time status updates for machine state            |
| /applianceMetaData                          | StatusCard                | Display appliance model and capabilities              |

---

## 🏋️ StatusCard

### 🎨 UI 시각적 구조 (코드 기반 도식)

```
┌─────────────────────────────────────────┐
│          CleanClosetCardStatus          │
│  ┌───────────────────────────────────┐  │
│  │        Machine Status Area        │  │ 
│  │    Status: [LAUNDRY_ERD_MACHINE_  │  │
│  │             STATUS]               │  │
│  │                                   │  │
│  │    Time Left: [실시간 업데이트]     │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌─ IBOutlet: statusCard ─────────────┐ │
│  │  Type: CleanClosetCardStatus!     │ │
│  │  Tag: [XIB에서 확인 필요]           │ │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### ✨ Supported Features

🔗 Related API
- WebSocket ERD monitoring for real-time updates
- Machine state parsing from ERD values

- **Class**: `CleanClosetCardStatus: CleanClosetStatusCard`
- **XIB**: CleanClosetStatusCard.xib (638 lines)

### UI Elements

| Element                 | Tag | Description                             |
| ----------------------- | --- | --------------------------------------- |
| statusCard              | TBD | Main status display card                |
| machineStateLabel       | TBD | Shows current machine operation status  |
| timeRemainingLabel      | TBD | Displays cycle time remaining           |
| statusIcon              | TBD | Visual indicator for machine state      |

### 🔁 UI 동작 규칙 (코드 기반 처리)

```swift
// From CleanClosetStatusCard.swift - Machine Status Update Logic
func updateMachineStatus() {
    if let machineStateErd = erdValues[LAUNDRY_ERD_MACHINE_STATUS],
       let machineState = Int(machineStateErd) {
        
        switch machineState {
        case 0:
            statusLabel.text = "Idle"
            statusIcon.image = UIImage(named: "status_idle")
        case 1:
            statusLabel.text = "Running"
            statusIcon.image = UIImage(named: "status_running")
            startTimeUpdateTimer()
        case 2:
            statusLabel.text = "Paused"
            statusIcon.image = UIImage(named: "status_paused")
        case 3:
            statusLabel.text = "Complete"
            statusIcon.image = UIImage(named: "status_complete")
            showCompletionNotification()
        default:
            statusLabel.text = "Unknown"
        }
        
        updateStatusCardAppearance()
    }
}
```

### Behavior

- Real-time ERD monitoring through WebSocketServiceDelegate
- Machine state updates trigger UI refresh
- Time remaining calculation and display
- Status icon animations based on machine state

### ERDs Used

| ERD                          | Description                         |
| ---------------------------- | ----------------------------------- |
| LAUNDRY_ERD_MACHINE_STATUS   | Current machine operational state   |
| LAUNDRY_ERD_CYCLE_NAME       | Active cycle type identifier        |
| LAUNDRY_ERD_MACHINE_SUB_CYCLE| Sub-cycle progress indicator        |

### API Dependency

- WebSocket connection for real-time ERD updates
- CleanClosetInteractor: WebSocketServiceDelegate implementation
- OtaServiceDelegate for firmware update status

### Test Skeleton

```swift
class CleanClosetStatusCardTests: XCTestCase {
    
    func testMachineStatusUpdate() {
        let statusCard = CleanClosetCardStatus()
        let testERD = ["LAUNDRY_ERD_MACHINE_STATUS": "1"]
        
        statusCard.updateERD(testERD)
        
        XCTAssertEqual(statusCard.statusLabel.text, "Running")
        XCTAssertNotNil(statusCard.statusIcon.image)
    }
    
    func testTimeRemainingCalculation() {
        // Test time remaining calculation logic
        XCTAssertTrue(true) // Implementation needed
    }
}
```

---

## 📅 CycleCard

### 🎨 UI 시각적 구조 (코드 기반 도식)

```swift
// From CleanClosetStatusCard.swift - Cycle Card Implementation
class CleanClosetCycleCard: CleanClosetStatusCard {
    
    func updateCycleInfo() {
        if let cycleErd = erdValues[LAUNDRY_ERD_CYCLE_NAME],
           let cycleValue = Int(cycleErd),
           let cycleTypeErd = erdValues[LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT],
           let cycleType = Int(cycleTypeErd) {
            
            let cycleName = getCycleName(cycleValue)
            let cycleFunction = getCycleFunction(cycleType)
            
            cycleNameLabel.text = cycleName
            cycleFunctionLabel.text = cycleFunction
            updateCycleProgress()
        }
    }
}
```

### ERDs Used

| ERD                               | Description                         |
| --------------------------------- | ----------------------------------- |
| LAUNDRY_ERD_CYCLE_NAME           | Active cycle identifier             |
| LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT| Current cycle function/mode         |

---

## 🏠 NightCareCard

### 🔁 UI 동작 규칙 (코드 기반 처리)

```swift
// From CleanClosetStatusCard.swift - Night Care Implementation
class CleanClosetNightCareCard: CleanClosetStatusCard {
    
    func updateNightCareStatus() {
        if let subCycle = erdValues[LAUNDRY_ERD_MACHINE_SUB_CYCLE],
           let subCycleValue = Int(subCycle) {
            
            switch subCycleValue {
            case 10...15:
                // Night care mode active
                enableNightCareDisplay()
                scheduleNightCareUpdates()
            default:
                disableNightCareDisplay()
            }
        }
    }
}
```

---

## 🚰 WaterTankStatusCard

### 🔁 UI 동작 규칙 (코드 기반 처리)

```swift
// From CleanClosetStatusCard.swift - Water Tank Status
class CleanClosetWaterTankStatusCard: CleanClosetStatusCard {
    
    func updateWaterTankStatus() {
        if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
           let tankStatus = Int(notificationErd) {
            
            let waterLevel = (tankStatus >> 4) & 0x0F  // Upper 4 bits
            updateWaterLevelIndicator(waterLevel)
            
            if waterLevel <= 2 {
                showLowWaterWarning()
            }
        }
    }
}
```

---

## 🚮 DrainTankStatusCard

### 🔁 UI 동작 규칙 (코드 기반 처리)

```swift
// From CleanClosetStatusCard.swift - Drain Tank Status  
class CleanClosetDrainTankStatusCard: CleanClosetStatusCard {
    
    func updateDrainTankStatus() {
        if let notificationErd = erdValues[LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS],
           let tankStatus = Int(notificationErd) {
            
            let drainLevel = tankStatus & 0x0F  // Lower 4 bits
            updateDrainLevelIndicator(drainLevel)
            
            if drainLevel >= 14 {
                showDrainTankFullWarning()
            }
        }
    }
}
```

---

## 📊 ERD Definitions

| ERD Name                           | Code Constant                          | Type            | Description                                   | UI Behavior                                          |
| ---------------------------------- | -------------------------------------- | --------------- | --------------------------------------------- | ---------------------------------------------------- |
| Machine Status                     | LAUNDRY_ERD_MACHINE_STATUS            | Integer         | Current operational state (0-3)              | Updates status icon and text in StatusCard          |
| Cycle Name                         | LAUNDRY_ERD_CYCLE_NAME                | Integer         | Active cycle identifier                       | Displays cycle name in CycleCard                    |
| Cycle Function                     | LAUNDRY_ERD_CYCLE_FUNCTION_CURRENT    | Integer         | Current cycle mode/function                   | Shows specific cycle operation in CycleCard         |
| Sub Cycle                          | LAUNDRY_ERD_MACHINE_SUB_CYCLE         | Integer         | Sub-cycle progress (10-15 for night care)    | Triggers NightCareCard display logic                |
| Tank Status                        | LAUNDRY_ERD_CLEAN_CLOSET_TANK_STATUS  | Integer         | Combined water/drain tank levels (8-bit)     | Updates both water and drain tank status indicators |
| ACM Version                        | ERD_ACM_AVAILABLE_VERSION             | String          | Available ACM firmware version               | Triggers OTA update notifications                   |
| Appliance Version                  | ERD_APPLIANCE_AVAILABLE_VERSION       | String          | Available appliance firmware version        | Triggers appliance update notifications             |

---

## 💬 Popup Display Logic

| Popup Type                 | Trigger Condition                                       | Handler Location                                                                           |
| -------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Low Water Warning          | Water tank level <= 2 (from tank status ERD)          | WaterTankStatusCard.showLowWaterWarning()                                                 |
| Drain Tank Full            | Drain level >= 14 (from tank status ERD)               | DrainTankStatusCard.showDrainTankFullWarning()                                            |
| Cycle Complete             | Machine status == 3 (Complete)                         | StatusCard.showCompletionNotification()                                                   |
| OTA Update Available       | Version ERD comparison in CleanClosetPresenter         | CleanClosetPresenter.handleOTAUpdateAvailable()                                           |

---

## 🔄 Update Flow

```
WebSocket ERD Update → CleanClosetInteractor → CleanClosetPresenter → CleanClosetViewController → Individual Status Cards

1. WebSocketServiceDelegate receives ERD updates
2. CleanClosetInteractor processes ERD values  
3. CleanClosetPresenter formats data for UI
4. CleanClosetViewController distributes updates to cards
5. Each card updates its specific UI elements
```

---

## ✨ Notes for Migration

### Swift-Specific Patterns
- **IBOutlet connections**: Flutter requires StatefulWidget equivalents
- **WebSocket delegates**: Convert to Stream listeners in Flutter
- **ERD bit manipulation**: Preserve exact bit operations for tank status
- **Timer management**: Replace NSTimer with Flutter Timer class
- **UI threading**: Convert DispatchQueue.main to Flutter's main isolate

### Architecture Migration
- **VIPER → BLoC**: Convert CleanClosetPresenter to Cubit/Bloc pattern
- **Protocols → Abstract classes**: Transform Swift protocols to Dart abstracts
- **Extensions → Mixins**: Convert Swift extensions to Dart mixins
- **Delegates → Streams**: Replace delegate patterns with Stream controllers

### Key Implementation Notes
- Preserve exact ERD parsing logic for appliance compatibility
- Maintain bit manipulation operations for tank status (upper/lower 4 bits)
- Keep WebSocket message handling identical for real-time updates
- Ensure timer precision for cycle countdown accuracy
