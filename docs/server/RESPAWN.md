# Respawn

## Overview

The Respawn system manages entity respawning after death. Each respawn point can spawn one entity at a time, with configurable timers and entity selection. Respawns are integrated with the Maps system and persist to the database.

## Respawn Class

### Properties

```typescript
class Respawn {
    public Id: string
    public Settings: IRespawn
    public Map: Maps
    public EntityRespawned: Entity
    private Removed: boolean
}
```

### Respawn Settings Interface

```typescript
interface IRespawn {
    map: string
    respawnOnStart: boolean
    timeout: number (timestamp)
    timer: number (seconds)
    x: number
    y: number
    z: number
    entities: string[] (entity names)
}
```

## Respawn Lifecycle

### Initialization

**Constructor:**
```typescript
constructor(id: string, settings: IRespawn)
```

**Process:**
1. Store ID and settings
2. Get map reference
3. If respawnOnStart: create entity immediately
4. Otherwise: wait for timeout

### Entity Creation

**createEntity:**
```typescript
createEntity(): void
```

**Process:**
1. Destroy existing entity if present
2. Select random entity from entities array
3. Create entity using Entity.create()
4. Set respawn reference
5. Subscribe to OnDie and OnDestroy events
6. Set transform and respawn position
7. Join entity to map

**Entity Selection:**
- Random selection from entities array
- Supports multiple entity types per respawn
- Equal probability for each type

### Entity Death

**entityDie:**
```typescript
entityDie(): void
```

**Process:**
1. Set new timeout (current time + timer)
2. Clear EntityRespawned reference
3. Entity will respawn on next tick if timeout passed

**Event Subscription:**
- OnDie: triggers entityDie()
- OnDestroy: triggers entityDie()
- Automatic cleanup

### Tick System

**tick:**
```typescript
tick(): void
```

**Conditions:**
- No entity currently spawned
- Respawn not removed
- Current time > timeout

**Process:**
1. Check conditions
2. If met: update timeout and create entity
3. Otherwise: wait for next tick

## Respawn Management

### Destroy

**destroy:**
```typescript
destroy(): void
```

**Process:**
1. Set Removed flag to true
2. Prevents further respawning
3. Entity can still exist until destroyed

### Remove Respawn

**removeRespawn:**
```typescript
removeRespawn(): void
```

**Process:**
1. Set Removed flag
2. Call map.removeRespawn()
3. Map handles database removal

## Respawn Types

### Normal Respawn

**Timer:**
- Default: 300 seconds (5 minutes)
- Configurable per respawn
- Bosses: 86400 seconds (24 hours)

**Behavior:**
- Standard respawn cycle
- Entity dies → timer starts → respawns

### Respawn On Start

**Behavior:**
- Entity spawned immediately on map load
- No initial delay
- Used for initial world population

## Integration with Maps

### Map Respawn Storage

**Storage:**
- Maps.respawns: Map<string, Respawn>
- Indexed by respawn ID
- Managed by Maps class

### Respawn Loading

**getRespawns:**
- Loads from database
- Creates Respawn instances
- Sets appropriate timers
- Stores in map

### Respawn Creation

**createRespawn:**
- Creates in database
- Creates Respawn instance
- Adds to map respawns
- Returns respawn ID

### Respawn Removal

**removeRespawn:**
- Removes from database
- Destroys Respawn instance
- Removes from map
- Destroys spawned entity

## Timer System

### Timer Calculation

**Boss Entities:**
- Timer: 86400 seconds (24 hours)
- Long respawn time
- Rare encounters

**Normal Entities:**
- Timer: 300 seconds (5 minutes)
- Standard respawn time
- Common encounters

**Custom Timers:**
- Can be set per respawn
- Stored in database
- Applied on load

### Timeout Management

**Timeout Format:**
- Unix timestamp (milliseconds)
- Calculated: current time + (timer * 1000)
- Updated on entity death
- Checked each tick

## Entity Selection

### Random Selection

**Selection Process:**
1. Get entities array length
2. Generate random index (0 to length-1)
3. Select entity name at index
4. Create entity with that name

**Multiple Entity Types:**
- Supports array of entity names
- Random selection each respawn
- Equal probability
- Allows variety

## Event Subscriptions

### OnDie Subscription

```typescript
newEntity.OnDie.subscribe({ 
    next: (entity) => entity.respawn.entityDie() 
})
```

**Behavior:**
- Triggers on entity death
- Calls entityDie()
- Starts respawn timer

### OnDestroy Subscription

```typescript
newEntity.OnDetroy.subscribe({ 
    next: (entity) => entity.respawn.entityDie() 
})
```

**Behavior:**
- Triggers on entity destruction
- Calls entityDie()
- Starts respawn timer
- Handles cleanup

## Position Management

### Respawn Position

**Position Storage:**
- x, y, z coordinates
- Stored in settings
- Used for entity spawn

**Transform Creation:**
```typescript
new Transform(
    new Vector3(settings.x, settings.y, settings.z),
    Rotator.Zero(),
    Vector3.One()
)
```

**Position Assignment:**
- Set entity.transform
- Set entity.respawnPosition
- Used for return behavior

## Database Integration

### Respawn Data

**Stored Fields:**
- id: Unique identifier
- map: Map namespace
- respawnOnStart: Boolean flag
- timeout: Timestamp
- timer: Seconds
- x, y, z: Position
- entities: JSON array of entity names

### Loading

**Load Process:**
1. Query database for map respawns
2. Parse entities JSON
3. Create Respawn instances
4. Store in map

### Saving

**Save Process:**
1. Create respawn data object
2. Save to database
3. Create Respawn instance
4. Store in map

### Removal

**Remove Process:**
1. Delete from database
2. Destroy Respawn instance
3. Remove from map
4. Clean up entity

## Implementation Notes

### Performance

**Optimizations:**
- Tick only when needed
- Check timeout efficiently
- Minimal overhead
- Batch operations

### Thread Safety

**Async Operations:**
- Database operations async
- Proper await usage
- Error handling
- Cleanup on errors

### Error Handling

**Entity Creation:**
- Null check on Entity.create()
- Handles invalid entity names
- Graceful failure

**Map Reference:**
- Null check on map
- Handles map not found
- Safe operations

## Related Documentation

- [MAPS.md](./MAPS.md) - Maps system
- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [GATHERING.md](../player/GATHERING.md) - Gathering system

