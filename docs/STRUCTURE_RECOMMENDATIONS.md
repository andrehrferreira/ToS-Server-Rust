# Structure Recommendations for Rust Implementation

## Quick Start: Recommended Structure

Based on industry standards and your requirements (10k players, 600k packets/sec), here's the recommended structure:

## 1. Workspace Structure

```
Server/
├── Cargo.toml                    # Workspace root
├── .gitignore
├── README.md
│
├── crates/                       # All library crates
│   ├── tos-core/                # Core game logic
│   ├── tos-network/             # Networking
│   ├── tos-protocol/            # Protocol definitions
│   ├── tos-database/            # Database layer (SQLite for game server)
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
└── docs/                         # Documentation (existing)
```

## 2. Core Crate Structure (tos-core)

```
tos-core/
├── Cargo.toml
└── src/
    ├── lib.rs
    │
    ├── ecs/                      # Custom ECS framework
    │   ├── mod.rs
    │   ├── entity.rs            # Entity ID management
    │   ├── component.rs         # Component storage (SoA)
    │   ├── system.rs            # System trait and scheduler
    │   ├── query.rs             # Query system
    │   └── world.rs             # ECS World
    │
    ├── components/              # Game components
    │   ├── mod.rs
    │   ├── transform.rs         # Position, rotation
    │   ├── health.rs            # Health, mana, stamina
    │   ├── stats.rs             # Character stats
    │   ├── combat.rs            # Combat-related
    │   ├── inventory.rs         # Inventory, equipment
    │   └── player.rs            # Player-specific
    │
    ├── systems/                 # Game systems
    │   ├── mod.rs
    │   ├── movement.rs          # Movement system
    │   ├── combat.rs            # Combat system
    │   ├── ai.rs                # AI system
    │   ├── conditions.rs        # Condition system
    │   ├── buffs.rs             # Buff/debuff system
    │   └── respawn.rs           # Respawn system
    │
    ├── world/                   # World management
    │   ├── mod.rs
    │   ├── map.rs               # Map management
    │   ├── aoi.rs               # Area of Interest
    │   └── spatial.rs           # Spatial partitioning
    │
    ├── combat/                  # Combat mechanics
    │   ├── mod.rs
    │   ├── damage.rs            # Damage calculation
    │   ├── skills.rs            # Skill system
    │   └── actions.rs           # Action system
    │
    ├── crafting/                # Crafting system
    │   ├── mod.rs
    │   ├── recipes.rs
    │   └── materials.rs
    │
    ├── player/                  # Player systems
    │   ├── mod.rs
    │   ├── progression.rs       # Level, XP
    │   ├── skills.rs            # Skill progression
    │   └── quests.rs            # Quest system
    │
    └── telemetry/               # Telemetry and monitoring
        ├── mod.rs
        ├── collector.rs         # Metrics collector
        ├── metrics.rs           # Metrics definitions
        ├── hardware.rs          # CPU, memory monitoring
        ├── network.rs           # Network metrics (latency, packets/sec)
        ├── performance.rs       # FPS, tick rate, update cycles
        ├── exporter.rs          # Metrics exporter (Prometheus format)
        └── aggregator.rs        # Metrics aggregation
```

## 3. Network Crate Structure (tos-network)

```
tos-network/
├── Cargo.toml
└── src/
    ├── lib.rs
    │
    ├── server.rs                # UDP server
    ├── connection.rs            # Connection management
    │
    ├── packet/                  # Packet handling
    │   ├── mod.rs
    │   ├── header.rs            # Packet header
    │   ├── deserializer.rs      # Deserialization
    │   ├── serializer.rs        # Serialization
    │   └── handler.rs           # Packet handlers
    │
    ├── reliable/                # Reliable UDP
    │   ├── mod.rs
    │   ├── ack.rs               # Acknowledgment
    │   ├── retransmit.rs        # Retransmission
    │   └── sequence.rs          # Sequence management
    │
    ├── encryption/              # Encryption layer
    │   ├── mod.rs
    │   ├── chacha20.rs          # ChaCha20-Poly1305
    │   ├── x25519.rs            # Key exchange
    │   └── replay.rs            # Replay protection
    │
    └── compression/              # Compression
        ├── mod.rs
        ├── lz4.rs               # LZ4 compression
        └── delta.rs             # Delta compression
```

## 4. Protocol Crate Structure (tos-protocol)

```
tos-protocol/
├── Cargo.toml
└── src/
    ├── lib.rs
    │
    ├── packets/                 # Packet definitions
    │   ├── mod.rs
    │   ├── server.rs            # Server->Client packets
    │   ├── client.rs            # Client->Server packets
    │   └── control.rs           # Control packets
    │
    ├── serialization/           # Serialization
    │   ├── mod.rs
    │   ├── bytebuffer.rs        # ByteBuffer implementation
    │   ├── varint.rs            # VarInt encoding
    │   ├── quantization.rs      # Quantization
    │   └── delta.rs             # Delta compression
    │
    └── types/                   # Protocol types
        ├── mod.rs
        ├── entity.rs            # Entity types
        ├── player.rs            # Player types
        └── world.rs             # World types
```

## 5. API Principal Structure (tos-api)

```
tos-api/
├── Cargo.toml
└── src/
    ├── lib.rs
    │
    ├── server.rs                # HTTP server (Axum/Warp)
    ├── config.rs                # API configuration
    │
    ├── routes/                  # API routes
    │   ├── mod.rs
    │   ├── auth.rs              # Authentication endpoints
    │   ├── config.rs            # Configuration API
    │   ├── admin/               # Admin endpoints
    │   │   ├── mod.rs
    │   │   ├── servers.rs       # Server management
    │   │   ├── players.rs       # Player management
    │   │   ├── content.rs       # Content management
    │   │   └── monitoring.rs    # Monitoring endpoints
    │   └── health.rs            # Health check endpoints
    │
    ├── handlers/                # Request handlers
    │   ├── mod.rs
    │   ├── auth.rs              # Auth handlers
    │   ├── config.rs            # Config handlers
    │   └── admin.rs             # Admin handlers
    │
    ├── services/               # Business logic
    │   ├── mod.rs
    │   ├── auth_service.rs      # Authentication service
    │   ├── config_service.rs    # Configuration service
    │   ├── player_service.rs    # Player management service
    │   ├── server_service.rs    # Server management service
    │   └── backup_service.rs    # Backup service
    │
    ├── database/                # Database layer (PostgreSQL)
    │   ├── mod.rs
    │   ├── pool.rs              # Connection pooling
    │   ├── migrations/          # Database migrations
    │   └── models/              # Database models
    │       ├── mod.rs
    │       ├── account.rs
    │       ├── player.rs
    │       ├── server.rs
    │       └── config.rs
    │
    ├── middleware/             # HTTP middleware
    │   ├── mod.rs
    │   ├── auth.rs              # Authentication middleware
    │   ├── cors.rs              # CORS handling
    │   ├── logging.rs           # Request logging
    │   └── rate_limit.rs        # Rate limiting
    │
    ├── sse/                    # Server-Sent Events
    │   ├── mod.rs
    │   ├── broadcaster.rs       # Event broadcasting
    │   └── events.rs            # Event types
    │
    └── backup/                 # Backup system
        ├── mod.rs
        ├── cloud_storage.rs     # S3/Spaces integration
        └── scheduler.rs         # Backup scheduling
```

## 6. Dashboard Structure (Frontend)

```
dashboard/
├── package.json
├── tsconfig.json
├── vite.config.ts              # Build tool (Vite)
│
├── public/                     # Static assets
│   ├── index.html
│   └── assets/
│
├── src/
│   ├── main.tsx                # Entry point
│   ├── App.tsx                 # Root component
│   │
│   ├── api/                    # API client
│   │   ├── client.ts           # HTTP client
│   │   ├── auth.ts             # Auth API
│   │   ├── servers.ts          # Server API
│   │   ├── players.ts          # Player API
│   │   └── content.ts          # Content API
│   │
│   ├── components/            # React components
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── Layout.tsx
│   │   ├── server/
│   │   │   ├── ServerList.tsx
│   │   │   ├── ServerCard.tsx
│   │   │   └── ServerMetrics.tsx
│   │   ├── player/
│   │   │   ├── PlayerList.tsx
│   │   │   ├── PlayerDetails.tsx
│   │   │   └── PlayerInventory.tsx
│   │   ├── content/
│   │   │   ├── ItemEditor.tsx
│   │   │   ├── NPCEditor.tsx
│   │   │   └── QuestEditor.tsx
│   │   └── map/
│   │       └── InteractiveMap.tsx
│   │
│   ├── pages/                 # Page components
│   │   ├── Login.tsx
│   │   ├── Dashboard.tsx
│   │   ├── Servers.tsx
│   │   ├── Players.tsx
│   │   ├── Content.tsx
│   │   └── Map.tsx
│   │
│   ├── hooks/                # React hooks
│   │   ├── useAuth.ts
│   │   ├── useServers.ts
│   │   ├── useSSE.ts          # Server-Sent Events
│   │   └── useWebSocket.ts
│   │
│   ├── store/                # State management (Zustand/Redux)
│   │   ├── auth.ts
│   │   ├── servers.ts
│   │   └── players.ts
│   │
│   ├── types/                # TypeScript types
│   │   ├── api.ts
│   │   ├── server.ts
│   │   └── player.ts
│   │
│   └── utils/                # Utilities
│       ├── format.ts
│       └── validation.ts
```

**Dashboard Tech Stack:**
- **Framework**: React + TypeScript
- **Build Tool**: Vite
- **State Management**: Zustand or Redux Toolkit
- **HTTP Client**: Axios or fetch
- **UI Components**: Material-UI, Ant Design, or Tailwind CSS
- **Charts**: Recharts or Chart.js
- **Maps**: Leaflet or Mapbox for interactive map

## 7. Telemetry System Structure

### Telemetry Module (tos-core/src/telemetry/)

```
telemetry/
├── mod.rs
├── collector.rs              # Main metrics collector
├── metrics.rs                # Metrics definitions and types
├── hardware.rs               # Hardware metrics (CPU, memory)
├── network.rs                # Network metrics (latency, packets/sec)
├── performance.rs            # Performance metrics (FPS, tick rate)
├── exporter.rs               # Prometheus format exporter
└── aggregator.rs             # Metrics aggregation and statistics
```

### Metrics Collected

**Hardware Metrics:**
- CPU usage percentage (per core and total)
- Memory usage (used, free, total)
- Memory pressure indicators
- Disk I/O (read/write rates)
- Network interface statistics

**Network Metrics:**
- Packets per second (incoming/outgoing)
- Bytes per second (incoming/outgoing)
- Average packet latency
- Average player ping (per player and overall)
- Connection count (active, total)
- Packet loss rate
- Retransmission rate (reliable UDP)

**Performance Metrics:**
- Server tick rate (FPS) - target 60 FPS
- Average tick duration (milliseconds)
- Frame time percentiles (p50, p95, p99)
- Update cycle duration
- System processing time
- Entity update time
- Network processing time

**Game Metrics:**
- Active player count
- Entities per map
- AOI update frequency
- Database query performance
- Cache hit rates

### Implementation Details

```rust
// tos-core/src/telemetry/metrics.rs
#[derive(Debug, Clone, Serialize)]
pub struct ServerMetrics {
    pub timestamp: DateTime<Utc>,
    
    // Hardware
    pub hardware: HardwareMetrics,
    
    // Network
    pub network: NetworkMetrics,
    
    // Performance
    pub performance: PerformanceMetrics,
    
    // Game
    pub game: GameMetrics,
}

#[derive(Debug, Clone, Serialize)]
pub struct HardwareMetrics {
    pub cpu_usage_percent: f64,           // Total CPU usage
    pub cpu_usage_per_core: Vec<f64>,     // Per-core usage
    pub memory_used_bytes: u64,
    pub memory_total_bytes: u64,
    pub memory_usage_percent: f64,
    pub disk_read_bytes_per_sec: u64,
    pub disk_write_bytes_per_sec: u64,
}

#[derive(Debug, Clone, Serialize)]
pub struct NetworkMetrics {
    pub packets_per_sec_incoming: u64,
    pub packets_per_sec_outgoing: u64,
    pub bytes_per_sec_incoming: u64,
    pub bytes_per_sec_outgoing: u64,
    pub avg_packet_latency_ms: f64,
    pub avg_player_ping_ms: f64,
    pub player_ping_percentiles: PingPercentiles,
    pub active_connections: usize,
    pub total_connections: usize,
    pub packet_loss_rate: f64,
    pub retransmission_rate: f64,
}

#[derive(Debug, Clone, Serialize)]
pub struct PingPercentiles {
    pub p50: f64,
    pub p95: f64,
    pub p99: f64,
}

#[derive(Debug, Clone, Serialize)]
pub struct PerformanceMetrics {
    pub tick_rate_fps: f64,               // Current FPS
    pub target_fps: f64,                   // Target FPS (60.0)
    pub avg_tick_duration_ms: f64,
    pub tick_duration_percentiles: TickPercentiles,
    pub update_cycle_duration_ms: f64,
    pub system_processing_time_ms: f64,
    pub entity_update_time_ms: f64,
    pub network_processing_time_ms: f64,
}

#[derive(Debug, Clone, Serialize)]
pub struct TickPercentiles {
    pub p50: f64,
    pub p95: f64,
    pub p99: f64,
}

#[derive(Debug, Clone, Serialize)]
pub struct GameMetrics {
    pub active_player_count: usize,
    pub entities_per_map: HashMap<MapId, usize>,
    pub aoi_update_frequency: f64,
    pub db_query_avg_time_ms: f64,
    pub cache_hit_rate: f64,
}
```

### Telemetry Collector

```rust
// tos-core/src/telemetry/collector.rs
pub struct TelemetryCollector {
    metrics: Arc<RwLock<ServerMetrics>>,
    hardware_monitor: HardwareMonitor,
    network_monitor: NetworkMonitor,
    performance_monitor: PerformanceMonitor,
    aggregation_window: Duration,
}

impl TelemetryCollector {
    pub async fn collect_metrics(&self) -> ServerMetrics {
        // Collect all metrics
        let hardware = self.hardware_monitor.collect().await;
        let network = self.network_monitor.collect().await;
        let performance = self.performance_monitor.collect().await;
        let game = self.collect_game_metrics().await;
        
        ServerMetrics {
            timestamp: Utc::now(),
            hardware,
            network,
            performance,
            game,
        }
    }
    
    pub fn get_current_metrics(&self) -> ServerMetrics {
        self.metrics.read().unwrap().clone()
    }
    
    pub async fn start_collection_loop(&self) {
        let mut interval = tokio::time::interval(self.aggregation_window);
        loop {
            interval.tick().await;
            let metrics = self.collect_metrics().await;
            *self.metrics.write().unwrap() = metrics;
        }
    }
}
```

### Hardware Monitoring

```rust
// tos-core/src/telemetry/hardware.rs
pub struct HardwareMonitor {
    sys: System,
}

impl HardwareMonitor {
    pub async fn collect(&self) -> HardwareMetrics {
        // CPU usage
        let cpu_usage = self.sys.cpu_usage();
        let cpu_per_core = self.sys.cpu_usage_per_core();
        
        // Memory usage
        let memory = self.sys.memory();
        
        // Disk I/O
        let disk = self.sys.disk_usage();
        
        HardwareMetrics {
            cpu_usage_percent: cpu_usage,
            cpu_usage_per_core: cpu_per_core,
            memory_used_bytes: memory.used,
            memory_total_bytes: memory.total,
            memory_usage_percent: (memory.used as f64 / memory.total as f64) * 100.0,
            disk_read_bytes_per_sec: disk.read_bytes_per_sec,
            disk_write_bytes_per_sec: disk.write_bytes_per_sec,
        }
    }
}
```

### Network Monitoring

```rust
// tos-core/src/telemetry/network.rs
pub struct NetworkMonitor {
    packet_counter: Arc<AtomicU64>,
    byte_counter: Arc<AtomicU64>,
    latency_tracker: LatencyTracker,
    ping_tracker: PingTracker,
    connection_manager: Arc<ConnectionManager>,
}

impl NetworkMonitor {
    pub async fn collect(&self) -> NetworkMetrics {
        // Calculate packets per second (from counter reset)
        let packets_in = self.packet_counter.swap(0, Ordering::Relaxed);
        let bytes_in = self.byte_counter.swap(0, Ordering::Relaxed);
        
        // Get latency statistics
        let avg_latency = self.latency_tracker.average();
        
        // Get ping statistics
        let ping_stats = self.ping_tracker.get_statistics();
        
        // Connection counts
        let (active, total) = self.connection_manager.get_counts();
        
        NetworkMetrics {
            packets_per_sec_incoming: packets_in,
            packets_per_sec_outgoing: 0, // Track separately
            bytes_per_sec_incoming: bytes_in,
            bytes_per_sec_outgoing: 0,
            avg_packet_latency_ms: avg_latency,
            avg_player_ping_ms: ping_stats.average,
            player_ping_percentiles: ping_stats.percentiles,
            active_connections: active,
            total_connections: total,
            packet_loss_rate: self.calculate_packet_loss(),
            retransmission_rate: self.calculate_retransmission_rate(),
        }
    }
}
```

### Performance Monitoring

```rust
// tos-core/src/telemetry/performance.rs
pub struct PerformanceMonitor {
    tick_tracker: TickTracker,
    frame_time_tracker: FrameTimeTracker,
}

impl PerformanceMonitor {
    pub async fn collect(&self) -> PerformanceMetrics {
        let tick_stats = self.tick_tracker.get_statistics();
        let frame_stats = self.frame_time_tracker.get_statistics();
        
        PerformanceMetrics {
            tick_rate_fps: tick_stats.current_fps,
            target_fps: 60.0,
            avg_tick_duration_ms: tick_stats.average_duration_ms,
            tick_duration_percentiles: tick_stats.percentiles,
            update_cycle_duration_ms: frame_stats.update_cycle_ms,
            system_processing_time_ms: frame_stats.system_time_ms,
            entity_update_time_ms: frame_stats.entity_time_ms,
            network_processing_time_ms: frame_stats.network_time_ms,
        }
    }
}

pub struct TickTracker {
    frame_times: VecDeque<Duration>,
    window_size: usize,
}

impl TickTracker {
    pub fn record_tick(&mut self, duration: Duration) {
        self.frame_times.push_back(duration);
        if self.frame_times.len() > self.window_size {
            self.frame_times.pop_front();
        }
    }
    
    pub fn get_statistics(&self) -> TickStatistics {
        let durations: Vec<f64> = self.frame_times
            .iter()
            .map(|d| d.as_secs_f64() * 1000.0)
            .collect();
        
        let avg = durations.iter().sum::<f64>() / durations.len() as f64;
        let fps = 1000.0 / avg;
        
        let mut sorted = durations.clone();
        sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());
        
        TickStatistics {
            current_fps: fps,
            average_duration_ms: avg,
            percentiles: TickPercentiles {
                p50: percentile(&sorted, 0.50),
                p95: percentile(&sorted, 0.95),
                p99: percentile(&sorted, 0.99),
            },
        }
    }
}
```

### Metrics Exporter (Prometheus Format)

```rust
// tos-core/src/telemetry/exporter.rs
pub struct MetricsExporter;

impl MetricsExporter {
    pub fn export_prometheus_format(metrics: &ServerMetrics) -> String {
        format!(
            "# HELP server_cpu_usage_percent CPU usage percentage\n\
             # TYPE server_cpu_usage_percent gauge\n\
             server_cpu_usage_percent {}\n\
             \n\
             # HELP server_memory_used_bytes Memory used in bytes\n\
             # TYPE server_memory_used_bytes gauge\n\
             server_memory_used_bytes {}\n\
             \n\
             # HELP server_packets_per_sec_incoming Incoming packets per second\n\
             # TYPE server_packets_per_sec_incoming gauge\n\
             server_packets_per_sec_incoming {}\n\
             \n\
             # HELP server_tick_rate_fps Server tick rate in FPS\n\
             # TYPE server_tick_rate_fps gauge\n\
             server_tick_rate_fps {}\n\
             \n\
             # HELP server_avg_player_ping_ms Average player ping in milliseconds\n\
             # TYPE server_avg_player_ping_ms gauge\n\
             server_avg_player_ping_ms {}\n",
            metrics.hardware.cpu_usage_percent,
            metrics.hardware.memory_used_bytes,
            metrics.network.packets_per_sec_incoming,
            metrics.performance.tick_rate_fps,
            metrics.network.avg_player_ping_ms
        )
    }
}
```

### Integration with API Principal

The game server sends telemetry data to the API principal via HTTP POST:

```rust
// In game server binary
async fn send_telemetry_to_api(metrics: ServerMetrics) -> Result<()> {
    let client = reqwest::Client::new();
    let response = client
        .post("http://api-principal:8080/api/v1/telemetry")
        .header("Authorization", format!("Bearer {}", SERVER_TOKEN))
        .json(&metrics)
        .send()
        .await?;
    Ok(())
}
```

## 8. Database Structure

### Game Server Database (SQLite)

```
tos-database/
├── Cargo.toml
└── src/
    ├── lib.rs
    │
    ├── sqlite/                 # SQLite implementation
    │   ├── mod.rs
    │   ├── pool.rs             # Connection pooling
    │   ├── migrations/         # SQLite migrations
    │   └── models/             # Game data models
    │       ├── mod.rs
    │       ├── player.rs       # Player data
    │       ├── character.rs    # Character data
    │       ├── inventory.rs    # Inventory data
    │       └── world.rs        # World state
```

### API Principal Database (PostgreSQL)

```
tos-api/src/database/
├── migrations/                 # PostgreSQL migrations
│   ├── 001_initial_schema.sql
│   ├── 002_accounts.sql
│   ├── 003_servers.sql
│   └── 004_content.sql
│
└── models/                     # PostgreSQL models
    ├── account.rs              # Account management
    ├── server.rs               # Server registry
    ├── player.rs               # Player data (sync from game servers)
    └── config.rs               # Configuration data
```

## 9. Key Design Decisions

### ECS: Custom vs Framework

**Decision: Custom ECS Framework (DEFINITIVE)**

**Rationale:**
- Zero-allocation requirement (critical for 600k packets/sec)
- Cache-friendly SoA (Structure of Arrays) layout for maximum performance
- Lock-free design for concurrent access
- Optimized specifically for server workloads (not game engine)
- Minimal dependencies (no external ECS framework)
- Full control over memory layout and access patterns
- Custom query system optimized for game server patterns

**Framework Comparison (All Rejected):**
- ❌ Bevy: Too heavy, game engine focused, not suitable for server
- ❌ Flecs: C-based, external dependency, adds complexity
- ❌ Legion: Good but may have overhead, not optimized for our use case
- ❌ Specs: Rust ECS but may not meet zero-allocation requirements
- ✅ Custom: Full control, optimized for our specific needs, zero-allocation guarantee

**Implementation Approach:**
- Build from scratch in `tos-core/src/ecs/`
- SoA component storage for cache efficiency
- Entity ID system with generational indices
- System scheduler optimized for server tick rate (60 FPS)
- Query system with compile-time optimizations

### Networking: Tokio vs Mio

**Decision: Tokio for Phase 1**

**Rationale:**
- Industry standard
- Excellent async support
- Rich ecosystem
- Good performance
- Can optimize later with Mio if needed

### Serialization: Custom vs bincode

**Decision: Custom ByteBuffer**

**Rationale:**
- Zero-allocation requirement
- Quantization support
- Delta compression
- Protocol-specific optimizations
- Legacy code reference available

### Database: sqlx vs diesel

**Decision: sqlx**

**Rationale:**
- Async/await support
- Type-safe queries
- Compile-time verification
- Good performance
- Active development

### Architecture: Game Server vs API Principal

**Decision: Separate Services**

**Game Server (UDP):**
- High-performance UDP server for game client communication
- SQLite database for local game data persistence
- Zero-allocation design for maximum performance
- Handles 10,000 concurrent players, 600k packets/sec

**API Principal (HTTP/REST):**
- HTTP/REST API for admin dashboard and web services
- PostgreSQL database for centralized data management
- Server replication for high availability
- Handles admin operations, configuration, monitoring

**Rationale:**
- Separation of concerns (game logic vs admin operations)
- Different performance requirements (UDP vs HTTP)
- Different database needs (SQLite embedded vs PostgreSQL centralized)
- Independent scaling and deployment
- Game server can run standalone, API provides management layer

## 10. Dependencies Strategy

### Core Dependencies

```toml
# Cargo.toml (workspace)
[workspace]
members = ["crates/*", "bin/*"]
resolver = "2"

[workspace.dependencies]
# Async runtime
tokio = { version = "1", features = ["full"] }

# Serialization
serde = { version = "1", features = ["derive"] }
bincode = "1.3"

# Compression
lz4 = "1.24"

# Cryptography
chacha20poly1305 = "0.10"
x25519-dalek = "2.0"
sha2 = "0.10"
hmac = "0.12"

# Database
sqlx = { version = "0.7", features = ["sqlite", "postgres", "runtime-tokio"] }
# Note: sqlite for game server, postgres for API principal

# HTTP Server (API Principal)
axum = "0.7"                      # Web framework
tower = "0.4"                     # Middleware
tower-http = { version = "0.5", features = ["cors", "compression-gzip"] }
serde_json = "1.0"                # JSON serialization

# Cloud Storage (Backup)
aws-sdk-s3 = "1.0"                # S3 support (optional)
object_store = "0.7"              # Generic object storage

# Utilities
dashmap = "5.5"
parking_lot = "0.12"
crossbeam = "0.8"
rayon = "1.8"

# Logging
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

# Telemetry and Monitoring
sysinfo = "0.30"                  # System information (CPU, memory)
prometheus = "0.13"               # Prometheus metrics format
reqwest = { version = "0.11", features = ["json"] }  # HTTP client for telemetry

# Error handling
thiserror = "1.0"
anyhow = "1.0"

# Configuration
config = "0.13"
```

### Development Dependencies

```toml
[workspace.dev-dependencies]
# Testing
criterion = "0.5"
proptest = "1.4"
mockall = "0.12"

# Linting
clippy = { version = "1", features = ["cargo"] }
```

## 11. Code Organization Patterns

### Pattern 1: Module Organization

**Rule**: One file per logical unit
```
components/
├── mod.rs          # Re-exports
├── transform.rs    # Transform component
└── health.rs       # Health component
```

### Pattern 2: Error Handling

**Rule**: Custom error types per module
```rust
// tos-core/src/combat/error.rs
#[derive(Error, Debug)]
pub enum CombatError {
    #[error("Invalid target")]
    InvalidTarget,
    #[error("Out of range")]
    OutOfRange,
}
```

### Pattern 3: Configuration

**Rule**: Structured config with validation
```rust
// tos-core/src/config.rs
#[derive(Debug, Deserialize)]
pub struct ServerConfig {
    pub network: NetworkConfig,
    pub world: WorldConfig,
    pub combat: CombatConfig,
}
```

## 12. Testing Structure

```
crates/tos-core/
├── src/
└── tests/
    ├── integration/
    │   ├── combat_test.rs
    │   └── movement_test.rs
    └── unit/
        └── damage_test.rs
```

**Pattern:**
- Unit tests: `#[cfg(test)]` in same file
- Integration tests: `tests/` directory
- Benchmarks: `benches/` directory

## 13. Documentation Standards

### Code Documentation

```rust
/// Calculates damage based on base damage and multiplier.
///
/// # Arguments
///
/// * `base` - Base damage value
/// * `multiplier` - Damage multiplier (1.0 = 100%)
///
/// # Returns
///
/// Calculated damage value
///
/// # Examples
///
/// ```
/// use tos_core::combat::calculate_damage;
/// let damage = calculate_damage(100, 1.5);
/// assert_eq!(damage, 150);
/// ```
pub fn calculate_damage(base: u32, multiplier: f32) -> u32 {
    (base as f32 * multiplier) as u32
}
```

## 14. CI/CD Structure

```
.github/
└── workflows/
    ├── ci.yml              # Continuous Integration
    ├── benchmark.yml       # Performance benchmarks
    └── release.yml         # Release pipeline
```

## Implementation Steps

### Step 1: Create Workspace (Day 1)

```bash
cd Server
cargo init --name tos-server
# Edit Cargo.toml to create workspace
# Create crate structure
```

### Step 2: Set Up Core Crates (Week 1)

```bash
cd crates
cargo new --lib tos-core
cargo new --lib tos-network
cargo new --lib tos-protocol
cargo new --lib tos-database
cargo new --lib tos-api
cargo new --lib tos-common
```

### Step 2.1: Set Up API Principal (Week 1)

```bash
cd bin
cargo new --bin api
# Configure tos-api crate dependencies
```

### Step 2.2: Set Up Dashboard (Week 1)

```bash
npm create vite@latest dashboard -- --template react-ts
cd dashboard
npm install
# Install additional dependencies: axios, zustand, recharts, leaflet
```

### Step 3: Implement Custom ECS Framework (Week 2)

**Custom ECS Framework Implementation:**
- Entity ID system with generational indices
- Component storage (SoA layout for cache efficiency)
- System scheduler (optimized for 60 FPS tick rate)
- Query system (compile-time optimizations)
- World management (entity lifecycle, component access)
- Zero-allocation guarantees throughout
- Lock-free data structures for concurrent access

### Step 4: Network Layer (Week 3)

- UDP server
- Connection management
- Packet processing

## See Also

- [STRUCTURE_ANALYSIS.md](./STRUCTURE_ANALYSIS.md) - Detailed analysis
- [ROADMAP.md](./server/ROADMAP.md) - Implementation roadmap
- [ARCHITECTURE.md](./server/ARCHITECTURE.md) - Architecture details

