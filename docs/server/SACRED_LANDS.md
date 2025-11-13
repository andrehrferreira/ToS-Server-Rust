# Sacred Lands System Documentation

## Overview

Sacred Lands are special protected areas found in every map, centered around a Goddess Statue. These areas serve as safe respawn points, teleportation hubs, and completely safe zones where no combat can occur. The system ensures players always have a safe location to return to after death and provides convenient travel options within maps.

## Key Features

- **Universal Presence**: Every map has at least one Sacred Land area
- **Goddess Statue**: Central statue that serves as respawn point and teleportation hub
- **Complete Safety**: No attacks possible within Sacred Land boundaries
- **Respawn Point**: Players respawn at the Goddess Statue after death
- **Teleportation**: Statue allows teleportation to other areas on the same map
- **Creature Behavior**: Creatures lose targets and retreat when entering Sacred Lands
- **Immunity Zone**: All entities are immune to damage within Sacred Lands

## Sacred Land Structure

### Goddess Statue

**Description:**
- Central monument in every Sacred Land
- Represents the goddess protecting that area
- Serves as respawn point for dead players
- Provides teleportation services to other map areas
- Visually distinct and easily recognizable

**Functions:**
1. **Respawn Point**: Players respawn here after death
2. **Teleportation Hub**: Allows travel to other Sacred Lands on the same map
3. **Safe Zone Marker**: Clearly marks the protected area
4. **Visual Landmark**: Helps players orient themselves

### Protected Area

**Size:**
- Area extends around the Goddess Statue
- Radius varies by map size and importance
- Clearly marked boundaries (visual indicators)
- Large enough to accommodate multiple players

**Boundaries:**
- Visual markers indicate Sacred Land borders
- Map indicators show protected area
- Clear transition between safe and dangerous zones
- Players receive notification when entering/leaving

## Safety Mechanics

### Complete Immunity

**Combat Restrictions:**
- **No Player Attacks**: Players cannot attack other players
- **No Creature Attacks**: Creatures cannot attack players
- **No Damage**: All entities are immune to all damage types
- **No Skills**: Combat skills cannot be used
- **No PvP**: PvP is completely disabled

**Implementation:**
```rust
pub fn is_in_sacred_land(position: Vec3, sacred_lands: &[SacredLand]) -> bool {
    sacred_lands.iter().any(|land| {
        let distance = position.distance(land.statue_position);
        distance <= land.protection_radius
    })
}

pub fn can_attack(attacker: Entity, target: Entity, sacred_lands: &[SacredLand]) -> bool {
    let attacker_pos = get_position(attacker);
    let target_pos = get_position(target);
    
    // Cannot attack if either entity is in Sacred Land
    !is_in_sacred_land(attacker_pos, sacred_lands) 
        && !is_in_sacred_land(target_pos, sacred_lands)
}
```

### Creature Behavior

**Target Loss:**
- Creatures immediately lose their current target when entering Sacred Land
- Aggro is cleared when creature enters protected area
- Creatures stop pursuing players who enter Sacred Land

**Retreat Behavior:**
- Creatures retreat from Sacred Land boundaries
- Creatures move away from protected area
- Creatures return to their spawn area or patrol route
- No creatures spawn inside Sacred Lands

**Implementation:**
```rust
pub fn handle_creature_sacred_land_entrance(
    creature: Entity,
    sacred_land: &SacredLand,
) {
    // Clear all aggro
    clear_aggro(creature);
    
    // Stop current action
    stop_combat(creature);
    
    // Retreat from Sacred Land
    retreat_from_position(creature, sacred_land.statue_position);
    
    // Return to spawn/patrol
    return_to_spawn_area(creature);
}
```

### Player Immunity

**Protection:**
- Players are immune to all damage sources
- Status effects may still apply (non-damaging)
- Buffs and debuffs can still be active
- Players can still interact with NPCs and objects

**Status Effects:**
- Damage-over-time effects are paused
- Healing effects continue to work
- Buffs remain active
- Debuffs remain active (but no damage)

## Respawn System

### Respawn Mechanics

**Death Handling:**
- When a player dies, they respawn at the nearest Goddess Statue
- Respawn is instant (no delay)
- Player appears at the statue location
- Full HP, Mana, and Stamina restored

**Respawn Priority:**
1. Nearest Sacred Land on current map
2. If no Sacred Land on current map, respawn at map entrance Sacred Land
3. Always respawn at a Sacred Land (never in dangerous areas)

**Implementation:**
```rust
pub fn respawn_player(player: Entity, maps: &MapSystem) -> Vec3 {
    let current_map = get_player_map(player);
    let sacred_lands = maps.get_sacred_lands(current_map);
    
    // Find nearest Sacred Land
    let player_pos = get_position(player);
    let nearest = sacred_lands
        .iter()
        .min_by_key(|land| {
            (player_pos.distance(land.statue_position) * 1000.0) as u32
        })
        .unwrap();
    
    // Respawn at statue
    nearest.statue_position
}

pub fn handle_player_respawn(player: Entity) {
    let respawn_pos = respawn_player(player, &maps);
    
    // Restore player
    set_position(player, respawn_pos);
    restore_resources(player); // Full HP, Mana, Stamina
    clear_death_state(player);
    
    // Notify player
    notify_respawn(player, respawn_pos);
}
```

### Respawn Points

**Goddess Statue Location:**
- Always at the center of Sacred Land
- Clearly visible and accessible
- Large enough area for multiple respawning players
- No obstacles blocking respawn location

**Multiple Respawns:**
- Multiple players can respawn simultaneously
- No collision issues
- Players appear slightly offset from exact statue position
- Smooth respawn experience

## Teleportation System

### Teleportation Mechanics

**Note:** Sacred Land teleportation is part of a larger teleportation system. See [TELEPORTATION.md](./TELEPORTATION.md) for complete teleportation documentation including City Portals, Mage Portals, and Ancient Monoliths.

**How It Works:**
- Players interact with Goddess Statue
- Statue shows list of available destinations
- Destinations are other Sacred Lands on the same map
- Teleportation is instant (no loading screen for same map)
- May have cooldown or cost (configurable)
- **Limitation**: Only works within same map (cannot cross map boundaries)

**Available Destinations:**
- All other Sacred Lands on the current map
- Cannot teleport to different maps (use other systems)
- Destinations shown on map/minimap
- Distance and zone type displayed

**Implementation:**
```rust
pub struct TeleportDestination {
    pub destination_id: SacredLandId,
    pub name: String,
    pub position: Vec3,
    pub zone_type: ZoneType,
    pub distance: f32,
    pub cost: Option<u64>, // Gold cost, if any
}

pub fn get_teleport_destinations(
    current_map: MapId,
    current_sacred_land: SacredLandId,
) -> Vec<TeleportDestination> {
    let sacred_lands = get_sacred_lands(current_map);
    let current_pos = get_sacred_land_position(current_sacred_land);
    
    sacred_lands
        .iter()
        .filter(|land| land.id != current_sacred_land)
        .map(|land| {
            let distance = current_pos.distance(land.statue_position);
            TeleportDestination {
                destination_id: land.id,
                name: land.name.clone(),
                position: land.statue_position,
                zone_type: land.zone_type,
                distance,
                cost: calculate_teleport_cost(distance),
            }
        })
        .collect()
}

pub fn teleport_player(
    player: Entity,
    destination: SacredLandId,
) -> Result<(), TeleportError> {
    // Validate player is at Sacred Land
    if !is_at_sacred_land(player) {
        return Err(TeleportError::NotAtSacredLand);
    }
    
    // Check cooldown
    if has_teleport_cooldown(player) {
        return Err(TeleportError::OnCooldown);
    }
    
    // Check cost
    if let Some(cost) = get_teleport_cost(destination) {
        if !has_enough_gold(player, cost) {
            return Err(TeleportError::InsufficientGold);
        }
        deduct_gold(player, cost);
    }
    
    // Teleport
    let destination_pos = get_sacred_land_position(destination);
    set_position(player, destination_pos);
    
    // Apply cooldown
    apply_teleport_cooldown(player);
    
    // Notify player
    notify_teleport(player, destination_pos);
    
    Ok(())
}
```

### Teleportation Costs

**Cost System:**
- May be free (for convenience)
- May cost gold (based on distance)
- May have cooldown (to prevent abuse)
- Costs scale with distance traveled

**Cost Examples:**
- Short distance: Free or minimal cost
- Medium distance: Moderate gold cost
- Long distance: Higher gold cost
- Same Sacred Land: Free (no teleport needed)

### Teleportation UI

**Interface:**
- Interact with Goddess Statue opens teleport menu
- List of available destinations
- Shows destination name, zone type, distance
- Shows cost (if any)
- Confirmation before teleporting

**Visual Indicators:**
- Sacred Lands marked on map
- Teleportation paths shown
- Distance indicators
- Zone type indicators

## Map Distribution

### Sacred Land Placement Requirements

**Mandatory Requirements:**
- **Every map MUST have at least one Sacred Land**
- This is a mandatory requirement for all maps
- Ensures players always have a safe respawn point
- Provides teleportation hub for same-map travel

**Placement Strategy:**
- Usually near map entrance/exit (for new arrivals)
- Central locations (for map exploration)
- Near important content (dungeons, resources)
- Balanced distribution across map
- Strategically placed for accessibility

**Large Maps:**
- May have multiple Sacred Lands for convenience
- Reduces long travel times
- Provides multiple respawn options
- Better player experience
- Still requires minimum of 1 Sacred Land

### Zone Type Distribution

**Sacred Lands in Different Zones:**
- **Blue Zones**: Multiple Sacred Lands, all safe
- **Yellow Zones**: Sacred Lands provide safe respawn
- **Red Zones**: Sacred Lands are crucial safe havens
- **Black Zones**: Sacred Lands are only safe areas (except cities)
- **Event Zones**: Sacred Lands follow event rules

## Visual Design

### Goddess Statue Appearance

**Visual Elements:**
- Large, impressive statue
- Glowing effects (indicates protection)
- Distinctive design per map/region
- Easily visible from distance
- Clear indication of interactivity

**Effects:**
- Subtle glow or aura
- Particle effects
- Light source
- Makes area feel sacred/protected

### Area Markers

**Boundary Indicators:**
- Visual markers at Sacred Land borders
- Subtle but clear boundaries
- Smooth transition effects
- Map indicators show protected area

**UI Indicators:**
- Minimap markers for Sacred Lands
- Zone name display when entering
- Protection status indicator
- Distance to nearest Sacred Land

## Integration with Zone System

### Relationship to Zones

**Sacred Lands vs Zone Types:**
- Sacred Lands exist in all zone types
- Sacred Land protection overrides zone PvP rules
- Sacred Lands provide respawn regardless of zone
- Teleportation works within same map regardless of zones

**Zone-Specific Behavior:**
- **Blue Zones**: Sacred Lands blend with safe environment
- **Yellow Zones**: Sacred Lands provide additional safety
- **Red Zones**: Sacred Lands are important safe havens
- **Black Zones**: Sacred Lands are critical safe points (alongside cities)

### Death and Respawn Integration

**Zone Death Mechanics:**
- Death mechanics vary by zone (see [ZONES.md](./ZONES.md))
- Respawn always at Sacred Land (safe)
- Item loss still applies based on zone rules
- Gold loss still applies based on zone rules
- Experience loss still applies based on zone rules

**Example Flow:**
```
Player dies in Black Zone
→ Item loss occurs (unequipped items)
→ Gold loss occurs
→ Experience loss occurs
→ Respawn at nearest Sacred Land (safe)
→ Player can recover and plan next move
```

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Component, Clone)]
pub struct SacredLand {
    pub id: SacredLandId,
    pub map_id: MapId,
    pub name: String,
    pub statue_position: Vec3,
    pub protection_radius: f32,
    pub zone_type: ZoneType,
    pub teleport_destinations: Vec<TeleportDestination>,
}

#[derive(Component)]
pub struct SacredLandProtection {
    pub is_protected: bool,
    pub sacred_land_id: Option<SacredLandId>,
}

#[derive(Component)]
pub struct TeleportCooldown {
    pub last_teleport_time: Option<Instant>,
    pub cooldown_duration: Duration,
}
```

### Systems

**Protection System:**
```rust
pub fn sacred_land_protection_system(
    mut players: Query<(Entity, &Transform, &mut SacredLandProtection)>,
    sacred_lands: Query<&SacredLand>,
) {
    for (entity, transform, mut protection) in players.iter_mut() {
        let position = transform.translation;
        
        // Check if in any Sacred Land
        let in_sacred_land = sacred_lands.iter().any(|land| {
            let distance = position.distance(land.statue_position);
            distance <= land.protection_radius
        });
        
        protection.is_protected = in_sacred_land;
    }
}
```

**Combat Prevention System:**
```rust
pub fn prevent_sacred_land_combat(
    mut attack_events: EventReader<AttackEvent>,
    sacred_land_protection: Query<&SacredLandProtection>,
) {
    for event in attack_events.read() {
        let attacker_protection = sacred_land_protection.get(event.attacker);
        let target_protection = sacred_land_protection.get(event.target);
        
        // Cancel attack if either entity is protected
        if attacker_protection.is_ok_and(|p| p.is_protected) 
            || target_protection.is_ok_and(|p| p.is_protected) {
            event.cancel();
        }
    }
}
```

**Creature Behavior System:**
```rust
pub fn sacred_land_creature_behavior(
    mut creatures: Query<(Entity, &Transform, &mut CreatureAI)>,
    sacred_lands: Query<&SacredLand>,
) {
    for (entity, transform, mut ai) in creatures.iter_mut() {
        let position = transform.translation;
        
        // Check if creature entered Sacred Land
        let in_sacred_land = sacred_lands.iter().any(|land| {
            let distance = position.distance(land.statue_position);
            distance <= land.protection_radius
        });
        
        if in_sacred_land {
            // Clear aggro and retreat
            ai.clear_aggro();
            ai.set_state(CreatureState::Retreating);
            ai.set_target_position(find_retreat_position(entity, sacred_lands));
        }
    }
}
```

**Respawn System:**
```rust
pub fn handle_player_death_respawn(
    mut dead_players: Query<(Entity, &Transform, &mut PlayerState)>,
    maps: Res<MapSystem>,
) {
    for (entity, transform, mut state) in dead_players.iter_mut() {
        if state.is_dead {
            let current_map = get_map_for_position(transform.translation);
            let respawn_pos = find_nearest_sacred_land(
                transform.translation,
                current_map,
                &maps,
            );
            
            // Respawn player
            transform.translation = respawn_pos;
            state.is_dead = false;
            restore_resources(entity);
            
            // Notify
            notify_respawn(entity, respawn_pos);
        }
    }
}
```

**Teleportation System:**
```rust
pub fn handle_teleportation_request(
    player: Entity,
    destination: SacredLandId,
    sacred_lands: Res<Assets<SacredLand>>,
    mut players: Query<&mut Transform>,
) -> Result<(), TeleportError> {
    // Validate player is at Sacred Land
    let player_pos = players.get(player)?.translation;
    if !is_at_sacred_land(player_pos, &sacred_lands) {
        return Err(TeleportError::NotAtSacredLand);
    }
    
    // Get destination
    let destination_land = sacred_lands.get(destination)?;
    
    // Check cooldown
    // Check cost
    // Apply cost
    
    // Teleport
    players.get_mut(player)?.translation = destination_land.statue_position;
    
    // Apply cooldown
    // Notify player
    
    Ok(())
}
```

## Performance Optimizations

### Spatial Partitioning

**Optimization:**
- Use spatial hash or quadtree for Sacred Land queries
- Cache Sacred Land positions
- Pre-calculate protection radii
- Efficient distance checks

### Caching

**Cache:**
- Sacred Land positions per map
- Protection status per entity
- Teleportation destinations
- Nearest Sacred Land calculations

## Integration with Other Systems

### Zone System

**Integration:**
- Sacred Lands exist in all zone types
- Protection overrides zone PvP rules
- Respawn mechanics work with zone death penalties
- See [ZONES.md](./ZONES.md) for zone details

### Death System

**Integration:**
- Death penalties apply (items, gold, experience)
- Respawn always at Sacred Land
- Safe respawn regardless of death location
- Recovery point after death

### Map System

**Integration:**
- Sacred Lands are map features
- Placement during map generation
- Multiple Sacred Lands per large map
- Map transitions preserve Sacred Land access

### Teleportation System

**Integration:**
- Goddess Statue provides teleportation
- Works within same map
- May integrate with other teleportation systems
- Provides convenient travel option

## Testing Considerations

### Unit Tests

**Test Cases:**
- Sacred Land detection (position in radius)
- Protection status calculation
- Combat prevention in Sacred Lands
- Creature behavior when entering Sacred Land
- Respawn point selection
- Teleportation validation

### Integration Tests

**Test Cases:**
- Player death and respawn at Sacred Land
- Combat prevention in protected area
- Creature retreat behavior
- Teleportation between Sacred Lands
- Multiple players in Sacred Land
- Zone transitions with Sacred Lands

### Balance Tests

**Test Cases:**
- Sacred Land distribution across maps
- Teleportation cost balance
- Respawn point accessibility
- Protection radius size
- Multiple Sacred Lands on large maps

## Map Requirements

### Mandatory Sacred Land Placement

**Every Map Must Have:**
- **At least 1 Sacred Land** (mandatory requirement)
- Goddess Statue at the center of each Sacred Land
- Protected area around the statue
- Safe respawn point for all players

**Additional Requirements:**
- Every map must also have **1-2 teleportation points** (City Portals or Ancient Monoliths)
- See [TELEPORTATION.md](./TELEPORTATION.md) for teleportation point requirements
- Ensures complete player mobility across all maps

**Placement Guidelines:**
- Sacred Land usually near map entrance/exit
- Additional Sacred Lands recommended for large maps
- Strategic placement for accessibility
- Balanced distribution for player convenience

## Related Documentation

- [MAPS.md](./MAPS.md) - Map system documentation (Sacred Lands are loaded per map)
- [ZONES.md](./ZONES.md) - Zone system documentation (Sacred Lands work across all zone types)
- [TELEPORTATION.md](./TELEPORTATION.md) - Teleportation system (Goddess Statue teleportation)

## Summary

The Sacred Lands system:

- Provides safe respawn points in every map via Goddess Statues
- **Mandatory Requirement**: Every map must have at least 1 Sacred Land
- Creates completely safe zones where no combat can occur
- Allows creatures to lose targets and retreat when entering protected areas
- Provides teleportation services between Sacred Lands on the same map
- Ensures players always have a safe location to return to after death
- Reduces long travel times through teleportation
- Integrates seamlessly with the zone system
- Works across all zone types (Blue, Yellow, Red, Black, Event)
- **Map Infrastructure**: Works together with teleportation points (1-2 per map) to ensure player mobility
- **Map Loading**: Sacred Lands are loaded from map metadata files (see [MAPS.md](./MAPS.md))

**Key Benefits:**
- **Safety**: Complete protection from all attacks
- **Convenience**: Easy respawn and teleportation
- **Accessibility**: Every map has at least one Sacred Land
- **Gameplay**: Reduces frustration from long travel times
- **Balance**: Works with zone death penalties while providing safe recovery

This system ensures players always have a safe haven to return to, making death less frustrating while maintaining the risk/reward balance of different zone types. The teleportation system reduces tedious travel while encouraging exploration of different areas within maps.

