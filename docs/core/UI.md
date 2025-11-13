# UI System

## Overview

The UI system manages window types and UI state through the WindowType enum. Windows are used for various game interactions including crafting, vendors, quests, dialogs, admin panels, and trading.

## WindowType Enum

### Enum Definition

```typescript
enum WindowType {
    Crafting,
    Vendor,
    Quest,
    Dialog,
    ADM,
    Trade
}
```

## Window Types

### Crafting Window

**Type:** `WindowType.Crafting`

**Purpose:**
- Crafting interface
- Recipe selection
- Resource display
- Crafting progress

**Usage:**
- Open crafting dialog
- Show crafting recipes
- Display resources
- Craft items

### Vendor Window

**Type:** `WindowType.Vendor`

**Purpose:**
- Vendor interaction
- Buy/sell items
- Vendor inventory
- Transaction interface

**Usage:**
- Open vendor dialog
- Display vendor items
- Buy items
- Sell items

### Quest Window

**Type:** `WindowType.Quest`

**Purpose:**
- Quest interface
- Quest list
- Quest details
- Quest progress

**Usage:**
- Open quest log
- View quest details
- Track progress
- Accept/complete quests

### Dialog Window

**Type:** `WindowType.Dialog`

**Purpose:**
- NPC dialogue
- Conversation interface
- Dialog options
- Story progression

**Usage:**
- Open NPC dialog
- Display dialogue
- Show options
- Handle responses

### Admin Window

**Type:** `WindowType.ADM`

**Purpose:**
- Admin panel
- Administrative tools
- Server management
- Debug interface

**Usage:**
- Admin access only
- Server controls
- Debug tools
- Management interface

### Trade Window

**Type:** `WindowType.Trade`

**Purpose:**
- Player trading
- Trade interface
- Item exchange
- Trade confirmation

**Usage:**
- Open trade window
- Add/remove items
- Set gold amount
- Accept trade

## Window Management

### Window Opening

**Process:**
1. Determine window type
2. Create window data
3. Send OpenWindow packet
4. Client displays window

**Packet:**
- OpenWindow packet
- WindowType enum
- Window data

### Window Closing

**Process:**
1. Determine window type
2. Send CloseWindow packet
3. Client closes window
4. Cleanup state

**Packet:**
- CloseWindow packet
- WindowType enum
- Cleanup data

## UI State Management

### Window State

**State Tracking:**
- Current open window
- Window data
- Interaction state
- UI state synchronization

### Window Data

**Data Structure:**
- Window type
- Window content
- Interaction data
- State information

## Integration

### Crafting Integration

**Crafting Window:**
- Opens on crafting interaction
- Shows available recipes
- Displays resources
- Handles crafting requests

### Vendor Integration

**Vendor Window:**
- Opens on vendor interaction
- Shows vendor inventory
- Handles buy/sell
- Manages transactions

### Quest Integration

**Quest Window:**
- Opens quest log
- Shows active quests
- Displays quest details
- Tracks progress

### Trade Integration

**Trade Window:**
- Opens on trade request
- Shows trade interface
- Manages trade state
- Handles trade completion

## Implementation Notes

### Window Lifecycle

**Opening:**
1. Request window open
2. Validate permissions
3. Create window data
4. Send packet
5. Client displays

**Closing:**
1. Request window close
2. Cleanup state
3. Send packet
4. Client closes

### State Synchronization

**Synchronization:**
- Server manages state
- Client displays UI
- Packets sync state
- Consistent experience

## Related Documentation

- [UI_PACKETS.md](../packets/UI_PACKETS.md) - UI packets
- [CRAFTING.md](../items/CRAFTING.md) - Crafting system
- [TRADE.md](../player/TRADE.md) - Trade system
- [QUESTS.md](../player/QUESTS.md) - Quest system

