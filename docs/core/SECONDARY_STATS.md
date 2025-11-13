# Secondary Statistics System Documentation

## Overview

The secondary statistics system defines defensive mechanics that depend directly on primary attributes and equipment. These statistics provide additional layers of defense beyond basic HP and resistances, creating diverse defensive build options. The system includes Evasion, Block, Armor, and Energy Shield, each with distinct mechanics and scaling methods.

**Note:** All values are multiplied by 10 to provide a greater sense of progression, consistent with the damage and status systems.

## Key Features

- **Attribute-Based Scaling**: Secondary stats scale with primary attributes (STR, DEX, INT)
- **Equipment Integration**: Equipment provides bonuses to secondary stats
- **Skill Integration**: Skills can enhance secondary stat effectiveness
- **Balanced Caps**: Maximum values prevent invincibility scenarios
- **Build Diversity**: Different defensive builds create distinct playstyles
- **PvP Balance**: Secondary stats balanced against damage system for fair combat

## Secondary Statistics

### 1. Evasion

**Description:**
Evasion represents the chance to completely avoid taking damage from attacks, both physical and magical. When evasion triggers, the attack deals no damage. Evasion scales primarily with Dexterity and can be enhanced through equipment and skills.

**Primary Attribute:** Dexterity (DEX)

**Formula:**
```
Evasion Chance = Base Evasion + (DEX * DEX_Evasion_Multiplier) + Equipment Bonuses + Skill Bonuses
Final Evasion = min(Evasion Chance, Evasion_Cap)
```

**Current Implementation:**
```
Evasion Chance = 0 + (DEX * 0.1) + Equipment Bonuses + Skill Bonuses
Evasion Cap: 75% (to prevent invincibility)
```

**Components:**
- **Base Evasion**: 0% (starting evasion for all entities)
- **DEX Contribution**: DEX * 0.1% per point (primary evasion attribute)
- **Equipment Bonuses**: Flat evasion bonuses from equipment
- **Skill Bonuses**: Evasion bonuses from skills

**Example Calculations:**
```
Level 1 Player (DEX: 10):
Evasion = 0 + (10 * 0.1) = 1.0%

Level 50 Player (DEX: 50):
Evasion = 0 + (50 * 0.1) = 5.0%

High DEX Build (DEX: 150, +20% from equipment, +10% from skills):
Evasion = 0 + (150 * 0.1) + 20 + 10 = 45.0%

Max Evasion Build (DEX: 200, +40% from equipment, +20% from skills):
Evasion = min(0 + (200 * 0.1) + 40 + 20, 75) = 75.0% (capped)
```

**Balance Considerations:**
- Evasion applies to both physical and magical damage
- High evasion builds rely on avoiding damage rather than mitigating it
- Evasion cap prevents invincibility scenarios
- Evasion is less reliable than mitigation but can completely negate damage
- Evasion builds typically have lower HP pools

**Rust Implementation:**
```rust
pub fn calculate_evasion(
    base_evasion: f32,
    dex: u16,
    equipment_evasion_bonus: f32,
    skill_evasion_bonus: f32,
) -> f32 {
    const EVASION_CAP: f32 = 75.0;
    const DEX_EVASION_MULTIPLIER: f32 = 0.1;
    
    let dex_bonus = dex as f32 * DEX_EVASION_MULTIPLIER;
    let total_evasion = base_evasion + dex_bonus + equipment_evasion_bonus + skill_evasion_bonus;
    
    total_evasion.min(EVASION_CAP)
}

pub fn roll_evasion(evasion_chance: f32) -> bool {
    let roll = rand::random::<f32>() * 100.0;
    roll < evasion_chance
}
```

**Damage Calculation Integration:**
```rust
pub fn apply_evasion(damage: u16, evasion_chance: f32) -> (u16, bool) {
    if roll_evasion(evasion_chance) {
        return (0, true); // Evaded, no damage
    }
    (damage, false) // Not evaded, full damage
}
```

### 2. Block

**Description:**
Block represents the chance to reduce or completely negate physical damage and projectiles. Block requires a shield to be equipped and scales with shield-related skills and equipment attributes. When block triggers, damage is reduced by a percentage based on block effectiveness.

**Requirement:** Shield must be equipped

**Formula:**
```
Block Chance = Base Block + Shield Base Block + (Block Skill * Skill_Multiplier) + Equipment Bonuses
Block Effectiveness = Base Effectiveness + (Block Skill * Effectiveness_Multiplier) + Equipment Bonuses
Final Block Chance = min(Block Chance, Block_Cap)
```

**Current Implementation:**
```
Block Chance = 0 + Shield Base Block + (Block Skill * 0.5) + Equipment Bonuses
Block Cap: 75% (to prevent invincibility)
Block Effectiveness = 30% + (Block Skill * 0.2) + Equipment Bonuses
Max Block Effectiveness: 100% (complete damage negation)
```

**Components:**
- **Base Block**: 0% (no block without shield)
- **Shield Base Block**: Varies by shield type and tier (typically 15-40%)
- **Block Skill**: Skill level affects both chance and effectiveness
- **Equipment Bonuses**: Flat block chance and effectiveness bonuses

**Example Calculations:**
```
Basic Shield (Base Block: 20%, Block Skill: 10):
Block Chance = 0 + 20 + (10 * 0.5) = 25.0%
Block Effectiveness = 30 + (10 * 0.2) = 32.0%

Advanced Shield (Base Block: 35%, Block Skill: 50, +10% from equipment):
Block Chance = min(0 + 35 + (50 * 0.5) + 10, 75) = 70.0%
Block Effectiveness = 30 + (50 * 0.2) + 5 = 45.0%

Max Block Build (Base Block: 40%, Block Skill: 70, +20% from equipment):
Block Chance = min(0 + 40 + (70 * 0.5) + 20, 75) = 75.0% (capped)
Block Effectiveness = 30 + (70 * 0.2) + 15 = 59.0%
```

**Balance Considerations:**
- Block only works against physical damage and projectiles
- Requires shield equipment (trade-off with two-handed weapons)
- Block effectiveness determines damage reduction when block triggers
- Block chance and effectiveness scale with skill investment
- High block builds sacrifice offensive potential for defense

**Rust Implementation:**
```rust
pub struct BlockStats {
    pub chance: f32,
    pub effectiveness: f32,
}

pub fn calculate_block(
    shield_base_block: f32,
    block_skill_level: u16,
    equipment_block_chance_bonus: f32,
    equipment_block_effectiveness_bonus: f32,
) -> BlockStats {
    const BLOCK_CAP: f32 = 75.0;
    const SKILL_CHANCE_MULTIPLIER: f32 = 0.5;
    const BASE_EFFECTIVENESS: f32 = 30.0;
    const SKILL_EFFECTIVENESS_MULTIPLIER: f32 = 0.2;
    const MAX_EFFECTIVENESS: f32 = 100.0;
    
    let block_chance = shield_base_block 
        + (block_skill_level as f32 * SKILL_CHANCE_MULTIPLIER)
        + equipment_block_chance_bonus;
    
    let block_effectiveness = BASE_EFFECTIVENESS
        + (block_skill_level as f32 * SKILL_EFFECTIVENESS_MULTIPLIER)
        + equipment_block_effectiveness_bonus;
    
    BlockStats {
        chance: block_chance.min(BLOCK_CAP),
        effectiveness: block_effectiveness.min(MAX_EFFECTIVENESS),
    }
}

pub fn roll_block(block_chance: f32) -> bool {
    let roll = rand::random::<f32>() * 100.0;
    roll < block_chance
}

pub fn apply_block(damage: u16, block_effectiveness: f32) -> u16 {
    let damage_reduction = damage as f32 * (block_effectiveness / 100.0);
    (damage as f32 - damage_reduction) as u16
}
```

**Damage Calculation Integration:**
```rust
pub fn apply_block_to_damage(damage: u16, block_stats: &BlockStats) -> (u16, bool) {
    if roll_block(block_stats.chance) {
        let blocked_damage = apply_block(damage, block_stats.effectiveness);
        return (blocked_damage, true); // Blocked, reduced damage
    }
    (damage, false) // Not blocked, full damage
}
```

### 3. Armor

**Description:**
Armor represents a percentage reduction in physical damage received from attacks and spells. Armor provides consistent damage mitigation that applies to all physical damage sources. The maximum armor value is capped at 90% to prevent invincibility.

**Formula:**
```
Armor = Base Armor + Equipment Armor + Skill Bonuses + Attribute Bonuses
Final Armor = min(Armor, Armor_Cap)
```

**Current Implementation:**
```
Armor = 0 + Equipment Armor + Skill Bonuses + (STR * STR_Armor_Multiplier)
Armor Cap: 90% (maximum physical damage reduction)
```

**Components:**
- **Base Armor**: 0% (starting armor for all entities)
- **Equipment Armor**: Primary source of armor (from armor pieces)
- **Skill Bonuses**: Armor bonuses from defensive skills
- **STR Contribution**: STR * 0.1% per point (secondary armor attribute)

**Example Calculations:**
```
Light Armor Set (Equipment Armor: 15%, STR: 50):
Armor = 0 + 15 + (50 * 0.1) = 20.0%

Medium Armor Set (Equipment Armor: 35%, STR: 75, +5% from skill):
Armor = 0 + 35 + (75 * 0.1) + 5 = 47.5%

Heavy Armor Set (Equipment Armor: 60%, STR: 100, +10% from skill):
Armor = 0 + 60 + (100 * 0.1) + 10 = 80.0%

Max Armor Build (Equipment Armor: 70%, STR: 150, +15% from skill):
Armor = min(0 + 70 + (150 * 0.1) + 15, 90) = 90.0% (capped)
```

**Balance Considerations:**
- Armor only reduces physical damage (not magical)
- Armor cap prevents invincibility scenarios
- Heavy armor provides more armor but reduces mobility
- Armor is consistent mitigation (unlike evasion/block which are chance-based)
- High armor builds typically have lower evasion and mobility

**Rust Implementation:**
```rust
pub fn calculate_armor(
    equipment_armor: f32,
    str: u16,
    skill_armor_bonus: f32,
) -> f32 {
    const ARMOR_CAP: f32 = 90.0;
    const STR_ARMOR_MULTIPLIER: f32 = 0.1;
    
    let str_bonus = str as f32 * STR_ARMOR_MULTIPLIER;
    let total_armor = equipment_armor + str_bonus + skill_armor_bonus;
    
    total_armor.min(ARMOR_CAP)
}

pub fn apply_armor(damage: u16, armor: f32) -> u16 {
    let damage_reduction = damage as f32 * (armor / 100.0);
    (damage as f32 - damage_reduction) as u16
}
```

**Damage Calculation Integration:**
```rust
pub fn apply_armor_to_damage(damage: u16, armor: f32) -> u16 {
    // Armor only applies to physical damage
    apply_armor(damage, armor)
}
```

### 4. Energy Shield

**Description:**
Energy Shield is a protective barrier that functions as a secondary health pool, represented by segmented blue bars surrounding the red health bar. Any damage taken consumes Energy Shield first before HP. Energy Shield recharges over time when not taking damage. Energy Shield scales with Intelligence and can be enhanced through equipment and skills.

**Primary Attribute:** Intelligence (INT)

**Formula:**
```
Max Energy Shield = Base Energy Shield + (INT * INT_Energy_Shield_Multiplier) + Equipment Bonuses + Skill Bonuses
Energy Shield Regen per Tick = Base Regen + (INT * INT_Regen_Multiplier) + Equipment Bonuses + Skill Bonuses
```

**Current Implementation:**
```
Max Energy Shield = 0 + (INT * 20) + Equipment Bonuses + Skill Bonuses
Energy Shield Regen per Tick = 10 + (INT * 0.5) + Equipment Bonuses + Skill Bonuses
Regen Delay: 3 seconds after last damage taken
Tick Interval: 3 seconds
```

**Components:**
- **Base Energy Shield**: 0 (starting energy shield for all entities)
- **INT Contribution**: INT * 20 per point (primary energy shield attribute)
- **Equipment Bonuses**: Flat energy shield bonuses from equipment
- **Skill Bonuses**: Energy shield bonuses from skills
- **Regeneration**: Energy shield regenerates over time when not taking damage

**Example Calculations:**
```
Level 1 Player (INT: 10):
Max Energy Shield = 0 + (10 * 20) = 200
Regen per Tick = 10 + (10 * 0.5) = 15 per 3 seconds

Level 50 Player (INT: 50):
Max Energy Shield = 0 + (50 * 20) = 1,000
Regen per Tick = 10 + (50 * 0.5) = 35 per 3 seconds

High INT Build (INT: 150, +500 from equipment, +200 from skill):
Max Energy Shield = 0 + (150 * 20) + 500 + 200 = 4,000
Regen per Tick = 10 + (150 * 0.5) + 20 + 10 = 115 per 3 seconds
```

**Balance Considerations:**
- Energy Shield absorbs all damage types (physical and magical)
- Energy Shield regenerates automatically when not taking damage
- High Energy Shield builds typically have lower HP pools
- Energy Shield provides effective HP but requires time to regenerate
- Energy Shield is more effective against burst damage than sustained damage

**Rust Implementation:**
```rust
pub struct EnergyShield {
    pub max: u16,
    pub current: u16,
    pub regen_per_tick: u16,
    pub last_damage_tick: u64,
    pub regen_delay_ticks: u64,
}

impl EnergyShield {
    pub fn calculate_max_energy_shield(
        int: u16,
        equipment_bonus: u16,
        skill_bonus: u16,
    ) -> u16 {
        const INT_ENERGY_SHIELD_MULTIPLIER: u16 = 20;
        
        0 + (int * INT_ENERGY_SHIELD_MULTIPLIER) + equipment_bonus + skill_bonus
    }
    
    pub fn calculate_regen_per_tick(
        int: u16,
        equipment_bonus: u16,
        skill_bonus: u16,
    ) -> u16 {
        const BASE_REGEN: u16 = 10;
        const INT_REGEN_MULTIPLIER: f32 = 0.5;
        
        BASE_REGEN + (int as f32 * INT_REGEN_MULTIPLIER) as u16 + equipment_bonus + skill_bonus
    }
    
    pub fn take_damage(&mut self, damage: u16, current_tick: u64) -> u16 {
        self.last_damage_tick = current_tick;
        
        if self.current >= damage {
            self.current -= damage;
            return 0; // All damage absorbed by energy shield
        }
        
        let remaining_damage = damage - self.current;
        self.current = 0;
        remaining_damage // Remaining damage goes to HP
    }
    
    pub fn regenerate(&mut self, current_tick: u64) {
        const REGEN_DELAY_TICKS: u64 = 3; // 3 seconds delay
        
        if current_tick - self.last_damage_tick < REGEN_DELAY_TICKS {
            return; // Still in regen delay period
        }
        
        if self.current < self.max {
            self.current = (self.current + self.regen_per_tick).min(self.max);
        }
    }
}
```

**Damage Calculation Integration:**
```rust
pub fn apply_energy_shield_to_damage(
    damage: u16,
    energy_shield: &mut EnergyShield,
    current_tick: u64,
) -> u16 {
    // Energy shield absorbs damage first
    energy_shield.take_damage(damage, current_tick)
}
```

## Complete Damage Calculation Flow

### Damage Application Order

**Step-by-Step Process:**
1. **Evasion Check**: Roll for evasion chance (reduced by EvasionPenetration)
   - Calculate effective evasion: Evasion - EvasionPenetration
   - If evaded: No damage taken, stop
   - If not evaded: Continue to next step

2. **Block Check** (Physical damage and projectiles only, requires shield):
   - Calculate effective block chance: Block Chance - BlockPenetration
   - Calculate effective block effectiveness: Block Effectiveness - BlockPenetration
   - Roll for block chance
   - If blocked: Reduce damage by effective block effectiveness, continue
   - If not blocked: Continue with full damage

3. **Armor Reduction** (Physical damage only):
   - Calculate effective armor: Armor - ArmorPenetration
   - Apply effective armor percentage reduction
   - Continue with reduced damage

4. **Resistance Reduction** (All damage types):
   - Calculate effective resistance: Resistance - Penetration (PhysicalPenetration or ElementalPenetration or AllResistancePenetration)
   - Apply appropriate effective resistance (physical or elemental)
   - Continue with reduced damage

5. **Energy Shield Absorption** (All damage types):
   - Calculate bypass percentage: EnergyShieldPenetration
   - Bypass damage: Damage * (EnergyShieldPenetration / 100)
   - Shield absorption: Damage * (1 - EnergyShieldPenetration / 100)
   - Absorb shield absorption with energy shield first
   - Remaining damage (bypass + unabsorbed) goes to HP

6. **HP Damage**:
   - Apply remaining damage to HP

**Rust Implementation:**
```rust
pub struct DamageResult {
    pub hp_damage: u16,
    pub energy_shield_damage: u16,
    pub evaded: bool,
    pub blocked: bool,
}

pub struct PenetrationStats {
    pub physical_resistance: f32,
    pub elemental_resistance: f32,
    pub armor: f32,
    pub evasion: f32,
    pub block_chance: f32,
    pub block_effectiveness: f32,
    pub energy_shield: f32,
    pub all_resistance: f32,
    pub defense: f32,
}

pub fn calculate_final_damage(
    base_damage: u16,
    damage_type: DamageType,
    target: &mut EntityDefenses,
    penetration: &PenetrationStats,
    current_tick: u64,
) -> DamageResult {
    let mut damage = base_damage;
    let mut evaded = false;
    let mut blocked = false;
    
    // Step 1: Evasion check (with penetration)
    let effective_evasion = (target.evasion - penetration.evasion - penetration.defense).max(0.0);
    if roll_evasion(effective_evasion) {
        return DamageResult {
            hp_damage: 0,
            energy_shield_damage: 0,
            evaded: true,
            blocked: false,
        };
    }
    
    // Step 2: Block check (physical damage and projectiles only, with penetration)
    if damage_type == DamageType::Physical || damage_type == DamageType::Projectile {
        if target.has_shield {
            let effective_block_chance = (target.block_stats.chance - penetration.block_chance - penetration.defense).max(0.0);
            let effective_block_effectiveness = (target.block_stats.effectiveness - penetration.block_effectiveness - penetration.defense).max(0.0);
            
            if roll_block(effective_block_chance) {
                damage = apply_block(damage, effective_block_effectiveness);
                blocked = true;
            }
        }
    }
    
    // Step 3: Armor reduction (physical damage only, with penetration)
    if damage_type == DamageType::Physical {
        let effective_armor = (target.armor - penetration.armor - penetration.defense).max(0.0);
        damage = apply_armor(damage, effective_armor);
    }
    
    // Step 4: Resistance reduction (with penetration)
    let base_resistance = match damage_type {
        DamageType::Physical => target.physical_resistance,
        DamageType::Fire => target.fire_resistance,
        DamageType::Cold => target.cold_resistance,
        DamageType::Poison => target.poison_resistance,
        DamageType::Energy => target.energy_resistance,
        DamageType::Light => target.light_resistance,
        DamageType::Dark => target.dark_resistance,
        DamageType::Projectile => target.physical_resistance,
    };
    
    let effective_resistance = if damage_type == DamageType::Physical {
        (base_resistance - penetration.physical_resistance - penetration.all_resistance).max(0.0)
    } else {
        (base_resistance - penetration.elemental_resistance - penetration.all_resistance).max(0.0)
    };
    damage = apply_resistance(damage, effective_resistance);
    
    // Step 5: Energy Shield absorption (with penetration)
    let bypass_percentage = penetration.energy_shield;
    let bypass_damage = (damage as f32 * (bypass_percentage / 100.0)) as u16;
    let shield_damage = damage - bypass_damage;
    
    let energy_shield_damage = shield_damage.min(target.energy_shield.current);
    let remaining_shield_damage = target.energy_shield.take_damage(shield_damage, current_tick);
    let total_hp_damage = bypass_damage + remaining_shield_damage;
    
    DamageResult {
        hp_damage: total_hp_damage,
        energy_shield_damage,
        evaded: false,
        blocked,
    }
}
```

## Balance Considerations

### Stat Interactions

**Evasion vs Armor:**
- Evasion: Chance-based, works against all damage types, requires high DEX
- Armor: Consistent mitigation, only physical damage, requires heavy equipment
- High evasion builds typically have low armor
- High armor builds typically have low evasion

**Block vs Armor:**
- Block: Chance-based, requires shield, works against physical and projectiles
- Armor: Consistent mitigation, no equipment requirement, only physical
- Block builds sacrifice two-handed weapons for defense
- Armor builds can use any weapon configuration

**Energy Shield vs HP:**
- Energy Shield: Regenerates automatically, scales with INT, absorbs all damage types
- HP: Does not regenerate automatically, scales with STR/VIG, base health pool
- High Energy Shield builds typically have lower HP
- High HP builds typically have lower Energy Shield

### Build Diversity

**Pure Evasion Build:**
- High DEX, light armor, low HP
- Relies on avoiding damage completely
- Vulnerable when evasion fails

**Pure Armor Build:**
- High STR, heavy armor, moderate HP
- Consistent physical damage reduction
- Vulnerable to magical damage

**Block Build:**
- Shield required, high block skill, moderate HP
- Chance-based physical defense
- Requires skill investment

**Energy Shield Build:**
- High INT, energy shield equipment, low HP
- Regenerating defense pool
- Effective against burst damage

**Hybrid Builds:**
- Balanced secondary stats
- More versatile but less specialized
- Adaptable to different situations

## Integration with Other Systems

### Status System

**Integration:**
- DEX affects evasion chance
- STR affects armor value
- INT affects energy shield maximum and regeneration
- See [STATUS.md](./STATUS.md) for primary attribute details

### Damage System

**Integration:**
- Secondary stats modify damage before HP application
- Evasion can completely negate damage
- Block reduces physical damage
- Armor reduces physical damage
- Energy Shield absorbs damage before HP
- Penetration attributes reduce effectiveness of all secondary stats
- See [DAMAGE.md](./DAMAGE.md) for damage calculation details

### Penetration System

**Integration:**
- Penetration attributes are essential for DPS builds
- PhysicalPenetration reduces physical resistance effectiveness
- ElementalPenetration reduces elemental resistance effectiveness
- ArmorPenetration reduces armor effectiveness
- EvasionPenetration reduces evasion chance
- BlockPenetration reduces block chance and effectiveness
- EnergyShieldPenetration allows damage to bypass energy shield
- AllResistancePenetration reduces all resistances (physical and elemental)
- DefensePenetration reduces all defensive stats (armor, evasion, block)
- See [ATTRIBUTES.md](../items/ATTRIBUTES.md) for penetration attribute details

### Resistance System

**Integration:**
- Secondary stats apply before resistance calculations
- Evasion works against all damage types
- Block works against physical damage
- Armor works against physical damage
- Energy Shield absorbs all damage types
- See [RESISTANCES.md](./RESISTANCES.md) for resistance details

### Equipment System

**Integration:**
- Equipment provides bonuses to secondary stats
- Shields enable block mechanics
- Armor pieces provide armor values
- Equipment can enhance energy shield
- Equipment attributes affect secondary stat scaling

## Rust Implementation Considerations

### Data Structures

**Secondary Stats Component:**
```rust
#[derive(Component, Clone)]
pub struct SecondaryStats {
    pub evasion: f32,
    pub block: Option<BlockStats>, // None if no shield equipped
    pub armor: f32,
    pub energy_shield: EnergyShield,
}

#[derive(Clone, Copy)]
pub struct BlockStats {
    pub chance: f32,
    pub effectiveness: f32,
}

#[derive(Clone)]
pub struct EnergyShield {
    pub max: u16,
    pub current: u16,
    pub regen_per_tick: u16,
    pub last_damage_tick: u64,
}
```

### Performance Optimizations

**Caching:**
- Cache calculated secondary stats until attributes/equipment change
- Pre-calculate evasion, block, armor values
- Cache energy shield regeneration calculations

**Batch Processing:**
- Calculate secondary stats for multiple entities in parallel
- Batch energy shield regeneration updates
- Pre-calculate damage reduction values

**Memory Layout:**
- Store secondary stats in compact format
- Use Option<BlockStats> to save memory when no shield equipped
- Align data structures for cache efficiency

### Serialization

**Secondary Stats Serialization:**
```rust
impl Serialize for SecondaryStats {
    fn serialize(&self) -> Vec<u8> {
        let mut buffer = Vec::with_capacity(32);
        
        buffer.extend_from_slice(&self.evasion.to_le_bytes());
        
        if let Some(block) = self.block {
            buffer.push(1); // Has block
            buffer.extend_from_slice(&block.chance.to_le_bytes());
            buffer.extend_from_slice(&block.effectiveness.to_le_bytes());
        } else {
            buffer.push(0); // No block
        }
        
        buffer.extend_from_slice(&self.armor.to_le_bytes());
        buffer.extend_from_slice(&self.energy_shield.max.to_le_bytes());
        buffer.extend_from_slice(&self.energy_shield.current.to_le_bytes());
        buffer.extend_from_slice(&self.energy_shield.regen_per_tick.to_le_bytes());
        
        buffer
    }
}
```

## Testing Considerations

### Unit Tests

**Test Cases:**
- Evasion calculation with various DEX values
- Block calculation with various shield and skill levels
- Armor calculation with various equipment and STR values
- Energy Shield calculation with various INT values
- Energy Shield regeneration timing
- Stat caps enforcement

### Integration Tests

**Test Cases:**
- Complete damage calculation flow with all secondary stats
- Evasion triggering correctly
- Block triggering correctly
- Armor reducing damage correctly
- Energy Shield absorbing damage correctly
- Energy Shield regenerating after delay
- Stat recalculation on attribute change
- Stat recalculation on equipment change

### Balance Tests

**Test Cases:**
- Evasion effectiveness vs damage output
- Block effectiveness vs damage output
- Armor effectiveness vs damage output
- Energy Shield effectiveness vs damage output
- Build diversity (different secondary stat distributions)
- PvP balance with secondary stats

### Performance Tests

**Test Cases:**
- Secondary stat calculation performance
- Damage calculation with secondary stats performance
- Energy Shield regeneration performance
- Batch processing performance

## Summary

The secondary statistics system:

- Provides four defensive mechanics: Evasion, Block, Armor, and Energy Shield
- Scales with primary attributes (DEX, STR, INT)
- Integrates with equipment and skills
- Creates diverse defensive build options
- Balances chance-based and consistent mitigation
- Prevents invincibility through caps
- Integrates with penetration attributes (penetration reduces effectiveness of all secondary stats)
- Integrates with damage, resistance, and equipment systems

This system creates depth and variety in defensive builds, allowing players to specialize in different defensive approaches while maintaining balance with the damage system for fair and engaging combat. Penetration attributes provide counterplay for DPS builds, ensuring that high-defense builds can still be challenged by well-equipped attackers.

