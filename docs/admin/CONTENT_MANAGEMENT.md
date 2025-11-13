# Content Management

## Overview

Content management allows administrators to create and configure all game content including items, NPCs, quests, zones, events, respawns, and creatures.

## Respawn Management

**Features:**
- View all respawns
- Create respawns
- Remove respawns
- Configure respawn rates
- Set respawn locations

**API Endpoints:**
- `GET /content/respawns` - List all respawns
- `POST /content/respawns` - Create respawn
- `DELETE /content/respawns/{respawn_id}` - Remove respawn
- `PUT /content/respawns/{respawn_id}` - Update respawn

## Creature Management

**Features:**
- View all creatures
- Create creatures
- Configure creature properties
- Set creature spawn locations
- Configure creature AI

**API Endpoints:**
- `GET /content/creatures` - List all creatures
- `POST /content/creatures` - Create creature
- `PUT /content/creatures/{creature_id}` - Update creature
- `DELETE /content/creatures/{creature_id}` - Remove creature

## NPC Management

**Features:**
- View all NPCs
- Create NPCs
- Configure NPC dialogue
- Set NPC locations
- Configure NPC behavior

**API Endpoints:**
- `GET /content/npcs` - List all NPCs
- `POST /content/npcs` - Create NPC
- `PUT /content/npcs/{npc_id}` - Update NPC
- `DELETE /content/npcs/{npc_id}` - Remove NPC

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

## Summary

Content management provides comprehensive tools for creating and configuring all game content, ensuring consistent content across all servers through automatic replication.

