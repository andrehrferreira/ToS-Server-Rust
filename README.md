# Tales of Shadowland MMORPG Server

## Overview

The Tales of Shadowland MMORPG server is a high-performance game server designed to support 10,000 concurrent players with a 60 FPS tick rate and processing 600,000 UDP packets per second. The server is being implemented in Rust to achieve maximum performance while maintaining security and scalability.

## Documentation

Complete documentation is available in the [`docs/`](./docs/) directory:

### Architecture Documentation

- **[ARCHITECTURE.md](./docs/server/ARCHITECTURE.md)** - Complete architecture documentation
  - Network layer (UDP protocol, batch processing, fragmentation)
  - Serialization system (FlatBuffer, quantization, compression)
  - Security (encryption, replay protection, integrity)
  - Entity system (state management, delta compression)
  - Area of Interest (AOI) system
  - Reliable UDP system
  - Authority system
  - Buffs and debuffs system
  - Evolution and lessons learned

### Implementation Status

- **[STATUS.md](./docs/server/STATUS.md)** - Current implementation status
  - C# version status (completed systems, limitations)
  - Rust version status (planned systems, targets)
  - Performance metrics
  - Known limitations
  - Historical versions

### Implementation Roadmap

- **[ROADMAP.md](./docs/server/ROADMAP.md)** - Implementation roadmap
  - 9 implementation phases
  - Dependencies between phases
  - Risk mitigation
  - Success metrics

### System Documentation

- **[FLAGS.md](./docs/core/FLAGS.md)** - Flags system documentation
  - Entity states (bitwise flags)
  - Buff/debuff states
  - Item states
  - Flag operations
  - Serialization
  - State synchronization

- **[CONDITIONS.md](./docs/core/CONDITIONS.md)** - Conditions system documentation
  - Condition types (DoT, HoT, status effects)
  - Condition lifecycle (apply, update, refresh, remove)
  - Buff/debuff system
  - Condition management
  - Serialization

- **[TEAMS.md](./docs/core/TEAMS.md)** - Teams system documentation
  - Team types (Monsters, Players, NPCs, etc.)
  - Team relationships (ally/enemy)
  - PvP and PvE logic
  - Team-based gameplay
  - Team relationships matrix

- **[ENTITIES.md](./docs/core/ENTITIES.md)** - Entity system architecture documentation
  - Entity hierarchy (TypeScript and C# implementations)
  - Performance concerns (vtable overhead, cache misses, memory allocation)
  - Rust implementation strategies (ECS pattern, SoA layout, zero-allocation pooling)
  - Component-based architecture
  - System-based processing
  - Performance comparison (inheritance vs ECS)

- **[MOVEMENT.md](./docs/core/MOVEMENT.md)** - Movement and collision system documentation
  - Grid-based collision system (2D and 3D)
  - Movement validation (prevent hacks: fly, wall clipping, speed)
  - Pathfinding system (A* for AI navigation)
  - Patrol route system (monster AI)
  - Swimming system (water areas, state management, animation triggers)
  - Security considerations (anti-cheat, rate limiting)
  - Performance optimizations (spatial hashing, broad/narrow phase)

- **[BYTEBUFFER.md](./docs/core/BYTEBUFFER.md)** - ByteBuffer architecture documentation
  - ByteBuffer evolution (multiple versions analyzed: ByteBuffer.cs, FlatBuffer.cs, UFlatBuffer.cpp, bytebuffer.ts)
  - VarInt/ZigZag encoding (variable-length integer encoding for bandwidth optimization)
  - Quantization system (position, rotation, float quantization with delta compression)
  - Symbol pooling (string symbol caching per connection)
  - Encryption integration (AES-GCM encryption/decryption built-in)
  - Buffer pooling (ConcurrentByteBufferPool for zero-allocation buffer reuse)
  - Historical problems (byte alignment issues, packet boundaries, buffer overflow/underflow)
  - Solutions and best practices (alignment validation, packet markers, bounds checking)
  - Rust implementation strategy (zero-allocation, memory safety, performance optimizations)
  - Performance optimizations (SIMD, zero-copy, inline functions, thread-local storage)

- **[GOLD.md](./docs/core/GOLD.md)** - Gold economy system documentation
  - Primary currency system (base of all economic transactions)
  - Gold acquisition (monster drops, quests, events, trade, auction house)
  - Gold usage (crafting, repair, housing, NPC purchases, guild creation)
  - Death penalty (gold drop on death, percentage-based loss)
  - Bank storage (city banks, safe storage, account-wide)
  - Item value calculation (tier, quantity, attributes, rarity, durability)
  - Rare attribute bonus (increased value for rare attributes)
  - Administrator fixed prices (override calculated values)
  - Proportional cost system (crafting and repair costs proportional to item value)
  - Gold management (inventory storage, bank operations)
  - Rust implementation considerations

### Items Documentation

- **[ITEMS.md](./docs/items/ITEMS.md)** - Items system documentation
  - Multi-level item hierarchy (Item, Stackable, Equipment, Weapon, Tool, etc.)
  - Item types (equipment, consumables, resources, quest items)
  - Rarity system (Common, Uncommon, Rare, Magic, Legendary, Unique, Quest)
  - Tier system (T0-T8) with attribute and durability bonuses, stat requirements (Level 1-100)
  - Prefix/Suffix system (Path of Exile 2 style, positive/negative modifiers)
  - Material-based attributes (Physical, Elemental, Hybrid materials)
  - Tier-based attribute maximums (limits per tier for balance)
  - Stat requirements by tier (Str/Dex/Int requirements for class filtering)
  - Attribute system (80+ attribute types for equipment bonuses)
  - Durability system (equipment degradation over time)
  - Card system (card slots for equipment, card insertion)
  - Resistance system (Physical, Fire, Cold, Poison, Energy, Light, Dark)
  - Item states/flags (Exceptional, Insured, Blessed, Broken)
  - Dynamic item registration (API/dashboard, YAML format)
  - CDN image loading (on-demand loading with local caching)
  - Gold cost calculation (rarity and flag modifiers)
  - Crafting system (modifier rerolling, locking, removal)
  - RMT economy (rare combinations, perfect rolls, niche markets)
  - Item serialization (TypeScript and C# implementations)
  - Rust implementation considerations (performance, memory, type safety)

- **[CONTAINERS.md](./docs/items/CONTAINERS.md)** - Container system documentation
  - Container types (Inventory, Equipment, Storage, Loot, Vendor, Trade)
  - Weight system (weight limits, weight-based mobility)
  - Item stacking (automatic stacking of stackable items)
  - Equipment slots (specific slots for different equipment types)
  - Event system (notifications for item modifications)
  - Item ID management (unique IDs for container items)
  - Container serialization (full container state persistence)
  - Weight-based mobility (0-100% normal, 100-200% progressive reduction)
  - Container queries (find items by name, type, tool type)
  - Item usage (consumable items, blueprint placement)
  - Rust implementation considerations (performance, memory, type safety)

- **[DURABILITY.md](./docs/items/DURABILITY.md)** - Durability and repair system documentation
  - Durability degradation (combat, usage, gathering)
  - Repair methods (self-repair, skill-based repair, NPC artisan repair)
  - Skill-based repair (no durability loss with crafting skills)
  - Durability enhancement (increasing maximum durability with materials)
  - Progressive durability loss (repairs without skill reduce max durability)
  - NPC artisan services (premium repair at higher costs)
  - Broken item state (0 durability effects)
  - Repair cost calculation (based on item value and missing durability)
  - Repair skill requirements (skill mapping by item type and tier)
  - Durability enhancement materials (ores, gemstones, enhancement values)
  - Rust implementation considerations

- **[MAGIC_STONES.md](./docs/items/MAGIC_STONES.md)** - Magic Stones system documentation
  - 14 Magic Stone types with creative names (Ascension Shard, Metamorphic Core, Divine Spark, Primordial Fragment, etc.)
  - Rare drops from elite monsters and bosses (low probability)
  - Guaranteed effects (100% success rate, no RNG on use)
  - Rarity transformation stones (Ascension Shard, Metamorphic Core, Arcane Catalyst, Divine Spark, Primordial Fragment)
  - Attribute enhancement stones (Essence Fragment, Primal Essence, Enchanted Essence, Chaos Essence)
  - Modification stones (Vortex Shard, Flux Crystal, Apex Gem, Transcendence Orb, Receptacle Stone)
  - Drop system (elite and boss drop rates, probability tables)
  - Stone usage system (validation, guaranteed effects, item requirements)
  - Trade currency (stones can be used in auction house, stone trading)
  - Stone rarity and value (rarity classification, value factors, market-driven pricing)
  - Rust implementation considerations

### Player Documentation

- **[PLAYER.md](./docs/player/PLAYER.md)** - Player system documentation
  - Level progression system (Level 1-100)
  - Experience system (non-linear progression curve)
  - Monster XP system (level-based XP penalties)
  - Stat point allocation (5 points per level)
  - Six core attributes (STR, DEX, INT, LCK, CON, CHA)
  - Attribute effects and formulas (damage, HP, resistance, etc.)
  - Build examples (Warrior, Rogue, Mage, Crafter)
  - Level-up system and rewards
  - Stat point respec system
  - Integration with equipment, combat, crafting, and economy systems
  - Rust implementation considerations

- **[SKILLS.md](./docs/player/SKILLS.md)** - Skills system documentation
  - 28 skills covering combat, magic, gathering, crafting, and social
  - Skill categories (Combat, Magic, Gathering, Crafting, Social, Special)
  - Experience-based progression system
  - Skill caps and maximum levels
  - Stat experience integration (primary/secondary stats)
  - Skill bonuses and effectiveness calculations
  - Stat point rewards from skill level-ups
  - Weapon-skill mapping (automatic skill assignment)
  - Crafting skill requirements and success rates
  - Skill serialization and persistence
  - Achievements system (skill-based achievements)
  - Rust implementation considerations

- **[GATHERING.md](./docs/player/GATHERING.md)** - Gathering system documentation
  - No skill restrictions (all resources collectable)
  - Skill-based efficiency (speed and success rate)
  - Equipment bonuses (percentage yield increases)
  - Tick-based collection system (1-20 ticks per resource)
  - Rare resource limits (5 ticks maximum for rarest)
  - RMT spots (larger spots through real money transactions)
  - Server-controlled respawn (timeout-based, blocked by nearby players)
  - Progressive rewards (double/triple chances at higher skills)
  - Resource yield calculation (skill + equipment bonuses)
  - Respawn mechanics (proximity checks, timer management)
  - Skill experience gain (limited by max skill gain thresholds)
  - Rust implementation considerations

### Client Documentation

- **[ENTITY_SYNC.md](./docs/client/ENTITY_SYNC.md)** - Entity synchronization documentation
  - Entity synchronization architecture (Unreal Engine plugin)
  - Position interpolation (smooth interpolation, extrapolation)
  - Animation synchronization (state machine, blending)
  - Quantized position system (reduce bandwidth)
  - Delta compression (only send changed data)
  - Client-side prediction (reduce latency)
  - Network replication system (AOI-based updates)
  - Unreal Engine integration (C++ plugin structure)
  - Performance optimization (adaptive update rate, priority-based updates, packet batching)

- **[SUBSYSTEM.md](./docs/client/SUBSYSTEM.md)** - Unreal Engine subsystem architecture documentation
  - Subsystem architecture (GameInstanceSubsystem for global reactive access)
  - Blueprint integration (BlueprintCallable, BlueprintAssignable delegates)
  - UI integration (reactive UI updates with network events)
  - Threading architecture (background packet polling, game thread processing)
  - Handshake system (cookie, crypto, reliable handshake)
  - Configuration system (centralized, validated configuration)

### Documentation Overview

- **[README.md](./docs/README.md)** - Detailed documentation overview
  - Architecture overview
  - Performance targets
  - Key features
  - Requirements
  - Development guide

## Quick Start

### Requirements

- Rust 1.70+ (edition 2021)
- Tokio for async I/O
- Unreal Engine 5.5+ for client

### Project Structure

```
Server/
├── docs/           # Documentation
│   ├── server/     # Server-side documentation
│   │   ├── ARCHITECTURE.md
│   │   ├── STATUS.md
│   │   └── ROADMAP.md
│   ├── core/       # Core systems documentation
│   │   ├── BYTEBUFFER.md
│   │   ├── ENTITIES.md
│   │   ├── FLAGS.md
│   │   ├── CONDITIONS.md
│   │   ├── TEAMS.md
│   │   ├── MOVEMENT.md
│   │   └── GOLD.md
│   ├── items/      # Items and containers documentation
│   │   ├── ITEMS.md
│   │   ├── CONTAINERS.md
│   │   ├── DURABILITY.md
│   │   └── MAGIC_STONES.md
│   ├── player/     # Player system documentation
│   │   ├── PLAYER.md
│   │   ├── SKILLS.md
│   │   └── GATHERING.md
│   ├── client/     # Client-side documentation
│   │   ├── ENTITY_SYNC.md
│   │   └── SUBSYSTEM.md
│   └── README.md
├── src/            # Rust source code (to be implemented)
├── Cargo.toml      # Rust project configuration (to be created)
└── README.md       # This file
```

## Performance Targets

- **Tick Rate**: 60 FPS (16.67ms per frame)
- **Concurrent Players**: 10,000 simultaneous connections
- **Packet Throughput**: 600,000 UDP packets per second
- **Latency**: < 50ms (p99)
- **Bandwidth Optimization**: 60-75% reduction through compression and quantization

## Key Features

- **High Performance**: Zero-allocation, lock-free data structures, batch processing
- **Security**: End-to-end encryption, replay protection, integrity checks
- **Scalability**: AOI system, entity pooling, adaptive sync
- **Bandwidth Efficiency**: Quantization, compression, delta compression
- **Reliability**: Reliable UDP, fragmentation, heartbeat system

## Implementation Status

The server is currently in the planning phase. See [STATUS.md](./docs/STATUS.md) for detailed status information.

- **C# Version**: Functional but limited to ~1,000 concurrent players
- **Rust Version**: Implementation planned (target: 10,000 concurrent players)

## Roadmap

The implementation is divided into 9 phases over 20 weeks:

1. **Phase 1**: Core Network Layer (Weeks 1-2)
2. **Phase 2**: Serialization and Compression (Weeks 3-4)
3. **Phase 3**: Security and Encryption (Weeks 5-6)
4. **Phase 4**: Entity System (Weeks 7-8)
5. **Phase 5**: Area of Interest (AOI) System (Weeks 9-10)
6. **Phase 6**: Reliable UDP (Weeks 11-12)
7. **Phase 7**: Authority System (Weeks 13-14)
8. **Phase 8**: Buffs and Debuffs (Weeks 15-16)
9. **Phase 9**: Optimization and Testing (Weeks 17-20)

See [ROADMAP.md](./docs/ROADMAP.md) for detailed roadmap information.

## Contributing

### Code Style

- Follow Rust style guidelines
- Use `rustfmt` for formatting
- Use `clippy` for linting
- Write comprehensive tests

### Documentation

- Document all public APIs
- Write inline documentation
- Update architecture documentation
- Keep roadmap up to date

### Testing

- Write unit tests for all functions
- Write integration tests for systems
- Write load tests for performance
- Write stress tests for robustness

## License

This project is licensed under the MIT License.

## Contact

For questions or issues, please contact the development team.

## Acknowledgments

- C# version implementation for architecture reference
- Unreal Engine team for client integration support
- Rust community for excellent libraries and tools
