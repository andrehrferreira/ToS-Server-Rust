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

**Implementation:**
```rust
pub struct SpeedHackDetector {
    max_speed: f32, // Maximum allowed speed
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
        let mut history = self.speed_history.write().await;
        let player_history = history.entry(player_id).or_insert_with(Vec::new);
        
        if let Some(last_record) = player_history.last() {
            let distance = new_position.distance(last_record.position);
            let time_delta = timestamp.duration_since(last_record.timestamp);
            let speed = distance / time_delta.as_secs_f32();
            
            // Check against maximum speed
            if speed > self.max_speed {
                // Speed hack detected!
                let violation = SpeedHackViolation {
                    player_id,
                    detected_speed: speed,
                    max_allowed_speed: self.max_speed,
                    position: new_position,
                    timestamp,
                };
                
                // Immediate actions
                self.handle_speed_hack(violation).await;
                
                return Err(violation);
            }
        }
        
        // Record valid movement
        player_history.push(SpeedRecord {
            position: new_position,
            timestamp,
            calculated_speed: 0.0, // Will be calculated next time
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

**Implementation:**
```rust
pub struct FlyhackDetector {
    max_jump_height: f32,
    ground_level: HashMap<MapId, f32>,
    player_positions: Arc<RwLock<HashMap<CharacterId, Vec<Position>>>>,
}

impl FlyhackDetector {
    pub async fn validate_position(
        &self,
        player_id: CharacterId,
        position: Position,
        map_id: MapId,
    ) -> Result<(), FlyhackViolation> {
        let ground_level = self.ground_level.get(&map_id)
            .copied()
            .unwrap_or(0.0);
        
        // Check if player is too high above ground
        let height_above_ground = position.z - ground_level;
        
        if height_above_ground > self.max_jump_height {
            // Player is flying!
            let violation = FlyhackViolation {
                player_id,
                position,
                map_id,
                height_above_ground,
                max_allowed_height: self.max_jump_height,
                timestamp: Utc::now(),
            };
            
            // Immediate actions
            self.handle_flyhack(violation).await;
            
            return Err(violation);
        }
        
        // Check vertical movement speed
        let mut positions = self.player_positions.write().await;
        let player_history = positions.entry(player_id).or_insert_with(Vec::new);
        
        if let Some(last_pos) = player_history.last() {
            let vertical_speed = (position.z - last_pos.z).abs() / 0.1; // Assuming 100ms between updates
            
            if vertical_speed > 50.0 { // Max vertical speed
                let violation = FlyhackViolation {
                    player_id,
                    position,
                    map_id,
                    height_above_ground,
                    max_allowed_height: self.max_jump_height,
                    timestamp: Utc::now(),
                };
                
                self.handle_flyhack(violation).await;
                return Err(violation);
            }
        }
        
        // Record valid position
        player_history.push(position);
        player_history.retain(|pos| {
            // Keep only recent positions
            true // Simplified
        });
        
        Ok(())
    }
    
    async fn handle_flyhack(&self, violation: FlyhackViolation) {
        // Immediate kick
        kick_player(violation.player_id).await;
        
        // Temporary ban
        ban_player_temporary(
            violation.player_id,
            Duration::hours(24),
            "Flyhack detected",
        ).await;
        
        // Notify administrators
        notify_administrators(&format!(
            "Flyhack detected: Player {} at height {:.2} (max: {:.2})",
            violation.player_id,
            violation.height_above_ground,
            violation.max_allowed_height
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
```

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

