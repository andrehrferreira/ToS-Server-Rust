# Persistence Architecture Documentation

## Overview

The persistence architecture is designed to maximize performance and minimize latency by using an in-memory data model with SQLite as a backup mechanism. Each server operates independently with its own local SQLite database, limiting player capacity to 10,000 players per server. This design ensures optimal query performance, protects server economies, and enables regional server deployment for reduced latency.

## Key Design Principles

- **In-Memory Primary Storage**: All active data loaded into memory at server startup
- **SQLite as Backup**: SQLite serves as persistent backup, not primary query source
- **Server Isolation**: Each server has its own independent database
- **Player Capacity Limits**: Maximum 10,000 players per server
- **Regional Deployment**: Servers deployed regionally for low latency
- **Economy Protection**: Player limits protect server economies from inflation
- **Zero Data Loss**: Player and item data NEVER lost (critical requirement)
- **Frequent Snapshots**: Complete snapshots every 2 minutes
- **Regular Backups**: Database backup every 1 hour
- **Background Processing**: All saves in separate thread (no gameplay impact)
- **Delta Updates**: Only update active players for efficiency
- **Guaranteed Inserts**: All item creation guaranteed with async queues
- **Recovery System**: Automatic recovery from backups on corruption/crash

## Architecture Overview

### Data Flow

```
Server Startup:
SQLite Database → Load All Data → In-Memory Structures → Ready to Serve

Runtime:
Client Requests → In-Memory Queries → Instant Response
Periodic Saves → In-Memory → SQLite Database (Backup)

Server Shutdown:
In-Memory Data → Save to SQLite → Graceful Shutdown
```

### Storage Layers

**Layer 1: In-Memory (Primary)**
- All active game data
- Optimized data structures
- Fastest access (nanoseconds)
- Volatile (lost on crash)

**Layer 2: SQLite (Backup)**
- Persistent storage
- Periodic saves
- Crash recovery
- Backup mechanism

## SQLite Database Structure

### Per-Server Database

Each server maintains its own SQLite database file:

```
Server_001.db
Server_002.db
Server_003.db
...
```

### Database Schema

**Characters Table:**
```sql
CREATE TABLE characters (
    character_id INTEGER PRIMARY KEY,
    account_id INTEGER NOT NULL,
    character_name TEXT UNIQUE NOT NULL,
    level INTEGER NOT NULL DEFAULT 1,
    experience INTEGER NOT NULL DEFAULT 0,
    str INTEGER NOT NULL DEFAULT 10,
    dex INTEGER NOT NULL DEFAULT 10,
    int INTEGER NOT NULL DEFAULT 10,
    con INTEGER NOT NULL DEFAULT 10,
    lck INTEGER NOT NULL DEFAULT 10,
    cha INTEGER NOT NULL DEFAULT 10,
    gold INTEGER NOT NULL DEFAULT 0,
    position_x REAL NOT NULL,
    position_y REAL NOT NULL,
    position_z REAL NOT NULL,
    map_id TEXT NOT NULL,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_characters_account ON characters(account_id);
CREATE INDEX idx_characters_name ON characters(character_name);
CREATE INDEX idx_characters_map ON characters(map_id);
```

**Inventory Table:**
```sql
CREATE TABLE inventory (
    inventory_id INTEGER PRIMARY KEY,
    character_id INTEGER NOT NULL,
    item_id TEXT NOT NULL,
    slot INTEGER NOT NULL,
    quantity INTEGER NOT NULL DEFAULT 1,
    properties TEXT, -- JSON serialized properties
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE
);

CREATE INDEX idx_inventory_character ON inventory(character_id);
CREATE INDEX idx_inventory_slot ON inventory(character_id, slot);
```

**Equipment Table:**
```sql
CREATE TABLE equipment (
    equipment_id INTEGER PRIMARY KEY,
    character_id INTEGER NOT NULL,
    slot_type TEXT NOT NULL, -- head, chest, legs, etc.
    item_id TEXT NOT NULL,
    properties TEXT, -- JSON serialized properties
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE
);

CREATE INDEX idx_equipment_character ON equipment(character_id);
```

**Skills Table:**
```sql
CREATE TABLE skills (
    skill_id INTEGER PRIMARY KEY,
    character_id INTEGER NOT NULL,
    skill_name TEXT NOT NULL,
    skill_level INTEGER NOT NULL DEFAULT 0,
    experience INTEGER NOT NULL DEFAULT 0,
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE,
    UNIQUE(character_id, skill_name)
);

CREATE INDEX idx_skills_character ON skills(character_id);
```

**Quests Table:**
```sql
CREATE TABLE quests (
    quest_id INTEGER PRIMARY KEY,
    character_id INTEGER NOT NULL,
    quest_name TEXT NOT NULL,
    quest_status TEXT NOT NULL, -- active, completed, failed
    progress TEXT, -- JSON serialized progress
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE,
    UNIQUE(character_id, quest_name)
);

CREATE INDEX idx_quests_character ON quests(character_id);
```

**Guilds Table:**
```sql
CREATE TABLE guilds (
    guild_id INTEGER PRIMARY KEY,
    guild_name TEXT UNIQUE NOT NULL,
    leader_id INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (leader_id) REFERENCES characters(character_id)
);

CREATE INDEX idx_guilds_leader ON guilds(leader_id);
```

**Guild Members Table:**
```sql
CREATE TABLE guild_members (
    member_id INTEGER PRIMARY KEY,
    guild_id INTEGER NOT NULL,
    character_id INTEGER NOT NULL,
    rank TEXT NOT NULL DEFAULT 'member',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (guild_id) REFERENCES guilds(guild_id) ON DELETE CASCADE,
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE,
    UNIQUE(guild_id, character_id)
);

CREATE INDEX idx_guild_members_guild ON guild_members(guild_id);
CREATE INDEX idx_guild_members_character ON guild_members(character_id);
```

## In-Memory Data Structures

### Character Storage

```rust
use std::collections::HashMap;
use std::sync::Arc;
use tokio::sync::RwLock;

pub struct CharacterCache {
    // Primary lookup: character_id -> Character
    characters: Arc<RwLock<HashMap<CharacterId, Character>>>,
    
    // Secondary indexes for fast queries
    by_account: Arc<RwLock<HashMap<AccountId, Vec<CharacterId>>>>,
    by_name: Arc<RwLock<HashMap<String, CharacterId>>>,
    by_map: Arc<RwLock<HashMap<MapId, Vec<CharacterId>>>>,
    
    // Player count tracking
    player_count: Arc<RwLock<usize>>,
    max_players: usize, // 10,000
}

impl CharacterCache {
    pub fn new(max_players: usize) -> Self {
        Self {
            characters: Arc::new(RwLock::new(HashMap::new())),
            by_account: Arc::new(RwLock::new(HashMap::new())),
            by_name: Arc::new(RwLock::new(HashMap::new())),
            by_map: Arc::new(RwLock::new(HashMap::new())),
            player_count: Arc::new(RwLock::new(0)),
            max_players,
        }
    }
    
    pub async fn can_create_character(&self) -> bool {
        let count = *self.player_count.read().await;
        count < self.max_players
    }
    
    pub async fn get_character(&self, id: CharacterId) -> Option<Character> {
        self.characters.read().await.get(&id).cloned()
    }
    
    pub async fn get_character_by_name(&self, name: &str) -> Option<Character> {
        let name_map = self.by_name.read().await;
        name_map.get(name)
            .and_then(|&id| {
                self.characters.read().await.get(&id).cloned()
            })
    }
    
    pub async fn get_characters_by_account(&self, account_id: AccountId) -> Vec<Character> {
        let account_map = self.by_account.read().await;
        let char_ids = account_map.get(&account_id)
            .cloned()
            .unwrap_or_default();
        
        let characters = self.characters.read().await;
        char_ids.iter()
            .filter_map(|id| characters.get(id).cloned())
            .collect()
    }
}
```

### Inventory Cache

```rust
pub struct InventoryCache {
    // character_id -> Vec<Item>
    inventories: Arc<RwLock<HashMap<CharacterId, Vec<Item>>>>,
    
    // Equipment: character_id -> slot -> Item
    equipment: Arc<RwLock<HashMap<CharacterId, HashMap<EquipmentSlot, Item>>>>,
}

impl InventoryCache {
    pub async fn get_inventory(&self, character_id: CharacterId) -> Vec<Item> {
        self.inventories.read().await
            .get(&character_id)
            .cloned()
            .unwrap_or_default()
    }
    
    pub async fn add_item(&self, character_id: CharacterId, item: Item) {
        let mut inventories = self.inventories.write().await;
        inventories.entry(character_id)
            .or_insert_with(Vec::new)
            .push(item);
    }
}
```

## Server Startup Process

### Data Loading Sequence

```rust
pub struct Server {
    db_path: PathBuf,
    character_cache: CharacterCache,
    inventory_cache: InventoryCache,
    skill_cache: SkillCache,
    quest_cache: QuestCache,
    guild_cache: GuildCache,
}

impl Server {
    pub async fn start(&mut self) -> Result<(), ServerError> {
        // Step 1: Connect to SQLite database
        let conn = Connection::open(&self.db_path)?;
        
        // Step 2: Load all characters
        self.load_characters(&conn).await?;
        
        // Step 3: Load all inventories
        self.load_inventories(&conn).await?;
        
        // Step 4: Load all equipment
        self.load_equipment(&conn).await?;
        
        // Step 5: Load all skills
        self.load_skills(&conn).await?;
        
        // Step 6: Load all quests
        self.load_quests(&conn).await?;
        
        // Step 7: Load all guilds
        self.load_guilds(&conn).await?;
        
        // Step 8: Start periodic save task
        self.start_save_task().await;
        
        // Step 9: Start accepting connections
        self.start_accepting_connections().await?;
        
        Ok(())
    }
    
    async fn load_characters(&mut self, conn: &Connection) -> Result<(), ServerError> {
        let mut stmt = conn.prepare(
            "SELECT * FROM characters"
        )?;
        
        let characters = stmt.query_map([], |row| {
            Ok(Character {
                character_id: row.get(0)?,
                account_id: row.get(1)?,
                character_name: row.get(2)?,
                level: row.get(3)?,
                experience: row.get(4)?,
                str: row.get(5)?,
                dex: row.get(6)?,
                int: row.get(7)?,
                con: row.get(8)?,
                lck: row.get(9)?,
                cha: row.get(10)?,
                gold: row.get(11)?,
                position: Position {
                    x: row.get(12)?,
                    y: row.get(13)?,
                    z: row.get(14)?,
                },
                map_id: row.get(15)?,
                last_login: row.get(16)?,
                created_at: row.get(17)?,
                updated_at: row.get(18)?,
            })
        })?;
        
        let mut count = 0;
        for character_result in characters {
            let character = character_result?;
            let id = character.character_id;
            let account_id = character.account_id;
            let name = character.character_name.clone();
            let map_id = character.map_id.clone();
            
            // Add to primary cache
            self.character_cache.characters.write().await
                .insert(id, character.clone());
            
            // Add to account index
            self.character_cache.by_account.write().await
                .entry(account_id)
                .or_insert_with(Vec::new)
                .push(id);
            
            // Add to name index
            self.character_cache.by_name.write().await
                .insert(name, id);
            
            // Add to map index
            self.character_cache.by_map.write().await
                .entry(map_id)
                .or_insert_with(Vec::new)
                .push(id);
            
            count += 1;
        }
        
        *self.character_cache.player_count.write().await = count;
        
        info!("Loaded {} characters into memory", count);
        Ok(())
    }
}
```

## Runtime Operations

### Query Operations (In-Memory)

**Character Lookup:**
```rust
// O(1) lookup by ID
pub async fn get_character(&self, id: CharacterId) -> Option<Character> {
    self.character_cache.get_character(id).await
}

// O(1) lookup by name
pub async fn get_character_by_name(&self, name: &str) -> Option<Character> {
    self.character_cache.get_character_by_name(name).await
}

// O(n) where n = characters per account (typically small)
pub async fn get_characters_by_account(&self, account_id: AccountId) -> Vec<Character> {
    self.character_cache.get_characters_by_account(account_id).await
}
```

**Inventory Operations:**
```rust
// O(1) lookup
pub async fn get_inventory(&self, character_id: CharacterId) -> Vec<Item> {
    self.inventory_cache.get_inventory(character_id).await
}

// O(1) insert
pub async fn add_item(&self, character_id: CharacterId, item: Item) {
    self.inventory_cache.add_item(character_id, item).await;
}
```

### Write Operations

**Character Updates:**
```rust
pub async fn update_character(&self, character: Character) -> Result<(), ServerError> {
    // Update in-memory cache
    let id = character.character_id;
    self.character_cache.characters.write().await
        .insert(id, character.clone());
    
    // Queue for database save (async, non-blocking)
    self.save_queue.enqueue(SaveTask::UpdateCharacter(character)).await;
    
    Ok(())
}
```

## Critical Data Protection System

### Zero Data Loss Guarantee

**Critical Requirement:**
- **Player Data**: NEVER lost under any circumstances
- **Item Data**: NEVER lost under any circumstances
- **All Characteristics**: Preserved in both database and memory
- **Guaranteed Inserts**: All item creation must succeed

### Snapshot System

**Complete Snapshots:**
- **Frequency**: Every 2 minutes (120 seconds)
- **Scope**: All player data, all items, all game state
- **Thread**: Separate background thread (no gameplay impact)
- **Delta Optimization**: Only updates active players since last snapshot

**Snapshot Process:**
```rust
pub struct SnapshotSystem {
    snapshot_interval: Duration, // 2 minutes
    last_snapshot: Arc<RwLock<Instant>>,
    active_players: Arc<RwLock<HashSet<CharacterId>>>, // Track active players
    snapshot_thread: Option<JoinHandle<()>>,
}

impl SnapshotSystem {
    pub fn start(&mut self) {
        let interval = self.snapshot_interval;
        let last_snapshot = Arc::clone(&self.last_snapshot);
        let active_players = Arc::clone(&self.active_players);
        let character_cache = Arc::clone(&self.character_cache);
        let inventory_cache = Arc::clone(&self.inventory_cache);
        let db_path = self.db_path.clone();
        
        self.snapshot_thread = Some(tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Create complete snapshot
                Self::create_snapshot(
                    &character_cache,
                    &inventory_cache,
                    &active_players,
                    &db_path,
                ).await;
                
                // Update last snapshot time
                *last_snapshot.write().await = Instant::now();
            }
        }));
    }
    
    async fn create_snapshot(
        character_cache: &CharacterCache,
        inventory_cache: &InventoryCache,
        active_players: &Arc<RwLock<HashSet<CharacterId>>>,
        db_path: &Path,
    ) {
        let start_time = Instant::now();
        
        // Get active players (delta optimization)
        let active = active_players.read().await.clone();
        
        // Open database connection
        let conn = Connection::open(db_path).expect("Failed to open database");
        
        // Begin transaction for atomicity
        conn.execute("BEGIN TRANSACTION", []).unwrap();
        
        // Save all active players (delta update)
        let characters = character_cache.characters.read().await;
        for character_id in &active {
            if let Some(character) = characters.get(character_id) {
                Self::save_character_atomic(&conn, character).await;
                Self::save_inventory_atomic(&conn, *character_id, inventory_cache).await;
                Self::save_equipment_atomic(&conn, *character_id, inventory_cache).await;
                Self::save_skills_atomic(&conn, *character_id, character_cache).await;
            }
        }
        
        // Commit transaction
        conn.execute("COMMIT", []).unwrap();
        
        let duration = start_time.elapsed();
        info!("Snapshot completed in {:?} ({} active players)", duration, active.len());
    }
}
```

### Database Backup System

**Backup Strategy:**
- **Frequency**: Every 1 hour (3600 seconds)
- **Method**: Complete database file copy
- **Thread**: Separate background thread (no gameplay impact)
- **Retention**: Keep last 24 hours of backups
- **Location**: Separate backup directory

**Backup Process:**
```rust
pub struct BackupSystem {
    backup_interval: Duration, // 1 hour
    backup_dir: PathBuf,
    db_path: PathBuf,
    backup_thread: Option<JoinHandle<()>>,
}

impl BackupSystem {
    pub fn start(&mut self) {
        let interval = self.backup_interval;
        let backup_dir = self.backup_dir.clone();
        let db_path = self.db_path.clone();
        
        self.backup_thread = Some(tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Create backup
                Self::create_backup(&db_path, &backup_dir).await;
                
                // Clean old backups (keep last 24 hours)
                Self::clean_old_backups(&backup_dir).await;
            }
        }));
    }
    
    async fn create_backup(db_path: &Path, backup_dir: &Path) {
        let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
        let backup_filename = format!("backup_{}.db", timestamp);
        let backup_path = backup_dir.join(&backup_filename);
        
        // Copy database file
        tokio::fs::copy(db_path, &backup_path).await
            .expect("Failed to create backup");
        
        info!("Database backup created: {}", backup_filename);
    }
    
    async fn clean_old_backups(backup_dir: &Path) {
        // Keep backups from last 24 hours
        let cutoff = Utc::now() - chrono::Duration::hours(24);
        
        let mut entries = tokio::fs::read_dir(backup_dir).await.unwrap();
        while let Some(entry) = entries.next_entry().await.unwrap() {
            let metadata = entry.metadata().await.unwrap();
            if let Ok(modified) = metadata.modified() {
                let modified_time: DateTime<Utc> = modified.into();
                if modified_time < cutoff {
                    tokio::fs::remove_file(entry.path()).await.unwrap();
                    info!("Removed old backup: {:?}", entry.path());
                }
            }
        }
    }
}
```

### Delta Update System

**Active Player Tracking:**
- Track players who have been active since last snapshot
- Only save active players (delta optimization)
- Reduces I/O operations significantly
- No gameplay impact

**Implementation:**
```rust
pub struct ActivePlayerTracker {
    active_players: Arc<RwLock<HashSet<CharacterId>>>,
    last_activity: Arc<RwLock<HashMap<CharacterId, Instant>>>,
}

impl ActivePlayerTracker {
    pub fn mark_active(&self, character_id: CharacterId) {
        self.active_players.write().await.insert(character_id);
        self.last_activity.write().await.insert(character_id, Instant::now());
    }
    
    pub fn get_active_players(&self) -> HashSet<CharacterId> {
        self.active_players.read().await.clone()
    }
    
    pub fn clear_after_snapshot(&self) {
        self.active_players.write().await.clear();
    }
}
```

### Item Creation System

**Guaranteed Item Creation:**
- All items created in async queues
- Random ID generation for each item
- Guaranteed successful insert
- All characteristics preserved

**Item Creation Queue:**
```rust
pub struct ItemCreationQueue {
    queue: Arc<RwLock<VecDeque<ItemCreationTask>>>,
    db_connection: Arc<Mutex<Connection>>,
    worker_thread: Option<JoinHandle<()>>,
}

#[derive(Clone)]
pub struct ItemCreationTask {
    pub item_id: ItemId, // Random UUID generated
    pub character_id: CharacterId,
    pub item_type: String,
    pub properties: ItemProperties, // All characteristics preserved
    pub created_at: DateTime<Utc>,
}

impl ItemCreationQueue {
    pub fn new(db_path: &Path) -> Self {
        let conn = Connection::open(db_path).expect("Failed to open database");
        
        Self {
            queue: Arc::new(RwLock::new(VecDeque::new())),
            db_connection: Arc::new(Mutex::new(conn)),
            worker_thread: None,
        }
    }
    
    pub fn start_worker(&mut self) {
        let queue = Arc::clone(&self.queue);
        let db_conn = Arc::clone(&self.db_connection);
        
        self.worker_thread = Some(tokio::spawn(async move {
            loop {
                // Process queue
                while let Some(task) = queue.write().await.pop_front() {
                    Self::create_item_guaranteed(&db_conn, &task).await;
                }
                
                // Small delay to prevent busy waiting
                tokio::time::sleep(Duration::from_millis(10)).await;
            }
        }));
    }
    
    pub async fn enqueue_item_creation(
        &self,
        character_id: CharacterId,
        item_type: String,
        properties: ItemProperties,
    ) -> ItemId {
        // Generate random ID
        let item_id = Uuid::new_v4().to_string();
        
        let task = ItemCreationTask {
            item_id: item_id.clone(),
            character_id,
            item_type,
            properties,
            created_at: Utc::now(),
        };
        
        // Add to queue
        self.queue.write().await.push_back(task);
        
        // Return ID immediately (item will be created async)
        item_id
    }
    
    async fn create_item_guaranteed(
        db_conn: &Arc<Mutex<Connection>>,
        task: &ItemCreationTask,
    ) {
        let conn = db_conn.lock().await;
        
        // Retry logic for guaranteed insert
        let mut retries = 0;
        loop {
            match conn.execute(
                "INSERT INTO items (item_id, character_id, item_type, properties, created_at) VALUES (?, ?, ?, ?, ?)",
                params![
                    task.item_id,
                    task.character_id,
                    task.item_type,
                    serde_json::to_string(&task.properties).unwrap(),
                    task.created_at,
                ]
            ) {
                Ok(_) => {
                    info!("Item created successfully: {}", task.item_id);
                    break;
                }
                Err(e) => {
                    retries += 1;
                    if retries > 10 {
                        error!("Failed to create item after 10 retries: {:?}", e);
                        panic!("Critical: Item creation failed - data loss risk!");
                    }
                    tokio::time::sleep(Duration::from_millis(100 * retries)).await;
                }
            }
        }
    }
}
```

### Data Preservation

**All Characteristics Preserved:**
- Item properties stored as JSON in database
- All attributes preserved in memory
- Synchronized between memory and database
- No data loss during saves

**Item Properties Storage:**
```sql
CREATE TABLE items (
    item_id TEXT PRIMARY KEY,
    character_id INTEGER NOT NULL,
    item_type TEXT NOT NULL,
    properties TEXT NOT NULL, -- JSON with all characteristics
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (character_id) REFERENCES characters(character_id) ON DELETE CASCADE
);

-- Example properties JSON:
-- {
--   "durability": 100,
--   "enchantments": [...],
--   "attributes": {...},
--   "custom_data": {...}
-- }
```

**Save Queue:**
```rust
pub struct SaveQueue {
    queue: Arc<RwLock<VecDeque<SaveTask>>>,
    db_connection: Arc<Mutex<Connection>>,
}

#[derive(Clone)]
pub enum SaveTask {
    UpdateCharacter(Character),
    UpdateInventory(CharacterId, Vec<Item>),
    UpdateEquipment(CharacterId, HashMap<EquipmentSlot, Item>),
    UpdateSkills(CharacterId, Vec<Skill>),
    UpdateQuests(CharacterId, Vec<Quest>),
    UpdateGuild(Guild),
}

impl SaveQueue {
    pub async fn start_save_worker(&self) {
        loop {
            // Process save queue
            while let Some(task) = self.queue.write().await.pop_front() {
                match task {
                    SaveTask::UpdateCharacter(character) => {
                        self.save_character(character).await;
                    }
                    SaveTask::UpdateInventory(id, items) => {
                        self.save_inventory(id, items).await;
                    }
                    // ... other save tasks
                }
            }
            
            // Wait before next batch
            tokio::time::sleep(Duration::from_secs(30)).await;
        }
    }
    
    async fn save_character(&self, character: Character) {
        let conn = self.db_connection.lock().await;
        conn.execute(
            "INSERT OR REPLACE INTO characters VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)",
            params![
                character.character_id,
                character.account_id,
                character.character_name,
                character.level,
                character.experience,
                character.str,
                character.dex,
                character.int,
                character.con,
                character.lck,
                character.cha,
                character.gold,
                character.position.x,
                character.position.y,
                character.position.z,
                character.map_id,
                character.last_login,
                character.updated_at,
            ]
        ).unwrap();
    }
}
```

## Player Capacity Management

### 10,000 Player Limit

**Why 10,000 Players?**

1. **Memory Constraints**: Each player consumes memory (character data, inventory, skills, etc.)
2. **Query Performance**: Larger datasets slow down in-memory operations
3. **Economy Protection**: Prevents inflation from too many players
4. **Server Stability**: Limits load on single server instance
5. **Regional Distribution**: Encourages multiple regional servers

**Capacity Check:**
```rust
impl Server {
    pub async fn can_create_character(&self) -> bool {
        self.character_cache.can_create_character().await
    }
    
    pub async fn create_character(&self, character: Character) -> Result<CharacterId, ServerError> {
        // Check capacity
        if !self.can_create_character().await {
            return Err(ServerError::ServerFull);
        }
        
        // Create character
        let id = character.character_id;
        self.character_cache.add_character(character).await?;
        
        // Increment count
        *self.character_cache.player_count.write().await += 1;
        
        Ok(id)
    }
}
```

**Server Full Handling:**
```rust
pub enum ServerError {
    ServerFull,
    CharacterNameTaken,
    InvalidData,
    // ... other errors
}

// When server is full:
// - Character creation is blocked
// - Players see "Server Full" message
// - Suggest alternative servers
// - Waitlist option (optional)
```

## Advantages of This Architecture

### Performance Benefits

1. **Ultra-Low Latency**: In-memory queries are nanoseconds vs milliseconds for database
2. **No Database Bottlenecks**: SQLite only used for backup, not queries
3. **Optimized Data Structures**: Custom indexes for common queries
4. **Parallel Access**: Multiple readers, single writer pattern
5. **Cache Locality**: Data structures optimized for CPU cache

### Scalability Benefits

1. **Horizontal Scaling**: Add more servers instead of scaling database
2. **Regional Deployment**: Deploy servers close to players
3. **Load Distribution**: Players distributed across servers
4. **Independent Scaling**: Each server scales independently

### Economy Protection

1. **Player Limits**: Prevents economy inflation
2. **Resource Management**: Limited players = limited resources
3. **Market Stability**: Smaller player base = more stable economy
4. **Competition Balance**: Prevents overcrowding

### Operational Benefits

1. **Simple Deployment**: Single SQLite file per server
2. **Easy Backup**: Copy SQLite file for backup
3. **Fast Recovery**: Load from SQLite on restart
4. **Low Maintenance**: No complex database cluster

## Disadvantages and Mitigations

### Data Loss Risk

**Risk:** In-memory data lost on crash before save

**Mitigation:**
- Frequent periodic saves (every 2-5 minutes)
- Immediate saves for critical operations
- Transaction logs for recovery
- Graceful shutdown saves all data

### Memory Usage

**Risk:** High memory usage with 10k players

**Mitigation:**
- Player limit prevents excessive memory
- Efficient data structures
- Memory monitoring
- Server restart if memory issues

### Single Point of Failure

**Risk:** Server crash affects all players

**Mitigation:**
- Regular backups
- Fast recovery from SQLite
- Multiple regional servers
- Load balancing

## Regional Server Deployment

### Geographic Distribution

**Server Regions:**
- **North America**: US East, US West, Canada
- **Europe**: UK, Germany, France
- **Asia**: Japan, Korea, Singapore
- **South America**: Brazil, Argentina
- **Oceania**: Australia, New Zealand

**Benefits:**
- **Low Latency**: Players connect to nearest server
- **Better Experience**: Reduced ping improves gameplay
- **Load Distribution**: Players spread across regions
- **Compliance**: Regional data storage requirements

### Server Selection

```rust
pub struct ServerRegistry {
    servers: Vec<ServerInfo>,
}

pub struct ServerInfo {
    pub server_id: ServerId,
    pub server_name: String,
    pub region: Region,
    pub ip_address: String,
    pub port: u16,
    pub player_count: usize,
    pub max_players: usize,
    pub is_online: bool,
    pub latency_ms: Option<u32>, // Measured from client
}

pub enum Region {
    NorthAmerica,
    Europe,
    Asia,
    SouthAmerica,
    Oceania,
}

impl ServerRegistry {
    pub fn get_recommended_servers(&self, client_region: Region) -> Vec<ServerInfo> {
        self.servers.iter()
            .filter(|s| s.region == client_region)
            .filter(|s| s.is_online)
            .filter(|s| s.player_count < s.max_players)
            .sorted_by_key(|s| s.latency_ms.unwrap_or(u32::MAX))
            .cloned()
            .collect()
    }
}
```

## Recovery and Corruption Handling

### Automatic Recovery System

**Corruption Detection:**
- Check database integrity on startup
- Verify database file integrity
- Detect corrupted pages
- Automatic recovery from backup

**Recovery Process:**
```rust
pub struct RecoverySystem {
    db_path: PathBuf,
    backup_dir: PathBuf,
}

impl RecoverySystem {
    pub fn check_and_recover(&self) -> Result<(), RecoveryError> {
        // Check database integrity
        match self.check_database_integrity() {
            Ok(_) => {
                info!("Database integrity check passed");
                Ok(())
            }
            Err(e) => {
                warn!("Database corruption detected: {:?}", e);
                self.recover_from_backup()?;
                Ok(())
            }
        }
    }
    
    fn check_database_integrity(&self) -> Result<(), DatabaseError> {
        let conn = Connection::open(&self.db_path)?;
        
        // Run integrity check
        let mut stmt = conn.prepare("PRAGMA integrity_check")?;
        let result: String = stmt.query_row([], |row| row.get(0))?;
        
        if result == "ok" {
            Ok(())
        } else {
            Err(DatabaseError::CorruptionDetected(result))
        }
    }
    
    fn recover_from_backup(&self) -> Result<(), RecoveryError> {
        // Find most recent backup
        let backup = self.find_most_recent_backup()?;
        
        info!("Recovering from backup: {:?}", backup);
        
        // Copy backup to database location
        std::fs::copy(&backup, &self.db_path)?;
        
        // Verify recovered database
        self.check_database_integrity()?;
        
        info!("Database recovered successfully from backup");
        Ok(())
    }
    
    fn find_most_recent_backup(&self) -> Result<PathBuf, RecoveryError> {
        let mut backups: Vec<_> = std::fs::read_dir(&self.backup_dir)?
            .filter_map(|entry| {
                let entry = entry.ok()?;
                let path = entry.path();
                if path.extension()? == "db" {
                    Some((entry.metadata().ok()?.modified().ok()?, path))
                } else {
                    None
                }
            })
            .collect();
        
        backups.sort_by_key(|(modified, _)| *modified);
        backups.last()
            .map(|(_, path)| path.clone())
            .ok_or(RecoveryError::NoBackupFound)
    }
}
```

### Database Maintenance

**VACUUM Operation:**
- Periodically run VACUUM to clean old data
- Reclaim disk space
- Optimize database structure
- Run during low-traffic periods

**VACUUM Schedule:**
```rust
pub struct DatabaseMaintenance {
    vacuum_interval: Duration, // Daily
    db_path: PathBuf,
}

impl DatabaseMaintenance {
    pub fn start(&mut self) {
        let db_path = self.db_path.clone();
        let interval = self.vacuum_interval;
        
        tokio::spawn(async move {
            loop {
                tokio::time::sleep(interval).await;
                
                // Run VACUUM during low traffic (e.g., 3 AM)
                let hour = Utc::now().hour();
                if hour == 3 {
                    Self::run_vacuum(&db_path).await;
                }
            }
        });
    }
    
    async fn run_vacuum(db_path: &Path) {
        let conn = Connection::open(db_path).expect("Failed to open database");
        
        info!("Running VACUUM to clean database...");
        conn.execute("VACUUM", []).expect("VACUUM failed");
        info!("VACUUM completed successfully");
    }
}
```

### Normal Shutdown Process

**Graceful Shutdown:**
```
1. Stop accepting new connections
2. Wait for active operations to complete
3. Create final snapshot (all players)
4. Save all in-memory data to SQLite
5. Create final backup
6. Close database connection
7. Shutdown server
```

**Crash Recovery:**
```
1. Server detects crash/restart
2. Check database integrity
3. If corrupted: Restore from most recent backup
4. Load all data from SQLite
5. Rebuild in-memory structures
6. Resume normal operation
7. Players reconnect
```

## Activity Logging System

### Daily Activity Logs

**Critical Activities Logged:**
- Player kills (PvP and PvE)
- Items dropped/looted
- Item creation/destruction
- Character level ups
- Gold transactions
- Guild activities
- Admin actions

**Log Structure:**
```rust
pub struct ActivityLog {
    log_id: u64,
    timestamp: DateTime<Utc>,
    event_type: EventType,
    character_id: Option<CharacterId>,
    target_id: Option<EntityId>,
    details: serde_json::Value, // JSON with event-specific data
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum EventType {
    PlayerKill {
        killer_id: CharacterId,
        victim_id: CharacterId,
        location: Position,
    },
    ItemDropped {
        item_id: ItemId,
        character_id: CharacterId,
        location: Position,
    },
    ItemLooted {
        item_id: ItemId,
        character_id: CharacterId,
        source: String, // "creature", "player", "chest", etc.
    },
    ItemCreated {
        item_id: ItemId,
        character_id: CharacterId,
        item_type: String,
    },
    LevelUp {
        character_id: CharacterId,
        old_level: u8,
        new_level: u8,
    },
    GoldTransaction {
        from_id: Option<CharacterId>,
        to_id: Option<CharacterId>,
        amount: i64,
        reason: String,
    },
    // ... other event types
}
```

**Logging Implementation:**
```rust
pub struct LoggingSystem {
    log_file: Arc<Mutex<BufWriter<File>>>,
    daily_rotation: JoinHandle<()>,
}

impl LoggingSystem {
    pub fn new(log_dir: &Path) -> Self {
        let log_file = Self::open_daily_log(log_dir);
        let log_file_clone = Arc::clone(&log_file);
        let log_dir = log_dir.to_path_buf();
        
        // Rotate logs daily
        let daily_rotation = tokio::spawn(async move {
            loop {
                // Wait until midnight
                let now = Utc::now();
                let tomorrow = (now.date_naive() + chrono::Duration::days(1))
                    .and_hms(0, 0, 0);
                let duration = (tomorrow.and_utc() - now).to_std().unwrap();
                tokio::time::sleep(duration).await;
                
                // Rotate log file
                *log_file_clone.lock().await = Self::open_daily_log(&log_dir);
            }
        });
        
        Self {
            log_file,
            daily_rotation,
        }
    }
    
    fn open_daily_log(log_dir: &Path) -> Arc<Mutex<BufWriter<File>>> {
        let date = Utc::now().format("%Y%m%d");
        let log_path = log_dir.join(format!("activity_{}.log", date));
        let file = File::create(&log_path).expect("Failed to create log file");
        Arc::new(Mutex::new(BufWriter::new(file)))
    }
    
    pub async fn log_activity(&self, activity: ActivityLog) {
        let mut log_file = self.log_file.lock().await;
        let json = serde_json::to_string(&activity).unwrap();
        writeln!(log_file, "{}", json).unwrap();
        log_file.flush().unwrap();
    }
}
```

**Log File Structure:**
```
logs/
├── activity_20240101.log
├── activity_20240102.log
├── activity_20240103.log
└── ...
```

**Example Log Entry:**
```json
{
  "log_id": 12345,
  "timestamp": "2024-01-01T12:34:56Z",
  "event_type": {
    "PlayerKill": {
      "killer_id": "char_abc123",
      "victim_id": "char_xyz789",
      "location": {"x": 1000.0, "y": 2000.0, "z": 500.0}
    }
  },
  "character_id": "char_abc123",
  "target_id": "char_xyz789",
  "details": {
    "weapon_used": "Sword_Steel",
    "damage_dealt": 150,
    "zone": "Red_Zone_01"
  }
}
```

## Performance Metrics

### Expected Performance

**Query Times:**
- Character lookup: < 1 microsecond
- Inventory access: < 1 microsecond
- Account character list: < 10 microseconds
- Map character list: < 100 microseconds

**Save Times:**
- Character save: < 10 milliseconds
- Snapshot (active players): < 500 milliseconds
- Full snapshot (all players): < 5 seconds
- Database backup: < 10 seconds

**Memory Usage:**
- Per character: ~10-50 KB (depending on inventory)
- 10,000 characters: ~100-500 MB
- Total server memory: ~1-2 GB (with overhead)

**Background Thread Impact:**
- Snapshot thread: < 1% CPU during snapshot
- Backup thread: < 5% CPU during backup
- Item creation queue: < 0.1% CPU
- No gameplay impact (separate threads)

## Testing Considerations

### Unit Tests

**Test Cases:**
- Character cache operations
- Inventory cache operations
- Capacity limit enforcement
- Save queue processing
- Data loading from SQLite

### Integration Tests

**Test Cases:**
- Full server startup and data loading
- Character creation with capacity limit
- Periodic save system
- Crash recovery
- Backup and restore

### Performance Tests

**Test Cases:**
- Query performance under load
- Memory usage with max players
- Save performance
- Concurrent access patterns

## Summary

The persistence architecture:

- **Uses In-Memory Primary Storage**: All data loaded into memory at startup
- **SQLite as Backup**: Periodic saves to SQLite for persistence
- **10,000 Player Limit**: Prevents overload and protects economy
- **Server Isolation**: Each server has independent database
- **Regional Deployment**: Servers deployed close to players
- **Ultra-Low Latency**: In-memory queries are nanoseconds
- **Simple Operations**: Single SQLite file per server
- **Fast Recovery**: Quick restart from SQLite backup

**Key Benefits:**
- **Performance**: Nanosecond query times
- **Scalability**: Horizontal scaling with multiple servers
- **Economy Protection**: Player limits prevent inflation
- **Regional Optimization**: Low latency for players
- **Simplicity**: Easy deployment and maintenance

**Trade-offs:**
- **Memory Usage**: High memory consumption
- **Data Loss Risk**: Mitigated by frequent saves
- **Single Server Failure**: Mitigated by multiple servers

This architecture prioritizes performance and player experience while maintaining data safety through periodic backups. The 10,000 player limit ensures optimal performance and protects server economies from inflation.

