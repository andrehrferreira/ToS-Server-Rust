# Zone System Documentation

## Overview

The zone system divides the game world into five distinct types of areas, each with different PvP rules, death penalties, and gameplay mechanics. This system creates diverse gameplay experiences, from safe training areas to high-risk, high-reward zones for experienced players.

## Key Features

- **Five Zone Types**: Blue (Safe), Yellow (Neutral), Red (War), Black (Full PvP), and Event zones
- **Progressive Risk**: From completely safe zones to full-loot PvP areas
- **Death Mechanics**: Different death penalties based on zone type
- **Resistance Penalties**: Zones can reduce all resistances (up to 50% reduction)
- **PK System**: Player Killer mechanics vary by zone type
- **Resource Distribution**: Better resources and loot in higher-risk zones
- **Guard System**: Guards protect safe zones and attack PKs

## Zone Types

### Blue Zone (Safe Zone)

**Color Indicator:** Blue

**PvP Rules:**
- **No PvP**: Players cannot attack other players
- **Complete Safety**: Players never die, only become unconscious
- **PK Protection**: Player Killers (PKs) are attacked by guards near cities

**Death Mechanics:**
- Players become **unconscious** when HP reaches 0
- Creatures stop attacking unconscious players
- Players remain unconscious for a period of time
- No item loss
- No experience loss
- No gold loss

**Guard System:**
- Guards patrol near cities and settlements
- Guards automatically attack PKs on sight
- Guards are high-level NPCs that can kill PKs
- PKs must avoid guard patrol routes

**Use Cases:**
- New player training areas
- Safe questing zones
- Trading and social hubs
- Low-level content areas
- Crafting and gathering areas

**Typical Locations:**
- Starting towns and villages
- Main cities and capitals
- Safe roads between settlements
- Training grounds

**Resistance Penalties:**
- **No Penalty**: Blue zones do not reduce resistances
- Players maintain full resistance values
- Safe for low-level players with low resistance

### Yellow Zone (Neutral Zone)

**Color Indicator:** Yellow

**PvP Rules:**
- **Consensual PvP Only**: Players can engage in arranged PvP duels
- **No Random PvP**: Players cannot attack others without consent
- **No Item Drops**: No items are lost on death

**Death Mechanics:**
- Players can be killed by aggressive creatures
- Creatures **cannot finish off** unconscious players
- Players have two revival options:
  - **Wait 3 minutes** to revive at death location
  - **Respawn** immediately at respawn point
- No item loss
- No gold loss
- Experience loss may apply (configurable)

**Creature Behavior:**
- Aggressive creatures begin appearing
- Creatures attack players on sight
- Creatures stop attacking when player becomes unconscious
- Creatures do not loot unconscious players

**Use Cases:**
- Mid-level content areas
- Areas with moderate risk/reward
- Training areas for PvP preparation
- Areas where players can practice combat

**Typical Locations:**
- Mid-level hunting grounds
- Neutral territories
- Areas between safe and dangerous zones
- Moderate resource gathering areas

**Resistance Penalties:**
- **Low Penalty**: 0-10% resistance reduction (zone-dependent)
- Encourages basic resistance equipment
- Still manageable for mid-level players

### Red Zone (War Zone)

**Color Indicator:** Red

**PvP Rules:**
- **Full PvP Enabled**: Players can attack anyone, anywhere in the zone
- **Karma Penalties**: Killing peaceful players causes negative karma
- **PK Consequences**: PKs attacking peaceful players may lose items
- **Recommended Level**: 70+ (due to high risk)

**Death Mechanics:**
- Players can be killed by other players or creatures
- **Gold Drop**: Players drop gold when killed
- **Inventory Items Only**: Only items in inventory (not equipped) can be lost
- **Equipped Items Protected**: Equipped items are NEVER lost
- **Blessed/Insured Items Protected**: Blessed and insured items are NEVER lost
- Item loss chance depends on:
  - PK status (PKs lose more items)
  - Karma level
  - Item protection status
- Full experience loss on death

**PK System:**
- Killing peaceful players marks you as PK
- PKs have higher item drop rates for inventory items
- **Equipped items are still protected** (never lost)
- Karma system tracks player behavior
- PKs can be attacked by guards in safe zones

**Item Protection:**
- **Equipped items**: Always protected, never lost
- **Blessed items**: Always protected, never lost
- **Insured items**: Always protected, never lost
- Quest items: Protected
- Only unequipped inventory items can be lost

**Use Cases:**
- High-level content areas
- Competitive resource gathering
- Guild warfare areas
- High-risk, high-reward zones
- End-game content areas

**Typical Locations:**
- Border territories
- Contested resource areas
- High-level dungeons
- War-torn regions

**Resistance Penalties:**
- **Moderate Penalty**: 10-30% resistance reduction (zone-dependent)
- Requires good resistance equipment
- High-level players recommended (70+)
- Makes combat more dangerous

### Black Zone (Full PvP Zone)

**Color Indicator:** Black

**PvP Rules:**
- **Full PvP**: Complete PvP enabled everywhere except safe cities
- **No Restrictions**: No karma penalties, no restrictions
- **Complete Loot**: Full inventory and gold loss on death
- **High Risk, High Reward**: Best dungeons, resources, and loot

**Death Mechanics:**
- **Complete Inventory Loss**: All unequipped items in inventory are dropped
- **Gold Loss**: All gold is dropped
- **Equipped Items Protected**: Equipped items are NEVER lost
- **Blessed/Insured Items Protected**: Blessed and insured items are NEVER lost
- Full experience loss
- Players drop all unequipped inventory items and gold (equipped items always safe)

**Safe Areas:**
- **Cities Only**: Only cities scattered throughout the territory are safe
- **Sacred Lands**: Goddess Statues provide safe respawn points (see [SACRED_LANDS.md](./SACRED_LANDS.md))
- Cities are clearly marked and protected
- Guards patrol city perimeters
- No PvP allowed inside city boundaries or Sacred Lands
- Players can rest and store items in cities
- Players respawn at Sacred Lands after death

**Item Protection:**
- **Equipped Items**: Always protected, never lost (regardless of zone)
- **Blessed Items**: Always protected, never lost
- **Insured Items**: Always protected, never lost
- **Magic Bags**: Items stored in magic bags are protected
- **God Areas**: Areas belonging to game gods are protected
- Only unequipped, non-blessed, non-insured inventory items can be lost

**Resource Distribution:**
- **Best Dungeons**: Highest-tier dungeons are in Black Zones
- **Best Resources**: Rare materials and resources
- **Best Loot**: Highest-quality equipment and items
- **Exclusive Content**: Some content only available in Black Zones

**Use Cases:**
- End-game content
- Competitive PvP
- High-stakes resource gathering
- Guild territory control
- Maximum risk/reward gameplay

**Typical Locations:**
- End-game territories
- High-level dungeons
- Rare resource nodes
- God territories (protected areas)
- Competitive PvP arenas

**Resistance Penalties:**
- **High Penalty**: 20-50% resistance reduction (zone-dependent)
- Maximum penalty can reach 50% reduction
- Requires maximum resistance equipment
- Makes combat extremely dangerous
- Forces players to invest in resistance gear

**Strategies:**
- Equip your best items (they're always safe)
- Use magic bags to protect valuable unequipped items
- Insure important unequipped inventory items
- Travel in groups for safety
- Use cities as safe havens
- Plan routes to minimize risk
- Keep valuable items equipped when possible

### Event Zone

**Color Indicator:** Varies (depends on event)

**PvP Rules:**
- **Event-Specific Rules**: Rules vary depending on the event
- **Team-Based**: Teams are defined by event teams, not guilds or parties
- **Friendly Fire**: May or may not be enabled depending on event
- **Loot Loss**: May or may not have item loss depending on event

**Death Mechanics:**
- Varies by event type
- May have full loot loss or no loot loss
- May have respawn mechanics or permanent death
- Event-specific penalties apply

**Team System:**
- Players are assigned to teams by the event
- Teams override guild and party affiliations
- Friendly fire rules apply based on team membership
- Team colors/indicators show team affiliation

**Event Types:**
- **Capture the Flag**: Team-based objective PvP
- **King of the Hill**: Territory control events
- **Battle Royale**: Last player/team standing
- **Resource Wars**: Competitive resource gathering
- **Boss Raids**: Cooperative or competitive boss fights
- **Custom Events**: Special events with unique rules

**Use Cases:**
- Special event participation
- Competitive tournaments
- Limited-time content
- Team-based competitions
- Seasonal events

**Typical Locations:**
- Event-specific arenas
- Temporary zones created for events
- Modified existing zones for events
- Special event instances

**Resistance Penalties:**
- **Event-Specific**: Varies by event type
- May have no penalty or severe penalties
- Check event rules before entering

## Zone Resistance System

### Overview

Zones can apply resistance penalties to all players within them. This system encourages players to invest in resistance equipment and creates additional challenge in dangerous areas. The penalty is applied to **all resistances** (physical and all elemental types) and is configured per zone.

### Resistance Penalty Mechanics

**How It Works:**
- Each zone has a **resistance penalty percentage** (0% to 50%)
- Penalty applies to all players in the zone
- Affects all resistance types equally:
  - Physical Resistance
  - Fire Resistance
  - Cold Resistance
  - Poison Resistance
  - Energy Resistance
  - Light Resistance
  - Dark Resistance

**Calculation:**
```
Final Resistance = Base Resistance * (1 - Zone Penalty)
```

**Example:**
```
Player has 60% Physical Resistance
Zone has 30% resistance penalty
Final Resistance = 60% * (1 - 0.30) = 60% * 0.70 = 42%
```

**Maximum Penalty:**
- Maximum zone penalty: **50%**
- Even with 50% penalty, players can still reach 35% resistance (if they had 70% max)
- Encourages maximum resistance investment for dangerous zones

### Zone Penalty Ranges

**Blue Zones:**
- **Penalty: 0%**
- No resistance reduction
- Safe for all players

**Yellow Zones:**
- **Penalty: 0-10%**
- Minimal resistance reduction
- Encourages basic resistance gear

**Red Zones:**
- **Penalty: 10-30%**
- Moderate resistance reduction
- Requires good resistance equipment
- Makes combat more dangerous

**Black Zones:**
- **Penalty: 20-50%**
- High to maximum resistance reduction
- Requires maximum resistance investment
- Can reach 50% penalty in most dangerous areas
- Forces players to use best resistance gear

**Event Zones:**
- **Penalty: Varies**
- Depends on event configuration
- Can range from 0% to 50%

### Strategic Implications

**Equipment Planning:**
- Players must invest in resistance equipment for dangerous zones
- Higher resistance values needed to compensate for penalties
- Resistance enchantments become more valuable
- Equipment upgrades become necessary for zone progression

**Zone Progression:**
- Low-level players stay in Blue/Yellow zones (low/no penalty)
- Mid-level players can handle Red zones with good gear
- High-level players can survive Black zones with maximum resistance
- Resistance penalties create natural progression gates

**Combat Impact:**
- Higher damage taken in zones with penalties
- Makes combat more dangerous and strategic
- Encourages defensive builds in dangerous zones
- Rewards players who invest in resistance

**Example Scenarios:**

**Scenario 1: Yellow Zone (10% penalty)**
```
Player Resistance: 50%
Zone Penalty: 10%
Final Resistance: 50% * 0.90 = 45%
Damage Reduction: Still manageable
```

**Scenario 2: Red Zone (25% penalty)**
```
Player Resistance: 60%
Zone Penalty: 25%
Final Resistance: 60% * 0.75 = 45%
Damage Reduction: Noticeable increase in damage taken
```

**Scenario 3: Black Zone (50% penalty)**
```
Player Resistance: 70% (max)
Zone Penalty: 50%
Final Resistance: 70% * 0.50 = 35%
Damage Reduction: Significant increase in damage taken
Requires: Maximum resistance investment to survive
```

## Zone Identification

### Visual Indicators

**Zone Borders:**
- Color-coded borders on minimap
- Zone name displayed when entering
- Visual effects at zone boundaries
- Warning messages when entering dangerous zones

**UI Indicators:**
- Zone type icon in HUD
- Current zone name display
- PvP status indicator
- Safety level indicator
- Resistance penalty indicator (shows current penalty percentage)

**Map Markers:**
- Color-coded zones on world map
- Safe area markers
- City locations
- Resource locations

## Death and Revival System

### Unconscious State (Blue Zones)

**Mechanics:**
- Player falls unconscious when HP reaches 0
- Creatures stop attacking
- Player cannot move or act
- Timer counts down to revival
- No penalties applied

**Revival:**
- Automatic revival after timer expires
- Player revives at same location
- Full HP and resources restored
- No penalties

### Death State (Yellow, Red, Black Zones)

**Mechanics:**
- Player dies when HP reaches 0
- Death animation plays
- Corpse remains at death location
- Loot drops (depending on zone)
- Experience loss applies

**Revival Options:**

**Yellow Zone:**
- Wait 3 minutes to revive at death location
- Respawn immediately at respawn point

**Red Zone:**
- Respawn at nearest respawn point
- Corpse remains for looting
- Unequipped inventory items can be recovered if not looted
- Equipped items and blessed/insured items are never lost

**Black Zone:**
- Respawn at nearest Sacred Land (Goddess Statue) or city (safe area)
- Corpse remains for looting
- Unequipped inventory items cannot be recovered (unless blessed/insured)
- Equipped items and blessed/insured items are never lost
- See [SACRED_LANDS.md](./SACRED_LANDS.md) for Sacred Land details

## PK (Player Killer) System

### PK Status

**Becoming a PK:**
- Killing peaceful players in Red/Black zones
- Attacking players without consent
- Guild warfare kills (may not count)

**PK Consequences:**
- Guards attack PKs in safe zones
- Higher item drop rates for inventory items when killed
- **Equipped items are still protected** (never lost, even for PKs)
- Negative karma accumulation
- Restricted access to some areas

### Karma System

**Karma Levels:**
- **Positive Karma**: Peaceful players, helpful actions
- **Neutral Karma**: Normal players
- **Negative Karma**: PKs, aggressive players

**Karma Effects:**
- Inventory item drop rates affected by karma (equipped items always safe)
- Guard behavior towards player
- Access to certain areas
- NPC interactions

## Guard System

### Guard Behavior

**Blue Zones:**
- Guards patrol near cities
- Attack PKs on sight
- High-level NPCs
- Cannot be killed by players
- Respawn quickly if destroyed

**Other Zones:**
- Guards only in cities (Black Zones)
- Protect city perimeters
- Attack hostile players
- Maintain safe zones

### Resistance Calculation

**Implementation:**
```rust
pub fn calculate_zone_modified_resistance(
    base_resistance: f32,
    zone: &Zone,
) -> f32 {
    // Apply zone resistance penalty
    // Penalty reduces resistance multiplicatively
    base_resistance * (1.0 - zone.resistance_penalty)
}

pub fn apply_zone_resistance_penalty(
    player: Entity,
    zone: &Zone,
    resistances: &mut Resistances,
) {
    // Apply penalty to all resistance types
    resistances.physical = calculate_zone_modified_resistance(
        resistances.physical,
        zone,
    );
    
    resistances.fire = calculate_zone_modified_resistance(
        resistances.fire,
        zone,
    );
    
    resistances.cold = calculate_zone_modified_resistance(
        resistances.cold,
        zone,
    );
    
    resistances.poison = calculate_zone_modified_resistance(
        resistances.poison,
        zone,
    );
    
    resistances.energy = calculate_zone_modified_resistance(
        resistances.energy,
        zone,
    );
    
    resistances.light = calculate_zone_modified_resistance(
        resistances.light,
        zone,
    );
    
    resistances.dark = calculate_zone_modified_resistance(
        resistances.dark,
        zone,
    );
}
```

### Guard Mechanics

**Detection:**
- Guards detect PKs automatically
- Guards have detection radius
- Guards prioritize PKs over normal players
- Guards can track PKs

**Combat:**
- Guards are high-level NPCs
- Guards deal significant damage
- Guards have high HP and defenses
- Guards can kill PKs

## Item Protection Systems

### Item Protection Rules

**Always Protected (Never Lost):**
- **Equipped Items**: All items currently equipped are always protected
- **Blessed Items**: Items marked as blessed are always protected
- **Insured Items**: Items that are insured are always protected
- **Quest Items**: Quest-related items are always protected

**Can Be Lost:**
- Unequipped inventory items (in Red/Black zones)
- Gold (in Red/Black zones)
- Items must be unequipped, non-blessed, and non-insured to be lost

### Magic Bags

**Description:**
- Special containers that protect unequipped items
- Items in magic bags are not dropped on death
- Limited capacity
- Can be obtained from quests or crafting
- Additional protection for inventory items

**Usage:**
- Store valuable unequipped items in magic bags
- Protect important resources
- Safeguard rare items
- Essential for Black Zone exploration
- Note: Equipped items don't need magic bags (already protected)

### Insurance System

**Description:**
- Unequipped items can be insured for gold
- Insured items are protected from loss (never dropped)
- Insurance costs scale with item value
- Insurance duration varies
- **Note**: Equipped items don't need insurance (already protected)

**Usage:**
- Insure important unequipped inventory items
- Protect valuable resources in inventory
- Reduce risk in dangerous zones
- Trade-off between cost and protection
- Essential for carrying valuable items in Black Zones

### Protected Areas

**Sacred Lands:**
- Goddess Statues provide safe respawn points in every map
- Complete immunity from all attacks
- Creatures lose targets and retreat
- Teleportation hubs for same-map travel
- See [SACRED_LANDS.md](./SACRED_LANDS.md) for complete details

**God Territories:**
- Areas belonging to game gods
- Items are protected in these areas
- No PvP allowed
- Safe havens in Black Zones

**Cities:**
- All cities are safe zones
- No item loss in cities
- No PvP in cities
- Storage and rest areas

## Zone Progression

### Recommended Progression

**Level 1-30: Blue Zones**
- Safe training and leveling
- Learn game mechanics
- Build character foundation

**Level 30-50: Yellow Zones**
- Moderate risk areas
- Practice combat
- Prepare for PvP

**Level 50-70: Yellow/Red Zones**
- Transition to higher risk
- Learn PvP mechanics
- Build resources

**Level 70+: Red Zones**
- High-risk, high-reward
- Competitive content
- End-game preparation

**Level 80+: Black Zones**
- Maximum risk/reward
- End-game content
- Competitive PvP
- Best resources and loot

## Rust Implementation Considerations

### Zone Data Structure

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ZoneType {
    Blue,    // Safe zone
    Yellow,  // Neutral zone
    Red,     // War zone
    Black,   // Full PvP zone
    Event(EventId), // Event zone
}

#[derive(Component)]
pub struct Zone {
    pub zone_type: ZoneType,
    pub zone_id: ZoneId,
    pub name: String,
    pub min_level: Option<u8>,
    pub max_level: Option<u8>,
    pub pvp_enabled: bool,
    pub item_drop_enabled: bool,
    pub gold_drop_enabled: bool,
    pub full_loot: bool,
    pub resistance_penalty: f32, // 0.0 to 0.50 (0% to 50%)
    pub safe_areas: Vec<SafeArea>,
}

#[derive(Debug, Clone)]
pub struct SafeArea {
    pub area_id: AreaId,
    pub bounds: BoundingBox,
    pub guard_spawns: Vec<GuardSpawn>,
}
```

### Death System

```rust
#[derive(Component)]
pub struct DeathState {
    pub death_location: Vec3,
    pub death_time: Instant,
    pub revival_timer: Option<Duration>,
    pub revival_location: Option<Vec3>,
    pub items_dropped: Vec<ItemId>,
    pub gold_dropped: u64,
}

pub fn handle_player_death(
    player: Entity,
    zone: &Zone,
    inventory: &Inventory,
    equipment: &Equipment,
) -> DeathState {
    match zone.zone_type {
        ZoneType::Blue => {
            // Unconscious state, no penalties
            handle_unconscious_state(player)
        }
        ZoneType::Yellow => {
            // Death with revival options, no item loss
            handle_yellow_zone_death(player, zone)
        }
        ZoneType::Red => {
            // Partial inventory drop (equipped items always safe)
            handle_red_zone_death(player, zone, inventory, equipment)
        }
        ZoneType::Black => {
            // Full inventory drop (equipped items always safe)
            handle_black_zone_death(player, zone, inventory, equipment)
        }
        ZoneType::Event(event_id) => {
            // Event-specific death handling
            handle_event_death(player, event_id, zone, inventory, equipment)
        }
    }
}

fn handle_red_zone_death(
    player: Entity,
    zone: &Zone,
    inventory: &Inventory,
    equipment: &Equipment,
) -> DeathState {
    // Only drop unequipped, non-blessed, non-insured items
    let items_to_drop: Vec<ItemId> = inventory
        .items
        .iter()
        .filter(|item| {
            !equipment.is_equipped(item.id) // Not equipped
                && !item.is_blessed()       // Not blessed
                && !item.is_insured()       // Not insured
        })
        .map(|item| item.id)
        .collect();
    
    // Drop gold
    let gold_dropped = inventory.gold;
    
    DeathState {
        items_dropped: items_to_drop,
        gold_dropped,
        // Equipped items never dropped
    }
}
```

### PK System

```rust
#[derive(Component)]
pub struct PlayerKiller {
    pub pk_count: u32,
    pub karma: i32,
    pub last_pk_time: Instant,
    pub pk_status: PKStatus,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum PKStatus {
    Peaceful,
    Aggressive,
    PK,
    NotoriousPK,
}

pub fn handle_pk_action(
    attacker: Entity,
    victim: Entity,
    zone: &Zone,
) {
    if zone.zone_type == ZoneType::Blue {
        // Cannot PK in blue zones
        return;
    }
    
    // Update PK status
    update_pk_status(attacker, victim, zone);
    
    // Apply karma penalties
    apply_karma_penalty(attacker, victim);
    
    // Spawn guards if in safe area
    if is_near_safe_area(attacker, zone) {
        spawn_guards(attacker, zone);
    }
}
```

### Guard System

```rust
#[derive(Component)]
pub struct Guard {
    pub guard_level: u8,
    pub detection_radius: f32,
    pub attack_range: f32,
    pub target_pks_only: bool,
}

pub fn guard_ai_system(
    guards: Query<(&Guard, &Transform)>,
    players: Query<(&PlayerKiller, &Transform)>,
) {
    for (guard, guard_pos) in guards.iter() {
        for (pk_status, player_pos) in players.iter() {
            let distance = guard_pos.translation.distance(player_pos.translation);
            
            if distance <= guard.detection_radius {
                if guard.should_attack(pk_status) {
                    guard.attack_player(player_pos);
                }
            }
        }
    }
}
```

## Integration with Other Systems

### Combat System

**Integration:**
- Zone type affects PvP availability
- Death mechanics vary by zone
- Damage calculations consider zone rules
- Zone resistance penalties affect damage taken
- Resistance values are modified by zone penalties

### Inventory System

**Integration:**
- Item drop mechanics based on zone
- Magic bag protection
- Insurance system integration

### Resistance System

**Integration:**
- **Zone penalties reduce all resistances multiplicatively** (0% to 50% reduction)
- Penalty applies to all resistance types: Physical, Fire, Cold, Poison, Energy, Light, Dark
- See [RESISTANCES.md](../core/RESISTANCES.md) for resistance calculations
- Damage type determines which resistance is used
- Zone penalties apply before damage calculation
- Formula: `Final Resistance = Base Resistance * (1 - Zone Penalty)`
- Encourages players to invest in resistance equipment
- Creates natural progression gates based on resistance values

### Economy System

**Integration:**
- Gold drop mechanics
- Item value affects drop rates
- Insurance costs
- Resistance equipment becomes more valuable in dangerous zones

### Guild System

**Integration:**
- Guild warfare rules
- Territory control in zones
- Guild vs team conflicts in events

## Testing Considerations

### Unit Tests

**Test Cases:**
- Zone type detection
- Death mechanics per zone type
- Item drop calculations
- PK status updates
- Guard spawning logic
- Resistance penalty calculations
- Zone resistance penalty application

### Integration Tests

**Test Cases:**
- Player death in each zone type
- Item loss calculations
- PK system functionality
- Guard AI behavior
- Event zone mechanics
- Resistance penalty application per zone
- Damage calculations with zone penalties

### Balance Tests

**Test Cases:**
- Risk vs reward balance
- Item drop rates
- PK penalty severity
- Guard effectiveness
- Zone progression flow
- Resistance penalty balance (0% to 50%)
- Equipment requirements per zone type

## Related Documentation

- [MAPS.md](./MAPS.md) - Map system documentation (zones are loaded per map)
- [SACRED_LANDS.md](./SACRED_LANDS.md) - Sacred Lands system (safe zones within maps)
- [TELEPORTATION.md](./TELEPORTATION.md) - Teleportation system (teleports within zones)

## Summary

The zone system:

- Divides the world into five distinct zone types with different risk levels
- Provides progressive risk/reward from safe training to full-loot PvP
- Implements death mechanics appropriate for each zone type
- **Applies resistance penalties**: Zones reduce all resistances (0% to 50% reduction)
- **Protects equipped items**: Equipped items are NEVER lost, regardless of zone
- **Protects blessed/insured items**: Blessed and insured items are NEVER lost
- Only unequipped inventory items and gold can be lost (in Red/Black zones)
- Includes PK and karma systems for player behavior tracking
- Features guard systems to protect safe areas
- Offers item protection systems for high-risk zones
- Supports event zones with custom rules
- Encourages resistance equipment investment through zone penalties
- **Map Integration**: Each map typically has one primary zone (see [MAPS.md](./MAPS.md))

**Key Protection Rules:**
- **Equipped Items**: Always protected, never lost
- **Blessed Items**: Always protected, never lost
- **Insured Items**: Always protected, never lost
- **Only Inventory Loss**: Only unequipped, non-blessed, non-insured inventory items can be lost
- **Gold Loss**: Gold can be lost in Red/Black zones

This system creates diverse gameplay experiences, from safe learning environments to high-stakes competitive PvP, ensuring players can find content appropriate for their skill level and risk tolerance while providing clear progression paths and meaningful choices. Players can always keep their equipped gear safe, making death penalties meaningful but not devastating.

