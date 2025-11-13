# Resistance System Documentation

## Overview

The resistance system is a core combat mechanic that determines how entities receive and mitigate damage. Every entity possesses both physical and elemental resistances that directly affect damage calculations. Resistances can be increased through attributes, equipment, skills, and enchantments, while zones can reduce resistances to encourage equipment upgrades. The system includes a maximum resistance cap, immunity mechanics for creatures, and a dynamic skill-based resistance system that adapts to combat situations.

## Key Features

- **Universal System**: All entities have physical and elemental resistances
- **Attribute-Based**: Constitution increases physical resistance, Intelligence increases elemental resistances
- **Equipment Modifiers**: Items can increase or decrease resistances
- **Zone Effects**: Map zones can reduce resistances to force equipment upgrades
- **Damage Types**: Physical and elemental damage types (Fire, Cold, Poison, Energy, Light, Dark)
- **Dynamic Skill**: Magic Resistance skill increases with elemental damage taken
- **Resistance Cap**: Maximum 70% resistance for all entities
- **Creature Immunities**: Creatures can be immune to specific damage types
- **Negative Resistance**: Negative resistance values increase damage taken
- **Damage Calculation**: Damage is directly calculated based on resistance values

## Resistance Types

### Physical Resistance

**Description:**
Physical resistance reduces damage from physical attacks, including melee weapons, ranged weapons (bows, crossbows), and physical skills.

**Sources:**
- Base resistance (entity-specific)
- Constitution attribute
- Equipment bonuses
- Enchantments (see separate enchantment documentation)
- Zone modifiers

**Formula:**
```
Physical Resistance = Base Resistance + (CON * CON_Physical_Resistance_Multiplier) + Equipment Bonuses + Enchantments + Zone Modifiers
```

**Example:**
```rust
// Player with 100 CON, +10 from equipment, +5 from enchantment, -5 from zone
physical_resistance = 0 + (100 * 0.8) + 10 + 5 - 5 = 90
```

### Elemental Resistances

**Description:**
Elemental resistances reduce damage from magical and elemental attacks. There are six elemental damage types, each with its own resistance value.

**Elemental Types:**
1. **Fire Resistance** - Reduces fire damage
2. **Cold Resistance** - Reduces cold/ice damage
3. **Poison Resistance** - Reduces poison/toxin damage
4. **Energy Resistance** - Reduces energy/lightning damage
5. **Light Resistance** - Reduces light/holy damage
6. **Dark Resistance** - Reduces dark/shadow damage

**Sources:**
- Base resistance (entity-specific)
- Intelligence attribute
- Equipment bonuses
- Enchantments (see separate enchantment documentation)
- Zone modifiers
- Magic Resistance skill (up to 25% of total)

**Formula:**
```
Elemental Resistance = Base Resistance + (INT * INT_Elemental_Resistance_Multiplier) + Equipment Bonuses + Enchantments + Zone Modifiers + Magic Resistance Skill Bonus
```

**Example:**
```rust
// Player with 150 INT, +15 Fire from equipment, +8 from enchantment, -10 from zone, +5 from Magic Resistance skill
fire_resistance = 0 + (150 * 0.5) + 15 + 8 - 10 + 5 = 93
```

## Attribute Effects

### Constitution (CON)

**Effect:**
Increases physical resistance for all entities.

**Multiplier:**
- CON Physical Resistance Multiplier: 0.8 per point of CON

**Implementation:**
```rust
pub fn calculate_physical_resistance(&self) -> f32 {
    let base = self.base_physical_resistance;
    let con_bonus = self.constitution as f32 * 0.8;
    let equipment_bonus = self.equipment_physical_resistance;
    let enchantment_bonus = self.enchantment_physical_resistance;
    let zone_modifier = self.current_zone.physical_resistance_modifier;
    
    base + con_bonus + equipment_bonus + enchantment_bonus + zone_modifier
}
```

### Intelligence (INT)

**Effect:**
Increases all elemental resistances (Fire, Cold, Poison, Energy, Light, Dark).

**Multiplier:**
- INT Elemental Resistance Multiplier: 0.5 per point of INT

**Implementation:**
```rust
pub fn calculate_elemental_resistance(&self, element: ElementType) -> f32 {
    let base = self.base_elemental_resistance(element);
    let int_bonus = self.intelligence as f32 * 0.5;
    let equipment_bonus = self.equipment_elemental_resistance(element);
    let enchantment_bonus = self.enchantment_elemental_resistance(element);
    let zone_modifier = self.current_zone.elemental_resistance_modifier(element);
    let magic_resistance_bonus = self.magic_resistance_skill_bonus(element);
    
    base + int_bonus + equipment_bonus + enchantment_bonus + zone_modifier + magic_resistance_bonus
}
```

## Equipment Modifiers

### Equipment Resistance Bonuses

**Description:**
Equipment items can provide resistance bonuses or penalties. These modifiers are applied directly to the entity's total resistance.

**Positive Modifiers:**
- Armor pieces can provide physical resistance
- Armor pieces can provide elemental resistances
- Weapons can provide resistance bonuses (less common)
- Accessories can provide resistance bonuses

**Negative Modifiers:**
- Cursed items can reduce resistances
- Fragile items can reduce physical resistance
- Vulnerable items can reduce elemental resistances

**Implementation:**
```rust
pub struct EquipmentResistances {
    pub physical: f32,
    pub fire: f32,
    pub cold: f32,
    pub poison: f32,
    pub energy: f32,
    pub light: f32,
    pub dark: f32,
}

impl Entity {
    pub fn calculate_equipment_resistances(&self) -> EquipmentResistances {
        let mut total = EquipmentResistances::default();
        
        for item in &self.equipped_items {
            total.physical += item.physical_resistance;
            total.fire += item.fire_resistance;
            total.cold += item.cold_resistance;
            total.poison += item.poison_resistance;
            total.energy += item.energy_resistance;
            total.light += item.light_resistance;
            total.dark += item.dark_resistance;
        }
        
        total
    }
}
```

### Weapon Damage Type Conversion

**Description:**
Weapons determine the damage type of attacks. An archer, for example, deals physical damage by default, but the equipped bow can change the damage type to elemental.

**Examples:**
- **Physical Bow**: Deals physical damage
- **Fire Bow**: Deals fire damage (elemental)
- **Ice Bow**: Deals cold damage (elemental)
- **Poison Bow**: Deals poison damage (elemental)

**Implementation:**
```rust
pub enum DamageType {
    Physical,
    Fire,
    Cold,
    Poison,
    Energy,
    Light,
    Dark,
}

impl Weapon {
    pub fn get_damage_type(&self) -> DamageType {
        // Weapon's inherent damage type or enchantment override
        self.damage_type
    }
}
```

## Zone Effects

### Zone Resistance Modifiers

**Description:**
Map zones can apply resistance modifiers to entities within them. This mechanic is used to encourage players to upgrade their equipment before entering more dangerous areas.

**Purpose:**
- Force equipment upgrades
- Create difficulty progression
- Encourage strategic preparation
- Balance end-game content

**Implementation:**
```rust
pub struct ZoneResistanceModifiers {
    pub physical_modifier: f32,  // Can be negative
    pub fire_modifier: f32,
    pub cold_modifier: f32,
    pub poison_modifier: f32,
    pub energy_modifier: f32,
    pub light_modifier: f32,
    pub dark_modifier: f32,
}

impl Zone {
    pub fn apply_resistance_modifiers(&self, entity: &mut Entity) {
        entity.zone_physical_modifier = self.resistance_modifiers.physical_modifier;
        entity.zone_fire_modifier = self.resistance_modifiers.fire_modifier;
        // ... other elements
    }
}
```

**Example Zones:**
- **Normal Zone**: No modifiers (0)
- **Cursed Forest**: -10% to all resistances
- **Volcanic Area**: -20% Fire resistance, -10% Physical resistance
- **Frozen Wasteland**: -20% Cold resistance, -15% Physical resistance
- **Toxic Swamp**: -20% Poison resistance, -10% all resistances

## Magic Resistance Skill

### Skill Mechanics

**Description:**
Magic Resistance is a dynamic skill that increases as the player takes elemental damage. This creates an adaptive defense system where players become more resistant to damage types they encounter frequently.

**Skill Name:** Magic Resistance (MagicResistence)

**Progression:**
- Skill increases when player receives elemental damage
- Skill level determines bonus resistance
- Maximum contribution: 25% of total resistance
- Skill resets on death or logout (if configured)

**Formula:**
```
Magic Resistance Skill Bonus = min(Skill Level * Skill_Multiplier, Total_Resistance * 0.25)
```

**Implementation:**
```rust
impl Entity {
    pub fn on_elemental_damage_taken(&mut self, damage_type: ElementType, damage: f32) {
        // Increase Magic Resistance skill experience
        self.skills.add_experience(SkillType::MagicResistance, damage);
        
        // Update skill bonus
        self.update_magic_resistance_bonus();
    }
    
    pub fn get_magic_resistance_bonus(&self, element: ElementType) -> f32 {
        let skill_level = self.skills.get_level(SkillType::MagicResistance);
        let base_resistance = self.calculate_base_elemental_resistance(element);
        let total_resistance = self.calculate_total_elemental_resistance(element);
        
        // Skill bonus is limited to 25% of total resistance
        let skill_bonus = skill_level as f32 * 0.5; // Example multiplier
        let max_bonus = total_resistance * 0.25;
        
        skill_bonus.min(max_bonus)
    }
}
```

**Skill Cap:**
- Maximum skill level: 100
- Maximum contribution: 25% of total resistance
- Example: If total resistance is 100, Magic Resistance can contribute up to 25 points

## Resistance Limits

### Maximum Resistance Cap

**Description:**
All entities have a maximum resistance cap of 70%. This prevents invincibility and ensures combat remains challenging even with maximum gear.

**Cap:**
- Maximum Physical Resistance: 70%
- Maximum Elemental Resistance: 70% (per element)

**Implementation:**
```rust
impl Entity {
    pub fn calculate_final_resistance(&self, base_resistance: f32) -> f32 {
        let total = self.calculate_total_resistance();
        total.min(70.0).max(-100.0) // Cap at 70%, minimum at -100%
    }
}
```

**Examples:**
- Entity with 100 total resistance → Final: 70% (capped)
- Entity with 50 total resistance → Final: 50% (no cap)
- Entity with -20 total resistance → Final: -20% (takes 20% more damage)

## Creature Immunities

### Immunity System

**Description:**
Creatures can be immune to specific damage types. When a creature is immune to a damage type, that damage type deals 0 damage regardless of the attack's base damage.

**Implementation:**
```rust
pub struct CreatureImmunities {
    pub physical: bool,
    pub fire: bool,
    pub cold: bool,
    pub poison: bool,
    pub energy: bool,
    pub light: bool,
    pub dark: bool,
}

impl Creature {
    pub fn is_immune_to(&self, damage_type: DamageType) -> bool {
        match damage_type {
            DamageType::Physical => self.immunities.physical,
            DamageType::Fire => self.immunities.fire,
            DamageType::Cold => self.immunities.cold,
            DamageType::Poison => self.immunities.poison,
            DamageType::Energy => self.immunities.energy,
            DamageType::Light => self.immunities.light,
            DamageType::Dark => self.immunities.dark,
        }
    }
}
```

**Examples:**
- **Fire Elemental**: Immune to Fire damage
- **Ice Golem**: Immune to Cold damage
- **Shadow Demon**: Immune to Dark damage
- **Undead Skeleton**: Immune to Poison damage
- **Light Guardian**: Immune to Light damage

## Damage Calculation

### Resistance-Based Damage Formula

**Description:**
Damage calculation is directly related to resistance values. Positive resistance reduces damage, while negative resistance increases damage taken.

**Formula:**
```
Final Damage = Base Damage * (1 - (Resistance / 100))
```

**Detailed Formula:**
```
if Resistance >= 0:
    Damage Multiplier = 1 - (Resistance / 100)
    Final Damage = Base Damage * Damage Multiplier
else:
    Damage Multiplier = 1 + (|Resistance| / 100)
    Final Damage = Base Damage * Damage Multiplier
```

**Implementation:**
```rust
pub fn calculate_damage(base_damage: f32, resistance: f32) -> f32 {
    // Check for immunity first
    if resistance >= 100.0 {
        return 0.0;
    }
    
    // Cap resistance at 70%
    let capped_resistance = resistance.min(70.0);
    
    if capped_resistance >= 0.0 {
        // Positive resistance reduces damage
        let multiplier = 1.0 - (capped_resistance / 100.0);
        base_damage * multiplier
    } else {
        // Negative resistance increases damage
        let multiplier = 1.0 + (capped_resistance.abs() / 100.0);
        base_damage * multiplier
    }
}
```

### Damage Calculation Examples

**Example 1: Positive Resistance**
```
Base Damage: 1000
Resistance: 50%
Final Damage = 1000 * (1 - 0.50) = 500
```

**Example 2: Maximum Resistance (Capped)**
```
Base Damage: 1000
Resistance: 90% (capped at 70%)
Final Damage = 1000 * (1 - 0.70) = 300
```

**Example 3: Negative Resistance**
```
Base Damage: 1000
Resistance: -25%
Final Damage = 1000 * (1 + 0.25) = 1250
```

**Example 4: Zero Resistance**
```
Base Damage: 1000
Resistance: 0%
Final Damage = 1000 * (1 - 0.00) = 1000
```

**Example 5: Immunity**
```
Base Damage: 1000
Resistance: 100% (or immunity flag)
Final Damage = 0
```

### Complete Damage Calculation Flow

**Step-by-Step Process:**
1. Check for immunity (if immune, return 0 damage)
2. Calculate total resistance (base + attributes + equipment + enchantments + zone + skills)
3. Apply resistance cap (70% maximum)
4. Calculate damage multiplier based on resistance
5. Apply multiplier to base damage
6. Return final damage

**Implementation:**
```rust
pub fn calculate_final_damage(
    attacker: &Entity,
    defender: &Entity,
    base_damage: f32,
    damage_type: DamageType,
) -> f32 {
    // Step 1: Check immunity
    if defender.is_immune_to(damage_type) {
        return 0.0;
    }
    
    // Step 2: Calculate total resistance
    let total_resistance = match damage_type {
        DamageType::Physical => defender.calculate_physical_resistance(),
        DamageType::Fire => defender.calculate_fire_resistance(),
        DamageType::Cold => defender.calculate_cold_resistance(),
        DamageType::Poison => defender.calculate_poison_resistance(),
        DamageType::Energy => defender.calculate_energy_resistance(),
        DamageType::Light => defender.calculate_light_resistance(),
        DamageType::Dark => defender.calculate_dark_resistance(),
    };
    
    // Step 3: Apply cap
    let capped_resistance = total_resistance.min(70.0);
    
    // Step 4 & 5: Calculate final damage
    calculate_damage(base_damage, capped_resistance)
}
```

## Enchantments

### Enchantment System

**Description:**
Enchantments can be applied to weapons and armor to modify resistances and damage types. This system is documented separately but integrates directly with the resistance system.

**Integration Points:**
- Enchantments can add resistance bonuses
- Enchantments can convert weapon damage types
- Enchantments can add resistance penalties (cursed enchantments)
- Enchantment bonuses are included in total resistance calculation

**See separate enchantment documentation for detailed information.**

## Rust Implementation Considerations

### Data Structures

**Resistance Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct Resistances {
    pub physical: f32,
    pub fire: f32,
    pub cold: f32,
    pub poison: f32,
    pub energy: f32,
    pub light: f32,
    pub dark: f32,
}

#[derive(Component, Clone, Copy)]
pub struct ResistanceModifiers {
    pub physical: f32,
    pub fire: f32,
    pub cold: f32,
    pub poison: f32,
    pub energy: f32,
    pub light: f32,
    pub dark: f32,
}

#[derive(Component, Clone, Copy)]
pub struct CreatureImmunities {
    pub physical: bool,
    pub fire: bool,
    pub cold: bool,
    pub poison: bool,
    pub energy: bool,
    pub light: bool,
    pub dark: bool,
}
```

### Performance Optimizations

**Caching:**
- Cache calculated resistances until modifiers change
- Invalidate cache when equipment changes
- Invalidate cache when entering/leaving zones
- Invalidate cache when Magic Resistance skill increases

**Batch Processing:**
- Calculate resistances for multiple entities in parallel
- Use SIMD for resistance calculations when possible
- Pre-calculate zone modifiers for all entities in zone

**Memory Layout:**
- Store resistances in Structure of Arrays (SoA) format for cache efficiency
- Align resistance data for SIMD operations
- Use compact representations (i16 for resistance values if precision allows)

### Serialization

**Resistance Serialization:**
```rust
impl Serialize for Resistances {
    fn serialize(&self) -> Vec<u8> {
        let mut buffer = Vec::with_capacity(28); // 7 * 4 bytes
        
        buffer.extend_from_slice(&self.physical.to_le_bytes());
        buffer.extend_from_slice(&self.fire.to_le_bytes());
        buffer.extend_from_slice(&self.cold.to_le_bytes());
        buffer.extend_from_slice(&self.poison.to_le_bytes());
        buffer.extend_from_slice(&self.energy.to_le_bytes());
        buffer.extend_from_slice(&self.light.to_le_bytes());
        buffer.extend_from_slice(&self.dark.to_le_bytes());
        
        buffer
    }
}
```

## Integration with Other Systems

### Combat System

**Integration:**
- Resistance calculations are performed during damage application
- Resistance values affect damage over time (DoT) effects
- Resistance values affect status effect application chances

### Equipment System

**Integration:**
- Equipment provides resistance bonuses
- Equipment changes trigger resistance recalculation
- Equipment durability affects resistance bonuses (if implemented)

### Skill System

**Integration:**
- Magic Resistance skill provides dynamic resistance bonuses
- Skill experience gained from taking elemental damage
- Skill level affects maximum resistance contribution

### Zone System

**Integration:**
- Zones apply resistance modifiers to entities
- Zone entry/exit triggers resistance recalculation
- Zone difficulty scales with resistance modifiers

## Testing Considerations

### Unit Tests

**Test Cases:**
- Resistance calculation with various attribute values
- Resistance cap enforcement (70% maximum)
- Negative resistance damage increase
- Immunity system functionality
- Magic Resistance skill contribution limits
- Zone modifier application
- Equipment modifier stacking

### Integration Tests

**Test Cases:**
- Complete damage calculation flow
- Resistance recalculation on equipment change
- Resistance recalculation on zone entry/exit
- Magic Resistance skill progression
- Multiple resistance sources stacking correctly

### Performance Tests

**Test Cases:**
- Resistance calculation performance (target: <1μs per entity)
- Batch resistance calculation performance
- Cache invalidation performance
- Memory usage for resistance data structures

## Summary

The resistance system is a fundamental combat mechanic that:

- Provides defense through physical and elemental resistances
- Integrates with attributes (CON for physical, INT for elemental)
- Allows equipment to modify resistances
- Uses zones to create difficulty progression
- Includes a dynamic Magic Resistance skill system
- Enforces a 70% maximum resistance cap
- Supports creature immunities
- Calculates damage based on resistance values
- Supports negative resistance for increased damage

This system creates strategic depth by requiring players to balance their resistances, upgrade equipment, and adapt to different combat situations and zones.

