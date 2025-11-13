# Entities System Overview

## Introduction

The entities system is the core of the game world, representing all interactive objects, creatures, players, NPCs, projectiles, and environmental elements. This document provides an overview of the entity hierarchy and their relationships.

## Entity Hierarchy

```
Entity (Base)
├── Creature (Abstract)
│   ├── Humanoid
│   │   └── Player
│   ├── Mount
│   ├── Pet
│   └── Summon
├── ProjectileEntity
├── TeleportEntity
└── Other Entities
```

## Entity Types

### Core Entities

1. **Entity** - Base class for all game entities
2. **Creature** - Abstract base for all living entities
3. **Humanoid** - Entities with equipment slots and humanoid features
4. **Player** - Player-controlled characters
5. **Mount** - Mountable creatures
6. **Pet** - Companion creatures
7. **Summon** - Temporary summoned entities
8. **ProjectileEntity** - Projectiles and ranged attacks
9. **TeleportEntity** - Portal and teleportation points
10. **NonPlayerCharacter** - NPCs and vendors

## Entity Categories

### Living Entities

- **Players**: User-controlled characters with full game mechanics
- **Creatures**: Monsters, animals, and hostile entities
- **Pets**: Companion creatures that follow and assist players
- **Mounts**: Rideable creatures for faster movement
- **Summons**: Temporary entities created by skills/actions
- **NPCs**: Non-player characters for quests, vendors, and interactions

### Non-Living Entities

- **Projectiles**: Arrows, spells, and ranged attacks
- **Teleports**: Portals and waypoints
- **Items**: Ground items and containers
- **Environmental**: Static world objects

## Common Properties

All entities share common properties:

- **Position**: World coordinates (x, y, z)
- **Rotation**: Facing direction
- **Team**: Team/group affiliation
- **States**: Entity state flags
- **Life/Mana/Stamina**: Resource pools (for living entities)
- **Collision**: Collision shape and detection
- **Visibility**: Area of interest and visibility management

## Entity Lifecycle

1. **Creation**: Entity instantiated and initialized
2. **Spawn**: Entity added to world/map
3. **Update**: Entity updated each tick
4. **Interaction**: Entity interacts with other entities
5. **Death**: Entity dies (for living entities)
6. **Removal**: Entity removed from world
7. **Destruction**: Entity cleaned up

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Creature system details
- [HUMANOIDS.md](./HUMANOIDS.md) - Humanoid and equipment system
- [PLAYER.md](./PLAYER.md) - Player entity details
- [MOUNTS_PETS.md](./MOUNTS_PETS.md) - Mounts and pets
- [SUMMONS.md](./SUMMONS.md) - Summoned entities
- [PROJECTILES.md](./PROJECTILES.md) - Projectile system
- [TELEPORTS.md](./TELEPORTS.md) - Teleportation entities
- [NPCS.md](./NPCS.md) - Non-player characters

