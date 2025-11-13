# Creatures System

## Overview

Creatures are the base class for all living entities in the game world, including monsters, animals, NPCs, pets, mounts, and summons. The `Creature` class provides AI state management, combat behavior, targeting, movement, and action execution.

## Creature Class

### Base Properties

```typescript
abstract class Creature extends Entity {
    protected IAState: CreatureIAState
    public creatureType: CreatureType
    public owner: Player
    
    public isCreature: boolean
    public idleTick: number
    public moveToPosition: Vector3
    public lastMoveToPosition: Vector3
    public passive: boolean
    public pawnSenseRadius: number
    public baseDamage: Dices
    public maxRandomOffset: number
    public minDistanceTarget: number
    
    // Special Actions
    public inAction: boolean
    public actions: Map<ActionRef, number>
    public actionsArrList: Array<ActionRef>
    
    // Target
    public targetMode: CreatureTargetMode
    public combatMode: CreatureCombatMode
    
    // Movement
    public lerpStartTime: number
    public lerpDuration: number
    public startPosition: Vector3
    
    // Taming
    public tamable: boolean
    public skillTamingMin: number
    public petSockets: number
    public isTamed: boolean
    public maxSkillTamingUp: number
    public itemTamingFinish: { new (): any }
}
```

## Creature Types

### CreatureType Enum

```typescript
enum CreatureType {
    None,
    Animals,      // Neutral animals
    Undead,       // Undead creatures
    NPC,          // Non-player characters
    Summon,       // Summoned entities
    Pet,          // Tamed pets
    Dragon,       // Dragon-type creatures
    Common,       // Common monsters
    Demon         // Demon-type creatures
}
```

## AI States

### CreatureIAState

Creatures use a state machine for AI behavior:

- **Idle**: Creature is idle, waiting for events
- **Patrol**: Creature is patrolling random positions
- **InCombat**: Creature is in combat with target
- **GoingToSpawn**: Creature returning to spawn point
- **FollowingMaster**: Creature following owner/master

### State Transitions

**Idle → Patrol:**
- After `idleTick` exceeds `nextMovement`
- Random interval between 10-20 ticks

**Idle → InCombat:**
- Target acquired
- Enemy detected in sense radius

**Patrol → InCombat:**
- Target acquired during patrol
- Enemy detected

**InCombat → GoingToSpawn:**
- Target lost or too far from spawn
- Beyond `maxDistanceToRespawn`

**InCombat → Idle:**
- Target cleared
- Combat ended

**GoingToSpawn → Idle:**
- Reached spawn area
- After 100 ticks idle

## Targeting System

### Target Modes

**CreatureTargetMode:**

- **Closer**: Target closest enemy
- **ShorterLife**: Target enemy with lowest life
- **DamageCaused**: Target enemy that caused most damage

### Target Selection Logic

```typescript
trackTarget() {
    const enemies = this.getEnemiesInRadius(
        this.transform.position, 
        this.pawnSenseRadius
    );
    
    switch(this.targetMode) {
        case CreatureTargetMode.Closer:
            // Find closest enemy
            break;
        case CreatureTargetMode.ShorterLife:
            // Find weakest enemy
            break;
        case CreatureTargetMode.DamageCaused:
            // Find enemy that caused most damage
            break;
    }
}
```

### Target Validation

- Target must exist and not be dead
- Target must be within sense radius
- Target must be enemy (team check)
- Target cleared if invalid

## Combat Modes

### CreatureCombatMode

- **Melee**: Close-range combat
- **Ranged**: Long-range combat with positioning

### Combat Behavior

**Melee Combat:**
- Moves directly to target
- Maintains `minDistanceTarget` from target
- Attacks when in range

**Ranged Combat:**
- Maintains distance from target
- Uses random offset positioning
- Attacks from optimal range

## Movement System

### Patrol Movement

```typescript
getRandomPosition() {
    const moveDistance = Math.min(this.movementDistance, 1000);
    const angle = Math.random() * Math.PI * 2;
    const dx = Math.cos(angle) * moveDistance;
    const dy = Math.sin(angle) * moveDistance;
    
    return new Vector3(
        this.respawnPosition.x + dx,
        this.respawnPosition.y + dy,
        this.respawnPosition.z
    );
}
```

### Combat Movement

- Moves toward target position
- Uses lerp for smooth movement
- Maintains combat distance
- Returns to spawn if too far

### Following Master

- Follows owner position
- Maintains `minDistanceTarget`
- Updates position each tick

## Action System

### Action References

```typescript
class ActionRef {
    action: BaseAction
    targetType: ActionType
    allDamagesActors: boolean
}
```

### Action Execution

**Action Types:**
- **TargetSelf**: Action on self
- **DirectionalCamera**: Directional action
- **Target**: Action on target entity
- **Area**: Area-of-effect action

**Action Selection:**
- Random selection from `actionsArrList`
- Weighted by chance value
- Cooldown and resource checks

**Action Execution Flow:**
1. Validate target and action
2. Set `inAction` flag
3. Broadcast action packet
4. Execute action after pre-cast delay
5. Reset `inAction` flag

## Damage System

### Auto-Attack

```typescript
checkHitAutoAttack(data: ICheckHitAutoAttack) {
    const actor = this.map.findEntityById(data.actorId);
    actor.takeDamage(
        this, 
        this.baseDamage, 
        DamageType.Physic, 
        0
    );
}
```

### Taking Damage

- Validates attacker
- Sets target if no current target
- Enters combat state
- Updates damage tracking

### Base Damage

- Uses `baseDamage` dice
- Rolled with `rollDice()`
- Modified by stats and equipment

## Stat System

### Stat Setters

Creatures can have randomized stats:

```typescript
setLife(min: number, max?: number)
setStr(min: number, max?: number)
setDex(min: number, max?: number)
setInt(min: number, max?: number)
setVig(min: number, max?: number)
setAgi(min: number, max?: number)
setLuc(min: number, max?: number)
```

### Resistance Setters

```typescript
setPhysicalResistence(min: number, max?: number)
setFireResistence(min: number, max?: number)
setColdResistence(min: number, max?: number)
setPoisonResistence(min: number, max?: number)
setEnergyResistence(min: number, max?: number)
setLightResistence(min: number, max?: number)
setDarkResistence(min: number, max?: number)
setAllResistence(min: number, max?: number)
```

## Taming System

### Taming Properties

- **tamable**: Can be tamed
- **skillTamingMin**: Minimum taming skill required
- **petSockets**: Number of pet slots when tamed
- **isTamed**: Currently tamed status
- **maxSkillTamingUp**: Maximum skill gain from taming
- **itemTamingFinish**: Item class for taming completion

### Taming Process

1. Player uses taming skill/item
2. Check `skillTamingMin` requirement
3. Check `tamable` flag
4. Apply taming attempt
5. Grant skill experience
6. Create pet item on success

## Skinning System

### Skinning Properties

- **skinnerTick**: Number of skinning attempts remaining
- **skinnerResources**: Resource class to create
- **skinnerAmount**: Base amount per skinning
- **skinnerGainExp**: Skill experience per skinning

### Skinning Process

```typescript
skinning(entity: Player) {
    // Validate creature is dead
    // Check distance
    // Decrement skinnerTick
    // Create items based on skill level
    // Grant skill experience
    // Dissolve entity when complete
}
```

## Death and Loot

### Death Handling

```typescript
die(causer: Entity) {
    super.die(causer);
    
    if (causer instanceof Player) {
        if (!this.destroyOnDie) {
            await this.loot.generateLoot(causer);
            
            if (this.skinnerTick === 0 && this.loot.count() <= 0) {
                this.dieTimeout = new Date().getTime() + (5 * 1000);
            }
        }
    }
}
```

### Loot Generation

- Generates loot for killer (if Player)
- Skinning available if `skinnerTick > 0`
- Corpse dissolves after timeout
- Loot container created

## Tick System

### Update Cycle

```typescript
tick(tickNumber: number) {
    super.tick(tickNumber);
    
    if (!this.isDead && tickNumber % 5 === 0) {
        this.validateTarget();
        
        if (!this.target && !this.passive && tickNumber % 10 === 0) {
            this.trackTarget();
        }
        
        // State machine update
        switch(this.IAState) {
            case CreatureIAState.Idle: 
                this.stateIdle(tickNumber); 
                break;
            // ... other states
        }
    }
}
```

### Tick-Based Operations

- **Every tick**: Basic updates
- **Every 5 ticks**: Target validation
- **Every 10 ticks**: Target tracking (if no target)
- **Every 10 ticks**: Action casting (in combat)

## Spawn System

### Respawn Position

- **respawnPosition**: Original spawn location
- **maxDistanceToRespawn**: Maximum distance before returning
- Returns to spawn if too far from target

### Spawn Behavior

- Returns to spawn when:
  - Target lost
  - Too far from spawn
  - Combat ended
  - Health regenerates at spawn

## Implementation Notes

### TypeScript vs C#

**TypeScript (`creature.ts`):**
- Abstract base class
- State machine implementation
- Action system with ActionRef
- Tick-based updates

**C# (`CreatureEntity.cs`):**
- Base for all creature types
- More detailed combat system
- Condition system
- Damage counters
- Floating text system

### Key Differences

- C# version has more detailed damage calculation
- C# version includes condition system (stun, slow, etc.)
- C# version has damage type counters
- TypeScript version focuses on AI states

## Related Documentation

- [HUMANOIDS.md](./HUMANOIDS.md) - Humanoid creatures with equipment
- [MOUNTS_PETS.md](./MOUNTS_PETS.md) - Mount and pet implementations
- [SUMMONS.md](./SUMMONS.md) - Summoned entities
- [NPCS.md](./NPCS.md) - Non-player characters
- [ACTIONS.md](../skills/ACTIONS.md) - Action system

