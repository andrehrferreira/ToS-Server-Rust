# Items Systems Documentation

## Overview

This directory contains comprehensive documentation for all item-related systems in the Tales of Shadowland MMORPG. These systems cover item management, equipment, crafting, enhancements, and the various ways items can be modified and improved.

## Quick Start

**New to items systems?** Start here:
1. [ITEMS.md](./ITEMS.md) - Core items system and hierarchy
2. [EQUIPMENT.md](./EQUIPMENT.md) - Equipment system and types
3. [CRAFTING.md](./CRAFTING.md) - Crafting system and recipes

## Documentation Index

### Core Item Systems

- **[ITEMS.md](./ITEMS.md)** - Core items system
  - Multi-level item hierarchy
  - Base Item class and subclasses
  - Stackable system
  - Item registration (dynamic via API)
  - CDN image loading
  - Rarity system
  - Weight system
  - Item flags and states
  - Item references and namespacing

- **[EQUIPMENT.md](./EQUIPMENT.md)** - Equipment system
  - 9 equipment types (Armor, Boots, Gloves, Helmet, Mainhand, Offhand, Belt, Ring, Necklace)
  - Resource types (Metal/Mineral, Leather, Cloth, Bone)
  - Build segmentation through resource types
  - Visual representation and appearance
  - Attribute bonuses
  - Stat requirements
  - Equipment tiers
  - Visual skins and dye system
  - Tools (gathering equipment)

- **[CONTAINERS.md](./CONTAINERS.md)** - Container system
  - Inventory management
  - Container types
  - Item storage
  - Container capacity
  - Weight limits
  - Container operations

### Item Enhancement Systems

- **[ATTRIBUTES.md](./ATTRIBUTES.md)** - Attribute system
  - Equipment attributes
  - Attribute types and bonuses
  - Attribute generation
  - Prefix and suffix system
  - Attribute tiers
  - Attribute calculations

- **[ENCHANTMENTS.md](./ENCHANTMENTS.md)** - Enchantment system
  - Elemental enchantments (Fire, Cold, Poison, Energy, Light, Dark)
  - Enchantment levels (+1 to +6)
  - Visual effects (VFX particles, color changes)
  - Skill-based success rates
  - Material requirements (essences, star dust)
  - Upgrade system
  - Disenchantment
  - Failure risks and costs

- **[CARDS.md](./CARDS.md)** - Cards system
  - Extremely rare collectible items
  - Card drop rates (less than 1%)
  - Boss cards (even rarer)
  - Powerful attribute bonuses
  - Collectible function (Steam rewards)
  - Card game integration
  - Wagering system
  - Card slot system (shared with runes)

- **[RUNES.md](./RUNES.md)** - Runes system
  - Rune types and effects
  - Rune slot system (shared with cards)
  - Rune insertion and removal
  - Rune destruction on removal
  - Rune rarity and power
  - Rune combinations

- **[MAGIC_STONES.md](./MAGIC_STONES.md)** - Magic stones system
  - Rare droppable items from elite monsters/bosses
  - Guaranteed effects (no RNG)
  - Rarity transformation (Common → Uncommon → Rare → Magic → Legendary)
  - Attribute enhancement
  - Equipment enhancement (card slots, tier increases)
  - Trade currency in auction house
  - Stone types and effects

### Item Modification Systems

- **[DURABILITY.md](./DURABILITY.md)** - Durability system
  - Equipment degradation
  - Durability loss on use
  - Repair mechanics
  - Repair costs
  - Broken equipment effects
  - Durability management

- **[ITEM_NAMING.md](./ITEM_NAMING.md)** - Item naming system
  - Dynamic item names
  - Prefix and suffix generation
  - Rarity-based naming
  - Custom names
  - Naming conventions

### Consumable Systems

- **[CONSUMABLES.md](./CONSUMABLES.md)** - Consumables system
  - Potions (health, mana, stamina, buffs)
  - Food (basic, buff, special)
  - Gift boxes (regular, premium, event)
  - Experience items
  - Special consumables (Power of Gods, transformations, teleportation)
  - Cooldown system
  - Usage effects and restrictions

### Crafting Systems

- **[CRAFTING.md](./CRAFTING.md)** - Crafting system
  - No crafting failures (always succeeds with resources)
  - Skill-based quality and rarity
  - Resource variety (same item, different resources)
  - Progressive XP system
  - Crafting skills (Blacksmithing, Leatherworking, Tailoring, Jewelry, Carpentry, Alchemy)
  - Recipe system
  - Gold cost requirements
  - Specialized tools
  - Destruction and resource recovery
  - Component tracking

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Items Systems                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Core Systems          Enhancement      Modification       │
│  ───────────          ────────────      ───────────        │
│  • Items              • Attributes      • Durability       │
│  • Equipment          • Enchantments    • Naming           │
│  • Containers         • Cards                              │
│                       • Runes                              │
│                       • Magic Stones                        │
│                                                             │
│  Consumable Systems   Crafting Systems                     │
│  ──────────────────   ──────────────────                   │
│  • Consumables        • Crafting                           │
│    (Potions, Food,                                          │
│     Gift Boxes, etc.)                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Item Hierarchy

```
Item (Abstract)
├── Stackable (Abstract)
│   ├── Resource (Abstract)
│   │   └── Consumable (Abstract)
│   └── [Stackable Items]
├── Equipament (Abstract)
│   ├── Accessory (Abstract)
│   │   ├── Ring (Abstract)
│   │   └── Necklace (Abstract)
│   ├── Offhand (Abstract)
│   ├── Weapon (Abstract)
│   ├── Tool (Abstract)
│   │   ├── PickaxeTool (Abstract)
│   │   ├── AxeTool (Abstract)
│   │   └── ScytheTool (Abstract)
│   ├── PetItem (Abstract)
│   └── MountItem (Abstract)
└── Card (Abstract)
```

## Key Features

### Item Management

**Dynamic Registration:**
- Items can be created/modified via API/dashboard
- No client recompilation required
- Real-time item updates

**CDN Integration:**
- Item images loaded from CDN
- Local caching for performance
- On-demand loading

**Rarity System:**
- Common, Uncommon, Rare, Magic, Legendary
- Rarity affects attributes and value
- Rarity-based visual effects

### Equipment System

**Resource Types:**
- Metal/Mineral: Heavy, strength-focused
- Leather: Balanced, mobile
- Cloth: Light, magical
- Bone: Organic accessories

**Build Segmentation:**
- Resource types create natural build segmentation
- No hard class restrictions
- Hybrid builds supported

**Visual Customization:**
- Equipment changes appearance
- Visual skins (cosmetic only)
- Dye system for customization

### Enhancement Systems

**Attributes:**
- Extensive attribute system
- Prefix and suffix generation
- Attribute tiers and bonuses

**Enchantments:**
- Elemental properties
- Visual effects (VFX particles)
- Progressive levels (+1 to +6)
- Skill-based success

**Cards:**
- Extremely rare collectibles
- Powerful bonuses
- Collectible function
- Card game integration

**Runes:**
- Rune slot system
- Various rune types
- Rune combinations
- Destruction on removal

**Magic Stones:**
- Guaranteed effects (no RNG)
- Rarity transformation
- Attribute enhancement
- Trade currency

### Crafting System

**Key Principles:**
- Never fails (always succeeds with resources)
- Skill determines quality and rarity
- Resource variety affects outcome
- Progressive XP system

**Crafting Skills:**
- Blacksmithing (Metal/Mineral)
- Leatherworking (Leather)
- Tailoring (Cloth)
- Jewelry (Accessories)
- Carpentry (Wooden items)
- Alchemy (Potions)

## System Integration

### Items → Equipment
- Equipment items inherit from base Item class
- Equipment provides attribute bonuses
- Equipment has durability and requirements

### Equipment → Attributes
- Equipment can have multiple attributes
- Attributes generated based on rarity
- Prefix/suffix system for naming

### Equipment → Enchantments
- Equipment can be enchanted
- Enchantments add elemental properties
- Visual effects on enchanted items

### Equipment → Cards/Runes
- Equipment has card/rune slots
- Cards and runes provide bonuses
- Shared slot system (cannot use 2 cards or 2 runes)

### Equipment → Magic Stones
- Magic stones can transform equipment rarity
- Magic stones can enhance attributes
- Magic stones can add card slots

### Items → Crafting
- All items have recipes
- Crafting creates items from resources
- Destruction recovers partial resources

### Equipment → Durability
- Equipment degrades with use
- Durability affects effectiveness
- Repair system restores durability

### Items → Consumables
- Consumables extend Stackable items
- Potions restore resources (health, mana, stamina)
- Food provides buffs and regeneration
- Gift boxes contain random rewards
- Experience items grant XP

## Item Progression Flow

```
Common Item
    ↓ (Ascension Shard)
Uncommon Item (+1 attribute)
    ↓ (Essence Fragment)
Uncommon Item (+2 attributes)
    ↓ (Transmutation Crystal)
Rare Item (+3 attributes)
    ↓ (Enchantment)
Rare Item +1 Enchantment (+3 attributes, elemental bonuses)
    ↓ (Card/Rune)
Rare Item +1 Enchantment + Card (+3 attributes, elemental bonuses, card bonuses)
    ↓ (Magic Stone upgrades)
Legendary Item +6 Enchantment + Card (+max attributes, max elemental bonuses, card bonuses)
```

## Common Use Cases

### Obtaining Items

1. **Drops from Creatures**
   - Equipment drops from monsters
   - Cards drop from all creatures (<1% chance)
   - Magic stones drop from elite monsters/bosses

2. **Crafting**
   - Create equipment using crafting skills
   - Use different resource types/qualities
   - Skill level affects rarity chances

3. **Trading**
   - Trade with other players
   - Use auction house
   - Buy from NPCs

### Enhancing Items

1. **Rarity Transformation**
   - Use Magic Stones to upgrade rarity
   - Common → Uncommon → Rare → Magic → Legendary
   - Each upgrade adds attributes

2. **Enchantment**
   - Apply elemental enchantments
   - Upgrade enchantment levels (+1 to +6)
   - Visual effects and bonuses

3. **Cards/Runes**
   - Insert cards or runes into equipment slots
   - Gain powerful bonuses
   - Cards preserved on removal, runes destroyed

4. **Attribute Enhancement**
   - Use Magic Stones to add attributes
   - Randomize prefixes/suffixes
   - Increase attribute tiers

### Crafting Items

1. **Gather Resources**
   - Collect materials through gathering
   - Use different resource qualities
   - Higher quality = better results

2. **Craft Equipment**
   - Use appropriate crafting skill
   - Follow recipe requirements
   - Pay gold cost

3. **Optimize Results**
   - Use higher skill levels
   - Use specialized tools
   - Use higher quality resources

## Related Documentation

### Server Systems
- [Server Architecture](../server/ARCHITECTURE.md) - Server architecture
- [Economy Systems](../server/ECONOMY.md) - Advanced economy systems
- [Auction House](../server/AUCTION_HOUSE.md) - Auction house system

### Player Systems
- [Player Systems](../player/) - Player progression
- [Skills](../player/SKILLS.md) - Skills system
- [Gathering](../player/GATHERING.md) - Gathering system

### Core Systems
- [Core Systems](../core/) - Core game systems
- [Experience](../core/EXPERIENCE.md) - Experience system

## Implementation Notes

### Data Persistence

All item data is persisted:
- Item properties and attributes
- Equipment durability
- Enchantment levels
- Card/rune slots
- Crafting recipes
- Item ownership

### Server Authority

The server is authoritative for:
- Item creation and modification
- Attribute generation
- Enchantment success/failure
- Card/rune insertion validation
- Crafting result determination
- Durability calculations

### Performance Considerations

- Efficient item lookup by reference
- Cached attribute calculations
- Optimized crafting recipes
- Efficient container operations
- Batch item operations

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Player README](../player/README.md) - Player systems documentation

