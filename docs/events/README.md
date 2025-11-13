# Events Systems Documentation

## Overview

This directory contains comprehensive documentation for all event systems in the Tales of Shadowland MMORPG. Events are special instanced gameplay experiences that create isolated map instances for players to participate in various competitive and cooperative activities.

## Quick Start

**New to events systems?** Start here:
1. [EVENT_INSTANCE.md](./EVENT_INSTANCE.md) - Event instance system and architecture
2. [BATTLECASTLE.md](./BATTLECASTLE.md) - Battle Castle PvP event example
3. [TOWER_DEFENDER.md](./TOWER_DEFENDER.md) - Tower Defender PvE event example

## Documentation Index

### Core Event Systems

- **[EVENT_INSTANCE.md](./EVENT_INSTANCE.md)** - Event instance system
  - Event instance base class
  - Event types and registration
  - Map instance management
  - Player participation
  - Event lifecycle
  - Queue system (Solo, Party, Queue)

### PvP Events

- **[BATTLECASTLE.md](./BATTLECASTLE.md)** - Battle Castle event
  - Large-scale PvP battle (40 players)
  - Two-team system (Red vs Blue)
  - Point-based victory (150 points or highest score)
  - 15-minute timer
  - Kill/death ranking system
  - Respawn mechanics
  - Reward distribution

- **[CAPTURE_THE_FLAG.md](./CAPTURE_THE_FLAG.md)** - Capture the Flag event
  - Team-based flag capture PvP
  - Two teams competing for flag control
  - Three flags (2 team flags + 1 neutral flag)
  - 30-second capture mechanic (no damage)
  - Point system (10 points per flag per minute)
  - 15-minute duration
  - Defense and capture mechanics

### PvE Events

- **[TOWER_DEFENDER.md](./TOWER_DEFENDER.md)** - Tower Defender event
  - Cooperative PvE defense (up to 4 players)
  - Defend central crystal from waves
  - Wave-based progression system
  - Mini-bosses every 5 waves
  - Bosses every 20 waves
  - Enemies attack only crystal
  - Progressive individual rewards

- **[CHAMPIONS.md](./CHAMPIONS.md)** - Champions system
  - Progressive level-based encounters
  - Dynamic creature spawning
  - 8 levels of increasing difficulty
  - Boss encounter after all levels
  - Persistent world content
  - Auto-reset system

- **[TREASURE_HUNT.md](./TREASURE_HUNT.md)** - Treasure Hunt event
  - Treasure hunting gameplay
  - Exploration and discovery
  - Reward collection
  - Event-specific mechanics

## Event Types Overview

### PvP Events

**Battle Castle:**
- 40-player large-scale PvP
- Team-based combat
- Kill-based scoring
- 15-minute duration

**Capture the Flag:**
- Team-based flag capture
- Strategic flag control
- Defense mechanics
- Point-based scoring

### PvE Events

**Tower Defender:**
- Cooperative wave defense
- Crystal protection
- Progressive difficulty
- Individual rewards

**Champions:**
- Progressive encounters
- Level-based difficulty
- Boss encounters
- World content

**Treasure Hunt:**
- Exploration gameplay
- Treasure discovery
- Collection mechanics

## Key Concepts

### Event Instances

**Isolated Maps:**
- Each event creates isolated map instance
- Separate thread for event map
- Isolated entity lists
- No interaction with main world

**Instance Lifecycle:**
1. Event creation
2. Map instance creation
3. Player participation
4. Event execution
5. Event completion
6. Instance cleanup

### Player Participation

**Entry Methods:**
- Admin-triggered events
- Portal entry
- Queue system
- Party-based entry

**Team Assignment:**
- Random assignment (Battle Castle)
- Balanced teams
- Party members kept together
- Team-based spawn points

### Reward Systems

**Team Rewards:**
- Winning team bonuses
- Participation rewards
- Team performance bonuses

**Individual Rewards:**
- Performance-based bonuses
- Participation rewards
- Progressive accumulation
- Milestone rewards

## Event Architecture

### Event Base Class

**Common Properties:**
- Event type
- Map namespace
- Map instance
- Player list
- Event state

**Common Methods:**
- Join/Leave
- Start/End
- Update/Tick
- Cleanup

### Map Instance Integration

**Instance Creation:**
- Unique map ID (GUID)
- Isolated from other maps
- Event-specific namespace
- Separate thread execution

**Instance Management:**
- Lifecycle management
- Resource cleanup
- Player tracking
- State synchronization

## Integration Points

### With Map Instance System
- Uses MAPS_INSTANCE system
- Creates isolated map instances
- Separate threads per instance
- Isolated entity management

### With Team System
- Team assignment and management
- Team-based spawn points
- Team-based rewards
- Team coordination mechanics

### With Combat System
- PvP combat (Battle Castle, CTF)
- PvE combat (Tower Defender, Champions)
- Damage and death mechanics
- Respawn systems

### With Reward System
- Team-based reward distribution
- Individual performance bonuses
- Progressive reward accumulation
- Event-specific rewards

## Related Documentation

- [Server Systems](../server/) - Server architecture
- [Map Instances](../server/MAPS_INSTANCE.md) - Map instance system
- [Team System](../core/TEAMS.md) - Team system
- [Player Systems](../player/) - Player systems

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Core README](../core/README.md) - Core systems documentation

