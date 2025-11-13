# Tales of Shadowland MMORPG Server

## Overview

The Tales of Shadowland MMORPG server is a high-performance game server designed to support 10,000 concurrent players with a 60 FPS tick rate and processing 600,000 UDP packets per second. The server is being implemented in **Rust (Edition 2024, nightly 1.85+)** to achieve maximum performance while maintaining security and scalability.

The project follows **spec-driven development** using **OpenSpec** to ensure alignment between requirements and implementation before code is written.

> **Note**: Legacy TypeScript and C# implementations serve as **reference material only** for understanding game systems. The production server is being built from scratch in Rust. See [VERSIONS.md](./VERSIONS.md) for details.

## Architecture

The server architecture is designed around the following principles:

- **High Performance**: Zero-allocation, lock-free data structures, batch processing
- **Security**: End-to-end encryption, replay protection, integrity checks
- **Scalability**: AOI system, entity pooling, adaptive sync
- **Bandwidth Efficiency**: Quantization, compression, delta compression
- **Reliability**: Reliable UDP, fragmentation, heartbeat system

For detailed architecture documentation, see [ARCHITECTURE.md](./server/ARCHITECTURE.md).

## Status

### Current Status

The **Rust version** is the only active development target. Legacy TypeScript and C# versions are **deprecated** and serve only as reference material for understanding game systems and mechanics.

**Rust Version (Active Development):**
- 🔲 Implementation in progress (see [ROADMAP.md](./server/ROADMAP.md))
- 🎯 Target: 10,000 concurrent players
- 🎯 Target: 600,000 packets/second
- 🎯 Target: 60 FPS stable tick rate
- ⚡ Zero-allocation design
- 🔒 Advanced security (ChaCha20-Poly1305, X25519)
- 📊 Complete telemetry system (hardware, network, performance metrics)
- 💰 Advanced economy systems (auction house, magic stone market, bulk orders, lottery)
- 🏗️ Custom ECS framework (zero-allocation, lock-free)
- 🗄️ Dual database architecture (SQLite for game server, PostgreSQL for API principal)
- 🌐 API principal with admin dashboard

**Legacy Versions (Reference Only):**
- 📚 TypeScript: Complete game systems reference
- 📚 C#: Historical architecture reference
- ⚠️ Not recommended for production use
- ⚠️ Limited to ~1,000 concurrent players

For detailed status information, see [STATUS.md](./server/STATUS.md).  
For version clarification, see [VERSIONS.md](./VERSIONS.md).  
For migration guidelines, see [MIGRATION.md](./MIGRATION.md).

## Roadmap

The implementation is divided into 9 phases:

1. **Phase 1**: Core Network Layer (Weeks 1-2)
2. **Phase 2**: Serialization and Compression (Weeks 3-4)
3. **Phase 3**: Security and Encryption (Weeks 5-6)
4. **Phase 4**: Entity System (Weeks 7-8)
5. **Phase 5**: Area of Interest (AOI) System (Weeks 9-10)
6. **Phase 6**: Reliable UDP (Weeks 11-12)
7. **Phase 7**: Authority System (Weeks 13-14)
8. **Phase 8**: Buffs and Debuffs (Weeks 15-16)
9. **Phase 9**: Optimization and Testing (Weeks 17-20)

For detailed roadmap information, see [ROADMAP.md](./server/ROADMAP.md).

## Performance Targets

### Server Targets
- **Tick Rate**: 60 FPS (16.67ms per frame)
- **Concurrent Players**: 10,000 simultaneous connections
- **Packet Throughput**: 600,000 UDP packets per second
- **Latency**: < 50ms (p99)
- **Bandwidth Optimization**: 60-75% reduction through compression and quantization

### Network Targets
- **Minimal Header**: 14 bytes per packet
- **MTU**: 1200 bytes (optimized for UDP)
- **Overhead Reduction**: Batch packet queuing to reduce protocol overhead
- **Compression**: LZ4 for packets > 512 bytes with low entropy
- **Delta Compression**: 60-75% bandwidth reduction for entity updates

## Key Features

### Network Layer
- Binary UDP protocol
- Minimal packet header (14 bytes)
- Batch packet processing
- Packet fragmentation and reassembly
- Zero-allocation buffer management

### Serialization
- FlatBuffer implementation (zero-allocation)
- VarInt/ZigZag encoding (75% size reduction)
- Quantization (float to int16, 60% bandwidth reduction)
- World origin rebasing (large world support)
- LZ4 compression (30-50% size reduction)
- Delta compression (60-75% bandwidth reduction)

### Security
- X25519 key exchange (perfect forward secrecy)
- ChaCha20-Poly1305 AEAD encryption
- Replay protection (64-position sliding window)
- Cookie anti-spoof (HMAC-SHA256, DDoS protection)
- Heartbeat system (connection monitoring)
- Integrity system (client verification)
- Certificate rotation (rekey system)

### Entity System
- **Custom ECS framework** (zero-allocation, lock-free)
- Structure of Arrays (SoA) layout for cache efficiency
- Zero-allocation entity pooling
- Component-based entity types (Player, NPC, Monster, etc.)
- System-based processing (movement, combat, AI, conditions)
- Entity creation, update, removal packets
- Entity delta compression

### Movement and Collision System
- 3D grid-based spatial partitioning
- Server-side movement validation (prevent hacks)
- Collision detection (static and dynamic entities)
- Pathfinding system (A* for AI navigation)
- Patrol route system (monster AI)
- Swimming system (water areas, state management)
- Anti-cheat measures (fly hack, wall clipping, speed hack detection)

### ByteBuffer Architecture
- ByteBuffer evolution (multiple versions analyzed)
- VarInt/ZigZag encoding (variable-length integer encoding)
- Quantization system (position, rotation, float quantization)
- Delta compression (position delta compression)
- Symbol pooling (string symbol caching)
- Encryption integration (AES-GCM encryption/decryption)
- Buffer pooling (ConcurrentByteBufferPool for buffer reuse)
- Historical problems (byte alignment, packet boundaries, buffer overflow)
- Solutions and best practices (alignment, bounds checking, packet markers)
- Rust implementation strategy (zero-allocation, memory safety, performance)

### Client-Server Synchronization
- Entity synchronization architecture (Unreal Engine plugin)
- Position interpolation (smooth interpolation, extrapolation)
- Animation synchronization (state machine, blending)
- Quantized position system (reduce bandwidth)
- Delta compression (only send changed data)
- Client-side prediction (reduce latency)
- Network replication system (AOI-based updates)
- Unreal Engine integration (C++ plugin structure)
- Performance optimization (adaptive update rate, priority-based updates, packet batching)
- Subsystem architecture (GameInstanceSubsystem for global reactive access)
- Blueprint integration (BlueprintCallable, BlueprintAssignable delegates)
- UI integration (reactive UI updates with network events)
- Threading architecture (background packet polling, game thread processing)
- Handshake system (cookie, crypto, reliable handshake)
- Configuration system (centralized, validated configuration)

### Area of Interest (AOI)
- Grid-based AOI system
- Distance-based filtering
- Real-time entity updates
- Adaptive sync (movement-based)
- Configurable entity distances

### Reliable UDP
- Reliable messaging system
- Acknowledgment system
- Retransmission mechanism
- Reliable handshake
- Sequence number management

### Authority System
- Server authority for position
- Server authority for rotation
- Server authority for actions
- Movement validation
- Action validation

### Buffs and Debuffs
- Entity state flags
- Enemy/friend detection (1-to-1)
- Real-time state updates
- State serialization

## Structure Review

The server structure has been analyzed against industry standards. See:

- [STRUCTURE_SUMMARY.md](./STRUCTURE_SUMMARY.md) - Quick reference and key decisions
- [STRUCTURE_ANALYSIS.md](./STRUCTURE_ANALYSIS.md) - Detailed market comparison
- [STRUCTURE_RECOMMENDATIONS.md](./STRUCTURE_RECOMMENDATIONS.md) - Implementation recommendations

## Documentation

### Architecture Documentation

- [ARCHITECTURE.md](./server/ARCHITECTURE.md) - Complete architecture documentation
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

- [STATUS.md](./server/STATUS.md) - Current implementation status
  - C# version status (completed systems, limitations)
  - Rust version status (planned systems, targets)
  - Performance metrics
  - Known limitations
  - Historical versions

### Implementation Roadmap

- [ROADMAP.md](./server/ROADMAP.md) - Implementation roadmap
  - 9 implementation phases
  - Dependencies between phases
  - Risk mitigation
  - Success metrics

### Telemetry and Monitoring

- [TELEMETRY.md](./server/TELEMETRY.md) - Server telemetry system
  - Hardware metrics (CPU, memory, disk I/O)
  - Network metrics (latency, packets/sec, ping)
  - Performance metrics (FPS, tick rate, update cycles)
  - Game metrics (players, entities, database)
  - Prometheus export format
  - API principal integration

### Advanced Economy Systems

- [ECONOMY.md](./server/ECONOMY.md) - Advanced economy systems
  - Gold burn system (inflation control)
  - Auction house (buyout and bidding)
  - Magic stone market (PoE2-style currency exchange)
  - Bulk orders (player-created crafting orders)
  - NPC bulk orders (NPC-created orders)
  - Lottery system (weekly lottery with gold burn integration)

### System Documentation

- [FLAGS.md](./core/FLAGS.md) - Flags system documentation
  - Entity states (bitwise flags)
  - Buff/debuff states
  - Item states
  - Flag operations
  - Serialization
  - State synchronization

- [CONDITIONS.md](./core/CONDITIONS.md) - Conditions system documentation
  - Condition types (DoT, HoT, status effects)
  - Condition lifecycle (apply, update, refresh, remove)
  - Buff/debuff system
  - Condition management
  - Serialization

- [TEAMS.md](./core/TEAMS.md) - Teams system documentation
  - Team types (Monsters, Players, NPCs, etc.)
  - Team relationships (ally/enemy)
  - PvP and PvE logic
  - Team-based gameplay
  - Team relationships matrix

- [ENTITIES.md](./core/ENTITIES.md) - Entity system architecture documentation
  - Entity hierarchy (TypeScript and C# implementations)
  - Performance concerns (vtable overhead, cache misses, memory allocation)
  - Rust implementation strategies (ECS pattern, SoA layout, zero-allocation pooling)
  - Component-based architecture
  - System-based processing
  - Performance comparison (inheritance vs ECS)

- [MOVEMENT.md](./core/MOVEMENT.md) - Movement and collision system documentation
  - Grid-based collision system (2D and 3D)
  - Movement validation (prevent hacks: fly, wall clipping, speed)
  - Pathfinding system (A* for AI navigation)
  - Patrol route system (monster AI)
  - Swimming system (water areas, state management, animation triggers)
  - Security considerations (anti-cheat, rate limiting)
  - Performance optimizations (spatial hashing, broad/narrow phase)

- [BYTEBUFFER.md](./core/BYTEBUFFER.md) - ByteBuffer architecture documentation
  - ByteBuffer evolution (ByteBuffer.cs, FlatBuffer.cs, UFlatBuffer.cpp, bytebuffer.ts)
  - VarInt/ZigZag encoding (variable-length integer encoding)
  - Quantization system (position, rotation, float quantization)
  - Delta compression (position delta compression)
  - Symbol pooling (string symbol caching)
  - Encryption integration (AES-GCM encryption/decryption)
  - Buffer pooling (ConcurrentByteBufferPool for buffer reuse)
  - Byte alignment issues (historical problems and solutions)
  - Packet boundary issues (packet isolation and markers)
  - Buffer overflow/underflow (bounds checking and validation)
  - Thread safety (thread-local storage, lock-free structures)
  - Memory management (zero-allocation, buffer pooling)
  - Rust implementation strategy (zero-allocation, memory safety, performance)
  - Performance optimizations (SIMD, zero-copy, inline functions)
  - Testing strategy (unit tests, integration tests, performance tests)

- [GOLD.md](./core/GOLD.md) - Gold economy system documentation
  - Gold acquisition methods (monster drops, quests, events, trade, auction house)
  - Gold usage (crafting, repair, housing, NPC purchases)
  - Death penalty (gold drops on death)
  - Bank storage system
  - Item value calculation (tier, quantity, attributes)
  - Crafting and repair cost formulas
  - Rust implementation considerations

- [RESISTANCES.md](./core/RESISTANCES.md) - Resistance system documentation
  - Physical and elemental resistance types (Fire, Cold, Poison, Energy, Light, Dark)
  - Attribute effects (Constitution for physical, Intelligence for elemental)
  - Equipment resistance modifiers
  - Zone resistance modifiers (difficulty progression)
  - Weapon damage type conversion
  - Magic Resistance skill (dynamic resistance based on damage taken)
  - Maximum resistance cap (70%)
  - Creature immunities
  - Damage calculation formulas (positive and negative resistance)
  - Rust implementation considerations

- [DAMAGE.md](./core/DAMAGE.md) - Damage system documentation
  - Dice-based damage system for stable growth and PvP balance
  - Equipment dice configurations (D4, D6, D8, D10, D12, D20)
  - Multiple dice system (e.g., 2D6, 5D10, 6D12)
  - 10x multiplier for progression feel (all damage values multiplied by 10)
  - PvP damage limit (50% of target's max HP per attack)
  - Damage components (equipment dice roll * 10, attribute modifiers * 10, equipment * 10, skills * 10)
  - Linear damage scaling to prevent exponential growth
  - Endgame balance (HP vs maximum damage)
  - Mobility factor (optional zero padding for build flexibility)
  - Physical and elemental damage calculations
  - Complete damage calculation flow
  - PvP balance examples and scenarios
  - Rust implementation considerations

- [STATUS.md](./core/STATUS.md) - Status system documentation
  - Core attributes (STR, DEX, INT, VIG) and their effects
  - Resource calculations (HP, Mana, Stamina) with 10x multiplier for progression feel
  - HP formula: Base + (VIG * 50) + (STR * 10)
  - Mana formula: Base + (INT * 30)
  - Stamina formula: Base + (STR * 20) + (DEX * 30) + (VIG * 10)
  - Balance with damage system (HP vs damage, Mana vs spell costs, Stamina vs action costs)
  - Resource regeneration system (multiplied by 10)
  - Build examples (Tank, DPS, Mage, Hybrid, Rogue)
  - Attribute distribution examples
  - Rust implementation considerations

- [SECONDARY_STATS.md](./core/SECONDARY_STATS.md) - Secondary statistics system documentation
  - Defensive mechanics that depend on primary attributes and equipment
  - Evasion system (chance to avoid damage, scales with DEX, cap 75%)
  - Block system (chance to reduce/negate physical damage and projectiles, requires shield)
  - Armor system (percentage reduction in physical damage, cap 90%)
  - Energy Shield system (secondary health pool that absorbs damage, scales with INT, regenerates over time)
  - Complete damage calculation flow with all secondary stats
  - Build diversity (Evasion, Armor, Block, Energy Shield builds)
  - Balance considerations and stat interactions
  - Integration with damage, resistance, and equipment systems
  - Rust implementation considerations

- [ANCIENT_SCROLLS.md](./core/ANCIENT_SCROLLS.md) - Ancient Scrolls system documentation
  - God-exclusive drops (only obtainable by defeating gods)
  - Party rewards (6 random scrolls per god defeat)
  - Skill cap increases (permanently increase caps to 105, 110, 115, or 120)
  - Random distribution (scrolls randomly target skills and levels)
  - Tradeable items (can be traded between players)
  - Usage system (double-click + confirmation to consume)
  - Impact on combat, crafting, gathering, and all skill systems
  - RMT integration (completely random RMT scrolls)
  - Economic impact and trading dynamics
  - Rust implementation considerations

- [POWER_OF_GODS.md](./core/POWER_OF_GODS.md) - Power of Gods system documentation
  - Final boss exclusive drop (only obtainable by defeating the final boss)
  - Massive attribute bonuses (+10, +20, +30, or +50 attribute points)
  - Flexible distribution (can be allocated to STR, DEX, INT, CON, LCK, CHA)
  - Permanent enhancement (once used, permanently increases attributes)
  - Ultimate power (up to +10% additional stat points - 50/500)
  - Competitive advantage (creates massive power differences between players)
  - System-wide impact (affects combat, crafting, gathering, and all systems)
  - Rare drop (extremely rare, only from final boss)
  - Legendary item (highest tier item in the game)
  - Economic value (one of the most valuable items)
  - Rust implementation considerations

### Items Documentation

- [EQUIPMENT.md](./items/EQUIPMENT.md) - Equipment system documentation
  - 9 equipment types (Armor, Boots, Gloves, Helmet, Mainhand, Offhand, Belt, Ring x2, Necklace)
  - Visual representation and appearance alteration
  - Attribute bonuses from equipment
  - Equipment slots and inventory system
  - Visual skins (cosmetic equipment, no stats)
  - Dye system for visual skins
  - Tools (optional gathering equipment)
  - Drop system (creatures and players)
  - Two-handed weapon restrictions
  - Ring slots (2 separate slots)
  - Rust implementation considerations

- [CRAFTING.md](./items/CRAFTING.md) - Crafting system documentation
  - No crafting failures (always succeeds with resources)
  - Skill-based quality (low skill = common, high skill = better rarity)
  - Resource variety (same item with different resources, Ultima Online style)
  - Progressive XP system (higher quality resources give more XP, capped at 10% above minimum)
  - Rarity lottery system (chance-based uncommon, rare, magic, legendary)
  - Gold costs for all crafting
  - Specialized tools (improve rarity chances, cost more to use)
  - Destruction and recovery system (destroy items to recover partial resources)
  - Component tracking (items store their component resources)
  - Recipe system (all items have recipes with required resources)
  - Maximum skill requirement: 100 for any equipment
  - Crafting skills: Blacksmithing, Leatherworking, Tailoring, Jewelry
  - Rust implementation considerations

- [ITEMS.md](./items/ITEMS.md) - Items system documentation
  - Multi-level item hierarchy (Item, Stackable, Equipment, Weapon, Tool, etc.)
  - Item types (equipment, consumables, resources, quest items)
  - Rarity system (Common, Uncommon, Rare, Magic, Legendary, Unique, Quest)
  - Tier system (T0-T8) with attribute and durability bonuses, stat requirements (Level 1-100)
  - Prefix/Suffix system (Path of Exile 2 style, positive/negative modifiers)
  - Material-based attributes (Physical, Elemental, Hybrid materials)
  - Tier-based attribute maximums (limits per tier for balance)
  - Stat requirements by tier (Str/Dex/Int requirements for class filtering)
  - Attribute system (90+ attribute types for equipment bonuses)
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

- [ITEM_NAMING.md](./items/ITEM_NAMING.md) - Item naming system documentation
  - Automatic name generation based on base name and most important attributes
  - Rarity-based naming patterns (Normal, Magic, Rare, Unique)
  - Three-layer mod system (Implicit, Explicit, Enchantment)
  - Magic items: Prefix and/or suffix in name (e.g., "Fiery Iron Ring of the Fox")
  - Rare items: Pseudo-random titles linked to affixes (e.g., "Doom Veil Leather Hood")
  - Unique items: Completely custom names (e.g., "Kaom's Heart")
  - Attribute to name mapping (prefixes, suffixes, titles)
  - Tooltip display format with mod separation
  - Integration with attribute, enchantment, and rarity systems
  - Rust implementation considerations

- [ATTRIBUTES.md](./items/ATTRIBUTES.md) - Item attributes system documentation
  - Complete list of 90+ attribute types
  - Attribute categories (Core Stats, Resources, Damage, Defense, Speed, Efficiency, Luck, Gathering)
  - Spawn rates for each attribute (Common, Uncommon, Rare, Very Rare)
  - Equipment restrictions (Weapons, Armor, Accessories, Tools)
  - Attribute effects and value ranges (tier-dependent)
  - Value impact system (how attributes affect item value)
  - Tier system (T1-T8) with value scaling
  - Material type restrictions (Physical, Elemental, Hybrid)
  - Attribute generation system (random attribute creation)
  - Rust implementation considerations

- [ENCHANTMENTS.md](./items/ENCHANTMENTS.md) - Enchantment system documentation
  - Elemental enchantments (Fire, Cold, Poison, Energy, Light, Dark)
  - Six enchantment levels (+1 to +6) with progressive bonuses (20% to 100%)
  - Skill-based success rates and requirements
  - Material requirements (elemental essences + star dust)
  - Visual effects (VFX particles and color changes)
  - Property multiplication (enchants multiply related elemental properties)
  - Upgrade system (upgrade existing enchantments to higher levels)
  - Disenchantment system (remove enchantments with scrolls)
  - Failure mechanics (chance to fail, losing partial resources)
  - Value impact (enchanted items are significantly more valuable)
  - Rust implementation considerations

- [CONTAINERS.md](./items/CONTAINERS.md) - Container system documentation
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

- [DURABILITY.md](./items/DURABILITY.md) - Durability and repair system documentation
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

- [MAGIC_STONES.md](./items/MAGIC_STONES.md) - Magic Stones system documentation
  - 14 Magic Stone types with creative names (Ascension Shard, Metamorphic Core, Divine Spark, Primordial Fragment, etc.)
  - Rare drops from elite monsters and bosses (low probability)
  - Guaranteed effects (100% success rate, no RNG on use)
  - Rarity transformation stones (Ascension Shard, Metamorphic Core, Arcane Catalyst, Divine Spark, Primordial Fragment)
  - Attribute enhancement stones (Essence Fragment, Primal Essence, Enchanted Essence, Chaos Essence)
  - Modification stones (Vortex Shard, Flux Crystal, Apex Gem, Transcendence Orb, Receptacle Stone)

- [RUNES.md](./items/RUNES.md) - Runes system documentation
  - Slot system for equipment (items can drop with slots or have slots added via Magic Stones)
  - Equipment-specific slot limits (two-handed weapons: 3, shields/offhands: 2, armor: 6, others: 1, accessories: 0)
  - Rune types (Physical, Fire, Cold, Poison, Energy, Light, Dark resistance)
  - Rune levels (1-10) with progressive resistance bonuses (1% to 15%)
  - Rune drop system (dropable from creatures, level distribution based on creature rarity)
  - Rune upgrade system (combine lower level runes to create higher level runes)
  - Single rune limit per equipment (cannot use 2 runes)
  - Visual effects (rune type colors, level-based glow intensity)
  - Integration with resistance, equipment, and cards systems
  - Rust implementation considerations

- [CARDS.md](./items/CARDS.md) - Cards system documentation
  - Extremely rare collectible items (less than 1% drop chance from all creatures)
  - Boss cards (even rarer, 0.1-0.3% drop chance)
  - Card types (Attribute, Damage, Defense, Resistance, Immunity, Utility, Special)
  - Immunity cards (grant immunity to damage types, rarest cards)
  - Single card limit per equipment (cannot use 2 cards)
  - Cards preserved on removal (unlike runes)
  - Steam integration (collectible rewards, badges, profile items)
  - Card game integration (internal MMO card game, deck building)
  - Wagering system (bet cards on PvP, bosses, tournaments)
  - Card trading and market system
  - Visual design and collection UI
  - Rust implementation considerations

### Player Documentation

- [RACES.md](./player/RACES.md) - Player races system documentation
  - 9 playable races (Humans, Elves, Half-Elves, Orcs, Devas/Dark Elves, Dwarves, Half-Orcs, Goblins, Half-Humans)
  - Racial attribute bonuses (base attribute modifiers)
  - Racial skills (unique skills per race)
  - Racial passives (permanent passive bonuses)
  - Race selection at character creation (permanent choice)
  - Build recommendations by race
  - Integration with status, skill, equipment, and experience systems
  - Rust implementation considerations

- [PLAYER.md](./player/PLAYER.md) - Player system documentation
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

- [SKILLS.md](./player/SKILLS.md) - Skills system documentation
  - 24 skills covering combat, magic, gathering, crafting, and special
  - Skill categories (Combat, Magic, Gathering, Crafting, Special)
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

### Detailed Skill Documentation

- [COMBAT_WITH_WEAPONS.md](./skills/COMBAT_WITH_WEAPONS.md) - One-handed melee weapons skill documentation
  - Weapon types (Axe, Dagger, Sword, Hammer)
  - Damage amplification and accuracy enhancement
  - No miss system (damage reduction instead)
  - Progressive perks at levels 2, 4, 6, 8, and 10
  - Stat bonuses from perks (+5 Strength, +5 Dexterity)
  - Build recommendations and integration with combat system
  - Rust implementation considerations

- [LONG_RANGE_WEAPONS.md](./skills/LONG_RANGE_WEAPONS.md) - Ranged weapons skill documentation
  - Weapon types (Bow, Crossbow)
  - Damage amplification, accuracy, and range enhancement
  - Range penalty system and optimal range mechanics
  - Progressive perks at levels 2, 4, 6, 8, and 10
  - Stat bonuses from perks (+5 Strength, +5 Dexterity)
  - Long-range combat mastery and close-range penalty elimination
  - Build recommendations and integration with combat system
  - Rust implementation considerations

- [TWO_HANDED_WEAPONS.md](./skills/TWO_HANDED_WEAPONS.md) - Two-handed melee weapons skill documentation
  - Weapon types (Two-Handed Axe, Sword, Hammer, Spear, Staff)
  - Higher damage multiplier (6% per level vs 5% for one-handed)
  - Damage amplification and accuracy enhancement
  - Progressive perks at levels 2, 4, 6, 8, and 10
  - Stat bonuses from perks (+5 Strength, +5 Dexterity)
  - Critical hit mastery (enhanced chance and damage)
  - Highest single-hit damage potential
  - Build recommendations and integration with combat system
  - Rust implementation considerations

- [UNARMED_COMBAT.md](./skills/UNARMED_COMBAT.md) - Unarmed combat skill documentation
  - No weapon requirement (always ready to fight)
  - Attribute-based damage (scales with Strength and Dexterity)
  - Damage amplification and accuracy enhancement
  - Attack speed bonuses (+15% total from perks)
  - Progressive perks at levels 2, 4, 6, 8, and 10
  - Stat bonuses from perks (+5 Strength, +5 Dexterity)
  - Fastest attack speed of all combat skills
  - Build recommendations and integration with combat system
  - Rust implementation considerations

- [GATHERING.md](./player/GATHERING.md) - Gathering system documentation
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

### Events Documentation

- [TREASURE_HUNT.md](./events/TREASURE_HUNT.md) - Treasure Hunt event system documentation
  - Map discovery through fishing (bottles with maps)
  - 10 difficulty levels with progressive challenge
  - Guardian creatures and boss encounters
  - Rich rewards including legendary equipment sets
  - Location-based treasure hunting mechanics
  - Shovel requirement and digging mechanics
  - Integration with fishing, combat, and equipment systems
  - Rust implementation considerations

### Client Documentation

- [ENTITY_SYNC.md](./client/ENTITY_SYNC.md) - Entity synchronization documentation
  - Entity synchronization architecture (Unreal Engine plugin)
  - Position interpolation (smooth interpolation, extrapolation)
  - Animation synchronization (state machine, blending)
  - Quantized position system (reduce bandwidth)
  - Delta compression (only send changed data)
  - Client-side prediction (reduce latency)
  - Network replication system (AOI-based updates)
  - Unreal Engine integration (C++ plugin structure)
  - Performance optimization (adaptive update rate, priority-based updates, packet batching)

- [SUBSYSTEM.md](./client/SUBSYSTEM.md) - Unreal Engine subsystem architecture documentation
  - Subsystem architecture (GameInstanceSubsystem for global reactive access)
  - Blueprint integration (BlueprintCallable, BlueprintAssignable delegates)
  - UI integration (reactive UI updates with network events)
  - Threading architecture (background packet polling, game thread processing)
  - Handshake system (cookie, crypto, reliable handshake)
  - Configuration system (centralized, validated configuration)

## Requirements

### Rust Implementation
- **Rust 1.85+ (Edition 2024, nightly toolchain)**
- Tokio for async I/O
- Mio for low-level UDP operations
- sqlx for database access (SQLite for game server, PostgreSQL for API principal)
- chacha20poly1305 for encryption
- LZ4 for compression
- sysinfo for telemetry
- prometheus for metrics export

### Database
- **Game Server**: SQLite (embedded, via sqlx)
- **API Principal**: PostgreSQL (via sqlx)

### Unreal Engine Client
- Unreal Engine 5.5+
- C++ plugin for network communication
- Hardware-accelerated CRC32C
- Optimized deserializer

### Development Tools
- cargo-nextest for fast test execution
- cargo-llvm-cov for code coverage (95%+ required)
- rustfmt (nightly toolchain)
- clippy (warnings as errors)
- codespell for typo detection

## Development

### Project Structure
```
Server/
├── Cargo.toml                    # Workspace root
├── README.md                     # Main README
│
├── crates/                       # All library crates
│   ├── tos-core/                # Core game logic + ECS
│   ├── tos-network/             # Networking
│   ├── tos-protocol/            # Protocol definitions
│   ├── tos-database/            # Database layer (SQLite)
│   ├── tos-api/                 # API principal (REST API)
│   └── tos-common/              # Shared utilities
│
├── bin/                          # Executables
│   ├── server/                   # Game server binary (UDP)
│   └── api/                      # API principal binary (HTTP/REST)
│
├── dashboard/                    # Web dashboard (frontend)
│   ├── package.json
│   ├── src/
│   └── public/
│
├── tests/                        # Integration tests
├── benches/                      # Benchmarks
├── examples/                     # Example code
│
├── openspec/                     # OpenSpec specifications
│   ├── project.md
│   ├── AGENTS.md
│   ├── specs/
│   └── changes/
│
└── docs/                         # Documentation
    ├── server/     # Server-side documentation
    │   ├── ARCHITECTURE.md
    │   ├── STATUS.md
    │   ├── ROADMAP.md
    │   ├── TELEMETRY.md
    │   └── ECONOMY.md
    ├── core/       # Core systems documentation
    │   ├── BYTEBUFFER.md
    │   ├── ENTITIES.md
    │   ├── FLAGS.md
    │   ├── CONDITIONS.md
    │   ├── TEAMS.md
    │   ├── MOVEMENT.md
    │   ├── GOLD.md
    │   ├── RESISTANCES.md
    │   ├── DAMAGE.md
    │   └── STATUS.md
    ├── items/      # Items and containers documentation
    │   ├── ITEMS.md
    │   ├── CONTAINERS.md
    │   ├── DURABILITY.md
    │   ├── MAGIC_STONES.md
    │   └── ATTRIBUTES.md
    ├── player/     # Player system documentation
    │   ├── PLAYER.md
    │   ├── SKILLS.md
    │   └── GATHERING.md
    ├── client/     # Client-side documentation
    │   ├── ENTITY_SYNC.md
    │   └── SUBSYSTEM.md
    └── README.md
```

### Building
```bash
# Build workspace (all crates)
cargo build --release

# Build game server
cargo build --release --bin server

# Build API principal
cargo build --release --bin api

# Run game server
cargo run --release --bin server

# Run API principal
cargo run --release --bin api

# Run all tests
cargo test --workspace --all-features

# Run tests with nextest (faster)
cargo nextest run --workspace --all-features

# Check code coverage (95%+ required)
cargo llvm-cov --all

# Format code (nightly toolchain)
cargo +nightly fmt --all

# Lint code (warnings as errors)
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Run benchmarks
cargo bench
```

### Dashboard Development
```bash
cd dashboard
npm install
npm run dev      # Development server
npm run build    # Production build
```

### Configuration

**Game Server Configuration:**
- Network configuration (port, MTU, buffers)
- Security configuration (encryption, integrity, heartbeat)
- Performance configuration (tick rate, timeouts)
- AOI configuration (distances, grid size)
- World origin rebasing configuration (quadrants, scale)
- SQLite database path and settings
- Telemetry collection settings

**API Principal Configuration:**
- HTTP server port and bind address
- PostgreSQL connection string
- CORS configuration
- Rate limiting settings
- Authentication token settings
- Telemetry endpoint configuration

## Testing

### Unit Tests
- Test coverage target: > 95%
- Test all critical paths
- Test error handling
- Test edge cases

### Integration Tests
- Test packet processing
- Test entity system
- Test AOI system
- Test reliable UDP

### Load Tests
- Test with 1,000 concurrent players
- Test with 5,000 concurrent players
- Test with 10,000 concurrent players
- Measure performance metrics

### Stress Tests
- Test under high load
- Test with packet loss
- Test with high latency
- Test with connection failures

## Performance Optimization

### Zero-Allocation
- Use stack allocation where possible
- Pre-allocate buffers
- Use object pooling
- Avoid Vec allocations in hot paths

### Lock-Free
- Use atomic operations
- Use lock-free queues
- Use RCU (Read-Copy-Update) patterns
- Minimize locking overhead

### Async I/O
- Use tokio for async I/O
- Use channels for message passing
- Use futures for concurrent operations
- Avoid blocking operations

### Hardware Acceleration
- Use hardware-accelerated CRC32C
- Use hardware-accelerated encryption (when available)
- Use SIMD for vector operations
- Optimize for CPU cache locality

## Security

### Encryption
- All packets encrypted with ChaCha20-Poly1305
- X25519 key exchange for perfect forward secrecy
- Separate TX/RX keys for bidirectional encryption
- Replay protection with 64-position sliding window

### Authentication
- Cookie anti-spoof for DDoS protection
- Integrity system for client verification
- Heartbeat system for connection monitoring
- Certificate rotation for long-term security

### Validation
- Server authority for all game state
- Movement validation
- Action validation
- Anti-cheat checks

## Contributing

### Development Workflow

**CRITICAL RULES:**
1. Always reference `@AGENTS.md` before coding
2. Write tests first (95%+ coverage required)
3. Run quality checks before committing:
   - Type check: `cargo check --workspace`
   - Format check: `cargo +nightly fmt --all -- --check`
   - Linter: `cargo clippy --workspace --all-targets --all-features -- -D warnings`
   - All tests: `cargo test --workspace --all-features` (100% pass rate)
   - Coverage: `cargo llvm-cov --all` (95%+ required)
4. Update docs/ when implementing features
5. Follow strict documentation structure

### Code Style
- Follow Rust style guidelines
- Use `rustfmt` with nightly toolchain: `cargo +nightly fmt --all`
- Use `clippy` with `-D warnings` (warnings as errors)
- Write comprehensive tests (95%+ coverage)
- Use Rust Edition 2024 (never 2021)

### OpenSpec Workflow
- Create OpenSpec proposals for new features/breaking changes
- Validate specs: `openspec validate --strict`
- Get approval before implementation
- Archive changes after deployment

### Documentation
- Document all public APIs with `///` doc comments
- Write inline documentation
- Update architecture documentation
- Keep roadmap up to date
- Update docs/ when implementing features

### Testing
- Write unit tests for all functions (`#[cfg(test)]` in same file)
- Write integration tests in `/tests` directory
- Write load tests for performance
- Write stress tests for robustness
- Use `cargo-nextest` for faster test execution
- Achieve 95%+ code coverage

## License

This project is licensed under the MIT License.

## Contact

For questions or issues, please contact the development team.

## Acknowledgments

- C# version implementation for architecture reference
- Unreal Engine team for client integration support
- Rust community for excellent libraries and tools
