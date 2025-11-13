# ActionBar System Documentation

## Overview

The ActionBar system provides quick access to spells, consumables, and mounts through hotkey-bound slots. The system consists of a configurable number of slots (10 or 20) that can be bound to keys and configured in settings. Each slot supports cooldown tracking, enable/disable states, inventory integration, and quantity display.

## Key Features

- **Configurable Slots**: 10 or 20 slots configurable in settings
- **Hotkey Binding**: Each slot bound to a key (1-0, or extended keys)
- **Multiple Item Types**: Supports spells, consumables, and mounts
- **Cooldown System**: Visual cooldown display per slot
- **Enable/Disable States**: Slots automatically disable when items unavailable
- **Inventory Integration**: Direct integration with inventory for consumables
- **Quantity Display**: Shows item quantity for consumables
- **Server Synchronization**: Server tracks which actions are set in each slot
- **Action Call System**: Client sends action call packets when slot activated

## ActionBar Structure

### Slot Widget

**Slot Widget (`USlot`):**
- Base widget for actionbar slots
- Supports multiple item types
- Cooldown visualization
- Enable/disable states
- Inventory integration
- Quantity label display

**Slot Properties:**
- **Slot Index**: Slot position in actionbar (0-9 or 0-19)
- **Bound Key**: Key bound to this slot
- **Item Type**: Type of item in slot (Spell, Consumable, Mount)
- **Item Reference**: Reference to actual item/spell
- **Cooldown State**: Current cooldown status
- **Enabled State**: Whether slot is enabled
- **Quantity**: Item quantity (for consumables)

### ActionBar Widget

**ActionBar Widget (`UActionBar`):**
- Container for all slot widgets
- Manages slot layout and positioning
- Handles key bindings
- Coordinates with settings system
- Updates slot states

**ActionBar Properties:**
- **Slot Count**: Number of slots (10 or 20)
- **Slot Array**: Array of slot widgets
- **Key Bindings**: Mapping of keys to slots
- **Settings Integration**: Reads configuration from settings

## Slot Types

### Spell Slots

**Spell Slot:**
- Contains a spell reference
- Displays spell icon
- Shows cooldown when spell on cooldown
- Enabled when spell available
- Disabled when spell unavailable (not learned, on cooldown, insufficient resources)

**Spell Slot Behavior:**
- Click or key press triggers spell cast
- Cooldown displayed visually
- Icon changes based on spell state
- Tooltip shows spell details

### Consumable Slots

**Consumable Slot:**
- Contains consumable item reference
- Direct inventory integration
- Shows item quantity
- Enabled when item available in inventory
- Disabled when item not in inventory or quantity 0

**Consumable Slot Behavior:**
- Click or key press uses consumable
- Quantity label updates in real-time
- Slot disables when quantity reaches 0
- Re-enables when item added to inventory
- Inventory changes reflected immediately

### Mount Slots

**Mount Slot:**
- Contains mount reference
- Shows mount icon
- Enabled when mount available
- Disabled when mount unavailable

**Mount Slot Behavior:**
- Click or key press summons/dismounts mount
- State changes based on mount status
- Visual feedback for mount state

## Slot Widget Implementation

### Cooldown System

**Cooldown Display:**
- Visual cooldown overlay on slot
- Cooldown timer countdown
- Progress bar or radial fill
- Color changes during cooldown
- Updates in real-time

**Cooldown States:**
- **Ready**: No cooldown, slot available
- **On Cooldown**: Cooldown active, slot disabled
- **Cooldown Complete**: Cooldown finished, slot re-enabled

**Cooldown Integration:**
- Receives cooldown updates from server
- Updates visual display
- Manages slot enable/disable state
- Handles cooldown completion

### Enable/Disable System

**Enable Conditions:**
- Item/spell available
- Not on cooldown
- Sufficient resources (mana/stamina for spells)
- Item in inventory (for consumables)
- Spell learned (for spells)

**Disable Conditions:**
- Item/spell unavailable
- On cooldown
- Insufficient resources
- Item not in inventory (consumables)
- Spell not learned (spells)

**State Management:**
- Automatic state updates
- Real-time inventory checking
- Resource validation
- Visual feedback (grayed out, disabled appearance)

### Inventory Integration

**Direct Integration:**
- Slot directly checks inventory for consumables
- Real-time inventory updates
- Quantity synchronization
- Automatic enable/disable based on availability

**Inventory Checking:**
- Checks if item exists in inventory
- Checks item quantity
- Updates quantity label
- Handles item removal (quantity 0)
- Handles item addition (re-enable slot)

**Quantity Display:**
- Shows current quantity for consumables
- Updates when inventory changes
- Hides quantity for non-consumables
- Format: "x[quantity]" or "[quantity]"

### Length Label

**Quantity Label:**
- Displays item quantity for consumables
- Positioned on slot (typically bottom-right)
- Updates in real-time
- Hidden for spells and mounts
- Styled for visibility

**Label Format:**
- Numeric display: "5", "10", "99+"
- Optional "+" for quantities over display limit
- Color coding (optional)
- Font size and style configurable

## Key Binding System

### Default Key Bindings

**Standard Bindings (10 slots):**
- **1**: Slot 0
- **2**: Slot 1
- **3**: Slot 2
- **4**: Slot 3
- **5**: Slot 4
- **6**: Slot 5
- **7**: Slot 6
- **8**: Slot 7
- **9**: Slot 8
- **0**: Slot 9

**Extended Bindings (20 slots):**
- **1-0**: Slots 0-9 (same as above)
- **Additional keys**: Slots 10-19 (configurable)
- Can use F1-F10, or other key combinations
- Configurable in settings

### Key Binding Configuration

**Settings Integration:**
- Key bindings stored in settings
- Configurable per slot
- Can rebind keys
- Supports multiple keys per slot (optional)
- Settings persist across sessions

**Binding Management:**
- Settings menu for key binding
- Drag-and-drop key assignment
- Conflict detection
- Reset to defaults option
- Per-character or account-wide (configurable)

## Action Call System

### Action Call Packet

**When Slot Activated:**
- Player presses bound key OR
- Player double-clicks slot in HUD
- Client sends `ActionCall` packet to server
- Packet contains actionbar slot index

**Packet Format:**
```
[PacketType: ActionCall]
[SlotIndex: uint8]  // 0-9 or 0-19
```

**Packet Handling:**
- Server receives ActionCall packet
- Server looks up action in slot index
- Server validates action can be executed
- Server executes action
- Server sends response/update

### Action Execution Flow

**Client Side:**
1. Player activates slot (key press or click)
2. Client validates slot has action
3. Client sends ActionCall packet
4. Client shows visual feedback (pressing animation)
5. Client waits for server response

**Server Side:**
1. Server receives ActionCall packet
2. Server validates slot index
3. Server retrieves action from player's actionbar
4. Server validates action can be executed
5. Server executes action
6. Server sends action result/update

**Response Handling:**
- Success: Action executed, cooldown started
- Failure: Error message, action not executed
- Cooldown: Cooldown information sent
- Resource Update: Mana/stamina updates sent

## Server Synchronization

### ActionBar State

**Server Tracks:**
- Which actions are set in each slot
- Slot index to action mapping
- Action types (spell, consumable, mount)
- Action references (spell ID, item ID, mount ID)

**State Persistence:**
- ActionBar state saved in player data
- Persisted across sessions
- Loaded on character login
- Synced to client on login

### State Updates

**When ActionBar Changes:**
- Player adds action to slot
- Player removes action from slot
- Player swaps actions between slots
- Server updates actionbar state
- Server sends update to client

**Update Packets:**
- `ActionBarAdd`: Add action to slot
- `ActionBarRemove`: Remove action from slot
- `ActionBarUpdate`: Update slot action
- `ActionBarFull`: Full actionbar state (on login)

## ActionBar Packets

### Client to Server Packets

**ActionCall:**
- Sent when slot activated
- Contains slot index
- Triggers action execution

**ActionBarAdd:**
- Add action to slot
- Contains slot index and action reference
- Server validates and adds

**ActionBarRemove:**
- Remove action from slot
- Contains slot index
- Server removes action

**ActionBarSwap:**
- Swap actions between slots
- Contains source and target slot indices
- Server swaps actions

### Server to Client Packets

**ActionBarFull:**
- Full actionbar state
- Sent on login
- Contains all slot assignments

**ActionBarUpdate:**
- Update single slot
- Contains slot index and action reference
- Updates slot on client

**ActionBarCooldown:**
- Cooldown update for slot
- Contains slot index and cooldown time
- Updates cooldown display

**ActionBarEnable/Disable:**
- Enable/disable slot
- Contains slot index and enabled state
- Updates slot state

## Visual Design

### Slot Appearance

**Normal State:**
- Full opacity
- Normal icon display
- Quantity label visible (if consumable)
- Ready for use

**Disabled State:**
- Reduced opacity (grayed out)
- Disabled icon appearance
- Cannot be activated
- Visual feedback

**Cooldown State:**
- Cooldown overlay
- Timer or progress bar
- Color changes
- Countdown display

**Pressed State:**
- Visual feedback when activated
- Brief animation
- Confirms activation
- Returns to normal/cooldown state

### Icon Display

**Spell Icons:**
- Spell-specific icon
- Changes based on spell state
- Cooldown overlay when on cooldown
- Tooltip with spell details

**Consumable Icons:**
- Item icon from inventory
- Quantity label overlay
- Updates when inventory changes
- Tooltip with item details

**Mount Icons:**
- Mount-specific icon
- State-based appearance
- Tooltip with mount details

## Settings Integration

### ActionBar Settings

**Slot Count:**
- Configurable: 10 or 20 slots
- Stored in settings
- Applied on login
- Can be changed in settings menu

**Key Bindings:**
- Configurable per slot
- Stored in settings
- Supports rebinding
- Conflict detection

**Display Options:**
- Show/hide actionbar
- Position on screen
- Size and scale
- Opacity settings

**Visual Options:**
- Cooldown display style
- Quantity label style
- Icon size
- Slot spacing

## Related Documentation

- [INPUT.md](./INPUT.md) - Input system and key bindings
- [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) - Player controller system
- [SPELLS.md](./SPELLS.md) - Client-side spells system
- [ACTIONBAR_PACKETS.md](../packets/ACTIONBAR_PACKETS.md) - ActionBar packet specifications

## See Also

- [Client README](../client/) - Client systems overview
- [Player Spells](../player/SPELLS.md) - Player spells system
- [Player Packets](../packets/PLAYER_PACKETS.md) - Player packet system

