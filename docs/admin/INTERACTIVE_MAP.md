# Interactive Map System

## Overview

The interactive map provides real-time visualization of the game world, showing player locations, creatures, NPCs, waypoints, events, and zone boundaries.

## Map Features

**Displayed Information:**
- Player locations (real-time)
- Creature locations
- NPC locations
- Waypoints
- Active events
- Zone boundaries
- Respawn points

**Interactive Features:**
- Click to view entity details
- Filter by entity type
- Search for specific players/entities
- Zoom in/out
- Pan across map
- Real-time updates via SSE

**Implementation:**
```rust
pub struct InteractiveMap {
    map_data: Arc<RwLock<MapData>>,
    entity_positions: Arc<RwLock<HashMap<EntityId, Position>>>,
}

pub struct MapData {
    pub map_id: MapId,
    pub map_name: String,
    pub bounds: BoundingBox,
    pub zones: Vec<Zone>,
    pub waypoints: Vec<Waypoint>,
    pub events: Vec<ActiveEvent>,
}

pub struct EntityPosition {
    pub entity_id: EntityId,
    pub entity_type: EntityType,
    pub position: Position,
    pub map_id: MapId,
    pub last_update: DateTime<Utc>,
}

#[derive(Debug, Clone)]
pub enum EntityType {
    Player { character_id: CharacterId, character_name: String },
    Creature { creature_id: CreatureId, creature_type: String },
    NPC { npc_id: NpcId, npc_name: String },
}
```

## API Endpoints

- `GET /map/{map_id}` - Get map data
- `GET /map/{map_id}/entities` - Get all entities on map
- `GET /events/map/{map_id}` - Real-time entity updates (SSE)

## Real-Time Updates

Entity positions are updated in real-time via Server-Sent Events. See [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) for details.

## Summary

The interactive map provides administrators with real-time visibility into the game world, allowing them to monitor player activity, creature spawns, events, and all game entities visually.

