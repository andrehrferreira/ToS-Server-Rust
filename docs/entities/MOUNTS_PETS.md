# Mounts and Pets System

## Overview

Mounts and pets are special creature types that belong to players. Mounts provide faster movement and can be ridden, while pets are companion creatures that follow and assist their owners. Both systems integrate with the equipment system and have unique behaviors.

## Mount System

### Mount Class (TypeScript)

```typescript
class Mount extends Creature {
    public owner: Player;
    
    constructor(owner: Player) {
        super();
        this.kind = EntitiesKind.Mount;
        this.team = new Team(TeamKind.Pet, this);
        this.teamOwner = owner;
        this.owner = owner;
        this.guild = owner.guild;
        this.life = 10000;
    }
}
```

### Mount Entity (C#)

```csharp
public class MountEntity : MinionEntity {
    public CharacterEntity Owner { get; init; }
    
    public override void Initialize() {
        base.Initialize();
        Kind = CreatureKind.Mount;
    }
}
```

### Mount Properties

**Base Properties:**
- **owner**: Player who owns the mount
- **team**: Team set to Pet team
- **teamOwner**: Owner for team purposes
- **guild**: Inherits owner's guild
- **life**: Base life value (10000)

**AI Behavior:**
- Always follows master
- Cannot be targeted by owner
- Shares team with owner
- Inherits guild affiliation

### Mount Health System

**Health Management:**
- Mount has separate health pool
- Health stored in equipment item
- Health updates on damage/heal
- Mount unequipped if health reaches 0

**Health Updates:**
```csharp
protected override int OnTakeDamage(int value, DamageType type) {
    int returnValue = base.OnTakeDamage(value, type);
    
    if (Owner?.Equipments.Mount != null) {
        if (Health <= 0) {
            Owner.Equipments.Mount = null;
        } else {
            Owner.Equipments.Mount.Health = (uint)Math.Min(
                Math.Max(Health, 0), 
                50000
            );
            Owner.Equipments.OnItemHealthChanged(
                Owner.Equipments.Mount, 
                (uint)(EquipmentItemSlot.Mount), 
                Owner.Equipments.Mount.Health
            );
        }
    }
    
    return returnValue;
}
```

### Mount Interaction

**Mounting:**
- Player interacts with mount entity
- Mount must belong to player
- Mount must not be dead
- Triggers mount action

**Unmounting:**
- Player dismounts from mount
- Global cooldown applied (3 seconds)
- Movement speed reset
- Mount entity remains

### Mount Visual

**Create Packet:**
- Mount appears as Mount kind to owner
- Mount appears as Monster kind to others
- Visual based on mount item

## Pet System

### Pet Class (TypeScript)

```typescript
class Pet extends Creature {
    public override creatureType = CreatureType.Pet;
    public override passive: boolean = true;
    
    constructor(owner: Player) {
        super();
        this.kind = EntitiesKind.Pet;
        this.team = new Team(TeamKind.Pet, this);
        this.teamOwner = owner;
        this.owner = owner;
        this.guild = owner.guild;
        this.life = 10000;
    }
}
```

### Pet Properties

**Base Properties:**
- **creatureType**: Set to Pet
- **passive**: Always passive (won't attack)
- **owner**: Player who owns the pet
- **team**: Team set to Pet team
- **teamOwner**: Owner for team purposes
- **guild**: Inherits owner's guild
- **life**: Base life value (10000)

**AI Behavior:**
- Always follows master
- Passive (doesn't attack)
- Cannot be targeted by owner
- Shares team with owner

### Pet Damage Handling

**Taking Damage:**
```typescript
takeDamage(
    causer: Entity, 
    dice: Dices, 
    damageType: DamageType, 
    bonusDamage: number = 0, 
    action: BaseAction = null
): void {
    this.IAState = CreatureIAState.FollowingMaster;
}
```

- Pet doesn't take damage normally
- Damage causes pet to follow master
- Pet protected from combat

### Pet AI States

**State Transitions:**
- **Idle → FollowingMaster**: Always transitions
- **Patrol → FollowingMaster**: Always transitions
- **InCombat → FollowingMaster**: Always transitions
- **GoingToSpawn → FollowingMaster**: Always transitions

**Following Behavior:**
- Follows owner position
- Maintains minimum distance
- Updates position each tick
- Smooth movement with lerp

## Minion Entity (C#)

### Minion Base Class

```csharp
public class MinionEntity : MonsterBase {
    public float Lifetime;
    public CreatureEntity Master;
    private CombatState CurrentCombatState;
    private float FleeTimer;
    private Vector2 steerForWallDirection;
    public bool RemoveOnDie = false;
    
    public override CreatureEntity Parent => Master ?? this;
}
```

### Minion Properties

**Base Properties:**
- **Lifetime**: Time until despawn
- **Master**: Owner/master entity
- **RemoveOnDie**: Remove immediately on death
- **Parent**: Master or self

**Combat States:**
- **Following**: Following master
- **Fleeing**: Fleeing from threat
- **None**: No specific combat state

### Minion AI

**Idle State:**
- Follows master if too far
- Picks target if enemies nearby
- Uses separation steering
- Deactivates if idle too long

**Following Master State:**
- Moves toward master position
- Maintains distance (3.0f + radius)
- Uses separation and avoidance steering
- Teleports if too far

**In Combat State:**
- Targets enemies
- Uses skills when available
- Maintains combat distance
- Flees if too close
- Follows if too far

### Minion Targeting

**Target Priority:**
- **Priority 0**: No target
- **Priority 2**: Attacked by enemy
- **Priority 3**: Master attacked enemy

**Target Selection:**
- Closest enemy in eyesight radius
- Must be visible
- Must be enemy team
- Must not be stealth/invulnerable

### Minion Experience

**Experience Sharing:**
```csharp
public override void AddExperience(int value) {
    if (Master != null) {
        Master.AddExperience(value);
    }
}
```

- Experience goes to master
- Master receives all experience
- No experience for minion itself

### Minion Health Regen

**Regeneration Loop:**
```csharp
public async void HealthRegenLoop(int healthPerSecond) {
    while (!IsDestroyed && !IsDead) {
        if (Health < MaxHealth) {
            AddHealth(healthPerSecond);
        }
        await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(1));
    }
}
```

- Regenerates health over time
- Configurable health per second
- Stops when dead or destroyed

### Minion Lifetime

**Lifetime Management:**
```csharp
public override async void Initialize() {
    Master.DamageInflicted += OnMasterDamageInflicted;
    base.Initialize();
    
    if (!float.IsInfinity(Lifetime) && Lifetime > 0) {
        await Scene.TimerQueue.Delay(TimeSpan.FromSeconds(Lifetime));
        Die(Master);
    }
}
```

- Minions have limited lifetime
- Despawn after lifetime expires
- Lifetime can be infinite
- Dies when lifetime expires

### Minion Damage Handling

**Taking Damage:**
- Sets target if attacked
- Enters combat state
- Sets target priority
- Master damage triggers targeting

**Master Damage:**
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

### Minion Death

**Death Handling:**
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

## Equipment Integration

### Mount Equipment

**Mount Item:**
- Equipped in mount slot
- Contains mount entity data
- Health stored in item
- Visual from item

**Mount Equip:**
```typescript
case EquipamentType.Mount: 
    this.mount = equipamentRef; 
    (equipament as MountItem).onEquip(this);
break;
```

**Mount Desequip:**
```typescript
case EquipamentType.Mount: 
    if (this.mount) {
        const mountRef = this.mount.ItemRef;
        this.mount = null;
        this.addToInventory(mountRef, 1, slotId);
        (Items.getItemByRef(mountRef) as MountItem)?.onDesequip(this);
        // ... broadcast and save
    }
break;
```

### Pet Equipment

**Pet Item:**
- Equipped in pet slot
- Contains pet entity data
- Visual from item
- Pet instance created

**Pet Equip:**
```typescript
case EquipamentType.Pet: 
    this.pet = equipamentRef; 
    (equipament as PetItem).onEquip(this);
break;
```

**Pet Desequip:**
```typescript
case EquipamentType.Pet: 
    if (this.pet) {
        const petRef = this.pet.ItemRef;
        this.pet = null;
        this.addToInventory(petRef, 1, slotId);
        (Items.getItemByRef(petRef) as PetItem)?.onDesequip(this);
        // ... broadcast and save
    }
break;
```

## Team System

### Team Assignment

**Mount/Pet Teams:**
- **Team Kind**: Pet
- **Team Owner**: Player owner
- **Enemy Check**: Based on owner's enemies
- **Ally Check**: Based on owner's allies

**Team Inheritance:**
- Inherits owner's team relationships
- Shares guild with owner
- Follows owner's PvP status

## Implementation Notes

### TypeScript vs C#

**TypeScript:**
- Simple mount/pet classes
- Basic following behavior
- Integrated with equipment

**C#:**
- More complex minion system
- Advanced AI with combat states
- Experience sharing
- Health regeneration
- Lifetime management

### Key Differences

- C# has more detailed AI
- C# has experience sharing
- C# has health regeneration
- C# has lifetime system
- TypeScript focuses on basic following

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Base creature system
- [HUMANOIDS.md](./HUMANOIDS.md) - Equipment system
- [SUMMONS.md](./SUMMONS.md) - Summoned entities
- [ITEMS.md](../items/ITEMS.md) - Item system

