# Summons System

## Overview

Summons are temporary entities created by player actions or skills. They have a limited lifetime and are automatically removed when their lifetime expires. Summons assist players in combat and can have various behaviors based on their type.

## Summon Class (TypeScript)

### Base Properties

```typescript
class Summon extends Creature {
    public owner: Player;
    public lifeTime: number;
    
    constructor(owner: Player) {
        super();
        this.kind = EntitiesKind.Summon;
        this.team = new Team(TeamKind.Pet, this);
        this.teamOwner = owner;
        this.owner = owner;
        
        setInterval(this.tickLifetime.bind(this), 1000);
    }
}
```

### Summon Properties

**Base Properties:**
- **owner**: Player who summoned the entity
- **lifeTime**: Timestamp when summon expires
- **kind**: Set to Summon
- **team**: Team set to Pet team
- **teamOwner**: Owner for team purposes

**Lifetime Management:**
- Lifetime checked every second
- Summon removed when lifetime expires
- Lifetime set as timestamp (milliseconds)

### Lifetime System

**Start Lifetime:**
```typescript
startLifeTime(lifetime: number) {
    this.lifeTime = new Date().getTime() + lifetime * 1000;
}
```

**Tick Lifetime:**
```typescript
tickLifetime() {
    if (!this.map || this.removed) {
        return;
    }
    
    if (this.lifeTime < new Date().getTime()) {
        this.map.removeEntity(this);
    }
}
```

**Lifetime Behavior:**
- Checks every 1 second
- Removes entity when expired
- Validates map and removed status
- Automatic cleanup

## Minion Entity (C#)

### Minion Base Class

The C# implementation uses `MinionEntity` as the base for summons, which extends `MonsterBase` and provides advanced AI and combat behavior.

### Minion Properties

```csharp
public class MinionEntity : MonsterBase {
    public float Lifetime;
    public CreatureEntity Master;
    public bool RemoveOnDie = false;
    
    public override CreatureEntity Parent => Master ?? this;
}
```

### Lifetime Initialization

```csharp
public override async void Initialize() {
    Health = Health == 0 ? MaxHealth : Health;
    
    Master.DamageInflicted += OnMasterDamageInflicted;
    
    base.Initialize();
    
    if (!float.IsInfinity(Lifetime) && Lifetime > 0) {
        await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(Lifetime));
        Die(Master);
    }
}
```

**Lifetime Behavior:**
- Lifetime can be infinite (float.PositiveInfinity)
- Lifetime in seconds
- Dies when lifetime expires
- Master receives death event

## Summon Creation

### Static Registration

**TypeScript:**
```typescript
Entity.Summons.set("SummonName", SummonClass);
```

**C#:**
```csharp
// Created via action/skill system
MinionEntity.Create(scene, position, master);
```

### Creation Parameters

**Required:**
- **Scene/Map**: World to spawn in
- **Position**: Spawn location
- **Master**: Owner entity

**Optional:**
- **Lifetime**: Duration in seconds
- **Skills**: Available skills
- **Stats**: Base statistics
- **Visual**: Appearance

## Summon Behavior

### AI States

**Following Master:**
- Follows master position
- Maintains distance
- Uses separation steering
- Teleports if too far

**Combat:**
- Targets master's enemies
- Uses available skills
- Maintains combat distance
- Flees if too close

**Idle:**
- Waits near master
- Activates on enemy detection
- Deactivates when idle too long

### Target Selection

**Target Priority:**
1. Enemy attacking master (Priority 3)
2. Enemy attacking minion (Priority 2)
3. Closest enemy (Priority 0)

**Target Criteria:**
- Must be enemy team
- Must be visible
- Must not be stealth
- Must not be invulnerable
- Must be within eyesight radius

### Combat Behavior

**Combat States:**
- **Following**: Moving toward target
- **Fleeing**: Moving away from target
- **None**: Maintaining position

**Distance Management:**
- **MinDistanceFromTarget**: Minimum combat distance
- **MaxDistanceFromTarget**: Maximum combat distance
- Adjusts position based on distance

**Skill Usage:**
- Checks skill cooldowns
- Validates skill range
- Executes skills based on chance
- Interrupts movement for casting

## Experience System

### Experience Sharing

```csharp
public override void AddExperience(int value) {
    if (Master != null) {
        Master.AddExperience(value);
    }
}
```

**Experience Behavior:**
- All experience goes to master
- Master receives experience from kills
- No experience for summon itself
- Shared with party if applicable

## Health System

### Health Regeneration

```csharp
public async void HealthRegenLoop(int healthPerSecond) {
    await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(1));
    
    while (!IsDestroyed && !IsDead) {
        if (Health < MaxHealth) {
            AddHealth(healthPerSecond);
        }
        
        await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(1));
    }
}
```

**Regeneration Behavior:**
- Regenerates health over time
- Configurable health per second
- Stops when dead or destroyed
- Continuous regeneration loop

## Damage System

### Taking Damage

```csharp
public override int TakeDamage(
    Entity eventInstigator, 
    CreatureEntity damageCauser, 
    int value, 
    DamageType damageType, 
    bool isDirectional = false, 
    bool ignoreResistence = false, 
    bool ignoreReflect = false
) {
    int damageValue = base.TakeDamage(
        eventInstigator, 
        damageCauser, 
        value, 
        damageType, 
        isDirectional, 
        ignoreResistence, 
        ignoreReflect
    );
    
    if (damageValue > 0 && TargetPriority <= 2 && Target == null) {
        Target = damageCauser;
        State = MonsterState.InCombat;
        TargetPriority = 2;
    }
    
    return damageValue;
}
```

**Damage Behavior:**
- Sets target if attacked
- Enters combat state
- Sets target priority
- Responds to threats

### Master Damage Tracking

```csharp
public void OnMasterDamageInflicted(
    Entity other, 
    DamageType damageType, 
    int value
) {
    if (Team.IsEnemyOf(other.Team) && 
        other is CreatureEntity creature && 
        value > 0 && 
        TargetPriority < 3) {
        Target = creature;
        State = MonsterState.InCombat;
        TargetPriority = 3;
    }
}
```

**Master Damage Behavior:**
- Subscribes to master damage events
- Targets enemies attacking master
- Highest priority (3)
- Enters combat immediately

## Death System

### Death Handling

```csharp
public override async void OnDie(Entity killedBy) {
    if (RemoveOnDie) {
        base.OnDie(killedBy);
        Scene.Remove(this);
    } else {
        base.OnDie(killedBy);
        
        if (Master != null) {
            Master.DamageInflicted -= OnMasterDamageInflicted;
            Master = null;
        }
        
        await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(5));
        Scene.Remove(this);
    }
}
```

**Death Options:**
- **RemoveOnDie**: Remove immediately
- **Normal**: Wait 5 seconds then remove
- Unsubscribe from master events
- Clear master reference

### Destruction

```csharp
public override void OnDestroy() {
    base.OnDestroy();
    
    if (Master != null) {
        Master.DamageInflicted -= OnMasterDamageInflicted;
        Master = null;
    }
    
    Target = null;
}
```

**Cleanup:**
- Unsubscribes from events
- Clears master reference
- Clears target
- Base cleanup

## Movement System

### Following Master

**Behavior:**
- Moves toward master position
- Maintains distance (3.0f + radius)
- Uses separation steering
- Uses avoidance steering
- Teleports if too far

**Separation:**
- Avoids other allies
- Maintains personal space
- Prevents clustering

**Avoidance:**
- Avoids obstacles
- Uses raycast detection
- Steers around walls

### Combat Movement

**Combat States:**
- **Following**: Moving toward target
- **Fleeing**: Moving away from target
- **None**: Maintaining position

**Movement Logic:**
- Calculates direction to target
- Maintains combat distance
- Adjusts position based on state
- Uses steering behaviors

## Skill System

### Skill Usage

**Skill Checks:**
- Cooldown validation
- Range validation
- Chance roll
- Resource validation

**Skill Execution:**
- Executes skill on target
- Applies skill effects
- Enters cooldown
- Broadcasts skill packet

### Skill Types

**Available Skills:**
- Defined in `Skills` array
- Each skill has chance
- Each skill has target type
- Each skill has duration

## Team System

### Team Assignment

**Team Properties:**
- **Team Kind**: Pet
- **Team Owner**: Master
- **Enemy Predicate**: Master's enemies
- **Ally Predicate**: Master's allies

**Team Behavior:**
- Inherits master's team
- Shares master's enemies
- Shares master's allies
- Follows master's PvP status

## Implementation Notes

### TypeScript vs C#

**TypeScript:**
- Simple lifetime system
- Basic following behavior
- Interval-based lifetime check

**C#:**
- Advanced AI system
- Experience sharing
- Health regeneration
- Complex combat behavior
- Steering behaviors

### Key Differences

- C# has more detailed AI
- C# has experience sharing
- C# has health regeneration
- C# has advanced movement
- TypeScript focuses on lifetime

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Base creature system
- [MOUNTS_PETS.md](./MOUNTS_PETS.md) - Mounts and pets
- [ACTIONS.md](../skills/ACTIONS.md) - Action system
- [SKILLS.md](../skills/SKILLS.md) - Skill system

