# Server Dependency Graph (DAG)

## Overview

This document provides a Directed Acyclic Graph (DAG) of the server architecture, showing dependencies between crates, modules, and systems. The graph ensures no circular dependencies and provides a clear view of the build order and module relationships.

## High-Level Crate Dependencies

```mermaid
graph TD
    %% Base layer (no dependencies on other project crates)
    tos_common[tos-common<br/>Shared utilities, types, errors]
    
    %% Protocol layer
    tos_protocol[tos-protocol<br/>Packet definitions, serialization]
    tos_protocol --> tos_common
    
    %% Network layer
    tos_network[tos-network<br/>UDP server, connections, encryption]
    tos_network --> tos_protocol
    tos_network --> tos_common
    
    %% Database layer
    tos_database[tos-database<br/>SQLite/PostgreSQL, migrations]
    tos_database --> tos_common
    
    %% Core game logic
    tos_core[tos-core<br/>ECS, systems, game logic]
    tos_core --> tos_protocol
    tos_core --> tos_network
    tos_core --> tos_database
    tos_core --> tos_common
    
    %% API principal
    tos_api[tos-api<br/>REST API, admin endpoints]
    tos_api --> tos_database
    tos_api --> tos_common
    
    %% Binaries
    bin_server[bin/server<br/>Game server executable]
    bin_server --> tos_core
    bin_server --> tos_network
    bin_server --> tos_protocol
    bin_server --> tos_database
    bin_server --> tos_common
    
    bin_api[bin/api<br/>API principal executable]
    bin_api --> tos_api
    bin_api --> tos_database
    bin_api --> tos_common
    
    %% External dependencies (shown for context)
    ext_tokio[Tokio<br/>Async runtime]
    ext_sqlx[sqlx<br/>Database driver]
    ext_crypto[Crypto crates<br/>ChaCha20, X25519]
    
    tos_network -.-> ext_tokio
    tos_database -.-> ext_sqlx
    tos_network -.-> ext_crypto
```

## Detailed Module Dependencies

### tos-common (Base Layer)

```mermaid
graph TD
    common_types[types<br/>Common types, enums]
    common_errors[errors<br/>Error types, Result types]
    common_utils[utils<br/>Helper functions]
    common_config[config<br/>Configuration structures]
    
    common_types --> common_errors
    common_utils --> common_types
    common_config --> common_types
```

### tos-protocol (Protocol Layer)

```mermaid
graph TD
    protocol_types[types<br/>Entity, Player, World types]
    protocol_packets[packets<br/>Packet definitions]
    protocol_serialization[serialization<br/>ByteBuffer, VarInt, Quantization]
    
    protocol_types --> common_types
    protocol_packets --> protocol_types
    protocol_packets --> common_types
    protocol_serialization --> protocol_types
    protocol_serialization --> common_types
```

### tos-network (Network Layer)

```mermaid
graph TD
    network_server[server<br/>UDP server, event loop]
    network_connection[connection<br/>Connection management]
    network_packet[packet<br/>Packet handling, routing]
    network_reliable[reliable<br/>Reliable UDP, ACK, retransmit]
    network_encryption[encryption<br/>ChaCha20, X25519, replay]
    network_compression[compression<br/>LZ4, delta compression]
    
    network_server --> network_connection
    network_server --> network_packet
    network_connection --> network_reliable
    network_connection --> network_encryption
    network_packet --> protocol_packets
    network_packet --> protocol_serialization
    network_reliable --> network_packet
    network_encryption --> network_packet
    network_compression --> network_packet
    network_server --> common_errors
    network_connection --> common_errors
```

### tos-database (Database Layer)

```mermaid
graph TD
    db_pool[pool<br/>Connection pooling]
    db_migrations[migrations<br/>Schema migrations]
    db_models[models<br/>Database models]
    db_queries[queries<br/>Type-safe queries]
    
    db_pool --> common_config
    db_migrations --> db_pool
    db_models --> common_types
    db_queries --> db_pool
    db_queries --> db_models
```

### tos-core (Core Game Logic)

```mermaid
graph TD
    %% ECS Framework
    ecs_entity[ecs/entity<br/>Entity ID management]
    ecs_component[ecs/component<br/>Component storage SoA]
    ecs_system[ecs/system<br/>System trait, scheduler]
    ecs_query[ecs/query<br/>Query system]
    ecs_world[ecs/world<br/>ECS World]
    
    ecs_component --> ecs_entity
    ecs_system --> ecs_component
    ecs_system --> ecs_query
    ecs_query --> ecs_component
    ecs_world --> ecs_system
    ecs_world --> ecs_query
    
    %% Components
    comp_transform[components/transform<br/>Position, rotation]
    comp_health[components/health<br/>Health, mana, stamina]
    comp_stats[components/stats<br/>Character stats]
    comp_combat[components/combat<br/>Combat state]
    comp_inventory[components/inventory<br/>Inventory, equipment]
    comp_player[components/player<br/>Player-specific]
    
    comp_transform --> ecs_component
    comp_health --> ecs_component
    comp_stats --> ecs_component
    comp_combat --> ecs_component
    comp_inventory --> ecs_component
    comp_player --> ecs_component
    
    %% Systems
    sys_movement[systems/movement<br/>Movement system]
    sys_combat[systems/combat<br/>Combat system]
    sys_ai[systems/ai<br/>AI system]
    sys_conditions[systems/conditions<br/>Condition system]
    sys_buffs[systems/buffs<br/>Buff/debuff system]
    sys_respawn[systems/respawn<br/>Respawn system]
    
    sys_movement --> ecs_world
    sys_movement --> comp_transform
    sys_combat --> ecs_world
    sys_combat --> comp_combat
    sys_combat --> comp_health
    sys_ai --> ecs_world
    sys_ai --> comp_transform
    sys_conditions --> ecs_world
    sys_conditions --> comp_health
    sys_buffs --> ecs_world
    sys_buffs --> comp_stats
    sys_respawn --> ecs_world
    sys_respawn --> comp_health
    
    %% World Management
    world_map[world/map<br/>Map management]
    world_aoi[world/aoi<br/>Area of Interest]
    world_spatial[world/spatial<br/>Spatial partitioning]
    
    world_map --> ecs_world
    world_aoi --> world_spatial
    world_aoi --> ecs_world
    world_spatial --> ecs_world
    
    %% Combat Mechanics
    combat_damage[combat/damage<br/>Damage calculation]
    combat_skills[combat/skills<br/>Skill system]
    combat_actions[combat/actions<br/>Action system]
    
    combat_damage --> comp_stats
    combat_damage --> comp_combat
    combat_skills --> comp_stats
    combat_actions --> combat_skills
    combat_actions --> protocol_packets
    
    %% Player Systems
    player_progression[player/progression<br/>Level, XP]
    player_skills[player/skills<br/>Skill progression]
    player_quests[player/quests<br/>Quest system]
    
    player_progression --> comp_player
    player_skills --> comp_stats
    player_quests --> comp_player
    
    %% Telemetry
    telemetry_collector[telemetry/collector<br/>Metrics collector]
    telemetry_metrics[telemetry/metrics<br/>Metrics definitions]
    telemetry_hardware[telemetry/hardware<br/>CPU, memory]
    telemetry_network[telemetry/network<br/>Network metrics]
    telemetry_performance[telemetry/performance<br/>FPS, tick rate]
    telemetry_exporter[telemetry/exporter<br/>Prometheus export]
    
    telemetry_collector --> telemetry_metrics
    telemetry_hardware --> telemetry_collector
    telemetry_network --> telemetry_collector
    telemetry_performance --> telemetry_collector
    telemetry_exporter --> telemetry_collector
    
    %% Core dependencies on other crates
    tos_core --> protocol_packets
    tos_core --> network_connection
    tos_core --> db_queries
    sys_movement --> network_packet
    sys_combat --> network_packet
    combat_actions --> network_packet
```

### tos-api (API Principal)

```mermaid
graph TD
    api_server[server<br/>HTTP server Axum/Warp]
    api_config[config<br/>API configuration]
    api_routes[routes<br/>API routes]
    api_handlers[handlers<br/>Request handlers]
    api_services[services<br/>Business logic]
    api_middleware[middleware<br/>Auth, CORS, logging]
    api_sse[sse<br/>Server-Sent Events]
    api_backup[backup<br/>Backup system]
    
    api_server --> api_routes
    api_server --> api_middleware
    api_routes --> api_handlers
    api_handlers --> api_services
    api_services --> db_queries
    api_middleware --> common_errors
    api_sse --> api_services
    api_backup --> db_pool
    api_config --> common_config
```

## System Execution Flow

### Game Server Startup

```mermaid
graph TD
    start[Server Start]
    load_config[Load Configuration]
    init_db[Initialize Database]
    init_network[Initialize Network]
    init_ecs[Initialize ECS]
    init_systems[Initialize Systems]
    start_event_loop[Start Event Loop]
    
    start --> load_config
    load_config --> init_db
    init_db --> init_network
    init_network --> init_ecs
    init_ecs --> init_systems
    init_systems --> start_event_loop
```

### Packet Processing Flow

```mermaid
graph TD
    receive[Receive UDP Packet]
    decrypt[Decrypt Packet]
    decompress[Decompress Packet]
    deserialize[Deserialize Packet]
    route[Route to Handler]
    process[Process in System]
    update_ecs[Update ECS]
    serialize_response[Serialize Response]
    compress_response[Compress Response]
    encrypt_response[Encrypt Response]
    send[Send Response]
    
    receive --> decrypt
    decrypt --> decompress
    decompress --> deserialize
    deserialize --> route
    route --> process
    process --> update_ecs
    update_ecs --> serialize_response
    serialize_response --> compress_response
    compress_response --> encrypt_response
    encrypt_response --> send
```

### ECS System Execution

```mermaid
graph TD
    tick[Game Tick]
    movement[Movement System]
    combat[Combat System]
    ai[AI System]
    conditions[Conditions System]
    buffs[Buffs System]
    respawn[Respawn System]
    aoi_update[AOI Update]
    sync[Network Sync]
    
    tick --> movement
    movement --> combat
    combat --> ai
    ai --> conditions
    conditions --> buffs
    buffs --> respawn
    respawn --> aoi_update
    aoi_update --> sync
    sync --> tick
```

## Dependency Rules

### Build Order

1. **tos-common** - No dependencies on other project crates
2. **tos-protocol** - Depends only on tos-common
3. **tos-database** - Depends only on tos-common
4. **tos-network** - Depends on tos-protocol and tos-common
5. **tos-core** - Depends on tos-protocol, tos-network, tos-database, and tos-common
6. **tos-api** - Depends on tos-database and tos-common
7. **bin/server** - Depends on all game server crates
8. **bin/api** - Depends on tos-api, tos-database, and tos-common

### Circular Dependency Prevention

**Rules:**
- Lower-level crates cannot depend on higher-level crates
- `tos-common` has no dependencies on other project crates
- `tos-protocol` only depends on `tos-common`
- `tos-network` depends on `tos-protocol` but not `tos-core`
- `tos-core` can depend on all lower-level crates
- `tos-api` is independent of game server crates (only database)

**Forbidden Dependencies:**
- ❌ `tos-common` → any other crate
- ❌ `tos-protocol` → `tos-network` or `tos-core`
- ❌ `tos-network` → `tos-core`
- ❌ `tos-database` → `tos-core` or `tos-network`
- ❌ `tos-api` → `tos-core` or `tos-network`

## Module Import Guidelines

### tos-common
```rust
// Only external dependencies
use std::...;
use serde::...;
```

### tos-protocol
```rust
// Can import from tos-common
use tos_common::types::...;
use tos_common::errors::...;
```

### tos-network
```rust
// Can import from tos-protocol and tos-common
use tos_protocol::packets::...;
use tos_protocol::serialization::...;
use tos_common::errors::...;
```

### tos-core
```rust
// Can import from all lower-level crates
use tos_protocol::packets::...;
use tos_network::connection::...;
use tos_database::queries::...;
use tos_common::types::...;
```

### tos-api
```rust
// Can import from tos-database and tos-common
use tos_database::queries::...;
use tos_database::models::...;
use tos_common::errors::...;
```

## Testing Dependencies

```mermaid
graph TD
    test_common[tests/common<br/>Common test utilities]
    test_protocol[tests/protocol<br/>Protocol tests]
    test_network[tests/network<br/>Network tests]
    test_core[tests/core<br/>Core integration tests]
    test_api[tests/api<br/>API integration tests]
    
    test_protocol --> test_common
    test_network --> test_common
    test_network --> test_protocol
    test_core --> test_common
    test_core --> test_protocol
    test_core --> test_network
    test_api --> test_common
```

## See Also

- [STRUCTURE_RECOMMENDATIONS.md](../STRUCTURE_RECOMMENDATIONS.md) - Detailed structure recommendations
- [ARCHITECTURE.md](./ARCHITECTURE.md) - Complete architecture documentation
- [ROADMAP.md](./ROADMAP.md) - Implementation roadmap

