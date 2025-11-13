# Status System Documentation

## Overview

The status system defines how core attributes (STR, DEX, INT, CON, LCK, CHA) affect an entity's primary resources: Health Points (HP), Mana, and Stamina. These calculations are balanced against the damage system to ensure fair and engaging combat. The system uses linear scaling to maintain predictable growth and prevent exponential power creep, making PvP balance achievable.

**Note:** All resource values are multiplied by 10 to provide a greater sense of progression. This means HP, Mana, and Stamina calculations result in larger numbers (e.g., 3,100 HP instead of 310) while maintaining the same balance ratios with the damage system.

**Attribute Cap:** Maximum 200 points can be allocated to a single attribute. This cap ensures balanced builds and prevents extreme min-maxing while maintaining build diversity.

## Key Features

- **Attribute-Based Resources**: HP, Mana, and Stamina scale with core attributes
- **10x Multiplier**: All resource values multiplied by 10 for greater progression feel
- **Balanced Scaling**: Resource growth balanced against damage growth
- **Linear Progression**: Predictable, linear scaling prevents exponential power creep
- **Build Diversity**: Different attribute distributions create distinct playstyles
- **PvP Balance**: Resource pools balanced to work with 50% HP damage cap

## Core Attributes

### Strength (STR)

**Primary Functions:**
- Increases Physical Damage (damage modifier)
- Increases Health Points (HP) - moderate contribution
- Increases Physical Resistance (see RESISTANCES.md)
- Increases Weight Capacity

**Resource Contributions:**
- **HP**: Moderate contribution (+25 HP per STR point, with 10x multiplier)

**Attribute Cap:** Maximum 200 points
**Max HP Contribution:** 200 * 25 = 5,000 HP

### Dexterity (DEX)

**Primary Functions:**
- Increases Ranged Damage (damage modifier for bows/crossbows)
- Increases Stamina (primary contribution)
- Increases Evasion/Dodge Chance
- Increases Movement Speed and Attack Speed

**Resource Contributions:**
- **Stamina**: Primary contribution (+25 Stamina per DEX point, with 10x multiplier)

**Attribute Cap:** Maximum 200 points
**Max Stamina Contribution:** 200 * 25 = 5,000 Stamina

### Intelligence (INT)

**Primary Functions:**
- Increases Magic/Elemental Damage (damage modifier)
- Increases Mana (primary contribution)
- Increases Magic Barrier (magical damage absorption)
- Increases Elemental Resistances (see RESISTANCES.md)

**Resource Contributions:**
- **Mana**: Primary contribution (+35 Mana per INT point, with 10x multiplier)

**Attribute Cap:** Maximum 200 points
**Max Mana Contribution:** 200 * 35 = 7,000 Mana

### Constitution (CON)

**Primary Functions:**
- Increases Health Points (HP) - primary contribution (more than STR)
- Increases Weight Capacity (more than STR)
- Increases Physical Resistance

**Resource Contributions:**
- **HP**: Primary contribution (+60 HP per CON point, with 10x multiplier)

**Attribute Cap:** Maximum 200 points
**Max HP Contribution:** 200 * 60 = 12,000 HP

### Luck (LCK)

**Primary Functions:**
- Increases Crafting Success chance
- Increases Rare Drop Rate
- Increases Hit Chance (accuracy)
- Increases Rare Collection chance

**Resource Contributions:**
- **No direct resource contributions** (affects crafting, drops, and combat accuracy)

### Charisma (CHA)

**Primary Functions:**
- Reduces Crafting Cost
- Reduces NPC Prices
- Reduces Taxes (trading, guild, etc.)

**Resource Contributions:**
- **No direct resource contributions** (affects economic systems)

## Resource Calculations

### Health Points (HP)

**Formula:**
```
Max HP = Base HP + (STR * STR_HP_Multiplier) + (CON * CON_HP_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max HP = 100 + (STR * 25) + (CON * 60) + Equipment Bonuses
```

**Components:**
- **Base HP**: 100 (starting HP for all entities, with 10x multiplier)
- **STR Contribution**: STR * 25 (moderate HP contribution, 2.5 multiplier × 10)
- **CON Contribution**: CON * 60 (primary HP contribution, 6.0 multiplier × 10)
- **Equipment Bonuses**: Flat HP bonuses from equipment

**Example Calculations:**
```
Level 1 Player (STR: 10, CON: 10):
HP = 100 + (10 * 25) + (10 * 60) = 950 HP

Level 50 Player (STR: 50, CON: 50):
HP = 100 + (50 * 25) + (50 * 60) = 4,350 HP

Tank Build (STR: 50, CON: 200):
HP = 100 + (50 * 25) + (200 * 60) = 13,350 HP

DPS Build (STR: 200, CON: 50):
HP = 100 + (200 * 25) + (50 * 60) = 5,600 HP

Balanced Build (STR: 100, CON: 100):
HP = 100 + (100 * 25) + (100 * 60) = 8,600 HP
```

**Balance Considerations:**
- HP must be sufficient to survive multiple hits in PvP
- With 50% damage cap, players need at least 2 hits to die
- Average HP should allow 3-5 hits before death for engaging combat
- Tank builds should survive significantly more hits

**Rust Implementation:**
```rust
pub fn calculate_max_hp(
    base_hp: u16,
    str: u16,
    con: u16,
    equipment_hp_bonus: u16,
) -> u16 {
    // Cap attributes at 200
    let str_capped = str.min(200);
    let con_capped = con.min(200);
    
    base_hp + (str_capped * 25) + (con_capped * 60) + equipment_hp_bonus
}
```

### Mana

**Formula:**
```
Max Mana = Base Mana + (INT * INT_Mana_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max Mana = 100 + (INT * 35) + Equipment Bonuses
```

**Components:**
- **Base Mana**: 100 (starting mana for all entities, with 10x multiplier)
- **INT Contribution**: INT * 35 (primary mana attribute, 3.5 multiplier × 10)
- **Equipment Bonuses**: Flat mana bonuses from equipment

**Example Calculations:**
```
Level 1 Player (INT: 10):
Mana = 100 + (10 * 35) = 450 Mana

Level 50 Player (INT: 50):
Mana = 100 + (50 * 35) = 1,850 Mana

Mage Build (INT: 200):
Mana = 100 + (200 * 35) = 7,100 Mana

Hybrid Build (INT: 100):
Mana = 100 + (100 * 35) = 3,600 Mana
```

**Balance Considerations:**
- Mana must be sufficient for multiple spell casts
- Spell costs typically range from 100-500 mana (multiplied by 10)
- Mages should be able to cast 5-10 spells before running out
- Mana regeneration helps maintain spell casting capability

**Rust Implementation:**
```rust
pub fn calculate_max_mana(
    base_mana: u16,
    int: u16,
    equipment_mana_bonus: u16,
) -> u16 {
    // Cap INT at 200
    let int_capped = int.min(200);
    
    base_mana + (int_capped * 35) + equipment_mana_bonus
}
```

### Stamina

**Formula:**
```
Max Stamina = Base Stamina + (DEX * DEX_Stamina_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max Stamina = 100 + (DEX * 25) + Equipment Bonuses
```

**Components:**
- **Base Stamina**: 100 (starting stamina for all entities, with 10x multiplier)
- **DEX Contribution**: DEX * 25 (primary stamina attribute, 2.5 multiplier × 10)
- **Equipment Bonuses**: Flat stamina bonuses from equipment

**Example Calculations:**
```
Level 1 Player (DEX: 10):
Stamina = 100 + (10 * 25) = 350 Stamina

Level 50 Player (DEX: 50):
Stamina = 100 + (50 * 25) = 1,350 Stamina

Rogue Build (DEX: 200):
Stamina = 100 + (200 * 25) = 5,100 Stamina

Warrior Build (DEX: 50):
Stamina = 100 + (50 * 25) = 1,350 Stamina
```

**Balance Considerations:**
- Stamina is consumed by sprinting, rolling, and some skills
- Sprint typically consumes 100 stamina per second (multiplied by 10)
- Skills consume 200-500 stamina per use (multiplied by 10)
- High stamina allows for extended mobility and skill usage

**Rust Implementation:**
```rust
pub fn calculate_max_stamina(
    base_stamina: u16,
    dex: u16,
    equipment_stamina_bonus: u16,
) -> u16 {
    // Cap DEX at 200
    let dex_capped = dex.min(200);
    
    base_stamina + (dex_capped * 25) + equipment_stamina_bonus
}
```

## Complete Status Calculation

### Entity Status Structure

**Rust Implementation:**
```rust
pub struct EntityStatus {
    pub str: u16,
    pub dex: u16,
    pub int: u16,
    pub con: u16,
    pub lck: u16,
    pub cha: u16,
    
    // Bonus attributes from equipment
    pub bonus_str: u16,
    pub bonus_dex: u16,
    pub bonus_int: u16,
    pub bonus_con: u16,
    pub bonus_lck: u16,
    pub bonus_cha: u16,
    
    // Calculated resources
    pub max_hp: u16,
    pub max_mana: u16,
    pub max_stamina: u16,
    
    // Current resources
    pub current_hp: u16,
    pub current_mana: u16,
    pub current_stamina: u16,
}

impl EntityStatus {
    pub fn calculate_resources(&mut self, equipment_bonuses: &EquipmentBonuses) {
        // Cap attributes at 200 (including bonuses)
        let total_str = (self.str + self.bonus_str).min(200);
        let total_dex = (self.dex + self.bonus_dex).min(200);
        let total_int = (self.int + self.bonus_int).min(200);
        let total_con = (self.con + self.bonus_con).min(200);
        
        // Calculate HP
        self.max_hp = 100 + (total_str * 25) + (total_con * 60) + equipment_bonuses.hp;
        
        // Calculate Mana
        self.max_mana = 100 + (total_int * 35) + equipment_bonuses.mana;
        
        // Calculate Stamina
        self.max_stamina = 100 + (total_dex * 25) + equipment_bonuses.stamina;
        
        // Ensure current resources don't exceed maximums
        self.current_hp = self.current_hp.min(self.max_hp);
        self.current_mana = self.current_mana.min(self.max_mana);
        self.current_stamina = self.current_stamina.min(self.max_stamina);
    }
}
```

## Balance with Damage System

### HP vs Damage Balance

**PvP Damage Cap:**
- Maximum damage per attack: 50% of target's max HP
- This means players need at least 2 hits to die (worst case)
- Average case: 3-5 hits before death for engaging combat

**HP Requirements by Level (with 200 cap):**
```
Level 1-20:  950-4,350 HP   → Can survive 2-4 hits
Level 21-40: 4,350-8,600 HP  → Can survive 3-5 hits
Level 41-60: 8,600-12,850 HP  → Can survive 4-6 hits
Level 61-80: 12,850-17,100 HP → Can survive 5-7 hits
Level 81-100: 17,100-21,350 HP → Can survive 6-8 hits

Max HP Build (CON: 200, STR: 200): ~17,100 HP base (without equipment)
Tank Build (CON: 200, STR: 50): ~13,350 HP base
DPS Build (STR: 200, CON: 50): ~5,600 HP base
```

**Damage Scaling (balanced for max HP ~18,000):**
- Equipment dice: 1D4 to 6D12 (1-72 base damage range, multiplied by 10)
- Attribute modifiers: ((STR/INT - 10) / 2) * 15 (balanced multiplier)
- Skill bonuses: (Skill Level * 2) * 15 (balanced multiplier)
- Equipment bonuses: +20 to +200 (multiplied by 10)
- **Balance Rationale**: Damage scales proportionally with HP maximum to prevent immortal builds

**Example Balance Check (with 200 cap and proportional damage):**
```
Endgame DPS Build (STR: 200):
- Attribute Modifier: ((200-10)/2)*15 = 1,425 (balanced for max HP)
- Max Damage: ~10,830 base (with equipment, skills, multipliers)
- After 70% Resistance: ~3,249 damage per hit
- Average Player HP: ~12,000
- Hits to Kill Average: 12,000 / 3,249 = ~3.7 hits (balanced!)

Tank Build (CON: 200):
- Max HP: ~13,350-18,000 (with equipment)
- Max PvP Damage Taken: ~3,249 per hit (after 70% resistance)
- Hits to Survive: 18,000 / 3,249 = ~5.5 hits (tanky but killable - not immortal!)
- Balance: Tank builds are tanky but can be killed in reasonable time
```

### Mana vs Spell Costs Balance

**Spell Cost Ranges:**
- Low-tier spells: 100-200 mana (multiplied by 10)
- Mid-tier spells: 200-400 mana (multiplied by 10)
- High-tier spells: 400-600 mana (multiplied by 10)

**Mana Pool Requirements (with 200 cap):**
```
Level 1 Mage (INT: 10):  450 Mana  → 2-4 low-tier spells
Level 50 Mage (INT: 50): 1,850 Mana → 4-8 mid-tier spells
Endgame Mage (INT: 200): 7,100 Mana → 14-18 high-tier spells
Hybrid Mage (INT: 100): 3,600 Mana → 7-12 mid-tier spells
```

**Balance Considerations:**
- Mages should cast 5-10 spells before running out
- Mana regeneration helps maintain casting capability
- Equipment can provide additional mana bonuses

### Stamina vs Action Costs Balance

**Stamina Cost Ranges:**
- Sprint: 100 stamina/second (multiplied by 10)
- Roll: 100 stamina per roll (multiplied by 10)
- Skills: 200-500 stamina per use (multiplied by 10)

**Stamina Pool Requirements (with 200 cap):**
```
Level 1 Player:  350 Stamina  → 35 seconds sprint, 1-3 skill uses
Level 50 Player: 1,350 Stamina → 135 seconds sprint, 2-5 skill uses
Rogue Build (DEX: 200): 5,100 Stamina → 510 seconds sprint, 10-25 skill uses
Balanced Build (DEX: 100): 2,600 Stamina → 260 seconds sprint, 5-13 skill uses
```

**Balance Considerations:**
- Players should sprint for 20-30 seconds before running out
- Multiple skill uses should be possible
- Stamina regeneration helps maintain mobility

## Attribute Distribution Examples

### Pure Tank Build

**Attributes:**
- STR: 50 (moderate damage + HP)
- DEX: 10 (minimal)
- INT: 10 (minimal)
- CON: 200 (max HP - capped)
- LCK: 30 (some hit chance)
- CHA: 10 (minimal)

**Resources:**
```
HP = 100 + (50 * 25) + (200 * 60) = 13,350 HP
Mana = 100 + (10 * 35) = 450 Mana
Stamina = 100 + (10 * 25) = 350 Stamina
```

**Characteristics:**
- Very high HP (survives many hits, ~2-3 hits from max damage)
- Low mana (minimal spell casting)
- Low stamina (limited mobility)

### Pure DPS Build

**Attributes:**
- STR: 200 (max damage - capped)
- DEX: 50 (moderate stamina)
- INT: 10 (minimal)
- CON: 50 (moderate HP)
- LCK: 30 (some hit chance)
- CHA: 10 (minimal)

**Resources:**
```
HP = 100 + (200 * 25) + (50 * 60) = 5,600 HP
Mana = 100 + (10 * 35) = 450 Mana
Stamina = 100 + (50 * 25) = 1,350 Stamina
```

**Characteristics:**
- Moderate HP (survives 2 hits, relies on damage output)
- Low mana (minimal spell casting)
- Moderate stamina (good mobility)
- High physical damage output

### Pure Mage Build

**Attributes:**
- STR: 20 (minimal)
- DEX: 30 (some attack speed)
- INT: 200 (max damage + mana - capped)
- CON: 120 (survivability)
- LCK: 50 (some hit chance)
- CHA: 30 (minimal)

**Resources:**
```
HP = 100 + (20 * 25) + (120 * 60) = 7,400 HP
Mana = 100 + (200 * 35) = 7,100 Mana
Stamina = 100 + (30 * 25) = 850 Stamina
```

**Characteristics:**
- Moderate HP (survives 2-3 hits, needs protection)
- Very high mana (many spell casts, ~14-18 high-tier spells)
- Low stamina (limited mobility)
- High magic damage output

### Hybrid Build

**Attributes:**
- STR: 75 (moderate damage + HP)
- DEX: 75 (moderate stamina)
- INT: 75 (moderate damage + mana)
- CON: 75 (moderate HP)
- LCK: 50 (moderate hit chance)
- CHA: 50 (moderate cost reduction)

**Resources:**
```
HP = 100 + (75 * 25) + (75 * 60) = 6,475 HP
Mana = 100 + (75 * 35) = 2,725 Mana
Stamina = 100 + (75 * 25) = 1,975 Stamina
```

**Characteristics:**
- Balanced HP (survives 2-3 hits)
- Moderate mana (some spell casting)
- Good stamina (good mobility)
- Versatile build

### Rogue Build

**Attributes:**
- STR: 30 (minimal)
- DEX: 200 (max stamina + evasion - capped)
- INT: 20 (minimal)
- CON: 30 (minimal)
- LCK: 150 (high hit chance, rare drops)
- CHA: 20 (minimal)

**Resources:**
```
HP = 100 + (30 * 25) + (30 * 60) = 3,250 HP
Mana = 100 + (20 * 35) = 800 Mana
Stamina = 100 + (200 * 25) = 5,100 Stamina
```

**Characteristics:**
- Low HP (survives 1-2 hits, relies heavily on evasion)
- Low mana (minimal spell casting)
- Very high stamina (excellent mobility, ~510 seconds sprint)
- High evasion and ranged damage

## Resource Regeneration

### HP Regeneration

**Formula:**
```
HP Regen per Tick = Base Regen + (Life_Regeneration / 10)
```

**Current Implementation:**
```
HP Regen per Tick = max(10, 10 + (Life_Regeneration / 10))
Tick Interval: 3 seconds
```

**Balance Considerations:**
- Slow regeneration prevents infinite sustain
- Encourages strategic use of healing items/skills
- Out-of-combat regeneration helps recovery

### Mana Regeneration

**Formula:**
```
Mana Regen per Tick = Base Regen + (Mana_Regeneration / 10)
```

**Current Implementation:**
```
Mana Regen per Tick = max(10, 30 + (Mana_Regeneration / 10))
Tick Interval: 3 seconds
Heavy Armor Penalty: Regen / 2 (if wearing medium/heavy armor)
```

**Balance Considerations:**
- Faster than HP regen (mages need mana for combat)
- Heavy armor penalty encourages light armor for mages
- Allows sustained spell casting over time

### Stamina Regeneration

**Formula:**
```
Stamina Regen per Tick = Base Regen + (Stamina_Regeneration / 10)
```

**Current Implementation:**
```
Stamina Regen per Tick = max(10, 100 + (Stamina_Regeneration / 10))
Tick Interval: 3 seconds
Sprint Penalty: -100 stamina per tick (while sprinting)
Heavy Armor Penalty: Regen * 2 (if not wearing heavy armor)
```

**Balance Considerations:**
- Fastest regeneration (needed for mobility)
- Sprint consumes stamina quickly
- Heavy armor reduces stamina regen significantly

## Rust Implementation Considerations

### Data Structures

**Status Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct Status {
    pub str: u16,   // Max 200
    pub dex: u16,   // Max 200
    pub int: u16,   // Max 200
    pub con: u16,   // Max 200
    pub lck: u16,   // Max 200
    pub cha: u16,   // Max 200
}

#[derive(Component, Clone, Copy)]
pub struct StatusBonuses {
    pub str: u16,   // Max 200 (total with base)
    pub dex: u16,   // Max 200 (total with base)
    pub int: u16,   // Max 200 (total with base)
    pub con: u16,   // Max 200 (total with base)
    pub lck: u16,   // Max 200 (total with base)
    pub cha: u16,   // Max 200 (total with base)
}

#[derive(Component, Clone, Copy)]
pub struct Resources {
    pub max_hp: u16,
    pub current_hp: u16,
    pub max_mana: u16,
    pub current_mana: u16,
    pub max_stamina: u16,
    pub current_stamina: u16,
}
```

### Performance Optimizations

**Caching:**
- Cache calculated resources until attributes change
- Invalidate cache when equipment changes
- Pre-calculate resource pools on attribute allocation

**Batch Processing:**
- Calculate resources for multiple entities in parallel
- Use SIMD for attribute calculations when possible
- Pre-calculate regeneration amounts

**Memory Layout:**
- Store status in Structure of Arrays (SoA) format for cache efficiency
- Align status data for SIMD operations
- Use compact representations (u16 for attributes if sufficient)

### Serialization

**Status Serialization:**
```rust
impl Serialize for Status {
    fn serialize(&self) -> Vec<u8> {
        let mut buffer = Vec::with_capacity(12); // 6 * 2 bytes
        
        buffer.extend_from_slice(&self.str.to_le_bytes());
        buffer.extend_from_slice(&self.dex.to_le_bytes());
        buffer.extend_from_slice(&self.int.to_le_bytes());
        buffer.extend_from_slice(&self.con.to_le_bytes());
        buffer.extend_from_slice(&self.lck.to_le_bytes());
        buffer.extend_from_slice(&self.cha.to_le_bytes());
        
        buffer
    }
}
```

## Integration with Other Systems

### Damage System

**Integration:**
- STR affects physical damage modifier
- INT affects magical damage modifier
- DEX affects ranged damage modifier
- See [DAMAGE.md](./DAMAGE.md) for damage calculations

### Resistance System

**Integration:**
- CON (Constitution) affects physical resistance
- STR affects physical resistance
- INT affects elemental resistances
- See [RESISTANCES.md](./RESISTANCES.md) for resistance calculations

### Equipment System

**Integration:**
- Equipment can provide attribute bonuses
- Equipment can provide direct resource bonuses
- Equipment changes trigger resource recalculation

### Player System

**Integration:**
- Players gain 5 stat points per level
- Stat points can be allocated to any attribute
- Stat cap: 225 total points (configurable)

## Testing Considerations

### Unit Tests

**Test Cases:**
- Resource calculation with various attribute values
- Resource calculation with equipment bonuses
- Resource regeneration calculations
- Resource cap enforcement (current <= max)

### Integration Tests

**Test Cases:**
- Complete resource calculation flow
- Resource recalculation on attribute change
- Resource recalculation on equipment change
- Resource regeneration over time

### Balance Tests

**Test Cases:**
- HP vs damage balance (survive multiple hits)
- Mana vs spell costs balance (cast multiple spells)
- Stamina vs action costs balance (sprint/skills)
- Build diversity (different attribute distributions)

### Performance Tests

**Test Cases:**
- Resource calculation performance (target: <1μs per entity)
- Batch resource calculation performance
- Memory usage for status data structures

## Attribute Cap System

### Cap Rules

**Maximum Attribute Points:**
- Each attribute can have a maximum of **200 points** (including base stats and bonuses)
- This cap applies to the total value (base + equipment bonuses)
- Prevents extreme min-maxing while maintaining build diversity

**Enforcement:**
- Attribute allocation is capped at 200 during stat point distribution
- Equipment bonuses are included in the cap calculation
- Total attribute value (base + bonuses) cannot exceed 200

**Balance Rationale:**
- Prevents builds from becoming too specialized
- Ensures PvP remains balanced and engaging
- Maintains meaningful choices in attribute distribution
- Allows for diverse viable builds

### Cap Examples

**Pure DPS Build:**
- STR: 200 (capped) → Max physical damage modifier
- CON: 50 → Moderate HP
- Result: High damage, moderate survivability

**Pure Tank Build:**
- CON: 200 (capped) → Max HP
- STR: 50 → Moderate damage
- Result: High survivability, moderate damage

**Pure Mage Build:**
- INT: 200 (capped) → Max magic damage and mana
- CON: 120 → Moderate HP
- Result: High magic damage, high mana pool

**Rogue Build:**
- DEX: 200 (capped) → Max stamina and ranged damage
- CON: 30 → Low HP
- Result: High mobility, high stamina, relies on evasion

## Summary

The status system:

- Calculates HP, Mana, and Stamina based on core attributes (STR, DEX, INT, CON, LCK, CHA)
- Uses linear scaling to maintain predictable growth
- Enforces a 200-point cap per attribute to ensure balanced builds
- Balances resources against damage system for fair PvP
- Supports diverse builds through different attribute distributions
- Integrates with damage, resistance, equipment, and player systems

**Key Balance Points (with 200 cap):**
- Max HP Build: ~13,350-18,000 HP (CON: 200)
- Max DPS Build: ~5,600 HP, high damage (STR: 200)
- Max Mage Build: ~7,100 Mana, high magic damage (INT: 200)
- Max Rogue Build: ~5,100 Stamina, high mobility (DEX: 200)

This system creates balanced resource pools that work harmoniously with the damage system, ensuring engaging combat where players have sufficient resources to participate in extended encounters while maintaining strategic resource management and preventing extreme min-maxing.

