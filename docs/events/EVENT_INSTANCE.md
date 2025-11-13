# Event Instance

## Overview

The Event Instance system manages instanced events such as Tower Defense, Battle Castle, Moba, Card Game, and Arena. Each event creates its own map instance and manages player participation.

## EventInstance Base Class

### Base Properties

```typescript
abstract class EventInstance {
    public static EventsBase: Map<EventInstanceType, { new (): any }>
    public type: EventInstanceType
    public abstract MapNamespace: string
    public mapId: string
    public map: Maps
    public players: Array<Player>
}
```

## Event Types

### EventInstanceType Enum

```typescript
enum EventInstanceType {
    TowerDefense,
    BattleCastle,
    Moba,
    CardGame,
    Arena
}
```

**Event Types:**
- **TowerDefense**: Tower defense gameplay
- **BattleCastle**: Large-scale PvP battle
- **Moba**: MOBA-style gameplay
- **CardGame**: Card game events
- **Arena**: Arena combat

## Event Queue Types

### EventQueueType Enum

```typescript
enum EventQueueType {
    Solo,
    Party,
    Queue
}
```

**Queue Types:**
- **Solo**: Individual players
- **Party**: Party-based entry
- **Queue**: Matchmaking queue

## Event Registration

### Add Event Base

```typescript
static AddEventBase(eventType: EventInstanceType, event: { new (): any }): void
```

**Process:**
1. Register event class
2. Store in EventsBase map
3. Keyed by EventInstanceType
4. Used for event creation

### Get Event Base

```typescript
static GetEventBase(eventType: EventInstanceType): { new (): any } | null
```

**Process:**
1. Lookup in EventsBase map
2. Return event class or null
3. Used for event creation

## Event Creation

### Create Event

```typescript
static CreateEvent(eventType: EventInstanceType, mapsService: MapsService): EventInstance | null
```

**Process:**
1. Get event base class
2. Create event instance
3. Create map instance
4. Get map reference
5. Return event instance

**Map Instance:**
- Created using Maps.createInstance()
- Unique map ID (GUID)
- Isolated from other maps
- Event-specific namespace

## Event Map

### Map Namespace

**Abstract Property:**
- Each event defines MapNamespace
- Used for map creation
- Event-specific map

### Map Instance

**Properties:**
- **mapId**: Unique instance ID
- **map**: Maps instance reference
- Isolated from other instances
- Event-specific entities

## Player Management

### Join Event

```typescript
Join(player: Player): void
```

**Process:**
1. Add player to players array
2. Player can now participate
3. Event-specific logic

**Player Array:**
- Tracks event participants
- Used for event management
- Team assignment
- Rewards distribution

## Event Lifecycle

### Event Creation

1. **Register**: Event class registered
2. **Create**: Event instance created
3. **Map**: Map instance created
4. **Initialize**: Event initialized
5. **Ready**: Event ready for players

### Event Execution

1. **Join**: Players join event
2. **Start**: Event starts
3. **Progress**: Event progresses
4. **End**: Event ends
5. **Cleanup**: Event cleaned up

## Tower Defense Event

### Implementation

**Features:**
- Wave-based gameplay
- Tower placement
- Enemy spawning
- Resource management

**Map:**
- Event-specific map
- Tower placement zones
- Enemy spawn points
- Player areas

## Battle Castle Event

### Implementation

**Features:**
- Large-scale PvP
- Two teams
- Capture points
- Scoring system

**Map:**
- Castle map instance
- Team spawn points
- Capture zones
- Battle areas

## Moba Event

### Implementation

**Features:**
- MOBA-style gameplay
- Lanes and objectives
- Team-based combat
- Progression system

**Map:**
- MOBA map instance
- Lane structures
- Objective points
- Base areas

## Card Game Event

### Implementation

**Features:**
- Card-based gameplay
- Deck building
- Turn-based combat
- Strategy elements

**Map:**
- Card game arena
- Player areas
- Card zones
- Battle field

## Arena Event

### Implementation

**Features:**
- Arena combat
- Player vs Player
- Tournament brackets
- Rewards system

**Map:**
- Arena map instance
- Combat zones
- Spectator areas
- Spawn points

## Map Instance Management

### Instance Creation

**Process:**
1. Generate unique ID (GUID)
2. Create Maps instance
3. Use event namespace
4. Store in maps registry
5. Return instance ID

### Instance Isolation

**Isolation:**
- Separate from main maps
- Unique entity registry
- Isolated respawns
- Event-specific data

### Instance Cleanup

**Cleanup:**
- Remove when event ends
- Clean up entities
- Remove from registry
- Free resources

## Player Participation

### Join Requirements

**Requirements:**
- Valid player
- Event availability
- Queue type match
- Permission check

### Join Process

1. **Validate**: Check requirements
2. **Queue**: Add to queue (if needed)
3. **Assign**: Assign to event
4. **Teleport**: Teleport to event map
5. **Initialize**: Initialize player in event

### Leave Process

1. **Save**: Save player progress
2. **Teleport**: Teleport to safe location
3. **Remove**: Remove from event
4. **Cleanup**: Clean up event data

## Event Queue System

### Solo Queue

**Behavior:**
- Individual players
- Matchmaking
- Balanced teams
- Quick join

### Party Queue

**Behavior:**
- Party-based entry
- Keep party together
- Team assignment
- Group participation

### Queue Queue

**Behavior:**
- Matchmaking system
- Skill-based matching
- Wait for players
- Balanced teams

## Implementation Notes

### Performance

**Optimizations:**
- Isolated map instances
- Efficient player management
- Event-specific optimizations
- Resource cleanup

### Persistence

**Event Data:**
- Event state
- Player participation
- Event progress
- Rewards

### Thread Safety

**Async Operations:**
- Map creation async
- Player management
- Event updates
- Cleanup operations

## Related Documentation

- [BATTLECASTLE.md](./BATTLECASTLE.md) - Battle Castle event
- [MAPS.md](../server/MAPS.md) - Maps system
- [PARTY.md](../player/PARTY.md) - Party system

