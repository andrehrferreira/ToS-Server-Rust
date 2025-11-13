# Lag Compensation System Documentation

## Overview

The lag compensation system handles network latency between clients and the game server, ensuring smooth gameplay while maintaining security. When players have high latency, there can be a disparity between the client-reported position and the server's recorded position. The system calculates lag compensation to reconcile these differences, preventing false positives in speed hack detection while ensuring responsive player movement.

## Key Features

- **Non-Blocking Movement**: Client movement is never blocked by server validation to prevent input lag
- **Lag Compensation Calculation**: Calculates position compensation based on client latency
- **Server Validation**: Server validates and double-checks positions after client movement
- **Speed Hack Protection**: Prevents false positives in speed hack detection due to latency
- **Smooth Gameplay**: Ensures fluid gameplay experience regardless of network conditions
- **Client-Server Reconciliation**: Reconciles differences between client and server positions

## Problem Statement

### High Latency Issues

When players have high latency:
- **Client Position**: Client reports position based on local movement
- **Server Position**: Server has delayed position updates
- **Disparity**: Difference between client and server positions increases with latency
- **False Positives**: High latency can trigger false speed hack detections
- **Input Lag**: Blocking client movement causes noticeable input lag

### Movement Flow

1. **Client Movement**: Player moves locally (immediate response)
2. **Network Delay**: Position update sent to server (latency delay)
3. **Server Receives**: Server receives position update after delay
4. **Validation**: Server validates position against last known position
5. **Discrepancy**: Large time gap can cause large position differences
6. **False Detection**: System may incorrectly detect speed hack

## Lag Compensation Architecture

### Client-Side Movement

**Non-Blocking Design:**
- Client movement is **never blocked** by server validation
- Player sees immediate response to input
- Movement feels responsive and fluid
- Server validation happens asynchronously

**Client Movement Flow:**
1. Player provides input (keyboard/mouse)
2. Client immediately moves player locally
3. Client sends position update to server
4. Client continues moving based on input
5. Server responds with validation/correction

### Server-Side Validation

**Asynchronous Validation:**
- Server receives position updates asynchronously
- Validates position against last known position
- Calculates lag compensation
- Applies corrections if needed
- Never blocks client movement

**Server Validation Flow:**
1. Server receives position update
2. Calculate time delta since last update
3. Calculate lag compensation
4. Validate position with compensation
5. Apply correction if position is invalid
6. Send correction to client if needed

## Lag Compensation Calculation

### Basic Compensation Formula

```rust
pub fn calculate_lag_compensation(
    client_position: Position,
    server_position: Position,
    client_latency: Duration,
    time_since_last_update: Duration,
) -> Position {
    // Calculate expected movement based on latency
    let latency_seconds = client_latency.as_secs_f32();
    let time_delta = time_since_last_update.as_secs_f32();
    
    // Calculate velocity from last known position
    let velocity = calculate_velocity(server_position, client_position, time_delta);
    
    // Compensate for latency by projecting position forward
    let compensated_position = project_position(
        server_position,
        velocity,
        latency_seconds,
    );
    
    compensated_position
}
```

### Velocity Calculation

```rust
pub fn calculate_velocity(
    from: Position,
    to: Position,
    time_delta: f32,
) -> Vector3 {
    if time_delta <= 0.0 {
        return Vector3::ZERO;
    }
    
    let distance = to - from;
    distance / time_delta
}

pub fn project_position(
    position: Position,
    velocity: Vector3,
    time: f32,
) -> Position {
    position + (velocity * time)
}
```

### Maximum Movement Distance

```rust
pub fn validate_movement_with_compensation(
    client_position: Position,
    server_position: Position,
    client_latency: Duration,
    time_since_last_update: Duration,
    max_speed: f32,
) -> Result<Position, MovementError> {
    // Calculate lag compensation
    let compensated_position = calculate_lag_compensation(
        client_position,
        server_position,
        client_latency,
        time_since_last_update,
    );
    
    // Calculate actual distance moved
    let distance_moved = server_position.distance_to(client_position);
    
    // Calculate maximum allowed distance (with compensation)
    let max_distance = max_speed * time_since_last_update.as_secs_f32();
    let compensation_distance = max_speed * client_latency.as_secs_f32();
    let max_allowed_distance = max_distance + compensation_distance;
    
    // Validate movement
    if distance_moved > max_allowed_distance {
        // Movement exceeds maximum (even with compensation)
        return Err(MovementError::SpeedViolation {
            distance_moved,
            max_allowed_distance,
            client_latency: client_latency.as_secs_f32(),
        });
    }
    
    // Movement is valid, return compensated position
    Ok(compensated_position)
}
```

## Implementation

### Lag Compensation System

```rust
pub struct LagCompensationSystem {
    player_positions: Arc<RwLock<HashMap<CharacterId, PlayerPositionState>>>,
    player_latencies: Arc<RwLock<HashMap<CharacterId, LatencyTracker>>>,
    max_speed: f32,
    max_latency_compensation: Duration, // Maximum latency to compensate for
}

pub struct PlayerPositionState {
    pub last_server_position: Position,
    pub last_client_position: Position,
    pub last_update_time: DateTime<Utc>,
    pub velocity: Vector3,
}

pub struct LatencyTracker {
    pub current_latency: Duration,
    pub average_latency: Duration,
    pub latency_samples: Vec<Duration>,
    pub last_ping_time: DateTime<Utc>,
}

impl LagCompensationSystem {
    pub async fn validate_movement(
        &self,
        player_id: CharacterId,
        client_position: Position,
        timestamp: DateTime<Utc>,
    ) -> Result<Position, MovementError> {
        // Get player state
        let mut positions = self.player_positions.write().await;
        let player_state = positions.entry(player_id)
            .or_insert_with(|| PlayerPositionState {
                last_server_position: client_position,
                last_client_position: client_position,
                last_update_time: timestamp,
                velocity: Vector3::ZERO,
            });
        
        // Get player latency
        let latencies = self.player_latencies.read().await;
        let latency_tracker = latencies.get(&player_id)
            .ok_or_else(|| MovementError::NoLatencyData)?;
        
        let client_latency = latency_tracker.average_latency;
        
        // Limit compensation to maximum
        let compensation_latency = client_latency.min(self.max_latency_compensation);
        
        // Calculate time since last update
        let time_since_last_update = timestamp.signed_duration_since(player_state.last_update_time);
        
        // Validate movement with lag compensation
        let validated_position = self.validate_movement_with_compensation(
            client_position,
            player_state.last_server_position,
            compensation_latency,
            time_since_last_update,
            self.max_speed,
        )?;
        
        // Update player state
        let new_velocity = calculate_velocity(
            player_state.last_server_position,
            validated_position,
            time_since_last_update.as_secs_f32(),
        );
        
        player_state.last_server_position = validated_position;
        player_state.last_client_position = client_position;
        player_state.last_update_time = timestamp;
        player_state.velocity = new_velocity;
        
        Ok(validated_position)
    }
    
    fn validate_movement_with_compensation(
        &self,
        client_position: Position,
        server_position: Position,
        client_latency: Duration,
        time_since_last_update: Duration,
        max_speed: f32,
    ) -> Result<Position, MovementError> {
        // Calculate distance moved
        let distance_moved = server_position.distance_to(client_position);
        
        // Calculate maximum allowed distance
        let time_delta = time_since_last_update.as_secs_f32();
        let max_distance = max_speed * time_delta;
        
        // Add compensation for latency
        let latency_seconds = client_latency.as_secs_f32();
        let compensation_distance = max_speed * latency_seconds;
        let max_allowed_distance = max_distance + compensation_distance;
        
        // Validate movement
        if distance_moved > max_allowed_distance {
            return Err(MovementError::SpeedViolation {
                distance_moved,
                max_allowed_distance,
                client_latency: latency_seconds,
            });
        }
        
        // Movement is valid
        Ok(client_position)
    }
    
    pub async fn update_latency(
        &self,
        player_id: CharacterId,
        ping_time: DateTime<Utc>,
        pong_time: DateTime<Utc>,
    ) {
        let latency = pong_time.signed_duration_since(ping_time);
        
        let mut latencies = self.player_latencies.write().await;
        let tracker = latencies.entry(player_id)
            .or_insert_with(|| LatencyTracker {
                current_latency: latency,
                average_latency: latency,
                latency_samples: Vec::new(),
                last_ping_time: ping_time,
            });
        
        // Add latency sample
        tracker.latency_samples.push(latency);
        
        // Keep only last 100 samples
        if tracker.latency_samples.len() > 100 {
            tracker.latency_samples.remove(0);
        }
        
        // Calculate average latency
        let sum: Duration = tracker.latency_samples.iter().sum();
        let count = tracker.latency_samples.len() as u32;
        tracker.average_latency = sum / count;
        tracker.current_latency = latency;
        tracker.last_ping_time = ping_time;
    }
}
```

### Integration with Speed Hack Detection

```rust
pub struct SpeedHackDetector {
    lag_compensation: Arc<LagCompensationSystem>,
    max_speed: f32,
}

impl SpeedHackDetector {
    pub async fn validate_movement(
        &self,
        player_id: CharacterId,
        client_position: Position,
        timestamp: DateTime<Utc>,
    ) -> Result<(), SpeedHackViolation> {
        // First, validate with lag compensation
        match self.lag_compensation.validate_movement(
            player_id,
            client_position,
            timestamp,
        ).await {
            Ok(validated_position) => {
                // Movement is valid with lag compensation
                Ok(())
            }
            Err(MovementError::SpeedViolation { distance_moved, max_allowed_distance, .. }) => {
                // Movement exceeds maximum even with compensation
                // This is a real speed hack violation
                let violation = SpeedHackViolation {
                    player_id,
                    position: client_position,
                    distance_moved,
                    max_allowed_distance,
                    timestamp: Utc::now(),
                };
                
                self.handle_speed_hack(violation).await;
                Err(violation)
            }
            Err(e) => {
                // Other errors
                Err(SpeedHackViolation {
                    player_id,
                    position: client_position,
                    distance_moved: 0.0,
                    max_allowed_distance: 0.0,
                    timestamp: Utc::now(),
                })
            }
        }
    }
}
```

## Latency Tracking

### Ping-Pong System

```rust
pub struct PingPongSystem {
    pending_pings: Arc<RwLock<HashMap<CharacterId, DateTime<Utc>>>>,
    lag_compensation: Arc<LagCompensationSystem>,
}

impl PingPongSystem {
    pub async fn send_ping(&self, player_id: CharacterId) {
        let ping_time = Utc::now();
        
        let mut pings = self.pending_pings.write().await;
        pings.insert(player_id, ping_time);
        
        // Send ping packet to client
        send_ping_packet(player_id, ping_time).await;
    }
    
    pub async fn handle_pong(&self, player_id: CharacterId, ping_time: DateTime<Utc>) {
        let pong_time = Utc::now();
        
        // Calculate latency
        let latency = pong_time.signed_duration_since(ping_time);
        
        // Update lag compensation system
        self.lag_compensation.update_latency(
            player_id,
            ping_time,
            pong_time,
        ).await;
        
        // Remove from pending pings
        let mut pings = self.pending_pings.write().await;
        pings.remove(&player_id);
    }
}
```

### Latency Smoothing

```rust
impl LatencyTracker {
    pub fn calculate_smoothed_latency(&self) -> Duration {
        // Use exponential moving average for smoother latency
        let alpha = 0.1; // Smoothing factor
        
        if self.latency_samples.is_empty() {
            return self.current_latency;
        }
        
        // Calculate weighted average (recent samples weighted more)
        let mut weighted_sum = Duration::ZERO;
        let mut weight_sum = 0.0;
        
        for (i, &latency) in self.latency_samples.iter().enumerate() {
            let weight = (1.0 - alpha).powi(i as i32);
            weighted_sum = weighted_sum + (latency * weight as u32);
            weight_sum += weight;
        }
        
        if weight_sum > 0.0 {
            weighted_sum / weight_sum as u32
        } else {
            self.current_latency
        }
    }
}
```

## Client-Server Reconciliation

### Client Prediction

**Client-Side:**
- Client predicts movement locally
- Sends position updates to server
- Receives server corrections
- Smoothly interpolates between predicted and corrected positions

**Server-Side:**
- Receives client position updates
- Validates with lag compensation
- Sends corrections if needed
- Never blocks client movement

### Reconciliation Process

```rust
pub struct ReconciliationSystem {
    lag_compensation: Arc<LagCompensationSystem>,
    correction_buffer: Arc<RwLock<HashMap<CharacterId, Vec<PositionCorrection>>>>,
}

pub struct PositionCorrection {
    pub server_position: Position,
    pub client_position: Position,
    pub timestamp: DateTime<Utc>,
    pub correction_vector: Vector3,
}

impl ReconciliationSystem {
    pub async fn reconcile_position(
        &self,
        player_id: CharacterId,
        client_position: Position,
        timestamp: DateTime<Utc>,
    ) -> Option<PositionCorrection> {
        // Validate position with lag compensation
        match self.lag_compensation.validate_movement(
            player_id,
            client_position,
            timestamp,
        ).await {
            Ok(validated_position) => {
                // Check if correction is needed
                let distance = client_position.distance_to(validated_position);
                
                if distance > 0.1 { // Threshold for correction
                    // Position needs correction
                    let correction = PositionCorrection {
                        server_position: validated_position,
                        client_position,
                        timestamp,
                        correction_vector: validated_position - client_position,
                    };
                    
                    // Store correction
                    let mut buffer = self.correction_buffer.write().await;
                    let player_corrections = buffer.entry(player_id)
                        .or_insert_with(Vec::new);
                    player_corrections.push(correction.clone());
                    
                    // Keep only recent corrections
                    if player_corrections.len() > 10 {
                        player_corrections.remove(0);
                    }
                    
                    return Some(correction);
                }
                
                None
            }
            Err(_) => {
                // Movement invalid, needs correction
                // (handled by speed hack detector)
                None
            }
        }
    }
}
```

## Configuration

### Lag Compensation Settings

```rust
pub struct LagCompensationConfig {
    pub max_latency_compensation: Duration,  // Maximum latency to compensate (e.g., 500ms)
    pub max_speed: f32,                     // Maximum movement speed
    pub correction_threshold: f32,           // Distance threshold for corrections (e.g., 0.1 units)
    pub latency_smoothing_factor: f32,      // Smoothing factor for latency (e.g., 0.1)
    pub ping_interval: Duration,            // How often to ping clients (e.g., 1 second)
    pub max_latency_samples: usize,         // Maximum latency samples to keep (e.g., 100)
}
```

### Default Configuration

```rust
impl Default for LagCompensationConfig {
    fn default() -> Self {
        Self {
            max_latency_compensation: Duration::milliseconds(500),
            max_speed: 10.0, // units per second
            correction_threshold: 0.1, // units
            latency_smoothing_factor: 0.1,
            ping_interval: Duration::seconds(1),
            max_latency_samples: 100,
        }
    }
}
```

## Integration with Security System

### Speed Hack Detection Integration

The lag compensation system integrates with speed hack detection to prevent false positives:

1. **Movement Validation**: All movement is validated with lag compensation
2. **Compensation Applied**: Latency compensation is applied before speed check
3. **False Positive Prevention**: High latency players don't trigger false speed hack detections
4. **Real Violations**: Only real speed hacks (exceeding limits even with compensation) are detected

**See [SECURITY.md](./SECURITY.md) for speed hack detection details.**

## Performance Considerations

### Optimization

- **Caching**: Latency values cached per player
- **Batch Updates**: Multiple position updates processed together
- **Efficient Calculations**: Vector math optimized for performance
- **Memory Management**: Limited history size to prevent memory bloat

### Scalability

- **Per-Player State**: Each player has independent state
- **Concurrent Processing**: Multiple players processed concurrently
- **Minimal Overhead**: Compensation calculations are lightweight

## Testing Considerations

### Unit Tests

- Lag compensation calculation accuracy
- Velocity calculation correctness
- Maximum distance validation
- Latency tracking accuracy

### Integration Tests

- Client-server position reconciliation
- Speed hack detection with high latency
- Movement validation with various latencies
- Correction application and smoothing

### Performance Tests

- System performance with many players
- Latency tracking overhead
- Compensation calculation performance
- Memory usage with many players

## Related Documentation

- [SECURITY.md](./SECURITY.md) - Speed hack detection system (integrates with lag compensation)
- [MAPS.md](./MAPS.md) - Map system documentation (heightmap for position validation)

## Summary

The lag compensation system ensures smooth gameplay by:

- **Non-Blocking Movement**: Client movement is never blocked, ensuring responsive controls
- **Lag Compensation**: Calculates position compensation based on client latency
- **Server Validation**: Validates positions asynchronously with compensation applied
- **False Positive Prevention**: Prevents false speed hack detections due to latency
- **Client-Server Reconciliation**: Smoothly reconciles differences between client and server
- **Fluid Gameplay**: Ensures players experience smooth movement regardless of network conditions

The system maintains security by validating all movement while compensating for network latency, ensuring that legitimate high-latency players are not penalized while still detecting real speed hacks.

