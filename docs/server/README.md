# Server Documentation

## Overview

This directory contains comprehensive documentation for the Tales of Shadowland MMORPG server implementation. The server is built in **Rust (Edition 2024, nightly 1.85+)** and designed to support 10,000 concurrent players with a 60 FPS tick rate, processing 600,000 UDP packets per second.

## Quick Start

**New to the project?** Start here:
1. [ARCHITECTURE.md](./ARCHITECTURE.md) - Understand the overall architecture
2. [STATUS.md](./STATUS.md) - Check current implementation status
3. [ROADMAP.md](./ROADMAP.md) - See implementation phases
4. [DAG.md](./DAG.md) - Understand module dependencies

## Documentation Index

### Core Architecture

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Complete server architecture documentation
  - Network layer (UDP protocol, batch processing, fragmentation)
  - Serialization system (FlatBuffer, quantization, compression)
  - Security (encryption, replay protection, integrity)
  - Entity system (ECS framework, state management)
  - Area of Interest (AOI) system
  - Reliable UDP system
  - Authority system
  - Performance optimizations

- **[DAG.md](./DAG.md)** - Dependency Graph (DAG) of server modules
  - Crate dependencies
  - Module relationships
  - Build order
  - Circular dependency prevention
  - System execution flows

- **[STATUS.md](./STATUS.md)** - Implementation status tracking
  - Current implementation status
  - Completed systems
  - Planned features
  - Performance metrics
  - Known limitations

- **[ROADMAP.md](./ROADMAP.md)** - Implementation roadmap
  - 9 implementation phases
  - Phase dependencies
  - Success criteria
  - Risk mitigation
  - Timeline estimates

### Network & Security

- **[SECURITY.md](./SECURITY.md)** - Security architecture
  - Encryption (ChaCha20-Poly1305, X25519)
  - Replay protection
  - Integrity checks
  - Anti-cheat measures
  - Authentication system

- **[LAG_COMPENSATION.md](./LAG_COMPENSATION.md)** - Lag compensation system
  - Client-side prediction
  - Server-side reconciliation
  - Interpolation strategies
  - Latency handling

### World & Environment

- **[MAPS.md](./MAPS.md)** - Map system documentation
  - Map loading and management
  - Spatial partitioning
  - Zone management
  - Map data structures

- **[MAPS_INSTANCE.md](./MAPS_INSTANCE.md)** - Map instances system
  - Isolated map instances for events and dungeons
  - Thread separation per instance
  - On-demand creation and cleanup
  - Group isolation and performance optimization

- **[ZONES.md](./ZONES.md)** - Zone system
  - Zone definitions
  - Zone transitions
  - Zone-specific rules
  - Zone loading

- **[SACRED_LANDS.md](./SACRED_LANDS.md)** - Sacred Lands system
  - Special zone mechanics
  - Sacred Lands features
  - Rules and restrictions

- **[TELEPORTATION.md](./TELEPORTATION.md)** - Teleportation system
  - Teleport types
  - Teleport validation
  - Teleport mechanics
  - Teleport restrictions

- **[RESPAWN.md](./RESPAWN.md)** - Respawn system
  - Respawn mechanics
  - Respawn points
  - Respawn timers
  - Respawn rules

### Economy Systems

- **[ECONOMY.md](./ECONOMY.md)** - Economy systems overview
  - System overview and integration
  - Links to detailed economy documentation
  - Weekly maintenance tasks
  - Integration points

- **[GOLD_BURN.md](./GOLD_BURN.md)** - Gold burn system
  - Gold burn sources
  - Weekly aggregation
  - Lottery prize calculation
  - Gold burn logging

- **[AUCTION_HOUSE.md](./AUCTION_HOUSE.md)** - Auction house system
  - Listing types (buyout, auction)
  - Bidding system
  - Search and filtering
  - Anti-duplication protection
  - Fees and commissions

- **[MAGIC_STONE_MARKET.md](./MAGIC_STONE_MARKET.md)** - Magic stone market
  - Buy/sell orders
  - Real-time price quotes
  - Order matching system
  - Market depth visualization

- **[BULK_ORDERS.md](./BULK_ORDERS.md)** - Bulk orders system
  - Player-created orders
  - NPC-created orders
  - Escrow system
  - Deadlines and penalties

- **[LOTTERY.md](./LOTTERY.md)** - Lottery system
  - Weekly lottery cycle
  - Ticket purchases
  - Prize pool calculation
  - Winner drawing

- **[ECONOMIC_REBALANCING.md](./ECONOMIC_REBALANCING.md)** - Economic rebalancing
  - NPC price adjustments
  - Resource abundance management
  - Gold drop rate adjustments
  - Rare item scarcity control
  - Market liquidity management

### Monitoring & Operations

- **[TELEMETRY.md](./TELEMETRY.md)** - Telemetry and monitoring system
  - Hardware metrics (CPU, memory)
  - Network metrics (latency, packets/sec, ping)
  - Performance metrics (FPS, tick rate, update cycles)
  - Game metrics
  - Prometheus export
  - API integration

- **[PERSISTENCE.md](./PERSISTENCE.md)** - Data persistence
  - Database architecture
  - SQLite (game server)
  - PostgreSQL (API principal)
  - Backup system
  - Data synchronization

- **[ACCOUNTS.md](./ACCOUNTS.md)** - Account system
  - Account management
  - Authentication
  - Character selection
  - Account security

### Commands & Administration

- **[COMMANDS.md](./COMMANDS.md)** - Player commands
  - Command system
  - Available commands
  - Command permissions
  - Command implementation

- **[ADMIN_COMMANDS.md](./ADMIN_COMMANDS.md)** - Admin commands
  - Admin command system
  - Server management commands
  - Player management commands
  - Content management commands
  - Monitoring commands

### Visual & Presentation

- **[VISUAL.md](./VISUAL.md)** - Visual systems
  - Visual effects
  - Rendering systems
  - Client synchronization

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Game Server (UDP)                         │
├─────────────────────────────────────────────────────────────┤
│  Network Layer → Protocol → ECS → Systems → World           │
│       ↓            ↓        ↓       ↓         ↓            │
│   Security    Serialization  │   Combat   AOI/Spatial      │
│   Encryption  Compression     │   AI      Telemetry        │
│   Reliable    Quantization    │   Economy  Persistence      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              API Principal (HTTP/REST)                      │
├─────────────────────────────────────────────────────────────┤
│  HTTP Server → Routes → Handlers → Services → Database      │
│       ↓           ↓        ↓         ↓          ↓          │
│   Auth      Admin API   Business   PostgreSQL  Backup       │
│   CORS      Config API  Logic      Models      Cloud        │
│   SSE       Monitoring  Services   Migrations  Storage      │
└─────────────────────────────────────────────────────────────┘
```

## Key Design Decisions

### Performance
- **Zero-allocation design** where possible
- **Lock-free data structures** using channels
- **Batch processing** for network operations
- **Structure of Arrays (SoA)** for cache efficiency
- **Custom ECS framework** for entity management

### Security
- **End-to-end encryption** (ChaCha20-Poly1305)
- **Perfect forward secrecy** (X25519 key exchange)
- **Replay protection** (64-position sliding window)
- **Server authority** for all game state

### Scalability
- **Area of Interest (AOI)** system for efficient updates
- **Delta compression** (60-75% bandwidth reduction)
- **Quantization** (60% bandwidth reduction)
- **Adaptive sync** based on movement

### Database Architecture
- **SQLite** for game server (embedded, fast)
- **PostgreSQL** for API principal (centralized, robust)
- **Dual database** architecture for separation of concerns

## Performance Targets

| Metric | Target |
|--------|--------|
| Tick Rate | 60 FPS (16.67ms per frame) |
| Concurrent Players | 10,000 simultaneous |
| Packet Throughput | 600,000 UDP packets/second |
| Latency (p99) | < 50ms |
| Bandwidth Reduction | 60-75% (compression + quantization) |

## Implementation Status

**Current Phase:** See [ROADMAP.md](./ROADMAP.md) for current phase

**Completed Systems:**
- Architecture design and documentation
- System specifications
- Dependency graph (DAG)
- Economy system design
- Telemetry system design

**In Progress:**
- Rust implementation (see [ROADMAP.md](./ROADMAP.md))

**Planned:**
- All systems as outlined in [ROADMAP.md](./ROADMAP.md)

## Related Documentation

### Client Documentation
- [Client Architecture](../client/ARCHITECTURE.md) - Unreal Engine client architecture
- [Entity Synchronization](../client/ENTITY_SYNC.md) - Client-server sync
- [Spells System](../client/SPELLS.md) - Client-side spells (GAS)

### Core Systems
- [Core Systems](../core/) - Core game systems documentation
- [Entities](../entities/) - Entity system documentation
- [Player Systems](../player/) - Player-related systems

### Admin Documentation
- [Admin Overview](../admin/OVERVIEW.md) - Administrative system overview
- [Admin Dashboard](../admin/DASHBOARD.md) - Web dashboard documentation

## Getting Started

### For Developers

1. **Read the Architecture**
   - Start with [ARCHITECTURE.md](./ARCHITECTURE.md)
   - Review [DAG.md](./DAG.md) for module structure
   - Check [STATUS.md](./STATUS.md) for current state

2. **Understand the Roadmap**
   - Review [ROADMAP.md](./ROADMAP.md) for implementation phases
   - Check phase dependencies
   - Understand success criteria

3. **Study System Documentation**
   - Read relevant system documentation
   - Understand integration points
   - Review code examples

### For System Administrators

1. **Deployment**
   - Review [ARCHITECTURE.md](./ARCHITECTURE.md) for system requirements
   - Check [PERSISTENCE.md](./PERSISTENCE.md) for database setup
   - Review [TELEMETRY.md](./TELEMETRY.md) for monitoring

2. **Operations**
   - Review [ADMIN_COMMANDS.md](./ADMIN_COMMANDS.md) for management
   - Check [SECURITY.md](./SECURITY.md) for security configuration
   - Review [TELEMETRY.md](./TELEMETRY.md) for monitoring setup

### For Game Designers

1. **Economy Systems**
   - Start with [ECONOMY.md](./ECONOMY.md) for overview
   - Review individual economy system docs
   - Understand economic rebalancing

2. **World Systems**
   - Review [MAPS.md](./MAPS.md) and [ZONES.md](./ZONES.md)
   - Check [RESPAWN.md](./RESPAWN.md) for respawn mechanics
   - Review [TELEPORTATION.md](./TELEPORTATION.md) for teleportation

## Contributing

When contributing to the server:

1. **Follow the Roadmap**
   - Check [ROADMAP.md](./ROADMAP.md) for current phase
   - Understand phase dependencies
   - Follow implementation guidelines

2. **Maintain Documentation**
   - Update relevant documentation files
   - Keep [STATUS.md](./STATUS.md) updated
   - Document design decisions

3. **Follow Architecture**
   - Respect module dependencies (see [DAG.md](./DAG.md))
   - Follow performance guidelines
   - Maintain security standards

4. **Quality Standards**
   - 95%+ test coverage required
   - All tests must pass
   - No clippy warnings
   - Code must be formatted

## See Also

- [Main README](../../README.md) - Project overview
- [Structure Recommendations](../STRUCTURE_RECOMMENDATIONS.md) - Project structure
- [OpenSpec Project](../openspec/project.md) - Project specifications

