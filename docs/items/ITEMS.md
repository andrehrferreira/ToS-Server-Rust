# Items System Documentation

## Overview

The Items system is a multi-level hierarchical system that manages all game items, including equipment, consumables, resources, quest items, and more. The system supports dynamic item registration via API/dashboard, allowing items to be created and modified without client recompilation. Item images are stored in a CDN and loaded on-demand with local caching.

## Key Features

- **Multi-level Hierarchy**: Base `Item` class with specialized subclasses for different item types
- **Stackable System**: Support for stackable items (potions, coins, resources)
- **Equipment System**: Complex equipment with attributes, durability, resistances, and card slots
- **Dynamic Registration**: Items can be registered via API/dashboard without client recompilation
- **CDN Image Loading**: Item images loaded from CDN with local caching
- **Rarity System**: Items have rarity levels that affect attributes and value
- **Attribute System**: Extensive attribute system for equipment bonuses
- **Durability System**: Equipment degrades over time with usage
- **Card System**: Equipment can have card slots for additional bonuses
- **Weight System**: Items have weight that affects inventory capacity

## Item Hierarchy

### Base Classes

```
Item (Abstract)
├── Stackable (Abstract)
│   ├── Resource (Abstract)
│   │   └── Consumable (Abstract)
│   └── [Stackable Items]
├── Equipament (Abstract)
│   ├── Accessory (Abstract)
│   │   ├── Ring (Abstract)
│   │   └── Necklance (Abstract)
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

### TypeScript Implementation

**Base Item Class:**
```typescript
export abstract class Item {
    public abstract Namespace: string;
    public abstract Name: string;
    public ContainerId: string;
    public Amount: number = 1;
    public Rarity: ItemRarity = ItemRarity.Common;
    public Ref: string;
    public Weight: number = 0.1;
    public Hue: number;
    public Flags: StateFlags = new StateFlags(ItemStates.None);
    public GoldCost: number = 0;
    public CraftBy: string;
    public CraftingInfo: Map<string, string> = new Map<string, string>();
}
```

**Stackable Class:**
```typescript
export abstract class Stackable extends Item {
    public setAmount(amount: number) {
        if(amount > 0)
            this.Amount = amount;
    }
}
```

**Equipament Class:**
```typescript
export abstract class Equipament extends Item {
    public EquipamentType: EquipamentType = EquipamentType.None;
    public EquipmentWeight: EquipmentWeight = EquipmentWeight.Light;
    public MaxDurability: number = 25;
    public Durability: number = 25;
    public Armor: number = 0;
    public MaxAttrs: number = 0;
    public Tier: EquipamentTier = EquipamentTier.T1;
    
    // Stat Requirements
    public RequiredStrength: number = 0;
    public RequiredDexterity: number = 0;
    public RequiredIntelligence: number = 0;
    
    // Material Type (affects prefix/suffix pools)
    public MaterialType: MaterialType = MaterialType.None;
    
    // Resistances
    public FireResistence: number = 0;
    public ColdResistence: number = 0;
    public PoisonResistence: number = 0;
    public EnergyResistence: number = 0;
    public LightResistence: number = 0;
    public DarkResistence: number = 0;
    
    // Prefix/Suffix System
    public Prefixes: Array<ItemModifier> = new Array<ItemModifier>();
    public Suffixes: Array<ItemModifier> = new Array<ItemModifier>();
    
    // Legacy attribute system (for backward compatibility)
    public Attrs: Map<AttributeType, number> = new Map<AttributeType, number>();
    
    // Card System
    public Cards: Array<string> = new Array<string>();
    public MaxSlots: number = 3;
    public CardSlots: number = 0;
}

interface ItemModifier {
    id: string;
    name: string;
    attributes: Map<AttributeType, number>;
    isNegative: boolean;
}

enum MaterialType {
    None,
    // Physical Materials
    Iron, Steel, Plate, Obsidian,
    // Elemental Materials
    Mithril, Adamantite, Crystal, Arcane,
    // Hybrid Materials
    Leather, Chain, Scale, Enchanted
}
```

### C# Implementation

The C# version follows a similar structure but uses partial classes for organization:

```csharp
public partial class Item {
    public Item Prefab;
    public virtual uint Visual { get; }
    public virtual byte Color { get; }
    public virtual object Serialize();
    public virtual void Deserialize(IDictionary<string, object> model);
}

public partial class Equipment : Item {
    public uint MaxDurability;
    public uint Durability;
    public ItemQuality Quality;
    // ... attributes and resistances
}

public partial class Stackable : Item {
    public uint Quantity;
}
```

## Item Types

### 1. Stackable Items

Items that can be stacked together in inventory slots.

**Types:**
- **Resource**: Base materials (ores, wood, skins, etc.)
- **Consumable**: Items that can be used (potions, food, etc.)

**Key Features:**
- `Amount` property for stack size
- Automatic stacking when adding to inventory
- Stack splitting support

### 2. Equipment

Items that can be equipped on character slots.

**Equipment Types:**
- `Helmet` - Head armor
- `Chest` - Chest armor
- `Pants` - Leg armor
- `Boots` - Foot armor
- `Gloves` - Hand armor
- `Ring` - Finger accessory
- `Necklance` - Neck accessory
- `Robe` - Full body clothing
- `Cloak` - Back accessory
- `Weapon` - Main hand weapon
- `Offhand` - Off-hand item (shield, etc.)
- `Pet` - Pet summoning item
- `Mount` - Mount summoning item
- `Instrument` - Musical instrument
- `PickaxeTool` - Mining tool
- `AxeTool` - Woodcutting tool
- `ScytheTool` - Harvesting tool

**Equipment Weight:**
- `Light` - Light armor
- `Medium` - Medium armor
- `Heavy` - Heavy armor

### 3. Weapons

Specialized equipment for combat.

**Weapon Types:**
- `Sword` - One-handed sword
- `Axe` - One-handed axe
- `Staff` - Magic staff
- `Dagger` - Dagger
- `Hammer` - One-handed hammer
- `Bow` - Bow
- `Crossbow` - Crossbow
- `TwoHandedAxe` - Two-handed axe
- `TwoHandedSword` - Two-handed sword
- `TwoHandedHammer` - Two-handed hammer
- `Spear` - Spear
- `Pickaxe` - Pickaxe weapon

**Weapon Properties:**
- `Damage` - Base damage (Dice system)
- `BonusDamage` - Additional damage bonus
- `AttackSpeed` - Attack speed modifier
- `MaxAttrs` - Maximum random attributes

### 4. Tools

Items used for gathering resources.

**Tool Types:**
- `PickaxeTool` - Mining
- `AxeTool` - Woodcutting
- `ScytheTool` - Harvesting

### 5. Cards

Items that can be inserted into equipment card slots.

**Card Properties:**
- `Attack` - Attack bonus
- `HP` - Health bonus
- `Energy` - Energy bonus
- `Armor` - Armor bonus
- Resistances (Fire, Cold, Poison, Energy, Light, Dark)
- `Attrs` - Attribute bonuses

## Item Rarity System

### Rarity Levels

```typescript
export enum ItemRarity {
    Common,      // Basic items
    Uncommon,    // Slightly better
    Rare,        // Good items
    Magic,       // Magical items
    Legendary,   // Very rare
    Unique,      // Unique items
    Quest        // Quest items
}
```

### Rarity Effects

**On Equipment Generation:**
- **Uncommon**: +20 durability, +1 min attribute
- **Rare**: +50 durability, min 2 attributes
- **Magic**: +100 durability, min 3 attributes
- **Legendary**: +100 durability, min 4 attributes
- **Unique**: +300 durability, min 4 attributes

**On Gold Cost:**
- Uncommon: +30% cost
- Rare: +50% cost
- Magic: +80% cost
- Legendary: +100% cost

**On Weapons:**
- Each rarity level upgrades damage dice and adds bonus damage

## Item Tier System

### Tier Levels

```typescript
export enum EquipamentTier {
    T0, T1, T2, T3, T4, T5, T6, T7, T8
}
```

### Tier Effects

**Tier to Level Mapping:**
- **T0**: Level 1-10 (Early Game)
- **T1**: Level 11-20 (Early Game)
- **T2**: Level 21-30 (Early-Mid Game)
- **T3**: Level 31-40 (Mid Game)
- **T4**: Level 41-50 (Mid Game)
- **T5**: Level 51-60 (Mid-Late Game)
- **T6**: Level 61-70 (Late Game)
- **T7**: Level 71-80 (Late Game)
- **T8**: Level 81-100 (End Game)

**Attribute Value Range:**
- T0: 1-5
- T1: 1-10
- T2: 6-15
- T3: 11-20
- T4: 16-25
- T5: 21-30
- T6: 26-35
- T7: 31-40
- T8: 36-50

**Durability Bonus:**
- T2: +10 durability
- T3: +25 durability
- T4: +50 durability
- T5: +75 durability
- T6: +100 durability
- T7: +150 durability
- T8: +200 durability

**Stat Requirements:**
- T0: 0-5 Str/Dex/Int (Level 1-10)
- T1: 6-12 Str/Dex/Int (Level 11-20)
- T2: 13-20 Str/Dex/Int (Level 21-30)
- T3: 21-30 Str/Dex/Int (Level 31-40)
- T4: 31-42 Str/Dex/Int (Level 41-50)
- T5: 43-55 Str/Dex/Int (Level 51-60)
- T6: 56-68 Str/Dex/Int (Level 61-70)
- T7: 69-80 Str/Dex/Int (Level 71-80)
- T8: 81-100 Str/Dex/Int (Level 81-100)

**Prefix/Suffix Limits:**
- T0-T1: Max 1 prefix, max 1 suffix
- T2-T3: Max 2 prefixes, max 2 suffixes
- T4-T5: Max 3 prefixes, max 3 suffixes
- T6-T7: Max 4 prefixes, max 4 suffixes
- T8: Max 5 prefixes, max 5 suffixes

**Card Slots:**
- T3+: Chance to generate card slots (10% chance, 1-3 slots)
- T6+: Increased chance (15% chance, 1-4 slots)
- T8: Maximum chance (20% chance, 1-5 slots)
- Card slots replace random attributes

## Attribute System

### Attribute Types

```typescript
export enum AttributeType {
    // Regeneration
    HealthRegen,
    ManaRegen,
    StaminaRegen,
    
    // Combat
    BonusDamage,
    BonusMagicDamage,
    BonusSpeed,
    CriticalChance,
    CriticalDamage,
    DamageReduction,
    DodgeChance,
    
    // Stats
    BonusStr,
    BonusInt,
    BonusDex,
    BonusVig,
    BonusAgi,
    BonusLuc,
    
    // Resources
    BonusLife,
    BonusStam,
    BonusMana,
    
    // Magic
    SpellDamage,
    CastSpeed,
    LowerManaCost,
    FasterCasting,
    CooldownReduction,
    
    // Defense
    ReflectPhysical,
    ReflectSpell,
    
    // Utility
    Luck,
    IncreasedKarmaLoss,
    
    // Elemental Damage
    FireDamage,
    ColdDamage,
    EnergyDamage,
    PoisonDamage,
    LightDamage,
    DarkDamage,
    
    // Gathering
    BonusCollectsMineral,
    BonusCollectsSkins,
    BonusCollectsWood,
}
```

### Attribute Generation

**Note**: With the prefix/suffix system, attributes are now generated through modifiers rather than direct assignment. See the [Prefix and Suffix System](#prefix-and-suffix-system-path-of-exile-2-style) section for details.

**Legacy Attribute Pool (for items without prefixes/suffixes):**
- **Equipment Attributes**: Dex, Int, Life, Mana, Speed, Stam, Str, CastSpeed, CooldownReduction, CriticalChance, DamageReduction, DodgeChance, HealthRegen, ManaRegen, StaminaRegen, LowerManaCost, IncreasedKarmaLoss, Luck, ReflectPhysical, ReflectSpell, Agi, Vig, Luc

- **Weapon Attributes**: Includes all equipment attributes plus: BonusDamage, BonusMagicDamage, CriticalDamage, FasterCasting, SpellDamage, and all elemental damage types

- **Accessory Attributes**: Limited pool: HealthRegen, ManaRegen, StaminaRegen, Dex, Int, Life, Agi, Vig, Luc

## Prefix and Suffix System (Path of Exile 2 Style)

### Overview

The prefix/suffix system allows items to have both positive and negative modifiers, creating a deep crafting system similar to Path of Exile 2. This system enables:
- **Complex Crafting**: Players can craft items with specific attribute combinations
- **RMT Economy**: Rare combinations create valuable items for trading
- **Build Diversity**: Negative modifiers can enable unique builds
- **Material-Based Attributes**: Different materials favor different attribute types

### Prefix and Suffix Structure

**Item Name Format:**
```
[Prefix] [Base Item Name] [Suffix]
Example: "Cursed Iron Sword of Swiftness"
         Prefix: "Cursed" (negative modifier)
         Base: "Iron Sword"
         Suffix: "of Swiftness" (positive modifier)
```

**Modifier Types:**
- **Prefix**: Appears before item name (can be positive or negative)
- **Suffix**: Appears after item name (can be positive or negative)
- Items can have 0-3 prefixes and 0-3 suffixes depending on rarity and tier

### Prefix/Suffix Generation Rules

**Rarity-Based Modifier Count:**
- **Common**: 0 prefixes, 0 suffixes (base item only)
- **Uncommon**: 1 prefix OR 1 suffix
- **Rare**: 1 prefix AND 1 suffix
- **Magic**: 2 prefixes OR 2 suffixes OR 1 prefix + 1 suffix
- **Legendary**: 2 prefixes AND 2 suffixes
- **Unique**: Fixed prefixes/suffixes (defined per unique item)

**Tier-Based Modifier Limits:**
- **T0**: Max 1 prefix, max 1 suffix
- **T1**: Max 1 prefix, max 1 suffix
- **T2**: Max 2 prefixes, max 2 suffixes
- **T3**: Max 2 prefixes, max 2 suffixes
- **T4**: Max 3 prefixes, max 3 suffixes
- **T5**: Max 3 prefixes, max 3 suffixes
- **T6**: Max 4 prefixes, max 4 suffixes
- **T7**: Max 4 prefixes, max 4 suffixes
- **T8**: Max 5 prefixes, max 5 suffixes

### Material-Based Attribute Segregation

Different materials favor different attribute types, creating natural item specialization:

#### Armor Materials

**Physical Armor Materials (Iron, Steel, Plate):**
- **Primary Focus**: Physical resistance, Armor, Health regeneration
- **Common Prefixes**: "Fortified", "Reinforced", "Sturdy"
- **Common Suffixes**: "of Protection", "of Endurance", "of Resilience"
- **Negative Modifiers**: Reduced magic resistance, reduced cast speed

**Elemental Armor Materials (Mithril, Adamantite, Crystal):**
- **Primary Focus**: Elemental resistances (Fire, Cold, Energy), Magic barriers, Mana/Stamina regeneration
- **Common Prefixes**: "Enchanted", "Arcane", "Elemental"
- **Common Suffixes**: "of Warding", "of Elements", "of Regeneration"
- **Negative Modifiers**: Reduced physical resistance, reduced physical damage

**Hybrid Materials (Leather, Chain, Scale):**
- **Balanced**: Mix of physical and elemental resistances
- **Common Prefixes**: "Balanced", "Adaptive", "Versatile"
- **Common Suffixes**: "of Balance", "of Adaptation", "of Versatility"
- **Negative Modifiers**: Lower maximum values in both categories

#### Weapon Materials

**Physical Weapon Materials (Iron, Steel, Obsidian):**
- **Primary Focus**: Physical damage, Attack speed, Critical chance
- **Common Prefixes**: "Sharp", "Brutal", "Vicious"
- **Common Suffixes**: "of Slaying", "of Speed", "of Precision"
- **Negative Modifiers**: Reduced magic damage, reduced cast speed

**Magic Weapon Materials (Crystal, Arcane, Enchanted):**
- **Primary Focus**: Elemental damage, Cast speed, Spell damage, Mana efficiency
- **Common Prefixes**: "Arcane", "Enchanted", "Mystical"
- **Common Suffixes**: "of Power", "of Casting", "of Efficiency"
- **Negative Modifiers**: Reduced physical damage, reduced attack speed

**Hybrid Weapon Materials (Mithril, Adamantite):**
- **Balanced**: Mix of physical and magic damage
- **Common Prefixes**: "Balanced", "Harmonized", "Dual"
- **Common Suffixes**: "of Harmony", "of Balance", "of Duality"
- **Negative Modifiers**: Lower maximum values in both categories

### Armor-Specific Prefixes and Suffixes

#### Positive Prefixes (Armor)
- **"Fortified"**: +Physical Resistance, +Armor
- **"Enchanted"**: +Elemental Resistances, +Magic Barrier
- **"Reinforced"**: +Durability, +Physical Resistance
- **"Arcane"**: +Mana Regeneration, +Spell Resistance
- **"Sturdy"**: +Health Regeneration, +Armor
- **"Resilient"**: +All Resistances (small amounts)
- **"Regenerating"**: +Health/Mana/Stamina Regeneration

#### Negative Prefixes (Armor)
- **"Cursed"**: -All Resistances, -Armor
- **"Fragile"**: -Durability, -Physical Resistance
- **"Draining"**: -Mana Regeneration, -Stamina Regeneration
- **"Vulnerable"**: -Magic Barrier, -Elemental Resistances
- **"Heavy"**: -Movement Speed, +Weight

#### Positive Suffixes (Armor)
- **"of Protection"**: +Physical Resistance, +Armor
- **"of Warding"**: +Elemental Resistances, +Magic Barrier
- **"of Endurance"**: +Health Regeneration, +Stamina Regeneration
- **"of Resilience"**: +All Resistances (small amounts)
- **"of Regeneration"**: +Health/Mana/Stamina Regeneration
- **"of Fortitude"**: +Armor, +Health

#### Negative Suffixes (Armor)
- **"of Weakness"**: -Physical Resistance, -Armor
- **"of Vulnerability"**: -Elemental Resistances, -Magic Barrier
- **"of Decay"**: -Durability, -All Resistances
- **"of Burden"**: +Weight, -Movement Speed

### Weapon-Specific Prefixes and Suffixes

#### Positive Prefixes (Weapons)
- **"Sharp"**: +Physical Damage, +Critical Chance
- **"Arcane"**: +Elemental Damage, +Spell Damage
- **"Swift"**: +Attack Speed, +Cast Speed
- **"Brutal"**: +Physical Damage, +Critical Damage
- **"Enchanted"**: +Magic Damage, +Cast Speed
- **"Precise"**: +Critical Chance, +Accuracy
- **"Powerful"**: +All Damage Types

#### Negative Prefixes (Weapons)
- **"Dull"**: -Physical Damage, -Critical Chance
- **"Cursed"**: -All Damage Types
- **"Slow"**: -Attack Speed, -Cast Speed
- **"Unstable"**: -Critical Chance, -Accuracy
- **"Draining"**: -Mana Efficiency, -Stamina Efficiency

#### Positive Suffixes (Weapons)
- **"of Slaying"**: +Physical Damage, +Critical Damage
- **"of Power"**: +Magic Damage, +Spell Damage
- **"of Speed"**: +Attack Speed, +Cast Speed
- **"of Precision"**: +Critical Chance, +Accuracy
- **"of Efficiency"**: +Mana Efficiency, +Stamina Efficiency
- **"of Elements"**: +Elemental Damage Types

#### Negative Suffixes (Weapons)
- **"of Weakness"**: -Physical Damage, -Magic Damage
- **"of Slowness"**: -Attack Speed, -Cast Speed
- **"of Inaccuracy"**: -Critical Chance, -Accuracy
- **"of Waste"**: -Mana Efficiency, -Stamina Efficiency

### Tier-Based Attribute Maximums

Each tier limits the maximum value of attributes, creating natural progression from level 1 to 100:

**T0 (Tier 0 - Level 1-10):**
- Max attribute value: 1-5
- Max resistance: 5
- Max damage bonus: 2
- Max speed bonus: 5%

**T1 (Tier 1 - Level 11-20):**
- Max attribute value: 1-10
- Max resistance: 10
- Max damage bonus: 5
- Max speed bonus: 10%

**T2 (Tier 2 - Level 21-30):**
- Max attribute value: 6-15
- Max resistance: 15
- Max damage bonus: 8
- Max speed bonus: 15%

**T3 (Tier 3 - Level 31-40):**
- Max attribute value: 11-20
- Max resistance: 20
- Max damage bonus: 12
- Max speed bonus: 20%

**T4 (Tier 4 - Level 41-50):**
- Max attribute value: 16-25
- Max resistance: 25
- Max damage bonus: 15
- Max speed bonus: 25%

**T5 (Tier 5 - Level 51-60):**
- Max attribute value: 21-30
- Max resistance: 30
- Max damage bonus: 20
- Max speed bonus: 30%

**T6 (Tier 6 - Level 61-70):**
- Max attribute value: 26-35
- Max resistance: 35
- Max damage bonus: 25
- Max speed bonus: 35%

**T7 (Tier 7 - Level 71-80):**
- Max attribute value: 31-40
- Max resistance: 40
- Max damage bonus: 30
- Max speed bonus: 40%

**T8 (Tier 8 - Level 81-100):**
- Max attribute value: 36-50
- Max resistance: 50
- Max damage bonus: 40
- Max speed bonus: 50%

### Stat Requirements by Tier

Higher tier items require more stats, creating natural class filtering and progression:

**T0 (Level 1-10):**
- Required Strength: 0-5
- Required Dexterity: 0-5
- Required Intelligence: 0-5

**T1 (Level 11-20):**
- Required Strength: 6-12
- Required Dexterity: 6-12
- Required Intelligence: 6-12

**T2 (Level 21-30):**
- Required Strength: 13-20
- Required Dexterity: 13-20
- Required Intelligence: 13-20

**T3 (Level 31-40):**
- Required Strength: 21-30
- Required Dexterity: 21-30
- Required Intelligence: 21-30

**T4 (Level 41-50):**
- Required Strength: 31-42
- Required Dexterity: 31-42
- Required Intelligence: 31-42

**T5 (Level 51-60):**
- Required Strength: 43-55
- Required Dexterity: 43-55
- Required Intelligence: 43-55

**T6 (Level 61-70):**
- Required Strength: 56-68
- Required Dexterity: 56-68
- Required Intelligence: 56-68

**T7 (Level 71-80):**
- Required Strength: 69-80
- Required Dexterity: 69-80
- Required Intelligence: 69-80

**T8 (Level 81-100):**
- Required Strength: 81-100
- Required Dexterity: 81-100
- Required Intelligence: 81-100

**Class Filtering:**
- **Warrior Builds**: High Strength requirements → Physical armor/weapons
- **Rogue Builds**: High Dexterity requirements → Light armor, fast weapons
- **Mage Builds**: High Intelligence requirements → Elemental armor, magic weapons
- **Hybrid Builds**: Balanced requirements → Hybrid materials

**Progression Curve:**
- Early game (T0-T1): Low requirements, accessible to all builds
- Mid game (T2-T4): Moderate requirements, specialization begins
- Late game (T5-T7): High requirements, build-specific items
- End game (T8): Maximum requirements, requires optimized builds

### Prefix/Suffix Value Calculation

**Positive Modifiers:**
- Value range: Based on tier maximums
- Can roll up to tier maximum
- Multiple prefixes/suffixes stack additively

**Negative Modifiers:**
- Value range: 50-100% of positive modifier value
- Creates trade-offs for powerful items
- Can enable unique builds (e.g., negative resistance for damage boost)

**Example:**
```
T3 Iron Sword (Level 31-40):
- Prefix: "Sharp" (+12 Physical Damage, +8% Critical Chance)
- Suffix: "of Slowness" (-10% Attack Speed)
- Net Effect: High damage but slower attacks

T8 Enchanted Staff (Level 81-100):
- Prefixes: "Arcane" (+40 Elemental Damage), "Swift" (+40% Cast Speed), "Powerful" (+35 All Damage)
- Suffixes: "of Power" (+40 Spell Damage), "of Efficiency" (+45% Mana Efficiency)
- Net Effect: Maximum end-game magic weapon
```

### Crafting System Integration

**Crafting Mechanics:**
1. **Base Item**: Select base item and material type
2. **Tier Selection**: Choose tier (affects max attributes and requirements)
3. **Prefix/Suffix Rolling**: Randomly roll prefixes/suffixes based on rarity
4. **Modifier Rerolling**: Use crafting materials to reroll specific modifiers
5. **Modifier Locking**: Lock desired modifiers while rerolling others
6. **Modifier Removal**: Remove unwanted modifiers (costly)

**Crafting Materials:**
- **Essence of Strength**: Reroll physical-related modifiers
- **Essence of Magic**: Reroll magic-related modifiers
- **Essence of Speed**: Reroll speed-related modifiers
- **Essence of Resistance**: Reroll resistance-related modifiers
- **Chaos Essence**: Completely reroll all modifiers

**RMT Economy:**
- Rare prefix/suffix combinations create valuable items
- Perfect rolls (max tier values) are extremely rare
- Negative modifiers that enable builds create niche markets
- Material-specific perfect items command premium prices

### Prefix/Suffix Serialization

**Storage Format:**
```typescript
interface ItemModifiers {
    prefixes: Array<{
        id: string;           // Modifier ID
        name: string;         // Display name
        attributes: Map<AttributeType, number>;  // Attribute changes
        isNegative: boolean;  // Is negative modifier
    }>;
    suffixes: Array<{
        id: string;
        name: string;
        attributes: Map<AttributeType, number>;
        isNegative: boolean;
    }>;
}
```

**Serialization:**
```typescript
public serealize() : any {
    return {
        // ... base item properties
        prefixes: this.prefixes.map(p => ({
            id: p.id,
            name: p.name,
            attributes: Array.from(p.attributes),
            isNegative: p.isNegative
        })),
        suffixes: this.suffixes.map(s => ({
            id: s.id,
            name: s.name,
            attributes: Array.from(s.attributes),
            isNegative: s.isNegative
        }))
    };
}
```

### Display Name Generation

**Name Construction:**
```typescript
public getDisplayName(): string {
    const prefixNames = this.prefixes.map(p => p.name).join(" ");
    const suffixNames = this.suffixes.map(s => s.name).join(" ");
    
    if (prefixNames && suffixNames) {
        return `${prefixNames} ${this.Name} ${suffixNames}`;
    } else if (prefixNames) {
        return `${prefixNames} ${this.Name}`;
    } else if (suffixNames) {
        return `${this.Name} ${suffixNames}`;
    }
    return this.Name;
}
```

**Example Names:**
- "Cursed Iron Sword of Swiftness"
- "Fortified Enchanted Plate Armor of Protection"
- "Sharp Arcane Staff of Power"
- "Sturdy Leather Boots of Endurance"

## Item States (Flags)

Items use bitwise flags for state management. See [FLAGS.md](../core/FLAGS.md) for detailed documentation.

**Common Item States:**
- `None` - No special state
- `Exceptional` - Exceptional quality (+50 durability, +1 armor/resistances)
- `Insured` - Item is insured (prevents loss on death)
- `Blessed` - Item is blessed (prevents loss, +500% cost)
- `Broken` - Item is broken (durability = 0, cost = 1)

## Durability System

### Durability Mechanics

- Equipment has `MaxDurability` and current `Durability`
- Durability decreases with usage (10% chance per use)
- When durability reaches 0, item is marked as `Broken`
- Broken items have minimal value (GoldCost = 1)

### Durability Calculation

**Base Durability:**
- Starts at 25
- Tier bonuses add durability
- Rarity bonuses add durability
- Exceptional flag adds +50 durability
- Maximum durability: 450

**Durability Reduction:**
```typescript
public static async reduceDurability(ref: string, owner: Player) {
    let chanceReduce = Random.MinMaxInt(1, 10);
    if(item instanceof Equipament && chanceReduce === 1) {
        let newDurability = Math.max(equipament.Durability - 1, 0);
        equipament.setDurability(newDurability, equipament.MaxDurability);
        if(newDurability <= 0)
            equipament.Flags.addFlag(ItemStates.Broken);
    }
}
```

## Resistance System

Equipment can have resistances to different damage types:

**Resistance Types:**
- `Physical` - Physical damage resistance
- `Fire` - Fire damage resistance
- `Cold` - Cold damage resistance
- `Poison` - Poison damage resistance
- `Energy` - Energy damage resistance
- `Light` - Light damage resistance
- `Dark` - Dark damage resistance

**Resistance Properties:**
- Set via `setFireResistence()`, `setColdResistence()`, etc.
- Can be set with min/max range for randomization
- Exceptional items get +1 to all resistances if they have any

## Card System

### Card Slots

- Equipment T3+ can have card slots
- 10% chance to generate slots when item is created
- Slots replace random attributes (1-3 slots)
- Maximum slots: 3 (configurable per equipment type)

### Card Insertion

- Cards can be inserted into equipment card slots
- Cards provide additional bonuses (Attack, HP, Energy, Armor, Resistances, Attributes)
- Cards are stored in `Cards` array as string references

## Item Serialization

### TypeScript Serialization

```typescript
public serealize() : any {
    return {
        Rarity: this.Rarity,
        Hue: this.Hue,
        Flags: (this.Flags) ? this.Flags.getCurrentFlags() : new StateFlags(ItemStates.None).getCurrentFlags(),
        CraftBy: this.CraftBy
    }
}
```

**Equipment Serialization:**
```typescript
public override serealize() : any {
    let data = super.serealize();
    data.MaxDurability = this.MaxDurability;
    data.Durability = this.Durability;
    data.Armor = this.Armor;
    data.FireResistence = this.FireResistence;
    // ... other resistances
    data.Attr = Array.from(this.Attrs, ([type, value]) => ({ type, value }));
    data.Weight = this.Weight;
    data.GoldCost = this.GoldCost;
    data.CardSlots = this.CardSlots;
    data.Cards = this.Cards;
    return data;
}
```

### C# Serialization

```csharp
public override object Serialize() {
    return new {
        name = Name,
        craftBy = CraftBy,
        quality = Quality.ToString(),
        durability = Durability,
        maxDurability = MaxDurability,
        // ... attributes and properties
    };
}
```

## Item Management System

### Item Registration

**TypeScript:**
```typescript
export class Items {
    public static BaseItems: Map<string, { new (): any }> = new Map();
    public static CachedItems: Map<string, Item> = new Map();
    
    public static AddBaseItem(refs: string[] | string, clas: any) {
        // Register item class by namespace
    }
    
    public static loadFromDatabase(namespace: string, ref: string, props: any) : Item | null {
        // Load item from database with properties
    }
}
```

**C#:**
```csharp
public static Dictionary<string, Item> Items = new Dictionary<string, Item>();

public static void Reload() {
    // Load items from YAML via API
    string yml = HttpGetter.GetWithRetry(Config.Api + "/gamedata/items/public.yml");
    // Parse YAML and create item prefabs
}
```

### Item Creation

**From Base Item:**
```typescript
public static createItem(baseItemName: string, attrs: any, createBy: string = "") : Item | null {
    const base = Items.BaseItems.get(baseItemName);
    const item = new base();
    item.generateAttrs();
    item.generateRandomAttrs();
    item.updateGoldCost();
    item.CraftBy = createBy;
    return item;
}
```

**From Database:**
```typescript
public static itemFromDatabase(data: any) : Item | null {
    const item = Items.loadFromDatabase(itemName, ref, props);
    if(item) {
        item.ContainerId = containerId;
        if(amount > 1 && (item as Stackable))
            (item as Stackable).Amount = amount;
        Items.CachedItems.set(ref, item);
    }
    return item;
}
```

## Dynamic Item Registration

### API/Dashboard System

Items can be registered dynamically via API/dashboard without client recompilation:

1. **Item Definition**: Items are defined in YAML format
2. **API Endpoint**: `/gamedata/items/public.yml`
3. **Client Loading**: Client loads items on startup or on-demand
4. **Caching**: Items are cached locally for performance
5. **Image Loading**: Item images loaded from CDN with local caching

### YAML Structure (C# Example)

```yaml
- Type: Weapon
  Name: Iron Sword
  Damage: 10
  Durability: 100
  Price: 50
  WeaponType: Sword
  HandType: OneHand
  VisualName: IronSword
  Attrs:
    BonusDamage: 5
    BonusStr: 3
```

### Item Properties

**Common Properties:**
- `Name` - Item name
- `Type` - Item type (Weapon, Armor, Consumable, etc.)
- `Price` - Base price
- `Weight` - Item weight
- `VisualName` - Visual reference name
- `Rarity` - Item rarity

**Equipment Properties:**
- `Durability` - Max durability
- `PhysicalResistance` - Physical resistance
- `FireResistance` - Fire resistance
- `ColdResistance` - Cold resistance
- `PoisonResistance` - Poison resistance
- `EnergyResistance` - Energy resistance
- `LightningResistance` - Lightning resistance
- `DarknessResistance` - Darkness resistance
- `StrRequeriment` - Strength requirement
- `DexRequeriment` - Dexterity requirement
- `IntRequeriment` - Intelligence requirement
- `Attrs` - Attribute bonuses

## Image Loading System

### CDN Integration

- Item images stored in CDN
- Images loaded on-demand when needed
- Local caching for performance
- No need for client recompilation when adding items

### Image Reference

- Items reference images by `VisualName`
- Visual dictionary maps names to visual indices
- Client loads images based on visual index
- Supports multiple visual variants (colors, styles)

## Gold Cost Calculation

### Cost Formula

```typescript
public updateGoldCost() {
    switch(this.Rarity) {
        case ItemRarity.Uncommon: this.GoldCost += this.GoldCost * 0.3; break;
        case ItemRarity.Rare: this.GoldCost += this.GoldCost * 0.5; break;
        case ItemRarity.Magic: this.GoldCost += this.GoldCost * 0.8; break;
        case ItemRarity.Legendary: this.GoldCost += this.GoldCost; break;
    }
    
    if(this.Flags.hasFlag(ItemStates.Exceptional))
        this.GoldCost += this.GoldCost * 0.15;
    
    if(this.Flags.hasFlag(ItemStates.Insured))
        this.GoldCost += this.GoldCost;
    
    if(this.Flags.hasFlag(ItemStates.Blessed))
        this.GoldCost += this.GoldCost * 5;
    
    if(this.Flags.hasFlag(ItemStates.Broken))
        this.GoldCost = 1;
}
```

## Rust Implementation Considerations

### Performance Optimizations

1. **Zero-Allocation Item Pooling**: Pre-allocate item instances for common items
2. **Structure of Arrays (SoA)**: Store item properties in separate arrays for cache efficiency
3. **Component-Based**: Use ECS pattern for item properties
4. **Lazy Loading**: Load item definitions on-demand
5. **Cache-Friendly**: Organize item data for CPU cache efficiency

### Memory Management

- Use `Arc<ItemDefinition>` for shared item definitions
- Use `Rc<ItemInstance>` for item instances in containers
- Pool item instances to reduce allocations
- Use small string optimization for item names

### Serialization

- Use `bincode` or `postcard` for binary serialization
- Support both binary and JSON for API compatibility
- Implement delta serialization for item updates
- Use zero-copy deserialization where possible

### Type Safety

- Use Rust enums for item types instead of strings
- Use trait objects or generics for item behavior
- Leverage Rust's type system for compile-time safety
- Use `PhantomData` for type-level guarantees

## Integration with Other Systems

### Container System

Items are stored in containers (inventory, equipment slots, etc.). See [CONTAINERS.md](./CONTAINERS.md) for detailed documentation.

### Entity System

Items can be equipped on entities, affecting their stats and abilities. See [ENTITIES.md](../core/ENTITIES.md) for entity system documentation.

### Flags System

Items use the same flag system as entities. See [FLAGS.md](../core/FLAGS.md) for flag system documentation.

### Network System

Item updates are synchronized via network packets. Item serialization must be compatible with the ByteBuffer system. See [BYTEBUFFER.md](../core/BYTEBUFFER.md) for serialization details.

## Performance Considerations

### Item Lookup

- Use hash maps for O(1) item lookup by name/ref
- Cache frequently accessed items
- Use string interning for item names
- Pre-compute item properties where possible

### Attribute Calculation

- Cache calculated attribute totals
- Invalidate cache on item changes
- Use incremental updates where possible
- Batch attribute recalculations

### Serialization Performance

- Pre-allocate serialization buffers
- Use streaming serialization for large item lists
- Compress item data for network transmission
- Use delta compression for item updates

## Testing Strategy

### Unit Tests

- Test item creation and initialization
- Test attribute generation and calculation
- Test durability system
- Test serialization/deserialization
- Test rarity and tier effects

### Integration Tests

- Test item registration system
- Test item loading from database
- Test item stacking logic
- Test equipment slot assignment
- Test card insertion system

### Performance Tests

- Benchmark item lookup performance
- Benchmark attribute calculation
- Benchmark serialization performance
- Test memory usage with large item counts
- Test CDN image loading performance

