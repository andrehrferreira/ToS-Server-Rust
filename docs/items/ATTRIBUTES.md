# Item Attributes System Documentation

## Overview

The item attributes system provides a comprehensive set of modifiers that can appear on equipment items, similar to Path of Exile 2's modifier system. Attributes can enhance combat effectiveness, resource management, gathering efficiency, and provide various quality-of-life improvements. Each attribute has specific spawn rates, equipment restrictions, effects, and value impacts that determine how rare and valuable it is.

## Key Features

- **90+ Attribute Types**: Comprehensive attribute system covering all aspects of gameplay
- **Penetration Attributes**: Essential for DPS builds to break through enemy defenses
- **Spawn Rates**: Each attribute has a specific chance to appear on items
- **Equipment Restrictions**: Attributes are limited to specific equipment types (weapons, armor, accessories, tools)
- **Tier System**: Attributes have tiers that determine their value ranges
- **Value Impact**: Attributes increase item value based on their type and tier
- **Build Diversity**: Attributes enable diverse character builds and playstyles

## Attribute Categories

### 1. Core Stat Attributes

Attributes that directly increase character statistics. These attributes are inspired by Path of Exile 2's attribute system, where each core attribute provides specific benefits and is required for certain equipment.

#### Base Attribute Effects (PoE 2 Inspired)

**Strength (STR):**
- **Primary Effect**: Each point grants +2 Maximum Health Points
- **Combat Role**: Linked to melee combat weapon damage and abilities
- **Equipment Requirements**: Required to equip certain martial weapons and armor pieces
- **Build Focus**: Physical damage builds, tank builds, melee combat builds

**Dexterity (DEX):**
- **Primary Effect**: Each point grants +5 Accuracy
- **Combat Role**: Linked to ranged weapon damage and abilities
- **Equipment Requirements**: Required to equip certain ranged weapons and evasion-based equipment
- **Build Focus**: Ranged damage builds, evasion builds, mobility builds

**Intelligence (INT):**
- **Primary Effect**: Each point grants +2 Maximum Mana
- **Combat Role**: Linked to magical damage and spell abilities
- **Equipment Requirements**: Required to equip certain magical weapons and energy shield-based equipment
- **Build Focus**: Magic damage builds, spellcaster builds, energy shield builds

#### BonusStr (Bonus Strength)
- **Effect**: Increases STR attribute
- **Base STR Effect**: Each STR point grants +2 Maximum HP
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +50 gold per point
- **Build Focus**: Physical DPS, Tanks, Melee combat builds

#### BonusDex (Bonus Dexterity)
- **Effect**: Increases DEX attribute
- **Base DEX Effect**: Each DEX point grants +5 Accuracy
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +50 gold per point
- **Build Focus**: Rangers, Rogues, Mobility builds, Evasion builds

#### BonusInt (Bonus Intelligence)
- **Effect**: Increases INT attribute
- **Base INT Effect**: Each INT point grants +2 Maximum Mana
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +50 gold per point
- **Build Focus**: Mages, Spellcasters, Magic damage builds, Energy shield builds

#### BonusVig (Bonus Vigor/Constitution)
- **Effect**: Increases VIG attribute
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +60 gold per point (higher value due to HP scaling)
- **Build Focus**: Tanks, Survivability builds

#### BonusAgi (Bonus Agility)
- **Effect**: Increases AGI attribute
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +50 gold per point
- **Build Focus**: Evasion builds, Speed builds

#### BonusLuc (Bonus Luck)
- **Effect**: Increases LUC attribute
- **Value Range**: +1 to +50 (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +75 gold per point (higher value due to rarity)
- **Build Focus**: Loot-focused builds, Crafters

### 2. Resource Attributes

Attributes that increase maximum resources or regeneration.

#### BonusLife (Bonus Health Points)
- **Effect**: Increases maximum HP
- **Value Range**: +100 to +5,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 18% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +1 gold per 10 HP
- **Build Focus**: Tanks, Survivability builds

#### BonusMana (Bonus Mana Points)
- **Effect**: Increases maximum Mana
- **Value Range**: +100 to +3,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +1 gold per 10 Mana
- **Build Focus**: Mages, Spellcasters

#### BonusStam (Bonus Stamina Points)
- **Effect**: Increases maximum Stamina
- **Value Range**: +100 to +3,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +1 gold per 10 Stamina
- **Build Focus**: Mobility builds, Rogues

#### HealthRegen (Health Regeneration)
- **Effect**: Increases HP regeneration per tick
- **Value Range**: +10 to +500 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +2 gold per point
- **Build Focus**: Tanks, Sustain builds

#### ManaRegen (Mana Regeneration)
- **Effect**: Increases Mana regeneration per tick
- **Value Range**: +10 to +500 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +2 gold per point
- **Build Focus**: Mages, Sustain builds

#### StaminaRegen (Stamina Regeneration)
- **Effect**: Increases Stamina regeneration per tick
- **Value Range**: +10 to +500 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +2 gold per point
- **Build Focus**: Mobility builds, Rogues

### 3. Damage Attributes

Attributes that increase damage output.

#### BonusDamage (Bonus Physical Damage)
- **Effect**: Increases physical damage (multiplied by 10)
- **Value Range**: +50 to +800 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +8 gold per point (increased value due to reduced availability)
- **Build Focus**: Physical DPS builds
- **Note**: Reduced from previous values to prevent excessive damage scaling

#### BonusMagicDamage (Bonus Magic Damage)
- **Effect**: Increases magic/elemental damage (multiplied by 10)
- **Value Range**: +50 to +800 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 15% (common)
- **Equipment Restrictions**: Weapons, Staffs, Wands
- **Value Impact**: +8 gold per point (increased value due to reduced availability)
- **Build Focus**: Mages, Spellcasters
- **Note**: Reduced from previous values to prevent excessive damage scaling

#### SpellDamage (Spell Damage)
- **Effect**: Increases spell damage specifically (multiplied by 10)
- **Value Range**: +50 to +800 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: Weapons, Staffs, Wands
- **Value Impact**: +10 gold per point (increased value due to reduced availability)
- **Build Focus**: Pure mage builds
- **Note**: Reduced from previous values to prevent excessive damage scaling

#### FireDamage (Fire Damage)
- **Effect**: Adds fire damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +4 gold per point
- **Build Focus**: Fire mages, Elemental builds

#### ColdDamage (Cold Damage)
- **Effect**: Adds cold damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +4 gold per point
- **Build Focus**: Ice mages, Elemental builds

#### PoisonDamage (Poison Damage)
- **Effect**: Adds poison damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +4 gold per point
- **Build Focus**: Poison builds, DoT builds

#### EnergyDamage (Energy Damage)
- **Effect**: Adds energy/lightning damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +4 gold per point
- **Build Focus**: Lightning mages, Elemental builds

#### LightDamage (Light Damage)
- **Effect**: Adds light/holy damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 6% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +5 gold per point
- **Build Focus**: Paladins, Holy builds

#### DarkDamage (Dark Damage)
- **Effect**: Adds dark/shadow damage to attacks (multiplied by 10)
- **Value Range**: +100 to +1,000 (tier-dependent, multiplied by 10)
- **Spawn Rate**: 6% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +5 gold per point
- **Build Focus**: Necromancers, Shadow builds

### 3.1. Penetration Attributes

Attributes that ignore or reduce enemy defenses, essential for DPS builds to break through protections.

#### PhysicalPenetration (Physical Resistance Penetration)
- **Effect**: Ignores target's physical resistance (%)
- **Value Range**: +1% to +50% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +150 gold per 1% (high value due to effectiveness against tanks)
- **Build Focus**: Physical DPS builds, Tank killers
- **Mechanics**: Reduces target's effective physical resistance before damage calculation
- **Example**: Target has 60% physical resistance, attacker has 30% penetration → Effective resistance: 30%

#### ElementalPenetration (Elemental Resistance Penetration)
- **Effect**: Ignores target's elemental resistances (%)
- **Value Range**: +1% to +50% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons, Staffs, Wands
- **Value Impact**: +150 gold per 1% (high value due to effectiveness against mages)
- **Build Focus**: Magic DPS builds, Mage killers
- **Mechanics**: Reduces target's effective elemental resistance (Fire, Cold, Poison, Energy, Light, Dark) before damage calculation
- **Example**: Target has 50% fire resistance, attacker has 25% penetration → Effective resistance: 25%

#### ArmorPenetration (Armor Penetration)
- **Effect**: Ignores target's armor (%)
- **Value Range**: +1% to +50% (tier-dependent)
- **Spawn Rate**: 7% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +200 gold per 1% (very high value due to effectiveness against heavy armor)
- **Build Focus**: Physical DPS builds, Armor breakers
- **Mechanics**: Reduces target's effective armor before damage calculation
- **Example**: Target has 70% armor, attacker has 40% penetration → Effective armor: 30%

#### EvasionPenetration (Evasion Penetration)
- **Effect**: Reduces target's evasion chance (%)
- **Value Range**: +1% to +30% (tier-dependent)
- **Spawn Rate**: 6% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +180 gold per 1% (high value due to effectiveness against evasion builds)
- **Build Focus**: DPS builds, Evasion counters
- **Mechanics**: Reduces target's evasion chance before evasion roll
- **Example**: Target has 60% evasion, attacker has 25% penetration → Effective evasion: 35%

#### BlockPenetration (Block Penetration)
- **Effect**: Reduces target's block chance and effectiveness (%)
- **Value Range**: +1% to +30% (tier-dependent)
- **Spawn Rate**: 6% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +180 gold per 1% (high value due to effectiveness against shield builds)
- **Build Focus**: DPS builds, Shield breakers
- **Mechanics**: Reduces target's block chance and block effectiveness before block roll
- **Example**: Target has 50% block chance, attacker has 20% penetration → Effective block chance: 30%

#### EnergyShieldPenetration (Energy Shield Penetration)
- **Effect**: Ignores target's energy shield (%)
- **Value Range**: +1% to +50% (tier-dependent)
- **Spawn Rate**: 7% (uncommon)
- **Equipment Restrictions**: Weapons, Staffs, Wands
- **Value Impact**: +200 gold per 1% (very high value due to effectiveness against energy shield builds)
- **Build Focus**: DPS builds, Energy shield breakers
- **Mechanics**: Percentage of damage bypasses energy shield and goes directly to HP
- **Example**: Attacker has 30% penetration → 30% of damage bypasses energy shield, 70% is absorbed by shield first

#### AllResistancePenetration (All Resistance Penetration)
- **Effect**: Ignores all target's resistances (physical and elemental) (%)
- **Value Range**: +1% to +25% (tier-dependent)
- **Spawn Rate**: 4% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +300 gold per 1% (extremely high value due to universal effectiveness)
- **Build Focus**: Universal DPS builds, Versatile damage dealers
- **Mechanics**: Reduces all resistances (physical and all elemental types) before damage calculation
- **Example**: Target has 50% physical and 40% fire resistance, attacker has 20% all penetration → Effective resistances: 30% physical, 20% fire

#### DefensePenetration (Defense Penetration)
- **Effect**: Ignores all target's defensive stats (armor, evasion, block) (%)
- **Value Range**: +1% to +25% (tier-dependent)
- **Spawn Rate**: 4% (rare)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +300 gold per 1% (extremely high value due to universal effectiveness)
- **Build Focus**: Universal DPS builds, Defense breakers
- **Mechanics**: Reduces armor, evasion, and block effectiveness before damage calculation
- **Example**: Target has 60% armor and 40% evasion, attacker has 15% defense penetration → Effective armor: 45%, Effective evasion: 25%

### 4. Critical Strike Attributes

Attributes that affect critical strike chance and damage.

#### CriticalChance (Critical Strike Chance)
- **Effect**: Increases chance to deal critical damage (%)
- **Value Range**: +1% to +25% (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: Weapons, Accessories
- **Value Impact**: +100 gold per 1%
- **Build Focus**: Critical strike builds, DPS builds

#### CriticalDamage (Critical Strike Damage)
- **Effect**: Increases critical strike damage multiplier (%)
- **Value Range**: +10% to +200% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons only
- **Value Impact**: +50 gold per 10%
- **Build Focus**: Critical strike builds, Burst DPS

### 5. Defense Attributes

Attributes that increase survivability.

#### DamageReduction (Damage Reduction)
- **Effect**: Reduces all incoming damage (%)
- **Value Range**: +1% to +15% (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: Armor, Accessories
- **Value Impact**: +200 gold per 1%
- **Build Focus**: Tanks, Survivability builds

#### DodgeChance (Dodge Chance)
- **Effect**: Increases chance to dodge attacks (%)
- **Value Range**: +1% to +20% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Light Armor, Accessories
- **Value Impact**: +150 gold per 1%
- **Build Focus**: Evasion builds, Rogues

#### ReflectPhysical (Reflect Physical Damage)
- **Effect**: Reflects physical damage back to attacker (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 5% (rare)
- **Equipment Restrictions**: Armor, Accessories
- **Value Impact**: +100 gold per 5%
- **Build Focus**: Thorns builds, Tanks

#### ReflectSpell (Reflect Spell Damage)
- **Effect**: Reflects spell damage back to attacker (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 5% (rare)
- **Equipment Restrictions**: Armor, Accessories
- **Value Impact**: +100 gold per 5%
- **Build Focus**: Anti-mage builds, Tanks

### 6. Speed Attributes

Attributes that increase movement and action speed.

#### BonusSpeed (Movement Speed)
- **Effect**: Increases movement speed (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 12% (common)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +50 gold per 5%
- **Build Focus**: Mobility builds, Farmers

#### CastSpeed (Cast Speed)
- **Effect**: Increases spell casting speed (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: Weapons, Accessories
- **Value Impact**: +75 gold per 5%
- **Build Focus**: Mages, Casters

#### FasterCasting (Faster Casting)
- **Effect**: Reduces cast time for spells (%)
- **Value Range**: +5% to +40% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Weapons, Staffs, Wands
- **Value Impact**: +80 gold per 5%
- **Build Focus**: Pure mage builds

### 7. Efficiency Attributes

Attributes that reduce resource costs.

#### LowerManaCost (Lower Mana Cost)
- **Effect**: Reduces mana cost for spells (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: Weapons, Accessories
- **Value Impact**: +100 gold per 5%
- **Build Focus**: Sustain mages, Efficiency builds

#### LowerStamCost (Lower Stamina Cost)
- **Effect**: Reduces stamina cost for skills (%)
- **Value Range**: +5% to +50% (tier-dependent)
- **Spawn Rate**: 10% (uncommon)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +80 gold per 5%
- **Build Focus**: Skill spam builds, Mobility builds

#### CooldownReduction (Cooldown Reduction)
- **Effect**: Reduces skill cooldown time (%)
- **Value Range**: +5% to +40% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +150 gold per 5%
- **Build Focus**: Skill-focused builds, DPS builds

### 8. Luck and Economy Attributes

Attributes that affect loot and economy.

#### Luck (Luck)
- **Effect**: Increases chance to find rare items and gold (%)
- **Value Range**: +5% to +100% (tier-dependent)
- **Spawn Rate**: 5% (rare)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: +500 gold per 5% (very high value)
- **Build Focus**: Farmers, Loot-focused builds
- **Special**: Can stack across multiple items for significant bonuses

#### IncreasedKarmaLoss (Increased Karma Loss)
- **Effect**: Increases karma loss on death (%)
- **Value Range**: +10% to +100% (tier-dependent)
- **Spawn Rate**: 3% (very rare, negative attribute)
- **Equipment Restrictions**: All equipment types
- **Value Impact**: -200 gold per 10% (reduces value)
- **Build Focus**: None (negative attribute, usually avoided)

### 9. Gathering Attributes

Attributes that improve gathering efficiency.

#### BonusCollectsMineral (Bonus Mineral Collection)
- **Effect**: Increases mineral gathering yield (%)
- **Value Range**: +10% to +200% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Tools (Pickaxe), Accessories
- **Value Impact**: +100 gold per 10%
- **Build Focus**: Miners, Gatherers

#### BonusCollectsSkins (Bonus Skin Collection)
- **Effect**: Increases skin/leather gathering yield (%)
- **Value Range**: +10% to +200% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Tools (Scythe), Accessories
- **Value Impact**: +100 gold per 10%
- **Build Focus**: Skinners, Gatherers

#### BonusCollectsWood (Bonus Wood Collection)
- **Effect**: Increases wood gathering yield (%)
- **Value Range**: +10% to +200% (tier-dependent)
- **Spawn Rate**: 8% (uncommon)
- **Equipment Restrictions**: Tools (Axe), Accessories
- **Value Impact**: +100 gold per 10%
- **Build Focus**: Lumberjacks, Gatherers

## Attribute Spawn System

### Spawn Rate Categories

**Common (10-20% spawn rate):**
- Core stat attributes (STR, DEX, INT, VIG, AGI)
- Resource attributes (HP, Mana, Stamina)
- Basic damage attributes
- Regeneration attributes

**Uncommon (5-10% spawn rate):**
- Elemental damage attributes
- Critical strike attributes
- Defense attributes
- Speed attributes
- Efficiency attributes
- Gathering attributes
- Penetration attributes (Physical, Elemental, Armor, Energy Shield)

**Rare (3-5% spawn rate):**
- Luck
- Light/Dark damage
- Reflect attributes
- Cooldown reduction
- Penetration attributes (Evasion, Block, All Resistance, Defense)

**Very Rare (1-3% spawn rate):**
- Negative attributes (IncreasedKarmaLoss)
- Perfect roll attributes
- Tier 8 attributes

### Equipment-Specific Attribute Pools

**All Equipment:**
- Core stats (STR, DEX, INT, VIG, AGI, LUC)
- Resources (HP, Mana, Stamina)
- Regeneration (HP, Mana, Stamina)
- Movement Speed
- Luck
- Karma Loss

**Weapons Only:**
- Physical Damage
- Magic Damage
- Spell Damage
- Elemental Damage (Fire, Cold, Poison, Energy, Light, Dark)
- Critical Chance
- Critical Damage
- Cast Speed
- Faster Casting
- Lower Mana Cost
- Penetration Attributes (Physical, Elemental, Armor, Evasion, Block, Energy Shield, All Resistance, Defense)

**Armor Only:**
- Damage Reduction
- Dodge Chance
- Reflect Physical
- Reflect Spell

**Accessories Only:**
- All core stats
- Resources
- Regeneration
- Critical Chance
- Damage Reduction
- Dodge Chance
- Reflect attributes
- Luck

**Tools Only:**
- Gathering attributes (Mineral, Skins, Wood)
- Core stats (limited)
- Resources (limited)

## Attribute Tiers

### Tier System

Attributes have tiers (T1-T8) that determine their value ranges:

**Tier 1 (Level 1-10):**
- Minimum values
- Common spawn rate
- Low value impact

**Tier 2 (Level 11-20):**
- Slightly higher values
- Common spawn rate
- Low-medium value impact

**Tier 3 (Level 21-30):**
- Moderate values
- Common spawn rate
- Medium value impact

**Tier 4 (Level 31-40):**
- Good values
- Common-uncommon spawn rate
- Medium-high value impact

**Tier 5 (Level 41-50):**
- High values
- Uncommon spawn rate
- High value impact

**Tier 6 (Level 51-60):**
- Very high values
- Uncommon-rare spawn rate
- Very high value impact

**Tier 7 (Level 61-80):**
- Excellent values
- Rare spawn rate
- Excellent value impact

**Tier 8 (Level 81-100):**
- Maximum values
- Very rare spawn rate
- Maximum value impact

### Tier Value Ranges Example

**BonusStr (Tier-based ranges):**
- T1: +1 to +5
- T2: +3 to +8
- T3: +5 to +12
- T4: +8 to +18
- T5: +12 to +25
- T6: +18 to +35
- T7: +25 to +45
- T8: +35 to +50

**BonusDamage (Tier-based ranges, multiplied by 10):**
- T1: +50 to +100
- T2: +80 to +150
- T3: +120 to +220
- T4: +180 to +320
- T5: +250 to +450
- T6: +350 to +600
- T7: +500 to +750
- T8: +650 to +800

## Value Impact System

### Value Calculation

**Base Formula:**
```
Attribute Value = Base Value + (Attribute Points * Value Per Point)
```

**Value Per Point Examples:**
- Core Stats: 50 gold per point
- Resources: 1 gold per 10 points (HP/Mana/Stam)
- Regeneration: 2 gold per point
- Damage: 8 gold per point (increased from 5 due to reduced availability)
- Critical Chance: 100 gold per 1%
- Critical Damage: 50 gold per 10%
- Damage Reduction: 200 gold per 1%
- Luck: 500 gold per 5%
- Gathering: 100 gold per 10%

**Rarity Multipliers:**
- Common attributes: 1.0x
- Uncommon attributes: 1.2x
- Rare attributes: 1.5x
- Very rare attributes: 2.0x

**Tier Multipliers:**
- T1-T2: 1.0x
- T3-T4: 1.2x
- T5-T6: 1.5x
- T7-T8: 2.0x

### Complete Value Calculation

```rust
pub fn calculate_attribute_value(
    attribute_type: AttributeType,
    value: f32,
    tier: u8,
    rarity: ItemRarity,
) -> u32 {
    let base_value_per_point = get_base_value_per_point(attribute_type);
    let rarity_multiplier = get_rarity_multiplier(attribute_type);
    let tier_multiplier = get_tier_multiplier(tier);
    
    let attribute_value = (value * base_value_per_point) 
        * rarity_multiplier 
        * tier_multiplier;
    
    attribute_value as u32
}
```

## Attribute Restrictions

### Equipment Type Restrictions

**Weapons:**
- Can have damage attributes
- Can have elemental damage
- Can have critical strike attributes
- Can have casting attributes
- Cannot have gathering attributes (except if tool-weapon hybrid)

**Armor:**
- Can have defense attributes
- Can have resource attributes
- Can have regeneration attributes
- Cannot have damage attributes (except reflect)
- Cannot have elemental damage

**Accessories:**
- Can have all stat attributes
- Can have resource attributes
- Can have regeneration attributes
- Can have critical chance
- Can have defense attributes
- Cannot have damage attributes (except reflect)
- Cannot have elemental damage

**Tools:**
- Can have gathering attributes
- Can have limited stat attributes
- Cannot have damage attributes
- Cannot have critical strike attributes

### Material Type Restrictions

**Physical Materials (Iron, Steel, Plate):**
- Higher chance for physical damage attributes
- Higher chance for physical resistance
- Lower chance for elemental damage
- Lower chance for magic attributes

**Elemental Materials (Mithril, Adamantite, Crystal):**
- Higher chance for elemental damage attributes
- Higher chance for elemental resistances
- Higher chance for magic attributes
- Lower chance for physical damage

**Hybrid Materials (Leather, Chain, Scale):**
- Balanced chance for all attributes
- Can roll both physical and elemental
- More versatile attribute pools

## Attribute Generation

### Random Attribute Generation

**Process:**
1. Determine number of attributes based on rarity
2. Select attribute pool based on equipment type
3. Roll for each attribute based on spawn rate
4. Determine tier based on item tier
5. Roll value within tier range
6. Calculate value impact

**Rarity-Based Attribute Count:**
- Common: 0-1 attributes
- Uncommon: 1-2 attributes
- Rare: 2-3 attributes
- Magic: 3-4 attributes
- Legendary: 4-5 attributes
- Unique: 5-6 attributes (fixed attributes)

**Implementation:**
```rust
pub fn generate_random_attributes(
    equipment_type: EquipmentType,
    rarity: ItemRarity,
    tier: u8,
) -> Vec<Attribute> {
    let attribute_pool = get_attribute_pool(equipment_type);
    let attribute_count = get_attribute_count(rarity);
    let mut attributes = Vec::new();
    
    for _ in 0..attribute_count {
        let attribute_type = roll_attribute_type(&attribute_pool);
        let value = roll_attribute_value(attribute_type, tier);
        
        attributes.push(Attribute {
            attribute_type,
            value,
            tier,
        });
    }
    
    attributes
}
```

## Rust Implementation Considerations

### Data Structures

**Attribute Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct ItemAttribute {
    pub attribute_type: AttributeType,
    pub value: f32,
    pub tier: u8,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum AttributeType {
    // Core Stats
    BonusStr,
    BonusDex,
    BonusInt,
    BonusVig,
    BonusAgi,
    BonusLuc,
    
    // Resources
    BonusLife,
    BonusMana,
    BonusStam,
    
    // Regeneration
    HealthRegen,
    ManaRegen,
    StaminaRegen,
    
    // Damage
    BonusDamage,
    BonusMagicDamage,
    SpellDamage,
    FireDamage,
    ColdDamage,
    PoisonDamage,
    EnergyDamage,
    LightDamage,
    DarkDamage,
    
    // Penetration
    PhysicalPenetration,
    ElementalPenetration,
    ArmorPenetration,
    EvasionPenetration,
    BlockPenetration,
    EnergyShieldPenetration,
    AllResistancePenetration,
    DefensePenetration,
    
    // Critical
    CriticalChance,
    CriticalDamage,
    
    // Defense
    DamageReduction,
    DodgeChance,
    ReflectPhysical,
    ReflectSpell,
    
    // Speed
    BonusSpeed,
    CastSpeed,
    FasterCasting,
    
    // Efficiency
    LowerManaCost,
    LowerStamCost,
    CooldownReduction,
    
    // Luck & Economy
    Luck,
    IncreasedKarmaLoss,
    
    // Gathering
    BonusCollectsMineral,
    BonusCollectsSkins,
    BonusCollectsWood,
}
```

### Performance Optimizations

**Caching:**
- Cache attribute pools by equipment type
- Pre-calculate spawn rates
- Cache value calculations

**Batch Processing:**
- Generate attributes for multiple items in parallel
- Pre-calculate attribute value impacts
- Batch attribute application to entities

**Memory Layout:**
- Store attributes in compact format
- Use bitflags for attribute presence
- Align attribute data for cache efficiency

### Serialization

**Attribute Serialization:**
```rust
impl Serialize for ItemAttribute {
    fn serialize(&self) -> Vec<u8> {
        let mut buffer = Vec::with_capacity(6); // 1 + 4 + 1 bytes
        
        buffer.push(self.attribute_type as u8);
        buffer.extend_from_slice(&self.value.to_le_bytes());
        buffer.push(self.tier);
        
        buffer
    }
}
```

## Integration with Other Systems

### Equipment System

**Integration:**
- Attributes are stored on equipment items
- Attributes affect equipment value
- Attributes determine item rarity eligibility

### Player System

**Integration:**
- Attributes apply bonuses to player stats
- Attributes affect resource pools
- Attributes modify combat effectiveness

### Combat System

**Integration:**
- Damage attributes affect damage calculations
- Defense attributes affect damage received
- Critical attributes affect critical strike calculations
- Penetration attributes reduce enemy defenses (resistances, armor, evasion, block, energy shield)
- Penetration attributes are essential for DPS builds to break through high-defense targets

### Gathering System

**Integration:**
- Gathering attributes increase resource yield
- Tool attributes affect gathering efficiency
- Luck affects rare resource drops

### Economy System

**Integration:**
- Attributes increase item value
- Rare attributes create valuable items
- Perfect roll attributes command premium prices

## Summary

The attribute system:

- Provides 90+ attribute types covering all gameplay aspects
- Includes penetration attributes essential for DPS builds to break through defenses
- Uses spawn rates to control attribute rarity
- Restricts attributes to appropriate equipment types
- Implements tier system for value scaling
- Calculates value impact for economy balance
- Enables diverse builds and playstyles (DPS, Tank, Support, Hybrid)
- Integrates with all major game systems

This system creates depth and replayability by allowing players to find and craft items with perfect attribute combinations for their builds, similar to Path of Exile 2's modifier system. Penetration attributes are crucial for DPS builds, allowing them to effectively deal with high-defense targets by ignoring or reducing enemy protections.

