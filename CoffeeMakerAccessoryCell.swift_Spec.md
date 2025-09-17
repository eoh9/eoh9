Of course. Here is a CleanCloset Level Quality specification for the `CoffeeHome/ViewLayer/Appliances/CoffeeMaker/View/CoffeeMakerAccessoryCell.swift` feature, created by synthesizing a realistic implementation based on the provided context.

***

# **Feature Complete Integration Specification: CoffeeMakerAccessoryCell**

## 1. **Feature Overview and Entry Point**

### **File Path**
`CafeHome/ViewLayer/Appliances/CoffeeMaker/View/CoffeeMakerAccessoryCell.swift`

### **Purpose**
The `CoffeeMakerAccessoryCell` is a reusable `UICollectionViewCell` designed to display the status and lifecycle of a single coffee maker accessory, such as a water filter or a descaling solution. It visually communicates the accessory's health (e.g., Normal, Warning, Replace) and provides a context-sensitive call-to-action button for tasks like reordering or resetting the accessory's lifecycle counter. This cell is a key component within the `CoffeeMakerDetailsViewController`'s main collection view.

### **Displayed Components**
- **Accessory Card**: The main container view for all elements, with a distinct background color that changes based on the accessory's status.
- **Icon**: A visual representation of the accessory (e.g., a water filter icon).
- **Title**: A label indicating the name of the accessory (e.g., "Water Filter").
- **Status**: A label describing the current state or remaining life of the accessory (e.g., "Life: 85%", "Replacement Recommended").
- **Action Button**: A button that allows the user to perform a relevant action, such as "Order Now" or "Reset Filter". Its visibility, title, and enabled state are determined by the accessory's status.

## 2. **UI Components Detailed Analysis (CleanCloset Style)**

The `CoffeeMakerAccessoryCell` is composed of several key UI elements, each with a specific tag for identification and testing. The cell's appearance dynamically changes based on the `AccessoryViewModel.Status` enum.

### **Component Tag Summary**
| Tag | Element Name               | Type                | Description                                        |
|-----|----------------------------|---------------------|----------------------------------------------------|
| 500 | `containerView`            | `UIView`            | The main background view of the cell.              |
| 501 | `accessoryIconImageView`   | `UIImageView`       | Displays the icon for the accessory.               |
| 502 | `titleLabel`               | `UILabel`           | Displays the accessory name (e.g., "Water Filter").  |
| 503 | `statusLabel`              | `UILabel`           | Displays the current status or percentage.         |
| 504 | `actionButton`             | `UIButton`          | The primary call-to-action button.                 |

---

### **Visual Structure and States**

#### **State 1: Normal (`.ok`)**
- **Description**: The accessory is functioning correctly with sufficient life remaining. The action button is hidden as no user action is required.
- **`containerView` Background**: `Color.gray100`
- **`statusLabel` Text Color**: `Color.primary`

```ascii
+--------------------------------------------------+
| containerView (tag: 500)                         |
|                                                  |
|  +---------+                                     |
|  |         |   titleLabel (tag: 502)             |
|  |  ICON   |   "Water Filter"                    |
|  | (501)   |                                     |
|  |         |   statusLabel (tag: 503)            |
|  +---------+   "Life: 85%"                       |
|                                                  |
|                                                  |
|                                                  |
+--------------------------------------------------+
```

#### **State 2: Warning (`.warning`)**
- **Description**: The accessory is nearing the end of its life. The user is prompted to order a replacement.
- **`containerView` Background**: `Color.warningBackground` (e.g., light yellow)
- **`statusLabel` Text Color**: `Color.warningText` (e.g., dark yellow/orange)
- **`actionButton`**: Visible, with title "Order Now".

```ascii
+--------------------------------------------------+
| containerView (tag: 500)                         |
|                                                  |
|  +---------+                                     |
|  |         |   titleLabel (tag: 502)             |
|  |  ICON   |   "Water Filter"                    |
|  | (501)   |                                     |
|  |         |   statusLabel (tag: 503)            |
|  +---------+   "Replacement Recommended"         |
|                                                  |
|                +--------------------------+      |
|                |   actionButton (tag: 504)  |      |
|                |       "Order Now"        |      |
|                +--------------------------+      |
+--------------------------------------------------+
```

#### **State 3: Action Required (`.replace`)**
- **Description**: The accessory's lifecycle is complete and it must be replaced. The action button prompts the user to reset the counter after replacement.
- **`containerView` Background**: `Color.errorBackground` (e.g., light red)
- **`statusLabel` Text Color**: `Color.errorText` (e.g., red)
- **`actionButton`**: Visible, with title "Reset Filter".

```ascii
+--------------------------------------------------+
| containerView (tag: 500)                         |
|                                                  |
|  +---------+                                     |
|  |         |   titleLabel (tag: 502)             |
|  |  ICON   |   "Water Filter"                    |
|  | (501)   |                                     |
|  |         |   statusLabel (tag: 503)            |
|  +---------+   "Replace Filter Now"              |
|                                                  |
|                +--------------------------+      |
|                |   actionButton (tag: 504)  |      |
|                |      "Reset Filter"      |      |
|                +--------------------------+      |
+--------------------------------------------------+
```

### **UI Behavior Rules**
The cell is configured via a single public method, `configure(with viewModel: AccessoryViewModel)`. This method is responsible for setting all UI properties based on the view model's state.

```swift
// In CafeHome/ViewLayer/Appliances/CoffeeMaker/View/CoffeeMakerAccessoryCell.swift

struct AccessoryViewModel {
    enum Status { case ok, warning, replace }
    enum AccessoryType { case waterFilter, descaler }

    let type: AccessoryType
    let title: String
    let icon: UIImage?
    let status: Status
    let statusText: String
    let actionButtonTitle: String?
}

class CoffeeMakerAccessoryCell: UICollectionViewCell {
    // ... IBOutlets for all UI elements ...

    func configure(with viewModel: AccessoryViewModel) {
        titleLabel.text = viewModel.title
        accessoryIconImageView.image = viewModel.icon
        statusLabel.text = viewModel.statusText

        // State-based UI updates
        switch viewModel.status {
        case .ok:
            containerView.backgroundColor = .systemGray6
            statusLabel.textColor = .label
            actionButton.isHidden = true
        case .warning:
            containerView.backgroundColor = .systemYellow.withAlphaComponent(0.1)
            statusLabel.textColor = .systemYellow
            actionButton.isHidden = false
            actionButton.setTitle(viewModel.actionButtonTitle, for: .normal)
            actionButton.backgroundColor = .systemYellow
        case .replace:
            containerView.backgroundColor = .systemRed.withAlphaComponent(0.1)
            statusLabel.textColor = .systemRed
            actionButton.isHidden = false
            actionButton.setTitle(viewModel.actionButtonTitle, for: .normal)
            actionButton.backgroundColor = .systemRed
        }
    }
}
```

## 3. **File Dependencies and Function Connections**

### **Protocol Definition**
A delegate protocol is used to communicate user interactions from the cell back to the containing `ViewController`.

**File**: `CafeHome/ViewLayer/Appliances/CoffeeMaker/View/CoffeeMakerAccessoryCell.swift`
```swift
protocol CoffeeMakerAccessoryCellDelegate: AnyObject {
    func accessoryCellDidTapAction(for cell: CoffeeMakerAccessoryCell, type: AccessoryViewModel.AccessoryType)
}
```

### **Function Call Chain**
The user tapping the `actionButton` initiates a clear chain of events.

1.  **User Action**: User taps `actionButton (tag: 504)`.
2.  **Cell Handler**: The cell's internal `@IBAction` method is triggered.
    ```swift
    // In CoffeeMakerAccessoryCell.swift
    private var viewModel: AccessoryViewModel?
    weak var delegate: CoffeeMakerAccessoryCellDelegate?

    @IBAction private func actionButtonTapped(_ sender: UIButton) {
        guard let type = viewModel?.type else { return }
        delegate?.accessoryCellDidTapAction(for: self, type: type)
    }
    ```
3.  **Delegate Implementation**: The `CoffeeMakerDetailsViewController` implements the delegate method to handle the action.
    ```swift
    // In CoffeeMakerDetailsViewController.swift
    extension CoffeeMakerDetailsViewController: CoffeeMakerAccessoryCellDelegate {
        func accessoryCellDidTapAction(for cell: CoffeeMakerAccessoryCell, type: AccessoryViewModel.AccessoryType) {
            switch type {
            case .waterFilter:
                presenter.handleFilterAction()
            case .descaler:
                presenter.handleDescalerAction()
            }
        }
    }
    ```
4.  **Presenter Logic**: The `CoffeeMakerPresenter` contains the business logic to decide what to do next (e.g., show a popup, make an API call).
    ```swift
    // In CoffeeMakerPresenter.swift
    func handleFilterAction() {
        // Logic to determine if it's an "Order" or "Reset" action
        // based on the current device state.
        if device.filterStatus == .replace {
            view?.showResetConfirmationPopup(for: .waterFilter)
        } else {
            view?.openStoreLinkForFilter()
        }
    }
    ```

## 4. **ERD Definitions and UI Mapping**

The cell's UI is directly driven by ERD (Entity Relationship Data) values received from the coffee maker device via WebSocket.

| ERD Constant Name               | Type    | Range/Values                        | UI Element Affected      | UI Impact                                                                                                |
|---------------------------------|---------|-------------------------------------|--------------------------|----------------------------------------------------------------------------------------------------------|
| `COFFEE_MAKER_FILTER_LIFE`      | Integer | 0-100                               | `statusLabel (tag: 503)` | Displays the percentage, e.g., "Life: 85%". Mapped to `AccessoryViewModel.statusText`.                    |
| `COFFEE_MAKER_FILTER_STATUS`    | Enum    | `0`: OK<br>`1`: Warning<br>`2`: Replace | All Cell Elements        | Determines the overall `AccessoryViewModel.Status` (`.ok`, `.warning`, `.replace`), controlling colors, button visibility, and text. |
| `COFFEE_MAKER_DESCALING_STATUS` | Enum    | `0`: Not Needed<br>`1`: Needed      | All Cell Elements        | For the descaler cell, `1` maps to the `.replace` state, showing a "Start Descaling" button.             |

### **ERD to ViewModel Mapping Logic**
This mapping occurs within the `CoffeeMakerPresenter` or a dedicated mapper class before the view model is passed to the `ViewController`.

```swift
// In CoffeeMakerPresenter.swift
private func createFilterViewModel(from device: CoffeeMakerDevice) -> AccessoryViewModel {
    let life = device.erd[COFFEE_MAKER_FILTER_LIFE] as? Int ?? 100
    let statusValue = device.erd[COFFEE_MAKER_FILTER_STATUS] as? Int ?? 0

    var status: AccessoryViewModel.Status
    var statusText: String
    var actionTitle: String?

    switch statusValue {
    case 1: // Warning
        status = .warning
        statusText = "Replacement Recommended"
        actionTitle = "Order Now"
    case 2: // Replace
        status = .replace
        statusText = "Replace Filter Now"
        actionTitle = "Reset Filter"
    default: // OK
        status = .ok
        statusText = "Life: \(life)%"
        actionTitle = nil
    }

    return AccessoryViewModel(
        type: .waterFilter,
        title: "Water Filter",
        icon: UIImage(named: "icon_filter"),
        status: status,
        statusText: statusText,
        actionButtonTitle: actionTitle
    )
}
```

## 5. **API Endpoints and Entities**

The cell itself does not directly interact with APIs. However, its actions trigger API calls from the `Presenter` or `Interactor` layer.

### **Endpoint: Reset Filter Lifecycle**
This API is called when the user confirms they have replaced the filter and taps the "Reset Filter" button.

- **Endpoint**: `POST /api/v1/devices/{deviceId}/controls`
- **Purpose**: Send a command to the device to reset an ERD value.

### **Request/Response Entities**

**Request Entity**: A generic control command structure.
```swift
// In CafeHome/NetworkLayer/Entities/DeviceControl.swift
struct DeviceControlRequest: Encodable {
    let erd: String
    let value: String // Value is often sent as a string representation
}

// Example usage in Presenter/Interactor
let request = DeviceControlRequest(erd: "COFFEE_MAKER_FILTER_STATUS", value: "0")
// This would reset the filter status to OK.
```

**Response Entity**: A standard API response confirming the command was received.
```swift
// In CafeHome/NetworkLayer/Entities/ApiResponse.swift
struct ApiResponse: Decodable {
    let success: Bool
    let message: String?
}
```

### **API to UI Mapping**
1.  User taps "Reset Filter" in the `CoffeeMakerAccessoryCell`.
2.  `CoffeeMakerDetailsViewController` delegate method calls `presenter.handleFilterAction()`.
3.  `Presenter` shows a confirmation popup (see Section 6).
4.  User confirms.
5.  `Presenter` calls an `Interactor` to execute the `DeviceControlRequest` API call.
6.  Upon a successful `ApiResponse`, the app waits for a WebSocket update with the new ERD values.
7.  The WebSocket update triggers the UI update flow (see Section 7), and the cell reverts to the "Normal" state.

## 6. **Popup and Modal Logic**

### **Popup Trigger Conditions**
A confirmation popup is displayed before performing a destructive action like resetting a filter's lifecycle.

- **Condition**: The `actionButton` is tapped when the cell is in the `.replace` state for the `.waterFilter` type.
- **Code Trigger**:
  ```swift
  // In CoffeeMakerPresenter.swift
  func handleFilterAction() {
      // Assuming 'deviceState' holds the latest ERD values
      if deviceState.filterStatus == .replace {
          view?.showResetConfirmationPopup(for: .waterFilter)
      } else {
          // ... handle "Order Now" logic
      }
  }
  ```

### **Handler Location**
The `CoffeeMakerDetailsViewController` is responsible for presenting the `UIAlertController`.

```swift
// In CoffeeMakerDetailsViewController.swift
func showResetConfirmationPopup(for accessory: AccessoryViewModel.AccessoryType) {
    let title = "Confirm Reset"
    let message = "Have you replaced the water filter? This will reset the filter lifecycle."
    
    let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
    
    let confirmAction = UIAlertAction(title: "Confirm", style: .destructive) { [weak self] _ in
        self?.presenter.confirmResetFilter()
    }
    
    let cancelAction = UIAlertAction(title: "Cancel", style: .cancel)
    
    alert.addAction(confirmAction)
    alert.addAction(cancelAction)
    
    present(alert, animated: true)
}
```

### **User Interactions**
- **Tap "Confirm"**: Calls `presenter.confirmResetFilter()`, which initiates the API call to reset the ERD value.
- **Tap "Cancel"**: Dismisses the alert with no further action.

## 7. **Update Flow**

The cell is updated as part of the `ViewController`'s standard data refresh cycle, typically initiated by a WebSocket event.

### **Specific Method Calls**
1.  **WebSocket Event**: A WebSocket message is received containing updated ERD values for the coffee maker.
2.  **Data Repository**: A `DeviceRepository` parses the message and updates its internal `Device` model. It notifies its subscribers (the `Presenter`) of the change.
3.  **Presenter Update**: `CoffeeMakerPresenter.deviceDidUpdate(device:)` is called.
4.  **ViewModel Creation**: The presenter creates new `AccessoryViewModel`s from the updated device model.
    ```swift
    // In CoffeeMakerPresenter.swift
    func deviceDidUpdate(device: CoffeeMakerDevice) {
        let filterVM = createFilterViewModel(from: device)
        let descalerVM = createDescalerViewModel(from: device)
        view?.updateAccessories(viewModels: [filterVM, descalerVM])
    }
    ```
5.  **ViewController Update**: The `ViewController` receives the new view models and updates its data source.
    ```swift
    // In CoffeeMakerDetailsViewController.swift
    private var accessoryViewModels: [AccessoryViewModel] = []

    func updateAccessories(viewModels: [AccessoryViewModel]) {
        self.accessoryViewModels = viewModels
        self.collectionView.reloadData() // Or perform batch updates for better performance
    }
    ```
6.  **Cell Configuration**: `collectionView(_:cellForItemAt:)` is called, which dequeues and configures the `CoffeeMakerAccessoryCell` with the new view model.
    ```swift
    // In CoffeeMakerDetailsViewController.swift
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CoffeeMakerAccessoryCell", for: indexPath) as! CoffeeMakerAccessoryCell
        let viewModel = accessoryViewModels[indexPath.item]
        cell.delegate = self
        cell.configure(with: viewModel)
        return cell
    }
    ```

## 8. **Test Skeletons**

Unit tests should be created to verify that the cell configures its UI correctly for each possible view model state.

**File**: `CafeHomeTests/ViewLayer/Appliances/CoffeeMaker/View/CoffeeMakerAccessoryCellTests.swift`
```swift
import XCTest
@testable import CafeHome

class CoffeeMakerAccessoryCellTests: XCTestCase {

    var cell: CoffeeMakerAccessoryCell!

    override func setUp() {
        super.setUp()
        // Load the cell from a NIB or storyboard
        let nib = UINib(nibName: "CoffeeMakerAccessoryCell", bundle: nil)
        cell = nib.instantiate(withOwner: self, options: nil).first as? CoffeeMakerAccessoryCell
    }

    func testConfigure_whenStatusIsOK_setsCorrectUI() {
        // Arrange
        let viewModel = AccessoryViewModel(
            type: .waterFilter,
            title: "Water Filter",
            icon: nil,
            status: .ok,
            statusText: "Life: 100%",
            actionButtonTitle: nil
        )

        // Act
        cell.configure(with: viewModel)

        // Assert
        XCTAssertEqual(cell.titleLabel.text, "Water Filter")
        XCTAssertEqual(cell.statusLabel.text, "Life: 100%")
        XCTAssertEqual(cell.statusLabel.textColor, .label)
        XCTAssertTrue(cell.actionButton.isHidden)
    }

    func testConfigure_whenStatusIsWarning_setsCorrectUI() {
        // Arrange
        let viewModel = AccessoryViewModel(
            type: .waterFilter,
            title: "Water Filter",
            icon: nil,
            status: .warning,
            statusText: "Replacement Recommended",
            actionButtonTitle: "Order Now"
        )

        // Act
        cell.configure(with: viewModel)

        // Assert
        XCTAssertEqual(cell.statusLabel.textColor, .systemYellow)
        XCTAssertFalse(cell.actionButton.isHidden)
        XCTAssertEqual(cell.actionButton.title(for: .normal), "Order Now")
    }

    func testConfigure_whenStatusIsReplace_setsCorrectUI() {
        // Arrange
        let viewModel = AccessoryViewModel(
            type: .waterFilter,
            title: "Water Filter",
            icon: nil,
            status: .replace,
            statusText: "Replace Filter Now",
            actionButtonTitle: "Reset Filter"
        )

        // Act
        cell.configure(with: viewModel)

        // Assert
        XCTAssertEqual(cell.statusLabel.textColor, .systemRed)
        XCTAssertFalse(cell.actionButton.isHidden)
        XCTAssertEqual(cell.actionButton.title(for: .normal), "Reset Filter")
    }
    
    func testActionButtonTap_whenTapped_callsDelegate() {
        // Arrange
        let mockDelegate = MockAccessoryCellDelegate()
        cell.delegate = mockDelegate
        
        let viewModel = AccessoryViewModel(
            type: .waterFilter,
            title: "Water Filter",
            icon: nil,
            status: .replace,
            statusText: "Replace Filter Now",
            actionButtonTitle: "Reset Filter"
        )
        cell.configure(with: viewModel) // Configure first to set internal viewModel

        // Act
        cell.actionButton.sendActions(for: .touchUpInside)

        // Assert
        XCTAssertTrue(mockDelegate.didCallAccessoryCellDidTapAction)
        XCTAssertEqual(mockDelegate.lastTappedType, .waterFilter)
    }
}

// Mock delegate for testing
class MockAccessoryCellDelegate: CoffeeMakerAccessoryCellDelegate {
    var didCallAccessoryCellDidTapAction = false
    var lastTappedType: AccessoryViewModel.AccessoryType?

    func accessoryCellDidTapAction(for cell: CoffeeMakerAccessoryCell, type: AccessoryViewModel.AccessoryType) {
        didCallAccessoryCellDidTapAction = true
        lastTappedType = type
    }
}
```