# Damage System Documentation

## Overview

The damage system is based on the traditional D20 RPG system to maintain stable damage growth and facilitate PvP balance. The system is designed to prevent one-shot scenarios in PvP by limiting maximum damage per attack to 50% of a normal player's maximum HP. This creates engaging combat where players have time to react, use skills, and strategize. The system balances average HP values for players and creatures against maximum possible damage from endgame builds, ensuring fair and enjoyable PvP encounters.

**Note:** All damage values are multiplied by 10 to provide a greater sense of progression. This means equipment dice rolls, attribute modifiers, skill bonuses, and equipment bonuses are all multiplied by 10 in the final calculation. This creates larger numbers (e.g., 2,700 damage instead of 270) while maintaining the same balance ratios.

## Key Features

- **Dice-Based System**: Traditional RPG dice system for stable damage scaling
- **10x Multiplier**: All damage values multiplied by 10 for greater progression feel
- **PvP Balance**: Maximum damage per attack limited to 50% of normal player HP
- **Stable Growth**: Damage growth is controlled and predictable
- **Mobility Factor**: Optional zero padding for build flexibility
- **Endgame Balance**: HP and damage values balanced for endgame builds
- **Reactive Combat**: Players have time to react and counter-attack

## Dice-Based Damage System

### Core Concept

**Description:**
The damage system uses a traditional RPG dice system where each equipment has a damage dice configuration (e.g., D4, D6, D8, D12, D20) and can roll multiple dice. This provides stable, predictable damage ranges that scale linearly rather than exponentially, making it easier to balance PvP encounters.

**Base Formula:**
```
Base Damage = Equipment Dice Roll + Attribute Modifier + Skill Bonus + Equipment Bonuses
Final Damage = Base Damage * Damage Multiplier
```

**Equipment Dice System:**
- Each weapon/equipment has a dice configuration (e.g., "2D6", "5D10", "1D20")
- Format: `XDY` where X = number of dice, Y = number of faces
- Multiple dice provide more consistent damage (bell curve distribution)
- Single high-face dice (D20) provide more variance

### Dice Types and Configurations

**Available Dice Types:**
- **D4** (4-sided die): Range 1-4 per die
- **D6** (6-sided die): Range 1-6 per die
- **D8** (8-sided die): Range 1-8 per die
- **D10** (10-sided die): Range 1-10 per die
- **D12** (12-sided die): Range 1-12 per die
- **D20** (20-sided die): Range 1-20 per die

**Multiple Dice Examples:**
- **1D4**: 1 die, 4 faces → Range: 1-4, Average: 2.5
- **2D6**: 2 dice, 6 faces → Range: 2-12, Average: 7
- **3D8**: 3 dice, 8 faces → Range: 3-24, Average: 13.5
- **5D10**: 5 dice, 10 faces → Range: 5-50, Average: 27.5
- **6D12**: 6 dice, 12 faces → Range: 6-72, Average: 39
- **1D20**: 1 die, 20 faces → Range: 1-20, Average: 10.5

**Dice Roll Implementation:**
```rust
pub fn roll_dice(dice_config: &str) -> u16 {
    if dice_config == "None" {
        return 0;
    }
    
    // Parse format: "XDY" (e.g., "2D6", "5D10")
    let parts: Vec<&str> = dice_config.split('D').collect();
    if parts.len() != 2 {
        return 0;
    }
    
    let num_dice: u8 = parts[0].parse().unwrap_or(0);
    let num_sides: u8 = parts[1].parse().unwrap_or(0);
    
    if num_dice == 0 || num_sides == 0 {
        return 0;
    }
    
    let mut rng = rand::thread_rng();
    let mut total = 0;
    
    for _ in 0..num_dice {
        total += rng.gen_range(1..=num_sides);
    }
    
    total
}
```

**TypeScript Implementation (from codebase):**
```typescript
public rollDice(dice: Dices): number {
    if (dice === Dices.None) return 0; 
    
    const match = dice.match(/(\d+)D(\d+)/);
    
    if (!match) return 0; 
    
    const numDices = parseInt(match[1], 10);
    const numSides = parseInt(match[2], 10);
    
    let total = 0;
    
    for (let i = 0; i < numDices; i++) 
        total += Math.floor(Math.random() * numSides) + 1;
    
    return total;
}
```

### Equipment Damage Dice

**Description:**
Each equipment item (weapons, tools, etc.) has a `Damage` property that defines its dice configuration. This determines the base damage range for that equipment.

**Equipment Examples:**
- **Daggers**: Typically use D4 (e.g., D1D4, D3D4, D5D4)
- **Swords**: Mix of D6 and D10 (e.g., D2D6, D5D10, D1D12)
- **Axes**: Primarily D10 (e.g., D1D10, D3D10, D5D10)
- **Hammers**: Primarily D12 (e.g., D1D12, D3D12, D5D12)
- **Bows/Crossbows**: D6 and D8 (e.g., D1D6, D2D8, D6D6)
- **Staffs**: D4, D6, and D12 (e.g., D1D4, D5D6, D6D12)
- **Spears**: Primarily D10 (e.g., D1D10, D3D10, D5D10)

**Damage Distribution:**
- **Low-tier equipment**: Fewer dice, lower faces (e.g., D1D4, D1D6)
- **Mid-tier equipment**: More dice or higher faces (e.g., D2D8, D3D10)
- **High-tier equipment**: Many dice or high faces (e.g., D5D10, D6D12)

**Why Multiple Dice?**
- Multiple dice create a bell curve distribution (more consistent damage)
- Single high-face dice create uniform distribution (more variance)
- Allows fine-tuning of damage ranges for balance
- Provides predictable average damage while maintaining variance

### Damage Components

**1. Equipment Dice Roll**
- Base damage from equipment's dice configuration
- Examples: 2D6 (2-12), 5D10 (5-50), 1D20 (1-20)
- Provides base variance in damage
- Ensures no guaranteed one-shot scenarios
- Each equipment type has appropriate dice for its tier

**2. Attribute Modifier**
- Based on primary damage attribute (STR for physical, INT for magical)
- Scales linearly with attribute points
- Formula: `Attribute Modifier = ((Attribute - 10) / 2) * 10` (D&D style, multiplied by 10)
- Example: STR 150 → Modifier: ((150-10)/2)*10 = 700

**3. Skill Bonus**
- Based on relevant combat skill level
- Provides scaling damage bonus
- Formula: `Skill Bonus = (Skill Level * Skill_Multiplier) * 10` (multiplied by 10)
- Example: Skill Level 50, Multiplier 2 → Bonus: (50*2)*10 = 1,000

**4. Equipment Bonuses**
- Flat bonuses from equipment attributes (multiplied by 10)
- Equipment tier affects base damage
- Enchantments can add additional damage
- Critical chance and critical damage modifiers

**5. Damage Multiplier**
- Final multiplier applied to total
- Can be modified by buffs, debuffs, or special abilities
- Default: 1.0

**Implementation:**
```rust
use rand::Rng;

pub enum DiceConfig {
    None,
    Custom { num_dice: u8, num_sides: u8 },
}

pub struct DamageRoll {
    pub dice_roll: u16,
    pub attribute_modifier: i16,
    pub skill_bonus: i16,
    pub equipment_bonuses: i16,
    pub damage_multiplier: f32,
}

impl DamageRoll {
    pub fn roll_damage(&self) -> f32 {
        let base = self.dice_roll as f32 
            + self.attribute_modifier as f32 
            + self.skill_bonus as f32 
            + self.equipment_bonuses as f32;
        
        base * self.damage_multiplier
    }
    
    pub fn roll_dice(dice_config: &str) -> u16 {
        if dice_config == "None" {
            return 0;
        }
        
        let parts: Vec<&str> = dice_config.split('D').collect();
        if parts.len() != 2 {
            return 0;
        }
        
        let num_dice: u8 = parts[0].parse().unwrap_or(0);
        let num_sides: u8 = parts[1].parse().unwrap_or(0);
        
        if num_dice == 0 || num_sides == 0 {
            return 0;
        }
        
        let mut rng = rand::thread_rng();
        let mut total = 0;
        
        for _ in 0..num_dice {
            total += rng.gen_range(1..=num_sides);
        }
        
        total
    }
    
    pub fn new(
        dice_config: &str,
        attribute: u16,
        skill_level: u8,
        equipment_bonuses: i16,
        damage_multiplier: f32,
    ) -> Self {
        let dice_roll = Self::roll_dice(dice_config) * 10; // Multiply by 10 for progression feel
        let attribute_modifier = (((attribute as i16) - 10) / 2) * 10; // Multiply by 10
        let skill_bonus = ((skill_level as i16) * 2) * 10; // Example: 2 damage per skill level, multiplied by 10
        
        Self {
            dice_roll,
            attribute_modifier,
            skill_bonus,
            equipment_bonuses: equipment_bonuses * 10, // Multiply by 10
            damage_multiplier,
        }
    }
}
```

## PvP Damage Limit

### 50% HP Rule

**Description:**
No single attack can deal more than 50% of a normal player's maximum HP. This rule ensures PvP remains engaging and prevents one-shot scenarios where players die before they can react.

**Rationale:**
- Players need time to react to damage
- Allows for counter-play and strategy
- Prevents frustration from instant deaths
- Maintains engagement in PvP encounters

**Implementation:**
```rust
pub fn calculate_pvp_damage(
    base_damage: f32,
    target_max_hp: f32,
    is_pvp: bool,
) -> f32 {
    if is_pvp {
        let max_damage = target_max_hp * 0.5; // 50% of max HP
        base_damage.min(max_damage)
    } else {
        base_damage // No limit in PvE
    }
}
```

**Example:**
```
Player Max HP: 10,000
Calculated Damage: 6,000
Final Damage (PvP): 5,000 (capped at 50% = 5,000)
```

### Normal Player HP Definition

**Description:**
A "normal player" is defined as a player with average HP for their level, using standard equipment and attribute distribution. This serves as the baseline for damage calculations.

**Average HP Calculation:**
```
Average HP = Base HP + (CON * HP_per_CON) + Equipment HP Bonuses
```

**Level-Based HP Ranges:**
- **Level 1-20**: 2,000-5,000 HP
- **Level 21-40**: 5,000-8,000 HP
- **Level 41-60**: 8,000-12,000 HP
- **Level 61-80**: 12,000-18,000 HP
- **Level 81-100**: 18,000-25,000 HP

**Normal Player (Level 50):**
- Base HP: 5,000
- VIG: 50 (average)
- HP from VIG: 50 * 50 = 2,500
- STR: 50 (average)
- HP from STR: 50 * 10 = 500
- Equipment HP: +2,000
- **Total: ~10,000 HP**
- **50% Limit: 5,000 damage**

## Damage Growth Stability

### Linear Scaling

**Description:**
Damage scales linearly with attributes, equipment, and skills. This prevents exponential growth that would break PvP balance.

**Scaling Formula:**
```
Total Damage = (D20 + Attribute_Modifier + Equipment + Skill) * Multiplier
```

**Growth Per Level:**
- Attribute points: +5 per level
- Attribute modifier: +2.5 per level (assuming all points in damage attribute)
- Equipment: +5-10 per tier
- Skills: +2-5 per skill level

**Example Growth (with 10x multiplier):**
```
Level 1:  (Dice: 10.5*10 + STR: 0*10 + Equipment: 5*10 + Skill: 0*10) * 1.0 = 155 average
Level 50: (Dice: 10.5*10 + STR: 100*10 + Equipment: 50*10 + Skill: 50*10) * 1.0 = 2,105 average
Level 100: (Dice: 10.5*10 + STR: 200*10 + Equipment: 100*10 + Skill: 100*10) * 1.0 = 4,105 average
```

### Damage Range Control

**Minimum Damage:**
- Ensured by D20 minimum roll of 1
- Prevents zero-damage attacks
- Formula: `Min Damage = (1 + Min_Modifiers) * Multiplier`

**Maximum Damage:**
- Controlled by D20 maximum roll of 20
- Limited by PvP cap (50% HP)
- Formula: `Max Damage = min((20 + Max_Modifiers) * Multiplier, Target_HP * 0.5)`

**Variance:**
- D20 provides consistent variance
- Range: 1-20 (19 point spread)
- Standard deviation: ~5.77
- Prevents extreme damage spikes

## Endgame Balance

### Endgame Build Damage

**Maximum Endgame Damage Calculation (with 10x multiplier):**
```
Max Endgame Damage = (Dice: 72*10 + Attribute: 200*10 + Equipment: 150*10 + Skill: 150*10) * Multiplier: 1.5
Max Endgame Damage = (720 + 2,000 + 1,500 + 1,500) * 1.5 = 7,800 base damage
```

**After Resistance (70% cap):**
```
Final Damage = 7,800 * (1 - 0.70) = 2,340 damage
```

**With PvP Cap (50% HP):**
```
If target has 12,000 HP:
Max PvP Damage = min(2,340, 12,000 * 0.5) = 2,340 damage (no cap needed)
```

### Endgame HP Requirements

**To Balance Endgame Damage:**
```
Required HP = Max_Damage / 0.5
Required HP = 7,800 / 0.5 = 15,600 HP minimum
```

**Recommended Endgame HP:**
- **Minimum**: 15,000 HP (survives 2 hits from max damage)
- **Average**: 20,000 HP (comfortable margin)
- **Maximum**: 30,000 HP (tank builds)

**HP Sources:**
- Base HP: 5,000
- VIG (200): 200 * 50 = 10,000 HP
- STR (200): 200 * 10 = 2,000 HP
- Equipment: +5,000 HP
- **Total: 22,000 HP (tank build)**

### Balance Verification

**Endgame DPS Build vs Average Player:**
```
DPS Build Damage: 7,800 base
Average Player HP: 20,000
Average Player Resistance: 50%
Final Damage: 7,800 * 0.5 = 3,900
Damage %: 3,900 / 20,000 = 19.5% (well below 50% limit)
```

**Endgame DPS Build vs Tank Build:**
```
DPS Build Damage: 7,800 base
Tank Build HP: 30,000
Tank Build Resistance: 70%
Final Damage: 7,800 * 0.3 = 2,340
Damage %: 2,340 / 30,000 = 7.8% (very safe)
```

**Endgame DPS Build vs Low HP Build:**
```
DPS Build Damage: 7,800 base
Low HP Build HP: 12,000
Low HP Build Resistance: 30%
Final Damage: 7,800 * 0.7 = 5,460
Damage %: 5,460 / 12,000 = 45.5% (just below 50% limit)
PvP Cap Applied: min(5,460, 12,000 * 0.5) = 6,000 (capped at 50%)
```

## Mobility Factor (Optional Zero Padding)

### Description

**Optional Zero Padding:**
The system allows for an optional zero padding factor that can be added to damage calculations. This provides flexibility for builds that prioritize mobility over raw damage, allowing them to remain competitive while maintaining different playstyles.

**Purpose:**
- Allows mobility-focused builds to remain viable
- Provides build diversity
- Maintains balance while offering flexibility
- Can be used for special abilities or enchantments

**Implementation:**
```rust
pub struct DamageCalculation {
    pub base_damage: f32,
    pub mobility_factor: f32, // Optional zero padding
    pub final_damage: f32,
}

impl DamageCalculation {
    pub fn calculate(&mut self) {
        // Mobility factor adds a small flat bonus
        // This allows mobility builds to compete without breaking balance
        self.final_damage = self.base_damage + self.mobility_factor;
    }
}
```

**Example Usage:**
```
Base Damage: 2,000
Mobility Factor: +200 (from mobility-focused enchantments, multiplied by 10)
Final Damage: 2,200
```

**Balance Consideration:**
- Mobility factor should be small relative to base damage
- Recommended: 5-10% of base damage
- Prevents mobility builds from out-damaging pure DPS builds

## Damage Types and Calculations

### Physical Damage

**Formula:**
```
Physical Damage = Equipment_Dice_Roll + STR_Modifier + Weapon_Skill_Bonus + Equipment_Bonuses
Base Damage = Physical Damage * Physical_Multiplier
Final Physical Damage = Base Damage * (1 - Physical_Resistance / 100)
```

**Example:**
```rust
pub fn calculate_physical_damage(
    attacker: &Entity,
    weapon: &Weapon,
    target: &Entity,
) -> f32 {
    // Roll equipment dice (e.g., "5D10" = 5 dice with 10 faces each) and multiply by 10
    let dice_roll = roll_dice(weapon.damage_dice) * 10;
    
    // Calculate attribute modifier and multiply by 10
    let str_modifier = calculate_attribute_modifier(attacker.strength) * 10;
    
    // Calculate skill bonus and multiply by 10
    let skill_bonus = calculate_weapon_skill_bonus(attacker, weapon) * 10;
    
    // Get equipment bonuses (flat damage bonuses from equipment) and multiply by 10
    let equipment_bonuses = attacker.get_equipment_damage_bonuses() * 10;
    
    // Calculate base damage
    let base_damage = (dice_roll as f32 
        + str_modifier 
        + skill_bonus 
        + equipment_bonuses) 
        * attacker.physical_damage_multiplier;
    
    // Apply resistance
    let resistance = target.calculate_physical_resistance();
    let final_damage = apply_resistance(base_damage, resistance);
    
    // Apply PvP cap if applicable
    if attacker.is_player() && target.is_player() {
        calculate_pvp_damage(final_damage, target.max_hp, true)
    } else {
        final_damage
    }
}
```

**Example Calculation:**
```
Weapon: 5D10 (rolls 35, multiplied by 10 = 350)
STR: 150 → Modifier: 700 (70 * 10)
Skill Level: 50 → Bonus: 1,000 (100 * 10)
Equipment Bonuses: +200 (20 * 10)
Multiplier: 1.2

Base Damage = (350 + 700 + 1,000 + 200) * 1.2 = 2,700
Target Resistance: 40%
Final Damage = 2,700 * (1 - 0.40) = 1,620
```

### Elemental Damage

**Formula:**
```
Elemental Damage = Spell_Dice_Roll + INT_Modifier + Spell_Skill_Bonus + Equipment_Bonuses
Base Damage = Elemental Damage * Elemental_Multiplier
Final Elemental Damage = Base Damage * (1 - Elemental_Resistance / 100)
```

**Example:**
```rust
pub fn calculate_elemental_damage(
    attacker: &Entity,
    spell: &Spell,
    element: ElementType,
    target: &Entity,
) -> f32 {
    // Roll spell dice (e.g., "3D8" = 3 dice with 8 faces each) and multiply by 10
    let dice_roll = roll_dice(spell.damage_dice) * 10;
    
    // Calculate attribute modifier and multiply by 10
    let int_modifier = calculate_attribute_modifier(attacker.intelligence) * 10;
    
    // Calculate skill bonus and multiply by 10
    let skill_bonus = calculate_spell_skill_bonus(attacker, spell) * 10;
    
    // Get equipment bonuses (magic damage bonuses from equipment) and multiply by 10
    let equipment_bonuses = attacker.get_equipment_magic_damage_bonuses() * 10;
    
    // Calculate base damage
    let base_damage = (dice_roll as f32 
        + int_modifier 
        + skill_bonus 
        + equipment_bonuses) 
        * attacker.elemental_damage_multiplier;
    
    // Apply resistance
    let resistance = target.calculate_elemental_resistance(element);
    let final_damage = apply_resistance(base_damage, resistance);
    
    // Apply PvP cap if applicable
    if attacker.is_player() && target.is_player() {
        calculate_pvp_damage(final_damage, target.max_hp, true)
    } else {
        final_damage
    }
}
```

**Example Calculation:**
```
Spell: 3D8 (rolls 18, multiplied by 10 = 180)
INT: 150 → Modifier: 700 (70 * 10)
Skill Level: 50 → Bonus: 1,000 (100 * 10)
Equipment Bonuses: +150 (15 * 10)
Multiplier: 1.2

Base Damage = (180 + 700 + 1,000 + 150) * 1.2 = 2,436
Target Fire Resistance: 50%
Final Damage = 2,436 * (1 - 0.50) = 1,218
```

## Complete Damage Calculation Flow

### Step-by-Step Process

**1. Roll Equipment Dice**
```rust
let dice_roll = roll_dice(weapon.damage_dice) * 10; // e.g., "5D10" → rolls 5 dice, sums result, multiply by 10
```

**2. Calculate Attribute Modifier**
```rust
let attribute_modifier = (((attribute as i16) - 10) / 2) * 10; // Multiply by 10 for progression feel
```

**3. Calculate Skill Bonus**
```rust
let skill_bonus = (skill_level as i16 * skill_damage_per_level) * 10; // Multiply by 10 for progression feel
```

**4. Get Equipment Bonuses**
```rust
let equipment_bonuses = (weapon.enchantment_damage + attacker.get_equipment_damage_bonuses()) * 10; // Multiply by 10
```

**5. Calculate Base Damage**
```rust
let base_damage = (dice_roll as f32 
    + attribute_modifier 
    + skill_bonus 
    + equipment_bonuses) 
    * damage_multiplier;
```

**6. Apply Resistance**
```rust
let resistance = target.calculate_resistance(damage_type);
let damage_after_resistance = apply_resistance(base_damage, resistance);
```

**7. Apply PvP Cap (if applicable)**
```rust
let final_damage = if is_pvp {
    calculate_pvp_damage(damage_after_resistance, target.max_hp, true)
} else {
    damage_after_resistance
};
```

**8. Apply Mobility Factor (optional)**
```rust
let final_damage = final_damage + mobility_factor;
```

**Complete Implementation:**
```rust
pub fn calculate_damage(
    attacker: &Entity,
    target: &Entity,
    weapon: &Weapon,
    damage_type: DamageType,
    is_pvp: bool,
) -> f32 {
    // Step 1: Roll equipment dice (e.g., "5D10", "2D6", "1D20") and multiply by 10
    let dice_roll = roll_dice(weapon.damage_dice) * 10;
    
    // Step 2: Calculate attribute modifier and multiply by 10
    let attribute = match damage_type {
        DamageType::Physical => attacker.strength,
        _ => attacker.intelligence,
    };
    let attribute_modifier = (((attribute as i16) - 10) / 2) * 10;
    
    // Step 3: Calculate skill bonus and multiply by 10
    let skill = weapon.get_required_skill();
    let skill_level = attacker.get_skill_level(skill);
    let skill_bonus = ((skill_level as i16) * 2) * 10; // 2 damage per skill level, multiplied by 10
    
    // Step 4: Get equipment bonuses and multiply by 10
    let equipment_bonuses = (weapon.get_enchantment_damage() 
        + attacker.get_equipment_damage_bonuses(damage_type)) * 10;
    
    // Step 5: Calculate base damage
    let damage_multiplier = attacker.get_damage_multiplier(damage_type);
    let base_damage = (dice_roll as f32 
        + attribute_modifier as f32 
        + skill_bonus as f32 
        + equipment_bonuses as f32) 
        * damage_multiplier;
    
    // Step 6: Apply resistance
    let resistance = target.calculate_resistance(damage_type);
    let damage_after_resistance = apply_resistance(base_damage, resistance);
    
    // Step 7: Apply PvP cap
    let mut final_damage = if is_pvp {
        calculate_pvp_damage(damage_after_resistance, target.max_hp, true)
    } else {
        damage_after_resistance
    };
    
    // Step 8: Apply mobility factor (optional)
    final_damage += attacker.mobility_damage_factor;
    
    final_damage
}
```

## PvP Balance Examples

### Scenario 1: Equal Level Players

**Player A (DPS Build):**
- Level: 50
- STR: 150
- Weapon Damage: 80
- Skill Level: 50
- Damage Multiplier: 1.2

**Player B (Average Build):**
- Level: 50
- HP: 3,100
- Physical Resistance: 40%

**Damage Calculation:**
```
Weapon Dice: 5D10 (rolls 32, multiplied by 10 = 320)
STR Modifier: (150 - 10) / 2 * 10 = 700
Skill: 50 * 2 * 10 = 1,000
Equipment Bonuses: +200 (20 * 10)
Base: (320 + 700 + 1,000 + 200) * 1.2 = 2,664
After Resistance: 2,664 * (1 - 0.40) = 1,598.4
PvP Cap Check: min(1,598.4, 3,100 * 0.5) = 1,550
Final Damage: 1,550 (50% of HP - at cap)
```

**Result:** Player B survives with 50% HP remaining (at PvP cap), can react and counter-attack.

### Scenario 2: Endgame vs Average

**Player A (Endgame DPS):**
- Level: 100
- STR: 200
- Weapon Damage: 150
- Skill Level: 100
- Damage Multiplier: 1.5

**Player B (Average):**
- Level: 50
- HP: 3,100
- Physical Resistance: 30%

**Damage Calculation:**
```
Weapon Dice: 6D12 (rolls 45, multiplied by 10 = 450 - maximum would be 720)
STR Modifier: (200 - 10) / 2 * 10 = 950
Skill: 100 * 2 * 10 = 2,000
Equipment Bonuses: +500 (50 * 10)
Base: (450 + 950 + 2,000 + 500) * 1.5 = 5,850
After Resistance: 5,850 * (1 - 0.30) = 4,095
PvP Cap: min(4,095, 3,100 * 0.5) = 1,550
Final Damage: 1,550 (50% of HP - at cap)
```

**Result:** Player B survives with 50% HP (at PvP cap), still has time to react.

### Scenario 3: Maximum Damage Build

**Player A (Max Damage Build):**
- All damage optimizations
- Base Damage: 7,800 (from endgame example, multiplied by 10)

**Player B (Minimum HP Build):**
- HP: 2,700 (minimum viable mage build)
- Resistance: 20% (low)

**Damage Calculation:**
```
Weapon Dice: 6D12 (rolls 60, multiplied by 10 = 600 - high roll)
STR Modifier: (200 - 10) / 2 * 10 = 950
Skill: 100 * 2 * 10 = 2,000
Equipment Bonuses: +1,000 (100 * 10)
Base: (600 + 950 + 2,000 + 1,000) * 1.5 = 6,825
After Resistance: 6,825 * (1 - 0.20) = 5,460
PvP Cap: min(5,460, 2,700 * 0.5) = 1,350
Final Damage: 1,350 (50% of HP - at cap)
```

**Result:** Player B survives with exactly 50% HP, demonstrating the system working as intended.

## Rust Implementation Considerations

### Data Structures

**Damage Calculation Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct DamageStats {
    pub base_damage: f32,
    pub physical_damage_multiplier: f32,
    pub elemental_damage_multiplier: f32,
    pub mobility_factor: f32,
}

#[derive(Component)]
pub struct DamageRoll {
    pub d20_roll: u8,
    pub attribute_modifier: i16,
    pub equipment_bonus: i16,
    pub skill_bonus: i16,
    pub damage_multiplier: f32,
}
```

### Performance Optimizations

**Caching:**
- Cache attribute modifiers until attributes change
- Cache skill bonuses until skill levels change
- Pre-calculate equipment bonuses on equipment change
- Cache resistance values until modifiers change

**Random Number Generation:**
- Use fast RNG for D20 rolls
- Consider pre-rolling for batch processing
- Use thread-local RNG to avoid contention

**Batch Processing:**
- Calculate damage for multiple attacks in parallel
- Use SIMD for resistance calculations when possible
- Pre-calculate PvP caps for all targets in area

### Serialization

**Damage Roll Serialization:**
```rust
impl Serialize for DamageRoll {
    fn serialize(&self) -> Vec<u8> {
        let mut buffer = Vec::with_capacity(9); // 1 + 2 + 2 + 2 + 4 bytes
        
        buffer.push(self.d20_roll);
        buffer.extend_from_slice(&self.attribute_modifier.to_le_bytes());
        buffer.extend_from_slice(&self.equipment_bonus.to_le_bytes());
        buffer.extend_from_slice(&self.skill_bonus.to_le_bytes());
        buffer.extend_from_slice(&self.damage_multiplier.to_le_bytes());
        
        buffer
    }
}
```

## Integration with Other Systems

### Combat System

**Integration:**
- Damage calculation is performed during attack resolution
- PvP flag determines if cap is applied
- Damage is applied to target's HP
- Death check performed after damage application

### Resistance System

**Integration:**
- Resistance values directly affect final damage
- See [RESISTANCES.md](./RESISTANCES.md) for resistance calculations
- Damage type determines which resistance is used

### Equipment System

**Integration:**
- Weapon damage contributes to base damage
- Equipment enchantments can modify damage
- Equipment tier affects base damage values

### Skill System

**Integration:**
- Skill levels provide damage bonuses
- Higher skill levels = higher damage
- Skill experience gained from dealing damage

### Player System

**Integration:**
- Attributes (STR/INT) affect damage modifiers
- HP values determine PvP cap
- Level affects available attribute points

## Testing Considerations

### Unit Tests

**Test Cases:**
- D20 roll range (1-20)
- Attribute modifier calculation
- PvP cap enforcement (50% HP limit)
- Resistance application
- Damage multiplier application
- Mobility factor application

### Integration Tests

**Test Cases:**
- Complete damage calculation flow
- PvP vs PvE damage differences
- Endgame build damage verification
- HP balance verification
- Multiple damage types

### Balance Tests

**Test Cases:**
- Maximum damage vs minimum HP (should be ≤50%)
- Average damage vs average HP (should be reasonable)
- Endgame damage vs endgame HP (should be balanced)
- Level difference damage scaling

### Performance Tests

**Test Cases:**
- Damage calculation performance (target: <1μs per calculation)
- Batch damage calculation performance
- RNG performance for D20 rolls
- Memory usage for damage data structures

## Summary

The damage system:

- Uses D20-based calculations for stable, predictable damage growth
- Limits PvP damage to 50% of target's max HP to prevent one-shot scenarios
- Balances endgame builds with appropriate HP values
- Provides optional mobility factor for build diversity
- Scales linearly to maintain balance across all levels
- Integrates with resistance, equipment, skill, and player systems

This system creates engaging PvP encounters where players have time to react, strategize, and counter-attack, while maintaining fair and balanced combat across all levels and build types.

