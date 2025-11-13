# UI Widgets System Documentation

## Overview

The UI Widgets system provides a reactive UI architecture using Unreal Engine's UMG plugin, similar to React/Vue frameworks. The system leverages delegates and the subsystem architecture to enable automatic, real-time UI updates without unnecessary manual bindings. Widgets receive updates automatically through delegate subscriptions, creating a reactive data flow that keeps UI synchronized with game state.

## Key Features

- **Reactive Architecture**: Similar to React/Vue with automatic updates
- **Delegate-Based Updates**: Real-time updates via delegates from subsystems
- **No Manual Bindings**: Automatic synchronization without manual binding code
- **UMG Integration**: Uses Unreal Engine's UMG plugin for widget creation
- **C++/Blueprint Hybrid**: C++ for logic and data, Blueprint for UI layout
- **Subsystem Integration**: Leverages GameInstanceSubsystem delegates
- **Real-Time Updates**: UI updates automatically when data changes
- **Efficient Updates**: Only updates when data actually changes

## Architecture Overview

### Reactive Data Flow

**Data Flow Pattern:**
```
Subsystem (Data Source)
    ↓
Delegate Broadcast (Data Changed)
    ↓
Widget Subscription (Automatic)
    ↓
UI Update (Automatic)
```

**Key Components:**
1. **Subsystem**: Holds game state and data
2. **Delegates**: Broadcast changes when data updates
3. **Widgets**: Subscribe to delegates and update automatically
4. **C++ Base Classes**: Provide reactive functionality
5. **Blueprint Widgets**: Extend C++ classes for UI layout

### Similarity to React/Vue

**Reactive Principles:**
- **Data-Driven**: UI reflects data state automatically
- **One-Way Data Flow**: Data flows from subsystem to widgets
- **Automatic Updates**: UI updates when data changes
- **Component-Based**: Widgets are reusable components
- **State Management**: Centralized state in subsystems

**Key Differences:**
- Uses delegates instead of virtual DOM
- C++ performance with Blueprint flexibility
- Unreal Engine's UMG for rendering
- Native integration with game systems

## Subsystem Integration

### Delegate System

**Subsystem Delegates:**
- Subsystems broadcast delegates when data changes
- Widgets subscribe to relevant delegates
- Automatic updates when delegates fire
- No manual polling or checking required

**Delegate Types:**
- **Single Delegate**: One parameter, simple updates
- **Multicast Delegate**: Multiple subscribers, broadcast updates
- **Dynamic Delegates**: Blueprint-accessible delegates
- **Event Delegates**: One-way communication (subsystem → widget)

**Example Delegate Declaration:**
```cpp
// In Subsystem (C++)
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnPlayerGoldChanged, int32, NewGold);
FOnPlayerGoldChanged OnPlayerGoldChanged;

// Broadcast when gold changes
void UpdatePlayerGold(int32 NewGold) {
    CurrentGold = NewGold;
    OnPlayerGoldChanged.Broadcast(NewGold);
}
```

### Widget Subscription

**Automatic Subscription:**
- Widgets subscribe to delegates in C++ base class
- Subscription happens automatically on widget creation
- Unsubscription on widget destruction
- No manual subscription code needed

**Subscription Pattern:**
```cpp
// In Widget Base Class (C++)
void UReactiveWidget::NativeConstruct() {
    Super::NativeConstruct();
    
    // Subscribe to subsystem delegates
    if (UToSGameInstance* GameInstance = GetGameInstance<UToSGameInstance>()) {
        if (UPlayerDataSubsystem* PlayerSubsystem = GameInstance->GetSubsystem<UPlayerDataSubsystem>()) {
            PlayerSubsystem->OnPlayerGoldChanged.AddDynamic(this, &UReactiveWidget::OnGoldChanged);
        }
    }
}

void UReactiveWidget::NativeDestruct() {
    // Unsubscribe from delegates
    if (UToSGameInstance* GameInstance = GetGameInstance<UToSGameInstance>()) {
        if (UPlayerDataSubsystem* PlayerSubsystem = GameInstance->GetSubsystem<UPlayerDataSubsystem>()) {
            PlayerSubsystem->OnPlayerGoldChanged.RemoveDynamic(this, &UReactiveWidget::OnGoldChanged);
        }
    }
    
    Super::NativeDestruct();
}
```

## Widget Base Classes

### Reactive Widget Base

**UReactiveWidget (C++):**
- Base class for all reactive widgets
- Handles delegate subscription/unsubscription
- Provides update methods for common data types
- Blueprint-callable update functions

**Key Features:**
- Automatic delegate subscription
- Update methods for common types (int, float, string, bool)
- Blueprint-accessible update functions
- Efficient update batching

**Base Class Structure:**
```cpp
UCLASS(Abstract)
class ATOS_API UReactiveWidget : public UUserWidget
{
    GENERATED_BODY()

public:
    virtual void NativeConstruct() override;
    virtual void NativeDestruct() override;

protected:
    // Update methods (Blueprint-callable)
    UFUNCTION(BlueprintImplementableEvent, Category = "Reactive")
    void OnGoldChanged(int32 NewGold);
    
    UFUNCTION(BlueprintImplementableEvent, Category = "Reactive")
    void OnHealthChanged(float NewHealth, float MaxHealth);
    
    UFUNCTION(BlueprintImplementableEvent, Category = "Reactive")
    void OnManaChanged(float NewMana, float MaxMana);
    
    // Helper methods
    UFUNCTION(BlueprintCallable, Category = "Reactive")
    void UpdateTextBlock(UTextBlock* TextBlock, const FString& NewText);
    
    UFUNCTION(BlueprintCallable, Category = "Reactive")
    void UpdateProgressBar(UProgressBar* ProgressBar, float Percent);
};
```

### Specialized Widget Bases

**UInventoryWidget (C++):**
- Extends UReactiveWidget
- Handles inventory-specific updates
- Item add/remove/update delegates
- Inventory slot management

**UPlayerStatsWidget (C++):**
- Extends UReactiveWidget
- Handles player stat updates
- Health/mana/stamina delegates
- Level and experience delegates

**UActionBarWidget (C++):**
- Extends UReactiveWidget
- Handles actionbar updates
- Slot change delegates
- Cooldown update delegates

## C++ vs Blueprint Separation

### C++ Responsibilities

**C++ Handles:**
- Delegate subscription/unsubscription logic
- Update method implementations
- Data transformation and formatting
- Performance-critical operations
- Complex logic and calculations

**C++ Base Classes:**
- Provide reactive functionality
- Handle delegate management
- Provide Blueprint-callable update methods
- Manage widget lifecycle

### Blueprint Responsibilities

**Blueprint Handles:**
- UI layout and visual design
- Widget positioning and styling
- Visual effects and animations
- User interaction handling
- Simple data display

**Blueprint Widgets:**
- Extend C++ base classes
- Define UI layout (UMG Designer)
- Implement visual updates
- Handle user input
- Create visual feedback

### Hybrid Approach

**Best Practices:**
- **C++**: Data logic, delegate handling, performance
- **Blueprint**: UI layout, visual design, simple logic
- **C++ Base**: Provides reactive functionality
- **Blueprint Child**: Uses base functionality, defines UI

**Example Structure:**
```
UReactiveWidget (C++ Base)
    ↓
UPlayerStatsWidget (C++ Specialized)
    ↓
WBP_PlayerStats (Blueprint - UI Layout)
```

## Real-Time Updates

### Update Flow

**1. Data Change:**
- Game state changes in subsystem
- Subsystem updates internal data
- Subsystem broadcasts delegate

**2. Delegate Broadcast:**
- Delegate fires with new data
- All subscribed widgets receive update
- Update happens immediately

**3. Widget Update:**
- Widget's update method called
- Widget updates UI elements
- Visual changes reflected immediately

**Example Update Flow:**
```cpp
// 1. Data changes in subsystem
PlayerSubsystem->SetPlayerGold(1500);

// 2. Subsystem broadcasts delegate
void UPlayerDataSubsystem::SetPlayerGold(int32 NewGold) {
    CurrentGold = NewGold;
    OnPlayerGoldChanged.Broadcast(NewGold); // Delegate fires
}

// 3. Widget receives update
void UReactiveWidget::OnGoldChanged(int32 NewGold) {
    // Blueprint-implemented function
    // Updates UI automatically
}
```

### Update Efficiency

**Optimization Strategies:**
- **Delegate Filtering**: Only subscribe to needed delegates
- **Batch Updates**: Group multiple updates together
- **Dirty Flagging**: Only update when data actually changes
- **Update Throttling**: Limit update frequency if needed

**Efficient Updates:**
- Subsystems only broadcast when data changes
- Widgets only update when delegates fire
- No unnecessary updates or polling
- Minimal overhead

## Widget Types

### Data Display Widgets

**Player Stats Widget:**
- Health, mana, stamina bars
- Level and experience display
- Stat values (strength, dexterity, etc.)
- Updates automatically from player data

**Inventory Widget:**
- Item slots and containers
- Item quantity displays
- Item tooltips
- Updates when inventory changes

**ActionBar Widget:**
- Action slots with icons
- Cooldown displays
- Enable/disable states
- Updates when actions change

### Interactive Widgets

**Settings Widget:**
- Configuration options
- Key binding interface
- Visual settings
- Saves to subsystem

**Trade Widget:**
- Trade interface
- Item exchange
- Gold display
- Updates from trade subsystem

**Quest Widget:**
- Quest list and details
- Progress tracking
- Objectives display
- Updates from quest subsystem

## Widget Creation Workflow

### Step 1: Create C++ Base Class

**Create Base Widget:**
1. Create C++ class extending UReactiveWidget
2. Add delegate subscription in NativeConstruct
3. Add delegate unsubscription in NativeDestruct
4. Add BlueprintImplementableEvent update methods
5. Compile and generate Blueprint base

**Example:**
```cpp
UCLASS()
class ATOS_API UPlayerStatsWidget : public UReactiveWidget
{
    GENERATED_BODY()

public:
    virtual void NativeConstruct() override {
        Super::NativeConstruct();
        // Subscribe to player stats delegates
    }

protected:
    UFUNCTION(BlueprintImplementableEvent)
    void OnHealthChanged(float Health, float MaxHealth);
    
    UFUNCTION(BlueprintImplementableEvent)
    void OnManaChanged(float Mana, float MaxMana);
};
```

### Step 2: Create Blueprint Widget

**Create Blueprint:**
1. Create Blueprint widget extending C++ base
2. Design UI layout in UMG Designer
3. Bind UI elements to update methods
4. Implement visual updates in Blueprint
5. Test reactive updates

**Blueprint Implementation:**
- Use UMG Designer for layout
- Bind TextBlocks, ProgressBars, etc.
- Implement update methods (OnHealthChanged, etc.)
- Add visual effects and animations

### Step 3: Configure Subsystem

**Subsystem Setup:**
1. Ensure subsystem has delegates declared
2. Broadcast delegates when data changes
3. Test delegate firing
4. Verify widget subscriptions

## Delegate Management

### Delegate Declaration

**In Subsystem Header:**
```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnPlayerGoldChanged, int32, NewGold);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnHealthChanged, float, Health, float, MaxHealth);

UCLASS()
class ATOS_API UPlayerDataSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintAssignable)
    FOnPlayerGoldChanged OnPlayerGoldChanged;
    
    UPROPERTY(BlueprintAssignable)
    FOnHealthChanged OnHealthChanged;
};
```

### Delegate Broadcasting

**Broadcasting Pattern:**
```cpp
void UPlayerDataSubsystem::SetPlayerGold(int32 NewGold) {
    if (CurrentGold != NewGold) {
        CurrentGold = NewGold;
        OnPlayerGoldChanged.Broadcast(NewGold);
    }
}

void UPlayerDataSubsystem::SetHealth(float Health, float MaxHealth) {
    if (CurrentHealth != Health || CurrentMaxHealth != MaxHealth) {
        CurrentHealth = Health;
        CurrentMaxHealth = MaxHealth;
        OnHealthChanged.Broadcast(Health, MaxHealth);
    }
}
```

### Widget Subscription

**Subscription Pattern:**
```cpp
void UReactiveWidget::NativeConstruct() {
    Super::NativeConstruct();
    
    if (UToSGameInstance* GameInstance = GetGameInstance<UToSGameInstance>()) {
        // Get subsystem
        UPlayerDataSubsystem* PlayerSubsystem = GameInstance->GetSubsystem<UPlayerDataSubsystem>();
        
        if (PlayerSubsystem) {
            // Subscribe to delegates
            PlayerSubsystem->OnPlayerGoldChanged.AddDynamic(this, &UReactiveWidget::OnGoldChanged);
            PlayerSubsystem->OnHealthChanged.AddDynamic(this, &UReactiveWidget::OnHealthChanged);
        }
    }
}
```

## Blueprint Implementation

### Blueprint Widget Setup

**Widget Creation:**
1. Create Blueprint widget class
2. Parent class: C++ reactive widget base
3. Open in UMG Designer
4. Add UI elements (TextBlocks, ProgressBars, etc.)
5. Bind elements to variables

### Update Method Implementation

**Implementing Updates:**
1. Override BlueprintImplementableEvent methods
2. Update UI elements in method
3. Add visual effects if needed
4. Test with data changes

**Example Blueprint Implementation:**
- Event: OnHealthChanged(Health, MaxHealth)
- Actions:
  - Set ProgressBar Percent = Health / MaxHealth
  - Set TextBlock Text = FString::Printf("%.0f / %.0f", Health, MaxHealth)
  - Play animation if health low

## Performance Considerations

### Optimization Strategies

**Delegate Efficiency:**
- Use multicast delegates for multiple subscribers
- Avoid excessive delegate firing
- Batch updates when possible
- Filter unnecessary updates

**Widget Efficiency:**
- Only subscribe to needed delegates
- Unsubscribe when widget hidden
- Use dirty flagging for expensive updates
- Cache frequently accessed data

**Update Batching:**
- Group related updates
- Throttle high-frequency updates
- Use update queues for complex operations
- Minimize UI redraws

### Best Practices

**Delegate Management:**
- Declare delegates in subsystem headers
- Use appropriate delegate types (single vs multicast)
- Broadcast only when data actually changes
- Clean up subscriptions properly

**Widget Design:**
- Keep update methods simple
- Avoid heavy calculations in update methods
- Use Blueprint for visual updates only
- Keep C++ logic in base classes

## Integration with Subsystems

### Player Data Subsystem

**Player Stats:**
- Health, mana, stamina delegates
- Level and experience delegates
- Stat value delegates
- Gold and currency delegates

**Widget Integration:**
- PlayerStatsWidget subscribes to player delegates
- Updates health bars, stat displays
- Real-time synchronization

### Inventory Subsystem

**Inventory Updates:**
- Item add/remove delegates
- Item quantity change delegates
- Inventory slot update delegates

**Widget Integration:**
- InventoryWidget subscribes to inventory delegates
- Updates item slots and displays
- Real-time inventory synchronization

### ActionBar Subsystem

**ActionBar Updates:**
- Slot change delegates
- Cooldown update delegates
- Enable/disable delegates

**Widget Integration:**
- ActionBarWidget subscribes to actionbar delegates
- Updates slot displays and cooldowns
- Real-time actionbar synchronization

## Example Implementation

### Complete Example: Player Health Widget

**1. C++ Base Class:**
```cpp
UCLASS()
class ATOS_API UPlayerHealthWidget : public UReactiveWidget
{
    GENERATED_BODY()

public:
    virtual void NativeConstruct() override {
        Super::NativeConstruct();
        
        if (UToSGameInstance* GameInstance = GetGameInstance<UToSGameInstance>()) {
            if (UPlayerDataSubsystem* PlayerSubsystem = GameInstance->GetSubsystem<UPlayerDataSubsystem>()) {
                PlayerSubsystem->OnHealthChanged.AddDynamic(this, &UPlayerHealthWidget::OnHealthChanged);
            }
        }
    }

protected:
    UFUNCTION(BlueprintImplementableEvent, Category = "Reactive")
    void OnHealthChanged(float Health, float MaxHealth);
};
```

**2. Blueprint Widget:**
- Extends UPlayerHealthWidget
- Contains ProgressBar for health
- Contains TextBlock for health text
- Implements OnHealthChanged event
- Updates ProgressBar and TextBlock

**3. Subsystem:**
```cpp
// In PlayerDataSubsystem
void SetHealth(float Health, float MaxHealth) {
    CurrentHealth = Health;
    CurrentMaxHealth = MaxHealth;
    OnHealthChanged.Broadcast(Health, MaxHealth);
}
```

**4. Result:**
- Widget automatically updates when health changes
- No manual binding code needed
- Real-time synchronization
- Efficient updates

## Related Documentation

- [SUBSYSTEM.md](./SUBSYSTEM.md) - Subsystem architecture and delegates
- [ARCHITECTURE.md](./ARCHITECTURE.md) - Client architecture overview
- [ACTIONBAR.md](./ACTIONBAR.md) - ActionBar widget implementation
- [ENTITY_SYNC.md](./ENTITY_SYNC.md) - Entity synchronization system

## See Also

- [Client README](./README.md) - Client systems overview
- [Server README](../server/README.md) - Server documentation
- [UI Packets](../packets/UI_PACKETS.md) - UI-related network packets

