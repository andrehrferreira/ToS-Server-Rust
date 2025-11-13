# Player System Documentation

## Overview

The Player system is the core character progression system for the MMORPG. Players progress from level 1 to 100 through experience gained from combat, quests, events, and experience items. The system includes non-linear progression curves, stat point allocation, and six core attributes that define character builds and playstyles.

## Key Features

- **Level Progression**: Level 1 to 100 with non-linear experience curves
- **Experience System**: Gain XP from monsters, quests, events, and items
- **Stat Point Allocation**: 5 stat points per level to distribute among 6 attributes
- **Six Core Attributes**: Strength, Dexterity, Intelligence, Luck, Constitution, Charisma
- **Level-Based XP Penalties**: Reduced XP from monsters below player level
- **Build Diversity**: Different stat distributions create unique playstyles
- **Traditional RPG Feel**: Classic RPG mechanics adapted for MMORPG

## Level System

### Level Range

Players progress from **Level 1** to **Level 100**.

### Experience Progression Curve

The experience curve is designed to provide fast progression early game, with increasing difficulty as players approach end-game:

**Progression Phases:**

1. **Level 1-60 (Early-Mid Game)**: 
   - Relatively fast progression
   - Players can reach level 60 quickly
   - Focus on learning game mechanics

2. **Level 61-80 (Mid-Late Game)**:
   - Progressively slower progression
   - Requires more time and effort
   - Build optimization becomes important

3. **Level 80-90 (Late Game)**:
   - Takes approximately the same time as level 1-60
   - Significant time investment required
   - End-game content becomes accessible

4. **Level 90-100 (End Game)**:
   - Difficulty doubles compared to 80-90
   - Requires maximum dedication
   - Reserved for the most dedicated players

**Experience Curve Formula:**
```
Base XP for level N = BaseXP * (1.15 ^ (N - 1))

Level 1-60: Fast progression multiplier
Level 61-80: Moderate progression multiplier
Level 80-90: Slow progression multiplier (same time as 1-60)
Level 90-100: Very slow progression multiplier (double difficulty)
```

### Experience Sources

Players gain experience from multiple sources:

1. **Monster Kills**: 
   - Primary source of XP
   - XP amount based on monster level relative to player level
   - See [Monster XP System](#monster-xp-system) for details

2. **Quests**:
   - Story quests provide XP rewards
   - Daily quests provide repeatable XP
   - Event quests provide bonus XP

3. **Events**:
   - World events provide XP participation rewards
   - PvP events provide XP for participation
   - Seasonal events provide bonus XP

4. **Experience Items**:
   - Consumable items that grant XP directly
   - Can be crafted or obtained from quests
   - Useful for catching up or power leveling

## Monster XP System

### Level-Based XP Calculation

Monsters only grant XP if they are at or above the player's level. Monsters below the player's level suffer XP penalties.

**XP Calculation Rules:**

1. **Monster Level >= Player Level**:
   - Full XP granted
   - XP amount based on monster level and type

2. **Monster Level < Player Level**:
   - **1-5 levels below**: 50-100% XP penalty
   - **6+ levels below**: 0 XP (no experience granted)

**XP Penalty Formula:**
```
Level Difference = Player Level - Monster Level

If Level Difference <= 5:
    XP Penalty = 50% + (Level Difference * 10%)
    Final XP = Base XP * (1 - XP Penalty)

If Level Difference > 5:
    Final XP = 0
```

**Example:**
- Player Level 35 vs Monster Level 30:
  - Level Difference = 5
  - XP Penalty = 50% + (5 * 10%) = 100%
  - Final XP = 0 (no experience)

- Player Level 35 vs Monster Level 33:
  - Level Difference = 2
  - XP Penalty = 50% + (2 * 10%) = 70%
  - Final XP = Base XP * 30%

- Player Level 35 vs Monster Level 35+:
  - Level Difference <= 0
  - XP Penalty = 0%
  - Final XP = Base XP (full experience)

### Monster XP Base Values

**Base XP by Monster Level:**
- Level 1-10: 10-50 XP
- Level 11-20: 50-150 XP
- Level 21-30: 150-400 XP
- Level 31-40: 400-800 XP
- Level 41-50: 800-1500 XP
- Level 51-60: 1500-2500 XP
- Level 61-70: 2500-4000 XP
- Level 71-80: 4000-6000 XP
- Level 81-90: 6000-9000 XP
- Level 91-100: 9000-15000 XP

**Monster Type Multipliers:**
- Normal Monster: 1.0x
- Elite Monster: 2.0x
- Boss Monster: 5.0x
- World Boss: 10.0x

## Stat Point System

### Stat Points per Level

Each level grants **5 stat points** to distribute among the six core attributes.

**Total Stat Points:**
- Level 1-100: 500 stat points total (5 points × 100 levels)
- Starting stats: Base values (typically 10-20 per stat)
- Maximum potential: Base + 500 distributed points

### Stat Point Allocation

Players can allocate stat points at any time, allowing for:
- **Respec System**: Reallocate stat points (may require items or gold)
- **Build Flexibility**: Experiment with different stat distributions
- **Progressive Building**: Allocate points as you level or save for later

## Core Attributes

### 1. Strength (STR)

**Primary Effects:**
- **Physical Damage**: Increases melee and physical weapon damage
- **Health Points**: Increases maximum HP (moderate amount)
- **Physical Resistance**: Increases resistance to physical damage
- **Weight Capacity**: Increases maximum weight the player can carry

**Damage Formula:**
```
Physical Damage = Base Damage + (STR * STR_Damage_Multiplier)
```

**Health Formula:**
```
Max HP = Base HP + (STR * STR_HP_Multiplier) + (CON * CON_HP_Multiplier)
```

**Resistance Formula:**
```
Physical Resistance = Base Resistance + (STR * STR_Resistance_Multiplier) + (CON * CON_Resistance_Multiplier)
```

**Weight Formula:**
```
Max Weight = Base Weight + (STR * STR_Weight_Multiplier) + (CON * CON_Weight_Multiplier)
```

**Build Focus:**
- Warriors, Tanks, Melee DPS
- Heavy armor users
- Physical weapon specialists

### 2. Dexterity (DEX)

**Primary Effects:**
- **Ranged Damage**: Increases damage with bows, crossbows, and ranged weapons
- **Evasion**: Increases chance to dodge/evade attacks (chance to take no damage)
- **Stamina**: Increases maximum stamina points
- **Movement Speed**: Increases base movement speed
- **Attack Speed**: Increases attack speed for all weapons

**Ranged Damage Formula:**
```
Ranged Damage = Base Damage + (DEX * DEX_Damage_Multiplier)
```

**Evasion Formula:**
```
Evasion Chance = Base Evasion + (DEX * DEX_Evasion_Multiplier)
Max Evasion Cap: 75% (to prevent invincibility)
```

**Stamina Formula:**
```
Max Stamina = Base Stamina + (DEX * DEX_Stamina_Multiplier)
```

**Speed Formula:**
```
Movement Speed = Base Speed + (DEX * DEX_Speed_Multiplier)
Attack Speed = Base Attack Speed + (DEX * DEX_AttackSpeed_Multiplier)
```

**Build Focus:**
- Rangers, Archers, Rogues
- Light armor users
- Fast-attacking builds
- Mobility-focused builds

### 3. Intelligence (INT)

**Primary Effects:**
- **Magic Damage**: Increases spell damage and magical weapon damage
- **Magic Barrier**: Increases magical damage absorption shield
- **Mana**: Increases maximum mana points

**Magic Damage Formula:**
```
Magic Damage = Base Spell Damage + (INT * INT_Damage_Multiplier)
```

**Magic Barrier Formula:**
```
Magic Barrier = Base Barrier + (INT * INT_Barrier_Multiplier)
Magic Barrier absorbs magical damage before HP
```

**Mana Formula:**
```
Max Mana = Base Mana + (INT * INT_Mana_Multiplier)
```

**Build Focus:**
- Mages, Wizards, Spellcasters
- Elemental damage builds
- Support/healing builds
- Magic weapon users

### 4. Luck (LCK)

**Primary Effects:**
- **Crafting Success**: Increases chance to craft high-quality items
- **Rare Drop Rate**: Increases chance to receive rare item drops
- **Hit Chance**: Increases accuracy and chance to hit targets
- **Rare Collection**: Increases chance to collect rare materials

**Crafting Formula:**
```
Crafting Success Chance = Base Chance + (LCK * LCK_Crafting_Multiplier)
Rare Item Chance = Base Rare Chance + (LCK * LCK_Rare_Multiplier)
```

**Drop Rate Formula:**
```
Rare Drop Chance = Base Drop Chance + (LCK * LCK_Drop_Multiplier)
```

**Hit Chance Formula:**
```
Hit Chance = Base Hit Chance + (LCK * LCK_Hit_Multiplier)
Max Hit Chance Cap: 95% (to prevent guaranteed hits)
```

**Collection Formula:**
```
Rare Material Chance = Base Collection Chance + (LCK * LCK_Collection_Multiplier)
```

**Build Focus:**
- Crafters, Gatherers
- Builds focused on item acquisition
- Support builds that benefit from rare items
- Hybrid builds that want better drops

### 5. Constitution (CON)

**Primary Effects:**
- **Health Points**: Increases maximum HP (more than STR)
- **Weight Capacity**: Increases maximum weight capacity (more than STR)
- **Physical Resistance**: Increases resistance to physical damage

**Health Formula:**
```
Max HP = Base HP + (STR * STR_HP_Multiplier) + (CON * CON_HP_Multiplier)
CON provides more HP per point than STR
```

**Weight Formula:**
```
Max Weight = Base Weight + (STR * STR_Weight_Multiplier) + (CON * CON_Weight_Multiplier)
CON provides more weight capacity per point than STR
```

**Resistance Formula:**
```
Physical Resistance = Base Resistance + (STR * STR_Resistance_Multiplier) + (CON * CON_Resistance_Multiplier)
```

**Build Focus:**
- Tanks, Defensive builds
- High HP builds
- Builds that need to carry many items
- Survivability-focused builds

### 6. Charisma (CHA)

**Primary Effects:**
- **Crafting Cost Reduction**: Reduces material costs when crafting
- **NPC Price Reduction**: Reduces prices when buying from NPCs
- **Tax Reduction**: Reduces various taxes (trading, guild, etc.)

**Crafting Cost Formula:**
```
Crafting Cost = Base Cost * (1 - (CHA * CHA_Crafting_Reduction_Multiplier))
Max Reduction Cap: 50% (to prevent free crafting)
```

**NPC Price Formula:**
```
NPC Price = Base Price * (1 - (CHA * CHA_NPC_Reduction_Multiplier))
Max Reduction Cap: 30% (to prevent exploitation)
```

**Tax Reduction Formula:**
```
Tax Amount = Base Tax * (1 - (CHA * CHA_Tax_Reduction_Multiplier))
Max Reduction Cap: 40% (to prevent tax evasion)
```

**Build Focus:**
- Crafters, Traders
- Economic-focused builds
- Guild leaders
- Players focused on wealth accumulation

## Stat Multipliers

### Recommended Multiplier Values

**Strength:**
- STR_Damage_Multiplier: 2.0 (2 damage per STR)
- STR_HP_Multiplier: 3.0 (3 HP per STR)
- STR_Resistance_Multiplier: 0.5 (0.5 resistance per STR)
- STR_Weight_Multiplier: 0.5 (0.5 weight per STR)

**Dexterity:**
- DEX_Damage_Multiplier: 2.0 (2 damage per DEX for ranged)
- DEX_Evasion_Multiplier: 0.1% (0.1% evasion per DEX)
- DEX_Stamina_Multiplier: 2.0 (2 stamina per DEX)
- DEX_Speed_Multiplier: 0.5% (0.5% movement speed per DEX)
- DEX_AttackSpeed_Multiplier: 0.3% (0.3% attack speed per DEX)

**Intelligence:**
- INT_Damage_Multiplier: 2.5 (2.5 magic damage per INT)
- INT_Barrier_Multiplier: 4.0 (4 barrier per INT)
- INT_Mana_Multiplier: 3.0 (3 mana per INT)

**Luck:**
- LCK_Crafting_Multiplier: 0.1% (0.1% crafting success per LCK)
- LCK_Rare_Multiplier: 0.05% (0.05% rare item chance per LCK)
- LCK_Drop_Multiplier: 0.1% (0.1% rare drop chance per LCK)
- LCK_Hit_Multiplier: 0.15% (0.15% hit chance per LCK)
- LCK_Collection_Multiplier: 0.1% (0.1% rare collection chance per LCK)

**Constitution:**
- CON_HP_Multiplier: 5.0 (5 HP per CON, more than STR)
- CON_Weight_Multiplier: 1.0 (1 weight per CON, more than STR)
- CON_Resistance_Multiplier: 0.8 (0.8 resistance per CON)

**Charisma:**
- CHA_Crafting_Reduction_Multiplier: 0.1% (0.1% cost reduction per CHA)
- CHA_NPC_Reduction_Multiplier: 0.15% (0.15% price reduction per CHA)
- CHA_Tax_Reduction_Multiplier: 0.2% (0.2% tax reduction per CHA)

## Build Examples

### Warrior Build (STR/CON Focus)
```
Level 100 Distribution:
- STR: 200 (Physical damage, HP, resistance)
- DEX: 50 (Some attack speed)
- INT: 20 (Base)
- LCK: 30 (Some hit chance)
- CON: 150 (High HP, weight capacity)
- CHA: 50 (Some cost reduction)

Focus: Tank/DPS hybrid, high survivability
```

### Rogue Build (DEX/LCK Focus)
```
Level 100 Distribution:
- STR: 30 (Base)
- DEX: 250 (High evasion, attack speed, ranged damage)
- INT: 20 (Base)
- LCK: 150 (High hit chance, rare drops)
- CON: 30 (Base)
- CHA: 20 (Base)

Focus: High DPS, evasion tank, item acquisition
```

### Mage Build (INT/CON Focus)
```
Level 100 Distribution:
- STR: 20 (Base)
- DEX: 30 (Some attack speed)
- INT: 250 (High magic damage, barrier, mana)
- LCK: 50 (Some hit chance)
- CON: 120 (Survivability)
- CHA: 30 (Base)

Focus: High magic DPS, survivability through barrier
```

### Crafter Build (CHA/LCK Focus)
```
Level 100 Distribution:
- STR: 50 (Some weight)
- DEX: 30 (Base)
- INT: 50 (Some mana for crafting)
- LCK: 200 (High crafting success, rare items)
- CON: 50 (Some weight)
- CHA: 120 (Cost reduction, NPC prices)

Focus: Economic advantage, high-quality crafting
```

## Level-Up System

### Level-Up Process

1. **Experience Gain**: Player gains XP from various sources
2. **Level Check**: System checks if XP >= required XP for next level
3. **Level Up**: Player advances to next level
4. **Stat Points**: Player receives 5 stat points
5. **Notifications**: Client notified of level up, stat points available
6. **Rewards**: Any level-based rewards granted

### Level-Up Rewards

**Every Level:**
- 5 stat points
- Increased base stats (small automatic increases)

**Milestone Levels (10, 20, 30, etc.):**
- Additional rewards (titles, items, etc.)
- Unlock new content areas
- Unlock new abilities/skills

**End-Game Milestones (90, 95, 100):**
- Special rewards
- Exclusive titles
- End-game content access

## Stat Point Respec System

### Respec Mechanics

Players can reallocate stat points through:

1. **Respec Items**: Consumable items that allow full respec
2. **Respec NPC**: NPC that charges gold for respec
3. **Limited Respecs**: May have cooldown or limit per character

**Respec Costs:**
- Level 1-50: Low cost (allows experimentation)
- Level 51-80: Moderate cost
- Level 81-100: High cost (prevents constant respecs)

**Respec Limitations:**
- May require confirmation
- May have cooldown period
- May be limited per character lifetime

## Integration with Other Systems

### Equipment System

- Stat requirements on equipment (see [ITEMS.md](../items/ITEMS.md))
- Higher tier items require higher stats
- Stats affect which equipment players can use

### Combat System

- Stats directly affect damage, defense, and combat effectiveness
- Different builds excel in different combat scenarios
- Stats determine playstyle (tank, DPS, support, etc.)

### Crafting System

- Luck affects crafting success and rare item chance
- Charisma reduces crafting costs
- Intelligence may affect certain crafting recipes

### Economy System

- Charisma affects NPC prices and taxes
- Luck affects item acquisition
- Stats create economic advantages for certain builds

## Rust Implementation Considerations

### Performance Optimizations

1. **Stat Calculation Caching**: Cache calculated stat values
2. **Incremental Updates**: Only recalculate changed stats
3. **Batch Processing**: Process multiple stat updates together
4. **Lookup Tables**: Pre-calculate common stat combinations

### Data Structures

```rust
pub struct PlayerStats {
    pub strength: u32,
    pub dexterity: u32,
    pub intelligence: u32,
    pub luck: u32,
    pub constitution: u32,
    pub charisma: u32,
    pub available_points: u32,
}

pub struct PlayerLevel {
    pub level: u8,  // 1-100
    pub experience: u64,
    pub experience_to_next: u64,
}

pub struct StatMultipliers {
    pub str_damage: f32,
    pub str_hp: f32,
    pub str_resistance: f32,
    pub str_weight: f32,
    // ... other multipliers
}
```

### Experience Calculation

```rust
pub fn calculate_xp_penalty(player_level: u8, monster_level: u8) -> f32 {
    if monster_level >= player_level {
        return 1.0; // Full XP
    }
    
    let level_diff = player_level - monster_level;
    if level_diff > 5 {
        return 0.0; // No XP
    }
    
    let penalty = 0.5 + (level_diff as f32 * 0.1);
    1.0 - penalty.min(1.0)
}

pub fn calculate_xp_gain(
    base_xp: u64,
    player_level: u8,
    monster_level: u8,
    monster_type: MonsterType,
) -> u64 {
    let penalty = calculate_xp_penalty(player_level, monster_level);
    let type_multiplier = monster_type.xp_multiplier();
    
    (base_xp as f32 * penalty * type_multiplier) as u64
}
```

### Stat Calculation

```rust
impl PlayerStats {
    pub fn calculate_physical_damage(&self, base_damage: f32) -> f32 {
        base_damage + (self.strength as f32 * STR_DAMAGE_MULTIPLIER)
    }
    
    pub fn calculate_max_hp(&self, base_hp: f32) -> f32 {
        base_hp 
            + (self.strength as f32 * STR_HP_MULTIPLIER)
            + (self.constitution as f32 * CON_HP_MULTIPLIER)
    }
    
    pub fn calculate_evasion(&self, base_evasion: f32) -> f32 {
        let evasion = base_evasion + (self.dexterity as f32 * DEX_EVASION_MULTIPLIER);
        evasion.min(0.75) // Cap at 75%
    }
    
    // ... other stat calculations
}
```

## Testing Strategy

### Unit Tests

- Test XP calculation with various level differences
- Test XP penalty calculations
- Test stat point allocation
- Test stat calculations (damage, HP, etc.)
- Test level-up process
- Test respec system

### Integration Tests

- Test XP gain from monsters
- Test XP gain from quests
- Test stat requirements for equipment
- Test stat effects in combat
- Test crafting with different stat distributions

### Balance Tests

- Test progression curve (time to level)
- Test stat effectiveness (damage output, survivability)
- Test build diversity (all builds viable)
- Test economic balance (charisma effectiveness)

## Performance Considerations

### Experience Tracking

- Use efficient data structures for XP tracking
- Cache XP requirements per level
- Batch XP updates when possible
- Use incremental updates for stat changes

### Stat Calculations

- Cache calculated stat values
- Invalidate cache on stat changes
- Use lookup tables for common calculations
- Optimize hot paths (combat stat calculations)

### Level-Up Processing

- Process level-ups asynchronously
- Batch multiple level-ups if possible
- Minimize database writes
- Use efficient notification system

