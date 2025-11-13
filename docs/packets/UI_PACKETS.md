# UI Packets Documentation

## Overview

UI packets manage user interface windows, crafting interfaces, vendor interactions, trade windows, tooltips, and special messages. These packets control what the player sees and interacts with in the game interface.

## Server to Client Packets

### OpenWindow

**Type:** `ServerPacketType.OpenWindow`

**Purpose:** Open a UI window

**Data:**
```
[PacketType: byte]
[WindowType: byte]
```

**Window Types:**
- Inventory
- Character
- Skills
- Quest Log
- Settings
- Crafting
- Vendor
- Trade
- Guild
- Party

**Usage:**
- Player opens window
- Window triggered by action
- Window opened automatically
- Direct socket send

**Example:**
```typescript
packetOpenWindow.send(owner, WindowType.Inventory);
```

### CloseWindow

**Type:** `ServerPacketType.CloseWindow`

**Purpose:** Close a UI window

**Data:**
```
[PacketType: byte]
[WindowType: byte]
```

**Usage:**
- Player closes window
- Window closed automatically
- Window closed by action
- Direct socket send

**Example:**
```typescript
packetCloseWindow.send(owner, WindowType.Inventory);
```

### CraftingList

**Type:** `ServerPacketType.CraftingList`

**Purpose:** Send crafting recipe list for skill

**Data:**
```
[PacketType: byte]
[RecipesData: string (JSON)]
```

**Recipes JSON Structure:**
```json
[
    {
        "id": "RecipeId",
        "name": "Recipe Name",
        "result": "ItemName",
        "quantity": 1,
        "resources": [...],
        "skillLevel": 10,
        "category": "Weapons"
    }
]
```

**Conditions:**
- Entity exists and has socket
- Skill has recipes available
- Direct socket send

**Usage:**
- Open crafting window
- Crafting skill selected
- Recipe list refresh
- Crafting interface update

**Example:**
```typescript
packetCraftingList.send(owner, SkillName.Blacksmithing);
```

### CraftLog

**Type:** `ServerPacketType.CraftLog`

**Purpose:** Send crafting log message

**Data:**
```
[PacketType: byte]
[Message: string]
[Status: bool]
```

**Status:**
- `true`: Success message
- `false`: Error/failure message

**Usage:**
- Crafting attempt result
- Crafting feedback
- Error messages
- Success notifications

**Example:**
```typescript
packetCraftingLog.send(owner, "Item crafted successfully!", true);
```

### VendorWindow

**Type:** `ServerPacketType.VendorWindow`

**Purpose:** Open vendor interface with items

**Data:**
```
[PacketType: byte]
[VendorData: string (JSON)]
```

**Vendor JSON Structure:**
```json
{
    "vendorId": "VendorId",
    "vendorName": "Vendor Name",
    "items": [
        {
            "id": 123,
            "name": "Item Name",
            "price": 100,
            "quantity": 1
        }
    ],
    "buyback": [...]
}
```

**Usage:**
- Vendor interaction
- Vendor window open
- Vendor item list
- Buyback items

**Example:**
```typescript
packetVendorInteract.send(owner, vendorData);
```

### OpenTradeWindow

**Type:** `ServerPacketType.OpenTradeWindow`

**Purpose:** Open trade interface

**Data:**
```
[PacketType: byte]
[TradeId: string]
[OwnerContainer: string]
[OtherContainer: string]
[IsOwner: bool]
```

**Usage:**
- Trade request accepted
- Trade window open
- Trade interface initialization
- Container synchronization

**Example:**
```typescript
packetOpenTradeWindow.send(owner, {
    tradeId: "trade_123",
    ownerContainer: "trade_123_owner",
    otherContainer: "trade_123_other",
    isOwner: true
});
```

### ChangeStatusTrade

**Type:** `ServerPacketType.ChangeStatusTrade`

**Purpose:** Update trade acceptance status

**Data:**
```
[PacketType: byte]
[Status: bool]
```

**Status:**
- `true`: Trade accepted
- `false`: Trade not accepted

**Usage:**
- Trade acceptance toggle
- Trade status update
- Trade confirmation
- Trade negotiation

**Example:**
```typescript
packetChangeStatusTrade.send(owner, true);
```

### SpecialMessage

**Type:** `ServerPacketType.SpecialMessage`

**Purpose:** Send special formatted message

**Data:**
```
[PacketType: byte]
[Message: string]
```

**Usage:**
- Important notifications
- System announcements
- Event messages
- Special formatting

**Example:**
```typescript
packetSpecialMessage.send(owner, "Welcome to the game!");
```

### Tooltip

**Type:** `ServerPacketType.Tooltip`

**Purpose:** Send item tooltip information

**Data:**
```
[PacketType: byte]
[ItemRef: string]
[TooltipData: string (JSON)]
```

**Tooltip JSON Structure:**
```json
{
    "name": "Item Name",
    "description": "Item description",
    "quality": "Common",
    "stats": {...},
    "requirements": {...},
    "durability": 100,
    "maxDurability": 100
}
```

**Conditions:**
- Player has socket
- Item not already sent (deduplication)
- Direct socket send

**Usage:**
- Item hover
- Tooltip display
- Item information
- Item details

**Example:**
```typescript
packetTooltip.send(owner, "IronSword", tooltipData);
```

### RefreshTooltip

**Type:** `ServerPacketType.Tooltip`

**Purpose:** Refresh/update item tooltip

**Data:**
```
[PacketType: byte]
[ItemRef: string]
[TooltipData: string (JSON)]
```

**Usage:**
- Tooltip update
- Item stat changes
- Item modification
- Tooltip refresh

**Example:**
```typescript
packetRefreshTooltip.send(owner, "IronSword", updatedTooltipData);
```

## Client to Server Handlers

### OpenCraftingDialog

**Type:** `ClientPacketType.OpenCraftingDialog`

**Purpose:** Request to open crafting interface

**Data:**
```
[CraftingType: byte]
```

**Handler Logic:**
- Validates player not dead
- Gets crafting tool definition
- Opens crafting interface
- Sends crafting list

**Example:**
```csharp
CraftingType type = buffer.GetCraftingType();

if (!player.IsDead) {
    CraftingToolsDefination.GetTool(type)?.Interact(ctrl);
}
```

### CraftItem

**Type:** `ClientPacketType.CraftItem`

**Purpose:** Request to craft item

**Data:**
```
[Namespace: string]
[Variant: string (max 200 chars)]
```

**Handler Logic:**
- Validates player not dead
- Validates recipe exists
- Checks resources
- Checks skill level
- Creates item
- Removes resources
- Grants skill experience

**Example:**
```csharp
string Namespace = buffer.GetString();
string Variant = buffer.GetString(200);

if (!player.IsDead) {
    Crafting.CreateItem(ctrl, Namespace, Variant);
}
```

### CraftItemMulti

**Type:** `ClientPacketType.CraftItemMulti`

**Purpose:** Request to craft multiple items

**Data:**
```
[Namespace: string]
[Variant: string (max 200 chars)]
[Amount: int]
```

**Handler Logic:**
- Validates player not dead
- Validates recipe exists
- Checks resources for amount
- Checks skill level
- Creates items in loop
- Removes resources
- Grants skill experience

**Example:**
```csharp
string Namespace = buffer.GetString();
string Variant = buffer.GetString(200);
int Amount = buffer.GetInt();

if (!player.IsDead) {
    Crafting.CreateItem(ctrl, Namespace, Variant, Amount);
}
```

### TooltipItem

**Type:** `ClientPacketType.TooltipItem`

**Purpose:** Request item tooltip information

**Data:**
```
[ItemId: varuint]
[ItemContainerKind: varuint]
[AdditionalData: variable]
```

**Item Container Kinds:**
- Equipments (requires ItemClass and EquipmentItemSlot)
- Vendor (requires VendorId)
- Corpse (requires CorpseId)
- Stash (requires ContainerId)
- Inventory (default)

**Handler Logic:**
- Validates container kind
- Gets item from container
- Serializes item information
- Sends tooltip packet

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();
ItemContainerKind kind = (ItemContainerKind)buffer.GetVarUInt();

switch (kind) {
    case ItemContainerKind.Equipments:
        ItemClass itemClass = (ItemClass)buffer.GetVarUInt();
        EquipmentItemSlot slot = (EquipmentItemSlot)buffer.GetVarUInt();
        // Get item from equipment slot
        break;
    case ItemContainerKind.Vendor:
        uint vendorId = buffer.GetVarUInt();
        // Get item from vendor
        break;
    // ... other cases
}
```

## Packet Flow Examples

### Opening Crafting Window Flow

1. Client: Player interacts with crafting station
2. Client: Sends `OpenCraftingDialog` packet
3. Server: Validates interaction
4. Server: Gets crafting tool definition
5. Server: Sends `OpenWindow` packet (Crafting)
6. Server: Sends `CraftingList` packet with recipes
7. Client: Displays crafting window
8. Client: Shows available recipes

### Crafting Item Flow

1. Client: Player selects recipe and clicks craft
2. Client: Sends `CraftItem` packet
3. Server: Validates recipe
4. Server: Checks resources
5. Server: Checks skill level
6. Server: Creates item
7. Server: Removes resources
8. Server: Sends `CraftLog` packet (success)
9. Server: Sends `AddItemContainer` packet
10. Server: Sends `SkillExperience` packet
11. Client: Updates inventory
12. Client: Shows success message

### Vendor Interaction Flow

1. Client: Player interacts with vendor NPC
2. Client: Sends `Interact` packet
3. Server: Validates interaction
4. Server: Opens vendor interface
5. Server: Sends `OpenWindow` packet (Vendor)
6. Server: Sends `VendorWindow` packet with items
7. Client: Displays vendor window
8. Client: Shows vendor items

### Trade Window Flow

1. Client: Player accepts trade request
2. Server: Creates trade session
3. Server: Sends `OpenTradeWindow` packet
4. Server: Sends container data for both players
5. Client: Displays trade window
6. Client: Player adds items to trade
7. Client: Sends `ToggleItemFromTrade` packet
8. Server: Updates trade container
9. Server: Sends `ChangeStatusTrade` packet (false)
10. Client: Both players accept
11. Server: Sends `ChangeStatusTrade` packet (true)
12. Server: Completes trade
13. Server: Sends `CloseWindow` packet

## Implementation Notes

### Window Management

**Window States:**
- Windows tracked per player
- Only one window of each type open
- Window closing clears state
- Window data cached

**Window Types:**
- Modal windows (block other actions)
- Overlay windows (non-blocking)
- Persistent windows (stay open)
- Temporary windows (auto-close)

### Tooltip System

**Tooltip Deduplication:**
- Tracks sent tooltips per player
- Prevents duplicate tooltip sends
- Refreshes tooltip on update
- Clears tooltip cache on item change

**Tooltip Data:**
- Item name and description
- Item quality and rarity
- Item stats and bonuses
- Item requirements
- Item durability
- Item crafting information

### Crafting Interface

**Recipe Filtering:**
- Filter by skill level
- Filter by category
- Filter by available resources
- Search by name

**Crafting Feedback:**
- Success messages
- Error messages
- Resource warnings
- Skill level requirements

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [CONTAINER_PACKETS.md](./CONTAINER_PACKETS.md) - Container management packets
- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player-specific packets
- [CRAFTING.md](../items/CRAFTING.md) - Crafting system documentation
- [TRADE.md](../player/TRADE.md) - Trade system documentation

