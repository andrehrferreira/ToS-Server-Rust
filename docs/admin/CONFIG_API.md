# Unified Configuration API

## Overview

The Unified Configuration API is called whenever a server starts. It handles authentication, token generation, and configuration data distribution to all game servers. All configuration changes made in the admin dashboard are replicated to all game servers through this API.

## API Endpoints

**Base URL:** `https://admin-api.gameserver.com/api/v1`

## Server Startup Preload

### Endpoint: `POST /config/preload`

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

## Content Management

### Item Creation

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

### Item Correction

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

## Configuration Management

### Quests Configuration

**Endpoint:** `POST /config/quests/update`

**Purpose:** Update quest configuration. Replicates to all servers.

**Request:**
```json
{
  "quest_id": "quest_001",
  "updates": {
    "name": "New Quest Name",
    "rewards": [...],
    "requirements": [...]
  }
}
```

### NPCs Configuration

**Endpoint:** `POST /config/npcs/update`

**Purpose:** Update NPC configuration. Replicates to all servers.

**Request:**
```json
{
  "npc_id": "npc_001",
  "updates": {
    "dialogue": [...],
    "location": {"x": 1000, "y": 2000, "z": 500},
    "behavior": {...}
  }
}
```

### Zones Configuration

**Endpoint:** `POST /config/zones/update`

**Purpose:** Update zone configuration. Replicates to all servers.

**Request:**
```json
{
  "zone_id": "zone_001",
  "updates": {
    "resistance_penalty": 0.25,
    "pvp_enabled": true,
    "boundaries": [...]
  }
}
```

### Events Configuration

**Endpoint:** `POST /config/events/update`

**Purpose:** Update automatic events configuration. Replicates to all servers.

**Request:**
```json
{
  "event_id": "event_001",
  "updates": {
    "schedule": {...},
    "rules": {...},
    "rewards": [...]
  }
}
```

### Patrol Configuration

**Endpoint:** `POST /config/patrols/update`

**Purpose:** Update creature patrol routes. Replicates to all servers.

**Request:**
```json
{
  "patrol_id": "patrol_001",
  "updates": {
    "route": [...],
    "creature_type": "Goblin",
    "spawn_rate": 0.5
  }
}
```

## Replication System

### Replication Process

1. Configuration change made in admin dashboard
2. Change saved to PostgreSQL database
3. Change replicated to all active game servers
4. Servers update their in-memory configuration
5. Replication status reported back

### Replication Status

**Tracking:**
- Which servers received update
- Which servers failed to receive update
- Retry mechanism for failed servers
- Replication timestamp

**Implementation:**
```rust
pub struct ReplicationTracker {
    replications: Arc<RwLock<HashMap<ReplicationId, ReplicationStatus>>>,
}

pub struct ReplicationStatus {
    pub replication_id: ReplicationId,
    pub config_type: ConfigType,
    pub config_id: String,
    pub successful_servers: Vec<ServerId>,
    pub failed_servers: Vec<(ServerId, String)>,
    pub timestamp: DateTime<Utc>,
}
```

## Summary

The Unified Configuration API provides:

- **Server Startup Preload**: All configuration loaded on server start
- **Content Creation**: Create items, NPCs, quests, etc. from dashboard
- **Content Updates**: Update existing content and replicate changes
- **Replication**: All changes replicated to all game servers
- **Status Tracking**: Track replication success/failure

This system ensures all game servers have consistent configuration data while allowing centralized management through the admin dashboard.

