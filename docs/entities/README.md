# Entities Systems Documentation

## Overview

This directory contains comprehensive documentation for all entity-related systems in the Tales of Shadowland MMORPG. Entities are the core interactive objects in the game world, including creatures, players, NPCs, projectiles, mounts, pets, summons, and environmental elements.

## Quick Start

**New to entities systems?** Start here:
1. [OVERVIEW.md](./OVERVIEW.md) - Entity system overview and hierarchy
2. [BASE_CREATURES.md](./BASE_CREATURES.md) - Base creature system
3. [CREATURES.md](./CREATURES.md) - Creature types and behaviors

## Documentation Index

### Core Entity Systems

- **[OVERVIEW.md](./OVERVIEW.md)** - Entity system overview
  - Entity hierarchy and relationships
  - Entity types and categories
  - Common properties
  - Entity lifecycle
  - System integration

- **[ENTITY_REGISTRATION.md](./ENTITY_REGISTRATION.md)** - Entity registration system
  - Dynamic entity registration
  - Entity ID management
  - Entity spawning
  - Entity persistence
  - Registration workflow

- **[ENTITY_UPDATE.md](./ENTITY_UPDATE.md)** - Entity update system
  - Update cycle and tick system
  - Entity state updates
  - Position synchronization
  - Network updates
  - Performance optimization

- **[ENTITY_EVENTS.md](./ENTITY_EVENTS.md)** - Entity event system
  - Event types and handling
  - Event propagation
  - Event subscriptions
  - Event-driven architecture

- **[AREA_OF_INTEREST.md](./AREA_OF_INTEREST.md)** - Area of Interest (AOI) system
  - AOI calculation and management
  - Visibility tracking
  - Entity discovery and removal
  - Network optimization
  - Performance considerations

### Creature Systems

- **[BASE_CREATURES.md](./BASE_CREATURES.md)** - Base creature system
  - Creature base class
  - Common creature properties
  - Creature lifecycle
  - Creature states
  - Basic creature mechanics

- **[CREATURES.md](./CREATURES.md)** - Creature types and behaviors
  - Monster types
  - Animal types
  - Creature AI and behaviors
  - Creature spawning
  - Creature respawn system

- **[BOSS.md](./BOSS.md)** - Boss creature system
  - Boss mechanics and phases
  - Boss abilities and skills
  - Boss respawn timers
  - Boss loot system
  - Boss encounter design

- **[BEHAVIORS.md](./BEHAVIORS.md)** - Advanced creature behaviors
  - Strategic behavior (intelligent humanoids)
  - Aggressive behavior (predators, berserkers)
  - Stealthy behavior (assassins, rogues)
  - Powerful behavior (bosses, elites)
  - Fearful behavior (weak animals)
  - Behavior implementation patterns

- **[HUMANOIDS.md](./HUMANOIDS.md)** - Humanoid entity system
  - Humanoid base class
  - Equipment system integration
  - Humanoid-specific mechanics
  - Player and NPC humanoids

### Special Entity Types

- **[NPCS.md](./NPCS.md)** - Non-Player Character system
  - NPC types and roles
  - NPC behaviors and AI
  - NPC interactions
  - Quest NPCs
  - Vendor NPCs
  - Guard NPCs

- **[MOUNTS_PETS.md](./MOUNTS_PETS.md)** - Mounts and pets system
  - Mount system and mechanics
  - Pet system and mechanics
  - Mount/pet ownership
  - Mount/pet AI
  - Mount/pet abilities
  - Storage and management

- **[SUMMONS.md](./SUMMONS.md)** - Summon system
  - Summon spell mechanics
  - Temporary summon entities
  - Controllable and non-controllable summons
  - Summon AI and behaviors
  - Summon duration and expiration

- **[PROJECTILES.md](./PROJECTILES.md)** - Projectile system
  - Projectile entity types
  - Projectile physics and movement
  - Projectile collision detection
  - Projectile damage application
  - Projectile visual effects

- **[TELEPORTS.md](./TELEPORTS.md)** - Teleport entity system
  - Teleport point entities
  - Portal mechanics
  - Teleportation validation
  - Dynamic portals
  - Teleport interactions

### Action and Spell Systems

- **[ACTIONS.md](./ACTIONS.md)** - Entity action system
  - Action types and categories
  - Action execution
  - Action cooldowns
  - Action effects
  - Action integration with entities

- **[SPELLS.md](./SPELLS.md)** - Spell system
  - Spell types (projectiles, target area, camera direction, single target, target self, area centered)
  - Spell casting mechanics
  - Spell effects and conditions
  - Spell cooldowns and costs
  - Spell integration with creatures

## Entity Hierarchy

```
Entity (Base)
├── Creature (Abstract)
│   ├── Humanoid
│   │   ├── Player
│   │   └── NPC
│   ├── Mount
│   ├── Pet
│   ├── Summon
│   └── BaseCreature
│       ├── Monster
│       ├── Animal
│       └── Boss
├── ProjectileEntity
├── TeleportEntity
└── Other Entities
```

## Key Concepts

### Entity Lifecycle

1. **Creation**: Entity instantiated and initialized
2. **Registration**: Entity registered in the world
3. **Spawn**: Entity added to map and made visible
4. **Update**: Entity updated each game tick
5. **Interaction**: Entity interacts with other entities
6. **Death**: Entity dies (for living entities)
7. **Removal**: Entity removed from world
8. **Destruction**: Entity cleaned up and resources released

### Entity Properties

**Common Properties:**
- Position (x, y, z coordinates)
- Rotation (facing direction)
- Team/group affiliation
- State flags
- Life/Mana/Stamina (for living entities)
- Collision shape and detection
- Visibility and AOI management

**Living Entity Properties:**
- Health and resource pools
- Stats and attributes
- Equipment (for humanoids)
- AI and behaviors
- Combat state

**Non-Living Entity Properties:**
- Lifetime/duration
- Visual effects
- Collision detection
- Interaction triggers

### Entity Categories

**Living Entities:**
- Players (user-controlled characters)
- Creatures (monsters, animals, bosses)
- Pets (companion creatures)
- Mounts (rideable creatures)
- Summons (temporary entities)
- NPCs (non-player characters)

**Non-Living Entities:**
- Projectiles (arrows, spells, ranged attacks)
- Teleports (portals and waypoints)
- Items (ground items and containers)
- Environmental (static world objects)

## System Integration

### With Map System
- Entities exist within maps
- Map manages entity lists
- Spatial partitioning for entities
- Map-specific entity rules

### With Network System
- Entity state synchronization
- AOI-based network updates
- Entity creation/destruction packets
- Position and state updates

### With Combat System
- Entity combat interactions
- Damage application
- Status effects
- Combat state management

### With AI System
- Creature AI behaviors
- Pathfinding and movement
- Decision making
- Behavior state machines

## Related Documentation

- [Core Systems](../core/) - Core game systems
- [Player Systems](../player/) - Player-specific systems
- [Server Systems](../server/) - Server architecture
- [Items Systems](../items/) - Item and equipment systems

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Core README](../core/README.md) - Core systems documentation

