# Long Range Weapons Skill Documentation

## Overview

Long Range Weapons is a combat skill that represents proficiency with ranged weapons. This skill amplifies damage, accuracy, and effective range when wielding bows and crossbows, making it essential for archers, rangers, and ranged-focused builds. The skill progresses through combat experience gained by attacking with ranged weapons, and unlocks powerful perks at specific levels that enhance secondary statistics and ranged combat effectiveness.

## Key Features

- **Weapon Types**: Bow, Crossbow
- **Primary Stat**: Dexterity
- **Secondary Stat**: Strength
- **Damage Amplification**: Increases damage output with ranged weapons
- **Accuracy Enhancement**: Improves hit chance and reduces damage reduction
- **Range Enhancement**: Increases effective range of ranged weapons
- **No Miss System**: Attacks always hit but damage can be reduced significantly
- **Progressive Perks**: Unlocks perks at levels 20, 40, 60, 80, and 100
- **Secondary Stat Bonuses**: Perks provide bonuses to Strength and other secondary stats
- **Level Range**: 0 to 100 (maximum skill level)

## Skill Progression

### Experience Gain

**Combat-Based Progression:**
- Gain XP by attacking with ranged weapons
- Base XP gain: 3 XP per attack
- XP scales with combat effectiveness
- Successful hits grant more XP than reduced-damage hits
- Critical hits grant bonus XP
- Long-range hits grant bonus XP

**Weapons That Progress This Skill:**
- **Bow** (all types)
- **Crossbow** (all types)

**Weapons That Do NOT Progress This Skill:**
- One-handed melee weapons (progress Combat With Weapons skill)
- Two-handed melee weapons (progress Two Handed Weapons skill)
- Unarmed attacks (progress Unarmed Combat skill)

### Level Requirements

**Level Progression:**
- Level 0: No bonus (base effectiveness)
- Level 1-10: Small bonuses begin
- Level 11-50: Moderate damage, accuracy, and range bonuses
- Level 51-100: Significant damage, accuracy, and range bonuses
- Level 100: Maximum skill level (can be increased with Power Scrolls)

## Skill Effects

### Damage Amplification

**Damage Bonus Formula:**
```
Skill Bonus = max(0, Skill Level - 10)
Damage Multiplier = 1.0 + (Skill Bonus × 0.005)
Final Damage = Base Damage × Damage Multiplier
```

**Note:** Damage bonus scales at 0.5% per level after level 10, providing up to +45% damage at level 100.

**Example Calculations:**
- Level 0-10: No damage bonus (0% increase)
- Level 20: +5% damage (1.05x multiplier)
- Level 30: +10% damage (1.10x multiplier)
- Level 50: +20% damage (1.20x multiplier)
- Level 75: +32.5% damage (1.325x multiplier)
- Level 100: +45% damage (1.45x multiplier)

**Damage Application:**
- Bonus applies to all damage dealt with ranged weapons
- Multiplies final damage after all other calculations
- Works with physical and elemental damage (if weapon has elemental properties)
- Stacks multiplicatively with other damage bonuses

### Accuracy Enhancement

**Accuracy System:**
- No traditional "miss" mechanic
- All attacks connect with the target
- Low accuracy results in significant damage reduction instead
- Higher skill level reduces damage reduction penalty
- Range affects accuracy (closer = more accurate, farther = less accurate)

**Damage Reduction Formula:**
```
Base Accuracy = 100% - (Enemy Evasion × 0.5) - Range Penalty
Range Penalty = max(0, (Distance - Optimal Range) × 0.5%)
Skill Accuracy Bonus = Skill Level × 0.2%
Final Accuracy = Base Accuracy + Skill Accuracy Bonus
Damage Reduction = max(0, 100% - Final Accuracy)
Final Damage = Base Damage × (1.0 - Damage Reduction)
```

**Note:** Accuracy bonus scales at 0.2% per level, providing up to +20% accuracy at level 100.

**Range Penalty:**
- Optimal Range: Weapon's base range (no penalty)
- Close Range: -5% accuracy (too close for ranged weapons)
- Medium Range: 0% penalty (optimal)
- Long Range: -1% per unit beyond optimal range
- Maximum Range: -50% accuracy (extreme range)

**Example Scenarios:**
- **Level 0 vs High Evasion Enemy (Long Range):**
  - Base Accuracy: 50%
  - Range Penalty: -20%
  - Skill Bonus: 0%
  - Final Accuracy: 30%
  - Damage Reduction: 70%
  - Result: 30% of damage dealt

- **Level 50 vs High Evasion Enemy (Optimal Range):**
  - Base Accuracy: 50%
  - Range Penalty: 0%
  - Skill Bonus: 10%
  - Final Accuracy: 60%
  - Damage Reduction: 40%
  - Result: 60% of damage dealt

- **Level 100 vs High Evasion Enemy (Optimal Range):**
  - Base Accuracy: 50%
  - Range Penalty: 0%
  - Skill Bonus: 20%
  - Final Accuracy: 70%
  - Damage Reduction: 30%
  - Result: 70% of damage dealt

### Range Enhancement

**Range Bonus Formula:**
```
Base Range = Weapon Base Range
Skill Range Bonus = Skill Level × 0.2%
Enhanced Range = Base Range × (1.0 + Skill Range Bonus)
```

**Note:** Range bonus scales at 0.2% per level, providing up to +20% range at level 100.

**Example Calculations:**
- Level 0: No range bonus (base weapon range)
- Level 50: +10% range (1.10x multiplier)
- Level 100: +20% range (1.20x multiplier)

**Range Benefits:**
- Attack from farther distances
- Maintain optimal range more easily
- Better positioning options
- Reduced range penalty impact

## Perks System

### Perk Unlock Levels

Perks are unlocked at specific skill levels, providing permanent bonuses and enhancements:

- **Level 20**: First Perk - Strength Boost
- **Level 40**: Second Perk - Steady Aim
- **Level 60**: Third Perk - Archer's Precision
- **Level 80**: Fourth Perk - Long Shot Mastery
- **Level 100**: Fifth Perk - Master Archer

### Level 20 Perk: Strength Boost

**Effect:**
- Grants +2 Strength permanently
- Improves secondary stat for this skill
- Enhances physical damage and HP
- Synergizes with Strength-based ranged builds

**Benefits:**
- +2 Strength = +2% Physical Damage
- +2 Strength = +20 HP
- Better synergy with Strength-scaling equipment
- Improved survivability for ranged builds

### Level 40 Perk: Steady Aim

**Effect:**
- Reduces damage reduction by additional 5%
- Reduces range penalty by 25%
- Improves accuracy at all ranges
- Makes ranged attacks more consistent

**Benefits:**
- Attacks deal more damage even at long range
- More reliable damage output
- Better performance at extreme ranges
- Reduces RNG variance in combat

### Level 60 Perk: Archer's Precision

**Effect:**
- Grants +3 Strength permanently
- Increases critical hit chance by 3%
- Reduces range penalty by additional 25%
- Enhances ranged combat effectiveness

**Benefits:**
- Total +5 Strength from perks (Level 20 + Level 60)
- Better critical hit potential
- Improved long-range accuracy
- More dynamic ranged combat style

### Level 80 Perk: Long Shot Mastery

**Effect:**
- Grants +2 Dexterity permanently
- Increases range bonus by additional 10%
- Improves accuracy at long range
- Enhances overall ranged effectiveness

**Benefits:**
- +2 Dexterity = +2% Critical Hit Chance
- +2 Dexterity = +2% Evasion
- Additional 10% range on top of skill bonus
- Maximum effective range capabilities

### Level 100 Perk: Master Archer

**Effect:**
- Grants +3 Dexterity permanently
- Increases all damage by additional 10%
- Reduces damage reduction by additional 10%
- Eliminates close-range accuracy penalty
- Ultimate ranged combat mastery

**Benefits:**
- Total +5 Dexterity from perks (Level 80 + Level 100)
- Significant damage amplification
- Maximum accuracy and consistency
- No penalty for close-range combat
- Peak performance for ranged weapon builds

## Complete Perk Summary

**Total Stat Bonuses from Perks:**
- **Strength**: +5 (Level 20: +2, Level 60: +3)
- **Dexterity**: +5 (Level 80: +2, Level 100: +3)

**Total Combat Bonuses:**
- **Damage Reduction Reduction**: -15% (Level 40: -5%, Level 100: -10%)
- **Range Penalty Reduction**: -50% (Level 40: -25%, Level 60: -25%)
- **Critical Hit Chance**: +3% (Level 60)
- **Additional Range**: +10% (Level 80)
- **Additional Damage**: +10% (Level 100)
- **Close-Range Penalty**: Eliminated (Level 100)

## Build Recommendations

### Dexterity-Focused Build

**Attributes:**
- High Dexterity (primary)
- Moderate Strength (secondary)
- Moderate Constitution (survivability)

**Benefits:**
- Maximum critical hit chance
- Excellent evasion
- Fast attack speed
- Perks enhance Dexterity further
- Great with fast-firing bows

### Strength-Focused Build

**Attributes:**
- High Strength (primary)
- Moderate Dexterity (secondary)
- High Constitution (tanky archer)

**Benefits:**
- Maximum physical damage output
- Good HP pool
- Strong with heavy crossbows
- Perks enhance Strength further
- Better survivability

### Balanced Build

**Attributes:**
- Balanced Strength and Dexterity
- Moderate Constitution
- Some Intelligence (for hybrid builds)

**Benefits:**
- Benefits from all perks equally
- Versatile combat style
- Good with all ranged weapon types
- Adaptable to different situations

## Integration with Other Systems

### Equipment System

**Weapon Requirements:**
- Skill level may be required for certain ranged weapons
- Higher-tier weapons may require minimum skill level
- Skill level affects weapon effectiveness
- Range bonuses stack with weapon range

**Equipment Bonuses:**
- Skill bonuses stack with equipment bonuses
- Weapon-specific bonuses enhance skill effectiveness
- Set bonuses can synergize with skill perks
- Ammunition affects damage (if applicable)

### Combat System

**Damage Calculation:**
```
Base Damage = Equipment Dice Roll × 10
Attribute Damage = (Dexterity × 10) + (Strength × 5)
Skill Bonus = Skill Level Bonus × Base Damage
Equipment Bonus = Equipment Modifiers × Base Damage
Range Multiplier = Calculate Range Multiplier(Distance, Skill Level)
Final Damage = (Base Damage + Attribute Damage + Skill Bonus + Equipment Bonus) × Accuracy Multiplier × Range Multiplier
```

**Accuracy Calculation:**
```
Base Accuracy = 100% - (Enemy Evasion × 0.5)
Range Penalty = Calculate Range Penalty(Distance, Optimal Range, Skill Level, Perks)
Skill Accuracy = Skill Level × 2%
Perk Accuracy = Perk Bonuses
Final Accuracy = Base Accuracy + Skill Accuracy + Perk Accuracy - Range Penalty
Damage Reduction = max(0, 100% - Final Accuracy)
```

**Range Calculation:**
```
Base Range = Weapon Base Range
Skill Range Bonus = Skill Level × 2%
Perk Range Bonus = Perk Bonuses
Enhanced Range = Base Range × (1.0 + Skill Range Bonus + Perk Range Bonus)
```

### Secondary Stats System

**Evasion Integration:**
- Dexterity from perks improves evasion
- Higher evasion = better survivability
- Synergizes with evasion-focused builds
- Ranged builds benefit from high evasion

**Critical Hit Integration:**
- Dexterity from perks improves critical hit chance
- Level 6 perk provides additional critical hit chance
- Critical hits deal significantly more damage
- Ranged critical hits can be devastating

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy)]
pub struct LongRangeWeaponsSkill {
    pub level: u8,
    pub experience: u64,
    pub cap: u8,
    pub perks_unlocked: PerkSet,
}

#[derive(Debug, Clone, Copy)]
pub struct PerkSet {
    pub strength_boost: bool,           // Level 20
    pub steady_aim: bool,                // Level 40
    pub archers_precision: bool,         // Level 60
    pub long_shot_mastery: bool,         // Level 80
    pub master_archer: bool,             // Level 100
}

impl LongRangeWeaponsSkill {
    pub fn get_damage_multiplier(&self) -> f32 {
        let skill_bonus = (self.level as i32 - 10).max(0) as f32;
        let base_multiplier = 1.0 + (skill_bonus * 0.005); // 0.5% per level
        
        // Apply perk bonuses
        let mut multiplier = base_multiplier;
        if self.perks_unlocked.master_archer {
            multiplier += 0.10; // +10% from Level 100
        }
        
        multiplier
    }
    
    pub fn get_accuracy_bonus(&self) -> f32 {
        let base_bonus = self.level as f32 * 0.2; // 0.2% per level
        
        let mut bonus = base_bonus;
        if self.perks_unlocked.steady_aim {
            bonus += 5.0; // +5% from Level 40
        }
        if self.perks_unlocked.master_archer {
            bonus += 10.0; // +10% from Level 100
        }
        
        bonus / 100.0 // Convert to decimal
    }
    
    pub fn get_range_multiplier(&self) -> f32 {
        let base_multiplier = 1.0 + (self.level as f32 * 0.002); // 0.2% per level
        
        let mut multiplier = base_multiplier;
        if self.perks_unlocked.long_shot_mastery {
            multiplier += 0.10; // +10% from Level 80
        }
        
        multiplier
    }
    
    pub fn get_range_penalty_reduction(&self) -> f32 {
        let mut reduction = 0.0;
        if self.perks_unlocked.steady_aim {
            reduction += 0.25; // -25% from Level 40
        }
        if self.perks_unlocked.archers_precision {
            reduction += 0.25; // -25% from Level 60
        }
        reduction
    }
    
    pub fn has_close_range_penalty_elimination(&self) -> bool {
        self.perks_unlocked.master_archer
    }
    
    pub fn get_stat_bonuses(&self) -> StatBonuses {
        let mut strength = 0;
        let mut dexterity = 0;
        
        if self.perks_unlocked.strength_boost {
            strength += 2; // Level 20
        }
        if self.perks_unlocked.archers_precision {
            strength += 3; // Level 60
        }
        if self.perks_unlocked.long_shot_mastery {
            dexterity += 2; // Level 80
        }
        if self.perks_unlocked.master_archer {
            dexterity += 3; // Level 100
        }
        
        StatBonuses { strength, dexterity }
    }
    
    pub fn get_critical_hit_bonus(&self) -> f32 {
        if self.perks_unlocked.archers_precision {
            0.03 // +3% critical hit chance (Level 60)
        } else {
            0.0
        }
    }
    
    pub fn check_perk_unlock(&mut self, level: u8) {
        match level {
            20 => self.perks_unlocked.strength_boost = true,
            40 => self.perks_unlocked.steady_aim = true,
            60 => self.perks_unlocked.archers_precision = true,
            80 => self.perks_unlocked.long_shot_mastery = true,
            100 => self.perks_unlocked.master_archer = true,
            _ => {}
        }
    }
}
```

### Range Penalty Calculation

```rust
pub fn calculate_range_penalty(
    distance: f32,
    optimal_range: f32,
    skill: &LongRangeWeaponsSkill,
) -> f32 {
    // Close range penalty (too close)
    if distance < optimal_range * 0.5 {
        if skill.has_close_range_penalty_elimination() {
            return 0.0; // Level 100 perk eliminates close-range penalty
        }
        return 0.05; // -5% accuracy when too close
    }
    
    // Optimal range (no penalty)
    if distance <= optimal_range {
        return 0.0;
    }
    
    // Long range penalty
    let excess_range = distance - optimal_range;
    let base_penalty = excess_range * 0.005; // 0.5% per unit beyond optimal
    
    // Apply perk reduction
    let reduction = skill.get_range_penalty_reduction();
    let final_penalty = base_penalty * (1.0 - reduction);
    
    final_penalty.min(0.5) // Cap at -50% accuracy
}
```

## Summary

Long Range Weapons is a specialized combat skill that:

- **Progresses through combat**: Gain XP by attacking with ranged weapons
- **Amplifies damage**: Up to +45% damage at level 100, +55% with perks
- **Enhances accuracy**: Reduces damage reduction from enemy evasion and range penalties (up to +20% at level 100)
- **Increases range**: Up to +20% range at level 100, +30% with perks
- **Level range**: 0 to 100 (maximum skill level)
- **No miss system**: All attacks hit, but damage can be reduced
- **Progressive perks**: Unlocks powerful bonuses at levels 20, 40, 60, 80, and 100
- **Stat bonuses**: Grants +5 Strength and +5 Dexterity from perks
- **Range mastery**: Eliminates close-range penalty and reduces long-range penalty
- **Build diversity**: Supports Dexterity-focused, Strength-focused, and balanced builds

The skill is essential for any player using ranged weapons and provides significant combat advantages through damage amplification, accuracy enhancement, and range mastery.

