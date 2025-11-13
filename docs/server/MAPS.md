# Map System Documentation

## Overview

Maps are the fundamental units of the game world, each representing a distinct area with its own terrain, entities, and gameplay mechanics. Maps are loaded from the central API and managed by game servers, handling all entities within their boundaries including players, creatures, pets, mounts, NPCs, and interactive objects. Each map manages its own zones, Sacred Lands, teleportation points, resource gathering spots, respawns, and dynamic portals.

## Key Features

- **Central API Loading**: All map configurations loaded from central API at server startup
- **Entity Management**: Comprehensive management of all entities (players, creatures, pets, mounts, NPCs)
- **Waypoint System**: Waypoints for transitioning between maps
- **Teleportation Points**: Multiple teleportation methods within and between maps
- **Zone Management**: Each map typically has one primary zone (Blue, Yellow, Red, Black, or Event)
- **Sacred Lands**: Protected respawn areas with Goddess Statues
- **Resource Gathering**: Spots for collecting resources (trees, rocks, plants, herbs)
- **Respawn System**: Creature and resource respawn management
- **Dynamic Portals**: Event-based and admin-created temporary portals
- **Spatial Optimization**: Grid-based cell system for efficient entity queries
- **Game Loop**: Dedicated game loop per map with async loading

## Map Loading System

### Central API Integration

Maps are loaded from the central API during server startup. The configuration includes:

- **Map Settings**: Biome, client scene name, namespace, tier, difficulty, dungeon status
- **Resource Spots**: Available trees, rocks, plants, and herbs per biome
- **NPCs**: NPCs assigned to the map with positions, behaviors, and properties
- **Zones**: Zone configuration (typically one zone per map)
- **Respawns**: Creature and resource respawn points
- **Waypoints**: Transition points to other maps
- **Teleports**: Teleportation points within the map
- **Sacred Lands**: Protected areas with Goddess Statues

**Loading Process:**
1. Server requests map configuration from central API
2. Map settings loaded (biome, difficulty, tier, resources)
3. NPCs loaded and positioned
4. Waypoints loaded from metadata files
5. Teleports loaded from metadata files
6. Flags loaded (zones, Sacred Lands, respawns, resource spots)
7. Map initialized and game loop started

**API Endpoints:**
- `GET /gamedata/maps/Beta/public.yml` - Public map configurations
- `GET /gamedata/maps/Dungeons/public.yml` - Dungeon map configurations
- `GET /gamedata/npcs/public.yml` - NPC configurations

## Entity Management

Maps manage all entities within their boundaries:

### Player Entities

- **Character Entities**: Active player characters
- **Position Tracking**: Real-time position updates
- **State Management**: HP, Mana, Stamina, status effects
- **Inventory**: Equipment and inventory management
- **Controller System**: Player controllers for input handling

### Creature Entities

- **Monsters**: Hostile creatures
- **Animals**: Neutral creatures
- **Bosses**: Special high-level creatures
- **Spawn Management**: Respawn timers and limits
- **AI Behavior**: Pathfinding, combat, patrols

### Pet Entities

- **Pet System**: Player-owned pets
- **Combat Pets**: Pets that assist in combat
- **Utility Pets**: Pets with special abilities
- **Pet AI**: Pet behavior and commands

### Mount Entities

- **Mount System**: Player-owned mounts
- **Mount Types**: Different mount types with varying speeds
- **Mount Storage**: Mounts stored when not in use
- **Mount Abilities**: Special mount abilities

### NPC Entities

- **Non-Player Characters**: Quest NPCs, vendors, guards
- **NPC Behaviors**: Dialogue, trading, quest giving
- **NPC Positions**: Fixed positions on map
- **NPC Interactions**: Player interaction handling

### Dynamic Entities

- **Temporary Objects**: Items dropped on ground
- **Projectiles**: Spells, arrows, thrown weapons
- **Effects**: Visual and audio effects
- **Interactive Objects**: Chests, doors, switches

## Waypoint System

Waypoints are transition points that allow players to move between maps.

### Waypoint Structure

- **Name**: Waypoint identifier (default: "Start")
- **Position**: X, Y coordinates on map
- **Scene Name**: Associated map name
- **Destination**: Target map and waypoint

### Waypoint Loading

Waypoints are loaded from metadata files:
- `Metadata/Scenes/{SceneName}.Waypoints.yml`

**Format:**
```yaml
- !Waypoint
  Name: "Start"
  X: 1000
  Y: 2000
```

### Waypoint Usage

- Players interact with waypoint to transition
- Transition validates destination map
- Player position updated to destination waypoint
- Map change notification sent to client

## Teleportation Points

Maps contain various teleportation points:

### City Portals

- **Location**: Major cities and settlements
- **Function**: World-wide teleportation
- **Cost**: Gold (distance-based)
- **Availability**: Always available

### Sacred Land Statues

- **Location**: Sacred Lands on each map
- **Function**: Same-map teleportation
- **Cost**: Optional gold cost
- **Availability**: Always available

### Ancient Monoliths

- **Location**: Various locations on map
- **Function**: Teleportation to any location on current map
- **Cost**: Red Runic Crystals
- **Availability**: Always available

### Dynamic Portals

- **Location**: Event-triggered or admin-created
- **Function**: Teleportation to instanced levels or specific destinations
- **Cost**: Usually free
- **Availability**: Temporary

**Teleport Loading:**
Teleports are loaded from metadata files:
- `Metadata/Scenes/{SceneName}.Teleports.yml`

**Format:**
```yaml
- !Teleport
  X: 5000
  Y: 3000
  Width: 200
  Height: 200
  Rotation: 0
  SceneName: "map_002"
  WaypointName: "Entrance"
```

## Zone Management

Each map typically has **one primary zone** that defines PvP rules, death mechanics, and resistance penalties.

### Zone Types

- **Blue Zone**: Safe zone, no PvP, no death
- **Yellow Zone**: Neutral zone, consensual PvP, no item loss
- **Red Zone**: War zone, PvP enabled, partial item/gold loss
- **Black Zone**: Full PvP, complete item/gold loss
- **Event Zone**: Custom rules per event

### Zone Configuration

Zones are configured per map:
- **Zone Type**: Determines PvP and death rules
- **Boundaries**: Zone area boundaries
- **Resistance Penalties**: 0-50% resistance reduction
- **Guard System**: Guards for safe zones

**Zone Loading:**
Zones are loaded from metadata files:
- `Metadata/Scenes/{SceneName}.Flags.yml`

**Format:**
```yaml
- !SafeZone
  X: 1000
  Y: 2000
  Width: 500
  Height: 500
  Rotation: 0
```

## Sacred Lands

Every map must have **at least one Sacred Land** area.

### Sacred Land Features

- **Goddess Statue**: Central respawn point
- **Protected Area**: Complete immunity zone
- **Respawn Point**: Players respawn here after death
- **Teleportation**: Same-map teleportation hub
- **Creature Behavior**: Creatures lose targets and retreat

**Sacred Land Loading:**
Sacred Lands are loaded from metadata files as SafeZones with special properties.

## Resource Gathering Spots

Maps contain various resource gathering spots:

### Spot Types

- **Trees**: Wood gathering spots
- **Rocks**: Stone and ore gathering spots
- **Plants**: Herbs and plant gathering spots
- **Herbs**: Special herb gathering spots

### Spot Configuration

Spots are configured per biome and tier:
- **Biome**: Determines available resource types
- **Tier**: Determines resource quality
- **Chance System**: Weighted chance for each resource type
- **Respawn Rate**: How quickly spots respawn

**Spot Loading:**
Spots are loaded from metadata files:
- `Metadata/Scenes/{SceneName}.Flags.yml`

**Format:**
```yaml
- !Tree
  X: 3000
  Y: 4000
  Rotation: 0

- !Rock
  X: 3500
  Y: 4100
  Rotation: 0

- !Plant
  X: 3200
  Y: 3900
  Rotation: 0
```

### Spot Respawn System

- **Respawn Managers**: Manage spot respawn timers
- **Biome-Based**: Spots selected based on map biome
- **Tier-Based**: Resource quality based on map tier
- **Random Selection**: Random spot type from available pool

## Respawn System

Maps manage creature and resource respawns:

### Respawn Types

- **Creature Respawns**: Spawn points for monsters and animals
- **Resource Respawns**: Spawn points for gathering resources
- **Biome Respawns**: Biome-based creature spawning
- **Dungeon Respawns**: Special dungeon respawn mechanics

### Respawn Configuration

- **Position**: X, Y coordinates
- **Creature Type**: Creature name or resource type
- **Spawn Rate**: Time between spawns
- **Max Spawns**: Maximum concurrent spawns
- **Respawn Radius**: Area where spawns can occur

**Respawn Loading:**
Respawns are loaded from metadata files:
- `Metadata/Scenes/{SceneName}.Flags.yml`

**Format:**
```yaml
- !Respawn
  X: 5000
  Y: 6000
  CreatureName: "Goblin"

- !BiomeRespawn
  X: 7000
  Y: 8000
  Biome: "Forest"
  Tier: 3
```

### Respawn Managers

- **RespawnManager**: Standard creature respawn
- **BiomeRespawnManager**: Biome-based respawn
- **DungeonRespawnManager**: Dungeon-specific respawn
- **SpotRespawnManager**: Resource spot respawn

## Dynamic Portals

Maps can contain dynamic portals created by events or administrators.

### Portal Types

- **Event Portals**: Automatically spawned during events
- **Admin Portals**: Created by administrators via commands
- **Shadowgates**: Special event portals

### Portal Management

- **Creation**: Via admin command `/portal <position> <map> <destination>`
- **Removal**: Via admin interface or command
- **Temporary**: Portals can have duration limits
- **Access Control**: Public or restricted access

## Heightmap System

Maps include a heightmap system that maps terrain heights across the entire map. This system is essential for:
- **Flyhack Detection**: Validating player positions against terrain height
- **Pathfinding**: Navigation mesh generation
- **Collision Detection**: Ground collision validation
- **Visualization**: Map visualization and debugging

### Heightmap Generation

Heightmaps are generated from Unreal Engine using a specialized exporter that:
- **Raycast Sampling**: Performs raycasts from above to below terrain at regular intervals
- **Multi-Surface Detection**: Captures all surfaces at each X,Y position (multiple Z heights)
- **Parallel Processing**: Uses parallel processing for efficient generation
- **Configurable Resolution**: Resolution configurable via settings (default based on map size)

### Heightmap Data Structure

The heightmap is stored in a compact JSON format:

```json
{
  "LevelName": "Map_001",
  "LevelBounds": {
    "MinX": -5000,
    "MinY": -5000,
    "MinZ": -1000,
    "MaxX": 5000,
    "MaxY": 5000,
    "MaxZ": 1000
  },
  "Heightmap": {
    "Data": [
      {
        "X": 0,
        "Y": 0,
        "Z": [100, 150, 200]
      },
      {
        "X": 100,
        "Y": 0,
        "Z": [105, 155, 205]
      }
    ],
    "SampleCount": 10000,
    "ValidHits": 8500
  }
}
```

**Format Details:**
- **X, Y**: Grid coordinates (rounded to integers)
- **Z**: Array of all Z heights at this X,Y position (sorted, rounded to integers)
- **Multiple Heights**: Each position can have multiple Z values (bridges, platforms, multi-level structures)
- **Compact Format**: Minimal JSON structure for efficient storage and loading

### Heightmap Loading

Heightmaps are loaded from JSON files at map initialization:

1. **Load Heightmap File**: Load JSON file from map metadata directory
2. **Build Height Grid**: Build in-memory height grid for fast lookups
3. **Validate Bounds**: Validate heightmap bounds match map bounds
4. **Cache for Queries**: Cache heightmap data for efficient position validation

### Heightmap Queries

The heightmap system provides efficient height queries:

- **Get Height at Position**: Get ground height at specific X,Y coordinates
- **Get All Heights**: Get all surface heights at X,Y (for multi-level structures)
- **Validate Position**: Check if a Z position is valid for given X,Y
- **Find Nearest Ground**: Find nearest ground level below a position

### Heightmap Integration

**With Flyhack Detection:**
- Validates player Z position against terrain height
- Detects impossible heights (flying)
- Validates vertical movement speed

**With Pathfinding:**
- Provides ground level for navigation mesh
- Validates path waypoints against terrain
- Ensures creatures walk on ground

**With Collision Detection:**
- Validates ground collision
- Detects falling through terrain
- Validates movement constraints

### Heightmap Export Process

The heightmap is exported from Unreal Engine using the `NavMeshExporter`:

1. **Calculate World Bounds**: Determine map boundaries
2. **Generate Sample Grid**: Create grid of X,Y sample points
3. **Parallel Raycasting**: Perform parallel raycasts from above terrain
4. **Collect Hit Results**: Collect all surface hits at each position
5. **Build JSON Structure**: Build compact JSON structure
6. **Save to File**: Save heightmap JSON file
7. **Generate Visualization**: Optional Python script generates heightmap image

**Export Settings:**
- **Resolution**: Configurable sampling resolution (default: based on map size)
- **Buffer Zone**: Additional buffer around map bounds for better coverage
- **Raycast Range**: Configurable Z range for raycasting
- **Collision Channels**: Only traces against WorldStatic objects

### Heightmap Storage

- **File Format**: JSON (compact format, no indentation)
- **File Location**: `Metadata/Scenes/{MapName}_Heightmap.json`
- **File Size**: Optimized for minimal size (integers only, compact arrays)
- **Loading**: Loaded into memory at map initialization for fast queries

## Spatial Optimization System

Maps use a grid-based cell system for efficient spatial queries:

### Grid System

- **Cell Size**: 4.0 units per cell
- **Grid Structure**: 2D grid of cells
- **Cell Types**: Static and dynamic entities per cell
- **Out-of-Bounds**: Special handling for out-of-bounds cells

### Cell Operations

- **Entity Addition**: Entities added to appropriate cells
- **Entity Removal**: Entities removed from cells
- **Spatial Queries**: Efficient queries using cell boundaries
- **Collision Detection**: Cell-based collision detection

### Query Methods

- **CheckCircle**: Check entities within circle
- **CheckShape**: Check entities within shape
- **IntersectsCircle**: Check if shape intersects circle
- **Raycast**: Raycast through cells
- **ShapeCast**: Cast shape through cells

## Game Loop System

Each map runs its own dedicated game loop:

### Loop Structure

- **Delta Time**: Fixed 1/32 seconds per tick
- **Update Order**: Entities, controllers, network
- **Task Queue**: Async task execution
- **Timer Queue**: Scheduled timer execution

### Update Cycle

1. **Process Network Events**: Handle incoming packets
2. **Update Timers**: Process scheduled timers
3. **Update Entities**: Update all updatable entities
4. **Update Controllers**: Update player controllers
5. **Update Network**: Send outgoing packets
6. **Sleep Management**: Sleep when inactive

### Async Loading

- **Blueprints**: Load player-built structures
- **Respawns**: Load creature respawns
- **Spots**: Load resource gathering spots
- **Non-Blocking**: Loading doesn't block game loop

## Map Lifecycle

### Map Creation

1. **Request from SceneManager**: Map requested by UUID/name
2. **Load Configuration**: Load from central API
3. **Load Metadata**: Load waypoints, teleports, flags
4. **Initialize Grid**: Create spatial grid
5. **Start Game Loop**: Begin dedicated thread
6. **Register**: Add to SceneManager

### Map Updates

- **Entity Updates**: All entities updated per tick
- **Respawn Updates**: Respawn timers updated
- **Network Updates**: Network packets processed
- **State Synchronization**: State synchronized with clients

### Map Cleanup

- **Save Players**: Save all player data
- **Remove Entities**: Clean up all entities
- **Unregister**: Remove from SceneManager
- **Thread Termination**: Stop game loop thread

## Rust Implementation

### Map Structure

```rust
pub struct Map {
    pub map_id: MapId,
    pub name: String,
    pub uuid: String,
    
    // Configuration
    pub biome: Biome,
    pub tier: u8,
    pub difficulty: ZoneType,
    pub is_dungeon: bool,
    
    // Spatial System
    pub grid: Grid,
    pub origin: Vector2i,
    pub size: Vector2i,
    
    // Heightmap System
    pub heightmap: Heightmap,
    pub heightmap_bounds: BoundingBox,
    
    // Entities
    pub players: HashMap<CharacterId, PlayerEntity>,
    pub creatures: HashMap<CreatureId, CreatureEntity>,
    pub npcs: HashMap<NpcId, NpcEntity>,
    pub dynamic_entities: HashMap<EntityId, Entity>,
    
    // Systems
    pub respawn_managers: Vec<RespawnManager>,
    pub spot_managers: Vec<SpotRespawnManager>,
    pub teleports: Vec<TeleportEntity>,
    pub waypoints: HashMap<String, Vector2>,
    
    // Zones and Sacred Lands
    pub zone: Option<Zone>,
    pub sacred_lands: Vec<SacredLand>,
    
    // Dynamic Portals
    pub dynamic_portals: HashMap<PortalId, DynamicPortal>,
    
    // Game Loop
    pub timer_queue: TimerQueue,
    pub task_queue: Arc<Mutex<VecDeque<Task>>>,
    
    // Network
    pub net_manager: NetManager,
    
    // State
    pub current_time: TimeSpan,
    pub is_finalized: bool,
}
```

### Map Loading

```rust
impl Map {
    pub async fn load(
        map_id: MapId,
        name: String,
        api_client: &ApiClient,
    ) -> Result<Self, MapError> {
        // Load configuration from central API
        let config = api_client.get_map_config(&map_id).await?;
        
        // Load metadata files
        let waypoints = Self::load_waypoints(&name)?;
        let teleports = Self::load_teleports(&name)?;
        let flags = Self::load_flags(&name, &config)?;
        let heightmap = Self::load_heightmap(&name)?;
        
        // Initialize grid
        let (origin, size, grid) = Self::calculate_grid(&flags)?;
        
        // Create map
        let mut map = Map {
            map_id,
            name,
            uuid: map_id.to_string(),
            biome: config.biome,
            tier: config.tier,
            difficulty: config.difficulty,
            is_dungeon: config.is_dungeon,
            grid,
            origin,
            size,
            players: HashMap::new(),
            creatures: HashMap::new(),
            npcs: HashMap::new(),
            dynamic_entities: HashMap::new(),
            respawn_managers: flags.respawns,
            spot_managers: flags.spots,
            teleports: flags.teleports,
            waypoints,
            zone: flags.zone,
            sacred_lands: flags.sacred_lands,
            dynamic_portals: HashMap::new(),
            heightmap: heightmap.clone(),
            heightmap_bounds: heightmap.bounds(),
            timer_queue: TimerQueue::new(),
            task_queue: Arc::new(Mutex::new(VecDeque::new())),
            net_manager: NetManager::new(),
            current_time: TimeSpan::ZERO,
            is_finalized: false,
        };
        
        // Initialize respawn managers
        for respawn in &map.respawn_managers {
            respawn.initialize(&mut map)?;
        }
        
        // Initialize spot managers
        for spot in &map.spot_managers {
            spot.initialize(&mut map)?;
        }
        
        Ok(map)
    }
    
    fn load_waypoints(name: &str) -> Result<HashMap<String, Vector2>, MapError> {
        let path = format!("Metadata/Scenes/{}.Waypoints.yml", name);
        let content = std::fs::read_to_string(&path)?;
        let waypoints: Vec<WaypointObject> = serde_yaml::from_str(&content)?;
        
        let mut map = HashMap::new();
        for waypoint in waypoints {
            let key = format!("{}:{}", name, waypoint.name.unwrap_or("Start".to_string()));
            map.insert(key, Vector2::new(waypoint.x, waypoint.y));
        }
        
        Ok(map)
    }
    
    fn load_teleports(name: &str) -> Result<Vec<TeleportEntity>, MapError> {
        let path = format!("Metadata/Scenes/{}.Teleports.yml", name);
        if !std::path::Path::new(&path).exists() {
            return Ok(Vec::new());
        }
        
        let content = std::fs::read_to_string(&path)?;
        let objects: Vec<TeleportObject> = serde_yaml::from_str(&content)?;
        
        let mut teleports = Vec::new();
        for obj in objects {
            teleports.push(TeleportEntity {
                collision_shape: Shape::new_rectangle(
                    Vector2::new(obj.x, obj.y),
                    Vector2::new(obj.width, obj.height),
                    obj.rotation.to_direction(),
                ),
                scene_name: obj.scene_name,
                waypoint_name: obj.waypoint_name,
            });
        }
        
        Ok(teleports)
    }
    
    fn load_flags(
        name: &str,
        config: &MapConfig,
    ) -> Result<MapFlags, MapError> {
        let path = format!("Metadata/Scenes/{}.Flags.yml", name);
        if !std::path::Path::new(&path).exists() {
            return Ok(MapFlags::default());
        }
        
        let content = std::fs::read_to_string(&path)?;
        let objects: Vec<SceneObject> = serde_yaml::from_str(&content)?;
        
        let mut flags = MapFlags::default();
        
        for obj in objects {
            match obj {
                SceneObject::Respawn(r) => {
                    flags.respawns.push(RespawnManager {
                        position: Vector2::new(r.x, r.y),
                        creature_name: r.creature_name,
                        spawn_rate: 5.0, // Default
                        max_spawns: 10, // Default
                    });
                }
                SceneObject::BiomeRespawn(br) => {
                    flags.spots.push(SpotRespawnManager {
                        position: Vector2::new(br.x, br.y),
                        biome: config.biome.clone(),
                        tier: config.tier,
                    });
                }
                SceneObject::SafeZone(sz) => {
                    // Check if this is a Sacred Land
                    if sz.is_sacred_land {
                        flags.sacred_lands.push(SacredLand {
                            statue_position: Vector2::new(sz.x, sz.y),
                            protection_radius: (sz.width.max(sz.height) / 2.0) as f32,
                        });
                    } else {
                        // Regular safe zone
                        flags.zone = Some(Zone {
                            zone_type: ZoneType::Blue,
                            boundaries: Shape::new_rectangle(
                                Vector2::new(sz.x, sz.y),
                                Vector2::new(sz.width, sz.height),
                                sz.rotation.to_direction(),
                            ),
                            resistance_penalty: 0.0,
                        });
                    }
                }
                SceneObject::DuelZone(dz) => {
                    flags.zone = Some(Zone {
                        zone_type: ZoneType::Yellow,
                        boundaries: Shape::new_rectangle(
                            Vector2::new(dz.x, dz.y),
                            Vector2::new(dz.width, dz.height),
                            dz.rotation.to_direction(),
                        ),
                        resistance_penalty: 0.0,
                    });
                }
                SceneObject::Tree(t) => {
                    flags.spots.push(SpotRespawnManager {
                        position: Vector2::new(t.x, t.y),
                        biome: config.biome.clone(),
                        tier: config.tier,
                        spot_type: SpotType::Tree,
                    });
                }
                SceneObject::Rock(r) => {
                    flags.spots.push(SpotRespawnManager {
                        position: Vector2::new(r.x, r.y),
                        biome: config.biome.clone(),
                        tier: config.tier,
                        spot_type: SpotType::Rock,
                    });
                }
                SceneObject::Plant(p) => {
                    flags.spots.push(SpotRespawnManager {
                        position: Vector2::new(p.x, p.y),
                        biome: config.biome.clone(),
                        tier: config.tier,
                        spot_type: SpotType::Plant,
                    });
                }
                SceneObject::Npc(n) => {
                    flags.npcs.push(NpcEntity {
                        npc_id: generate_npc_id(),
                        name: n.creature_name,
                        position: Vector2::new(n.x, n.y),
                        rotation: n.rotation.to_direction(),
                    });
                }
                _ => {}
            }
        }
        
        Ok(flags)
    }
    
    fn load_heightmap(name: &str) -> Result<Heightmap, MapError> {
        let path = format!("Metadata/Scenes/{}_Heightmap.json", name);
        if !std::path::Path::new(&path).exists() {
            // Return empty heightmap if file doesn't exist
            return Ok(Heightmap::empty());
        }
        
        let content = std::fs::read_to_string(&path)?;
        let json: serde_json::Value = serde_json::from_str(&content)?;
        
        // Parse heightmap data
        let heightmap_data = json.get("Heightmap")
            .and_then(|h| h.get("Data"))
            .and_then(|d| d.as_array())
            .ok_or_else(|| MapError::InvalidHeightmapFormat)?;
        
        let mut height_grid = HashMap::new();
        
        for entry in heightmap_data {
            let x = entry.get("X")
                .and_then(|v| v.as_i64())
                .ok_or_else(|| MapError::InvalidHeightmapFormat)? as i32;
            
            let y = entry.get("Y")
                .and_then(|v| v.as_i64())
                .ok_or_else(|| MapError::InvalidHeightmapFormat)? as i32;
            
            let z_heights = entry.get("Z")
                .and_then(|v| v.as_array())
                .ok_or_else(|| MapError::InvalidHeightmapFormat)?;
            
            let mut heights = Vec::new();
            for z in z_heights {
                if let Some(z_val) = z.as_i64() {
                    heights.push(z_val as i32);
                }
            }
            
            heights.sort();
            height_grid.insert((x, y), heights);
        }
        
        // Parse bounds
        let bounds_obj = json.get("LevelBounds")
            .ok_or_else(|| MapError::InvalidHeightmapFormat)?;
        
        let min_x = bounds_obj.get("MinX")
            .and_then(|v| v.as_i64())
            .unwrap_or(0) as f32;
        let min_y = bounds_obj.get("MinY")
            .and_then(|v| v.as_i64())
            .unwrap_or(0) as f32;
        let max_x = bounds_obj.get("MaxX")
            .and_then(|v| v.as_i64())
            .unwrap_or(0) as f32;
        let max_y = bounds_obj.get("MaxY")
            .and_then(|v| v.as_i64())
            .unwrap_or(0) as f32;
        
        let bounds = BoundingBox {
            min: Vector2::new(min_x, min_y),
            max: Vector2::new(max_x, max_y),
        };
        
        // Get resolution from settings or use default
        let resolution = 100; // Default resolution (configurable)
        
        Ok(Heightmap {
            bounds,
            height_grid,
            resolution,
        })
    }
}
```

### Heightmap Structure

```rust
pub struct Heightmap {
    bounds: BoundingBox,
    height_grid: HashMap<(i32, i32), Vec<i32>>, // (X, Y) -> Vec<Z heights>
    resolution: i32,
}

impl Heightmap {
    pub fn empty() -> Self {
        Self {
            bounds: BoundingBox::default(),
            height_grid: HashMap::new(),
            resolution: 100,
        }
    }
    
    pub fn bounds(&self) -> BoundingBox {
        self.bounds
    }
    
    pub fn get_heights_at(&self, x: f32, y: f32) -> Vec<f32> {
        // Round to grid coordinates based on resolution
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
    
    pub fn find_nearest_ground(&self, x: f32, y: f32, player_z: f32) -> Option<f32> {
        let heights = self.get_heights_at(x, y);
        if heights.is_empty() {
            return None;
        }
        
        // Find the height closest to player's Z position
        heights.iter()
            .min_by(|a, b| {
                let dist_a = (player_z - **a).abs();
                let dist_b = (player_z - **b).abs();
                dist_a.partial_cmp(&dist_b).unwrap_or(std::cmp::Ordering::Equal)
            })
            .copied()
    }
}
```

### Game Loop

```rust
impl Map {
    pub fn run_game_loop(&mut self) {
        const DELTA_TIME: f32 = 1.0 / 32.0;
        
        // Async loading
        self.load_blueprints_async();
        self.load_respawns_async();
        self.load_spots_async();
        
        let mut frame_time = Duration::from_secs_f32(DELTA_TIME);
        let mut target_time = Instant::now() + frame_time;
        
        loop {
            // Process task queue
            while let Some(task) = self.task_queue.lock().unwrap().pop_front() {
                task.execute();
            }
            
            // Update map
            if !self.tick() {
                break; // Map finalized
            }
            
            // Sleep until next frame
            let sleep_time = target_time.saturating_duration_since(Instant::now());
            if sleep_time.as_millis() > 1 {
                std::thread::sleep(sleep_time);
            }
            
            target_time += frame_time;
        }
    }
    
    fn tick(&mut self) -> bool {
        if self.is_finalized {
            return false;
        }
        
        self.current_time += Duration::from_secs_f32(1.0 / 32.0);
        
        // Process network events
        self.net_manager.process_events();
        
        // Update timers
        self.timer_queue.update(self.current_time);
        
        // Update entities
        for entity in self.dynamic_entities.values_mut() {
            if let Some(updatable) = entity.as_updatable() {
                updatable.update(self.current_time);
            }
        }
        
        // Update respawn managers
        for respawn in &mut self.respawn_managers {
            respawn.update(self.current_time, &mut self);
        }
        
        // Update spot managers
        for spot in &mut self.spot_managers {
            spot.update(self.current_time, &mut self);
        }
        
        // Update players
        for player in self.players.values_mut() {
            player.update(self.current_time);
        }
        
        // Update network
        self.net_manager.update(Duration::from_secs_f32(1.0 / 32.0));
        
        true
    }
}
```

### Entity Management

```rust
impl Map {
    pub fn add_entity(&mut self, entity: Entity) {
        if entity.is_destroyed() {
            entity.set_id(self.generate_entity_id());
            entity.set_scene(self.map_id);
            entity.initialize();
            
            let bbox = entity.bounding_box();
            
            // Add to grid cells
            for x in bbox.from.x..=bbox.to.x {
                for y in bbox.from.y..=bbox.to.y {
                    let cell = self.get_cell(Vector2i::new(x, y));
                    cell.enter(&entity);
                }
            }
            
            // Store entity
            match entity.entity_type() {
                EntityType::Player => {
                    if let Some(player) = entity.as_player() {
                        self.players.insert(player.character_id(), player);
                    }
                }
                EntityType::Creature => {
                    if let Some(creature) = entity.as_creature() {
                        self.creatures.insert(creature.creature_id(), creature);
                    }
                }
                EntityType::Npc => {
                    if let Some(npc) = entity.as_npc() {
                        self.npcs.insert(npc.npc_id(), npc);
                    }
                }
                _ => {
                    self.dynamic_entities.insert(entity.id(), entity);
                }
            }
        }
    }
    
    pub fn remove_entity(&mut self, entity_id: EntityId) {
        if let Some(entity) = self.dynamic_entities.remove(&entity_id) {
            let bbox = entity.bounding_box();
            
            // Remove from grid cells
            for x in bbox.from.x..=bbox.to.x {
                for y in bbox.from.y..=bbox.to.y {
                    let cell = self.get_cell(Vector2i::new(x, y));
                    cell.leave(&entity);
                }
            }
            
            entity.on_destroy();
        }
    }
}
```

## Integration with Other Systems

### Zone System

- Maps load zone configuration from metadata
- Zone type determines PvP and death mechanics
- Zone boundaries checked for entity position
- Resistance penalties applied based on zone

### Heightmap System

- Heightmaps loaded from JSON files exported by Unreal Engine
- Heightmap data used for flyhack detection (see [SECURITY.md](./SECURITY.md))
- Heightmap provides ground level data for pathfinding
- Heightmap validates player positions against terrain

### Sacred Lands System

- Sacred Lands loaded as special SafeZones
- Goddess Statues positioned at Sacred Land centers
- Immunity checks performed for Sacred Land boundaries
- Respawn system uses Sacred Land positions

### Teleportation System

- Teleports loaded from metadata files
- Waypoints loaded for map transitions
- Dynamic portals managed per map
- Teleportation validation and execution

### Respawn System

- Respawn managers loaded per map
- Biome and tier determine spawn types
- Respawn timers managed per map
- Creature and resource respawns handled separately

### Resource Gathering System

- Resource spots loaded per map
- Biome determines available resources
- Tier determines resource quality
- Spot respawn managers handle respawns

## Testing Considerations

### Unit Tests

- Map loading from API
- Waypoint loading and lookup
- Teleport loading and validation
- Zone loading and application
- Sacred Land loading and detection
- Respawn manager initialization
- Spot manager initialization
- Entity addition and removal
- Grid cell operations

### Integration Tests

- Map creation and initialization
- Entity updates in game loop
- Respawn system integration
- Spot respawn integration
- Teleportation between maps
- Waypoint transitions
- Zone boundary checks
- Sacred Land immunity checks

### Performance Tests

- Grid query performance
- Entity update performance
- Respawn manager performance
- Large map handling
- Many entities handling
- Network update performance

## TypeScript Implementation

### Maps Class Structure

```typescript
class Maps extends LinkedList<Maps> {
    public static maps: Map<string, Maps>
    public static deltaTime: number = 0.3
    public static foliageInitialData: Map<string, string>
    
    public id: string
    public namespace: string
    public mapTick: number
    public mapIndex: number = 20
    public entitiesIndexById: Map<string, Entity>
    public entitiesMapIndex: Map<string, Entity>
    public entitiesPersistedId: number = 1
    public respawns: Map<string, Respawn>
    public foliageIndex: Map<number, string>
    public foliage: Map<string, Gatherable>
}
```

### Entity Management

**Dual Index System:**
- **entitiesIndexById**: Indexed by entity.id (GUID) for fast lookup
- **entitiesMapIndex**: Indexed by mapIndex (short ID) for network efficiency

**Join Map Process:**
1. Check for duplicate entity (same ID)
2. Remove old entity if found (disconnect, save, destroy)
3. Generate new mapIndex (starts at 20, increments)
4. Set entity map and mapIndex
5. Add to both indices
6. Update area of interest
7. Validate and clean indices

**Index Validation:**
- Detects duplicate characterIds
- Removes disconnected duplicates
- Keeps connected entity
- Cleans up invalid references

### Foliage System

**parseLevelData:**
- Parses index (mesh ID to name mapping)
- Parses data (gatherable positions)
- Format: `index@data` where index is `id:mesh|id:mesh|...` and data is `meshIndex,type,x,y,z|...`
- Creates Gatherable instances
- Stores by location and meshIndex
- Refreshes initial data for client loading

**refreshMapInitialData:**
- Generates initial data string: `locRef,foliageId,enable,tick|...`
- Stores in static map for client loading
- Updated when foliage is collected

### Tick System

**Map Tick:**
- Called every deltaTime (300ms)
- Increments mapTick
- Updates all entities (Players, Creatures, Others)

**Respawn Tick:**
- Called every 60 seconds
- Ticks all respawns
- Ticks all foliage
- Handles respawn timers

### Respawn Integration

**getRespawns:**
- Loads respawns from database
- Creates Respawn instances
- Sets timer based on entity type (Boss: 86400s, Others: 300s)
- Stores in respawns map

**createRespawn:**
- Creates new respawn point
- Saves to database
- Creates Respawn instance
- Timer based on entity type

**removeRespawn:**
- Destroys respawn
- Removes from database
- Destroys spawned entity
- Removes from respawns map

## Summary

Maps are the core units of the game world, loaded from the central API and managing all entities, systems, and gameplay mechanics within their boundaries. Each map handles players, creatures, NPCs, zones, Sacred Lands, teleportation points, resource gathering spots, respawns, and dynamic portals. The system uses a grid-based spatial optimization for efficient entity queries and runs a dedicated game loop per map with async loading capabilities. Maps ensure consistent world structure across all game servers while maintaining server-specific entity instances and player data.

The TypeScript implementation uses a dual-index system for efficient entity lookup, manages respawns and foliage through tick-based systems, and integrates with the database for persistence. Entity management includes duplicate detection, index validation, and efficient cleanup operations.

