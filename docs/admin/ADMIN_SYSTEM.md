# Administrative System Documentation

## Overview

This document provides an overview of the administrative system. For detailed documentation on specific components, see the individual documentation files.

The administrative system provides a comprehensive web-based dashboard for game administrators to manage all aspects of the game servers, players, and content. The system includes a unified API for configuration preloading, authentication, and real-time server management. Administrators can perform all management tasks without needing to log into the game client.

The central API server is designed with replication to support high request volumes, uses PostgreSQL for data storage with eventual consistency synchronization, and implements a comprehensive backup system using cloud storage (S3 or Digital Ocean Spaces) to ensure zero data loss.

## Documentation Structure

- [OVERVIEW.md](./OVERVIEW.md) - System overview and introduction
- [ARCHITECTURE.md](./ARCHITECTURE.md) - Central API server architecture
- [AUTHENTICATION.md](./AUTHENTICATION.md) - Authentication and token system
- [CONFIG_API.md](./CONFIG_API.md) - Unified configuration API
- [DASHBOARD.md](./DASHBOARD.md) - Administrative dashboard overview
- [SERVER_MANAGEMENT.md](./SERVER_MANAGEMENT.md) - Server management features
- [PLAYER_MANAGEMENT.md](./PLAYER_MANAGEMENT.md) - Player management features
- [CONTENT_MANAGEMENT.md](./CONTENT_MANAGEMENT.md) - Content management features
- [INTERACTIVE_MAP.md](./INTERACTIVE_MAP.md) - Interactive map system
- [SUPPORT_SYSTEM.md](./SUPPORT_SYSTEM.md) - Support ticket system
- [REWARD_SYSTEM.md](./REWARD_SYSTEM.md) - Reward distribution system
- [LOG_ANALYSIS.md](./LOG_ANALYSIS.md) - Log analysis and viewing
- [CHAT_MONITORING.md](./CHAT_MONITORING.md) - Chat monitoring system
- [BACKUP_SYSTEM.md](./BACKUP_SYSTEM.md) - Backup and recovery system
- [PERMISSIONS.md](./PERMISSIONS.md) - Security and permissions
- [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) - Real-time updates via SSE
- [PERSISTENCE.md](./PERSISTENCE.md) - Data persistence and PostgreSQL

## Key Features

- **Unified Configuration API**: Central API for all server configuration data
- **Token-Based Authentication**: Secure token system for player authentication
- **Web-Based Dashboard**: Full administrative control via web interface
- **Real-Time Server Management**: Control and monitor servers in real-time
- **Player Management**: Complete player account and character management
- **Content Management**: Create and configure items, NPCs, quests, zones, events
- **Telemetry and Monitoring**: Real-time server metrics and monitoring
- **Log Analysis**: Comprehensive log viewing and analysis
- **Interactive Maps**: Visual representation of game world state

## Central API Server Architecture

### High Availability Design

**Replication:**
- Multiple API server instances for load distribution
- Load balancer distributes requests across instances
- Horizontal scaling to handle high request volumes
- Health checks and automatic failover

**Database:**
- PostgreSQL as primary database
- Read replicas for read-heavy operations
- Write operations to primary database
- Eventual consistency synchronization

**Storage:**
- PostgreSQL for structured data
- Cloud storage (S3/Digital Ocean Spaces) for backups and files
- CDN for static assets

### Server Replication

**Architecture:**
```
Load Balancer
    ↓
API Server Instance 1 ──┐
API Server Instance 2 ──┼──→ PostgreSQL Primary
API Server Instance 3 ──┘         ↓
                          PostgreSQL Replicas (Read)
```

**Implementation:**
```rust
pub struct ApiServerCluster {
    instances: Vec<ApiServerInstance>,
    load_balancer: LoadBalancer,
    database_pool: PgPool, // Primary for writes
    read_replica_pool: PgPool, // Replicas for reads
}

pub struct ApiServerInstance {
    pub instance_id: InstanceId,
    pub health_status: HealthStatus,
    pub request_count: AtomicU64,
    pub last_health_check: Instant,
}

impl ApiServerCluster {
    pub async fn handle_request(&self, request: Request) -> Response {
        // Select instance via load balancer
        let instance = self.load_balancer.select_instance().await;
        
        // Route request to instance
        instance.handle_request(request).await
    }
    
    pub async fn health_check(&self) {
        // Check health of all instances
        for instance in &self.instances {
            if !instance.is_healthy().await {
                // Remove from load balancer
                self.load_balancer.remove_instance(instance.instance_id).await;
            }
        }
    }
}
```

### PostgreSQL Database

**Schema Design:**
- All configuration data stored in PostgreSQL
- Normalized schema for data integrity
- Indexes for query performance
- Foreign keys for referential integrity

**Connection Pooling:**
- Primary pool for write operations
- Read replica pool for read operations
- Connection limits per pool
- Automatic connection recovery

**Implementation:**
```rust
pub struct DatabaseManager {
    primary_pool: PgPool,
    read_replica_pools: Vec<PgPool>,
    current_read_index: AtomicUsize,
}

impl DatabaseManager {
    pub async fn write_query(&self, query: &str) -> Result<(), DbError> {
        // Execute on primary database
        sqlx::query(query)
            .execute(&self.primary_pool)
            .await?;
        Ok(())
    }
    
    pub async fn read_query(&self, query: &str) -> Result<Vec<Row>, DbError> {
        // Execute on read replica (round-robin)
        let index = self.current_read_index.fetch_add(1, Ordering::Relaxed);
        let pool = &self.read_replica_pools[index % self.read_replica_pools.len()];
        
        sqlx::query(query)
            .fetch_all(pool)
            .await
    }
}
```

### Eventual Consistency

**Synchronization Strategy:**
- Write operations go to primary database
- Asynchronous replication to read replicas
- Eventual consistency (replicas may lag slightly)
- Conflict resolution for concurrent writes

**Implementation:**
```rust
pub struct ReplicationManager {
    primary_db: PgPool,
    replicas: Vec<ReplicaConfig>,
}

pub struct ReplicaConfig {
    pub replica_id: ReplicaId,
    pub connection_string: String,
    pub lag_threshold: Duration,
    pub last_sync: Instant,
}

impl ReplicationManager {
    pub async fn sync_to_replicas(&self) {
        // PostgreSQL handles replication automatically
        // Monitor replication lag
        for replica in &self.replicas {
            let lag = self.check_replication_lag(replica).await;
            if lag > replica.lag_threshold {
                warn!("Replica {} lagging: {:?}", replica.replica_id, lag);
            }
        }
    }
    
    async fn check_replication_lag(&self, replica: &ReplicaConfig) -> Duration {
        // Query replication lag from PostgreSQL
        // Returns time difference between primary and replica
        Duration::from_millis(0) // Simplified
    }
}
```

## Unified Configuration API

### Purpose

The Unified Configuration API is called whenever a server starts. It handles authentication, token generation, and configuration data distribution to all game servers.

### API Endpoints

**Base URL:** `https://admin-api.gameserver.com/api/v1`

### Authentication and Token Management

#### Server Startup Preload

**Endpoint:** `POST /config/preload`

**Purpose:** Called on server startup to load all configuration data.

**Request:**
```json
{
  "server_id": "server_001",
  "server_version": "1.0.0",
  "startup_timestamp": "2024-01-01T00:00:00Z"
}
```

**Response:**
```json
{
  "server_token": "server_auth_token_xyz",
  "config_data": {
    "items": [...],
    "npcs": [...],
    "quests": [...],
    "zones": [...],
    "events": [...],
    "patrols": [...],
    "creatures": [...]
  },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

**Implementation:**
```rust
pub struct ConfigPreloadAPI {
    db: DatabaseConnection,
    cache: RedisCache,
}

impl ConfigPreloadAPI {
    pub async fn preload_config(
        &self,
        server_id: ServerId,
    ) -> Result<ConfigData, ApiError> {
        // Load all configuration data
        let items = self.load_items().await?;
        let npcs = self.load_npcs().await?;
        let quests = self.load_quests().await?;
        let zones = self.load_zones().await?;
        let events = self.load_events().await?;
        let patrols = self.load_patrols().await?;
        let creatures = self.load_creatures().await?;
        
        // Generate server authentication token
        let server_token = self.generate_server_token(server_id).await?;
        
        Ok(ConfigData {
            server_token,
            config_data: GameConfig {
                items,
                npcs,
                quests,
                zones,
                events,
                patrols,
                creatures,
            },
            timestamp: Utc::now(),
        })
    }
}
```

### Account Authentication

**Endpoint:** `POST /auth/authenticate`

**Purpose:** Authenticate player accounts and generate login tokens.

**Request:**
```json
{
  "account_id": "account_123",
  "authentication_method": "website",
  "credentials": {
    "email": "player@example.com",
    "password": "hashed_password"
  }
}
```

**Response:**
```json
{
  "login_token": "login_token_abc123",
  "token_expires_at": "2024-01-01T01:00:00Z",
  "account_info": {
    "account_id": "account_123",
    "username": "PlayerName",
    "characters": [
      {
        "character_id": "char_001",
        "server_id": "server_001",
        "character_name": "CharacterName",
        "level": 50
      }
    ]
  }
}
```

**Implementation:**
```rust
pub async fn authenticate_account(
    account_id: AccountId,
    auth_method: AuthenticationMethod,
    credentials: AuthCredentials,
) -> Result<AuthResponse, AuthError> {
    // Validate credentials
    let account = validate_credentials(account_id, auth_method, credentials).await?;
    
    // Generate login token
    let login_token = generate_login_token(account_id).await?;
    let token_expires_at = Utc::now() + chrono::Duration::hours(1);
    
    // Get account characters
    let characters = get_account_characters(account_id).await?;
    
    Ok(AuthResponse {
        login_token,
        token_expires_at,
        account_info: AccountInfo {
            account_id,
            username: account.username,
            characters,
        },
    })
}
```

### Character Selection Token

**Endpoint:** `POST /auth/select-character`

**Purpose:** Generate token when player selects character and enters game server.

**Request:**
```json
{
  "login_token": "login_token_abc123",
  "character_id": "char_001",
  "server_id": "server_001"
}
```

**Response:**
```json
{
  "character_token": "character_token_xyz789",
  "token_expires_at": "2024-01-01T02:00:00Z",
  "character_data": {
    "character_id": "char_001",
    "character_name": "CharacterName",
    "level": 50,
    "position": {"x": 1000.0, "y": 2000.0, "z": 500.0},
    "map_id": "map_001"
  }
}
```

**Implementation:**
```rust
pub async fn select_character(
    login_token: LoginToken,
    character_id: CharacterId,
    server_id: ServerId,
) -> Result<CharacterAuthResponse, AuthError> {
    // Validate login token
    let account_id = validate_login_token(login_token).await?;
    
    // Verify character belongs to account
    verify_character_ownership(account_id, character_id).await?;
    
    // Generate character token
    let character_token = generate_character_token(character_id, server_id).await?;
    let token_expires_at = Utc::now() + chrono::Duration::hours(24);
    
    // Load character data
    let character_data = load_character_data(character_id).await?;
    
    Ok(CharacterAuthResponse {
        character_token,
        token_expires_at,
        character_data,
    })
}
```

### Token Revalidation

**Endpoint:** `POST /auth/revalidate`

**Purpose:** Revalidate tokens on game server.

**Request:**
```json
{
  "token": "character_token_xyz789",
  "token_type": "character"
}
```

**Response:**
```json
{
  "valid": true,
  "token_info": {
    "character_id": "char_001",
    "account_id": "account_123",
    "expires_at": "2024-01-01T02:00:00Z"
  }
}
```

### Content Management

#### Item Creation

**Endpoint:** `POST /content/items/create`

**Purpose:** Create new items from admin dashboard. Replicates to all servers.

**Request:**
```json
{
  "item_name": "Sword_Steel",
  "item_type": "weapon",
  "properties": {
    "damage": 100,
    "durability": 1000,
    "rarity": "common"
  },
  "created_by": "admin_001"
}
```

**Response:**
```json
{
  "item_id": "item_sword_steel_001",
  "replicated_to_servers": ["server_001", "server_002", "server_003"],
  "created_at": "2024-01-01T00:00:00Z"
}
```

**Implementation:**
```rust
pub async fn create_item(
    item_data: ItemCreationRequest,
    admin_id: AdminId,
) -> Result<ItemCreationResponse, ApiError> {
    // Validate admin permissions
    verify_admin_permission(admin_id, Permission::CreateItems).await?;
    
    // Create item in database
    let item = create_item_in_database(item_data).await?;
    
    // Replicate to all game servers
    let servers = get_all_active_servers().await?;
    let replication_results = replicate_item_to_servers(&item, &servers).await?;
    
    Ok(ItemCreationResponse {
        item_id: item.item_id,
        replicated_to_servers: replication_results.successful_servers,
        created_at: item.created_at,
    })
}

async fn replicate_item_to_servers(
    item: &Item,
    servers: &[Server],
) -> Result<ReplicationResults, ApiError> {
    let mut successful = Vec::new();
    let mut failed = Vec::new();
    
    for server in servers {
        match send_item_to_server(server, item).await {
            Ok(_) => successful.push(server.server_id.clone()),
            Err(e) => {
                failed.push((server.server_id.clone(), e));
                // Log error but continue with other servers
            }
        }
    }
    
    Ok(ReplicationResults {
        successful_servers: successful,
        failed_servers: failed,
    })
}
```

#### Item Correction

**Endpoint:** `POST /content/items/correct`

**Purpose:** Fix problematic items. Replicates correction to all servers.

**Request:**
```json
{
  "item_id": "item_sword_steel_001",
  "corrections": {
    "damage": 150,
    "durability": 2000
  },
  "reason": "Balance adjustment",
  "corrected_by": "admin_001"
}
```

**Response:**
```json
{
  "item_id": "item_sword_steel_001",
  "corrections_applied": true,
  "replicated_to_servers": ["server_001", "server_002", "server_003"],
  "corrected_at": "2024-01-01T00:00:00Z"
}
```

### Configuration Management

#### Quests Configuration

**Endpoint:** `POST /config/quests/update`

**Purpose:** Update quest configuration. Replicates to all servers.

#### NPCs Configuration

**Endpoint:** `POST /config/npcs/update`

**Purpose:** Update NPC configuration. Replicates to all servers.

#### Zones Configuration

**Endpoint:** `POST /config/zones/update`

**Purpose:** Update zone configuration. Replicates to all servers.

#### Events Configuration

**Endpoint:** `POST /config/events/update`

**Purpose:** Update automatic events configuration. Replicates to all servers.

#### Patrol Configuration

**Endpoint:** `POST /config/patrols/update`

**Purpose:** Update creature patrol routes. Replicates to all servers.

## Administrative Dashboard

### Dashboard Overview

The administrative dashboard is a web-based interface that provides complete control over all game servers and content without requiring login to the game client.

### Server Management

#### Server Control

**Features:**
- View all servers and their status
- Start/stop/restart servers
- View server telemetry
- Recover backups
- Monitor server health

**Implementation:**
```rust
pub struct ServerManagement {
    servers: Arc<RwLock<HashMap<ServerId, ServerInfo>>>,
}

pub struct ServerInfo {
    pub server_id: ServerId,
    pub server_name: String,
    pub status: ServerStatus,
    pub player_count: usize,
    pub max_players: usize,
    pub uptime: Duration,
    pub last_backup: Option<DateTime<Utc>>,
    pub telemetry: ServerTelemetry,
}

#[derive(Debug, Clone)]
pub enum ServerStatus {
    Online,
    Offline,
    Starting,
    Stopping,
    Maintenance,
    Error(String),
}
```

**API Endpoints:**
- `GET /servers` - List all servers
- `GET /servers/{server_id}` - Get server details
- `POST /servers/{server_id}/restart` - Restart server
- `GET /servers/{server_id}/telemetry` - Get server telemetry
- `POST /servers/{server_id}/backup/recover` - Recover from backup

#### Server Telemetry

**Metrics Tracked:**
- CPU usage
- Memory usage
- Network traffic
- Player count
- Active connections
- Database size
- Response times
- Error rates

**Real-Time Updates:**
- WebSocket connection for live telemetry
- Updates every 5 seconds
- Historical data for charts

### Player Management

#### Account Management

**Features:**
- View account details
- Ban/unban accounts
- View account characters
- View account history
- Modify account settings

**API Endpoints:**
- `GET /players/accounts/{account_id}` - Get account details
- `POST /players/accounts/{account_id}/ban` - Ban account
- `POST /players/accounts/{account_id}/unban` - Unban account
- `GET /players/accounts/{account_id}/characters` - Get account characters
- `GET /players/accounts/{account_id}/history` - Get account history

#### Character Management

**Features:**
- View character details
- View character inventory
- View character bank
- Remove items
- Transfer items
- Modify character stats
- Teleport character

**API Endpoints:**
- `GET /players/characters/{character_id}` - Get character details
- `GET /players/characters/{character_id}/inventory` - Get inventory
- `GET /players/characters/{character_id}/bank` - Get bank contents
- `DELETE /players/characters/{character_id}/items/{item_id}` - Remove item
- `POST /players/characters/{character_id}/items/transfer` - Transfer item
- `PUT /players/characters/{character_id}/stats` - Modify stats
- `POST /players/characters/{character_id}/teleport` - Teleport character

#### Chat Monitoring

**Features:**
- View all chat channels
- View whispers (private messages)
- Search chat history
- Filter by player, channel, time

**API Endpoints:**
- `GET /chat/channels` - Get all chat channels
- `GET /chat/whispers` - Get all whispers
- `GET /chat/history` - Get chat history
- `GET /chat/search` - Search chat messages

**Implementation:**
```rust
pub struct ChatMonitoring {
    chat_logs: Arc<RwLock<Vec<ChatMessage>>>,
}

pub struct ChatMessage {
    pub message_id: MessageId,
    pub channel: ChatChannel,
    pub sender_id: CharacterId,
    pub sender_name: String,
    pub receiver_id: Option<CharacterId>, // For whispers
    pub message: String,
    pub timestamp: DateTime<Utc>,
    pub server_id: ServerId,
}
```

### Content Management

#### Respawn Management

**Features:**
- View all respawns
- Create respawns
- Remove respawns
- Configure respawn rates
- Set respawn locations

**API Endpoints:**
- `GET /content/respawns` - List all respawns
- `POST /content/respawns` - Create respawn
- `DELETE /content/respawns/{respawn_id}` - Remove respawn
- `PUT /content/respawns/{respawn_id}` - Update respawn

#### Creature Management

**Features:**
- View all creatures
- Create creatures
- Configure creature properties
- Set creature spawn locations
- Configure creature AI

**API Endpoints:**
- `GET /content/creatures` - List all creatures
- `POST /content/creatures` - Create creature
- `PUT /content/creatures/{creature_id}` - Update creature
- `DELETE /content/creatures/{creature_id}` - Remove creature

#### NPC Management

**Features:**
- View all NPCs
- Create NPCs
- Configure NPC dialogue
- Set NPC locations
- Configure NPC behavior

**API Endpoints:**
- `GET /content/npcs` - List all NPCs
- `POST /content/npcs` - Create NPC
- `PUT /content/npcs/{npc_id}` - Update NPC
- `DELETE /content/npcs/{npc_id}` - Remove NPC

### Interactive Map

#### Map Features

**Displayed Information:**
- Player locations (real-time)
- Creature locations
- NPC locations
- Waypoints
- Active events
- Zone boundaries
- Respawn points

**Interactive Features:**
- Click to view entity details
- Filter by entity type
- Search for specific players/entities
- Zoom in/out
- Pan across map
- Real-time updates via WebSocket

**Implementation:**
```rust
pub struct InteractiveMap {
    map_data: Arc<RwLock<MapData>>,
    entity_positions: Arc<RwLock<HashMap<EntityId, Position>>>,
    websocket_clients: Arc<RwLock<Vec<WebSocket>>>,
}

pub struct MapData {
    pub map_id: MapId,
    pub map_name: String,
    pub bounds: BoundingBox,
    pub zones: Vec<Zone>,
    pub waypoints: Vec<Waypoint>,
    pub events: Vec<ActiveEvent>,
}

pub struct EntityPosition {
    pub entity_id: EntityId,
    pub entity_type: EntityType,
    pub position: Position,
    pub map_id: MapId,
    pub last_update: DateTime<Utc>,
}

#[derive(Debug, Clone)]
pub enum EntityType {
    Player { character_id: CharacterId, character_name: String },
    Creature { creature_id: CreatureId, creature_type: String },
    NPC { npc_id: NpcId, npc_name: String },
}
```

**API Endpoints:**
- `GET /map/{map_id}` - Get map data
- `GET /map/{map_id}/entities` - Get all entities on map
- `WebSocket /map/{map_id}/live` - Real-time entity updates

### Support Ticket System

#### Ticket Management

**Features:**
- View all tickets
- Respond to tickets
- Assign tickets to admins
- Close tickets
- View ticket history

**API Endpoints:**
- `GET /support/tickets` - List all tickets
- `GET /support/tickets/{ticket_id}` - Get ticket details
- `POST /support/tickets/{ticket_id}/respond` - Respond to ticket
- `PUT /support/tickets/{ticket_id}/assign` - Assign ticket
- `POST /support/tickets/{ticket_id}/close` - Close ticket

**Implementation:**
```rust
pub struct SupportTicket {
    pub ticket_id: TicketId,
    pub player_id: CharacterId,
    pub player_name: String,
    pub subject: String,
    pub message: String,
    pub status: TicketStatus,
    pub assigned_to: Option<AdminId>,
    pub responses: Vec<TicketResponse>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

pub struct TicketResponse {
    pub response_id: ResponseId,
    pub responder_id: AdminId,
    pub responder_name: String,
    pub message: String,
    pub timestamp: DateTime<Utc>,
}
```

### Reward System

#### Automatic Rewards

**Features:**
- Create reward campaigns
- Schedule automatic rewards
- Target specific players or groups
- Set reward conditions
- Track reward distribution

**API Endpoints:**
- `POST /rewards/create` - Create reward campaign
- `GET /rewards/campaigns` - List all campaigns
- `POST /rewards/distribute` - Distribute rewards
- `GET /rewards/history` - Get reward history

**Implementation:**
```rust
pub struct RewardCampaign {
    pub campaign_id: CampaignId,
    pub name: String,
    pub description: String,
    pub target_players: RewardTarget,
    pub rewards: Vec<Reward>,
    pub conditions: RewardConditions,
    pub schedule: RewardSchedule,
    pub status: CampaignStatus,
    pub created_by: AdminId,
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Clone)]
pub enum RewardTarget {
    AllPlayers,
    SpecificPlayers(Vec<CharacterId>),
    LevelRange { min: u8, max: u8 },
    Server(ServerId),
    Guild(GuildId),
}

pub struct Reward {
    pub item_id: Option<ItemId>,
    pub item_type: String,
    pub quantity: u32,
    pub gold: Option<i64>,
    pub experience: Option<i64>,
}
```

### Log Analysis

#### Log Viewing

**Features:**
- View security logs
- View activity logs
- View error logs
- Search logs
- Filter by date, player, event type
- Export logs

**API Endpoints:**
- `GET /logs/security` - Get security logs
- `GET /logs/activity` - Get activity logs
- `GET /logs/errors` - Get error logs
- `GET /logs/search` - Search logs
- `POST /logs/export` - Export logs

**Implementation:**
```rust
pub struct LogViewer {
    log_storage: LogStorage,
}

pub struct LogQuery {
    pub log_type: LogType,
    pub start_date: Option<DateTime<Utc>>,
    pub end_date: Option<DateTime<Utc>>,
    pub player_id: Option<CharacterId>,
    pub event_type: Option<EventType>,
    pub server_id: Option<ServerId>,
    pub limit: Option<usize>,
}

#[derive(Debug, Clone)]
pub enum LogType {
    Security,
    Activity,
    Error,
    Chat,
    Transaction,
}
```

## Security and Permissions

### Permission System

**Permission Levels:**
- **Super Admin**: Full access to all features
- **Admin**: Server management, player management, content management
- **Moderator**: Player management, support tickets, log viewing
- **Support**: Support tickets, log viewing (read-only)

**Permission Checks:**
```rust
pub struct PermissionSystem {
    admin_permissions: Arc<RwLock<HashMap<AdminId, Vec<Permission>>>>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Permission {
    // Server Management
    RestartServers,
    ViewTelemetry,
    RecoverBackups,
    
    // Player Management
    BanAccounts,
    UnbanAccounts,
    ViewPlayerData,
    ModifyPlayerData,
    ViewChat,
    
    // Content Management
    CreateItems,
    ModifyItems,
    CreateRespawns,
    RemoveRespawns,
    CreateCreatures,
    CreateNPCs,
    
    // Support
    RespondTickets,
    AssignTickets,
    
    // Rewards
    CreateRewards,
    DistributeRewards,
    
    // Logs
    ViewLogs,
    ExportLogs,
}

pub async fn check_permission(
    admin_id: AdminId,
    permission: Permission,
) -> Result<(), PermissionError> {
    let permissions = get_admin_permissions(admin_id).await?;
    
    if permissions.contains(&permission) {
        Ok(())
    } else {
        Err(PermissionError::InsufficientPermissions)
    }
}
```

## API Authentication

### Admin Authentication

**Endpoint:** `POST /admin/auth/login`

**Request:**
```json
{
  "admin_id": "admin_001",
  "password": "hashed_password"
}
```

**Response:**
```json
{
  "admin_token": "admin_token_xyz",
  "permissions": ["RestartServers", "BanAccounts", "CreateItems"],
  "expires_at": "2024-01-01T01:00:00Z"
}
```

## Real-Time Updates

### Server-Sent Events (SSE)

**Architecture:**
All game servers connect to the central API and send events via Server-Sent Events (SSE). The API acts as an event hub, receiving events from servers and streaming them to connected admin clients.

**Server Connection:**
Game servers establish persistent SSE connections to the central API and send events as they occur.

**Client Connection:**
Admin dashboard clients connect to SSE endpoints to receive real-time updates.

### SSE Endpoints

**Server Event Stream:**
- `GET /events/servers` - Server status updates
- `GET /events/telemetry/{server_id}` - Server telemetry updates
- `GET /events/map/{map_id}` - Map entity updates
- `GET /events/chat` - Chat message updates
- `GET /events/tickets` - Support ticket updates
- `GET /events/players` - Player activity updates
- `GET /events/security` - Security incident updates

**Implementation:**
```rust
use axum::response::sse::{Event, Sse};
use futures::stream::{self, Stream};

pub struct EventStream {
    event_sender: broadcast::Sender<ServerEvent>,
}

#[derive(Debug, Clone, Serialize)]
pub enum ServerEvent {
    ServerStatusUpdate {
        server_id: ServerId,
        status: ServerStatus,
        player_count: usize,
    },
    TelemetryUpdate {
        server_id: ServerId,
        telemetry: ServerTelemetry,
    },
    MapEntityUpdate {
        map_id: MapId,
        entity_id: EntityId,
        entity_type: EntityType,
        position: Position,
    },
    ChatMessage {
        server_id: ServerId,
        channel: ChatChannel,
        sender_id: CharacterId,
        sender_name: String,
        message: String,
    },
    TicketUpdate {
        ticket_id: TicketId,
        status: TicketStatus,
    },
    PlayerActivity {
        server_id: ServerId,
        character_id: CharacterId,
        activity_type: ActivityType,
    },
    SecurityIncident {
        server_id: ServerId,
        incident_type: IncidentType,
        player_id: CharacterId,
    },
}

// Server-side: Game servers send events to central API
pub async fn send_event_to_api(
    api_url: &str,
    event: ServerEvent,
) -> Result<(), ApiError> {
    let client = reqwest::Client::new();
    
    // Send event via POST to API
    client
        .post(&format!("{}/api/v1/events/receive", api_url))
        .json(&event)
        .send()
        .await?;
    
    Ok(())
}

// API-side: Receive events from game servers
pub async fn receive_server_event(
    event: ServerEvent,
    event_broadcast: broadcast::Sender<ServerEvent>,
) -> Result<(), ApiError> {
    // Broadcast event to all connected admin clients
    event_broadcast.send(event)?;
    Ok(())
}

// API-side: Stream events to admin clients
pub async fn stream_events(
    event_receiver: broadcast::Receiver<ServerEvent>,
) -> Sse<impl Stream<Item = Result<Event, axum::Error>>> {
    let stream = stream::unfold(event_receiver, |mut receiver| async move {
        match receiver.recv().await {
            Ok(event) => {
                let event = Event::default()
                    .json_data(event)
                    .map_err(|e| axum::Error::from(e))?;
                Some((Ok(event), receiver))
            }
            Err(_) => None,
        }
    });
    
    Sse::new(stream)
}
```

### Server Event Sending

**Game Server Implementation:**
```rust
pub struct GameServerEventSender {
    api_url: String,
    server_id: ServerId,
    event_queue: Arc<RwLock<VecDeque<ServerEvent>>>,
    sender_thread: Option<JoinHandle<()>>,
}

impl GameServerEventSender {
    pub fn new(api_url: String, server_id: ServerId) -> Self {
        Self {
            api_url,
            server_id,
            event_queue: Arc::new(RwLock::new(VecDeque::new())),
            sender_thread: None,
        }
    }
    
    pub fn start(&mut self) {
        let api_url = self.api_url.clone();
        let server_id = self.server_id.clone();
        let queue = Arc::clone(&self.event_queue);
        
        self.sender_thread = Some(tokio::spawn(async move {
            loop {
                // Process event queue
                while let Some(event) = queue.write().await.pop_front() {
                    if let Err(e) = Self::send_event_to_api(&api_url, &server_id, event).await {
                        error!("Failed to send event to API: {:?}", e);
                        // Retry logic could be added here
                    }
                }
                
                tokio::time::sleep(Duration::from_millis(100)).await;
            }
        }));
    }
    
    pub async fn send_event(&self, event: ServerEvent) {
        self.event_queue.write().await.push_back(event);
    }
    
    async fn send_event_to_api(
        api_url: &str,
        server_id: &ServerId,
        event: ServerEvent,
    ) -> Result<(), ApiError> {
        let client = reqwest::Client::new();
        
        let response = client
            .post(&format!("{}/api/v1/events/receive", api_url))
            .header("X-Server-ID", server_id.to_string())
            .header("X-Server-Token", get_server_token().await?)
            .json(&event)
            .send()
            .await?;
        
        if !response.status().is_success() {
            return Err(ApiError::HttpError(response.status()));
        }
        
        Ok(())
    }
}

// Usage in game server
impl GameServer {
    pub async fn on_player_join(&self, player: CharacterId) {
        // Send event to API
        self.event_sender.send_event(ServerEvent::PlayerActivity {
            server_id: self.server_id.clone(),
            character_id: player,
            activity_type: ActivityType::PlayerJoined,
        }).await;
    }
    
    pub async fn on_chat_message(
        &self,
        channel: ChatChannel,
        sender: CharacterId,
        message: String,
    ) {
        // Send event to API
        self.event_sender.send_event(ServerEvent::ChatMessage {
            server_id: self.server_id.clone(),
            channel,
            sender_id: sender,
            sender_name: get_character_name(sender).await,
            message,
        }).await;
    }
    
    pub async fn on_telemetry_update(&self, telemetry: ServerTelemetry) {
        // Send telemetry update every 5 seconds
        self.event_sender.send_event(ServerEvent::TelemetryUpdate {
            server_id: self.server_id.clone(),
            telemetry,
        }).await;
    }
}
```

### Admin Client Connection

**Dashboard Implementation:**
```typescript
// Admin dashboard connects to SSE endpoint
class EventStream {
    private eventSource: EventSource;
    
    constructor(endpoint: string, adminToken: string) {
        this.eventSource = new EventSource(
            `${endpoint}?token=${adminToken}`
        );
        
        this.setupEventHandlers();
    }
    
    private setupEventHandlers() {
        // Listen for server events
        this.eventSource.addEventListener('server-status', (e) => {
            const event = JSON.parse(e.data);
            this.handleServerStatusUpdate(event);
        });
        
        this.eventSource.addEventListener('telemetry', (e) => {
            const event = JSON.parse(e.data);
            this.handleTelemetryUpdate(event);
        });
        
        this.eventSource.addEventListener('map-entity', (e) => {
            const event = JSON.parse(e.data);
            this.handleMapEntityUpdate(event);
        });
        
        this.eventSource.addEventListener('chat', (e) => {
            const event = JSON.parse(e.data);
            this.handleChatMessage(event);
        });
        
        this.eventSource.addEventListener('ticket', (e) => {
            const event = JSON.parse(e.data);
            this.handleTicketUpdate(event);
        });
        
        this.eventSource.addEventListener('security', (e) => {
            const event = JSON.parse(e.data);
            this.handleSecurityIncident(event);
        });
        
        this.eventSource.onerror = (error) => {
            console.error('SSE connection error:', error);
            // Implement reconnection logic
        };
    }
    
    private handleServerStatusUpdate(event: ServerStatusUpdate) {
        // Update server status in dashboard
        updateServerStatus(event.server_id, event.status);
    }
    
    private handleTelemetryUpdate(event: TelemetryUpdate) {
        // Update telemetry charts
        updateTelemetryChart(event.server_id, event.telemetry);
    }
    
    private handleMapEntityUpdate(event: MapEntityUpdate) {
        // Update map with entity position
        updateMapEntity(event.map_id, event.entity_id, event.position);
    }
    
    private handleChatMessage(event: ChatMessage) {
        // Add message to chat monitor
        addChatMessage(event);
    }
    
    private handleTicketUpdate(event: TicketUpdate) {
        // Update ticket status
        updateTicket(event.ticket_id, event.status);
    }
    
    private handleSecurityIncident(event: SecurityIncident) {
        // Show security alert
        showSecurityAlert(event);
    }
    
    public close() {
        this.eventSource.close();
    }
}
```

### Event Flow

**Architecture:**
```
Game Server → POST /api/v1/events/receive → Central API
                                              ↓
                                    Event Broadcast Channel
                                              ↓
                                    Admin Clients (SSE)
```

**Flow:**
1. Game server generates event (e.g., player joins, chat message, telemetry)
2. Game server sends event via POST to central API
3. Central API receives event and validates server token
4. Central API broadcasts event to all connected admin clients via SSE
5. Admin dashboard receives event and updates UI in real-time

### Event Types

**Server Status Events:**
- Server started
- Server stopped
- Server restarting
- Player count changes
- Server health status

**Telemetry Events:**
- CPU usage updates
- Memory usage updates
- Network traffic updates
- Database metrics updates

**Map Events:**
- Player position updates
- Creature spawn/despawn
- NPC position updates
- Event start/end
- Waypoint updates

**Chat Events:**
- New chat message
- Whisper sent/received
- Channel join/leave

**Ticket Events:**
- New ticket created
- Ticket status changed
- Ticket response added
- Ticket assigned

**Security Events:**
- Security violation detected
- Player banned
- Bot detected
- Hack detected

### Connection Management

**Server Connection:**
- Game servers maintain persistent connection to API
- Automatic reconnection on disconnect
- Heartbeat to keep connection alive
- Token validation on each event

**Client Connection:**
- Admin clients connect via SSE
- Automatic reconnection on disconnect
- Token-based authentication
- Filtered events based on permissions

## Backup System

### Zero Data Loss Guarantee

**Critical Requirement:**
- **Nothing can be lost**: All data must be backed up
- **Periodic full backups**: Complete backups of files and database
- **Cloud storage**: S3 or Digital Ocean Spaces for backup storage
- **File transfer**: Secure transfer between servers and cloud storage

### Backup Architecture

**Components:**
- Central backup coordinator
- Cloud storage (S3/Digital Ocean Spaces)
- File transfer system
- Backup scheduling system
- Backup verification system

**Architecture:**
```
Game Servers → Backup Coordinator → Cloud Storage (S3/DO Spaces)
     ↓              ↓
SQLite DBs    PostgreSQL DB
     ↓              ↓
  Files         Files
```

### Cloud Storage Integration

**Supported Providers:**
- Amazon S3
- Digital Ocean Spaces
- Compatible S3-compatible storage

**Implementation:**
```rust
use aws_sdk_s3::Client as S3Client;
use aws_sdk_s3::primitives::ByteStream;

pub struct CloudStorage {
    client: S3Client,
    bucket_name: String,
    provider: StorageProvider,
}

#[derive(Debug, Clone)]
pub enum StorageProvider {
    AmazonS3 {
        region: String,
        access_key: String,
        secret_key: String,
    },
    DigitalOceanSpaces {
        endpoint: String,
        region: String,
        access_key: String,
        secret_key: String,
    },
}

impl CloudStorage {
    pub async fn upload_backup(
        &self,
        backup_path: &Path,
        backup_name: &str,
    ) -> Result<String, BackupError> {
        let file_data = tokio::fs::read(backup_path).await?;
        let key = format!("backups/{}/{}", Utc::now().format("%Y%m%d"), backup_name);
        
        self.client
            .put_object()
            .bucket(&self.bucket_name)
            .key(&key)
            .body(ByteStream::from(file_data))
            .send()
            .await?;
        
        Ok(key)
    }
    
    pub async fn download_backup(
        &self,
        backup_key: &str,
        destination: &Path,
    ) -> Result<(), BackupError> {
        let response = self.client
            .get_object()
            .bucket(&self.bucket_name)
            .key(backup_key)
            .send()
            .await?;
        
        let data = response.body.collect().await?.into_bytes();
        tokio::fs::write(destination, data).await?;
        
        Ok(())
    }
    
    pub async fn list_backups(&self, prefix: &str) -> Result<Vec<String>, BackupError> {
        let response = self.client
            .list_objects_v2()
            .bucket(&self.bucket_name)
            .prefix(prefix)
            .send()
            .await?;
        
        let keys: Vec<String> = response.contents()
            .iter()
            .filter_map(|obj| obj.key().map(|k| k.to_string()))
            .collect();
        
        Ok(keys)
    }
    
    pub async fn delete_old_backups(&self, retention_days: u32) -> Result<(), BackupError> {
        let cutoff_date = Utc::now() - chrono::Duration::days(retention_days as i64);
        let prefix = format!("backups/{}", cutoff_date.format("%Y%m%d"));
        
        let backups = self.list_backups(&prefix).await?;
        
        for backup_key in backups {
            self.client
                .delete_object()
                .bucket(&self.bucket_name)
                .key(&backup_key)
                .send()
                .await?;
        }
        
        Ok(())
    }
}
```

### File Transfer System

**Purpose:**
Transfer backup files between game servers and central backup coordinator, and to/from cloud storage.

**Implementation:**
```rust
pub struct FileTransferSystem {
    sftp_client: Option<SftpClient>,
    http_client: reqwest::Client,
    cloud_storage: CloudStorage,
}

impl FileTransferSystem {
    pub async fn transfer_from_server(
        &self,
        server_id: ServerId,
        source_path: &Path,
        destination_path: &Path,
    ) -> Result<(), TransferError> {
        // Option 1: SFTP transfer
        if let Some(ref sftp) = self.sftp_client {
            sftp.download(server_id, source_path, destination_path).await?;
        }
        // Option 2: HTTP transfer
        else {
            let url = format!("https://{}/backup/{}", server_id, source_path.display());
            let response = self.http_client.get(&url).send().await?;
            let data = response.bytes().await?;
            tokio::fs::write(destination_path, data).await?;
        }
        
        Ok(())
    }
    
    pub async fn transfer_to_cloud(
        &self,
        local_path: &Path,
        cloud_key: &str,
    ) -> Result<(), TransferError> {
        self.cloud_storage.upload_backup(local_path, cloud_key).await?;
        Ok(())
    }
    
    pub async fn transfer_from_cloud(
        &self,
        cloud_key: &str,
        local_path: &Path,
    ) -> Result<(), TransferError> {
        self.cloud_storage.download_backup(cloud_key, local_path).await?;
        Ok(())
    }
}
```

### Backup Coordinator

**Responsibilities:**
- Schedule backups
- Coordinate backups across all servers
- Transfer files to cloud storage
- Verify backup integrity
- Manage backup retention

**Implementation:**
```rust
pub struct BackupCoordinator {
    servers: Vec<ServerConfig>,
    cloud_storage: CloudStorage,
    file_transfer: FileTransferSystem,
    schedule: BackupSchedule,
}

pub struct BackupSchedule {
    pub database_backup_interval: Duration, // Every 1 hour
    pub file_backup_interval: Duration, // Every 6 hours
    pub full_backup_interval: Duration, // Every 24 hours
    pub retention_days: u32, // Keep backups for 30 days
}

impl BackupCoordinator {
    pub async fn start(&mut self) {
        // Start database backup scheduler
        self.start_database_backup_scheduler().await;
        
        // Start file backup scheduler
        self.start_file_backup_scheduler().await;
        
        // Start full backup scheduler
        self.start_full_backup_scheduler().await;
        
        // Start cleanup scheduler
        self.start_cleanup_scheduler().await;
    }
    
    async fn start_database_backup_scheduler(&self) {
        let interval = self.schedule.database_backup_interval;
        let servers = self.servers.clone();
        let cloud_storage = self.cloud_storage.clone();
        let file_transfer = self.file_transfer.clone();
        
        tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Backup all game server databases
                for server in &servers {
                    Self::backup_server_database(
                        server,
                        &cloud_storage,
                        &file_transfer,
                    ).await;
                }
                
                // Backup central PostgreSQL database
                Self::backup_postgresql_database(&cloud_storage).await;
            }
        });
    }
    
    async fn start_file_backup_scheduler(&self) {
        let interval = self.schedule.file_backup_interval;
        let servers = self.servers.clone();
        let cloud_storage = self.cloud_storage.clone();
        let file_transfer = self.file_transfer.clone();
        
        tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Backup server files
                for server in &servers {
                    Self::backup_server_files(
                        server,
                        &cloud_storage,
                        &file_transfer,
                    ).await;
                }
            }
        });
    }
    
    async fn start_full_backup_scheduler(&self) {
        let interval = self.schedule.full_backup_interval;
        let servers = self.servers.clone();
        let cloud_storage = self.cloud_storage.clone();
        let file_transfer = self.file_transfer.clone();
        
        tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Full backup of everything
                for server in &servers {
                    Self::full_backup_server(
                        server,
                        &cloud_storage,
                        &file_transfer,
                    ).await;
                }
                
                // Full backup of central system
                Self::full_backup_central_system(&cloud_storage).await;
            }
        });
    }
    
    async fn backup_server_database(
        server: &ServerConfig,
        cloud_storage: &CloudStorage,
        file_transfer: &FileTransferSystem,
    ) {
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_name = format!("{}_{}_database.db", server.server_id, timestamp);
        let local_path = Path::new("/tmp/backups").join(&backup_name);
        
        // Request backup from game server
        match request_server_backup(server, &local_path).await {
            Ok(_) => {
                // Transfer to cloud storage
                let cloud_key = format!("servers/{}/database/{}", server.server_id, backup_name);
                if let Err(e) = cloud_storage.upload_backup(&local_path, &cloud_key).await {
                    error!("Failed to upload database backup: {:?}", e);
                } else {
                    info!("Database backup uploaded: {}", cloud_key);
                }
                
                // Clean up local file
                let _ = tokio::fs::remove_file(&local_path).await;
            }
            Err(e) => {
                error!("Failed to backup server database: {:?}", e);
            }
        }
    }
    
    async fn backup_postgresql_database(cloud_storage: &CloudStorage) {
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_name = format!("postgresql_{}.sql", timestamp);
        let local_path = Path::new("/tmp/backups").join(&backup_name);
        
        // Create PostgreSQL dump
        let output = tokio::process::Command::new("pg_dump")
            .arg("--host=localhost")
            .arg("--dbname=game_db")
            .arg("--username=admin")
            .arg("--file")
            .arg(&local_path)
            .output()
            .await
            .expect("Failed to execute pg_dump");
        
        if output.status.success() {
            // Transfer to cloud storage
            let cloud_key = format!("central/postgresql/{}", backup_name);
            if let Err(e) = cloud_storage.upload_backup(&local_path, &cloud_key).await {
                error!("Failed to upload PostgreSQL backup: {:?}", e);
            } else {
                info!("PostgreSQL backup uploaded: {}", cloud_key);
            }
            
            // Clean up local file
            let _ = tokio::fs::remove_file(&local_path).await;
        } else {
            error!("PostgreSQL dump failed: {:?}", String::from_utf8_lossy(&output.stderr));
        }
    }
    
    async fn backup_server_files(
        server: &ServerConfig,
        cloud_storage: &CloudStorage,
        file_transfer: &FileTransferSystem,
    ) {
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_name = format!("{}_{}_files.tar.gz", server.server_id, timestamp);
        let local_path = Path::new("/tmp/backups").join(&backup_name);
        
        // Create tar archive of server files
        let output = tokio::process::Command::new("tar")
            .arg("-czf")
            .arg(&local_path)
            .arg("-C")
            .arg(&server.data_directory)
            .arg(".")
            .output()
            .await
            .expect("Failed to create tar archive");
        
        if output.status.success() {
            // Transfer to cloud storage
            let cloud_key = format!("servers/{}/files/{}", server.server_id, backup_name);
            if let Err(e) = cloud_storage.upload_backup(&local_path, &cloud_key).await {
                error!("Failed to upload file backup: {:?}", e);
            } else {
                info!("File backup uploaded: {}", cloud_key);
            }
            
            // Clean up local file
            let _ = tokio::fs::remove_file(&local_path).await;
        }
    }
    
    async fn full_backup_server(
        server: &ServerConfig,
        cloud_storage: &CloudStorage,
        file_transfer: &FileTransferSystem,
    ) {
        // Full backup includes database + files + configuration
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_name = format!("{}_{}_full.tar.gz", server.server_id, timestamp);
        let local_path = Path::new("/tmp/backups").join(&backup_name);
        
        // Create full backup archive
        // Includes: database, files, logs, configuration
        // Transfer to cloud storage
        let cloud_key = format!("servers/{}/full/{}", server.server_id, backup_name);
        cloud_storage.upload_backup(&local_path, &cloud_key).await.ok();
    }
    
    async fn full_backup_central_system(cloud_storage: &CloudStorage) {
        // Full backup of central API system
        // Includes: PostgreSQL database, configuration files, logs
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_name = format!("central_{}_full.tar.gz", timestamp);
        let local_path = Path::new("/tmp/backups").join(&backup_name);
        
        // Create full backup archive
        // Transfer to cloud storage
        let cloud_key = format!("central/full/{}", backup_name);
        cloud_storage.upload_backup(&local_path, &cloud_key).await.ok();
    }
    
    async fn start_cleanup_scheduler(&self) {
        let retention_days = self.schedule.retention_days;
        let cloud_storage = self.cloud_storage.clone();
        
        tokio::spawn(async move {
            loop {
                // Run cleanup daily
                tokio::time::sleep(Duration::from_secs(86400)).await;
                
                // Delete old backups from cloud storage
                if let Err(e) = cloud_storage.delete_old_backups(retention_days).await {
                    error!("Failed to cleanup old backups: {:?}", e);
                }
            }
        });
    }
}
```

### Backup Verification

**Integrity Checks:**
- Verify backup file integrity
- Test database restore
- Verify file completeness
- Check backup size consistency

**Implementation:**
```rust
pub struct BackupVerification {
    cloud_storage: CloudStorage,
}

impl BackupVerification {
    pub async fn verify_backup(&self, backup_key: &str) -> Result<BackupIntegrity, VerifyError> {
        // Download backup
        let temp_path = Path::new("/tmp/verify").join(backup_key);
        self.cloud_storage.download_backup(backup_key, &temp_path).await?;
        
        // Verify file integrity
        let file_size = tokio::fs::metadata(&temp_path).await?.len();
        let checksum = calculate_checksum(&temp_path).await?;
        
        // If database backup, test restore
        if backup_key.contains("database") {
            self.test_database_restore(&temp_path).await?;
        }
        
        // Clean up
        tokio::fs::remove_file(&temp_path).await.ok();
        
        Ok(BackupIntegrity {
            file_size,
            checksum,
            verified_at: Utc::now(),
        })
    }
    
    async fn test_database_restore(&self, backup_path: &Path) -> Result<(), VerifyError> {
        // Test restore to temporary database
        // Verify data integrity
        Ok(())
    }
}
```

### Backup Recovery

**Recovery Process:**
- List available backups
- Select backup to restore
- Download from cloud storage
- Restore to server
- Verify restoration

**API Endpoint:**
- `POST /backup/recover` - Recover from backup

## Summary

The administrative system provides:

- **Unified Configuration API**: Central API for server configuration and authentication
- **Token Management**: Secure token system for player authentication
- **Web-Based Dashboard**: Complete administrative control via web interface
- **Server Management**: Full control over all game servers
- **Player Management**: Complete account and character management
- **Content Management**: Create and configure all game content
- **Interactive Maps**: Real-time visualization of game world
- **Support System**: Ticket management and response
- **Reward System**: Automatic reward distribution
- **Log Analysis**: Comprehensive log viewing and analysis
- **Chat Monitoring**: View all chat including whispers
- **No Game Client Required**: All management via web interface

**Key Features:**
- **Centralized Configuration**: All configs managed in one place
- **Real-Time Updates**: Live monitoring and updates
- **Comprehensive Control**: Full administrative capabilities
- **Secure Access**: Permission-based access control
- **Efficient Management**: Manage multiple servers from one interface

This system ensures efficient game management while maintaining security and providing administrators with all necessary tools to manage the game effectively.

