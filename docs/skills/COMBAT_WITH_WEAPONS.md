# Combat With Weapons Skill Documentation

## Overview

Combat With Weapons is a combat skill that represents proficiency with one-handed melee weapons. This skill amplifies damage and accuracy when wielding one-handed weapons, making it essential for warriors, rogues, and other melee-focused builds. The skill progresses through combat experience gained by attacking with appropriate weapons, and unlocks powerful perks at specific levels that enhance secondary statistics.

## Key Features

- **Weapon Types**: Axe, Dagger, Sword, Hammer (one-handed)
- **Primary Stat**: Strength
- **Secondary Stat**: Dexterity
- **Damage Amplification**: Increases damage output with one-handed weapons
- **Accuracy Enhancement**: Improves hit chance and reduces damage reduction
- **No Miss System**: Attacks always hit but damage can be reduced significantly
- **Progressive Perks**: Unlocks perks at levels 20, 40, 60, 80, and 100
- **Secondary Stat Bonuses**: Perks provide bonuses to Dexterity and other secondary stats
- **Level Range**: 0 to 100 (maximum skill level)

## Skill Progression

### Experience Gain

**Combat-Based Progression:**
- Gain XP by attacking with one-handed melee weapons
- Base XP gain: 3 XP per attack
- XP scales with combat effectiveness
- Successful hits grant more XP than reduced-damage hits
- Critical hits grant bonus XP

**Weapons That Progress This Skill:**
- **Axe** (one-handed)
- **Dagger** (one-handed)
- **Sword** (one-handed)
- **Hammer** (one-handed)

**Weapons That Do NOT Progress This Skill:**
- Two-handed weapons (progress Two Handed Weapons skill)
- Ranged weapons (progress Long Range Weapons skill)
- Unarmed attacks (progress Unarmed Combat skill)

### Level Requirements

**Level Progression:**
- Level 0: No bonus (base effectiveness)
- Level 1-10: Small bonuses begin
- Level 11-50: Moderate damage and accuracy bonuses
- Level 51-100: Significant damage and accuracy bonuses
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
- Bonus applies to all damage dealt with one-handed weapons
- Multiplies final damage after all other calculations
- Works with physical and elemental damage (if weapon has elemental properties)
- Stacks multiplicatively with other damage bonuses

### Accuracy Enhancement

**Accuracy System:**
- No traditional "miss" mechanic
- All attacks connect with the target
- Low accuracy results in significant damage reduction instead
- Higher skill level reduces damage reduction penalty

**Damage Reduction Formula:**
```
Base Accuracy = 100% - (Enemy Evasion × 0.5)
Skill Accuracy Bonus = Skill Level × 0.2%
Final Accuracy = Base Accuracy + Skill Accuracy Bonus
Damage Reduction = max(0, 100% - Final Accuracy)
Final Damage = Base Damage × (1.0 - Damage Reduction)
```

**Note:** Accuracy bonus scales at 0.2% per level, providing up to +20% accuracy at level 100.

**Example Scenarios:**
- **Level 0 vs High Evasion Enemy:**
  - Base Accuracy: 50%
  - Skill Bonus: 0%
  - Final Accuracy: 50%
  - Damage Reduction: 50%
  - Result: 50% of damage dealt

- **Level 50 vs High Evasion Enemy:**
  - Base Accuracy: 50%
  - Skill Bonus: 10%
  - Final Accuracy: 60%
  - Damage Reduction: 40%
  - Result: 60% of damage dealt

- **Level 100 vs High Evasion Enemy:**
  - Base Accuracy: 50%
  - Skill Bonus: 20%
  - Final Accuracy: 70%
  - Damage Reduction: 30%
  - Result: 70% of damage dealt

**Accuracy Benefits:**
- Reduces damage reduction from enemy evasion
- Makes attacks more consistent and reliable
- Essential for dealing with high-evasion enemies
- Improves overall DPS through consistency

## Perks System

### Perk Unlock Levels

Perks are unlocked at specific skill levels, providing permanent bonuses and enhancements:

- **Level 20**: First Perk - Dexterity Boost
- **Level 40**: Second Perk - Precision Strike
- **Level 60**: Third Perk - Weapon Mastery
- **Level 80**: Fourth Perk - Combat Expertise
- **Level 100**: Fifth Perk - Master Warrior

### Level 20 Perk: Dexterity Boost

**Effect:**
- Grants +2 Dexterity permanently
- Improves secondary stat for this skill
- Enhances accuracy and critical hit chance
- Synergizes with Dexterity-based builds

**Benefits:**
- +2 Dexterity = +2% Critical Hit Chance
- +2 Dexterity = +2% Evasion
- +2 Dexterity = +2% Attack Speed (if applicable)
- Better synergy with Dexterity-scaling equipment

### Level 40 Perk: Precision Strike

**Effect:**
- Reduces damage reduction by additional 5%
- Improves accuracy against evasive enemies
- Makes attacks more consistent
- Enhances overall DPS

**Benefits:**
- Attacks deal more damage even against high-evasion targets
- More reliable damage output
- Better performance in PvP against evasion builds
- Reduces RNG variance in combat

### Level 60 Perk: Weapon Mastery

**Effect:**
- Grants +3 Dexterity permanently
- Increases critical hit chance by 2%
- Improves weapon handling and speed
- Enhances combat fluidity

**Benefits:**
- Total +5 Dexterity from perks (Level 20 + Level 60)
- Better critical hit potential
- Improved evasion and mobility
- More dynamic combat style

### Level 80 Perk: Combat Expertise

**Effect:**
- Grants +2 Strength permanently
- Increases damage bonus by additional 5%
- Improves overall combat effectiveness
- Enhances both offense and defense

**Benefits:**
- +2 Strength = +2% Physical Damage
- +2 Strength = +20 HP
- Additional 5% damage on top of skill bonus
- More well-rounded combat capabilities

### Level 100 Perk: Master Warrior

**Effect:**
- Grants +3 Strength permanently
- Increases all damage by additional 10%
- Reduces damage reduction by additional 10%
- Ultimate combat mastery

**Benefits:**
- Total +5 Strength from perks (Level 8 + Level 10)
- Significant damage amplification
- Maximum accuracy and consistency
- Peak performance for one-handed weapon builds

## Complete Perk Summary

**Total Stat Bonuses from Perks:**
- **Dexterity**: +5 (Level 20: +2, Level 60: +3)
- **Strength**: +5 (Level 80: +2, Level 100: +3)

**Total Combat Bonuses:**
- **Damage Reduction Reduction**: -15% (Level 40: -5%, Level 100: -10%)
- **Critical Hit Chance**: +2% (Level 60)
- **Additional Damage**: +15% (Level 80: +5%, Level 100: +10%)

## Build Recommendations

### Strength-Focused Build

**Attributes:**
- High Strength (primary)
- Moderate Dexterity (secondary)
- Moderate Constitution (survivability)

**Benefits:**
- Maximum physical damage output
- Good HP pool
- Strong with heavy one-handed weapons (hammers, axes)
- Perks enhance Strength further

### Dexterity-Focused Build

**Attributes:**
- High Dexterity (primary)
- Moderate Strength (secondary)
- High Constitution (evasion tank)

**Benefits:**
- High critical hit chance
- Excellent evasion
- Fast attack speed
- Perks provide Dexterity bonuses
- Great with daggers and swords

### Balanced Build

**Attributes:**
- Balanced Strength and Dexterity
- Moderate Constitution
- Some Intelligence (for hybrid builds)

**Benefits:**
- Benefits from all perks equally
- Versatile combat style
- Good with all weapon types
- Adaptable to different situations

## Integration with Other Systems

### Equipment System

**Weapon Requirements:**
- Skill level may be required for certain weapons
- Higher-tier weapons may require minimum skill level
- Skill level affects weapon effectiveness

**Equipment Bonuses:**
- Skill bonuses stack with equipment bonuses
- Weapon-specific bonuses enhance skill effectiveness
- Set bonuses can synergize with skill perks

### Combat System

**Damage Calculation:**
```
Base Damage = Equipment Dice Roll × 10
Attribute Damage = (Strength × 10) + (Dexterity × 5)
Skill Bonus = Skill Level Bonus × Base Damage
Equipment Bonus = Equipment Modifiers × Base Damage
Final Damage = (Base Damage + Attribute Damage + Skill Bonus + Equipment Bonus) × Accuracy Multiplier
```

**Accuracy Calculation:**
```
Base Accuracy = 100% - (Enemy Evasion × 0.5)
Skill Accuracy = Skill Level × 2%
Perk Accuracy = Perk Bonuses
Final Accuracy = Base Accuracy + Skill Accuracy + Perk Accuracy
Damage Reduction = max(0, 100% - Final Accuracy)
```

### Secondary Stats System

**Evasion Integration:**
- Dexterity from perks improves evasion
- Higher evasion = better survivability
- Synergizes with evasion-focused builds

**Critical Hit Integration:**
- Dexterity from perks improves critical hit chance
- Level 6 perk provides additional critical hit chance
- Critical hits deal significantly more damage

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy)]
pub struct CombatWithWeaponsSkill {
    pub level: u8,
    pub experience: u64,
    pub cap: u8,
    pub perks_unlocked: PerkSet,
}

#[derive(Debug, Clone, Copy)]
pub struct PerkSet {
    pub dexterity_boost: bool,        // Level 20
    pub precision_strike: bool,        // Level 40
    pub weapon_mastery: bool,          // Level 60
    pub combat_expertise: bool,        // Level 80
    pub master_warrior: bool,          // Level 100
}

impl CombatWithWeaponsSkill {
    pub fn get_damage_multiplier(&self) -> f32 {
        let skill_bonus = (self.level as i32 - 10).max(0) as f32;
        let base_multiplier = 1.0 + (skill_bonus * 0.005); // 0.5% per level
        
        // Apply perk bonuses
        let mut multiplier = base_multiplier;
        if self.perks_unlocked.combat_expertise {
            multiplier += 0.05; // +5% from Level 80
        }
        if self.perks_unlocked.master_warrior {
            multiplier += 0.10; // +10% from Level 100
        }
        
        multiplier
    }
    
    pub fn get_accuracy_bonus(&self) -> f32 {
        let base_bonus = self.level as f32 * 0.2; // 0.2% per level
        
        let mut bonus = base_bonus;
        if self.perks_unlocked.precision_strike {
            bonus += 5.0; // +5% from Level 40
        }
        if self.perks_unlocked.master_warrior {
            bonus += 10.0; // +10% from Level 100
        }
        
        bonus / 100.0 // Convert to decimal
    }
    
    pub fn get_stat_bonuses(&self) -> StatBonuses {
        let mut strength = 0;
        let mut dexterity = 0;
        
        if self.perks_unlocked.dexterity_boost {
            dexterity += 2;
        }
        if self.perks_unlocked.weapon_mastery {
            dexterity += 3;
        }
        if self.perks_unlocked.combat_expertise {
            strength += 2;
        }
        if self.perks_unlocked.master_warrior {
            strength += 3;
        }
        
        StatBonuses { strength, dexterity }
    }
    
    pub fn get_critical_hit_bonus(&self) -> f32 {
        if self.perks_unlocked.weapon_mastery {
            0.02 // +2% critical hit chance
        } else {
            0.0
        }
    }
    
    pub fn check_perk_unlock(&mut self, level: u8) {
        match level {
            20 => self.perks_unlocked.dexterity_boost = true,
            40 => self.perks_unlocked.precision_strike = true,
            60 => self.perks_unlocked.weapon_mastery = true,
            80 => self.perks_unlocked.combat_expertise = true,
            100 => self.perks_unlocked.master_warrior = true,
            _ => {}
        }
    }
}
```

### Experience Gain

```rust
pub fn gain_combat_experience(
    skill: &mut CombatWithWeaponsSkill,
    attack_result: AttackResult,
) {
    let base_xp = 3;
    let mut xp_gain = base_xp;
    
    // Bonus XP for successful hits
    if attack_result.hit {
        xp_gain += 1;
    }
    
    // Bonus XP for critical hits
    if attack_result.critical {
        xp_gain += 2;
    }
    
    // Bonus XP for reduced-damage hits (learning from mistakes)
    if attack_result.damage_reduction > 0.5 {
        xp_gain += 1; // Learning from near-misses
    }
    
    skill.experience += xp_gain as u64;
    
    // Check for level up
    let new_level = calculate_level_from_experience(skill.experience);
    if new_level > skill.level && new_level <= skill.cap {
        let old_level = skill.level;
        skill.level = new_level;
        
        // Unlock perks
        skill.check_perk_unlock(new_level);
        
        // Notify player of level up and perk unlock
        notify_skill_level_up(old_level, new_level);
    }
}
```

### Damage Calculation Integration

```rust
pub fn calculate_one_handed_weapon_damage(
    base_damage: u32,
    strength: u16,
    dexterity: u16,
    skill: &CombatWithWeaponsSkill,
    equipment_bonus: f32,
    enemy_evasion: f32,
) -> DamageResult {
    // Base damage calculation
    let attribute_damage = (strength as f32 * 10.0) + (dexterity as f32 * 5.0);
    let skill_multiplier = skill.get_damage_multiplier();
    let skill_bonus = base_damage as f32 * (skill_multiplier - 1.0);
    
    let total_damage = base_damage as f32 + attribute_damage + skill_bonus;
    let final_damage = total_damage * equipment_bonus;
    
    // Accuracy calculation (no miss, but damage reduction)
    let base_accuracy = 1.0 - (enemy_evasion * 0.5);
    let skill_accuracy = skill.get_accuracy_bonus();
    let final_accuracy = (base_accuracy + skill_accuracy).min(1.0);
    
    let damage_reduction = 1.0 - final_accuracy;
    let actual_damage = final_damage * final_accuracy;
    
    DamageResult {
        base_damage: final_damage as u32,
        accuracy: final_accuracy,
        damage_reduction,
        actual_damage: actual_damage as u32,
        hit: true, // Always hits, but damage may be reduced
        critical: false, // Calculated separately
    }
}
```

## Summary

Combat With Weapons is a fundamental combat skill that:

- **Progresses through combat**: Gain XP by attacking with one-handed weapons
- **Amplifies damage**: Up to +45% damage at level 100, +60% with perks
- **Enhances accuracy**: Reduces damage reduction from enemy evasion (up to +20% at level 100)
- **No miss system**: All attacks hit, but damage can be reduced
- **Progressive perks**: Unlocks powerful bonuses at levels 20, 40, 60, 80, and 100
- **Stat bonuses**: Grants +5 Strength and +5 Dexterity from perks
- **Build diversity**: Supports Strength-focused, Dexterity-focused, and balanced builds
- **Level range**: 0 to 100 (maximum skill level)

The skill is essential for any player using one-handed melee weapons and provides significant combat advantages through both direct bonuses and powerful perk enhancements.

