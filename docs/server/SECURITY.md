# Security and Anti-Cheat System Documentation

## Overview

This document describes the comprehensive security system designed to protect the game from common MMORPG threats including item duplication, bots, hacks (speed, wallhack, flyhack), DDoS attacks, and various exploits. The system implements multiple layers of detection, monitoring, and automated responses to ensure game integrity and fair play.

## Critical Security Requirements

- **Zero Tolerance**: Any detected cheating results in immediate kick and temporary ban
- **Administrator Notification**: All security incidents are immediately reported to administrators
- **Real-Time Monitoring**: Continuous monitoring of player behavior and game state
- **Automated Response**: Immediate action on detected violations
- **Comprehensive Logging**: All suspicious activities are logged for analysis

## Item Security System

### Item Reference ID System

**Critical Requirement:**
Every item MUST have a unique reference ID and owner tracking to prevent duplication.

**Implementation:**
```rust
pub struct Item {
    pub item_id: ItemId, // Unique UUID - NEVER duplicated
    pub reference_id: ItemReferenceId, // Reference ID for tracking
    pub owner_id: CharacterId, // Current owner (changeable via trade/AH)
    pub item_type: String,
    pub properties: ItemProperties, // All characteristics preserved
    pub created_at: DateTime<Utc>,
    pub last_owner_change: DateTime<Utc>,
    pub ownership_history: Vec<OwnershipRecord>, // Track all ownership changes
}

pub struct OwnershipRecord {
    pub previous_owner: Option<CharacterId>,
    pub new_owner: CharacterId,
    pub transfer_method: TransferMethod, // Trade, AuctionHouse, Drop, Loot
    pub timestamp: DateTime<Utc>,
    pub transaction_id: Option<TransactionId>, // For trade/AH
}

#[derive(Debug, Clone)]
pub enum TransferMethod {
    Trade { trade_id: TransactionId },
    AuctionHouse { auction_id: AuctionId },
    Drop { location: Position },
    Loot { source: EntityId },
    Admin { admin_id: AccountId },
    Quest { quest_id: QuestId },
}
```

**Duplication Detection:**
```rust
pub struct ItemDuplicationDetector {
    item_registry: Arc<RwLock<HashMap<ItemReferenceId, ItemId>>>, // Reference -> Unique ID
    item_owners: Arc<RwLock<HashMap<ItemId, CharacterId>>>,
}

impl ItemDuplicationDetector {
    pub fn check_duplication(&self, item: &Item) -> Result<(), DuplicationError> {
        // Check if reference ID already exists with different unique ID
        let registry = self.item_registry.read().await;
        
        if let Some(existing_id) = registry.get(&item.reference_id) {
            if *existing_id != item.item_id {
                // Duplication detected!
                return Err(DuplicationError::DuplicateReference {
                    reference_id: item.reference_id.clone(),
                    existing_id: *existing_id,
                    duplicate_id: item.item_id,
                });
            }
        }
        
        Ok(())
    }
    
    pub fn register_item(&self, item: &Item) -> Result<(), DuplicationError> {
        // Check for duplication first
        self.check_duplication(item)?;
        
        // Register item
        self.item_registry.write().await
            .insert(item.reference_id.clone(), item.item_id);
        self.item_owners.write().await
            .insert(item.item_id, item.owner_id);
        
        Ok(())
    }
}
```

**Response to Duplication Detection:**
```rust
pub async fn handle_duplication_detection(
    player: CharacterId,
    item: Item,
    duplication_error: DuplicationError,
) {
    // Immediate actions
    kick_player(player).await;
    ban_player_temporary(player, Duration::hours(24), "Item duplication detected").await;
    notify_administrators(&format!("Duplication detected: Player {} attempted to duplicate item {}", player, item.item_id)).await;
    
    // Log incident
    log_security_incident(SecurityIncident {
        incident_type: IncidentType::ItemDuplication,
        player_id: player,
        details: serde_json::to_value(duplication_error).unwrap(),
        timestamp: Utc::now(),
    }).await;
    
    // Remove duplicate item
    remove_item(item.item_id).await;
}
```

### Stackable Item Security

**Critical Problem:**
Stackable items can be split, potentially generating massive amounts of resources. This is a clear sign of duplication.

**Security Rules:**
1. **Only Legitimate Transfers**: Resources can only transfer between players via:
   - Auction House (with transaction record)
   - Trade (with transaction record)
   - Quest rewards (with quest record)
   - Admin actions (with admin record)

2. **Transaction Tracking**: ALL trades and Auction House transactions MUST be logged

3. **Anomaly Detection**: Large amounts of stackable items without transaction records = duplication

**Implementation:**
```rust
pub struct StackableItemMonitor {
    transaction_log: Arc<RwLock<Vec<TransactionRecord>>>,
    player_inventories: Arc<RwLock<HashMap<CharacterId, InventorySnapshot>>>,
}

pub struct TransactionRecord {
    pub transaction_id: TransactionId,
    pub transaction_type: TransactionType,
    pub from_player: Option<CharacterId>,
    pub to_player: Option<CharacterId>,
    pub items: Vec<ItemTransfer>,
    pub timestamp: DateTime<Utc>,
    pub verified: bool,
}

#[derive(Debug, Clone)]
pub enum TransactionType {
    Trade,
    AuctionHouseBuy,
    AuctionHouseSell,
    QuestReward { quest_id: QuestId },
    AdminAction { admin_id: AccountId },
}

pub struct ItemTransfer {
    pub item_id: ItemId,
    pub item_type: String,
    pub quantity: u32,
    pub stackable: bool,
}

impl StackableItemMonitor {
    pub async fn validate_item_transfer(
        &self,
        from_player: Option<CharacterId>,
        to_player: Option<CharacterId>,
        item: &Item,
        quantity: u32,
        transfer_method: TransferMethod,
    ) -> Result<(), SecurityViolation> {
        // Check if transfer is legitimate
        match transfer_method {
            TransferMethod::Trade { trade_id } => {
                // Verify trade exists in transaction log
                if !self.verify_trade_exists(trade_id).await {
                    return Err(SecurityViolation::InvalidTransfer {
                        reason: "Trade transaction not found".to_string(),
                    });
                }
            }
            TransferMethod::AuctionHouse { auction_id } => {
                // Verify auction exists in transaction log
                if !self.verify_auction_exists(auction_id).await {
                    return Err(SecurityViolation::InvalidTransfer {
                        reason: "Auction transaction not found".to_string(),
                    });
                }
            }
            TransferMethod::Drop { .. } | TransferMethod::Loot { .. } => {
                // Single item drops/loots are OK
                if quantity > 1 {
                    return Err(SecurityViolation::InvalidTransfer {
                        reason: "Cannot drop/loot stackable items in quantity > 1".to_string(),
                    });
                }
            }
            _ => {
                // Other methods need verification
            }
        }
        
        // Check for suspicious quantities
        if item.is_stackable() && quantity > 10000 {
            // Flag for review
            self.flag_suspicious_activity(
                to_player,
                SuspiciousActivity::LargeStackableQuantity {
                    item_type: item.item_type.clone(),
                    quantity,
                },
            ).await;
        }
        
        Ok(())
    }
    
    pub async fn detect_duplication_anomaly(
        &self,
        player_id: CharacterId,
    ) -> Option<DuplicationAnomaly> {
        let inventory = self.get_player_inventory(player_id).await?;
        
        // Check for large quantities of stackable items
        for (item_type, quantity) in &inventory.stackable_items {
            if *quantity > 100000 {
                // Check transaction history
                let transactions = self.get_player_transactions(player_id).await;
                let total_received = self.calculate_total_received(
                    player_id,
                    item_type,
                    &transactions,
                ).await;
                
                if *quantity > total_received * 2 {
                    // Suspicious: player has more than double what they should have
                    return Some(DuplicationAnomaly {
                        player_id,
                        item_type: item_type.clone(),
                        current_quantity: *quantity,
                        expected_quantity: total_received,
                        discrepancy: *quantity - total_received,
                    });
                }
            }
        }
        
        None
    }
}
```

**Transaction Logging:**
```rust
pub async fn log_trade(
    trade_id: TransactionId,
    player1: CharacterId,
    player2: CharacterId,
    items_from_1: Vec<ItemTransfer>,
    items_from_2: Vec<ItemTransfer>,
) {
    let transaction = TransactionRecord {
        transaction_id: trade_id,
        transaction_type: TransactionType::Trade,
        from_player: Some(player1),
        to_player: Some(player2),
        items: items_from_1,
        timestamp: Utc::now(),
        verified: true,
    };
    
    transaction_log.write().await.push(transaction);
    
    // Also log reverse transaction
    let reverse_transaction = TransactionRecord {
        transaction_id: generate_transaction_id(),
        transaction_type: TransactionType::Trade,
        from_player: Some(player2),
        to_player: Some(player1),
        items: items_from_2,
        timestamp: Utc::now(),
        verified: true,
    };
    
    transaction_log.write().await.push(reverse_transaction);
}

pub async fn log_auction_house_transaction(
    auction_id: AuctionId,
    seller: CharacterId,
    buyer: CharacterId,
    item: ItemTransfer,
    price: i64,
) {
    let transaction = TransactionRecord {
        transaction_id: generate_transaction_id(),
        transaction_type: TransactionType::AuctionHouseBuy,
        from_player: Some(seller),
        to_player: Some(buyer),
        items: vec![item],
        timestamp: Utc::now(),
        verified: true,
    };
    
    transaction_log.write().await.push(transaction);
}
```

## Bot Detection System

### Behavioral Analysis

**Bot Indicators:**
1. **Predictable Behavior**: Very consistent, repetitive actions
2. **Sequential Logins**: Multiple accounts logging in sequence
3. **Same IP Multiple Accounts**: Multiple accounts from same IP address
4. **Perfect Timing**: Actions with inhuman precision
5. **No Human Errors**: Never makes mistakes humans would make

**Implementation:**
```rust
pub struct BotDetectionSystem {
    player_behaviors: Arc<RwLock<HashMap<CharacterId, PlayerBehavior>>>,
    ip_tracking: Arc<RwLock<HashMap<IpAddress, Vec<AccountId>>>>,
    login_patterns: Arc<RwLock<Vec<LoginPattern>>>,
}

pub struct PlayerBehavior {
    pub character_id: CharacterId,
    pub action_patterns: Vec<ActionPattern>,
    pub timing_consistency: f32, // 0.0 = random, 1.0 = perfect timing
    pub error_rate: f32, // 0.0 = no errors, 1.0 = all errors
    pub bot_probability: f32, // 0.0 = human, 1.0 = bot
    pub flagged: bool,
}

pub struct ActionPattern {
    pub action_type: ActionType,
    pub timestamps: Vec<DateTime<Utc>>,
    pub intervals: Vec<Duration>, // Time between actions
    pub consistency_score: f32,
}

impl BotDetectionSystem {
    pub async fn analyze_player_behavior(&self, player_id: CharacterId) {
        let behavior = self.get_player_behavior(player_id).await;
        
        // Calculate timing consistency
        let timing_consistency = self.calculate_timing_consistency(&behavior).await;
        
        // Calculate error rate (humans make mistakes)
        let error_rate = self.calculate_error_rate(&behavior).await;
        
        // Check for predictable patterns
        let predictability = self.calculate_predictability(&behavior).await;
        
        // Calculate bot probability
        let bot_probability = self.calculate_bot_probability(
            timing_consistency,
            error_rate,
            predictability,
        );
        
        // Update behavior
        let mut behaviors = self.player_behaviors.write().await;
        if let Some(player_behavior) = behaviors.get_mut(&player_id) {
            player_behavior.timing_consistency = timing_consistency;
            player_behavior.error_rate = error_rate;
            player_behavior.bot_probability = bot_probability;
            
            // Flag if bot probability is high
            if bot_probability > 0.7 {
                player_behavior.flagged = true;
                self.flag_as_possible_bot(player_id).await;
            }
        }
    }
    
    pub async fn check_ip_multiple_accounts(&self, ip: IpAddress) -> Vec<AccountId> {
        self.ip_tracking.read().await
            .get(&ip)
            .cloned()
            .unwrap_or_default()
    }
    
    pub async fn detect_sequential_logins(&self) -> Vec<LoginPattern> {
        let patterns = self.login_patterns.read().await.clone();
        
        patterns.into_iter()
            .filter(|pattern| {
                // Check if logins are too sequential (within seconds)
                pattern.accounts.len() > 3
                    && pattern.time_span < Duration::minutes(5)
            })
            .collect()
    }
    
    pub async fn flag_as_possible_bot(&self, player_id: CharacterId) {
        // Flag player for monitoring
        // Increase monitoring frequency
        // Track all actions more closely
        
        warn!("Player {} flagged as possible bot", player_id);
        
        // Notify administrators
        notify_administrators(&format!(
            "Player {} flagged as possible bot (probability: {:.2}%)",
            player_id,
            self.get_bot_probability(player_id).await * 100.0
        )).await;
    }
}
```

### Heartbeat System

**Purpose:**
Detect bots that simulate our protocol by using a dynamic heartbeat system that changes patterns.

**Implementation:**
```rust
pub struct HeartbeatSystem {
    current_multiplier: Arc<RwLock<u32>>, // Current heartbeat multiplier
    heartbeat_table: Arc<RwLock<Vec<u32>>>, // Table of valid heartbeat values
    player_heartbeats: Arc<RwLock<HashMap<CharacterId, HeartbeatState>>>,
    last_update: Arc<RwLock<Instant>>,
}

pub struct HeartbeatState {
    pub last_heartbeat: Instant,
    pub expected_heartbeat: Instant,
    pub multiplier: u32,
    pub valid_responses: u32,
    pub invalid_responses: u32,
}

impl HeartbeatSystem {
    pub async fn update_heartbeat_pattern(&self) {
        // Rarely triggered packet that changes heartbeat multiplier
        // This breaks bot scripts that hardcode heartbeat patterns
        
        let new_multiplier = self.generate_new_multiplier().await;
        *self.current_multiplier.write().await = new_multiplier;
        
        // Update heartbeat table
        let new_table = self.generate_heartbeat_table(new_multiplier).await;
        *self.heartbeat_table.write().await = new_table;
        
        // Send new heartbeat pattern to all players
        self.broadcast_new_heartbeat_pattern(new_multiplier).await;
        
        info!("Heartbeat pattern updated: multiplier = {}", new_multiplier);
    }
    
    pub async fn validate_heartbeat(
        &self,
        player_id: CharacterId,
        heartbeat_value: u32,
    ) -> Result<(), HeartbeatError> {
        let table = self.heartbeat_table.read().await;
        let expected_value = self.calculate_expected_heartbeat(player_id).await;
        
        // Check if heartbeat value matches expected pattern
        if !self.is_valid_heartbeat_value(heartbeat_value, expected_value, &table).await {
            // Invalid heartbeat detected
            let mut heartbeats = self.player_heartbeats.write().await;
            if let Some(state) = heartbeats.get_mut(&player_id) {
                state.invalid_responses += 1;
                
                // First invalid: kick
                if state.invalid_responses == 1 {
                    kick_player(player_id).await;
                    warn!("Player {} kicked for invalid heartbeat", player_id);
                }
                // Persistent: ban
                else if state.invalid_responses >= 3 {
                    ban_player_temporary(
                        player_id,
                        Duration::hours(24),
                        "Invalid heartbeat pattern (bot detected)",
                    ).await;
                    notify_administrators(&format!(
                        "Player {} banned for invalid heartbeat (bot detected)",
                        player_id
                    )).await;
                }
            }
            
            return Err(HeartbeatError::InvalidHeartbeat);
        }
        
        // Valid heartbeat
        let mut heartbeats = self.player_heartbeats.write().await;
        if let Some(state) = heartbeats.get_mut(&player_id) {
            state.valid_responses += 1;
            state.last_heartbeat = Instant::now();
        }
        
        Ok(())
    }
    
    async fn is_valid_heartbeat_value(
        &self,
        received: u32,
        expected: u32,
        table: &[u32],
    ) -> bool {
        // Check if received value matches expected pattern in table
        // Account for network delay and timing variations
        let tolerance = 100; // milliseconds
        
        table.iter().any(|&value| {
            let diff = (received as i64 - value as i64).abs();
            diff <= tolerance
        })
    }
}
```

**Heartbeat Pattern Update:**
```rust
// Rarely triggered (e.g., every 30-60 minutes)
pub async fn trigger_heartbeat_update() {
    heartbeat_system.update_heartbeat_pattern().await;
    
    // Monitor responses
    // All players should update their heartbeat pattern
    // Bots using hardcoded patterns will fail
}
```

## Hack Detection System

### Speed Hack Detection

**Critical Requirement:**
Players moving faster than allowed is unacceptable and must be detected immediately.

**Lag Compensation Integration:**
Speed hack detection integrates with the lag compensation system (see [LAG_COMPENSATION.md](./LAG_COMPENSATION.md)) to prevent false positives due to high latency. All movement validation applies lag compensation before checking speed limits.

**Implementation:**
```rust
pub struct SpeedHackDetector {
    max_speed: f32, // Maximum allowed speed
    lag_compensation: Arc<LagCompensationSystem>, // Lag compensation system
    speed_history: Arc<RwLock<HashMap<CharacterId, Vec<SpeedRecord>>>>,
}

pub struct SpeedRecord {
    pub position: Position,
    pub timestamp: Instant,
    pub calculated_speed: f32,
}

impl SpeedHackDetector {
    pub async fn validate_movement(
        &self,
        player_id: CharacterId,
        new_position: Position,
        timestamp: Instant,
    ) -> Result<(), SpeedHackViolation> {
        // First, validate with lag compensation
        // This prevents false positives from high latency players
        let validated_position = match self.lag_compensation.validate_movement(
            player_id,
            new_position,
            timestamp.into(),
        ).await {
            Ok(pos) => pos,
            Err(MovementError::SpeedViolation { distance_moved, max_allowed_distance, .. }) => {
                // Movement exceeds maximum even with lag compensation
                // This is a real speed hack violation
                let time_delta = if let Some(last_record) = self.speed_history.read().await.get(&player_id)
                    .and_then(|h| h.last()) {
                    timestamp.duration_since(last_record.timestamp).as_secs_f32()
                } else {
                    0.1 // Default time delta
                };
                
                let speed = distance_moved / time_delta;
                
                let violation = SpeedHackViolation {
                    player_id,
                    detected_speed: speed,
                    max_allowed_speed: self.max_speed,
                    position: new_position,
                    timestamp,
                };
                
                self.handle_speed_hack(violation).await;
                return Err(violation);
            }
            Err(_) => {
                // Other errors, use original position
                new_position
            }
        };
        
        // Continue with speed validation using compensated position
        let mut history = self.speed_history.write().await;
        let player_history = history.entry(player_id).or_insert_with(Vec::new);
        
        if let Some(last_record) = player_history.last() {
            let distance = validated_position.distance(last_record.position);
            let time_delta = timestamp.duration_since(last_record.timestamp);
            let speed = distance / time_delta.as_secs_f32();
            
            // Check against maximum speed (already compensated for latency)
            if speed > self.max_speed {
                // Speed hack detected!
                let violation = SpeedHackViolation {
                    player_id,
                    detected_speed: speed,
                    max_allowed_speed: self.max_speed,
                    position: validated_position,
                    timestamp,
                };
                
                // Immediate actions
                self.handle_speed_hack(violation).await;
                
                return Err(violation);
            }
        }
        
        // Record valid movement (using compensated position)
        player_history.push(SpeedRecord {
            position: validated_position,
            timestamp,
            calculated_speed: if let Some(last_record) = player_history.last() {
                let distance = validated_position.distance(last_record.position);
                let time_delta = timestamp.duration_since(last_record.timestamp);
                distance / time_delta.as_secs_f32()
            } else {
                0.0
            },
        });
        
        // Keep only recent history (last 10 seconds)
        player_history.retain(|record| {
            timestamp.duration_since(record.timestamp) < Duration::from_secs(10)
        });
        
        Ok(())
    }
    
    async fn handle_speed_hack(&self, violation: SpeedHackViolation) {
        // Immediate kick
        kick_player(violation.player_id).await;
        
        // Temporary ban
        ban_player_temporary(
            violation.player_id,
            Duration::hours(24),
            "Speed hack detected",
        ).await;
        
        // Notify administrators
        notify_administrators(&format!(
            "Speed hack detected: Player {} moving at {:.2} units/s (max: {:.2})",
            violation.player_id,
            violation.detected_speed,
            violation.max_allowed_speed
        )).await;
        
        // Log incident
        log_security_incident(SecurityIncident {
            incident_type: IncidentType::SpeedHack,
            player_id: violation.player_id,
            details: serde_json::to_value(violation).unwrap(),
            timestamp: Utc::now(),
        }).await;
    }
}
```

**Detection Features:**
- **Lag Compensation**: Integrates with lag compensation system to prevent false positives (see [LAG_COMPENSATION.md](./LAG_COMPENSATION.md))
- **Real-Time Monitoring**: Validates every movement update
- **Speed Calculation**: Calculates speed from position changes (with latency compensation applied)
- **Threshold Detection**: Detects speeds exceeding maximum (even with compensation)
- **False Positive Prevention**: High latency players don't trigger false detections
- **Immediate Response**: Kicks and bans detected speed hackers

### Wallhack Detection

**Critical Requirement:**
Players passing through walls is unacceptable.

**Implementation:**
```rust
pub struct WallhackDetector {
    map_collision: Arc<MapCollisionSystem>,
    player_positions: Arc<RwLock<HashMap<CharacterId, Vec<Position>>>>,
}

impl WallhackDetector {
    pub async fn validate_movement(
        &self,
        player_id: CharacterId,
        from: Position,
        to: Position,
    ) -> Result<(), WallhackViolation> {
        // Check if movement path intersects with walls
        let collision = self.map_collision.check_path_collision(from, to).await;
        
        if collision.has_wall_collision {
            // Player passed through wall!
            let violation = WallhackViolation {
                player_id,
                from_position: from,
                to_position: to,
                collision_point: collision.collision_point,
                timestamp: Utc::now(),
            };
            
            // Immediate actions
            self.handle_wallhack(violation).await;
            
            return Err(violation);
        }
        
        Ok(())
    }
    
    async fn handle_wallhack(&self, violation: WallhackViolation) {
        // Immediate kick
        kick_player(violation.player_id).await;
        
        // Temporary ban
        ban_player_temporary(
            violation.player_id,
            Duration::hours(24),
            "Wallhack detected",
        ).await;
        
        // Notify administrators
        notify_administrators(&format!(
            "Wallhack detected: Player {} passed through wall",
            violation.player_id
        )).await;
        
        // Log incident
        log_security_incident(SecurityIncident {
            incident_type: IncidentType::Wallhack,
            player_id: violation.player_id,
            details: serde_json::to_value(violation).unwrap(),
            timestamp: Utc::now(),
        }).await;
    }
}
```

### Flyhack Detection

**Critical Requirement:**
Players flying or moving in impossible ways is unacceptable.

**Heightmap Integration:**
Flyhack detection uses the map heightmap system (see [MAPS.md](./MAPS.md)) to validate player positions against actual terrain heights. The heightmap provides accurate ground level data for each X,Y position, including support for multi-level structures (bridges, platforms).

**Implementation:**
```rust
pub struct FlyhackDetector {
    max_jump_height: f32,
    max_fall_speed: f32,
    max_vertical_speed: f32,
    infinite_fall_threshold: f32,        // Height threshold for infinite fall detection
    infinite_fall_duration: Duration,    // Duration threshold for infinite fall
    heightmaps: Arc<RwLock<HashMap<MapId, Heightmap>>>,
    player_positions: Arc<RwLock<HashMap<CharacterId, Vec<Position>>>>,
    valid_position_history: Arc<RwLock<HashMap<CharacterId, Vec<ValidPosition>>>>, // History of valid positions
    falling_states: Arc<RwLock<HashMap<CharacterId, FallingState>>>, // Track falling state per player
}

impl FlyhackDetector {
    pub async fn validate_position(
        &self,
        player_id: CharacterId,
        position: Position,
        map_id: MapId,
    ) -> Result<(), FlyhackViolation> {
        // Get heightmap for this map
        let heightmaps = self.heightmaps.read().await;
        let heightmap = heightmaps.get(&map_id)
            .ok_or_else(|| FlyhackViolation {
                player_id,
                position,
                map_id,
                height_above_ground: 0.0,
                nearest_ground: None,
                max_allowed_height: self.max_jump_height,
                violation_type: ViolationType::NoHeightmap,
                timestamp: Utc::now(),
            })?;
        
        // Query heightmap for ground level at player's X,Y position
        let ground_heights = heightmap.get_heights_at(position.x, position.y);
        
        if ground_heights.is_empty() {
            // No ground data at this position - check if player is in valid bounds
            if !heightmap.is_in_bounds(position.x, position.y) {
                let violation = FlyhackViolation {
                    player_id,
                    position,
                    map_id,
                    height_above_ground: f32::MAX,
                    nearest_ground: None,
                    max_allowed_height: self.max_jump_height,
                    violation_type: ViolationType::OutOfBounds,
                    timestamp: Utc::now(),
                };
                self.handle_flyhack(violation).await;
                return Err(violation);
            }
            // Position might be valid but no ground data - allow with warning
            return Ok(());
        }
        
        // Find nearest valid ground level (can be above or below player)
        let nearest_ground = self.find_nearest_ground(position.z, &ground_heights);
        
        // Check if player is too high above ground
        let height_above_ground = position.z - nearest_ground;
        
        if height_above_ground > self.max_jump_height {
            // Player is flying!
            let violation = FlyhackViolation {
                player_id,
                position,
                map_id,
                height_above_ground,
                nearest_ground: Some(nearest_ground),
                max_allowed_height: self.max_jump_height,
                violation_type: ViolationType::TooHigh,
                timestamp: Utc::now(),
            };
            
            // Immediate actions
            self.handle_flyhack(violation).await;
            
            return Err(violation);
        }
        
        // Check if player is below ground (falling through terrain)
        if position.z < nearest_ground - 10.0 { // 10 unit tolerance for ground penetration
            let violation = FlyhackViolation {
                player_id,
                position,
                map_id,
                height_above_ground: position.z - nearest_ground,
                nearest_ground: Some(nearest_ground),
                max_allowed_height: self.max_jump_height,
                violation_type: ViolationType::BelowGround,
                timestamp: Utc::now(),
            };
            
            self.handle_flyhack(violation).await;
            return Err(violation);
        }
        
        // Check vertical movement speed
        let mut positions = self.player_positions.write().await;
        let player_history = positions.entry(player_id).or_insert_with(Vec::new);
        
        let mut falling_states = self.falling_states.write().await;
        let falling_state = falling_states.entry(player_id).or_insert_with(|| FallingState {
            start_time: Utc::now(),
            start_height: position.z,
            consecutive_falls: 0,
        });
        
        if let Some(last_pos) = player_history.last() {
            let time_delta = 0.1; // Assuming 100ms between updates
            let vertical_distance = (position.z - last_pos.z).abs();
            let vertical_speed = vertical_distance / time_delta;
            
            // Check upward movement (jumping/flying)
            if position.z > last_pos.z && vertical_speed > self.max_vertical_speed {
                // Reset falling state on upward movement
                falling_state.consecutive_falls = 0;
                falling_state.start_time = Utc::now();
                falling_state.start_height = position.z;
                
                let violation = FlyhackViolation {
                    player_id,
                    position,
                    map_id,
                    height_above_ground,
                    nearest_ground: Some(nearest_ground),
                    max_allowed_height: self.max_jump_height,
                    violation_type: ViolationType::ExcessiveVerticalSpeed,
                    timestamp: Utc::now(),
                };
                
                self.handle_flyhack(violation).await;
                return Err(violation);
            }
            
            // Check downward movement (falling)
            if position.z < last_pos.z {
                let fall_speed = vertical_distance / time_delta;
                
                // Check for excessive fall speed
                if fall_speed > self.max_fall_speed {
                    falling_state.consecutive_falls = 0;
                    falling_state.start_time = Utc::now();
                    falling_state.start_height = position.z;
                    
                    let violation = FlyhackViolation {
                        player_id,
                        position,
                        map_id,
                        height_above_ground,
                        nearest_ground: Some(nearest_ground),
                        max_allowed_height: self.max_jump_height,
                        violation_type: ViolationType::ExcessiveFallSpeed,
                        timestamp: Utc::now(),
                    };
                    
                    self.handle_flyhack(violation).await;
                    return Err(violation);
                }
                
                // Track consecutive falls (infinite fall detection)
                falling_state.consecutive_falls += 1;
                let fall_duration = Utc::now().signed_duration_since(falling_state.start_time);
                let height_fallen = falling_state.start_height - position.z;
                
                // Check for infinite fall: continuous falling for too long or too far
                if falling_state.consecutive_falls >= 10 && // At least 10 consecutive fall updates
                   (fall_duration.num_seconds() >= self.infinite_fall_duration.num_seconds() ||
                    height_fallen > self.infinite_fall_threshold) {
                    
                    // Get last valid position from history
                    let mut valid_history = self.valid_position_history.write().await;
                    let player_valid_history = valid_history.entry(player_id).or_insert_with(Vec::new);
                    
                    if let Some(last_valid) = player_valid_history.last() {
                        // Teleport player back to last valid position
                        self.correct_infinite_fall(
                            player_id,
                            position,
                            last_valid.position,
                            map_id,
                            height_fallen,
                            fall_duration,
                        ).await;
                        
                        // Reset falling state
                        falling_state.consecutive_falls = 0;
                        falling_state.start_time = Utc::now();
                        falling_state.start_height = last_valid.position.z;
                        
                        return Ok(()); // Position corrected, allow
                    } else {
                        // No valid history, use current position as fallback
                        falling_state.consecutive_falls = 0;
                        falling_state.start_time = Utc::now();
                        falling_state.start_height = position.z;
                    }
                }
            } else {
                // Not falling, reset falling state
                falling_state.consecutive_falls = 0;
                falling_state.start_time = Utc::now();
                falling_state.start_height = position.z;
            }
        }
        
        // Position is valid - record it
        player_history.push(position);
        if player_history.len() > 100 {
            player_history.remove(0); // Keep only last 100 positions
        }
        
        // Record valid position in history (only if on ground or reasonable height)
        if height_above_ground <= self.max_jump_height {
            let mut valid_history = self.valid_position_history.write().await;
            let player_valid_history = valid_history.entry(player_id).or_insert_with(Vec::new);
            
            player_valid_history.push(ValidPosition {
                position,
                timestamp: Utc::now(),
                height_above_ground,
            });
            
            // Keep only last 50 valid positions
            if player_valid_history.len() > 50 {
                player_valid_history.remove(0);
            }
        }
        
        Ok(())
    }
    
    async fn correct_infinite_fall(
        &self,
        player_id: CharacterId,
        current_position: Position,
        valid_position: Position,
        map_id: MapId,
        height_fallen: f32,
        fall_duration: Duration,
    ) {
        // Log the correction (this is NOT a violation, just a bug correction)
        info!(
            "Infinite fall detected for player {}: fell {:.2} units over {:?}, correcting to last valid position",
            player_id, height_fallen, fall_duration
        );
        
        // Teleport player to last valid position
        teleport_player(player_id, valid_position, map_id).await;
        
        // Notify player
        notify_player(
            player_id,
            "You were falling into a void. You have been returned to safety. Use /unstuck if you get stuck again.",
        ).await;
        
        // Log correction (non-violation, just bug correction)
        // This is logged for debugging purposes, not as a security incident
        log_position_correction(PositionCorrection {
            player_id,
            correction_type: CorrectionType::InfiniteFall,
            current_position,
            corrected_position: valid_position,
            height_fallen,
            fall_duration_seconds: fall_duration.num_seconds(),
            timestamp: Utc::now(),
        }).await;
    }
    
    fn find_nearest_ground(&self, player_z: f32, ground_heights: &[f32]) -> f32 {
        // Find the ground height closest to player's Z position
        // This handles multi-level structures (bridges, platforms)
        ground_heights.iter()
            .min_by(|a, b| {
                let dist_a = (player_z - **a).abs();
                let dist_b = (player_z - **b).abs();
                dist_a.partial_cmp(&dist_b).unwrap_or(std::cmp::Ordering::Equal)
            })
            .copied()
            .unwrap_or(player_z)
    }
    
    async fn handle_flyhack(&self, violation: FlyhackViolation) {
        // Immediate kick
        kick_player(violation.player_id).await;
        
        // Temporary ban
        ban_player_temporary(
            violation.player_id,
            Duration::hours(24),
            &format!("Flyhack detected: {:?}", violation.violation_type),
        ).await;
        
        // Notify administrators
        notify_administrators(&format!(
            "Flyhack detected: Player {} - Type: {:?}, Height: {:.2} (max: {:.2}), Ground: {:?}",
            violation.player_id,
            violation.violation_type,
            violation.height_above_ground,
            violation.max_allowed_height,
            violation.nearest_ground
        )).await;
        
        // Log incident
        log_security_incident(SecurityIncident {
            incident_type: IncidentType::Flyhack,
            player_id: violation.player_id,
            details: serde_json::to_value(violation).unwrap(),
            timestamp: Utc::now(),
        }).await;
    }
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct FlyhackViolation {
    pub player_id: CharacterId,
    pub position: Position,
    pub map_id: MapId,
    pub height_above_ground: f32,
    pub nearest_ground: Option<f32>,
    pub max_allowed_height: f32,
    pub violation_type: ViolationType,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum ViolationType {
    TooHigh,                    // Player too high above ground
    BelowGround,                 // Player below ground level
    ExcessiveVerticalSpeed,      // Moving up too fast
    ExcessiveFallSpeed,          // Falling too fast
    OutOfBounds,                 // Position outside map bounds
    NoHeightmap,                 // No heightmap data for map
    // Note: Infinite fall is NOT a violation - it's a known game bug that gets auto-corrected
}

#[derive(Debug, Clone)]
pub struct ValidPosition {
    pub position: Position,
    pub timestamp: DateTime<Utc>,
    pub height_above_ground: f32,
}

#[derive(Debug, Clone)]
pub struct FallingState {
    pub start_time: DateTime<Utc>,
    pub start_height: f32,
    pub consecutive_falls: u32,
}

// Position correction (non-violation, just bug correction)
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PositionCorrection {
    pub player_id: CharacterId,
    pub correction_type: CorrectionType,
    pub current_position: Position,
    pub corrected_position: Position,
    pub height_fallen: f32,
    pub fall_duration_seconds: i64,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum CorrectionType {
    InfiniteFall,  // Player falling infinitely (common bug, auto-corrected)
}

// Heightmap structure (loaded from JSON)
pub struct Heightmap {
    bounds: BoundingBox,
    height_grid: HashMap<(i32, i32), Vec<i32>>, // (X, Y) -> Vec<Z heights>
    resolution: i32,
}

impl Heightmap {
    pub fn get_heights_at(&self, x: f32, y: f32) -> Vec<f32> {
        // Round to grid coordinates
        let grid_x = (x / self.resolution as f32).round() as i32 * self.resolution;
        let grid_y = (y / self.resolution as f32).round() as i32 * self.resolution;
        
        self.height_grid
            .get(&(grid_x, grid_y))
            .map(|heights| heights.iter().map(|&z| z as f32).collect())
            .unwrap_or_default()
    }
    
    pub fn is_in_bounds(&self, x: f32, y: f32) -> bool {
        self.bounds.contains_point(Vector2::new(x, y))
    }
}
```

**Heightmap Loading:**
Heightmaps are loaded from JSON files exported by Unreal Engine's `NavMeshExporter`:
- **File Format**: JSON with compact structure
- **File Location**: `Metadata/Scenes/{MapName}_Heightmap.json`
- **Loading**: Loaded at map initialization, cached in memory for fast queries
- **Multi-Level Support**: Handles multiple Z heights per X,Y position (bridges, platforms)

**Detection Features:**
- **Height Validation**: Validates player Z position against terrain height from heightmap
- **Multi-Level Support**: Handles multi-level structures (bridges, platforms)
- **Vertical Speed**: Detects excessive upward or downward movement speed
- **Ground Penetration**: Detects players falling through terrain
- **Out of Bounds**: Detects players outside map boundaries
- **Infinite Fall Detection**: Detects and corrects infinite falling (common bug mitigation, NOT a violation)
- **Valid Position History**: Maintains history of valid positions for fall correction
- **Automatic Correction**: Teleports players back to last valid position when infinite fall detected
- **Manual Alternative**: Players can use `/unstuck` command if they get stuck (see [ADMIN_COMMANDS.md](./ADMIN_COMMANDS.md))
- **Real-Time**: Validates every position update from client

**Infinite Fall Mitigation:**
The system includes special handling for a common bug where players fall into holes or voids and fall infinitely. **This is NOT a violation** - it's a known game bug that gets automatically corrected:
- **Not a Security Violation**: Infinite fall is a common game bug, not a hack or exploit
- **No Kick/Ban**: Players are NOT kicked or banned for falling infinitely
- **Automatic Correction Only**: System automatically corrects the issue without penalty
- **Valid Position Tracking**: System maintains a history of valid positions (on ground or reasonable height)
- **Fall Detection**: Tracks consecutive fall updates and fall duration
- **Threshold Detection**: Detects infinite fall when:
  - Player has 10+ consecutive fall updates, AND
  - Fall duration exceeds threshold (e.g., 3 seconds), OR
  - Height fallen exceeds threshold (e.g., 500 units)
- **Automatic Correction**: When infinite fall detected:
  - Player is teleported to last valid position
  - Player receives notification about the correction
  - Correction is logged for debugging (not as security incident)
  - Falling state is reset
- **Manual Alternative**: Players can use `/unstuck` command to manually teleport to safe location if needed
- **Graceful Handling**: System prevents player frustration from map bugs while maintaining security

## DDoS Protection

### Rate Limiting

**Implementation:**
```rust
pub struct DDoSProtection {
    connection_rate_limiter: Arc<RwLock<HashMap<IpAddress, RateLimiter>>>,
    request_rate_limiter: Arc<RwLock<HashMap<IpAddress, RateLimiter>>>,
}

pub struct RateLimiter {
    requests: Vec<Instant>,
    max_requests: u32,
    time_window: Duration,
}

impl DDoSProtection {
    pub async fn check_connection_rate(&self, ip: IpAddress) -> Result<(), DDoSViolation> {
        let mut limiters = self.connection_rate_limiter.write().await;
        let limiter = limiters.entry(ip).or_insert_with(|| {
            RateLimiter {
                requests: Vec::new(),
                max_requests: 10, // Max 10 connections per minute
                time_window: Duration::from_secs(60),
            }
        });
        
        let now = Instant::now();
        limiter.requests.retain(|&time| {
            now.duration_since(time) < limiter.time_window
        });
        
        if limiter.requests.len() >= limiter.max_requests as usize {
            return Err(DDoSViolation::TooManyConnections);
        }
        
        limiter.requests.push(now);
        Ok(())
    }
    
    pub async fn check_request_rate(&self, ip: IpAddress) -> Result<(), DDoSViolation> {
        // Similar implementation for request rate limiting
        // Max requests per second from same IP
        Ok(())
    }
}
```

## Automated Response System

### Response Actions

**When Violation Detected:**
1. **Immediate Kick**: Player disconnected immediately
2. **Temporary Ban**: 24-hour ban (configurable)
3. **Administrator Notification**: All admins notified
4. **Incident Logging**: Full details logged
5. **Data Preservation**: Evidence preserved for review

**Implementation:**
```rust
pub async fn handle_security_violation(
    player_id: CharacterId,
    violation_type: ViolationType,
    details: serde_json::Value,
) {
    // Step 1: Immediate kick
    kick_player(player_id).await;
    
    // Step 2: Temporary ban
    ban_player_temporary(
        player_id,
        Duration::hours(24),
        &format!("{:?}", violation_type),
    ).await;
    
    // Step 3: Notify administrators
    notify_administrators(&format!(
        "Security violation detected: Player {} - {:?}",
        player_id,
        violation_type
    )).await;
    
    // Step 4: Log incident
    log_security_incident(SecurityIncident {
        incident_type: violation_type,
        player_id,
        details,
        timestamp: Utc::now(),
    }).await;
}

pub enum ViolationType {
    ItemDuplication,
    BotDetected,
    SpeedHack,
    Wallhack,
    Flyhack,
    InvalidHeartbeat,
    DDoSAttempt,
    OtherExploit(String),
}
```

## Real-Time Monitoring System

### Monitoring Dashboard

**Monitored Metrics:**
- Player behavior patterns
- Item transactions
- Movement anomalies
- Heartbeat responses
- Connection patterns
- Resource generation

**Implementation:**
```rust
pub struct RealTimeMonitoring {
    active_monitors: Arc<RwLock<HashMap<CharacterId, MonitorConfig>>>,
    alerts: Arc<RwLock<Vec<SecurityAlert>>>,
}

pub struct MonitorConfig {
    pub player_id: CharacterId,
    pub monitoring_level: MonitoringLevel,
    pub check_interval: Duration,
}

#[derive(Debug, Clone, Copy)]
pub enum MonitoringLevel {
    Normal,    // Standard monitoring
    Elevated,  // Increased monitoring (flagged players)
    Critical,  // Maximum monitoring (suspected bots)
}

impl RealTimeMonitoring {
    pub async fn start_monitoring(&self, player_id: CharacterId) {
        // Start monitoring player
        // Check behavior, transactions, movement, etc.
    }
    
    pub async fn generate_alert(&self, alert: SecurityAlert) {
        // Add to alerts queue
        self.alerts.write().await.push(alert);
        
        // Notify administrators if critical
        if alert.severity == AlertSeverity::Critical {
            notify_administrators(&format!("Critical alert: {}", alert.message)).await;
        }
    }
}
```

## Other Known Exploits

### Inventory Exploits

**Detection:**
- Negative quantities
- Items exceeding max stack size
- Items in invalid slots
- Duplicate items in same slot

### Economy Exploits

**Detection:**
- Gold generation without source
- Negative gold balances
- Suspicious gold transfers
- Auction house manipulation

### Quest Exploits

**Detection:**
- Quest completion without requirements
- Multiple quest rewards
- Quest state manipulation
- Quest item duplication

### Combat Exploits

**Detection:**
- Damage modification
- Cooldown bypass
- Skill point manipulation
- Status effect manipulation

## Logging and Auditing

### Security Incident Logging

**All Incidents Logged:**
- Timestamp
- Player ID
- Violation type
- Details (JSON)
- Actions taken
- Administrator notifications

**Log Retention:**
- Security incidents: 1 year
- Transaction logs: 6 months
- Behavior patterns: 3 months
- Heartbeat logs: 1 month

## Summary

The security system provides:

- **Item Security**: Reference IDs, owner tracking, duplication detection
- **Stackable Item Protection**: Transaction logging, anomaly detection
- **Bot Detection**: Behavioral analysis, heartbeat system, IP tracking
- **Hack Detection**: Speed, wallhack, flyhack monitoring
- **DDoS Protection**: Rate limiting, connection management
- **Automated Response**: Immediate kick, temporary ban, admin notification
- **Real-Time Monitoring**: Continuous player behavior analysis
- **Comprehensive Logging**: All security incidents logged

**Key Features:**
- **Zero Tolerance**: Any violation = immediate action
- **Real-Time Detection**: Continuous monitoring
- **Automated Response**: No manual intervention needed
- **Administrator Alerts**: All incidents reported immediately
- **Evidence Preservation**: All data logged for review

This system ensures game integrity and fair play by detecting and preventing common MMORPG exploits and attacks.

