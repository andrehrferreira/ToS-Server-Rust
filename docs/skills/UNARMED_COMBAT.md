# Unarmed Combat Skill Documentation

## Overview

Unarmed Combat is a combat skill that represents proficiency in hand-to-hand combat without weapons. This skill amplifies damage and accuracy when fighting unarmed, making it essential for monks, brawlers, and unarmed-focused builds. The skill progresses through combat experience gained by attacking without weapons, and unlocks powerful perks at specific levels that enhance secondary statistics and unarmed combat effectiveness.

## Key Features

- **Weapon Type**: None (unarmed attacks)
- **Primary Stat**: Strength
- **Secondary Stat**: Dexterity
- **Damage Amplification**: Increases damage output with unarmed attacks
- **Accuracy Enhancement**: Improves hit chance and reduces damage reduction
- **No Miss System**: Attacks always hit but damage can be reduced significantly
- **Progressive Perks**: Unlocks perks at levels 20, 40, 60, 80, and 100
- **Secondary Stat Bonuses**: Perks provide bonuses to Dexterity and other secondary stats
- **Unique Playstyle**: No weapon requirements, always ready to fight
- **Level Range**: 0 to 100 (maximum skill level)

## Skill Progression

### Experience Gain

**Combat-Based Progression:**
- Gain XP by attacking without any weapon equipped
- Base XP gain: 3 XP per attack
- XP scales with combat effectiveness
- Successful hits grant more XP than reduced-damage hits
- Critical hits grant bonus XP
- Combo attacks grant bonus XP

**Attacks That Progress This Skill:**
- **Unarmed punches** (all types)
- **Unarmed kicks** (all types)
- **Unarmed grapples** (if applicable)
- **Unarmed special attacks** (if applicable)

**Attacks That Do NOT Progress This Skill:**
- One-handed weapon attacks (progress Combat With Weapons skill)
- Two-handed weapon attacks (progress Two Handed Weapons skill)
- Ranged weapon attacks (progress Long Range Weapons skill)

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
Base Unarmed Damage = (Strength × 5) + (Dexterity × 3)
Skill Bonus = max(0, Skill Level - 10)
Damage Multiplier = 1.0 + (Skill Bonus × 0.005)
Final Damage = Base Unarmed Damage × Damage Multiplier
```

**Note:** Unarmed combat uses attribute-based damage instead of weapon dice, making it scale directly with player attributes. Damage bonus scales at 0.5% per level after level 10, providing up to +45% damage at level 100.

**Example Calculations:**
- **Level 0-10 (Strength: 50, Dexterity: 30):**
  - Base Damage: (50 × 5) + (30 × 3) = 250 + 90 = 340
  - Skill Bonus: 0%
  - Final Damage: 340

- **Level 50 (Strength: 50, Dexterity: 30):**
  - Base Damage: 340
  - Skill Bonus: +20% (1.20x multiplier)
  - Final Damage: 408

- **Level 100 (Strength: 50, Dexterity: 30):**
  - Base Damage: 340
  - Skill Bonus: +45% (1.45x multiplier)
  - Final Damage: 493

**Damage Application:**
- Bonus applies to all damage dealt with unarmed attacks
- Multiplies final damage after all other calculations
- Works with physical damage (unarmed attacks are physical)
- Stacks multiplicatively with other damage bonuses
- Scales with both Strength and Dexterity

### Accuracy Enhancement

**Accuracy System:**
- No traditional "miss" mechanic
- All attacks connect with the target
- Low accuracy results in significant damage reduction instead
- Higher skill level reduces damage reduction penalty
- Unarmed attacks are fast but require skill for effectiveness

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
- Fast attack speed compensates for lower base damage

## Perks System

### Perk Unlock Levels

Perks are unlocked at specific skill levels, providing permanent bonuses and enhancements:

- **Level 20**: First Perk - Dexterity Boost
- **Level 40**: Second Perk - Flurry of Blows
- **Level 60**: Third Perk - Martial Arts Mastery
- **Level 80**: Fourth Perk - Iron Fist
- **Level 100**: Fifth Perk - Master Brawler

### Level 20 Perk: Dexterity Boost

**Effect:**
- Grants +2 Dexterity permanently
- Improves secondary stat for this skill
- Enhances accuracy and critical hit chance
- Synergizes with Dexterity-based builds

**Benefits:**
- +2 Dexterity = +2% Critical Hit Chance
- +2 Dexterity = +2% Evasion
- +2 Dexterity = +6 unarmed damage (from attribute scaling)
- Better synergy with Dexterity-scaling equipment

### Level 40 Perk: Flurry of Blows

**Effect:**
- Reduces damage reduction by additional 5%
- Increases attack speed by 5%
- Improves accuracy against evasive enemies
- Makes unarmed attacks more fluid

**Benefits:**
- Attacks deal more damage even against high-evasion targets
- Faster attack rate increases DPS
- More reliable damage output
- Better performance in PvP against evasion builds

### Level 60 Perk: Martial Arts Mastery

**Effect:**
- Grants +3 Dexterity permanently
- Increases critical hit chance by 3%
- Improves attack speed by additional 5%
- Enhances combat fluidity and combo potential

**Benefits:**
- Total +5 Dexterity from perks (Level 20 + Level 60)
- Better critical hit potential
- Total +10% attack speed (Level 40 + Level 60)
- More dynamic combat style
- Better combo execution

### Level 80 Perk: Iron Fist

**Effect:**
- Grants +2 Strength permanently
- Increases damage bonus by additional 5%
- Improves overall combat effectiveness
- Enhances both offense and defense

**Benefits:**
- +2 Strength = +2% Physical Damage
- +2 Strength = +20 HP
- +2 Strength = +10 unarmed damage (from attribute scaling)
- Additional 5% damage on top of skill bonus
- More well-rounded combat capabilities

### Level 100 Perk: Master Brawler

**Effect:**
- Grants +3 Strength permanently
- Increases all damage by additional 10%
- Reduces damage reduction by additional 10%
- Increases attack speed by additional 5%
- Ultimate unarmed combat mastery

**Benefits:**
- Total +5 Strength from perks (Level 80 + Level 100)
- Total +15% attack speed (Level 40 + Level 60 + Level 100)
- Significant damage amplification
- Maximum accuracy and consistency
- Peak performance for unarmed builds
- Fastest attack speed of all combat styles

## Complete Perk Summary

**Total Stat Bonuses from Perks:**
- **Dexterity**: +5 (Level 20: +2, Level 60: +3)
- **Strength**: +5 (Level 80: +2, Level 100: +3)

**Total Combat Bonuses:**
- **Damage Reduction Reduction**: -15% (Level 40: -5%, Level 100: -10%)
- **Critical Hit Chance**: +3% (Level 60)
- **Attack Speed**: +15% (Level 40: +5%, Level 60: +5%, Level 100: +5%)
- **Additional Damage**: +15% (Level 80: +5%, Level 100: +10%)

## Build Recommendations

### Dexterity-Focused Build

**Attributes:**
- High Dexterity (primary)
- Moderate Strength (secondary)
- Moderate Constitution (agile brawler)

**Benefits:**
- Maximum critical hit chance
- Excellent evasion
- Fastest attack speed
- Perks enhance Dexterity further
- High DPS through speed and crits

### Strength-Focused Build

**Attributes:**
- High Strength (primary)
- Moderate Dexterity (secondary)
- High Constitution (tanky brawler)

**Benefits:**
- Maximum physical damage output
- Good HP pool
- Strong unarmed damage
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
- Good damage and speed balance
- Adaptable to different situations
- Well-rounded unarmed fighter

## Integration with Other Systems

### Equipment System

**Weapon Requirements:**
- No weapon required (unarmed)
- Cannot use weapons while using unarmed combat
- Can use armor and accessories
- Gloves can enhance unarmed damage (if applicable)

**Equipment Bonuses:**
- Skill bonuses stack with equipment bonuses
- Glove-specific bonuses enhance skill effectiveness
- Set bonuses can synergize with skill perks
- No weapon means more equipment slots available

### Combat System

**Damage Calculation:**
```
Base Unarmed Damage = (Strength × 5) + (Dexterity × 3)
Attribute Damage = Additional attribute bonuses
Skill Bonus = Skill Level Bonus × Base Unarmed Damage
Equipment Bonus = Equipment Modifiers × Base Unarmed Damage
Final Damage = (Base Unarmed Damage + Attribute Damage + Skill Bonus + Equipment Bonus) × Accuracy Multiplier
```

**Accuracy Calculation:**
```
Base Accuracy = 100% - (Enemy Evasion × 0.5)
Skill Accuracy = Skill Level × 2%
Perk Accuracy = Perk Bonuses
Final Accuracy = Base Accuracy + Skill Accuracy + Perk Accuracy
Damage Reduction = max(0, 100% - Final Accuracy)
```

**Attack Speed Calculation:**
```
Base Attack Speed = 1.0 (normal speed)
Perk Attack Speed Bonus = Perk Bonuses
Final Attack Speed = Base Attack Speed × (1.0 + Perk Attack Speed Bonus)
```

### Secondary Stats System

**Evasion Integration:**
- Dexterity from perks improves evasion
- Higher evasion = better survivability
- Synergizes with evasion-focused builds
- Important for unarmed builds (no weapon or shield)

**Critical Hit Integration:**
- Dexterity from perks improves critical hit chance
- Level 6 perk provides additional critical hit chance
- Critical hits deal significantly more damage
- Fast attack speed = more critical hit opportunities

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy)]
pub struct UnarmedCombatSkill {
    pub level: u8,
    pub experience: u64,
    pub cap: u8,
    pub perks_unlocked: PerkSet,
}

#[derive(Debug, Clone, Copy)]
pub struct PerkSet {
    pub dexterity_boost: bool,        // Level 20
    pub flurry_of_blows: bool,        // Level 40
    pub martial_arts_mastery: bool,   // Level 60
    pub iron_fist: bool,               // Level 80
    pub master_brawler: bool,         // Level 100
}

impl UnarmedCombatSkill {
    pub fn calculate_base_unarmed_damage(&self, strength: u16, dexterity: u16) -> u32 {
        (strength as u32 * 5) + (dexterity as u32 * 3)
    }
    
    pub fn get_damage_multiplier(&self) -> f32 {
        let skill_bonus = (self.level as i32 - 10).max(0) as f32;
        let base_multiplier = 1.0 + (skill_bonus * 0.005); // 0.5% per level
        
        // Apply perk bonuses
        let mut multiplier = base_multiplier;
        if self.perks_unlocked.iron_fist {
            multiplier += 0.05; // +5% from Level 80
        }
        if self.perks_unlocked.master_brawler {
            multiplier += 0.10; // +10% from Level 100
        }
        
        multiplier
    }
    
    pub fn get_accuracy_bonus(&self) -> f32 {
        let base_bonus = self.level as f32 * 0.2; // 0.2% per level
        
        let mut bonus = base_bonus;
        if self.perks_unlocked.flurry_of_blows {
            bonus += 5.0; // +5% from Level 40
        }
        if self.perks_unlocked.master_brawler {
            bonus += 10.0; // +10% from Level 100
        }
        
        bonus / 100.0 // Convert to decimal
    }
    
    pub fn get_attack_speed_multiplier(&self) -> f32 {
        let mut multiplier = 1.0;
        
        if self.perks_unlocked.flurry_of_blows {
            multiplier += 0.05; // +5% from Level 40
        }
        if self.perks_unlocked.martial_arts_mastery {
            multiplier += 0.05; // +5% from Level 60
        }
        if self.perks_unlocked.master_brawler {
            multiplier += 0.05; // +5% from Level 100
        }
        
        multiplier // Total: +15% attack speed
    }
    
    pub fn get_stat_bonuses(&self) -> StatBonuses {
        let mut strength = 0;
        let mut dexterity = 0;
        
        if self.perks_unlocked.dexterity_boost {
            dexterity += 2; // Level 20
        }
        if self.perks_unlocked.martial_arts_mastery {
            dexterity += 3; // Level 60
        }
        if self.perks_unlocked.iron_fist {
            strength += 2; // Level 80
        }
        if self.perks_unlocked.master_brawler {
            strength += 3; // Level 100
        }
        
        StatBonuses { strength, dexterity }
    }
    
    pub fn get_critical_hit_bonus(&self) -> f32 {
        if self.perks_unlocked.martial_arts_mastery {
            0.03 // +3% critical hit chance (Level 60)
        } else {
            0.0
        }
    }
    
    pub fn check_perk_unlock(&mut self, level: u8) {
        match level {
            20 => self.perks_unlocked.dexterity_boost = true,
            40 => self.perks_unlocked.flurry_of_blows = true,
            60 => self.perks_unlocked.martial_arts_mastery = true,
            80 => self.perks_unlocked.iron_fist = true,
            100 => self.perks_unlocked.master_brawler = true,
            _ => {}
        }
    }
}
```

### Experience Gain

```rust
pub fn gain_combat_experience(
    skill: &mut UnarmedCombatSkill,
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
    
    // Bonus XP for combo attacks (unarmed special)
    if attack_result.combo {
        xp_gain += 1;
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
pub fn calculate_unarmed_damage(
    strength: u16,
    dexterity: u16,
    skill: &UnarmedCombatSkill,
    equipment_bonus: f32,
    enemy_evasion: f32,
) -> DamageResult {
    // Base unarmed damage (attribute-based)
    let base_damage = skill.calculate_base_unarmed_damage(strength, dexterity);
    
    // Apply skill multiplier
    let skill_multiplier = skill.get_damage_multiplier();
    let skill_bonus = base_damage as f32 * (skill_multiplier - 1.0);
    
    let total_damage = base_damage as f32 + skill_bonus;
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

Unarmed Combat is a unique combat skill that:

- **Progresses through combat**: Gain XP by attacking without weapons
- **Amplifies damage**: Up to +45% damage at level 100, +60% with perks
- **Enhances accuracy**: Reduces damage reduction from enemy evasion (up to +20% at level 100)
- **Increases attack speed**: Up to +15% attack speed with perks
- **No miss system**: All attacks hit, but damage can be reduced
- **Progressive perks**: Unlocks powerful bonuses at levels 20, 40, 60, 80, and 100
- **Stat bonuses**: Grants +5 Strength and +5 Dexterity from perks
- **Attribute-based damage**: Scales directly with Strength and Dexterity
- **Fastest attack speed**: Highest attack speed of all combat skills
- **No weapon requirement**: Always ready to fight, no weapon needed
- **Build diversity**: Supports Dexterity-focused, Strength-focused, and balanced builds
- **Level range**: 0 to 100 (maximum skill level)

The skill is essential for any player using unarmed combat and provides significant combat advantages through damage amplification, accuracy enhancement, and superior attack speed. It offers a unique playstyle that doesn't rely on weapons, making it accessible and versatile.

