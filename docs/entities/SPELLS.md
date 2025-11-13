# Spells and Actions System

## Overview

This document describes the spell and action system used by creatures and players. Spells are classified into distinct groups based on their targeting and effect mechanics. The system supports both magical spells and melee abilities.

## Action Type Classification

Actions/spells are classified into six distinct groups:

1. **Projectiles** - Elemental projectiles with conditions
2. **Target Area** - Area effects centered on a point
3. **Camera Direction** - Directional abilities along camera line of sight
4. **Single Target** - Direct effects on selected target
5. **Target Self** - Self-targeted abilities (blessings and self-healing)
6. **Area (Centered)** - Spells originating from caster as center

---

## 1. Projectiles

### Overview

Projectiles are ranged abilities that travel through the air and can apply elemental damage and conditions on impact. They can be single or multiple projectiles depending on the spell.

### Characteristics

**Damage Types:**
- Elemental damage (Fire, Cold, Poison, Energy, Light, Dark)
- Can apply conditions/status effects on hit
- Examples:
  - Fireball: Fire damage + burn chance
  - Ice Bolt: Cold damage + slow chance
  - Poison Dart: Poison damage + poison condition
  - Lightning Bolt: Energy damage + stun chance

**Projectile Variants:**
- Single projectile: One projectile fired
- Multiple projectiles: Multiple projectiles fired simultaneously
- Homing projectiles: Projectiles that track targets
- Piercing projectiles: Projectiles that pass through multiple targets

### Implementation

```rust
pub struct ProjectileAction {
    pub action_id: ActionId,
    pub name: String,
    pub damage_type: DamageType,
    pub base_damage: Dices,
    pub projectile_count: u32,  // 1 for single, >1 for multiple
    pub projectile_speed: f32,
    pub range: f32,
    pub conditions: Vec<Condition>,  // Conditions applied on hit
    pub pierce_count: u32,  // 0 = no pierce, >0 = pierces through N targets
    pub homing: bool,
}

pub struct Condition {
    pub condition_type: ConditionType,
    pub chance: f32,  // 0.0 to 1.0
    pub duration: Duration,
    pub effect: ConditionEffect,
}

pub enum ConditionType {
    Burn,      // Fire damage over time
    Freeze,    // Movement slow + cold damage
    Poison,    // Poison damage over time
    Stun,      // Cannot act
    Slow,      // Movement speed reduction
    Blind,     // Reduced accuracy
    Weakness,  // Damage reduction
}
```

### Examples

**Fireball (Single Projectile):**
```rust
ProjectileAction {
    action_id: ActionId::Fireball,
    name: "Fireball".to_string(),
    damage_type: DamageType::Fire,
    base_damage: Dices::parse("3D6"),
    projectile_count: 1,
    projectile_speed: 15.0,
    range: 20.0,
    conditions: vec![
        Condition {
            condition_type: ConditionType::Burn,
            chance: 0.3,  // 30% chance to burn
            duration: Duration::seconds(5),
            effect: ConditionEffect::DamageOverTime { dice: Dices::parse("1D3") },
        }
    ],
    pierce_count: 0,
    homing: false,
}
```

**Multi-Shot (Multiple Projectiles):**
```rust
ProjectileAction {
    action_id: ActionId::MultiShot,
    name: "Multi-Shot".to_string(),
    damage_type: DamageType::Physic,
    base_damage: Dices::parse("1D6"),
    projectile_count: 5,  // 5 projectiles
    projectile_speed: 20.0,
    range: 15.0,
    conditions: vec![],
    pierce_count: 0,
    homing: false,
}
```

**Ice Lance (Piercing Projectile):**
```rust
ProjectileAction {
    action_id: ActionId::IceLance,
    name: "Ice Lance".to_string(),
    damage_type: DamageType::Cold,
    base_damage: Dices::parse("2D8"),
    projectile_count: 1,
    projectile_speed: 25.0,
    range: 30.0,
    conditions: vec![
        Condition {
            condition_type: ConditionType::Slow,
            chance: 0.5,
            duration: Duration::seconds(3),
            effect: ConditionEffect::MovementSpeedReduction { percent: 0.5 },
        }
    ],
    pierce_count: 3,  // Pierces through 3 targets
    homing: false,
}
```

### Usage Patterns

**Ranged Combat:**
- Engage enemies from distance
- Apply conditions before melee engagement
- Finish off fleeing enemies

**Condition Application:**
- Apply debuffs from safe distance
- Stack multiple conditions
- Control enemy movement

**Area Denial:**
- Force enemies to move
- Control battlefield positioning
- Create escape routes

---

## 2. Target Area

### Overview

Target Area abilities are used without a player-defined position. The spell creates effects centered on a point, affecting an area around that center. Effects can include healing, damage, mana recovery, buffs, and debuffs.

### Characteristics

**Effect Types:**
- Healing circles
- Damage areas
- Mana recovery areas
- Buff areas (apply buffs to allies)
- Debuff areas (apply debuffs to enemies)
- Mixed effects

**Targeting:**
- No player position selection required
- System determines optimal center point
- Area-of-effect around center
- Can affect multiple targets

### Implementation

```rust
pub struct TargetAreaAction {
    pub action_id: ActionId,
    pub name: String,
    pub radius: f32,
    pub effect_type: AreaEffectType,
    pub duration: Duration,  // How long area persists
    pub tick_interval: Duration,  // Effect application interval
    pub effects: Vec<AreaEffect>,
}

pub enum AreaEffectType {
    Instant,     // Effect applied once
    Persistent,  // Area persists and applies effects over time
    Delayed,     // Effect applied after delay
}

pub enum AreaEffect {
    Healing { amount: Dices, target_type: TargetType },
    Damage { amount: Dices, damage_type: DamageType, target_type: TargetType },
    ManaRecovery { amount: Dices, target_type: TargetType },
    Buff { buff_id: BuffId, target_type: TargetType },
    Debuff { debuff_id: DebuffId, target_type: TargetType },
    Condition { condition: Condition, target_type: TargetType },
}

pub enum TargetType {
    Allies,   // Only affects allies
    Enemies,  // Only affects enemies
    All,      // Affects all entities
    Self,     // Only affects caster
}
```

### Examples

**Healing Circle:**
```rust
TargetAreaAction {
    action_id: ActionId::HealingCircle,
    name: "Healing Circle".to_string(),
    radius: 5.0,
    effect_type: AreaEffectType::Persistent,
    duration: Duration::seconds(10),
    tick_interval: Duration::seconds(1),
    effects: vec![
        AreaEffect::Healing {
            amount: Dices::parse("1D8"),
            target_type: TargetType::Allies,
        }
    ],
}
```

**Poison Cloud:**
```rust
TargetAreaAction {
    action_id: ActionId::PoisonCloud,
    name: "Poison Cloud".to_string(),
    radius: 4.0,
    effect_type: AreaEffectType::Persistent,
    duration: Duration::seconds(15),
    tick_interval: Duration::seconds(2),
    effects: vec![
        AreaEffect::Damage {
            amount: Dices::parse("1D4"),
            damage_type: DamageType::Poison,
            target_type: TargetType::Enemies,
        },
        AreaEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Poison,
                chance: 1.0,  // Always applies
                duration: Duration::seconds(10),
                effect: ConditionEffect::DamageOverTime { dice: Dices::parse("1D2") },
            },
            target_type: TargetType::Enemies,
        }
    ],
}
```

**Mana Fountain:**
```rust
TargetAreaAction {
    action_id: ActionId::ManaFountain,
    name: "Mana Fountain".to_string(),
    radius: 6.0,
    effect_type: AreaEffectType::Persistent,
    duration: Duration::seconds(20),
    tick_interval: Duration::seconds(3),
    effects: vec![
        AreaEffect::ManaRecovery {
            amount: Dices::parse("2D6"),
            target_type: TargetType::Allies,
        }
    ],
}
```

**Buff Area:**
```rust
TargetAreaAction {
    action_id: ActionId::BlessingArea,
    name: "Area of Blessing".to_string(),
    radius: 8.0,
    effect_type: AreaEffectType::Instant,
    duration: Duration::zero(),
    tick_interval: Duration::zero(),
    effects: vec![
        AreaEffect::Buff {
            buff_id: BuffId::Strength,
            target_type: TargetType::Allies,
        },
        AreaEffect::Buff {
            buff_id: BuffId::Defense,
            target_type: TargetType::Allies,
        }
    ],
}
```

### Usage Patterns

**Area Control:**
- Control enemy movement
- Create safe zones for allies
- Deny area access

**Group Support:**
- Heal multiple allies
- Buff entire group
- Provide area-wide benefits

**Strategic Positioning:**
- Place effects at optimal locations
- Control battlefield flow
- Support tactical positioning

---

## 3. Camera Direction

### Overview

Camera Direction abilities are launched directly along the camera's line of sight. These abilities create directional effects based on the facing direction, including mobility abilities, environmental manipulation, and directional attacks.

### Characteristics

**Ability Types:**
- Mobility abilities (Blink, Teleport)
- Environmental walls (Ice Wall, Fire Wall)
- Directional attacks (Lightning Bolt, Beam)
- Area creation along direction

**Direction:**
- Based on camera/character facing direction
- Creates effects along line of sight
- Can extend for distance or create walls

### Implementation

```rust
pub struct CameraDirectionAction {
    pub action_id: ActionId,
    pub name: String,
    pub effect_type: DirectionalEffectType,
    pub range: f32,  // How far effect extends
    pub width: f32,  // Width of effect (for walls/beams)
    pub effects: Vec<DirectionalEffect>,
}

pub enum DirectionalEffectType {
    Instant,      // Instant effect (Blink, Teleport)
    Wall,         // Creates wall (Ice Wall, Fire Wall)
    Beam,         // Continuous beam (Lightning, Laser)
    Projectile,  // Projectile along direction
}

pub enum DirectionalEffect {
    Teleport { distance: f32 },
    Damage { amount: Dices, damage_type: DamageType },
    Wall { wall_type: WallType, duration: Duration },
    Condition { condition: Condition },
    Buff { buff_id: BuffId },
}

pub enum WallType {
    Ice,    // Slows enemies, blocks movement
    Fire,   // Damages enemies, blocks movement
    Shadow, // Blocks vision, damages enemies
    Stone,  // Blocks movement completely
}
```

### Examples

**Blink (Mobility):**
```rust
CameraDirectionAction {
    action_id: ActionId::Blink,
    name: "Blink".to_string(),
    effect_type: DirectionalEffectType::Instant,
    range: 10.0,
    width: 0.0,
    effects: vec![
        DirectionalEffect::Teleport { distance: 10.0 }
    ],
}
```

**Ice Wall (Environmental):**
```rust
CameraDirectionAction {
    action_id: ActionId::IceWall,
    name: "Ice Wall".to_string(),
    effect_type: DirectionalEffectType::Wall,
    range: 8.0,
    width: 2.0,
    effects: vec![
        DirectionalEffect::Wall {
            wall_type: WallType::Ice,
            duration: Duration::seconds(30),
        },
        DirectionalEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Slow,
                chance: 1.0,
                duration: Duration::seconds(5),
                effect: ConditionEffect::MovementSpeedReduction { percent: 0.7 },
            }
        }
    ],
}
```

**Fire Wall:**
```rust
CameraDirectionAction {
    action_id: ActionId::FireWall,
    name: "Fire Wall".to_string(),
    effect_type: DirectionalEffectType::Wall,
    range: 10.0,
    width: 3.0,
    effects: vec![
        DirectionalEffect::Wall {
            wall_type: WallType::Fire,
            duration: Duration::seconds(20),
        },
        DirectionalEffect::Damage {
            amount: Dices::parse("2D4"),
            damage_type: DamageType::Fire,
        },
        DirectionalEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Burn,
                chance: 0.5,
                duration: Duration::seconds(8),
                effect: ConditionEffect::DamageOverTime { dice: Dices::parse("1D3") },
            }
        }
    ],
}
```

**Lightning Bolt (Beam):**
```rust
CameraDirectionAction {
    action_id: ActionId::LightningBolt,
    name: "Lightning Bolt".to_string(),
    effect_type: DirectionalEffectType::Beam,
    range: 25.0,
    width: 1.0,
    effects: vec![
        DirectionalEffect::Damage {
            amount: Dices::parse("4D6"),
            damage_type: DamageType::Energy,
        },
        DirectionalEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Stun,
                chance: 0.3,
                duration: Duration::seconds(2),
                effect: ConditionEffect::CannotAct,
            }
        }
    ],
}
```

### Usage Patterns

**Mobility:**
- Repositioning during combat
- Escaping dangerous situations
- Closing distance quickly

**Environmental Manipulation:**
- Create barriers
- Block enemy movement
- Control battlefield layout

**Directional Attacks:**
- High-damage line attacks
- Piercing through multiple enemies
- Long-range precision attacks

---

## 4. Single Target

### Overview

Single Target abilities apply direct effects on a selected target. These can be healing, curses, buffs, or direct damage. The effect is focused on a single entity.

### Characteristics

**Effect Types:**
- Direct damage
- Healing
- Curses (debuffs)
- Buffs
- Condition application
- Mixed effects

**Targeting:**
- Requires target selection
- Can target allies or enemies
- Instant or over-time effects

### Implementation

```rust
pub struct SingleTargetAction {
    pub action_id: ActionId,
    pub name: String,
    pub range: f32,
    pub effects: Vec<TargetEffect>,
    pub cast_time: Duration,
}

pub enum TargetEffect {
    Damage { amount: Dices, damage_type: DamageType },
    Healing { amount: Dices },
    ManaRecovery { amount: Dices },
    Buff { buff_id: BuffId, duration: Duration },
    Debuff { debuff_id: DebuffId, duration: Duration },
    Condition { condition: Condition },
    Curse { curse_type: CurseType, duration: Duration },
    Dispel { dispel_type: DispelType },
}

pub enum CurseType {
    Weakness,    // Reduces damage
    Vulnerability, // Increases damage taken
    Slow,        // Reduces movement speed
    Silence,     // Prevents spell casting
    Blind,       // Reduces accuracy
    Poison,      // Damage over time
}
```

### Examples

**Heal:**
```rust
SingleTargetAction {
    action_id: ActionId::Heal,
    name: "Heal".to_string(),
    range: 15.0,
    effects: vec![
        TargetEffect::Healing {
            amount: Dices::parse("3D8"),
        }
    ],
    cast_time: Duration::milliseconds(1500),
}
```

**Curse of Weakness:**
```rust
SingleTargetAction {
    action_id: ActionId::CurseWeakness,
    name: "Curse of Weakness".to_string(),
    range: 20.0,
    effects: vec![
        TargetEffect::Curse {
            curse_type: CurseType::Weakness,
            duration: Duration::seconds(30),
        },
        TargetEffect::Debuff {
            debuff_id: DebuffId::DamageReduction,
            duration: Duration::seconds(30),
        }
    ],
    cast_time: Duration::milliseconds(2000),
}
```

**Direct Damage:**
```rust
SingleTargetAction {
    action_id: ActionId::MagicMissile,
    name: "Magic Missile".to_string(),
    range: 25.0,
    effects: vec![
        TargetEffect::Damage {
            amount: Dices::parse("2D10"),
            damage_type: DamageType::Energy,
        }
    ],
    cast_time: Duration::milliseconds(1000),
}
```

**Buff Target:**
```rust
SingleTargetAction {
    action_id: ActionId::Blessing,
    name: "Blessing".to_string(),
    range: 20.0,
    effects: vec![
        TargetEffect::Buff {
            buff_id: BuffId::Strength,
            duration: Duration::seconds(60),
        },
        TargetEffect::Buff {
            buff_id: BuffId::Defense,
            duration: Duration::seconds(60),
        }
    ],
    cast_time: Duration::milliseconds(1500),
}
```

### Usage Patterns

**Focused Damage:**
- High single-target damage
- Finishing moves
- Burst damage

**Targeted Healing:**
- Heal specific ally
- Emergency healing
- Focused support

**Debuffing:**
- Apply curses to enemies
- Reduce enemy effectiveness
- Control specific targets

---

## 5. Target Self

### Overview

Target Self abilities are self-targeted abilities including blessings and self-healing. In the client, these are triggered by pressing CTRL+Left before using the ability.

### Characteristics

**Effect Types:**
- Self-healing
- Self-blessings (buffs)
- Mana recovery
- Stamina recovery
- Defensive buffs
- Utility buffs

**Usage:**
- Self-preservation
- Pre-combat preparation
- Emergency survival
- Resource recovery

### Implementation

```rust
pub struct TargetSelfAction {
    pub action_id: ActionId,
    pub name: String,
    pub effects: Vec<SelfEffect>,
    pub cast_time: Duration,
}

pub enum SelfEffect {
    Healing { amount: Dices },
    ManaRecovery { amount: Dices },
    StaminaRecovery { amount: Dices },
    Buff { buff_id: BuffId, duration: Duration },
    ConditionRemoval { condition_types: Vec<ConditionType> },
    ResourceBoost { resource_type: ResourceType, amount: u32 },
}

pub enum ResourceType {
    Health,
    Mana,
    Stamina,
}
```

### Examples

**Self-Heal:**
```rust
TargetSelfAction {
    action_id: ActionId::SelfHeal,
    name: "Self-Heal".to_string(),
    effects: vec![
        SelfEffect::Healing {
            amount: Dices::parse("4D6"),
        }
    ],
    cast_time: Duration::milliseconds(2000),
}
```

**Blessing (Self-Buff):**
```rust
TargetSelfAction {
    action_id: ActionId::SelfBlessing,
    name: "Blessing".to_string(),
    effects: vec![
        SelfEffect::Buff {
            buff_id: BuffId::Strength,
            duration: Duration::seconds(300),
        },
        SelfEffect::Buff {
            buff_id: BuffId::Defense,
            duration: Duration::seconds(300),
        },
        SelfEffect::Buff {
            buff_id: BuffId::Resistance,
            duration: Duration::seconds(300),
        }
    ],
    cast_time: Duration::milliseconds(3000),
}
```

**Mana Recovery:**
```rust
TargetSelfAction {
    action_id: ActionId::Meditate,
    name: "Meditate".to_string(),
    effects: vec![
        SelfEffect::ManaRecovery {
            amount: Dices::parse("5D10"),
        },
        SelfEffect::Buff {
            buff_id: BuffId::ManaRegeneration,
            duration: Duration::seconds(60),
        }
    ],
    cast_time: Duration::milliseconds(5000),
}
```

**Condition Removal:**
```rust
TargetSelfAction {
    action_id: ActionId::Purify,
    name: "Purify".to_string(),
    effects: vec![
        SelfEffect::ConditionRemoval {
            condition_types: vec![
                ConditionType::Poison,
                ConditionType::Burn,
                ConditionType::Curse,
            ],
        },
        SelfEffect::Healing {
            amount: Dices::parse("2D4"),
        }
    ],
    cast_time: Duration::milliseconds(1500),
}
```

### Usage Patterns

**Self-Preservation:**
- Emergency healing
- Defensive buffs
- Condition removal

**Pre-Combat:**
- Buffing before engagement
- Resource preparation
- Setting up for combat

**Resource Management:**
- Mana/stamina recovery
- Health regeneration
- Resource optimization

---

## 6. Area (Centered on Caster)

### Overview

Area abilities are spells that originate from the caster as the center point. Effects radiate outward from the caster, affecting an area around them. These are typically powerful area-of-effect abilities.

### Characteristics

**Effect Types:**
- Area damage (Blizzard, Earthquake)
- Healing aura
- Mana burst
- Buff/debuff aura
- Environmental effects

**Area:**
- Centered on caster position
- Radius extends outward
- Can affect multiple targets
- Caster is always at center

### Implementation

```rust
pub struct AreaCenteredAction {
    pub action_id: ActionId,
    pub name: String,
    pub radius: f32,
    pub effect_type: CenteredAreaType,
    pub duration: Duration,  // For persistent effects
    pub tick_interval: Duration,  // For damage over time
    pub effects: Vec<CenteredAreaEffect>,
}

pub enum CenteredAreaType {
    Instant,     // One-time effect
    Persistent,  // Effect persists over time
    Channeled,   // Requires channeling
}

pub enum CenteredAreaEffect {
    Damage { amount: Dices, damage_type: DamageType, target_type: TargetType },
    Healing { amount: Dices, target_type: TargetType },
    ManaRecovery { amount: Dices, target_type: TargetType },
    Buff { buff_id: BuffId, target_type: TargetType },
    Debuff { debuff_id: DebuffId, target_type: TargetType },
    Condition { condition: Condition, target_type: TargetType },
    Environmental { env_type: EnvironmentalType },
}

pub enum EnvironmentalType {
    Blizzard,   // Cold damage + slow
    Earthquake, // Physical damage + knockback
    Meteor,     // Fire damage + burn
    HealingAura, // Continuous healing
    ManaBurst,  // Mana recovery burst
}
```

### Examples

**Blizzard:**
```rust
AreaCenteredAction {
    action_id: ActionId::Blizzard,
    name: "Blizzard".to_string(),
    radius: 12.0,
    effect_type: CenteredAreaType::Channeled,
    duration: Duration::seconds(10),
    tick_interval: Duration::seconds(1),
    effects: vec![
        CenteredAreaEffect::Damage {
            amount: Dices::parse("2D6"),
            damage_type: DamageType::Cold,
            target_type: TargetType::Enemies,
        },
        CenteredAreaEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Slow,
                chance: 0.4,
                duration: Duration::seconds(3),
                effect: ConditionEffect::MovementSpeedReduction { percent: 0.6 },
            },
            target_type: TargetType::Enemies,
        },
        CenteredAreaEffect::Environmental {
            env_type: EnvironmentalType::Blizzard,
        }
    ],
}
```

**Earthquake:**
```rust
AreaCenteredAction {
    action_id: ActionId::Earthquake,
    name: "Earthquake".to_string(),
    radius: 15.0,
    effect_type: CenteredAreaType::Instant,
    duration: Duration::zero(),
    tick_interval: Duration::zero(),
    effects: vec![
        CenteredAreaEffect::Damage {
            amount: Dices::parse("5D8"),
            damage_type: DamageType::Physic,
            target_type: TargetType::Enemies,
        },
        CenteredAreaEffect::Condition {
            condition: Condition {
                condition_type: ConditionType::Stun,
                chance: 0.6,
                duration: Duration::seconds(2),
                effect: ConditionEffect::CannotAct,
            },
            target_type: TargetType::Enemies,
        },
        CenteredAreaEffect::Environmental {
            env_type: EnvironmentalType::Earthquake,
        }
    ],
}
```

**Healing Aura:**
```rust
AreaCenteredAction {
    action_id: ActionId::HealingAura,
    name: "Healing Aura".to_string(),
    radius: 8.0,
    effect_type: CenteredAreaType::Persistent,
    duration: Duration::seconds(30),
    tick_interval: Duration::seconds(2),
    effects: vec![
        CenteredAreaEffect::Healing {
            amount: Dices::parse("1D6"),
            target_type: TargetType::Allies,
        },
        CenteredAreaEffect::Environmental {
            env_type: EnvironmentalType::HealingAura,
        }
    ],
}
```

**Mana Burst:**
```rust
AreaCenteredAction {
    action_id: ActionId::ManaBurst,
    name: "Mana Burst".to_string(),
    radius: 10.0,
    effect_type: CenteredAreaType::Instant,
    duration: Duration::zero(),
    tick_interval: Duration::zero(),
    effects: vec![
        CenteredAreaEffect::ManaRecovery {
            amount: Dices::parse("3D10"),
            target_type: TargetType::Allies,
        },
        CenteredAreaEffect::Environmental {
            env_type: EnvironmentalType::ManaBurst,
        }
    ],
}
```

### Usage Patterns

**Close-Range Area Effects:**
- Damage when surrounded
- Support when in group
- Environmental control

**Group Support:**
- Heal multiple allies
- Buff entire group
- Provide area-wide benefits

**Area Control:**
- Control enemy positioning
- Create safe zones
- Deny area access

---

## Action Type Selection by Behavior

Different behavior profiles prioritize different action types:

### Strategic Behavior
- **Projectiles**: Ranged engagement, condition application
- **Target Area**: Healing circles, buff areas, debuff areas
- **Camera Direction**: Blink, Ice Wall, Lightning Bolt
- **Single Target**: Curses, direct damage, healing
- **Target Self**: Self-healing, blessings, mana recovery
- **Area (Centered)**: Blizzard, Earthquake, Healing Aura

### Aggressive Behavior
- **Projectiles**: Fire projectiles, multiple projectiles
- **Target Area**: Damage areas (less frequent)
- **Camera Direction**: Charge, Leap, Fire Wall
- **Single Target**: Damage-boosting buffs, direct damage
- **Target Self**: Damage amplification, attack speed buffs
- **Area (Centered)**: Damage bursts when surrounded

### Stealthy Behavior
- **Projectiles**: Poison, Shadow projectiles (from stealth)
- **Target Area**: Smoke clouds, shadow areas
- **Camera Direction**: Blink, Shadow Step, Ice Wall
- **Single Target**: Backstab, critical strikes, curses
- **Target Self**: Stealth, evasion buffs, invisibility
- **Area (Centered)**: Smoke bomb, shadow burst

### Powerful Behavior (Bosses)
- **Projectiles**: Massive elemental projectiles, patterns
- **Target Area**: Massive damage areas, debuff areas
- **Camera Direction**: Meteor, beam attacks, environmental walls
- **Single Target**: Powerful curses, massive damage
- **Target Self**: Defensive buffs, damage reduction
- **Area (Centered)**: Blizzard, Earthquake, Meteor Shower (primary)

### Fearful Behavior
- **Projectiles**: Rarely used (last resort)
- **Target Area**: Not typically used
- **Camera Direction**: Blink, Teleport (primary escape)
- **Single Target**: Weak attacks when cornered
- **Target Self**: Speed buffs, invisibility
- **Area (Centered)**: Not typically used

---

## Integration with Actions System

Actions are defined in the [Actions System](../entities/ACTIONS.md) and can be of any of these types:

```rust
pub enum ActionType {
    None,
    Projectile,
    Target,
    TargetSelf,
    Area,
    DirectionalCamera,  // Camera Direction
}
```

The action type determines:
- Targeting requirements
- Effect application method
- Range and area calculations
- Visual/audio effects
- Client interaction

---

## See Also

- [ACTIONS.md](../../entities/ACTIONS.md) - Complete actions system documentation
- [BEHAVIORS.md](./BEHAVIORS.md) - Behavior profiles that use these spells
- [COMBAT.md](../combat/COMBAT.md) - Combat system integration

