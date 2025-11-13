# Container Packets Documentation

## Overview

Container packets manage item containers (inventory, stash, vendor, trade, corpse, etc.). These packets handle container opening/closing, item addition/removal, and item quantity updates.

## Server to Client Packets

### OpenContainer

**Type:** `ServerPacketType.OpenContainer`

**Purpose:** Open a container interface for player

**Data:**
```
[PacketType: byte]
[ContainerType: byte]
[ContainerId: string]
[EntityId: id (4 bytes)]
```

**Container Types:**
- Inventory
- Stash
- Vendor
- Trade
- Corpse
- Crafting station

**Usage:**
- Player interacts with container
- Container window opens
- Container data synchronized

**Example:**
```typescript
packetOpenContainer.send(owner, {
    containerId: "stash_123",
    type: ContainerType.Stash,
    entityId: stashEntity.mapIndex
});
```

### CloseContainer

**Type:** `ServerPacketType.CloseContainer`

**Purpose:** Close a container interface

**Data:**
```
[PacketType: byte]
[ContainerType: byte]
```

**Usage:**
- Player closes container window
- Container interaction ends
- Cleanup container state

**Example:**
```typescript
packetCloseContainer.send(owner, ContainerType.Stash);
```

### AddItemContainer

**Type:** `ServerPacketType.AddItemContainer`

**Purpose:** Add item to container slot

**Data:**
```
[PacketType: byte]
[ContainerId: string]
[SlotId: int32]
[ItemRef: string]
[ItemName: string]
[Amount: int32]
[ItemRarity: byte]
[GoldCost: int32]
[Weight: int32]
[ShowHint: bool]
```

**Usage:**
- Item added to container
- Item moved to container
- Item received from trade/vendor
- Initial container load

**Example:**
```typescript
packetAddItemContainer.send(owner, {
    containerId: "inventory_123",
    slotId: 5,
    itemRef: "IronSword",
    itemName: "Iron Sword",
    amount: 1,
    itemRarity: ItemQuality.Common,
    goldCost: 100,
    weight: 5
}, showHint);
```

### RemoveItemContainer

**Type:** `ServerPacketType.RemoveItemContainer`

**Purpose:** Remove item from container slot

**Data:**
```
[PacketType: byte]
[ContainerId: string]
[SlotId: int32]
[ItemRef: string]
```

**Usage:**
- Item removed from container
- Item consumed
- Item moved to another container
- Item sold/destroyed

**Example:**
```typescript
packetRemoveItemContainer.send(owner, {
    containerId: "inventory_123",
    slotId: 5,
    itemRef: "IronSword"
});
```

### ChangeItemAmountContainer

**Type:** `ServerPacketType.ChangeAmountItemContainer`

**Purpose:** Update item quantity in container slot

**Data:**
```
[PacketType: byte]
[ContainerId: string]
[SlotId: int32]
[Amount: int32]
```

**Usage:**
- Stackable item quantity changes
- Item consumed (partial)
- Item split/merged
- Quantity updates

**Example:**
```typescript
packetChangeItemAmountContainer.send(owner, {
    containerId: "inventory_123",
    slotId: 5,
    amount: 10
});
```

## Client to Server Handlers

### MoveItem

**Type:** `ClientPacketType.MoveItem`

**Purpose:** Move item between containers

**Data:**
```
[FromContainerId: varuint]
[ToContainerId: varuint]
[ItemId: varuint]
[Quantity: varuint]
```

**Handler Logic:**
- Validates both containers are open
- Removes item from source container
- Adds item to destination container
- Handles gold coin special case (adds to player gold)

**Special Cases:**
- Gold Coin: Adds directly to player gold instead of container
- Stackable items: Merges if same item type
- Container validation: Both containers must be open

**Example:**
```csharp
uint fromContainerId = buffer.GetVarUInt();
uint toContainerId = buffer.GetVarUInt();
uint itemId = buffer.GetVarUInt();
uint quantity = buffer.GetVarUInt();

if (fromContainer.TryRemove(itemId, quantity) != null) {
    if (item.Name == "Gold Coin") {
        player.AddGoldCoins(quantity);
    } else {
        toContainer.Add(item);
    }
}
```

### SwapItem

**Type:** `ClientPacketType.SwapItem`

**Purpose:** Swap items between container slots

**Data:**
```
[SourceContainerId: int]
[SourceSlotIndex: byte]
[TargetContainerId: int]
[TargetSlotIndex: byte]
[Count: ushort]
```

**Handler Logic:**
- Validates containers are open
- Validates slots are valid
- Handles stackable item merging
- Handles container closing (backpacks)
- Handles transactional containers (trade)

**Note:** Implementation currently commented out in C# version

**Example:**
```csharp
int sourceContainerId = buffer.GetInt();
int sourceSlotIndex = buffer.GetByte();
int targetContainerId = buffer.GetInt();
int targetSlotIndex = buffer.GetByte();
ushort count = buffer.GetUShort();
// Swap logic
```

### ChangeContainer

**Type:** `ClientPacketType.ChangeContainer`

**Purpose:** Change container view/filter

**Data:**
```
[ContainerId: varuint]
[Filter: byte] (optional)
```

**Handler Logic:**
- Changes container view
- Applies filters
- Updates container display

## Container Types

### Inventory

**Properties:**
- Player's main inventory
- Fixed size
- Always accessible
- ContainerId: "inventory_{playerId}"

### Stash

**Properties:**
- Storage container
- Persistent storage
- Access via stash entity
- ContainerId: "stash_{stashId}"

### Vendor

**Properties:**
- NPC vendor container
- Buy/sell interface
- Vendor-specific items
- ContainerId: "vendor_{vendorId}"

### Trade

**Properties:**
- Trade session container
- Two containers (owner/other)
- Transactional (can cancel)
- ContainerId: "trade_{tradeId}_{owner|other}"

### Corpse

**Properties:**
- Loot container
- Temporary (despawns)
- Contains dropped items
- ContainerId: "corpse_{corpseId}"

### Crafting Station

**Properties:**
- Crafting interface
- Recipe display
- Resource display
- ContainerId: "crafting_{stationId}"

## Packet Flow Examples

### Opening Container Flow

1. Client: Player interacts with container entity
2. Client: Sends `Interact` packet
3. Server: Validates interaction
4. Server: Opens container
5. Server: Sends `OpenContainer` packet
6. Server: Sends `AddItemContainer` for each item
7. Client: Displays container window

### Moving Item Flow

1. Client: Player drags item to another container
2. Client: Sends `MoveItem` packet
3. Server: Validates move
4. Server: Removes from source
5. Server: Adds to destination
6. Server: Sends `RemoveItemContainer` to source
7. Server: Sends `AddItemContainer` to destination
8. Client: Updates both containers

### Stackable Item Update Flow

1. Client: Player consumes stackable item
2. Client: Sends `UseItem` packet
3. Server: Validates use
4. Server: Decrements quantity
5. Server: Sends `ChangeItemAmountContainer`
6. Client: Updates item quantity display

## Implementation Notes

### Container ID Format

**Format:**
- `{type}_{identifier}`
- Examples: `inventory_123`, `stash_456`, `trade_789_owner`

**Uniqueness:**
- Container IDs must be unique per player session
- Trade containers have owner/other suffix
- Vendor containers use vendor entity ID

### Slot Management

**Slot IDs:**
- 0-based indexing
- Sequential slot numbers
- Fixed slot count per container type

**Slot Validation:**
- Server validates slot exists
- Server validates slot is empty (for add)
- Server validates slot has item (for remove)

### Item Reference System

**ItemRef:**
- Item namespace/identifier
- Used for item type identification
- Different from ItemId (instance ID)

**ItemId:**
- Unique item instance identifier
- Used for specific item operations
- Required for move/remove operations

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player packets
- [UI_PACKETS.md](./UI_PACKETS.md) - UI packets (vendor, trade)
- [CONTAINERS.md](../items/CONTAINERS.md) - Container system

