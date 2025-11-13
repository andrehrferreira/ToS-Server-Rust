# Treasure Hunt Event Documentation

## Overview

Treasure Hunt is a dynamic event system where players can discover treasure maps through fishing and embark on treasure hunting adventures. The system features progressive difficulty levels, with higher-level maps leading to more dangerous locations with stronger guardians and bosses. Successful treasure hunts reward players with valuable loot including legendary equipment, crafting materials, gemstones, skins, and other rare resources.

## Key Features

- **Map Discovery**: Treasure maps found through fishing (bottles with maps)
- **10 Difficulty Levels**: Maps range from level 1 to 10
- **Progressive Difficulty**: Higher levels = more dangerous areas and stronger enemies
- **Guardian System**: Creatures spawn to protect treasure locations
- **Boss Encounters**: High-level maps (levels 7-10) feature boss guardians
- **Legendary Rewards**: Complete legendary equipment sets can drop from treasure chests
- **Rich Loot**: Various resources, gemstones, crafting materials, and skins
- **Location-Based**: Maps lead to specific locations in the world
- **Shovel Requirement**: Players must use a shovel to dig up the treasure

## Map Discovery

### Obtaining Treasure Maps

**Fishing Method:**
- Players can catch bottles while fishing
- Bottles contain treasure maps of varying levels
- Higher Fishing skill increases chance of finding bottles
- Bottles are rare drops from fishing activities

**Map Types:**
- Maps are contained within bottles found while fishing
- Each bottle contains one treasure map
- Map level is random (1-10)
- Map level determines difficulty and reward quality

### Map Properties

**Map Levels:**
- **Level 1-3**: Beginner maps (safer areas, weaker guardians)
- **Level 4-6**: Intermediate maps (moderate danger, stronger guardians)
- **Level 7-10**: Advanced maps (dangerous areas, boss guardians, best rewards)

**Map Information:**
- Map shows treasure location coordinates
- Map indicates difficulty level
- Map displays recommended player level/gear
- Map cannot be used without possessing it

## Treasure Hunt Process

### Step 1: Map Acquisition

1. Player goes fishing
2. Player catches a bottle (rare drop)
3. Player opens bottle to obtain treasure map
4. Map is added to player inventory
5. Map shows location and level information

### Step 2: Travel to Location

1. Player reads map to identify treasure location
2. Player travels to the marked location on the world map
3. Location is marked on player's map/minimap
4. Player must reach exact location to start event

### Step 3: Digging Up Treasure

1. Player equips shovel (required tool)
2. Player uses shovel at treasure location
3. Digging action initiates the treasure hunt event
4. Event cannot start without shovel
5. Digging reveals the treasure chest location

### Step 4: Guardian Encounter

**Level 1-6 Maps:**
- Regular guardian creatures spawn
- Number of guardians increases with map level
- Guardians are hostile and attack player
- Player must defeat all guardians to access chest

**Level 7-10 Maps:**
- Regular guardian creatures spawn
- Boss guardian spawns after regular guardians
- Boss is significantly stronger than regular guardians
- Player must defeat boss to access chest
- Boss has unique abilities and higher HP

**Guardian Spawning:**
- Guardians spawn immediately after digging
- Guardians spawn around treasure location
- Guardians are aggressive and attack on sight
- Guardians despawn if player leaves area (event fails)

### Step 5: Opening Treasure Chest

1. Player defeats all guardians (and boss if applicable)
2. Treasure chest becomes accessible
3. Player interacts with chest to open it
4. Loot is distributed to player inventory
5. Event completes successfully

## Difficulty Scaling

### Level 1-3 (Beginner)

**Guardians:**
- 1-3 guardian creatures
- Weak to moderate strength
- Basic attack patterns
- Low HP and damage

**Rewards:**
- Common to Uncommon equipment
- Basic crafting materials
- Common gemstones
- Small amounts of resources

### Level 4-6 (Intermediate)

**Guardians:**
- 3-5 guardian creatures
- Moderate to strong strength
- Varied attack patterns
- Moderate HP and damage

**Rewards:**
- Uncommon to Rare equipment
- Quality crafting materials
- Uncommon to Rare gemstones
- Moderate amounts of resources
- Chance for skins

### Level 7-10 (Advanced)

**Guardians:**
- 5-8 guardian creatures
- Strong to very strong strength
- Complex attack patterns
- High HP and damage
- **Boss Guardian** (levels 7-10)

**Boss Characteristics:**
- Significantly higher HP than regular guardians
- Unique abilities and attack patterns
- Area-of-effect attacks
- Enrage mechanics at low HP
- Requires coordination and strategy

**Rewards:**
- Rare to Magic equipment
- High-quality crafting materials
- Rare to Magic gemstones
- Large amounts of resources
- Higher chance for skins
- **Complete Legendary Equipment Sets** (rare drop)
- Unique items and rare resources

## Reward System

### Reward Types

**Equipment:**
- Common, Uncommon, Rare, Magic, and Legendary items
- Complete legendary equipment sets (very rare)
- Equipment appropriate to map level
- Random attributes and enchantments

**Crafting Materials:**
- Ores and ingots (various tiers)
- Leather and hides (various tiers)
- Wood and boards (various tiers)
- Herbs and alchemical materials
- Enchantment materials (essences, dust)

**Gemstones:**
- Common to Magic gemstones
- Various gemstone types
- Quality scales with map level

**Resources:**
- Gold (varying amounts)
- Consumables (potions, food)
- Quest items (rare)

**Cosmetics:**
- Visual skins for equipment
- Cosmetic items
- Dyes and appearance modifiers

### Reward Scaling

**Level 1-3 Rewards:**
- 1-3 items per chest
- Common/Uncommon quality focus
- Basic materials
- Low gold amounts

**Level 4-6 Rewards:**
- 3-5 items per chest
- Uncommon/Rare quality focus
- Quality materials
- Moderate gold amounts

**Level 7-10 Rewards:**
- 5-8 items per chest
- Rare/Magic/Legendary quality focus
- High-quality materials
- Large gold amounts
- Chance for complete legendary sets

## Event Mechanics

### Map Usage

**Single Use:**
- Each map can only be used once
- Map is consumed when treasure hunt starts
- Cannot reuse same map location
- Multiple maps can lead to same location (different instances)

**Map Trading:**
- Maps can be traded between players
- Maps retain their level and location
- Maps are valuable trade items
- Economy around map trading

### Location System

**Unique Locations:**
- Each map leads to a specific world location
- Locations are marked on world map
- Locations may be in dangerous areas
- Some locations require specific access (dungeons, islands)

**Location Types:**
- Open world locations
- Dungeon entrances
- Hidden caves
- Remote islands
- Underwater locations (requires special access)

### Shovel Requirement

**Tool Requirement:**
- Shovel is required to dig up treasure
- Shovel must be equipped in tool slot
- Shovel durability decreases with use
- Shovel can break (needs repair or replacement)

**Shovel Types:**
- Basic Shovel: Standard digging
- Master Shovel: Faster digging, better condition
- Legendary Shovel: Fastest digging, best condition

### Event Failure Conditions

**Failure Scenarios:**
- Player dies during guardian encounter
- Player leaves treasure area before completing event
- Player runs out of time (if time limit exists)
- Shovel breaks during digging (rare)

**Failure Consequences:**
- Map is consumed (lost)
- Guardians despawn
- Treasure chest disappears
- No rewards obtained
- Player must obtain new map to retry

## Integration with Other Systems

### Fishing System Integration

**Bottle Drop Rates:**
- Rare drop from fishing activities
- Higher Fishing skill increases drop chance
- Bottles are stackable items
- Bottles can be stored in inventory

**Fishing Skill Benefits:**
- Higher skill = better chance for bottles
- Higher skill = chance for higher-level maps
- Fishing skill progression rewards treasure hunters

### Combat System Integration

**Guardian Combat:**
- Guardians use standard combat mechanics
- Guardians have appropriate level and stats
- Boss guardians have unique mechanics
- Player combat skills affect success rate

**Combat Rewards:**
- Guardians drop standard loot (if applicable)
- Guardian XP contributes to player progression
- Combat skills gain XP from guardian fights

### Equipment System Integration

**Legendary Sets:**
- Complete legendary equipment sets can drop
- Sets are extremely rare rewards
- Sets are appropriate to player level
- Sets have powerful set bonuses

**Equipment Scaling:**
- Equipment rewards scale with map level
- Higher-level maps drop better equipment
- Equipment has appropriate tier and rarity

### Economy Integration

**Map Trading:**
- Maps are valuable trade items
- Higher-level maps command higher prices
- Maps create economic activity
- Map trading between players

**Reward Economy:**
- Treasure hunt rewards enter economy
- Legendary equipment affects market
- Crafting materials from treasures
- Gemstones and resources

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone)]
pub struct TreasureMap {
    pub map_id: u32,
    pub level: u8,              // 1-10
    pub location: WorldLocation,
    pub discovered_by: Option<PlayerId>,
    pub discovered_at: Option<DateTime>,
}

#[derive(Debug, Clone, Copy)]
pub struct WorldLocation {
    pub x: f32,
    pub y: f32,
    pub z: f32,
    pub map_zone: ZoneId,
}

#[derive(Debug, Clone)]
pub struct TreasureHuntEvent {
    pub event_id: u64,
    pub map: TreasureMap,
    pub player_id: PlayerId,
    pub location: WorldLocation,
    pub state: TreasureHuntState,
    pub guardians: Vec<GuardianEntity>,
    pub boss: Option<BossEntity>,
    pub chest: Option<TreasureChest>,
    pub started_at: DateTime,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum TreasureHuntState {
    NotStarted,
    Digging,
    GuardiansSpawning,
    GuardiansActive,
    BossSpawning,
    BossActive,
    Completed,
    Failed,
}

#[derive(Debug, Clone)]
pub struct TreasureChest {
    pub chest_id: u64,
    pub rewards: Vec<TreasureReward>,
    pub opened: bool,
    pub opened_by: Option<PlayerId>,
}

#[derive(Debug, Clone)]
pub enum TreasureReward {
    Equipment(ItemId, ItemRarity),
    CraftingMaterial(ResourceId, u16),
    Gemstone(GemstoneId, GemstoneQuality),
    Gold(u32),
    Skin(SkinId),
    Consumable(ItemId, u16),
}
```

### Event Flow

```rust
pub fn start_treasure_hunt(
    player: &mut Player,
    map: TreasureMap,
    location: WorldLocation,
) -> Result<TreasureHuntEvent, String> {
    // Check if player has shovel
    if !player.has_shovel() {
        return Err("Shovel required to dig treasure".to_string());
    }
    
    // Verify location matches map
    if !map.location.matches(location) {
        return Err("Location does not match map".to_string());
    }
    
    // Consume map
    player.consume_map(map.map_id)?;
    
    // Create event
    let mut event = TreasureHuntEvent {
        event_id: generate_event_id(),
        map: map.clone(),
        player_id: player.id,
        location,
        state: TreasureHuntState::NotStarted,
        guardians: Vec::new(),
        boss: None,
        chest: None,
        started_at: current_time(),
    };
    
    Ok(event)
}

pub fn dig_treasure(
    event: &mut TreasureHuntEvent,
    player: &mut Player,
) -> Result<(), String> {
    if event.state != TreasureHuntState::NotStarted {
        return Err("Event already started".to_string());
    }
    
    // Use shovel
    player.use_shovel()?;
    
    // Spawn guardians
    event.state = TreasureHuntState::Digging;
    spawn_guardians(event)?;
    event.state = TreasureHuntState::GuardiansActive;
    
    Ok(())
}

pub fn spawn_guardians(event: &mut TreasureHuntEvent) -> Result<(), String> {
    let guardian_count = calculate_guardian_count(event.map.level);
    let guardian_level = calculate_guardian_level(event.map.level);
    
    for _ in 0..guardian_count {
        let guardian = spawn_guardian_creature(
            event.location,
            guardian_level,
        )?;
        event.guardians.push(guardian);
    }
    
    // Spawn boss for high-level maps
    if event.map.level >= 7 {
        let boss = spawn_boss_guardian(
            event.location,
            calculate_boss_level(event.map.level),
        )?;
        event.boss = Some(boss);
    }
    
    // Spawn treasure chest
    let chest = spawn_treasure_chest(event.location, event.map.level)?;
    event.chest = Some(chest);
    
    Ok(())
}

pub fn check_guardians_defeated(event: &mut TreasureHuntEvent) -> bool {
    let all_guardians_dead = event.guardians.iter().all(|g| g.is_dead());
    let boss_dead = event.boss.as_ref().map(|b| b.is_dead()).unwrap_or(true);
    
    all_guardians_dead && boss_dead
}

pub fn open_treasure_chest(
    event: &mut TreasureHuntEvent,
    player: &mut Player,
) -> Result<Vec<TreasureReward>, String> {
    if event.state != TreasureHuntState::GuardiansActive {
        return Err("Cannot open chest yet".to_string());
    }
    
    if !check_guardians_defeated(event) {
        return Err("All guardians must be defeated".to_string());
    }
    
    let chest = event.chest.as_mut().ok_or("Chest not found")?;
    if chest.opened {
        return Err("Chest already opened".to_string());
    }
    
    // Generate rewards based on map level
    let rewards = generate_treasure_rewards(event.map.level);
    
    // Distribute rewards to player
    for reward in &rewards {
        player.add_reward(reward.clone())?;
    }
    
    chest.opened = true;
    chest.opened_by = Some(player.id);
    event.state = TreasureHuntState::Completed;
    
    Ok(rewards)
}
```

### Reward Generation

```rust
pub fn generate_treasure_rewards(map_level: u8) -> Vec<TreasureReward> {
    let mut rewards = Vec::new();
    let reward_count = calculate_reward_count(map_level);
    
    for _ in 0..reward_count {
        let reward_type = roll_reward_type(map_level);
        let reward = match reward_type {
            RewardType::Equipment => generate_equipment_reward(map_level),
            RewardType::CraftingMaterial => generate_material_reward(map_level),
            RewardType::Gemstone => generate_gemstone_reward(map_level),
            RewardType::Gold => generate_gold_reward(map_level),
            RewardType::Skin => generate_skin_reward(map_level),
            RewardType::Consumable => generate_consumable_reward(map_level),
        };
        rewards.push(reward);
    }
    
    // Chance for legendary set (very rare)
    if should_drop_legendary_set(map_level) {
        let legendary_set = generate_legendary_set(map_level);
        rewards.extend(legendary_set);
    }
    
    rewards
}

fn calculate_reward_count(map_level: u8) -> u8 {
    match map_level {
        1..=3 => 1..=3,
        4..=6 => 3..=5,
        7..=10 => 5..=8,
        _ => 1,
    }
}

fn should_drop_legendary_set(map_level: u8) -> bool {
    if map_level < 7 {
        return false;
    }
    
    // Very rare chance, increases with level
    let base_chance = 0.001; // 0.1% base
    let level_bonus = (map_level as f32 - 7.0) * 0.0005; // +0.05% per level above 7
    let chance = base_chance + level_bonus;
    
    thread_rng().gen::<f32>() < chance
}
```

### Fishing Integration

```rust
pub fn on_fishing_success(
    player: &mut Player,
    fishing_result: FishingResult,
) -> Vec<Item> {
    let mut items = vec![fishing_result.fish];
    
    // Check for bottle drop
    if should_drop_bottle(player.fishing_skill) {
        let map_level = roll_treasure_map_level(player.fishing_skill);
        let bottle = create_treasure_map_bottle(map_level)?;
        items.push(bottle);
    }
    
    items
}

fn should_drop_bottle(fishing_skill: u16) -> bool {
    // Base chance increases with skill
    let base_chance = 0.01; // 1% base
    let skill_bonus = (fishing_skill as f32 / 100.0) * 0.02; // +2% per 100 skill
    let chance = base_chance + skill_bonus;
    
    thread_rng().gen::<f32>() < chance.min(0.1) // Cap at 10%
}

fn roll_treasure_map_level(fishing_skill: u16) -> u8 {
    // Higher skill = better chance for higher level maps
    let skill_factor = (fishing_skill as f32 / 100.0).min(1.0);
    let base_level = 1.0 + (skill_factor * 9.0); // 1-10 range
    
    thread_rng().gen_range(1..=10) // Simplified, should use skill_factor
}
```

## Summary

Treasure Hunt is an engaging event system that:

- **Integrates with Fishing**: Maps discovered through fishing bottles
- **Progressive Difficulty**: 10 levels with increasing challenge
- **Dynamic Encounters**: Guardian creatures and boss fights
- **Rich Rewards**: Equipment, materials, gemstones, and legendary sets
- **Location-Based**: Maps lead to specific world locations
- **Tool Requirement**: Shovel required to dig up treasure
- **Risk vs Reward**: Higher levels offer better rewards but greater danger
- **Economic Value**: Maps are tradeable items with market value

The system creates exciting gameplay moments, encourages exploration, and provides valuable rewards for players willing to take on the challenge of higher-level treasure hunts.

