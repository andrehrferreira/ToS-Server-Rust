# Interaction System Documentation

## Overview

The interaction system handles all player interactions with game objects, NPCs, containers, and resources. The system uses a strategy pattern with different interaction handlers for each interaction type. Players can interact with stashes, loot containers, gathering resources, crafting stations, and vendors.

## Key Features

- **Multiple Interaction Types**: Stash, Loot, Gathering, Crafting, Vendor
- **Strategy Pattern**: Extensible interaction system with separate handlers
- **Container System**: Unified container system for storage and loot
- **Pathfinding Integration**: Automatic pathfinding when interaction target is out of range
- **Tooltip System**: Detailed tooltips for special items
- **Skinning System**: Skinning dead entities for resources
- **Entity Management**: Proper entity cleanup after interaction

## Interaction Types

### Interaction Type Enum

```typescript
enum InteractType {
    Stash,      // Personal storage containers
    Loot,       // Loot containers from dead entities
    Gathering,  // Resource gathering spots
    Crafting,   // Crafting stations
    Vendor      // NPC vendors
}
```

## Interaction Architecture

### Base Interaction Class

**Abstract Base Class:**
```typescript
abstract class Interact {
    public static interacts: Map<InteractType, Interact> = new Map();
    
    public static interact(player: Player, type: InteractType, payload: string) {
        if (Interact.interacts.has(type)) {
            Interact.interacts.get(type).onInteract(player, payload);
        }
    }
    
    public abstract onInteract(player: Player, payload: string);
}
```

**Registration:**
- Each interaction type is registered at system initialization
- Handlers are singleton instances
- Extensible for new interaction types

## Stash Interaction

### Overview

Stash interaction allows players to access their personal storage containers. Stashes are typically placed in player houses or safe zones.

### Stash Container

**Container ID Format:**
- Format: `${player.characterId}::${payload}` (truncated to 12 characters)
- Unique per player and stash location
- Persists across sessions

**Interaction Process:**
1. Player interacts with stash object
2. System generates container ID
3. Container opened on client
4. All items in container sent to client
5. Tooltips sent for special items

**Implementation:**
```typescript
class StashInteract extends Interact {
    public onInteract(player: Player, payload: string) {
        // Generate container ID
        const containerId = `${player.characterId}::${payload}`.substring(0, 12);
        
        // Open container on client
        packetOpenContainer.send(player, {
            containerId,
            type: ContainerType.Stash,
            entityId: player.mapIndex
        });
        
        // Get container from system
        const container = Containers.get(containerId);
        
        if (container) {
            // Send all items to client
            container.slots?.forEach((item, slotId) => {
                packetAddItemContainer.send(player, {
                    amount: item.Amount,
                    containerId: container.containerId,
                    itemName: item.Namespace,
                    itemRef: item.Ref,
                    slotId,
                    itemRarity: item.Rarity,
                    goldCost: item.GoldCost,
                    weight: item.Weight
                });
                
                // Send tooltip for special items
                if (
                    item instanceof Equipament || 
                    item instanceof PowerScroll || 
                    item instanceof PetItem || 
                    item instanceof MountItem
                ) {
                    packetTooltip.send(player, item.Ref, item.serialize());
                }
            });
        }
    }
}
```

### Stash Features

**Storage:**
- Personal storage per stash location
- Item persistence
- Weight management
- Stackable items support

**Access:**
- Only owner can access stash
- Stash can be placed in houses
- Multiple stashes per player (different locations)

## Loot Interaction

### Overview

Loot interaction allows players to loot items from dead entities. The system handles loot containers, skinning, and entity cleanup.

### Loot Container

**Interaction Process:**
1. Player interacts with dead entity
2. System checks if entity has loot
3. If loot exists: Open loot container
4. If skinning available: Start skinning process
5. If no loot/skinning: Dissolve entity

**Loot Container:**
```typescript
class LootInteract extends Interact {
    public onInteract(player: Player, payload: string) {
        const entity = Maps.getEntity(player.socket, payload);
        
        if (entity) {
            // Check for loot container
            if (entity.loot && entity.loot.count() > 0) {
                // Open loot container
                packetOpenContainer.send(player, {
                    containerId: entity.loot.containerId,
                    type: ContainerType.Loot,
                    entityId: entity.mapIndex
                });
                
                // Send items after short delay (for UI animation)
                setTimeout(() => {
                    entity.loot.slots?.forEach((item, slotId) => {
                        packetAddItemContainer.send(player, {
                            amount: item.Amount,
                            containerId: entity.loot.containerId,
                            itemName: item.Namespace,
                            itemRef: item.Ref,
                            slotId,
                            itemRarity: item.Rarity,
                            goldCost: item.GoldCost,
                            weight: item.Weight
                        });
                        
                        // Send tooltip for special items
                        if (
                            item instanceof Equipament || 
                            item instanceof PowerScroll || 
                            item instanceof PetItem || 
                            item instanceof MountItem
                        ) {
                            packetTooltip.send(player, item.Ref, item.serialize());
                        }
                    });
                }, 300); // 300ms delay for UI
            }
            // Check for skinning
            else if (entity.skinnerTick > 0) {
                this.startSkinning(player, entity);
            }
            // No loot or skinning, dissolve entity
            else {
                this.dissolveEntity(entity, player);
            }
        }
    }
}
```

### Skinning System

**Skinning Process:**
1. Entity has `skinnerTick > 0` (skinning available)
2. Start skinning animation
3. Play montage animation for nearby entities
4. Player can collect resources from skinning

**Implementation:**
```typescript
private startSkinning(player: Player, entity: Entity): void {
    // Send skinning start packet
    packetStartSkinning.send(player);
    
    // Play animation for nearby entities
    player.areaOfInterece.forEach((nearbyEntity) => {
        packetPlayMontageEntity.send(player, nearbyEntity, 12); // Animation ID 12
    });
}
```

### Entity Dissolution

**Dissolution Process:**
1. Entity has no loot and no skinning
2. Dissolve entity for interacting player
3. Dissolve entity for all nearby entities
4. Destroy entity from map

**Implementation:**
```typescript
private dissolveEntity(entity: Entity, player: Player): void {
    // Dissolve for interacting player
    packetDissolveEntity.send(entity, player);
    
    // Dissolve for all nearby entities
    entity.areaOfInterece.forEach((otherEntity) => {
        packetDissolveEntity.send(entity, otherEntity);
    });
    
    // Remove entity from map
    entity.destroy();
}
```

## Gathering Interaction

### Overview

Gathering interaction allows players to collect resources from gathering spots (trees, stones, bushes, etc.).

### Gathering Process

**Interaction Flow:**
1. Player interacts with gathering spot
2. System validates gathering spot
3. Start gathering process
4. Resource collection begins

**Implementation:**
```typescript
class GatheringInteract extends Interact {
    public onInteract(player: Player, payload: string) {
        // Payload contains gathering spot ID
        player.startGathering(payload);
    }
}
```

**Gathering Details:**
- See [GATHERING.md](./GATHERING.md) for complete gathering system documentation
- Supports multiple gathering types (mining, lumberjack, herbalism)
- Tick-based collection system
- Tool bonuses and skill requirements

## Crafting Interaction

### Overview

Crafting interaction allows players to access crafting stations and view available recipes.

### Crafting Process

**Interaction Flow:**
1. Player interacts with crafting station
2. System extracts skill name from payload
3. Validates skill and recipes exist
4. Sends crafting list to client

**Implementation:**
```typescript
class CraftingInteract extends Interact {
    public onInteract(player: Player, payload: string) {
        // Extract skill name from payload (formatted string)
        const skill = getSkillNameFromFormattedString(payload);
        
        // Validate skill and recipes exist
        if (skill && CraftRecipe.RecipesList[skill]) {
            // Send crafting list to client
            packetCraftingList.send(player, skill);
        }
    }
}
```

### Crafting Stations

**Station Types:**
- Blacksmithing stations
- Alchemy stations
- Carpentry stations
- Tailoring stations
- Cooking stations
- And more...

**Recipe System:**
- Recipes organized by skill
- Skill level requirements
- Material requirements
- Success chance based on skill

## Vendor Interaction

### Overview

Vendor interaction allows players to trade with NPC vendors. Vendors sell items and may buy items from players.

### Vendor Process

**Interaction Flow:**
1. Player interacts with vendor NPC
2. System looks up vendor by payload (vendor ID)
3. Sets vendor list on player
4. Sends vendor data to client

**Implementation:**
```typescript
class VendorInteract extends Interact {
    public onInteract(player: Player, payload: string) {
        // Payload contains vendor ID
        if (BaseVendor.VendorList[payload]) {
            // Set vendor list on player
            player.vendorList = BaseVendor.VendorList[payload];
            
            // Send vendor interaction packet
            packetVendorInteract.send(player, BaseVendor.VendorList[payload]);
        }
    }
}
```

### Vendor Features

**Vendor Types:**
- General vendors (basic items)
- Specialized vendors (class-specific items)
- Guild vendors (guild-exclusive items)
- Event vendors (limited-time items)

**Vendor Functions:**
- Buy items from vendor
- Sell items to vendor
- Repair equipment (if vendor supports)
- Special services (if vendor supports)

## Pathfinding Integration

### Auto-Pathfinding

**When Interaction Target is Out of Range:**
- System checks interaction range
- If out of range but within pathfinding range:
  - Start automatic pathfinding
  - Player moves to target location
  - Interaction executes when in range

**See [PLAYER_CONTROLLER.md](../client/PLAYER_CONTROLLER.md) for pathfinding details.**

## Container System

### Container Types

**Container Type Enum:**
```typescript
enum ContainerType {
    Stash,    // Personal storage
    Loot,     // Loot containers
    Inventory, // Player inventory
    GuildChest, // Guild shared storage
    Vendor    // Vendor inventory
}
```

### Container Operations

**Item Management:**
- Add items to container
- Remove items from container
- Move items between containers
- Stack stackable items
- Weight management

**Container Data:**
```typescript
interface ContainerItem {
    Amount: number;
    Namespace: string;
    Ref: string;
    Rarity: ItemRarity;
    GoldCost: number;
    Weight: number;
}
```

## Tooltip System

### Tooltip Types

**Items with Tooltips:**
- Equipment (weapons, armor)
- Power Scrolls (skill scrolls)
- Pet Items
- Mount Items

**Tooltip Data:**
- Item properties
- Stats and bonuses
- Requirements
- Description
- Durability (if applicable)

**Tooltip Packet:**
```typescript
packetTooltip.send(player, itemRef, serializedData);
```

## Network Packets

### Interaction Packets

**Open Container Packet:**
```typescript
interface OpenContainerPacket {
    containerId: string;
    type: ContainerType;
    entityId: number;
}
```

**Add Item Container Packet:**
```typescript
interface AddItemContainerPacket {
    amount: number;
    containerId: string;
    itemName: string;
    itemRef: string;
    slotId: number;
    itemRarity: ItemRarity;
    goldCost: number;
    weight: number;
}
```

**Tooltip Packet:**
```typescript
interface TooltipPacket {
    itemRef: string;
    tooltipData: string; // Serialized item data
}
```

**Start Skinning Packet:**
```typescript
interface StartSkinningPacket {
    // No additional data, entity ID from context
}
```

**Crafting List Packet:**
```typescript
interface CraftingListPacket {
    skill: SkillName;
    recipes: Recipe[];
}
```

**Vendor Interact Packet:**
```typescript
interface VendorInteractPacket {
    vendorId: string;
    vendorData: VendorData;
}
```

## Interaction Flow

### Complete Interaction Flow

**1. Player Initiates Interaction:**
- Player clicks on interactive object
- Client sends interaction request
- Server receives interaction request

**2. Range Check:**
- Server checks if player is in interaction range
- If out of range: Start pathfinding
- If in range: Proceed with interaction

**3. Interaction Processing:**
- System determines interaction type
- Calls appropriate interaction handler
- Handler processes interaction

**4. Client Update:**
- Server sends interaction response
- Client opens appropriate UI
- Client displays interaction data

**5. Interaction Completion:**
- Player completes interaction (takes item, crafts item, etc.)
- Server validates action
- Server updates game state
- Client receives update

## Special Item Handling

### Items with Special Tooltips

**Equipment:**
- Weapons: Damage, durability, requirements
- Armor: Defense, durability, requirements
- Accessories: Bonuses, requirements

**Power Scrolls:**
- Skill name
- Skill level
- Experience gain
- Requirements

**Pet Items:**
- Pet type
- Pet stats
- Pet abilities
- Pet requirements

**Mount Items:**
- Mount type
- Mount speed
- Mount abilities
- Mount requirements

## Entity Management

### Entity Lifecycle

**Entity States:**
1. **Alive**: Entity is active
2. **Dead**: Entity is dead, loot available
3. **Skinnable**: Entity can be skinned
4. **Dissolved**: Entity removed from world

**Entity Cleanup:**
- Entities dissolved when no longer needed
- Loot containers cleaned up after looting
- Skinning completed after skinning process
- Proper cleanup prevents memory leaks

## Performance Considerations

### Optimization Strategies

**1. Container Caching:**
- Cache container data
- Reduce database queries
- Faster container access

**2. Lazy Loading:**
- Load container items on demand
- Don't load all items immediately
- Reduce initial load time

**3. Batch Updates:**
- Batch item updates
- Reduce network packets
- Improve performance

**4. Entity Pooling:**
- Reuse entity objects
- Reduce memory allocation
- Improve performance

## Integration with Other Systems

### Inventory System
- Items moved between containers and inventory
- Weight management
- Stackable item handling

### Gathering System
- Gathering interactions trigger gathering process
- Resource collection
- Skill experience gain

### Crafting System
- Crafting interactions open crafting UI
- Recipe display
- Crafting process

### Vendor System
- Vendor interactions open vendor UI
- Buy/sell operations
- Price calculations

### Pathfinding System
- Auto-pathfinding when out of range
- Pathfinding to interaction targets
- Range validation

## Error Handling

### Validation

**Interaction Validation:**
- Check if player exists
- Check if entity exists
- Check if interaction type is valid
- Check if player has permission
- Check if interaction is available

**Error Responses:**
- Invalid interaction: Error message to player
- Out of range: Start pathfinding
- No permission: Permission denied message
- Container full: Cannot add item message

## Testing Considerations

### Unit Tests
- Interaction type routing
- Container operations
- Tooltip generation
- Entity dissolution

### Integration Tests
- Full interaction flow
- Pathfinding integration
- Container synchronization
- Multi-player interactions

### Performance Tests
- Container loading performance
- Large container handling
- Multiple simultaneous interactions
- Entity cleanup performance

## Summary

The interaction system provides:

- **Multiple Interaction Types**: Stash, Loot, Gathering, Crafting, Vendor
- **Extensible Architecture**: Strategy pattern for easy extension
- **Container System**: Unified container management
- **Pathfinding Integration**: Auto-pathfinding when out of range
- **Tooltip System**: Detailed information for special items
- **Entity Management**: Proper cleanup and lifecycle management
- **Performance Optimized**: Caching and lazy loading

The system ensures smooth player interactions with all game objects while maintaining performance and proper resource management.

