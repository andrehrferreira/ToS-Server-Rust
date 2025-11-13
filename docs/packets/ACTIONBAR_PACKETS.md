# ActionBar Packets Documentation

## Overview

ActionBar packets handle communication between client and server for managing the actionbar system. The server maintains the authoritative state of which actions are assigned to each slot, and the client sends action call requests when slots are activated. The system supports adding, removing, and calling actions from actionbar slots.

## Key Features

- **Server Authority**: Server tracks which actions are in each slot
- **Action Call System**: Client sends slot index when slot activated
- **Add/Remove Actions**: Client can add or remove actions from slots
- **State Synchronization**: Server sends full actionbar state on login
- **Cooldown Updates**: Server sends cooldown information for slots
- **Enable/Disable Updates**: Server sends slot enable/disable states

## Client to Server Packets

### ActionCall

**Type:** `ClientPacketType.ActionCall`

**Purpose:** Request execution of action in actionbar slot

**Trigger:**
- Player presses hotkey bound to slot
- Player double-clicks slot in HUD

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]  // 0-9 for 10 slots, 0-19 for 20 slots
```

**Handler Logic:**
1. Validate slot index is within bounds
2. Retrieve action from player's actionbar at slot index
3. Validate action exists
4. Validate action can be executed (cooldown, resources, etc.)
5. Execute action
6. Send action result/update to client

**Validation:**
- Slot index must be valid (0-9 or 0-19)
- Action must exist in slot
- Action must not be on cooldown
- Player must have sufficient resources (mana/stamina)
- Action must be available (spell learned, item in inventory, etc.)

**Response:**
- Success: Action executed, cooldown started
- Failure: Error packet sent with reason
- Cooldown: Cooldown update packet sent
- Resource Update: Mana/stamina update sent

**Example:**
```rust
// Client sends
ActionCall { slot_index: 3 }

// Server handles
let action = player.actionbar.get_slot(3)?;
if action.can_execute(player)? {
    action.execute(player).await?;
    send_cooldown_update(player, 3, action.cooldown_duration()).await?;
}
```

### ActionBarAdd

**Type:** `ClientPacketType.ActionBarAdd`

**Purpose:** Add action to actionbar slot

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]
[ActionType: uint8]  // 0=Spell, 1=Consumable, 2=Mount
[ActionReference: varuint]  // Spell ID, Item ID, or Mount ID
```

**Handler Logic:**
1. Validate slot index is within bounds
2. Validate action type
3. Validate action reference exists
4. Check if player has access to action:
   - Spell: Check if spell is learned and in selected Codex tree
   - Consumable: Check if item exists in inventory
   - Mount: Check if mount is owned
5. Add action to slot
6. Save actionbar state
7. Send confirmation to client

**Validation:**
- Slot index must be valid
- Action type must be valid (0-2)
- Action reference must exist
- Player must have access to action
- Slot can be empty or overwrite existing action

**Response:**
- Success: ActionBarUpdate packet sent
- Failure: Error packet sent with reason

**Example:**
```rust
// Client sends
ActionBarAdd {
    slot_index: 5,
    action_type: 0,  // Spell
    action_reference: 123  // Spell ID
}

// Server handles
let spell = get_spell(123)?;
if player.has_spell(spell.id) && player.has_codex_tree(spell.tree_id)? {
    player.actionbar.set_slot(5, Action::Spell(spell.id))?;
    player.save_actionbar().await?;
    send_actionbar_update(player, 5, Action::Spell(spell.id)).await?;
}
```

### ActionBarRemove

**Type:** `ClientPacketType.ActionBarRemove`

**Purpose:** Remove action from actionbar slot

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]
```

**Handler Logic:**
1. Validate slot index is within bounds
2. Check if slot has action
3. Remove action from slot
4. Save actionbar state
5. Send confirmation to client

**Validation:**
- Slot index must be valid
- Slot must have an action (optional check)

**Response:**
- Success: ActionBarUpdate packet sent (empty slot)
- Failure: Error packet sent if slot index invalid

**Example:**
```rust
// Client sends
ActionBarRemove { slot_index: 3 }

// Server handles
player.actionbar.clear_slot(3)?;
player.save_actionbar().await?;
send_actionbar_update(player, 3, Action::None).await?;
```

### ActionBarSwap

**Type:** `ClientPacketType.ActionBarSwap`

**Purpose:** Swap actions between two slots

**Data:**
```
[PacketType: byte]
[SourceSlotIndex: uint8]
[TargetSlotIndex: uint8]
```

**Handler Logic:**
1. Validate both slot indices are within bounds
2. Retrieve actions from both slots
3. Swap actions
4. Save actionbar state
5. Send updates for both slots

**Validation:**
- Both slot indices must be valid
- Slots can be empty (swap with empty slot)

**Response:**
- Success: Two ActionBarUpdate packets sent (one per slot)
- Failure: Error packet sent

**Example:**
```rust
// Client sends
ActionBarSwap {
    source_slot_index: 2,
    target_slot_index: 7
}

// Server handles
let source_action = player.actionbar.get_slot(2)?;
let target_action = player.actionbar.get_slot(7)?;
player.actionbar.set_slot(2, target_action)?;
player.actionbar.set_slot(7, source_action)?;
player.save_actionbar().await?;
send_actionbar_update(player, 2, target_action).await?;
send_actionbar_update(player, 7, source_action).await?;
```

## Server to Client Packets

### ActionBarFull

**Type:** `ServerPacketType.ActionBarFull`

**Purpose:** Send complete actionbar state to client

**Trigger:**
- Player login
- Character selection
- Actionbar reset

**Data:**
```
[PacketType: byte]
[SlotCount: uint8]  // 10 or 20
[Slots: array of SlotData]
```

**SlotData Format:**
```
[SlotIndex: uint8]
[ActionType: uint8]  // 0=Spell, 1=Consumable, 2=Mount, 255=None
[ActionReference: varuint]  // Spell ID, Item ID, or Mount ID (0 if None)
[CooldownRemaining: float]  // Remaining cooldown in seconds (0 if ready)
[Enabled: bool]  // Whether slot is enabled
```

**Usage:**
- Sent on login to sync actionbar state
- Client updates all slots based on data
- Establishes initial actionbar state

**Example:**
```rust
// Server sends
ActionBarFull {
    slot_count: 10,
    slots: [
        SlotData { index: 0, action_type: 0, action_ref: 123, cooldown: 0.0, enabled: true },
        SlotData { index: 1, action_type: 1, action_ref: 456, cooldown: 0.0, enabled: true },
        SlotData { index: 2, action_type: 255, action_ref: 0, cooldown: 0.0, enabled: false },
        // ... more slots
    ]
}
```

### ActionBarUpdate

**Type:** `ServerPacketType.ActionBarUpdate`

**Purpose:** Update single actionbar slot

**Trigger:**
- Action added to slot
- Action removed from slot
- Action swapped between slots
- Action state changed

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]
[ActionType: uint8]  // 0=Spell, 1=Consumable, 2=Mount, 255=None
[ActionReference: varuint]  // Spell ID, Item ID, or Mount ID (0 if None)
[CooldownRemaining: float]  // Remaining cooldown in seconds (0 if ready)
[Enabled: bool]  // Whether slot is enabled
```

**Usage:**
- Sent when single slot changes
- Client updates specific slot
- More efficient than sending full actionbar

**Example:**
```rust
// Server sends
ActionBarUpdate {
    slot_index: 5,
    action_type: 0,  // Spell
    action_reference: 789,  // Spell ID
    cooldown_remaining: 0.0,
    enabled: true
}
```

### ActionBarCooldown

**Type:** `ServerPacketType.ActionBarCooldown`

**Purpose:** Update cooldown for actionbar slot

**Trigger:**
- Action executed (cooldown started)
- Cooldown completed
- Cooldown changed

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]
[CooldownRemaining: float]  // Remaining cooldown in seconds (0 if ready)
```

**Usage:**
- Sent when cooldown changes
- Client updates cooldown display
- Slot disabled during cooldown
- Slot re-enabled when cooldown completes

**Example:**
```rust
// Server sends
ActionBarCooldown {
    slot_index: 3,
    cooldown_remaining: 5.5  // 5.5 seconds remaining
}
```

### ActionBarEnable

**Type:** `ServerPacketType.ActionBarEnable`

**Purpose:** Enable or disable actionbar slot

**Trigger:**
- Item removed from inventory (consumable)
- Item added to inventory (consumable)
- Spell learned or unlearned
- Resources insufficient or sufficient
- Mount available or unavailable

**Data:**
```
[PacketType: byte]
[SlotIndex: uint8]
[Enabled: bool]  // Whether slot is enabled
```

**Usage:**
- Sent when slot enable state changes
- Client updates slot visual state
- Disabled slots grayed out
- Enabled slots normal appearance

**Example:**
```rust
// Server sends
ActionBarEnable {
    slot_index: 2,
    enabled: false  // Disable slot (e.g., consumable out of inventory)
}
```

## Server State Management

### ActionBar Storage

**Player ActionBar State:**
- Stored in player database record
- Array of slot assignments
- Each slot contains:
  - Action type (Spell, Consumable, Mount, None)
  - Action reference (Spell ID, Item ID, Mount ID)
  - Slot index

**State Structure:**
```rust
pub struct ActionBarSlot {
    pub slot_index: u8,
    pub action_type: ActionType,
    pub action_reference: u32,  // Spell ID, Item ID, or Mount ID
}

pub struct PlayerActionBar {
    pub slots: Vec<ActionBarSlot>,  // Up to 20 slots
    pub slot_count: u8,  // 10 or 20
}
```

### State Persistence

**Save Triggers:**
- Action added to slot
- Action removed from slot
- Actions swapped between slots
- Actionbar reset

**Load Triggers:**
- Player login
- Character selection
- Map change (if needed)

**Persistence:**
- Saved to database immediately on change
- Loaded on character login
- Synced to client via ActionBarFull packet

### State Validation

**Validation Checks:**
- Spell exists and is learned
- Spell is in selected Codex tree
- Consumable exists in inventory
- Mount is owned by player
- Slot indices are within bounds

**Invalid State Handling:**
- Invalid actions cleared from slots
- Slots set to empty if action unavailable
- Client notified of state changes
- State corrected on validation

## Action Execution Flow

### Execution Process

**1. Client Activation:**
- Player presses hotkey or clicks slot
- Client validates slot has action
- Client sends ActionCall packet

**2. Server Validation:**
- Server receives ActionCall packet
- Server validates slot index
- Server retrieves action from actionbar
- Server validates action can execute:
  - Not on cooldown
  - Sufficient resources
  - Action available
  - Valid target (if targeted)

**3. Action Execution:**
- Server executes action
- Server applies cooldown
- Server consumes resources
- Server sends updates:
  - Cooldown update
  - Resource update
  - Action result

**4. Client Update:**
- Client receives updates
- Client updates slot cooldown display
- Client updates resource bars
- Client shows action effects

## Related Documentation

- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player packet system
- [CLIENT_PACKETS.md](./CLIENT_PACKETS.md) - Client packet specifications
- [ACTIONBAR.md](../client/ACTIONBAR.md) - Client ActionBar implementation
- [SPELLS.md](../player/SPELLS.md) - Player spells system

## See Also

- [Packets Overview](./OVERVIEW.md) - Network packets overview
- [Server README](../server/README.md) - Server documentation
- [Client README](../client/) - Client systems overview

