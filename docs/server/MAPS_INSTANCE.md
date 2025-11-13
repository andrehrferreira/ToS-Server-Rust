# Map Instances System

## Overview

The Map Instances system allows creating isolated instances of maps for specific groups of players. This system is essential for events, dungeons, and scenarios where groups need their own isolated copy of a map. Each instance runs in a separate thread with its own entity list, preventing performance degradation from looping through the complete entity list of the main map.

## Key Features

- **Isolated Instances**: Each instance is completely isolated from the main map and other instances
- **Thread Separation**: Each instance runs in its own dedicated thread with its own game loop
- **On-Demand Creation**: Instances are created automatically when needed
- **Automatic Cleanup**: Instances are automatically destroyed when empty or expired
- **Group Isolation**: Players in an instance only see and interact with entities in their instance
- **Performance Optimization**: Prevents looping through all entities in the main map
- **Event Support**: Essential for event scenarios requiring isolated environments
- **Dungeon Support**: Used for dungeon instances where groups need private copies

## Use Cases

### Events

**Event Scenarios:**
- Battle Castle events requiring isolated battlefields
- Treasure Hunt events with private instances
- Champion events with isolated arenas
- Any event requiring group isolation

**Benefits:**
- Each group gets their own instance
- No interference from other players
- Clean entity lists for better performance
- Easy cleanup when event ends

### Dungeons

**Dungeon Scenarios:**
- Group dungeons requiring private instances
- Raid dungeons with multiple groups
- Challenge dungeons with isolated environments
- Any dungeon requiring group isolation

**Benefits:**
- Each group enters their own dungeon instance
- No competition for resources or bosses
- Isolated progression tracking
- Automatic cleanup when group leaves

## Instance Lifecycle

### Creation

**Trigger Conditions:**
- Player/group requests entry to instanced map
- Event starts requiring instances
- Dungeon entry point activated
- Admin command creates instance

**Creation Process:**
1. Check if instance pool available (if using pooling)
2. Create new map instance from template
3. Initialize instance with base map configuration
4. Start dedicated thread for instance game loop
5. Register instance in instance manager
6. Assign players/groups to instance
7. Begin instance game loop

**Creation Speed:**
- Instances must be created quickly (< 1 second)
- Template-based loading for fast initialization
- Minimal configuration required
- Pre-loaded resources when possible

### Runtime

**Instance Operation:**
- Runs independently in separate thread
- Own game loop with fixed tick rate
- Own entity list (players, creatures, NPCs, etc.)
- Own respawn managers
- Own spatial grid system
- Own network manager
- No interaction with main map or other instances

**Entity Isolation:**
- Players only see entities in their instance
- Creatures spawn only in their instance
- NPCs exist only in their instance
- Items dropped only in their instance
- No cross-instance visibility or interaction

**Performance Benefits:**
- Small entity lists per instance
- Fast loops through instance entities only
- No need to filter main map entities
- Reduced memory footprint per instance
- Better cache locality

### Cleanup

**Cleanup Triggers:**
- Instance becomes empty (no players)
- Instance expiration time reached
- Event ends
- Admin command destroys instance
- Instance error or crash

**Cleanup Process:**
1. Stop accepting new players
2. Save all player data
3. Notify players of instance closure
4. Transfer players back to main map (if applicable)
5. Stop instance game loop thread
6. Clean up all entities
7. Release instance resources
8. Unregister from instance manager
9. Destroy instance

**Cleanup Speed:**
- Cleanup must be fast (< 2 seconds)
- Graceful shutdown process
- Player data saved before destruction
- Resources released immediately

## Instance Management

### Instance Manager

**Responsibilities:**
- Track all active instances
- Create instances on demand
- Destroy instances when needed
- Assign players to instances
- Monitor instance health
- Balance instance load

**Instance Tracking:**
- Instance ID mapping
- Instance state tracking (active, closing, closed)
- Player-to-instance mapping
- Instance metadata (creation time, player count, etc.)

### Instance Pooling (Optional)

**Pooling Strategy:**
- Pre-create instance templates
- Keep warm instances ready
- Reuse instance structures
- Reduce creation overhead

**Benefits:**
- Faster instance creation
- Reduced allocation overhead
- Better memory management
- Predictable performance

**Trade-offs:**
- Memory overhead for pooled instances
- Complexity in pool management
- May not be needed if creation is fast enough

## Instance Configuration

### Base Map Template

**Template Loading:**
- Load base map configuration from API
- Load metadata (waypoints, teleports, flags)
- Load heightmap
- Load respawn points
- Load NPC configurations

**Template Caching:**
- Cache templates for fast instance creation
- Pre-load common templates
- Share templates across instances
- Update templates when map changes

### Instance-Specific Configuration

**Per-Instance Settings:**
- Instance ID (unique identifier)
- Player capacity (max players per instance)
- Expiration time (auto-cleanup timer)
- Event-specific rules (if event instance)
- Dungeon-specific rules (if dungeon instance)
- Custom spawn points
- Custom respawn timers

## Thread Management

### Thread Per Instance

**Thread Structure:**
- One thread per instance
- Dedicated game loop per thread
- Independent tick rate
- Isolated memory space
- No shared state with main map

**Thread Lifecycle:**
- Thread created when instance created
- Thread runs instance game loop
- Thread stopped when instance destroyed
- Thread cleanup on instance destruction

**Thread Safety:**
- No shared mutable state between instances
- Instance manager uses channels for communication
- Player assignment uses atomic operations
- Clean shutdown signals

### Game Loop Per Instance

**Loop Structure:**
- Same structure as main map game loop
- Fixed tick rate (1/32 seconds)
- Entity updates per tick
- Respawn manager updates
- Network packet processing
- Timer queue updates

**Performance:**
- Small entity lists = fast loops
- No need to filter entities
- Better cache locality
- Predictable performance

## Player Assignment

### Entry Points

**Entry Methods:**
- Portal/waypoint entry
- Event entry point
- Dungeon entrance
- Admin teleport
- Command-based entry

**Assignment Process:**
1. Player requests instance entry
2. Check if instance exists for player's group
3. Create instance if needed
4. Assign player to instance
5. Transfer player to instance
6. Update player's map reference
7. Notify client of map change

### Group Assignment

**Group-Based Instances:**
- Party members share instance
- Guild members can share instance (if configured)
- Event participants share instance
- Dungeon groups share instance

**Assignment Rules:**
- Same group = same instance
- Different groups = different instances
- Instance capacity limits
- Group size limits

### Transfer Process

**Player Transfer:**
1. Save player state in current map
2. Remove player from current map entities
3. Add player to instance entities
4. Update player's map reference
5. Send map change packet to client
6. Client loads new map instance

## Entity Management

### Instance Entities

**Entity Types:**
- Players (assigned to instance)
- Creatures (spawned in instance)
- NPCs (loaded in instance)
- Projectiles (created in instance)
- Items (dropped in instance)
- Effects (visual/audio in instance)

**Entity Isolation:**
- Entities exist only in their instance
- No cross-instance entity visibility
- No cross-instance entity interaction
- Independent entity IDs per instance

### Respawn System

**Instance Respawns:**
- Each instance has own respawn managers
- Creatures respawn independently per instance
- Resource spots respawn independently
- Respawn timers per instance
- No shared respawn state

**Respawn Configuration:**
- Same respawn points as base map
- Same respawn timers as base map
- Independent respawn tracking
- Instance-specific respawn limits

## Network Management

### Instance Network

**Network Isolation:**
- Each instance has own network manager
- Packets sent only to instance players
- No cross-instance packet routing
- Independent packet queues

**Packet Processing:**
- Process packets for instance players only
- Send updates to instance players only
- No need to filter by instance
- Reduced packet processing overhead

### Client Synchronization

**Client Updates:**
- Client receives instance map name
- Client loads instance map scene
- Client receives instance entity updates
- Client sends packets to instance
- No cross-instance updates

## Performance Considerations

### Memory Usage

**Memory Per Instance:**
- Base map template (shared)
- Instance-specific data (small)
- Entity lists (small per instance)
- Spatial grid (small per instance)
- Network buffers (small per instance)

**Memory Optimization:**
- Share base map templates
- Minimal instance-specific data
- Efficient entity storage
- Small spatial grids

### CPU Usage

**CPU Per Instance:**
- One thread per instance
- Game loop overhead per instance
- Entity update overhead per instance
- Network processing overhead per instance

**CPU Optimization:**
- Small entity lists = fast updates
- No filtering overhead
- Better cache locality
- Predictable performance

### Scalability

**Scaling Considerations:**
- Limit concurrent instances per server
- Instance pooling for common maps
- Instance cleanup when empty
- Resource limits per instance

**Scaling Limits:**
- Maximum instances per server
- Maximum players per instance
- Maximum entities per instance
- Maximum memory per instance

## Integration Points

### Event System

**Event Integration:**
- Events create instances automatically
- Event rules applied to instances
- Event cleanup destroys instances
- Event participants assigned to instances

### Dungeon System

**Dungeon Integration:**
- Dungeons create instances on entry
- Dungeon rules applied to instances
- Dungeon completion triggers cleanup
- Dungeon groups assigned to instances

### Map System

**Map Integration:**
- Instances based on map templates
- Map configuration loaded per instance
- Map metadata shared across instances
- Map changes propagate to new instances

### Player System

**Player Integration:**
- Players assigned to instances
- Player state saved on transfer
- Player data isolated per instance
- Player actions affect only their instance

## Monitoring and Debugging

### Instance Monitoring

**Metrics Tracked:**
- Active instance count
- Players per instance
- Instance creation rate
- Instance destruction rate
- Instance lifetime
- Instance memory usage
- Instance CPU usage

### Debugging Tools

**Debug Features:**
- Instance list command
- Instance details command
- Instance player list command
- Instance entity count command
- Instance state inspection
- Instance thread status

## Related Documentation

- [MAPS.md](./MAPS.md) - Base map system
- [ZONES.md](./ZONES.md) - Zone system
- [RESPAWN.md](./RESPAWN.md) - Respawn system
- [EVENT_INSTANCE.md](../events/EVENT_INSTANCE.md) - Event instance system

## See Also

- [Server README](./README.md) - Server documentation overview
- [Architecture](./ARCHITECTURE.md) - Server architecture

