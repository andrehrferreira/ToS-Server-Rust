# Teleportation System Documentation

## Overview

The teleportation system provides multiple methods for players to travel across the game world, ensuring mobility and reducing travel time. The system includes city portals, mage portals, Sacred Land statues, and ancient monoliths, each with different costs, ranges, and use cases.

## Key Features

- **Five Teleportation Methods**: City Portals, Mage Portals, Sacred Land Statues, Ancient Monoliths, Dynamic Portals
- **Different Ranges**: From same-map travel to world-wide teleportation
- **Various Costs**: Gold, Mana, and Runic Crystals (Blue and Red)
- **Guild Integration**: Mage portals support guild and party members
- **Strategic Marking**: Mages can mark locations for future portals
- **Resource Management**: Different crystal types for different teleportation methods
- **Dynamic Portals**: Event-based and admin-created temporary portals

## Teleportation Methods

### 1. City Portals

**Description:**
City portals are permanent teleportation structures found in all major cities and settlements. They provide reliable, world-wide teleportation services for a gold cost.

**Location:**
- Found in all major cities
- Usually in central city areas
- Clearly marked and easily accessible
- Multiple portals per city (if city is large)

**Map Requirements:**
- **Every map MUST have 1-2 teleportation points** (City Portals or Ancient Monoliths)
- Ensures player mobility across all maps
- At least one teleportation point per map is mandatory
- Second point recommended for large maps

**Functionality:**
- **Range**: Teleport to any other city portal in the game world
- **Cost**: Gold (varies by distance)
- **Availability**: Always available (24/7)
- **No Cooldown**: Can be used repeatedly
- **No Requirements**: Available to all players

**Cost System:**
```
Base Cost: 100 gold
Distance Multiplier: 1 gold per 100 units of distance
Final Cost = Base Cost + (Distance / 100)

Example:
- Same city: 100 gold
- Nearby city (5,000 units): 150 gold
- Far city (50,000 units): 600 gold
```

**Usage:**
1. Player approaches city portal
2. Interact with portal
3. Portal shows list of available destinations
4. Select destination city
5. Pay gold cost
6. Instant teleportation

**Implementation:**
```rust
pub struct CityPortal {
    pub portal_id: PortalId,
    pub city_name: String,
    pub position: Vec3,
    pub map_id: MapId,
}

pub fn get_city_portal_destinations(
    current_portal: PortalId,
    all_portals: &[CityPortal],
) -> Vec<PortalDestination> {
    all_portals
        .iter()
        .filter(|portal| portal.portal_id != current_portal)
        .map(|portal| {
            let distance = calculate_distance(current_portal, portal.portal_id);
            let cost = calculate_portal_cost(distance);
            
            PortalDestination {
                portal_id: portal.portal_id,
                city_name: portal.city_name.clone(),
                position: portal.position,
                map_id: portal.map_id,
                cost: cost,
                distance,
            }
        })
        .collect()
}

pub fn teleport_via_city_portal(
    player: Entity,
    destination: PortalId,
    portals: &[CityPortal],
) -> Result<(), TeleportError> {
    // Get current portal
    let current_portal = get_player_portal(player)?;
    
    // Calculate cost
    let distance = calculate_distance(current_portal, destination);
    let cost = calculate_portal_cost(distance);
    
    // Check gold
    if !has_enough_gold(player, cost) {
        return Err(TeleportError::InsufficientGold);
    }
    
    // Deduct gold
    deduct_gold(player, cost);
    
    // Get destination position
    let dest_portal = portals.iter()
        .find(|p| p.portal_id == destination)
        .ok_or(TeleportError::InvalidDestination)?;
    
    // Teleport
    teleport_player(player, dest_portal.position, dest_portal.map_id)?;
    
    // Notify player
    notify_teleport(player, dest_portal.city_name.clone());
    
    Ok(())
}
```

**Advantages:**
- Reliable and always available
- World-wide reach
- Simple gold cost
- No special requirements

**Disadvantages:**
- Only connects cities
- Cannot teleport to custom locations
- Gold cost can be expensive for long distances

### 2. Mage Portals

**Description:**
Mages with portal magic can mark locations and open portals for themselves, guild members, and other players. This provides flexible teleportation to custom locations but requires significant resources.

**Requirements:**
- **Mage Class**: Player must be a mage or have portal magic skill
- **Portal Magic Skill**: Requires high-level portal magic skill
- **Mana Cost**: Large amount of mana required
- **Runic Crystals**: Blue Runic Crystals (consumed per portal)

**Location Marking:**
- Mages can mark up to 5 locations (skill-dependent)
- Marked locations persist until changed
- Can mark any accessible location
- Marks are personal to the mage

**Portal Opening:**
- Mages can open portals to marked locations
- Portals can be opened for:
  - **Self**: Only the mage
  - **Guild Members**: All guild members can use
  - **Party Members**: All party members can use
  - **Public**: Any player can use (optional)
- Portal duration: Limited time (skill-dependent)
- Portal size: Allows multiple players to pass through

**Costs:**
- **Mana**: Large amount (500-2000 mana, skill-dependent)
- **Blue Runic Crystals**: 1-3 crystals per portal (distance-dependent)
- **Cooldown**: Portal magic has cooldown period

**Blue Runic Crystals:**
- **Obtainment**: Can be collected from world or purchased from NPC mages
- **NPC Mages**: Sell crystals in major cities
- **Collection**: Found in magical areas, dungeons, or as drops
- **Price**: Varies by NPC and location

**Usage Flow:**
1. Mage marks a location (uses skill)
2. Mage opens portal to marked location
3. Consumes mana and Blue Runic Crystals
4. Portal appears at mage's location
5. Designated players can use portal
6. Portal closes after duration expires

**Implementation:**
```rust
pub struct MarkedLocation {
    pub location_id: LocationId,
    pub name: String,
    pub position: Vec3,
    pub map_id: MapId,
    pub marked_time: Instant,
}

#[derive(Component)]
pub struct MagePortalMagic {
    pub skill_level: u8,
    pub max_marks: u8, // Based on skill level
    pub marked_locations: Vec<MarkedLocation>,
    pub portal_cooldown: Option<Instant>,
}

pub struct Portal {
    pub portal_id: PortalId,
    pub caster: Entity,
    pub destination: Vec3,
    pub destination_map: MapId,
    pub access_type: PortalAccess,
    pub created_time: Instant,
    pub duration: Duration,
    pub position: Vec3,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum PortalAccess {
    SelfOnly,
    Guild,
    Party,
    Public,
}

pub fn mark_location(
    mage: Entity,
    position: Vec3,
    map_id: MapId,
    name: String,
    portal_magic: &mut MagePortalMagic,
) -> Result<(), PortalError> {
    // Check mark limit
    if portal_magic.marked_locations.len() >= portal_magic.max_marks as usize {
        return Err(PortalError::MaxMarksReached);
    }
    
    // Create mark
    let mark = MarkedLocation {
        location_id: generate_location_id(),
        name,
        position,
        map_id,
        marked_time: Instant::now(),
    };
    
    portal_magic.marked_locations.push(mark);
    
    Ok(())
}

pub fn open_portal(
    mage: Entity,
    marked_location: LocationId,
    access_type: PortalAccess,
    portal_magic: &mut MagePortalMagic,
    inventory: &mut Inventory,
) -> Result<Portal, PortalError> {
    // Check cooldown
    if let Some(cooldown_end) = portal_magic.portal_cooldown {
        if cooldown_end > Instant::now() {
            return Err(PortalError::OnCooldown);
        }
    }
    
    // Find marked location
    let location = portal_magic.marked_locations
        .iter()
        .find(|loc| loc.location_id == marked_location)
        .ok_or(PortalError::LocationNotFound)?;
    
    // Check mana
    let mana_cost = calculate_mana_cost(portal_magic.skill_level, location);
    if !has_enough_mana(mage, mana_cost) {
        return Err(PortalError::InsufficientMana);
    }
    
    // Check Blue Runic Crystals
    let crystal_cost = calculate_crystal_cost(location);
    if !has_enough_crystals(inventory, CrystalType::Blue, crystal_cost) {
        return Err(PortalError::InsufficientCrystals);
    }
    
    // Consume resources
    consume_mana(mage, mana_cost);
    consume_crystals(inventory, CrystalType::Blue, crystal_cost);
    
    // Create portal
    let portal = Portal {
        portal_id: generate_portal_id(),
        caster: mage,
        destination: location.position,
        destination_map: location.map_id,
        access_type,
        created_time: Instant::now(),
        duration: calculate_portal_duration(portal_magic.skill_level),
        position: get_position(mage),
    };
    
    // Apply cooldown
    portal_magic.portal_cooldown = Some(
        Instant::now() + calculate_cooldown(portal_magic.skill_level)
    );
    
    // Spawn portal entity
    spawn_portal_entity(portal.clone());
    
    Ok(portal)
}

pub fn use_mage_portal(
    player: Entity,
    portal: &Portal,
) -> Result<(), PortalError> {
    // Check access
    match portal.access_type {
        PortalAccess::SelfOnly => {
            if player != portal.caster {
                return Err(PortalError::AccessDenied);
            }
        }
        PortalAccess::Guild => {
            if !are_same_guild(player, portal.caster) {
                return Err(PortalError::AccessDenied);
            }
        }
        PortalAccess::Party => {
            if !are_same_party(player, portal.caster) {
                return Err(PortalError::AccessDenied);
            }
        }
        PortalAccess::Public => {
            // Anyone can use
        }
    }
    
    // Check portal is still active
    if portal.created_time + portal.duration < Instant::now() {
        return Err(PortalError::PortalExpired);
    }
    
    // Teleport player
    teleport_player(player, portal.destination, portal.destination_map)?;
    
    Ok(())
}
```

**Advantages:**
- Custom location marking
- Guild and party support
- Flexible access control
- Strategic positioning

**Disadvantages:**
- Requires mage class/skill
- High resource cost (mana + crystals)
- Limited marks per mage
- Portal duration is limited
- Cooldown between portals

### 3. Sacred Land Statues

**Description:**
Goddess Statues in Sacred Lands provide teleportation services within the same map. This is the most accessible form of teleportation but limited to same-map travel.

**Location:**
- Every Sacred Land has a Goddess Statue
- See [SACRED_LANDS.md](./SACRED_LANDS.md) for complete details

**Functionality:**
- **Range**: Same map only (cannot cross map boundaries)
- **Destinations**: Other Sacred Lands on the same map
- **Cost**: May be free or have gold cost (distance-based)
- **Cooldown**: May have cooldown (configurable)

**Usage:**
1. Player interacts with Goddess Statue
2. Statue shows list of Sacred Lands on current map
3. Select destination
4. Pay cost (if any)
5. Instant teleportation to destination Sacred Land

**Advantages:**
- Available in every map
- Easy to access
- Low or no cost
- Convenient for map exploration

**Disadvantages:**
- Limited to same map
- Cannot teleport to custom locations
- Only connects Sacred Lands

**Note:** See [SACRED_LANDS.md](./SACRED_LANDS.md) for complete documentation on Sacred Land teleportation.

### 4. Ancient Monoliths

**Description:**
Ancient Monoliths are powerful teleportation structures that allow teleportation to any location on the current map. They are rare and require Red Runic Crystals to operate.

**Location:**
- Found in various locations across maps
- Usually in significant or ancient areas
- Can serve as teleportation points for maps without cities
- May be hidden or require discovery

**Map Requirements:**
- **Every map MUST have 1-2 teleportation points** (City Portals or Ancient Monoliths)
- If a map has no city, it must have Ancient Monoliths as teleportation points
- Ensures player mobility across all maps
- At least one teleportation point per map is mandatory

**Functionality:**
- **Range**: Any location on the current map
- **Cost**: Red Runic Crystals (consumed per teleport)
- **No Cooldown**: Can be used repeatedly (if you have crystals)
- **No Requirements**: Available to all players

**Red Runic Crystals:**
- **Obtainment**: Can be collected from world or purchased from Auction House
- **Auction House**: Players can buy/sell crystals
- **Collection**: Found in ancient areas, dungeons, or as rare drops
- **Price**: Varies by market demand

**Usage:**
1. Player approaches Ancient Monolith
2. Interact with monolith
3. Select destination on map (or enter coordinates)
4. Consume Red Runic Crystal(s)
5. Instant teleportation to destination

**Cost System:**
```
Base Cost: 1 Red Runic Crystal
Distance Multiplier: +1 crystal per 10,000 units of distance
Final Cost = 1 + (Distance / 10,000)

Example:
- Short distance (5,000 units): 1 crystal
- Medium distance (15,000 units): 2 crystals
- Long distance (50,000 units): 6 crystals
```

**Implementation:**
```rust
pub struct AncientMonolith {
    pub monolith_id: MonolithId,
    pub position: Vec3,
    pub map_id: MapId,
    pub name: String,
}

pub fn use_ancient_monolith(
    player: Entity,
    monolith: &AncientMonolith,
    destination: Vec3,
    inventory: &mut Inventory,
) -> Result<(), TeleportError> {
    // Validate destination is on same map
    let destination_map = get_map_for_position(destination);
    if destination_map != monolith.map_id {
        return Err(TeleportError::DifferentMap);
    }
    
    // Calculate distance and cost
    let distance = monolith.position.distance(destination);
    let crystal_cost = calculate_monolith_cost(distance);
    
    // Check Red Runic Crystals
    if !has_enough_crystals(inventory, CrystalType::Red, crystal_cost) {
        return Err(TeleportError::InsufficientCrystals);
    }
    
    // Consume crystals
    consume_crystals(inventory, CrystalType::Red, crystal_cost);
    
    // Teleport player
    teleport_player(player, destination, monolith.map_id)?;
    
    // Notify player
    notify_teleport(player, format!("Teleported via Ancient Monolith"));
    
    Ok(())
}

fn calculate_monolith_cost(distance: f32) -> u8 {
    let base_cost = 1;
    let distance_cost = (distance / 10_000.0).ceil() as u8;
    base_cost + distance_cost.max(0)
}
```

**Advantages:**
- Teleport to any location on map
- No class requirements
- Flexible destination selection
- Strategic positioning

**Disadvantages:**
- Requires Red Runic Crystals
- Crystals can be expensive
- Limited to same map
- Monoliths are not everywhere

### 5. Dynamic Portals

**Description:**
Dynamic Portals are temporary teleportation structures that can appear automatically during events or be created by administrators. They provide flexible teleportation to event instances or custom locations and can be removed at any time.

**Spawn Conditions:**
- **Event-Based**: Automatically spawn during events (e.g., Shadowgates)
- **Admin-Created**: Created by administrators via commands
- **Temporary**: Can disappear after event ends or be removed by admins
- **Dynamic Location**: Can appear in predefined or random locations

**Functionality:**
- **Range**: Can teleport to any map or instanced event level
- **Cost**: Usually free (event portals) or configurable
- **Availability**: Temporary, event or admin-controlled
- **Access**: May be restricted to event participants or open to all

**Event Portals:**
- Spawn automatically during events
- Lead to instanced event levels
- Examples: Shadowgates, seasonal events, special dungeons
- Usually free to use
- Disappear when event ends

**Admin Portals:**
- Created by administrators via command: `/portal <position> <map> <destination>`
- Can be placed anywhere
- Can lead to any map or location
- Useful for testing, events, or special situations
- Can be removed via admin interface

**Admin Commands:**
```
/portal <position> <map> <destination>
Creates a dynamic portal at specified position leading to destination
- position: Current position (x, y, z) or "here" for current location
- map: Destination map ID or name
- destination: Destination position (x, y, z) or landmark name

Examples:
/portal here map_001 1000 500 2000
/portal 5000 100 3000 map_002 2000 100 1500
/portal here shadowgate_dungeon entrance

/portal remove <portal_id>
Removes a specific dynamic portal by ID

/portal list
Lists all active dynamic portals
```

**Admin Interface:**
- Visual interface for managing dynamic portals
- Can create portals by clicking on map
- Can remove portals by selecting and deleting
- Shows all active portals on map
- Can set portal properties (access, cost, duration)

**Implementation:**
```rust
pub struct DynamicPortal {
    pub portal_id: PortalId,
    pub position: Vec3,
    pub map_id: MapId,
    pub destination: Vec3,
    pub destination_map: MapId,
    pub portal_type: DynamicPortalType,
    pub created_by: Option<AdminId>,
    pub event_id: Option<EventId>,
    pub access_control: PortalAccess,
    pub cost: Option<TeleportCost>,
    pub duration: Option<Duration>,
    pub created_time: Instant,
    pub is_active: bool,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum DynamicPortalType {
    Event,      // Spawned by event system
    Admin,      // Created by admin
    Shadowgate, // Special shadowgate event portal
}

pub fn create_admin_portal(
    admin: Entity,
    position: Vec3,
    map_id: MapId,
    destination: Vec3,
    destination_map: MapId,
) -> Result<DynamicPortal, PortalError> {
    // Validate admin permissions
    if !is_admin(admin) {
        return Err(PortalError::PermissionDenied);
    }
    
    // Validate positions
    if !is_valid_position(position, map_id) {
        return Err(PortalError::InvalidPosition);
    }
    
    if !is_valid_position(destination, destination_map) {
        return Err(PortalError::InvalidDestination);
    }
    
    // Create portal
    let portal = DynamicPortal {
        portal_id: generate_portal_id(),
        position,
        map_id,
        destination,
        destination_map,
        portal_type: DynamicPortalType::Admin,
        created_by: Some(get_admin_id(admin)),
        event_id: None,
        access_control: PortalAccess::Public,
        cost: None, // Free by default
        duration: None, // Permanent until removed
        created_time: Instant::now(),
        is_active: true,
    };
    
    // Spawn portal entity
    spawn_dynamic_portal_entity(portal.clone());
    
    // Notify admin
    notify_portal_created(admin, portal.portal_id);
    
    Ok(portal)
}

pub fn remove_dynamic_portal(
    admin: Entity,
    portal_id: PortalId,
    portals: &mut Vec<DynamicPortal>,
) -> Result<(), PortalError> {
    // Validate admin permissions
    if !is_admin(admin) {
        return Err(PortalError::PermissionDenied);
    }
    
    // Find portal
    let portal = portals.iter_mut()
        .find(|p| p.portal_id == portal_id)
        .ok_or(PortalError::PortalNotFound)?;
    
    // Deactivate portal
    portal.is_active = false;
    
    // Remove portal entity
    despawn_portal_entity(portal_id);
    
    // Remove from list
    portals.retain(|p| p.portal_id != portal_id);
    
    // Notify admin
    notify_portal_removed(admin, portal_id);
    
    Ok(())
}

pub fn spawn_event_portal(
    event_id: EventId,
    position: Vec3,
    map_id: MapId,
    destination: Vec3,
    destination_map: MapId,
    event_config: &EventConfig,
) -> DynamicPortal {
    let portal = DynamicPortal {
        portal_id: generate_portal_id(),
        position,
        map_id,
        destination,
        destination_map,
        portal_type: DynamicPortalType::Event,
        created_by: None,
        event_id: Some(event_id),
        access_control: event_config.portal_access,
        cost: event_config.portal_cost.clone(),
        duration: event_config.portal_duration,
        created_time: Instant::now(),
        is_active: true,
    };
    
    // Spawn portal entity
    spawn_dynamic_portal_entity(portal.clone());
    
    // Notify players in area
    broadcast_portal_spawned(position, map_id, event_id);
    
    portal
}

pub fn cleanup_expired_portals(
    portals: &mut Vec<DynamicPortal>,
) {
    let now = Instant::now();
    
    portals.retain(|portal| {
        // Check if portal expired
        if let Some(duration) = portal.duration {
            if portal.created_time + duration < now {
                despawn_portal_entity(portal.portal_id);
                return false; // Remove expired portal
            }
        }
        
        // Keep active portals
        portal.is_active
    });
}
```

**Usage:**
1. Portal appears (event spawn or admin creation)
2. Players see portal and can interact with it
3. Portal shows destination information
4. Player interacts and confirms teleportation
5. Instant teleportation to destination
6. Portal remains until event ends or admin removes it

**Advantages:**
- Flexible and temporary
- Can be created anywhere
- Useful for events and special situations
- Admin control for testing and management
- Can lead to instanced event levels

**Disadvantages:**
- Temporary (not permanent)
- Requires admin or event system
- May not be available when needed
- Can be removed at any time

**Event Integration:**
- Shadowgates spawn portals automatically
- Event portals lead to instanced event maps
- Portals disappear when event ends
- Can have event-specific access rules

**Admin Management:**
- Create portals via command or interface
- Remove portals via command or interface
- Monitor all active portals
- Set portal properties (access, cost, duration)

## Runic Crystals

### Blue Runic Crystals

**Usage:**
- Required for Mage Portals
- Consumed when opening portals
- Cost varies by portal distance

**Obtainment:**
- **Collection**: Found in magical areas, dungeons, magical creatures
- **NPC Mages**: Can be purchased from mage NPCs in cities
- **Price Range**: 500-2000 gold per crystal (NPC dependent)

**Properties:**
- Stackable item
- Can be traded between players
- Required for mage portal magic
- Cannot be used for other teleportation methods

### Red Runic Crystals

**Usage:**
- Required for Ancient Monolith teleportation
- Consumed per teleport
- Cost varies by distance

**Obtainment:**
- **Collection**: Found in ancient areas, high-level dungeons, rare drops
- **Auction House**: Can be purchased from other players
- **Price Range**: Varies by market demand (typically 1000-5000 gold)

**Properties:**
- Stackable item
- Can be traded between players
- Required for monolith teleportation
- Cannot be used for other teleportation methods

## Teleportation Comparison

### Range Comparison

| Method | Range | Cross-Map | Temporary |
|--------|-------|-----------|------------|
| City Portals | World-wide | Yes | No |
| Mage Portals | Marked locations | Yes (if marked) | Yes (duration-limited) |
| Sacred Land Statues | Same map | No | No |
| Ancient Monoliths | Same map | No | No |
| Dynamic Portals | Any location | Yes | Yes |

### Cost Comparison

| Method | Cost Type | Typical Cost |
|--------|-----------|--------------|
| City Portals | Gold | 100-600 gold |
| Mage Portals | Mana + Blue Crystals | 500-2000 mana + 1-3 crystals |
| Sacred Land Statues | Gold (optional) | Free or 50-200 gold |
| Ancient Monoliths | Red Crystals | 1-6 crystals |
| Dynamic Portals | Configurable | Usually free (events) or admin-set |

### Availability Comparison

| Method | Availability | Requirements |
|--------|--------------|--------------|
| City Portals | Always | None |
| Mage Portals | Skill-dependent | Mage class, Portal Magic skill |
| Sacred Land Statues | Always | None |
| Ancient Monoliths | Location-dependent | None |
| Dynamic Portals | Event/Admin-dependent | Event participation or admin creation |

## Strategic Usage

### Early Game

**Recommended Methods:**
- **City Portals**: Primary method for world travel
- **Sacred Land Statues**: For same-map exploration
- Focus on gold accumulation for portal costs

### Mid Game

**Recommended Methods:**
- **City Portals**: Still primary for world travel
- **Sacred Land Statues**: Convenient same-map travel
- **Mage Portals**: If playing mage, start marking locations
- Begin collecting crystals for future use

### Late Game

**Recommended Methods:**
- **City Portals**: Reliable world travel
- **Mage Portals**: Strategic guild/party portals
- **Ancient Monoliths**: Flexible same-map positioning
- **Sacred Land Statues**: Quick same-map travel
- Use all methods strategically based on situation

### Guild Strategies

**Mage Portal Coordination:**
- Designated portal mages mark strategic locations
- Guild leaders coordinate portal opening
- Mark locations near resources, dungeons, or objectives
- Share Blue Crystals among guild members

**Resource Management:**
- Stockpile crystals for important operations
- Coordinate crystal collection
- Use portals for guild events and raids

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Component)]
pub struct TeleportationComponent {
    pub last_teleport_time: Option<Instant>,
    pub teleport_cooldown: Duration,
}

pub struct PortalDestination {
    pub destination_id: DestinationId,
    pub name: String,
    pub position: Vec3,
    pub map_id: MapId,
    pub cost: TeleportCost,
    pub distance: f32,
}

#[derive(Debug, Clone)]
pub enum TeleportCost {
    Gold(u64),
    Crystals(CrystalType, u8),
    Free,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum CrystalType {
    Blue, // For mage portals
    Red,  // For ancient monoliths
}
```

### Teleportation System

```rust
pub fn teleport_player(
    player: Entity,
    destination: Vec3,
    destination_map: MapId,
) -> Result<(), TeleportError> {
    // Validate destination
    if !is_valid_destination(destination, destination_map) {
        return Err(TeleportError::InvalidDestination);
    }
    
    // Check cooldown
    let teleport_comp = get_component::<TeleportationComponent>(player)?;
    if let Some(last_teleport) = teleport_comp.last_teleport_time {
        if last_teleport + teleport_comp.teleport_cooldown > Instant::now() {
            return Err(TeleportError::OnCooldown);
        }
    }
    
    // Get current position
    let current_pos = get_position(player);
    let current_map = get_map_for_entity(player);
    
    // Cross-map teleportation
    if current_map != destination_map {
        handle_cross_map_teleport(player, current_map, destination_map)?;
    }
    
    // Set new position
    set_position(player, destination);
    
    // Update cooldown
    update_teleport_cooldown(player);
    
    // Notify player
    notify_teleport(player, destination);
    
    Ok(())
}
```

## Integration with Other Systems

### Zone System

**Integration:**
- Teleportation respects zone boundaries
- Cannot teleport into restricted areas
- Zone type affects teleportation availability
- See [ZONES.md](./ZONES.md) for zone details

### Guild System

**Integration:**
- Mage portals support guild members
- Guild coordination for portal usage
- Shared crystal resources
- Strategic portal placement

### Economy System

**Integration:**
- Gold costs for city portals
- Crystal trading in Auction House
- NPC crystal vendors
- Market prices for crystals

### Skill System

**Integration:**
- Portal Magic skill for mages
- Skill level affects portal capabilities
- Mana costs scale with skill
- Cooldowns reduce with skill level

### Event System

**Integration:**
- Dynamic portals spawn automatically during events
- Event portals lead to instanced event levels
- Shadowgates create portals automatically
- Portals disappear when events end
- Event-specific access rules and costs

### Admin System

**Integration:**
- Admins can create dynamic portals via commands
- Admin interface for portal management
- Portal removal and monitoring capabilities
- Testing and event management tools
- Portal property configuration (access, cost, duration)

## Testing Considerations

### Unit Tests

**Test Cases:**
- Cost calculations for each method
- Distance calculations
- Crystal consumption
- Cooldown enforcement
- Access control (mage portals)

### Integration Tests

**Test Cases:**
- Teleportation between cities
- Mage portal creation and usage
- Sacred Land teleportation
- Ancient Monolith teleportation
- Cross-map teleportation
- Guild/party portal access
- Dynamic portal creation and removal
- Event portal spawning
- Admin portal management

### Balance Tests

**Test Cases:**
- Cost balance (gold and crystals)
- Crystal availability and pricing
- Portal cooldown balance
- Mage portal mark limits
- Distance cost scaling

## Map Requirements

### Mandatory Teleportation Infrastructure

**Every Map Must Have:**
1. **At least 1 Sacred Land** (see [SACRED_LANDS.md](./SACRED_LANDS.md))
   - Provides safe respawn point
   - Enables same-map teleportation between Sacred Lands
   - Mandatory requirement for all maps

2. **1-2 Teleportation Points** (City Portals or Ancient Monoliths)
   - Ensures player mobility across the map
   - At least one teleportation point is mandatory
   - Second point recommended for large maps
   - Can be combination of City Portals and Ancient Monoliths

**Placement Strategy:**
- Sacred Land usually near map entrance
- Teleportation points distributed across map
- Large maps: Multiple Sacred Lands + 2 teleportation points
- Small maps: 1 Sacred Land + 1 teleportation point
- Ensures players can always travel efficiently

**Example Map Configurations:**

**Small Map:**
- 1 Sacred Land (near entrance)
- 1 City Portal or Ancient Monolith (central location)

**Medium Map:**
- 1 Sacred Land (near entrance)
- 1-2 City Portals or Ancient Monoliths (strategic locations)

**Large Map:**
- 2-3 Sacred Lands (distributed across map)
- 2 City Portals or Ancient Monoliths (key locations)

## Summary

The teleportation system:

- Provides five distinct teleportation methods with different ranges and costs
- **City Portals**: World-wide travel for gold (most accessible, permanent)
- **Mage Portals**: Custom locations for mages (guild/party support, requires Blue Crystals)
- **Sacred Land Statues**: Same-map travel (convenient, low cost, permanent)
- **Ancient Monoliths**: Any location on map (requires Red Crystals, permanent)
- **Dynamic Portals**: Temporary portals for events or admin use (flexible, temporary)
- Uses different resources: Gold, Mana, Blue Runic Crystals, Red Runic Crystals
- Supports various playstyles and strategies
- Integrates with guild, economy, skill, event, and admin systems
- **Mandatory Requirements**: Every map must have at least 1 Sacred Land and 1-2 teleportation points
- **Admin Control**: Dynamic portals can be created and managed by administrators

**Key Benefits:**
- **Mobility**: Reduces travel time significantly
- **Accessibility**: Multiple methods for different situations
- **Strategy**: Different methods for different needs
- **Economy**: Creates demand for crystals and gold
- **Guild Play**: Mage portals support coordinated guild activities

This system ensures players can travel efficiently while maintaining resource management and strategic decision-making. Each method serves different purposes, from reliable world travel to strategic positioning and guild coordination.

