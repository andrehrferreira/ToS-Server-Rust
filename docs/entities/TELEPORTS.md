# Teleport Entities

## Overview

Teleport entities provide portal and waypoint functionality for map transitions, event portals, and instanced map access. Teleports handle collision detection, team assignment, and map transitions.

## TeleportEntity Class

### Base Properties

```csharp
class TeleportEntity : Entity {
    public string SceneId
    public string SceneName
    public string WaypointName
    public string[] WaypointRandom
    public bool IsEventPortal
    public bool IsInstanced
    public List<Guid> PlayersId
}
```

## Teleport Types

### Standard Teleport

**Properties:**
- SceneName: Target map
- WaypointName: Target waypoint
- Standard map transition

**Usage:**
- Map transitions
- Waypoint travel
- Standard portals

### Event Portal

**Properties:**
- IsEventPortal: true
- WaypointRandom: Random waypoints
- PlayersId: Tracked players
- Team assignment

**Usage:**
- Event entry
- Team assignment
- Event portals
- Battle Castle entry

### Instanced Portal

**Properties:**
- IsInstanced: true
- SceneId: Instance ID
- Async initialization
- Instance management

**Usage:**
- Instanced maps
- Event instances
- Private areas
- Dynamic maps

## Collision Detection

### Collision Shape

**Shape:**
- Rectangle collision
- Configurable size
- Blocks creatures
- Blocks projectiles

**Properties:**
- BlocksCreature: true
- BlocksProjectile: true
- Collision detection
- Interaction trigger

### OnCollide Method

```csharp
override async void OnCollide(Entity other)
```

**Process:**
1. Check if CharacterEntity
2. Check if not dead
3. Check battle timer
4. Check teleport cooldown
5. Handle teleport based on type

## Teleport Logic

### Standard Teleport

**Process:**
1. Check teleport cooldown
2. Validate destination
3. Teleport player
4. Set cooldown

**Cooldown:**
- 10 seconds
- Prevents spam
- Time-based

### Event Portal

**Process:**
1. Check if player already entered
2. Assign team (Team1/Team2)
3. Select random waypoint
4. Teleport to event
5. Track player

**Team Assignment:**
- Alternating teams
- Balanced assignment
- Team1/Team2 counters
- Random waypoint selection

### Instanced Portal

**Process:**
1. Initialize instance async
2. Wait for initialization
3. Get instance ID
4. Teleport to instance
5. Set cooldown

**Initialization:**
- Async task
- Database creation
- Instance ID generation
- Ready state

## Waypoint System

### Waypoint Name

**Single Waypoint:**
- WaypointName: Target waypoint
- Direct teleport
- Fixed destination

### Random Waypoints

**Multiple Waypoints:**
- WaypointRandom: Array of waypoints
- Random selection
- Team-based assignment
- Event portals

**Selection:**
- Team-based (Team1/Team2)
- Alternating assignment
- Random fallback
- Event-specific

## Player Tracking

### Players ID List

**Tracking:**
- List<Guid> PlayersId
- Tracks entered players
- Prevents re-entry
- Event management

**Usage:**
- Event portals
- One-time entry
- Player tracking
- Event participation

## Teleport Cooldown

### Cooldown System

**Cooldown:**
- 10 seconds default
- Prevents spam
- Time-based check
- Player-specific

**Message:**
- Shows time remaining
- Prevents teleport
- User feedback
- Cooldown display

## Implementation Notes

### Async Operations

**Async Handling:**
- Instance initialization
- Database operations
- Proper await usage
- Error handling

### Thread Safety

**Concurrency:**
- Thread-safe operations
- Proper locking
- Safe state management

### Performance

**Optimizations:**
- Efficient collision checks
- Minimal overhead
- Batch operations

## Related Documentation

- [TELEPORTATION.md](../server/TELEPORTATION.md) - Teleportation system
- [MAPS.md](../server/MAPS.md) - Maps system
- [EVENT_INSTANCE.md](../events/EVENT_INSTANCE.md) - Event instances

