# Content Management

## Overview

Content management allows administrators to create and configure all game content including items, NPCs, quests, zones, events, respawns, and creatures. **All content created or modified is automatically replicated to all game servers**, ensuring identical world structure across all servers.

## Replicated Content

**Important:** The following content is replicated to all game servers:

- **Maps**: Map definitions, boundaries, and properties
- **Items**: All item definitions and properties
- **NPCs**: All NPCs, dialogues, and behaviors
- **Quests**: All quest definitions, requirements, and rewards
- **Zones**: All zone configurations, boundaries, and properties
- **Events**: All automatic event configurations
- **Patrols**: All creature patrol routes
- **Creatures**: All creature definitions and properties
- **Respawns**: All respawn points (both creature and resource respawns) - each associated with a map

When any of these content types are created, modified, or removed, the changes are automatically replicated to all active game servers.

**Map Management:** Each game server manages multiple maps. Maps must be registered in the admin dashboard and assigned to servers. While map definitions are replicated, each server maintains its own map instances with server-specific data (players, creatures, events, etc.).

## Map Management

**Features:**
- View all maps
- Create maps
- Remove maps
- Configure map properties
- Set map boundaries
- Configure map zones
- Assign maps to servers

**Map Properties:**
- Map ID and name
- Map boundaries (bounding box)
- Zones within the map
- Teleportation points
- Sacred Lands locations
- Map size and dimensions

**Important:** Each game server manages multiple maps. Maps must be registered in the admin dashboard and assigned to servers.

**API Endpoints:**
- `GET /content/maps` - List all maps
- `GET /content/maps/{map_id}` - Get map details
- `POST /content/maps` - Create map
- `PUT /content/maps/{map_id}` - Update map
- `DELETE /content/maps/{map_id}` - Remove map
- `POST /content/maps/{map_id}/assign` - Assign map to server(s)

**Replication:** Map definitions are replicated to all game servers, but each server manages its own instances of the maps.

## Respawn Management

**Features:**
- View all respawns (creature and resource respawns)
- Create respawns
- Remove respawns
- Configure respawn rates
- Set respawn locations
- Configure respawn types (creature or resource)
- Assign respawns to maps

**Respawn Types:**
- **Creature Respawns**: Spawn points for creatures (monsters, animals, etc.)
- **Resource Respawns**: Spawn points for resources (ores, herbs, trees, etc.)

**Map Association:**
- **Every respawn is associated with a map**: Each respawn must be assigned to a specific map
- Respawns are created within map boundaries
- Respawn locations are relative to the map coordinate system

**API Endpoints:**
- `GET /content/respawns` - List all respawns
- `GET /content/respawns?map_id={map_id}` - List respawns for a specific map
- `POST /content/respawns` - Create respawn (replicates to all servers)
- `DELETE /content/respawns/{respawn_id}` - Remove respawn (replicates to all servers)
- `PUT /content/respawns/{respawn_id}` - Update respawn (replicates to all servers)

**Request Example:**
```json
{
  "respawn_type": "creature",
  "map_id": "map_001",
  "location": {"x": 1000, "y": 2000, "z": 500},
  "creature_type": "Goblin",
  "spawn_rate": 0.5,
  "max_spawns": 10
}
```

**Replication:** All respawn changes are automatically replicated to all game servers, ensuring consistent world structure. Each server manages respawn instances on its assigned maps.

## Creature Management

**Features:**
- View all creatures
- Create creatures
- Configure creature properties
- Set creature spawn locations
- Configure creature AI

**API Endpoints:**
- `GET /content/creatures` - List all creatures
- `POST /content/creatures` - Create creature (replicates to all servers)
- `PUT /content/creatures/{creature_id}` - Update creature (replicates to all servers)
- `DELETE /content/creatures/{creature_id}` - Remove creature (replicates to all servers)

**Replication:** All creature changes are automatically replicated to all game servers.

## NPC Management

**Features:**
- View all NPCs
- Create NPCs
- Configure NPC dialogue
- Set NPC locations
- Configure NPC behavior

**API Endpoints:**
- `GET /content/npcs` - List all NPCs
- `POST /content/npcs` - Create NPC (replicates to all servers)
- `PUT /content/npcs/{npc_id}` - Update NPC (replicates to all servers)
- `DELETE /content/npcs/{npc_id}` - Remove NPC (replicates to all servers)

**Replication:** All NPC changes are automatically replicated to all game servers.

## Item Management

**Features:**
- Create items from dashboard
- Modify item properties
- Correct problematic items
- Replicate changes to all servers

See [CONFIG_API.md](./CONFIG_API.md) for item creation and replication details.

## Configuration Management

**Features:**
- Update quest configuration
- Update zone configuration
- Update event configuration
- Update patrol configuration

All configuration changes are replicated to all game servers automatically.

## Server-Specific vs Replicated Content

### Replicated Content (Same Across All Servers)

All game servers share identical world structure:

- **Maps**: Map definitions, boundaries, and properties
- **Items**: Item definitions and properties
- **NPCs**: NPCs, dialogues, behaviors
- **Quests**: Quest definitions, requirements, rewards
- **Zones**: Zone configurations, boundaries, properties
- **Events**: Automatic event configurations
- **Patrols**: Creature patrol routes
- **Creatures**: Creature definitions and properties
- **Respawns**: All respawn points (creature and resource) - each associated with a map

**Map Management:** Each game server manages multiple maps. Maps are registered in the admin dashboard and assigned to servers. While map definitions are replicated, each server maintains its own map instances.

### Server-Specific Content (Unique Per Server)

Each game server maintains its own:

- **Player Items**: Items owned by players (inventory, equipment, bank)
- **Players**: Player characters and their data
- **Rankings**: Server-specific player rankings
- **Auction House**: Server-specific auction house listings
- **Houses**: Player-owned houses and properties
- **Guilds**: Server-specific guilds and their data
- **Parties**: Active parties on the server
- **Player Progress**: Quest progress, skill levels, etc.

## Summary

Content management provides comprehensive tools for creating and configuring all game content. **All content changes are automatically replicated to all game servers**, ensuring consistent world structure across all servers while maintaining independent player economies and communities per server.

