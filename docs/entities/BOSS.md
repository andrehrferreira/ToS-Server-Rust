# Boss System

## Overview

The Boss system extends the Creature class to provide specialized boss mechanics including summons, reward distribution, Steam achievements, and damage tracking for multiple players.

## Boss Class

### Base Properties

```typescript
abstract class Boss extends Creature {
    public maxSummons: number = 5
    public summons: Array<Entity>
    public individualRewards: Array<{ new (): any }>
    public steamReward: string
}
```

## Summon System

### Max Summons

**Property:**
- **maxSummons**: Maximum number of summons allowed (default: 5)
- Prevents excessive summons
- Configurable per boss

### Summons Array

**Storage:**
- **summons**: Array of Entity instances
- Tracks active summons
- Used for cleanup and management

### Create Summons

```typescript
createSummons(entityName: string, causer: Entity): void
```

**Process:**
1. Check if summons.length < maxSummons
2. Create entity using Entity.create()
3. Set transform to causer's position
4. Set respawn position
5. Set maxDistanceToRespawn (6000)
6. Select target (causer)
7. Set destroyOnDie flag
8. Subscribe to OnDie event
9. Join entity to map
10. Add to summons array

**Summon Properties:**
- Position: Copied from causer
- Target: Causer entity
- Destroy on die: True
- Max distance: 6000 units
- Auto-target: Causer

### Summon Death

```typescript
summonDie(entity: Entity): void
```

**Process:**
1. Filter summons array
2. Remove entity by mapIndex
3. Clean up reference

**Event Subscription:**
- OnDie triggers summonDie()
- Automatic cleanup
- Maintains summons array

## Reward System

### Individual Rewards

**Property:**
- **individualRewards**: Array of item classes
- Given to all players who caused damage
- Per-player rewards

**Reward Distribution:**
- Iterates damageCauser map
- Gives each player individual rewards
- Uses addItemByClass() method

### Power Scroll Rewards

**Killer Reward:**
- Causer receives 1 PowerScroll
- Immediate reward
- Always given

**Random Rewards:**
- Up to 5 random players receive PowerScroll
- Selected from playersWhoCausedDamage set
- Random selection per reward
- Can receive multiple scrolls

**Selection Process:**
1. Create Set of players who caused damage
2. Exclude causer from set
3. Convert to array
4. Randomly select up to 5 players
5. Give PowerScroll to each

## Steam Achievements

### Steam Reward

**Property:**
- **steamReward**: Achievement name string
- Optional Steam achievement
- Awarded to all damage causers

**Award Process:**
- Iterates damageCauser map
- Calls setArchivement() for each player
- Uses steamReward property

## Damage Tracking

### Damage Causer Map

**Storage:**
- **damageCauser**: Map<string, number>
- Key: mapIndex (entity ID)
- Value: Total damage dealt
- Tracks all players who damaged boss

**Usage:**
- Used for reward distribution
- Tracks participation
- Prevents duplicate rewards

### Damage Causer Iteration

**Process:**
1. Iterate damageCauser map
2. Find entity by mapIndex
3. Check if Player instance
4. Add to playersWhoCausedDamage set
5. Distribute rewards

## Death Override

### Die Method

```typescript
override async die(causer: Entity): Promise<void>
```

**Process:**
1. Call super.die(causer)
2. Give PowerScroll to causer
3. Create playersWhoCausedDamage set
4. Iterate damageCauser map
5. Distribute individual rewards
6. Award Steam achievements
7. Randomly select 5 players for PowerScroll
8. Give PowerScroll to selected players

**Async Operations:**
- Uses async/await
- Handles item creation
- Proper error handling

## Boss Implementation

### Example Boss

```typescript
class ReaperBoss extends Boss {
    constructor() {
        super()
        this.maxSummons = 3
        this.individualRewards = [GoldCoin, HealthPotion]
        this.steamReward = "REAPER_KILLED"
    }
}
```

**Customization:**
- Set maxSummons
- Define individual rewards
- Set Steam achievement
- Customize behavior

## Summon Management

### Summon Lifecycle

1. **Creation**: createSummons() called
2. **Targeting**: Auto-targets causer
3. **Combat**: Fights causer
4. **Death**: OnDie triggered
5. **Cleanup**: summonDie() called
6. **Removal**: Removed from array

### Summon Limits

**Max Summons:**
- Prevents spam
- Configurable
- Per-boss setting

**Destroy on Die:**
- Summons destroyed on death
- No respawn
- Temporary entities

## Reward Distribution

### Reward Types

**Individual Rewards:**
- Given to all participants
- Same items for all
- Participation reward

**Power Scroll Rewards:**
- Killer: Always receives
- Random: Up to 5 players
- Valuable reward

**Steam Achievements:**
- Optional achievement
- All participants
- Steam integration

### Distribution Logic

**Fair Distribution:**
- All damage causers rewarded
- Random selection for PowerScroll
- Prevents favoritism

## Implementation Notes

### Performance

**Optimizations:**
- Efficient array filtering
- Set-based tracking
- Minimal overhead

### Thread Safety

**Async Operations:**
- Proper async/await usage
- Error handling
- Cleanup on errors

### Extensibility

**Custom Bosses:**
- Extend Boss class
- Override die() if needed
- Customize rewards
- Set summon limits

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Creature system
- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [REWARDS.md](../admin/REWARD.md) - Reward system

