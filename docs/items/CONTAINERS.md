# Container System Documentation

## Overview

The Container system manages item storage for various game entities and systems. Containers support weight limits, item stacking, equipment slots, and event-based notifications for item modifications. The system is designed to handle multiple container types including inventory, equipment slots, storage chests, guild storage, loot containers, vendor containers, and trade containers.

## Key Features

- **Multiple Container Types**: Inventory, equipment, storage, loot, vendor, trade containers
- **Weight System**: Containers track total weight and enforce weight limits
- **Item Stacking**: Automatic stacking of stackable items
- **Equipment Slots**: Specific slots for different equipment types
- **Event System**: Notifications for item additions, removals, and modifications
- **Weight-Based Mobility**: Player mobility affected by inventory weight (100%-200% weight range)
- **Item ID Management**: Unique item IDs for container items
- **Serialization**: Full container state serialization for persistence

## Container Types

### 1. Inventory Container

The main player inventory container.

**TypeScript Implementation:**
```typescript
// Inventory is typically a Map<string, Item> or similar structure
// Items are stored with unique IDs
```

**C# Implementation:**
```csharp
public class InventoryContainer : Dictionary<uint, Item>, ItemContainer {
    public uint Weight { get; set; }
    public CharacterEntity Owner { get; init; }
    public uint Id { get; set; }
    private uint LastId;
    
    // Methods for adding, removing, and managing items
}
```

**Key Features:**
- Weight tracking
- Item stacking support
- Unique item ID allocation
- Weight change notifications
- Owner reference for weight/strength calculations

### 2. Equipment Container

Container for equipped items with specific slots.

**C# Implementation:**
```csharp
public sealed class EquipmentsContainer : ItemContainer {
    public CharacterEntity Owner { get; init; }
    public uint Id { get; set; }
    
    // Equipment Slots
    public Weapon Weapon { get; set; }
    public Item OffHand { get; set; }
    public Mount Mount { get; set; }
    public PantsArmor Pants { get; set; }
    public BootsArmor Boots { get; set; }
    public ChestArmor Chest { get; set; }
    public HeadArmor Helmet { get; set; }
    public GlovesArmor Gloves { get; set; }
    public Tool Axe { get; set; }
    public Tool Pickaxe { get; set; }
    public Tool Knife { get; set; }
    public Tool Scythe { get; set; }
    public Robe Robe { get; set; }
    public Cape Cape { get; set; }
    public Ring Ring { get; set; }
    public Ring Ring2 { get; set; }
}
```

**Equipment Slots:**
- `MainHand` (Weapon)
- `OffHand` (Shield, etc.)
- `Mount`
- `Pants`
- `Feet` (Boots)
- `Armor` (Chest)
- `Head` (Helmet)
- `Gloves`
- `Axe` (Tool)
- `Pickaxe` (Tool)
- `Knife` (Tool)
- `Scythe` (Tool)
- `Robe`
- `Cape`
- `Ring`
- `Ring2`

**Slot Assignment:**
- Each slot has a specific item type requirement
- Setting an item to a slot triggers modification events
- Removing an item sets slot to null and triggers removal event

### 3. Corpse Container

Container for loot dropped from dead entities.

**C# Implementation:**
```csharp
public sealed class CorpseContainer : Dictionary<uint, Item>, ItemContainer {
    private uint LastId;
    public uint Id { get; set; }
    
    // Methods for adding/removing loot items
}
```

**Key Features:**
- Temporary container (disappears after loot timeout)
- No weight restrictions
- Items can be looted by players
- Auto-cleanup after timeout

### 4. Storage Container

Container for storage chests and guild storage.

**Features:**
- Persistent storage
- Access control (owner/guild members)
- Weight limits (configurable)
- Multiple storage types (personal chest, guild chest, etc.)

### 5. Vendor Container

Container for NPC vendor items.

**C# Implementation:**
```csharp
public sealed class VendorContainer : Dictionary<uint, Item>, ItemContainer {
    private uint LastId;
    public uint Id { get; set; }
    
    public new void Clear() {
        LastId = 0;
        base.Clear();
    }
    
    // Methods for managing vendor inventory
}
```

**Key Features:**
- Vendor-specific item management
- Item removal by name and amount
- Restocking support
- Price management

### 6. Trade Container

Container for player-to-player trading.

**C# Implementation:**
```csharp
public sealed class SelfTradeContainer : Dictionary<uint, Item>, ItemContainer {
    public uint Id { get; set; }
    public CharacterEntity Owner { get; set; }
    
    // Items reference inventory item IDs
    // Items are not removed from inventory until trade completes
}
```

**Key Features:**
- Temporary container for trade sessions
- Items reference original inventory items
- Trade confirmation required
- Rollback support on trade cancellation

## Container Interface

### C# Interface

```csharp
public interface ItemContainer {
    uint Id { get; }
    bool CanAdd(Item item);
    uint Add(Item item);
    Item TryRemove(uint itemId, uint amount);
    Item TryRemove(uint itemId);
    
    void OnItemHealthChanged(Item item, uint itemId, uint value);
    void OnItemDurabilityChanged(Item item, uint itemId, uint value);
    
    event ItemContainerModifiedHandler Modified;
}
```

**Methods:**
- `CanAdd(Item item)`: Check if item can be added
- `Add(Item item)`: Add item and return item ID
- `TryRemove(uint itemId)`: Remove item by ID
- `TryRemove(uint itemId, uint amount)`: Remove partial stack
- `OnItemHealthChanged()`: Notify health change (for mounts/pets)
- `OnItemDurabilityChanged()`: Notify durability change

**Events:**
- `Modified`: Fired when container is modified

## Item Modification Events

### Event Types

```csharp
public enum ItemModifiedType {
    Added,              // Item added to container
    Removed,           // Item removed from container
    QuantityChanged,   // Stack quantity changed
    HealthChanged,     // Item health changed (mount/pet)
    DurabilityChanged  // Item durability changed
}

public delegate void ItemContainerModifiedHandler(
    uint containerId, 
    Item item, 
    uint itemId, 
    ItemModifiedType type, 
    uint value = 0
);
```

### Event Usage

**Adding Item:**
```csharp
public uint Add(Item item) {
    uint id = AllocateId();
    Add(id, item);
    Weight += item.GetWeight();
    Modified?.Invoke(Id, item, id, ItemModifiedType.Added);
    Owner.WeightOrStrengthChanged();
    return id;
}
```

**Removing Item:**
```csharp
public Item TryRemove(uint itemId) {
    if (base.Remove(itemId, out Item item)) {
        Weight -= item.GetWeight();
        Modified?.Invoke(Id, item, itemId, ItemModifiedType.Removed);
        Owner.WeightOrStrengthChanged();
        return item;
    }
    return null;
}
```

**Quantity Change:**
```csharp
if (containerItem is Stackable containerSt) {
    containerSt.Quantity -= amount;
    Modified?.Invoke(Id, containerItem, itemId, ItemModifiedType.QuantityChanged, containerSt.Quantity);
}
```

## Weight System

### Weight Calculation

**Item Weight:**
- Each item has a `Weight` property
- Stackable items: `TotalWeight = ItemWeight * Quantity`
- Equipment weight affects total inventory weight

**Container Weight:**
```csharp
public uint Weight {
    get {
        uint totalWeight = 0;
        foreach (var item in Values) {
            totalWeight += item.GetWeight();
        }
        return totalWeight;
    }
}
```

### Weight Limits

**Player Strength:**
- Players have a `Strength` stat
- Maximum weight capacity based on strength
- Weight affects mobility when over capacity

**Mobility Effects:**
- **0-100% capacity**: Normal movement speed
- **100-150% capacity**: Progressive speed reduction
- **150-200% capacity**: Severe speed reduction
- **200%+ capacity**: Almost immobile

**Weight Calculation:**
```typescript
// TypeScript example
const weightRatio = currentWeight / maxWeight;
if (weightRatio <= 1.0) {
    speedMultiplier = 1.0;
} else if (weightRatio <= 1.5) {
    speedMultiplier = 1.0 - ((weightRatio - 1.0) * 0.5) * 2; // 1.0 to 0.5
} else if (weightRatio <= 2.0) {
    speedMultiplier = 0.5 - ((weightRatio - 1.5) * 0.5) * 2; // 0.5 to 0.0
} else {
    speedMultiplier = 0.05; // Almost immobile
}
```

### Weight Change Notifications

When container weight changes:
1. Update container `Weight` property
2. Notify owner entity (`Owner.WeightOrStrengthChanged()`)
3. Recalculate player mobility
4. Update client with new weight/mobility

## Item Stacking

### Stackable Items

Items that extend `Stackable` can be stacked together.

**Stacking Logic:**
```csharp
public uint Add(Item item) {
    if (item is Stackable st) {
        // Try to find existing stack
        foreach (var kv in this) {
            Item containerItem = kv.Value;
            if (containerItem.Name == st.Name && containerItem is Stackable containerStackable) {
                // Merge stacks
                containerStackable.Quantity += st.Quantity;
                Weight += item.GetWeight();
                Modified?.Invoke(Id, kv.Value, kv.Key, ItemModifiedType.QuantityChanged, containerStackable.Quantity);
                return kv.Key;
            }
        }
    }
    
    // Create new stack
    uint id = AllocateId();
    Add(id, item);
    Weight += item.GetWeight();
    Modified?.Invoke(Id, item, id, ItemModifiedType.Added);
    return id;
}
```

### Stack Splitting

**Partial Removal:**
```csharp
public Item TryRemove(uint itemId, uint amount) {
    if (TryGetValue(itemId, out Item containerItem)) {
        if (containerItem is Stackable containerSt) {
            if (containerSt.Quantity > amount) {
                // Split stack
                containerSt.Quantity -= amount;
                Weight = Weight + containerSt.GetWeight() - originalWeight;
                Modified?.Invoke(Id, containerItem, itemId, ItemModifiedType.QuantityChanged, containerSt.Quantity);
                return containerItem.Clone(amount);
            } else if (containerSt.Quantity == amount) {
                // Remove entire stack
                base.Remove(itemId);
                Weight -= containerItem.GetWeight();
                Modified?.Invoke(Id, containerItem, itemId, ItemModifiedType.Removed);
                return containerItem;
            }
        }
    }
    return null;
}
```

## Item ID Management

### ID Allocation

**Sequential IDs:**
```csharp
private uint LastId;

private uint AllocateId() {
    return LastId++;
}
```

**ID Reuse:**
- IDs are not reused (prevents conflicts)
- IDs are unique per container
- IDs persist for item lifetime in container

### ID Lookup

**By Item:**
```csharp
public bool TryGetId(Item item, out uint itemId) {
    foreach (var kv in this) {
        if (kv.Value == item) {
            itemId = kv.Key;
            return true;
        }
    }
    itemId = 0;
    return false;
}
```

**By Name:**
```csharp
public bool TryGetId(string itemName, out uint itemId) {
    foreach (var item in this) {
        if (item.Value.Name == itemName) {
            itemId = item.Key;
            return true;
        }
    }
    itemId = 0;
    return false;
}
```

## Container Serialization

### Serialize All Items

```csharp
public object[] SerializeAll() {
    return Values.Select(item => {
        if (item is Weapon weapon)
            return weapon.Serialize();
        else if (item is Shield shield)
            return shield.Serialize();
        else if (item is BootsArmor bootarmor)
            return bootarmor.Serialize();
        // ... other types
        else
            return item.Serialize();
    }).ToArray();
}
```

### Container State

**Serialized Container Data:**
```json
{
    "containerId": 123,
    "containerType": "Inventory",
    "weight": 150,
    "maxWeight": 200,
    "items": [
        {
            "itemId": 1,
            "itemName": "Iron Sword",
            "amount": 1,
            "properties": { /* item properties */ }
        },
        // ... more items
    ]
}
```

## Equipment Slot Management

### Slot Assignment

**Setting Equipment:**
```csharp
public Weapon Weapon {
    get { return m_Weapon; }
    set {
        m_Weapon = value;
        Modified?.Invoke(Id, m_Weapon, (uint)EquipmentItemSlot.MainHand, 
            value == null ? ItemModifiedType.Removed : ItemModifiedType.Added);
    }
}
```

### Slot Validation

**Type Checking:**
- Each slot only accepts specific item types
- Validation occurs before assignment
- Invalid assignments are rejected

**Requirement Checking:**
- Equipment may have stat requirements (Str, Dex, Int)
- Requirements checked before equipping
- Player stats must meet requirements

## Container Queries

### Find Items

**By Name:**
```csharp
public bool TryGetItem(string itemName, out Item item, out uint itemId) {
    foreach (var kv in this) {
        if (kv.Value.Name == itemName) {
            itemId = kv.Key;
            item = kv.Value;
            return true;
        }
    }
    itemId = 0;
    item = null;
    return false;
}
```

**By Type:**
```csharp
public bool TryGetByType(ItemClass itemClass, out Item item, out uint itemId) {
    foreach(var kv in this) {
        if(kv.Value.Class == itemClass) {
            itemId = kv.Key;
            item = kv.Value;
            return true;
        }
    }
    itemId = 0;
    item = null;
    return false;
}
```

**By Tool Type:**
```csharp
public bool TryGetTool(string type, out Item item, out uint itemId) {
    foreach (var kv in this) {
        if (kv.Value is Tool tool) {
            switch (type) {
                case "Pickaxe":
                    if(tool.Name == "Gold Pickaxe" || tool.Name == "Pickaxe") {
                        item = tool;
                        itemId = kv.Key;
                        return true;
                    }
                    break;
                // ... other tool types
            }
        }
    }
    itemId = 0;
    item = null;
    return false;
}
```

### Check Item Existence

**Has Item:**
```csharp
public bool HasItem(string itemName, int amount) {
    foreach(var item in Values) {   
        if(item.Name == itemName && item is Stackable stackable && stackable.Quantity >= amount) {
            return true;
        }
        else if (item.Name == itemName && amount == 1) {
            return true;
        }
    }
    return false;
}
```

## Item Usage

### Consumable Items

**Using Items:**
```csharp
public async void UseItem(CharacterEntity player, uint itemId) {
    Item item;
    if (TryGetValue(itemId, out item)) {
        if (item is Consumable consumable && consumable.Action != null) {
            if (!player.Actions.TryGetValue(consumable.Action, out Action action)) {
                action = Action.CreateInstance(player, consumable.Action);
                player.Actions.Add(consumable.Action, action);
            }
            
            if(action != null && !action.OnCooldown && action.CanExecute && player.CanCast == 0) {
                if(await player.Execute(action, player.Position) && TryRemove(itemId, 1) != null) {
                    // Item consumed
                }
            } 
        }
        else if (item is Blueprint blueprint) {
            // Start blueprint placement
            buffer.Put(ServerPacketType.StartPlacingBlueprint);
            buffer.PutVar(itemId);
            buffer.PutVar(blueprint.Definition.Id);
        }
    }
}
```

## Rust Implementation Considerations

### Performance Optimizations

1. **Structure of Arrays (SoA)**: Store container items in separate arrays for cache efficiency
2. **Zero-Allocation**: Pre-allocate container slots
3. **Lock-Free**: Use atomic operations for concurrent access
4. **Cache-Friendly**: Organize container data for CPU cache efficiency

### Memory Management

- Use `Arc<Item>` for shared item references
- Use `Rc<Item>` for container-local items
- Pool container instances
- Use small containers optimization

### Type Safety

- Use Rust enums for container types
- Use generics for type-safe item access
- Leverage Rust's type system for compile-time safety
- Use `PhantomData` for type-level guarantees

### Container Design

```rust
pub trait ItemContainer {
    fn id(&self) -> ContainerId;
    fn can_add(&self, item: &Item) -> bool;
    fn add(&mut self, item: Item) -> Option<ItemId>;
    fn try_remove(&mut self, item_id: ItemId) -> Option<Item>;
    fn try_remove_amount(&mut self, item_id: ItemId, amount: u32) -> Option<Item>;
    fn weight(&self) -> Weight;
    fn max_weight(&self) -> Weight;
}

pub struct InventoryContainer {
    id: ContainerId,
    owner: EntityId,
    items: HashMap<ItemId, Item>,
    weight: Weight,
    max_weight: Weight,
    last_id: ItemId,
}

pub struct EquipmentContainer {
    id: ContainerId,
    owner: EntityId,
    weapon: Option<Item>,
    offhand: Option<Item>,
    // ... other slots
}
```

### Event System

```rust
pub enum ItemModifiedType {
    Added,
    Removed,
    QuantityChanged(u32),
    HealthChanged(u32),
    DurabilityChanged(u32),
}

pub struct ContainerEvent {
    pub container_id: ContainerId,
    pub item: Item,
    pub item_id: ItemId,
    pub modification_type: ItemModifiedType,
}

// Use channels or observers for event notifications
pub type ContainerEventHandler = Box<dyn Fn(ContainerEvent) + Send + Sync>;
```

## Integration with Other Systems

### Entity System

Containers are owned by entities. See [ENTITIES.md](../core/ENTITIES.md) for entity system documentation.

### Movement System

Container weight affects entity mobility. See [MOVEMENT.md](../core/MOVEMENT.md) for movement system documentation.

### Network System

Container updates are synchronized via network packets. See [BYTEBUFFER.md](../core/BYTEBUFFER.md) for serialization details.

## Performance Considerations

### Container Lookup

- Use hash maps for O(1) item lookup by ID
- Cache frequently accessed items
- Use spatial indexing for large containers
- Pre-compute container statistics

### Weight Calculation

- Cache total weight
- Invalidate cache on item changes
- Use incremental weight updates
- Batch weight recalculations

### Event Performance

- Use event batching for multiple modifications
- Debounce frequent events
- Use async event handlers
- Filter events by relevance

## Testing Strategy

### Unit Tests

- Test item addition/removal
- Test item stacking logic
- Test weight calculation
- Test event firing
- Test slot assignment

### Integration Tests

- Test container persistence
- Test weight-based mobility
- Test equipment slot validation
- Test trade container rollback
- Test vendor container restocking

### Performance Tests

- Benchmark container operations
- Benchmark weight calculation
- Benchmark event system
- Test memory usage with large containers
- Test concurrent access performance

