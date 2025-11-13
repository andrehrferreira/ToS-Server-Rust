# Ancient Scrolls System Documentation

## Overview

Ancient Scrolls are extremely rare and powerful items that can only be obtained by defeating gods. These scrolls allow players to permanently increase the maximum level cap of specific skills beyond the standard limit of 100, providing significant competitive advantages in combat, crafting, gathering, and other gameplay systems. The system creates a high-end progression path that rewards players who can overcome the ultimate challenges in the game.

## Key Features

- **God-Exclusive Drops**: Only obtainable by defeating gods
- **Party Rewards**: Entire party receives 6 random Ancient Scrolls upon god defeat
- **Skill Cap Increase**: Permanently increases skill cap to 105, 110, 115, or 120
- **Random Distribution**: Scrolls are randomly assigned to skills
- **RMT Random**: Real Money Transaction scrolls are completely random
- **Tradeable Items**: Scrolls can be traded between players
- **Permanent Enhancement**: Once used, permanently increases skill cap
- **Competitive Advantage**: Significant impact on player power and progression
- **Ecosystem Impact**: Affects combat, crafting, gathering, and all skill-based systems

## Ancient Scroll Mechanics

### Obtaining Ancient Scrolls

**Source:**
- **Only Source**: Defeating gods (boss encounters)
- **Drop Rate**: Guaranteed drop (6 scrolls per god defeat)
- **Distribution**: Entire party receives scrolls (not individual drops)
- **Random Assignment**: Each scroll is randomly assigned to a skill

**God Defeat Rewards:**
- Party receives exactly 6 Ancient Scrolls
- Scrolls are distributed to party members
- Each scroll targets a random skill
- Scroll level (105, 110, 115, 120) is random
- No duplicate prevention (can receive multiple scrolls for same skill)

### Scroll Types

**Cap Increase Levels:**
- **Ancient Scroll +5**: Increases skill cap to 105 (from 100)
- **Ancient Scroll +10**: Increases skill cap to 110 (from 100)
- **Ancient Scroll +15**: Increases skill cap to 115 (from 100)
- **Ancient Scroll +20**: Increases skill cap to 120 (from 100)

**Scroll Distribution:**
- Each scroll randomly targets one of the 24 available skills
- Scroll level (105/110/115/120) is randomly determined
- No guarantee of specific skill or level
- RMT scrolls follow same random distribution

### Scroll Properties

**Item Properties:**
- **Item Type**: Consumable scroll item
- **Rarity**: Legendary/Unique tier
- **Stackable**: No (each scroll is individual)
- **Tradeable**: Yes (can be traded between players)
- **Droppable**: Yes (can be dropped/traded)
- **Bind on Use**: No (remains tradeable after use)

**Scroll Information:**
- Displays target skill name
- Displays cap increase amount (+5, +10, +15, +20)
- Shows current skill cap vs new cap
- Indicates if skill cap is already at or above scroll level

## Using Ancient Scrolls

### Usage Process

**Step 1: Double-Click**
- Player double-clicks Ancient Scroll in inventory
- System checks if scroll can be used
- Validation checks are performed

**Step 2: Confirmation Dialog**
- Confirmation dialog appears
- Shows skill name and cap increase
- Shows current cap vs new cap
- Player must confirm to proceed

**Step 3: Consumption**
- Scroll is consumed (removed from inventory)
- Skill cap is permanently increased
- Player receives notification of cap increase
- Change is saved to database

### Usage Requirements

**Validation Checks:**
- Player must have the target skill
- Current skill cap must be below scroll's target cap
- If skill cap is already at or above scroll level, scroll cannot be used
- Example: Cannot use +10 scroll if skill cap is already 110+

**Usage Restrictions:**
- Cannot use scroll if skill cap already exceeds scroll level
- Cannot use scroll if skill cap equals scroll level
- Can use scroll if skill cap is below scroll level (even if already increased)

**Example Scenarios:**
- **Skill Cap 100, Scroll +10**: ✅ Can use → Cap becomes 110
- **Skill Cap 105, Scroll +10**: ✅ Can use → Cap becomes 110
- **Skill Cap 110, Scroll +10**: ❌ Cannot use (already at or above)
- **Skill Cap 115, Scroll +10**: ❌ Cannot use (already above)

### Cap Increase Mechanics

**Permanent Enhancement:**
- Cap increase is permanent
- Cannot be reversed
- Persists through character death
- Persists through respecs
- Saved to character database

**Cap Stacking:**
- Multiple scrolls can be used on same skill
- Each scroll increases cap if current cap is below scroll level
- Example: Use +5 scroll (100→105), then +10 scroll (105→110), then +15 scroll (110→115)

**Maximum Cap:**
- Theoretical maximum: 120 (using +20 scroll)
- Can use multiple scrolls to reach maximum
- No hard limit beyond scroll availability

## Impact on Gameplay Systems

### Combat Skills Impact

**Damage Amplification:**
- Higher skill levels = more damage bonus
- Level 100: +45% damage
- Level 105: +47.5% damage (+2.5% increase)
- Level 110: +50% damage (+5% increase)
- Level 115: +52.5% damage (+7.5% increase)
- Level 120: +55% damage (+10% increase)

**Accuracy Enhancement:**
- Higher skill levels = better accuracy
- Level 100: +20% accuracy
- Level 105: +21% accuracy (+1% increase)
- Level 110: +22% accuracy (+2% increase)
- Level 115: +23% accuracy (+3% increase)
- Level 120: +24% accuracy (+4% increase)

**Perk Unlocks:**
- No additional perks beyond level 100
- Perks remain at levels 20, 40, 60, 80, 100
- Higher levels provide only stat bonuses and skill multipliers

**PvP Advantage:**
- Players with higher skill caps have significant advantages
- More damage output
- Better accuracy
- Superior combat effectiveness
- Creates meaningful power differences

### Crafting Skills Impact

**Crafting Success Rates:**
- Higher skill levels = better crafting success
- Can craft higher-tier items
- Better rarity chances
- More consistent high-quality results

**Crafting Efficiency:**
- Higher levels = faster crafting
- Better material efficiency
- Reduced failure rates
- Improved item quality

**Economic Impact:**
- Players with higher crafting caps can create better items
- Access to exclusive high-tier recipes
- Better market position
- More profitable crafting operations

### Gathering Skills Impact

**Gathering Efficiency:**
- Higher skill levels = faster gathering
- Better success rates
- More resources per gathering attempt
- Higher chance for rare materials

**Resource Advantage:**
- More consistent resource acquisition
- Better rare material rates
- Improved gathering speed
- Competitive advantage in resource markets

### Magic Skills Impact

**Spell Effectiveness:**
- Higher skill levels = more spell damage
- Better spell success rates
- Improved mana efficiency
- Enhanced magical capabilities

**Magic Resistance:**
- Higher Magic Resistance skill = better defense
- Reduced magical damage taken
- Better survivability
- Improved PvP performance

## Trading and Economy

### Trade System

**Tradeable Items:**
- Ancient Scrolls can be traded between players
- No binding restrictions
- Can be sold on auction house
- Can be traded directly

**Market Value:**
- Extremely valuable items
- High demand from competitive players
- Price varies by:
  - Scroll level (+5, +10, +15, +20)
  - Target skill popularity
  - Server economy state
  - Availability

**Trade Considerations:**
- Players may prefer specific skills
- Scrolls for popular skills command higher prices
- Combat skill scrolls typically more valuable
- Crafting skill scrolls valuable for crafters

### RMT (Real Money Transaction) System

**RMT Scrolls:**
- RMT scrolls are completely random
- Same distribution as god drops
- Random skill assignment
- Random level (105/110/115/120)
- No way to choose skill or level

**RMT Purchase:**
- Players purchase random Ancient Scrolls
- Cannot select specific skill
- Cannot select specific level
- Adds to economy and progression options

## Integration with God System

### God Defeat Rewards

**Party Distribution:**
- Entire party receives 6 Ancient Scrolls
- Distributed equally among party members
- Each member receives scrolls
- Random assignment per scroll

**Distribution Mechanics:**
- 6 scrolls total per god defeat
- Random skill assignment for each scroll
- Random level assignment for each scroll
- No duplicate prevention

**Example Distribution:**
- Party of 4 defeats god
- Each member receives 1-2 scrolls (depending on distribution method)
- Scrolls are random (e.g., Combat With Weapons +10, Mining +5, Alchemy +15, etc.)

### God System Integration

**Note:** Detailed god system documentation will be created in separate file.

**Basic Integration:**
- Gods are ultimate boss encounters
- Require coordinated party effort
- Drop Ancient Scrolls as primary reward
- Create endgame progression goals

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum AncientScrollLevel {
    Plus5,   // Cap to 105
    Plus10,  // Cap to 110
    Plus15,  // Cap to 115
    Plus20,  // Cap to 120
}

impl AncientScrollLevel {
    pub fn target_cap(&self) -> u8 {
        match self {
            AncientScrollLevel::Plus5 => 105,
            AncientScrollLevel::Plus10 => 110,
            AncientScrollLevel::Plus15 => 115,
            AncientScrollLevel::Plus20 => 120,
        }
    }
    
    pub fn from_value(value: u8) -> Option<Self> {
        match value {
            105 => Some(AncientScrollLevel::Plus5),
            110 => Some(AncientScrollLevel::Plus10),
            115 => Some(AncientScrollLevel::Plus15),
            120 => Some(AncientScrollLevel::Plus20),
            _ => None,
        }
    }
}

#[derive(Debug, Clone)]
pub struct AncientScroll {
    pub scroll_id: u64,
    pub target_skill: SkillName,
    pub scroll_level: AncientScrollLevel,
    pub target_cap: u8,
}

impl AncientScroll {
    pub fn new(target_skill: SkillName, scroll_level: AncientScrollLevel) -> Self {
        Self {
            scroll_id: generate_unique_id(),
            target_skill,
            scroll_level,
            target_cap: scroll_level.target_cap(),
        }
    }
    
    pub fn can_use(&self, current_cap: u8) -> bool {
        current_cap < self.target_cap
    }
    
    pub fn use(&self, skill: &mut SkillValue) -> Result<(), String> {
        if !self.can_use(skill.cap) {
            return Err(format!(
                "Skill cap {} is already at or above scroll target {}",
                skill.cap, self.target_cap
            ));
        }
        
        skill.cap = self.target_cap;
        Ok(())
    }
}
```

### Scroll Generation

```rust
pub fn generate_ancient_scrolls_for_party(party_size: usize) -> Vec<AncientScroll> {
    let mut scrolls = Vec::new();
    let mut rng = thread_rng();
    
    // Generate 6 random scrolls
    for _ in 0..6 {
        // Random skill selection
        let skills: Vec<SkillName> = SkillName::iter()
            .filter(|s| *s != SkillName::None)
            .collect();
        let target_skill = skills[rng.gen_range(0..skills.len())];
        
        // Random scroll level (105, 110, 115, 120)
        let scroll_level = match rng.gen_range(0..4) {
            0 => AncientScrollLevel::Plus5,
            1 => AncientScrollLevel::Plus10,
            2 => AncientScrollLevel::Plus15,
            _ => AncientScrollLevel::Plus20,
        };
        
        scrolls.push(AncientScroll::new(target_skill, scroll_level));
    }
    
    scrolls
}

pub fn distribute_scrolls_to_party(
    scrolls: Vec<AncientScroll>,
    party_members: Vec<PlayerId>,
) -> HashMap<PlayerId, Vec<AncientScroll>> {
    let mut distribution: HashMap<PlayerId, Vec<AncientScroll>> = HashMap::new();
    
    // Initialize distribution for each party member
    for member_id in &party_members {
        distribution.insert(*member_id, Vec::new());
    }
    
    // Distribute scrolls evenly (or randomly)
    for (index, scroll) in scrolls.into_iter().enumerate() {
        let member_index = index % party_members.len();
        let member_id = party_members[member_index];
        distribution.get_mut(&member_id).unwrap().push(scroll);
    }
    
    distribution
}
```

### Scroll Usage

```rust
pub fn use_ancient_scroll(
    player: &mut Player,
    scroll: AncientScroll,
) -> Result<(), String> {
    // Get target skill
    let skill_value = player.get_skill_mut(scroll.target_skill)
        .ok_or("Skill not found")?;
    
    // Validate usage
    if !scroll.can_use(skill_value.cap) {
        return Err(format!(
            "Cannot use scroll: skill cap {} is already at or above target {}",
            skill_value.cap, scroll.target_cap
        ));
    }
    
    // Apply scroll
    scroll.use(skill_value)?;
    
    // Notify player
    notify_scroll_used(player, &scroll);
    
    // Save to database
    player.save_skills();
    
    Ok(())
}

pub fn handle_scroll_double_click(
    player: &mut Player,
    scroll_item_id: ItemId,
) -> Result<(), String> {
    // Get scroll from inventory
    let scroll_item = player.inventory.get_item(scroll_item_id)
        .ok_or("Scroll not found")?;
    
    // Extract scroll data from item
    let scroll = extract_scroll_from_item(scroll_item)?;
    
    // Show confirmation dialog
    show_scroll_confirmation_dialog(player, &scroll)?;
    
    // Wait for player confirmation (handled by client/server protocol)
    // On confirmation, call use_ancient_scroll
    
    Ok(())
}
```

### God Defeat Integration

```rust
pub fn on_god_defeated(
    god: &GodEntity,
    party: &Party,
) -> Vec<AncientScroll> {
    // Generate 6 random scrolls
    let scrolls = generate_ancient_scrolls_for_party(party.members.len());
    
    // Distribute to party members
    let distribution = distribute_scrolls_to_party(
        scrolls.clone(),
        party.members.clone(),
    );
    
    // Give scrolls to each party member
    for (player_id, player_scrolls) in distribution {
        if let Some(player) = get_player_mut(player_id) {
            for scroll in player_scrolls {
                let scroll_item = create_scroll_item(scroll);
                player.inventory.add_item(scroll_item)?;
            }
            notify_god_reward(player, player_scrolls.len());
        }
    }
    
    scrolls
}
```

### Item Creation

```rust
pub fn create_scroll_item(scroll: AncientScroll) -> Item {
    Item::new()
        .with_type(ItemType::Consumable)
        .with_subtype(ItemSubtype::AncientScroll)
        .with_name(format!(
            "Ancient Scroll of {} (+{})",
            skill_name_to_string(scroll.target_skill),
            scroll.target_cap - 100
        ))
        .with_rarity(ItemRarity::Legendary)
        .with_description(format!(
            "Increases {} skill cap to {}",
            skill_name_to_string(scroll.target_skill),
            scroll.target_cap
        ))
        .with_metadata(ScrollMetadata {
            target_skill: scroll.target_skill,
            scroll_level: scroll.scroll_level,
            target_cap: scroll.target_cap,
        })
        .build()
}

pub fn extract_scroll_from_item(item: &Item) -> Result<AncientScroll, String> {
    let metadata = item.metadata.get::<ScrollMetadata>()
        .ok_or("Item is not an Ancient Scroll")?;
    
    Ok(AncientScroll {
        scroll_id: item.id,
        target_skill: metadata.target_skill,
        scroll_level: metadata.scroll_level,
        target_cap: metadata.target_cap,
    })
}
```

## Balance Considerations

### Power Scaling

**Cap Increase Impact:**
- Each +5 cap increase provides:
  - +2.5% damage (combat skills)
  - +1% accuracy (all skills)
  - Additional stat experience potential
  - Better skill bonuses

**Maximum Advantage:**
- Player with all skills at 120 vs player at 100:
  - +10% damage advantage
  - +4% accuracy advantage
  - Significant stat point advantage
  - Better overall performance

### Progression Balance

**Acquisition Rate:**
- Limited by god defeat frequency
- Requires coordinated party effort
- High difficulty barrier
- Creates meaningful progression goals

**Distribution Balance:**
- Random distribution prevents guaranteed power
- Players must trade or use what they get
- Creates economic activity
- Prevents instant max power

### Economic Balance

**Market Dynamics:**
- High-value items create trading activity
- Players trade scrolls for desired skills
- Creates economic ecosystem
- Supports player-to-player economy

**RMT Balance:**
- RMT scrolls are random (no pay-to-win advantage)
- Same distribution as god drops
- Provides alternative acquisition method
- Maintains game balance

## Integration with Other Systems

### Skill System Integration

**Cap Management:**
- Skills track current cap
- Cap can exceed 100 with scrolls
- Experience continues to accumulate
- Level progression continues to new cap

**Skill Bonus Calculation:**
- Bonuses scale with actual level (up to cap)
- Higher caps = higher potential bonuses
- Perks remain at fixed levels (20, 40, 60, 80, 100)
- No additional perks beyond level 100

### Player Progression Integration

**Character Power:**
- Higher skill caps = more character power
- Significant advantage in PvP
- Better performance in PvE
- Improved crafting/gathering capabilities

**Progression Goals:**
- God defeats become progression goals
- Scroll acquisition creates long-term goals
- Trading creates economic goals
- Skill cap increases create power goals

### Equipment System Integration

**Skill Requirements:**
- Higher-tier equipment may require skill levels > 100
- Scrolls enable access to exclusive equipment
- Creates progression gates
- Rewards high-skill-cap players

### Crafting System Integration

**Recipe Requirements:**
- Some recipes may require skill levels > 100
- Scrolls enable access to exclusive recipes
- Creates crafting progression
- Rewards dedicated crafters

## Summary

Ancient Scrolls are a high-end progression system that:

- **Reward God Defeats**: Only obtainable by defeating gods
- **Party Rewards**: Entire party receives 6 random scrolls
- **Cap Increases**: Permanently increase skill caps to 105, 110, 115, or 120
- **Random Distribution**: Scrolls randomly target skills and levels
- **Tradeable Items**: Can be traded between players
- **Permanent Enhancement**: Once used, permanently increases skill cap
- **Competitive Advantage**: Significant impact on player power
- **Ecosystem Impact**: Affects combat, crafting, gathering, and all skill systems
- **RMT Integration**: RMT scrolls follow same random distribution
- **Economic Activity**: Creates trading and market dynamics

The system creates meaningful endgame progression goals, rewards skilled players who can defeat gods, and provides significant competitive advantages while maintaining balance through random distribution and high acquisition barriers.

