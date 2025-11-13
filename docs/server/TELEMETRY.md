# Server Telemetry System

## Overview

The server telemetry system provides comprehensive monitoring and metrics collection for the game server. It tracks hardware usage, network performance, server performance metrics, and game-specific statistics. All metrics are collected in real-time and can be exported in Prometheus format or sent to the API principal for centralized monitoring.

## Architecture

The telemetry system is integrated into the game server and collects metrics continuously. Metrics are aggregated over configurable time windows and can be accessed locally or sent to the API principal for dashboard visualization.

**Components:**
- **TelemetryCollector**: Main coordinator for metric collection
- **HardwareMonitor**: CPU, memory, disk I/O monitoring
- **NetworkMonitor**: Network traffic, latency, ping tracking
- **PerformanceMonitor**: Server tick rate, frame times, update cycles
- **MetricsExporter**: Prometheus format export
- **MetricsAggregator**: Statistical aggregation (percentiles, averages)

## Metrics Collected

### Hardware Metrics

**CPU Metrics:**
- Total CPU usage percentage
- Per-core CPU usage percentage
- CPU load average

**Memory Metrics:**
- Total memory (bytes)
- Used memory (bytes)
- Free memory (bytes)
- Memory usage percentage
- Memory pressure indicators

**Disk I/O Metrics:**
- Disk read bytes per second
- Disk write bytes per second
- Disk read operations per second
- Disk write operations per second

**Network Interface Metrics:**
- Bytes received per second
- Bytes sent per second
- Packets received per second
- Packets sent per second

### Network Metrics

**Packet Statistics:**
- Packets per second (incoming)
- Packets per second (outgoing)
- Bytes per second (incoming)
- Bytes per second (outgoing)
- Total packets processed
- Total bytes processed

**Latency Metrics:**
- Average packet latency (milliseconds)
- Packet latency percentiles (p50, p95, p99)
- Minimum packet latency
- Maximum packet latency

**Player Ping Metrics:**
- Average player ping (milliseconds)
- Player ping percentiles (p50, p95, p99)
- Minimum player ping
- Maximum player ping
- Per-player ping tracking

**Connection Metrics:**
- Active connections count
- Total connections count
- Connections per second (new)
- Disconnections per second
- Connection duration statistics

**Reliability Metrics:**
- Packet loss rate (percentage)
- Retransmission rate (percentage)
- Acknowledgment latency
- Fragmentation statistics

### Performance Metrics

**Tick Rate Metrics:**
- Current tick rate (FPS)
- Target tick rate (60 FPS)
- Tick rate deviation from target
- Tick rate stability (variance)

**Frame Time Metrics:**
- Average tick duration (milliseconds)
- Tick duration percentiles (p50, p95, p99)
- Minimum tick duration
- Maximum tick duration
- Frame time distribution

**Update Cycle Metrics:**
- Total update cycle duration
- System processing time
- Entity update time
- Network processing time
- Database query time
- AOI update time

**Performance Breakdown:**
- Time spent in ECS systems
- Time spent in network processing
- Time spent in serialization
- Time spent in compression
- Time spent in encryption

### Game Metrics

**Player Metrics:**
- Active player count
- Total players connected
- Players per map
- Average session duration

**Entity Metrics:**
- Total entities
- Entities per map
- Entities per type (Player, NPC, Monster, etc.)
- Entity creation rate
- Entity removal rate

**World Metrics:**
- Active maps count
- AOI update frequency
- Spatial partitioning statistics
- Collision detection statistics

**Database Metrics:**
- Average query time (milliseconds)
- Query count per second
- Cache hit rate (percentage)
- Cache miss rate (percentage)
- Database size (bytes)

**System Metrics:**
- ECS component count
- System execution time per system
- Query execution time
- Memory allocations (if any)

## Implementation

### Telemetry Module Structure

```
tos-core/src/telemetry/
├── mod.rs                 # Module exports
├── collector.rs           # Main metrics collector
├── metrics.rs             # Metric type definitions
├── hardware.rs            # Hardware monitoring
├── network.rs             # Network monitoring
├── performance.rs         # Performance monitoring
├── exporter.rs            # Prometheus exporter
└── aggregator.rs          # Metrics aggregation
```

### Metric Collection

Metrics are collected on a configurable interval (default: 1 second). The collector coordinates all monitoring subsystems and aggregates their results into a unified `ServerMetrics` structure.

**Collection Flow:**
1. HardwareMonitor collects CPU, memory, disk metrics
2. NetworkMonitor collects network and connection metrics
3. PerformanceMonitor collects tick rate and frame time metrics
4. Game metrics are collected from game systems
5. All metrics are aggregated into ServerMetrics
6. Metrics are stored in memory and optionally exported

### Metric Storage

Metrics are stored in memory with configurable retention:
- **Current metrics**: Latest collected metrics (always available)
- **Historical metrics**: Ring buffer of recent metrics (configurable size)
- **Aggregated metrics**: Statistical summaries (percentiles, averages)

### Export Formats

**Prometheus Format:**
Metrics can be exported in Prometheus text format for scraping by Prometheus or compatible monitoring systems.

**JSON Format:**
Metrics can be serialized to JSON for API transmission or logging.

**Binary Format:**
Metrics can be serialized to binary format for efficient storage or transmission.

## API Integration

### Sending Metrics to API Principal

The game server sends telemetry data to the API principal via HTTP POST:

**Endpoint:** `POST /api/v1/telemetry`

**Request:**
```json
{
  "server_id": "server-001",
  "timestamp": "2025-01-15T10:30:00Z",
  "hardware": {
    "cpu_usage_percent": 45.2,
    "cpu_usage_per_core": [42.1, 48.3, 44.5, 46.0],
    "memory_used_bytes": 2147483648,
    "memory_total_bytes": 8589934592,
    "memory_usage_percent": 25.0
  },
  "network": {
    "packets_per_sec_incoming": 15000,
    "packets_per_sec_outgoing": 12000,
    "avg_player_ping_ms": 45.3,
    "player_ping_percentiles": {
      "p50": 42.0,
      "p95": 58.0,
      "p99": 72.0
    },
    "active_connections": 1250
  },
  "performance": {
    "tick_rate_fps": 59.8,
    "target_fps": 60.0,
    "avg_tick_duration_ms": 16.72,
    "tick_duration_percentiles": {
      "p50": 16.5,
      "p95": 17.2,
      "p99": 18.5
    }
  },
  "game": {
    "active_player_count": 1250,
    "entities_per_map": {
      "map-001": 5000,
      "map-002": 3200
    }
  }
}
```

**Authentication:**
Requests must include a valid server token in the Authorization header:
```
Authorization: Bearer <server_token>
```

**Frequency:**
Metrics are sent to the API principal every 5 seconds by default (configurable).

## Local Access

### Prometheus Endpoint

The game server can expose a Prometheus metrics endpoint (optional):

**Endpoint:** `GET /metrics`

**Response:** Prometheus text format

**Example:**
```
# HELP server_cpu_usage_percent CPU usage percentage
# TYPE server_cpu_usage_percent gauge
server_cpu_usage_percent 45.2

# HELP server_memory_used_bytes Memory used in bytes
# TYPE server_memory_used_bytes gauge
server_memory_used_bytes 2147483648

# HELP server_packets_per_sec_incoming Incoming packets per second
# TYPE server_packets_per_sec_incoming gauge
server_packets_per_sec_incoming 15000

# HELP server_tick_rate_fps Server tick rate in FPS
# TYPE server_tick_rate_fps gauge
server_tick_rate_fps 59.8

# HELP server_avg_player_ping_ms Average player ping in milliseconds
# TYPE server_avg_player_ping_ms gauge
server_avg_player_ping_ms 45.3
```

### Programmatic Access

Metrics can be accessed programmatically from within the server:

```rust
use tos_core::telemetry::{TelemetryCollector, ServerMetrics};

// Get current metrics
let metrics: ServerMetrics = telemetry_collector.get_current_metrics();

// Access specific metrics
println!("CPU Usage: {}%", metrics.hardware.cpu_usage_percent);
println!("Tick Rate: {} FPS", metrics.performance.tick_rate_fps);
println!("Player Ping: {} ms", metrics.network.avg_player_ping_ms);
```

## Configuration

### Telemetry Configuration

```rust
pub struct TelemetryConfig {
    pub collection_interval: Duration,      // Default: 1 second
    pub api_send_interval: Duration,         // Default: 5 seconds
    pub historical_retention: usize,        // Default: 300 samples (5 minutes at 1s interval)
    pub enable_prometheus_endpoint: bool,   // Default: false
    pub prometheus_port: u16,               // Default: 9090
    pub api_endpoint: Option<String>,       // API principal endpoint
    pub server_token: Option<String>,       // Server authentication token
}
```

### Performance Considerations

**Zero-Allocation Design:**
- Metric collection uses pre-allocated buffers
- No allocations during metric collection
- Ring buffers for historical data
- Atomic counters for thread-safe updates

**Low Overhead:**
- Metrics collection runs in background task
- Minimal impact on game server performance
- Configurable collection frequency
- Efficient serialization

**Thread Safety:**
- All metric updates use atomic operations
- Lock-free data structures where possible
- RwLock for metric reading (read-heavy workload)

## Alerting

### Performance Thresholds

The telemetry system can trigger alerts when metrics exceed thresholds:

**CPU Usage:**
- Warning: > 80%
- Critical: > 95%

**Memory Usage:**
- Warning: > 85%
- Critical: > 95%

**Tick Rate:**
- Warning: < 58 FPS
- Critical: < 55 FPS

**Player Ping:**
- Warning: p95 > 100ms
- Critical: p95 > 200ms

**Packet Loss:**
- Warning: > 1%
- Critical: > 5%

## Integration with Dashboard

The API principal receives telemetry data and makes it available to the admin dashboard via:

1. **Real-Time Updates**: Server-Sent Events (SSE) for live metrics
2. **Historical Data**: Stored in PostgreSQL for charting
3. **Alerts**: Triggered when thresholds are exceeded
4. **Visualization**: Charts and graphs in the dashboard

See [DASHBOARD.md](../admin/DASHBOARD.md) for dashboard integration details.

## See Also

- [ARCHITECTURE.md](./ARCHITECTURE.md) - Server architecture overview
- [STATUS.md](./STATUS.md) - Implementation status
- [SERVER_MANAGEMENT.md](../admin/SERVER_MANAGEMENT.md) - Server management features
- [REAL_TIME_UPDATES.md](../admin/REAL_TIME_UPDATES.md) - Real-time updates system

