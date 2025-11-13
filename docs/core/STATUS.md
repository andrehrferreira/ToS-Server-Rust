# Status System Documentation

## Overview

The status system defines how core attributes (STR, DEX, INT) affect an entity's primary resources: Health Points (HP), Mana, and Stamina. These calculations are balanced against the damage system to ensure fair and engaging combat. The system uses linear scaling to maintain predictable growth and prevent exponential power creep, making PvP balance achievable.

**Note:** All resource values are multiplied by 10 to provide a greater sense of progression. This means HP, Mana, and Stamina calculations result in larger numbers (e.g., 3,100 HP instead of 310) while maintaining the same balance ratios with the damage system.

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
- Increases Health Points (HP)
- Increases Physical Resistance (see RESISTANCES.md)
- Increases Stamina (secondary contribution)

**Resource Contributions:**
- **HP**: Direct contribution (+1 HP per STR point)
- **Stamina**: Secondary contribution (+2 Stamina per STR point)

### Dexterity (DEX)

**Primary Functions:**
- Increases Ranged Damage (damage modifier for bows/crossbows)
- Increases Stamina (primary contribution)
- Increases Evasion/Dodge Chance
- Increases Movement Speed and Attack Speed

**Resource Contributions:**
- **Stamina**: Primary contribution (+3 Stamina per DEX point)

### Intelligence (INT)

**Primary Functions:**
- Increases Magic/Elemental Damage (damage modifier)
- Increases Mana (primary contribution)
- Increases Elemental Resistances (see RESISTANCES.md)

**Resource Contributions:**
- **Mana**: Primary contribution (+3 Mana per INT point)

## Resource Calculations

### Health Points (HP)

**Formula:**
```
Max HP = Base HP + (VIG * VIG_HP_Multiplier) + (STR * STR_HP_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max HP = 100 + (VIG * 50) + (STR * 10) + Equipment Bonuses
```

**Components:**
- **Base HP**: 100 (starting HP for all entities)
- **VIG Contribution**: VIG * 50 (primary HP attribute)
- **STR Contribution**: STR * 10 (secondary HP contribution)
- **Equipment Bonuses**: Flat HP bonuses from equipment

**Example Calculations:**
```
Level 1 Player (STR: 10, VIG: 10):
HP = 100 + (10 * 50) + (10 * 10) = 700 HP

Level 50 Player (STR: 50, VIG: 50):
HP = 100 + (50 * 50) + (50 * 10) = 3,100 HP

Tank Build (STR: 100, VIG: 100):
HP = 100 + (100 * 50) + (100 * 10) = 6,100 HP

DPS Build (STR: 150, VIG: 50):
HP = 100 + (50 * 50) + (150 * 10) = 4,100 HP
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
    vig: u16,
    str: u16,
    equipment_hp_bonus: u16,
) -> u16 {
    base_hp + (vig * 50) + (str * 10) + equipment_hp_bonus
}
```

### Mana

**Formula:**
```
Max Mana = Base Mana + (INT * INT_Mana_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max Mana = 100 + (INT * 30) + Equipment Bonuses
```

**Components:**
- **Base Mana**: 100 (starting mana for all entities)
- **INT Contribution**: INT * 30 (primary mana attribute)
- **Equipment Bonuses**: Flat mana bonuses from equipment

**Example Calculations:**
```
Level 1 Player (INT: 10):
Mana = 100 + (10 * 30) = 400 Mana

Level 50 Player (INT: 50):
Mana = 100 + (50 * 30) = 1,600 Mana

Mage Build (INT: 150):
Mana = 100 + (150 * 30) = 4,600 Mana

Hybrid Build (INT: 75):
Mana = 100 + (75 * 30) = 2,350 Mana
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
    base_mana + (int * 30) + equipment_mana_bonus
}
```

### Stamina

**Formula:**
```
Max Stamina = Base Stamina + (STR * STR_Stamina_Multiplier) + (DEX * DEX_Stamina_Multiplier) + (VIG * VIG_Stamina_Multiplier) + Equipment Bonuses
```

**Current Implementation:**
```
Max Stamina = 100 + (STR * 20) + (DEX * 30) + (VIG * 10) + Equipment Bonuses
```

**Components:**
- **Base Stamina**: 100 (starting stamina for all entities)
- **STR Contribution**: STR * 20 (secondary stamina attribute)
- **DEX Contribution**: DEX * 30 (primary stamina attribute)
- **VIG Contribution**: VIG * 10 (tertiary stamina contribution)
- **Equipment Bonuses**: Flat stamina bonuses from equipment

**Example Calculations:**
```
Level 1 Player (STR: 10, DEX: 10, VIG: 10):
Stamina = 100 + (10 * 20) + (10 * 30) + (10 * 10) = 700 Stamina

Level 50 Player (STR: 50, DEX: 50, VIG: 50):
Stamina = 100 + (50 * 20) + (50 * 30) + (50 * 10) = 3,100 Stamina

Rogue Build (STR: 50, DEX: 150, VIG: 50):
Stamina = 100 + (50 * 20) + (150 * 30) + (50 * 10) = 5,600 Stamina

Warrior Build (STR: 150, DEX: 50, VIG: 50):
Stamina = 100 + (150 * 20) + (50 * 30) + (50 * 10) = 4,100 Stamina
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
    str: u16,
    dex: u16,
    vig: u16,
    equipment_stamina_bonus: u16,
) -> u16 {
    base_stamina + (str * 20) + (dex * 30) + (vig * 10) + equipment_stamina_bonus
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
    pub vig: u16,
    pub agi: u16,
    pub luc: u16,
    
    // Bonus attributes from equipment
    pub bonus_str: u16,
    pub bonus_dex: u16,
    pub bonus_int: u16,
    pub bonus_vig: u16,
    pub bonus_agi: u16,
    pub bonus_luc: u16,
    
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
        let total_vig = self.vig + self.bonus_vig;
        let total_str = self.str + self.bonus_str;
        let total_dex = self.dex + self.bonus_dex;
        let total_int = self.int + self.bonus_int;
        
        // Calculate HP
        self.max_hp = 100 + (total_vig * 50) + (total_str * 10) + equipment_bonuses.hp;
        
        // Calculate Mana
        self.max_mana = 100 + (total_int * 30) + equipment_bonuses.mana;
        
        // Calculate Stamina
        self.max_stamina = 100 
            + (total_str * 20) 
            + (total_dex * 30) 
            + (total_vig * 10) 
            + equipment_bonuses.stamina;
        
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

**HP Requirements by Level:**
```
Level 1-20:  700-3,100 HP   → Can survive 2-4 hits
Level 21-40: 3,100-6,100 HP  → Can survive 3-5 hits
Level 41-60: 6,100-9,100 HP  → Can survive 4-6 hits
Level 61-80: 9,100-12,100 HP → Can survive 5-7 hits
Level 81-100: 12,100-15,100 HP → Can survive 6-8 hits
```

**Damage Scaling:**
- Equipment dice: 1D4 to 6D12 (1-72 base damage range)
- Attribute modifiers: (STR/INT - 10) / 2
- Skill bonuses: Skill Level * 2
- Equipment bonuses: +20 to +100

**Example Balance Check:**
```
Endgame Build:
- Max Damage: ~7,800 (from DAMAGE.md)
- Average Player HP: ~12,000
- Max PvP Damage: 6,000 (50% cap)
- Hits to Kill: 12,000 / 6,000 = 2 hits (worst case)
- Average Hits: 12,000 / 4,000 = 3 hits (more realistic)
```

### Mana vs Spell Costs Balance

**Spell Cost Ranges:**
- Low-tier spells: 100-200 mana (multiplied by 10)
- Mid-tier spells: 200-400 mana (multiplied by 10)
- High-tier spells: 400-600 mana (multiplied by 10)

**Mana Pool Requirements:**
```
Level 1 Mage (INT: 10):  400 Mana  → 2-4 low-tier spells
Level 50 Mage (INT: 50): 1,600 Mana → 4-8 mid-tier spells
Endgame Mage (INT: 150): 4,600 Mana → 8-15 high-tier spells
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

**Stamina Pool Requirements:**
```
Level 1 Player:  700 Stamina  → 70 seconds sprint, 3-7 skill uses
Level 50 Player: 3,100 Stamina → 310 seconds sprint, 6-15 skill uses
Rogue Build:     5,600 Stamina → 560 seconds sprint, 11-28 skill uses
```

**Balance Considerations:**
- Players should sprint for 20-30 seconds before running out
- Multiple skill uses should be possible
- Stamina regeneration helps maintain mobility

## Attribute Distribution Examples

### Pure Tank Build

**Attributes:**
- STR: 100 (damage + HP)
- DEX: 10 (minimal)
- INT: 10 (minimal)
- VIG: 100 (primary HP)

**Resources:**
```
HP = 100 + (100 * 50) + (100 * 10) = 6,100 HP
Mana = 100 + (10 * 30) = 400 Mana
Stamina = 100 + (100 * 20) + (10 * 30) + (100 * 10) = 3,400 Stamina
```

**Characteristics:**
- Very high HP (survives many hits)
- Low mana (minimal spell casting)
- Moderate stamina (good mobility)

### Pure DPS Build

**Attributes:**
- STR: 150 (max damage)
- DEX: 50 (moderate stamina)
- INT: 10 (minimal)
- VIG: 50 (moderate HP)

**Resources:**
```
HP = 100 + (50 * 50) + (150 * 10) = 4,100 HP
Mana = 100 + (10 * 30) = 400 Mana
Stamina = 100 + (150 * 20) + (50 * 30) + (50 * 10) = 4,100 Stamina
```

**Characteristics:**
- Moderate HP (survives 2-3 hits)
- Low mana (minimal spell casting)
- High stamina (good mobility)

### Pure Mage Build

**Attributes:**
- STR: 10 (minimal)
- DEX: 50 (moderate stamina)
- INT: 150 (max damage + mana)
- VIG: 50 (moderate HP)

**Resources:**
```
HP = 100 + (50 * 50) + (10 * 10) = 2,700 HP
Mana = 100 + (150 * 30) = 4,600 Mana
Stamina = 100 + (10 * 20) + (50 * 30) + (50 * 10) = 2,200 Stamina
```

**Characteristics:**
- Low HP (survives 1-2 hits, needs protection)
- Very high mana (many spell casts)
- Low stamina (limited mobility)

### Hybrid Build

**Attributes:**
- STR: 75 (moderate damage + HP)
- DEX: 75 (moderate stamina)
- INT: 75 (moderate damage + mana)
- VIG: 75 (moderate HP)

**Resources:**
```
HP = 100 + (75 * 50) + (75 * 10) = 4,600 HP
Mana = 100 + (75 * 30) = 2,350 Mana
Stamina = 100 + (75 * 20) + (75 * 30) + (75 * 10) = 4,600 Stamina
```

**Characteristics:**
- Balanced HP (survives 2-3 hits)
- Moderate mana (some spell casting)
- High stamina (good mobility)

### Rogue Build

**Attributes:**
- STR: 50 (moderate damage)
- DEX: 150 (max stamina + evasion)
- INT: 10 (minimal)
- VIG: 50 (moderate HP)

**Resources:**
```
HP = 100 + (50 * 50) + (50 * 10) = 3,100 HP
Mana = 100 + (10 * 30) = 400 Mana
Stamina = 100 + (50 * 20) + (150 * 30) + (50 * 10) = 5,600 Stamina
```

**Characteristics:**
- Moderate HP (survives 2 hits, relies on evasion)
- Low mana (minimal spell casting)
- Very high stamina (excellent mobility)

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
    pub str: u16,
    pub dex: u16,
    pub int: u16,
    pub vig: u16,
    pub agi: u16,
    pub luc: u16,
}

#[derive(Component, Clone, Copy)]
pub struct StatusBonuses {
    pub str: u16,
    pub dex: u16,
    pub int: u16,
    pub vig: u16,
    pub agi: u16,
    pub luc: u16,
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
        buffer.extend_from_slice(&self.vig.to_le_bytes());
        buffer.extend_from_slice(&self.agi.to_le_bytes());
        buffer.extend_from_slice(&self.luc.to_le_bytes());
        
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
- VIG (Constitution) affects physical resistance
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

## Summary

The status system:

- Calculates HP, Mana, and Stamina based on core attributes (STR, DEX, INT, VIG)
- Uses linear scaling to maintain predictable growth
- Balances resources against damage system for fair PvP
- Supports diverse builds through different attribute distributions
- Integrates with damage, resistance, equipment, and player systems

This system creates balanced resource pools that work harmoniously with the damage system, ensuring engaging combat where players have sufficient resources to participate in extended encounters while maintaining strategic resource management.

