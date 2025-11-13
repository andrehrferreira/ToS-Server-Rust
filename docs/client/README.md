# Client Systems Documentation

## Overview

This directory contains comprehensive documentation for all client-side systems in the Tales of Shadowland MMORPG. The client is built using **Unreal Engine** with a C++ plugin architecture, providing Blueprint accessibility and a clean separation between engine code and game-specific functionality. All client systems focus on visual presentation, user input, entity synchronization, and seamless integration with the server.

## Quick Start

**New to client systems?** Start here:
1. [ARCHITECTURE.md](./ARCHITECTURE.md) - Complete client architecture and project structure
2. [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) - Player controller and movement system
3. [ENTITY_SYNC.md](./ENTITY_SYNC.md) - Entity synchronization with server

## Documentation Index

### Core Architecture

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Client architecture
  - Plugin structure (C++ plugin with Blueprint access)
  - Content organization (`_Game` folder structure)
  - Custom GameInstance (server config, connection, authentication)
  - Custom PlayerController (entity spawning, possession control)
  - Entity management systems
  - Network subsystem (UDP client, secure session)
  - GAS (Gameplay Ability System) integration
  - DataTables and configuration

### Player Systems

- **[PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md)** - Player controller system
  - Motion Matching for character movement
  - Thread-safe movement system
  - Target lock system (enemy/ally focus)
  - Swimming mechanics
  - Glider flight system
  - Server synchronization
  - Lag compensation
  - Pathfinding integration

- **[INPUT.md](./INPUT.md)** - Input system
  - Movement controls (WASD with separate axes)
  - Actionbar controls (1-0 keys)
  - Combat controls (targeting, auto-attack)
  - Interaction controls (E/F keys)
  - System controls (settings, menus)
  - Key binding configuration

- **[ACTIONBAR.md](./ACTIONBAR.md)** - ActionBar system
  - Configurable slots (10 or 20 slots)
  - Hotkey binding system
  - Slot types (spells, consumables, mounts)
  - Cooldown visualization
  - Enable/disable states
  - Inventory integration
  - Quantity display for consumables
  - Server synchronization

### Entity and Synchronization

- **[ENTITY_SYNC.md](./ENTITY_SYNC.md)** - Entity synchronization system
  - Server-authoritative entity updates
  - Position synchronization
  - State synchronization
  - Network update optimization
  - Delta compression
  - Interpolation and prediction

### Visual Systems

- **[CAMERA.md](./CAMERA.md)** - Camera system
  - Third-person camera controls
  - Camera rotation and zoom
  - Target lock camera behavior
  - Smooth camera transitions
  - Camera collision detection

- **[VISUAL.md](./VISUAL.md)** - Visual effects system
  - Particle effects
  - Visual feedback systems
  - Rendering optimizations
  - Effect synchronization

### UI Systems

- **[UI_WIDGETS.md](./UI_WIDGETS.md)** - UI Widgets system
  - Reactive UI architecture (React/Vue-like)
  - UMG plugin integration
  - Delegate-based automatic updates
  - C++/Blueprint hybrid approach
  - Real-time synchronization
  - No manual bindings required
  - Subsystem integration

### Spell Systems

- **[SPELLS.md](./SPELLS.md)** - Client-side spells system
  - Gameplay Ability System (GAS) integration
  - Visual components (particles, animations, sounds)
  - Cast points system (invisible spheres for spell origin)
  - Animation notifies (precise spell timing)
  - Generic spell execution function
  - Server-triggered spell execution
  - Spell types (projectiles, target area, camera direction, single target, target self, area centered)
  - Summon spells implementation

### Subsystems

- **[SUBSYSTEM.md](./SUBSYSTEM.md)** - GameInstance subsystems
  - Subsystem architecture
  - Global system access
  - Reactive system updates
  - System lifecycle management

## Key Technologies

### Unreal Engine

**Engine Version:**
- Unreal Engine (version to be specified)
- C++ plugin architecture
- Blueprint integration
- Gameplay Ability System (GAS)

**Core Systems:**
- Motion Matching for movement
- Gameplay Ability System for spells
- Animation Blueprints
- Particle systems
- Audio system

### Network Architecture

**UDP Client:**
- Custom UDP protocol
- Secure session management
- Packet queuing and batching
- Reliable/unreliable channels
- Encryption (ChaCha20-Poly1305)

**Synchronization:**
- Server-authoritative updates
- Client-side prediction
- Lag compensation
- Delta compression
- Interpolation

## Project Structure

### Plugin Architecture

**C++ Plugin (`ToSClient`):**
- All C++ code in plugin
- Blueprint accessibility
- Clean separation from engine code
- Modular architecture

**Key Modules:**
- Core (GameInstance, PlayerController, PlayerState)
- Network (UDP client, secure session)
- Entities (SyncEntity, SyncPlayer, SyncCreature)
- Spells (GAS integration, spell execution)
- Gathering (Gatherable resources)

### Content Organization

**`_Game` Folder:**
- DataTables (spells, creatures, items, resources)
- VFX (spell effects, projectiles, impacts)
- SFX (spells, combat, ambient, UI)
- Textures (spells, items, creatures, UI)
- Models (spells, creatures, gatherable resources)
- Systems (creatures, gathering blueprints)

## System Integration

### Client ↔ Server Communication

**Network Protocol:**
- Custom binary UDP protocol
- Packet-based communication
- Reliable and unreliable channels
- Encryption and compression

**Synchronization:**
- Entity state synchronization
- Position updates
- Action execution
- Inventory updates
- UI state updates

### Gameplay Systems

**Movement:**
- Client-side prediction
- Server reconciliation
- Smooth interpolation
- Motion Matching integration

**Combat:**
- Server-authoritative damage
- Client-side visual feedback
- Spell execution via GAS
- ActionBar integration

**Spells:**
- GAS-based spell system
- Visual effects and animations
- Server-triggered execution
- Cast points and animation notifies

## Key Features

### Performance Optimizations

**Entity Management:**
- Efficient entity updates
- Culling and LOD systems
- Batch updates
- Delta compression

**Rendering:**
- Optimized particle systems
- Efficient texture streaming
- LOD management
- Culling optimizations

### User Experience

**Responsive Controls:**
- Low-latency input handling
- Smooth movement
- Immediate visual feedback
- Configurable key bindings

**Visual Polish:**
- High-quality particle effects
- Smooth animations
- Professional audio
- Polished UI

## Development Workflow

### C++ Development

**Plugin Development:**
- C++ code in plugin
- Blueprint exposure via UFUNCTION/UPROPERTY
- Hot reload support
- Modular architecture

**Code Organization:**
- Public headers for Blueprint access
- Private implementation files
- Clear module separation
- Documentation in code

### Blueprint Development

**Blueprint Usage:**
- Extend C++ classes in Blueprint
- Configure data in Blueprint
- Create UI widgets
- Design gameplay sequences

**Content Creation:**
- VFX creation
- Animation setup
- Audio integration
- UI design

## Related Documentation

### Server Systems
- [Server Architecture](../server/ARCHITECTURE.md) - Server architecture
- [Network Packets](../packets/OVERVIEW.md) - Network protocol
- [Entity Systems](../entities/) - Entity systems

### Player Systems
- [Player Systems](../player/) - Player progression and systems
- [Spells System](../player/SPELLS.md) - Player spells system

### Items Systems
- [Items Systems](../items/) - Items and equipment
- [Consumables](../items/CONSUMABLES.md) - Consumable items

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Packets Overview](../packets/OVERVIEW.md) - Network packets

