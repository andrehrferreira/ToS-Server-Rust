# Gathering System Documentation

## Overview

The Gathering system is a unique resource collection mechanic where players can collect virtually any resource they encounter, regardless of skill level. However, skills and equipment bonuses significantly impact collection speed and profitability. The system uses a tick-based collection mechanism where resources have a limited number of collection attempts (ticks), with rarer resources having fewer ticks. Resource respawn is controlled server-side by timeout and is prevented when players are nearby to avoid spot camping.

## Key Features

- **No Skill Restrictions**: Players can collect any resource regardless of skill level
- **No Tool Requirements**: Tools are optional, not required for collection
- **Skill-Based Efficiency**: Skills determine collection speed and success rate
- **Optional Tool Bonuses**: Tools provide significant bonuses when equipped (skill boost, rare item chance, yield increase, speed boost)
- **Tick-Based Collection**: Resources have 1-20 ticks (collection attempts)
- **Rare Resource Limits**: Rarer resources limited to 5 ticks maximum
- **RMT Spots**: Larger resource spots available through RMT (Real Money Transaction)
- **Server-Controlled Respawn**: Respawn managed by timeout, blocked when players nearby
- **Progressive Rewards**: Higher skills provide chances to double/triple collection amounts
- **Crafted Tools**: Tools are crafted by blacksmiths using rare minerals and gemstones

## Resource Collection Mechanics

### Basic Collection Rules

**No Skill Requirement:**
- All resources can be collected regardless of skill level
- Skill level affects efficiency, not accessibility
- Minimum skill level (typically 1) ensures basic return and standard collection speed

**Skill Below Minimum:**
- More collection failures (missed attempts)
- Slower collection speed
- Reduced resource yield
- Still collectable but inefficient

**Skill At Minimum:**
- Standard collection speed
- Normal success rate
- Base resource yield

**Skill Above Required:**
- Faster collection speed
- Higher success rate
- Chance to double or triple collection amount
- Better resource quality chances

### Tick System

**Tick Range:**
- Common resources: 10-20 ticks
- Uncommon resources: 10-20 ticks
- Rare resources: 5-10 ticks
- Very rare resources: 1-5 ticks
- Ultra-rare resources: 1 tick (single collection)

**Tick Consumption:**
- Each collection attempt consumes 1 tick (default)
- Some resources consume all ticks at once (`consumeAllTicks = true`)
- When ticks reach 0, resource is exhausted
- Exhausted resources cannot be collected until respawn

**Tick Examples:**
```typescript
// Common resources
TreeSpot: 10-20 ticks
StoneSpot: 10-20 ticks
BushSpot: 10-20 ticks

// Uncommon resources
CooperSpot: 10-20 ticks
IronSpot: 10-20 ticks

// Rare resources
SilverSpot: 5-10 ticks
GoldSpot: 5-10 ticks

// Very rare resources
DarkSpot: 2-5 ticks
MithrilSpot: 1-3 ticks

// Ultra-rare resources
HeavenlySpot: 1 tick (single collection)
```

## Resource Types

### Gathering Categories

**1. Mining Resources**
- **Skill**: Mining
- **Optional Tool**: Pickaxe (provides bonuses)
- **Resources**: Stone, Copper Ore, Iron Ore, Silver Ore, Gold Ore, Dark Ore, Mithril Ore, Heavenly Ore, Gemstones

**2. Lumberjack Resources**
- **Skill**: Lumberjack
- **Optional Tool**: Axe (provides bonuses)
- **Resources**: Stick, Wood, Leaves, Ipe Wood, Oak Wood, Maple Wood, Root

**3. Herbalism Resources**
- **Skill**: Herbalism
- **Optional Tool**: Scythe (provides bonuses)
- **Resources**: Fiber, Cotton, Arcane Fiber, Root, Demon Mushroom, Leaves, Blood Berry, Fire Flower, Red Fruit, Yellow Flower, Mana Mushroom, Garlic, Oil Plant, Edgy Root

### Resource Spot Types

**Common Spots:**
- **TreeSpot**: Basic tree, 10-20 ticks
- **StoneSpot**: Basic stone, 10-20 ticks
- **BushSpot**: Basic bush, 10-20 ticks

**Uncommon Spots:**
- **CooperSpot**: Copper ore vein, 10-20 ticks
- **IronSpot**: Iron ore vein, 10-20 ticks
- **BigStoneSpot**: Large stone deposit, 30-40 ticks

**Rare Spots:**
- **SilverSpot**: Silver ore vein, 5-10 ticks
- **GoldSpot**: Gold ore vein, 5-10 ticks

**Very Rare Spots:**
- **DarkSpot**: Dark ore vein, 2-5 ticks
- **MithrilSpot**: Mithril ore vein, 1-3 ticks

**Ultra-Rare Spots:**
- **HeavenlySpot**: Heavenly ore vein, 1 tick

**RMT Spots:**
- Larger resource spots available through Real Money Transactions
- More ticks than standard spots
- Better resource yield rates
- Exclusive to RMT purchases

## Skill-Based Collection

### Skill Level Effects

**Collection Speed:**
- Skill level directly affects collection speed
- Higher skills = faster collection animations
- Lower skills = slower, more interrupted collection

**Success Rate:**
- Skill level affects chance of successful collection
- Below minimum skill: Higher failure rate
- At/above minimum skill: Normal success rate
- Much above required: Near-perfect success rate

**Resource Yield:**
- Base yield: `Random.MinMaxInt(1, max)`
- Max yield: `Math.max(skill, itemBase.Max)`
- Equipment bonus adds percentage to base yield
- Higher skills increase maximum potential yield

**Multiplier Chances:**
- Skills significantly above required level have chances to:
  - Double collection amount (2x)
  - Triple collection amount (3x)
- Exact chance calculation based on skill difference

### Skill Requirements by Resource

**Mining:**
- Stone: Skill 1+ (basic)
- Copper Ore: Skill 2+ (better yield)
- Iron Ore: Skill 3+ (better yield)
- Silver Ore: Skill 4+ (better yield)
- Gold Ore: Skill 5+ (better yield)
- Dark Ore: Skill 7+ (required)
- Mithril Ore: Skill 9+ (required)
- Heavenly Ore: Skill 11+ (required)

**Lumberjack:**
- Stick/Wood: Skill 1+ (basic)
- Ipe Wood: Skill 3+ (unlocked)
- Oak Wood: Skill 5+ (unlocked)
- Maple Wood: Skill 5+ (rare chance)

**Herbalism:**
- Fiber/Cotton: Skill 1+ (basic)
- Arcane Fiber: Skill 5+ (unlocked)
- Rare herbs: Skill 1+ (low chance, improves with skill)

## Optional Tool System

### Tool Overview

**Tools Are Optional:**
- Tools are **not required** to collect resources
- Players can collect any resource without tools
- Tools provide significant bonuses when equipped
- Tools are crafted items, not basic equipment

**Tool Types:**
- **Pickaxe**: Optional tool for mining resources
- **Axe**: Optional tool for lumberjack resources
- **Scythe**: Optional tool for herbalism resources

### Tool Bonuses

When equipped, tools provide multiple bonuses:

**1. Effective Skill Level Increase:**
- Tools add a skill level bonus to the player's actual skill
- Example: Skill 5 + Tool (+3) = Effective Skill 8
- Higher effective skill = better resource tables and yields
- Formula: `effectiveSkill = playerSkill + toolSkillBonus`

**2. Rare Item Collection Chance:**
- Tools increase the chance of collecting rare items
- Higher tier tools = higher rare item chance
- Applies to gemstones, rare ores, special materials
- Formula: `rareChance = baseRareChance + toolRareBonus`

**3. Collection Yield Increase:**
- Tools provide percentage bonus to collection amounts
- Example: 50% bonus = +50% to base collection amount
- Example: 100% bonus = +100% to base collection amount
- Formula: `yield = baseYield + (baseYield * toolYieldBonus / 100)`

**4. Collection Speed Reduction:**
- Tools reduce the time required for each collection
- Faster collection = more resources per time unit
- Higher tier tools = greater speed bonuses
- Formula: `collectionTime = baseTime * (1 - toolSpeedBonus / 100)`

### Tool Crafting

**Crafting Requirements:**
- Tools are crafted by **Blacksmiths** (Blacksmithing skill)
- Require **rare minerals** as primary materials
- Require **gemstones** as secondary materials
- Higher tier tools require rarer materials

**Crafting Materials:**
- **Common Tools**: Basic ores (Iron, Copper)
- **Uncommon Tools**: Uncommon ores (Silver, Gold) + Common gemstones
- **Rare Tools**: Rare ores (Dark, Mithril) + Uncommon gemstones
- **Very Rare Tools**: Very rare ores (Heavenly) + Rare gemstones

**Tool Tiers:**
- **Tier 1**: Basic tools, small bonuses
- **Tier 2**: Improved tools, moderate bonuses
- **Tier 3**: Advanced tools, significant bonuses
- **Tier 4**: Master tools, maximum bonuses

**Example Tool Recipe:**
```
Advanced Pickaxe:
- Dark Ore x10
- Mithril Ore x5
- Emerald x2
- Diamond x1
- Blacksmithing Skill: 8+
```

### Tool Attributes

**Tool Attribute Structure:**
```typescript
interface ToolAttributes {
    skillBonus: number;        // Effective skill level increase
    rareChanceBonus: number;   // Percentage increase to rare item chance
    yieldBonus: number;        // Percentage increase to collection yield
    speedBonus: number;        // Percentage reduction to collection time
    durability: number;       // Tool durability (reduces with use)
}
```

**Example Tool Stats:**
```
Master Pickaxe:
- Skill Bonus: +5 levels
- Rare Chance Bonus: +15%
- Yield Bonus: +100%
- Speed Bonus: -30% collection time
- Durability: 1000/1000
```

### Tool Durability

**Durability System:**
- Tools lose durability with each collection
- Durability loss varies by tool tier (higher tier = slower degradation)
- Broken tools (0 durability) still provide bonuses but at reduced effectiveness
- Tools can be repaired by blacksmiths using materials

**Durability Loss:**
- Each collection: -1 to -3 durability (varies by tool)
- Higher tier tools: Less durability loss per use
- Broken tools: Bonuses reduced by 50%

### Bonus Calculation

**Effective Skill Calculation:**
```typescript
const baseSkill = player.getSkillValue(skill);
const toolSkillBonus = tool ? tool.skillBonus : 0;
const effectiveSkill = baseSkill + toolSkillBonus;
```

**Yield Calculation:**
```typescript
const baseYield = Random.MinMaxInt(1, max);
const toolYieldBonus = tool ? tool.yieldBonus : 0;
const yieldBonus = Math.round(baseYield * (toolYieldBonus / 100));
const totalYield = baseYield + yieldBonus;
```

**Rare Item Chance:**
```typescript
const baseRareChance = 1.0; // Base 1% chance
const toolRareBonus = tool ? tool.rareChanceBonus : 0;
const totalRareChance = baseRareChance + toolRareBonus;
```

**Collection Speed:**
```typescript
const baseCollectionTime = 3000; // 3 seconds base
const toolSpeedBonus = tool ? tool.speedBonus : 0;
const actualCollectionTime = baseCollectionTime * (1 - toolSpeedBonus / 100);
```

**Example Calculation:**
- Base skill: 5
- Tool skill bonus: +3
- Effective skill: 8 (uses skill level 8 resource table)
- Base yield: 5-10 resources
- Tool yield bonus: 100%
- Calculation: `Random(5, 10) + round(10 * 1.0)`
- Result: 15-20 resources per collection
- Collection time: 3s * (1 - 0.30) = 2.1 seconds

## Resource Yield System

### Yield Calculation

**Base Formula (Without Tool):**
```typescript
const max = Math.max(itemBase.Max ? itemBase.Max : 1, skill);
let amount = Random.MinMaxInt(1, max);
```

**With Tool Formula:**
```typescript
// Calculate effective skill
const effectiveSkill = skill + (tool ? tool.skillBonus : 0);
const max = Math.max(itemBase.Max ? itemBase.Max : 1, effectiveSkill);

// Base yield
const baseYield = Random.MinMaxInt(1, max);

// Tool yield bonus
const toolYieldBonus = tool ? tool.yieldBonus : 0;
const yieldBonus = Math.round(baseYield * (toolYieldBonus / 100));

// Total yield
let amount = baseYield + yieldBonus;
```

**Components:**
1. **Effective Skill**: `skill + toolSkillBonus` (if tool equipped)
   - Determines which resource table to use
   - Affects maximum yield potential
2. **Base Random**: `Random.MinMaxInt(1, max)`
   - Minimum: 1
   - Maximum: `max(effectiveSkill, itemBase.Max)`
3. **Tool Yield Bonus**: `round(baseYield * (toolYieldBonus / 100))`
   - Percentage of base yield added
   - Only applies if tool is equipped

**Example Calculations:**

**Low Skill, No Tool:**
- Skill: 2
- Max: 2
- Tool: None
- Yield: 1-2 resources

**Medium Skill, No Tool:**
- Skill: 5
- Max: 5
- Tool: None
- Yield: 1-5 resources

**Medium Skill, Basic Tool:**
- Skill: 5
- Tool Skill Bonus: +2
- Effective Skill: 7
- Max: 7
- Base Yield: 1-7 resources
- Tool Yield Bonus: 50%
- Yield Bonus: +3-4 resources
- Total Yield: 4-11 resources

**High Skill, Master Tool:**
- Skill: 10
- Tool Skill Bonus: +5
- Effective Skill: 15 (capped at resource max)
- Max: 15
- Base Yield: 1-15 resources
- Tool Yield Bonus: 100%
- Yield Bonus: +1-15 resources
- Total Yield: 2-30 resources

### Resource Per Level Tables

Resources have different yield tables based on skill level:

**TreeSpot Example:**
```typescript
resourcePerLevel = {
    1: [
        { Item: Stick, Chance: 16, Max: 20 },
        { Item: Wood, Chance: 80, Max: 20 },
        { Item: Leaves, Chance: 5, Max: 20 }
    ],
    2: [
        { Item: Stick, Chance: 5, Max: 20 },
        { Item: Wood, Chance: 90, Max: 20 },
        { Item: Leaves, Chance: 5, Max: 20 }
    ],
    3: [
        { Item: Stick, Chance: 10, Max: 20 },
        { Item: Wood, Chance: 80, Max: 20 },
        { Item: Leaves, Chance: 5, Max: 20 },
        { Item: IpeWood, Chance: 5, Max: 20 }  // Unlocked at skill 3
    ],
    // ... higher levels unlock better resources
}
```

**Selection Process:**
1. Determine skill level (rounded)
2. Look up resource table for that level
3. Use weighted random selection based on chances
4. Apply max yield limit for selected resource

## Respawn System

### Respawn Mechanics

**Server-Controlled:**
- Respawn is managed entirely by the server
- Client cannot influence respawn timing
- Prevents exploitation and camping

**Timeout-Based:**
- Each resource spot has a respawn timer
- Default timer: 120 seconds (2 minutes)
- Timer starts when resource is collected/exhausted
- Resource respawns when timeout expires

**Player Proximity Check:**
- Respawn is blocked if players are nearby
- Prevents players from camping spots
- Distance check: ~1000 units (configurable)
- Respawn only occurs when no players in range

### Respawn Implementation

**TypeScript:**
```typescript
collected() {
    this.entityRespawned = null;
    this.settings.timeout = new Date().getTime() + (this.settings.timer * 1000);
}

tick() {
    if (!this.entityRespawned && new Date().getTime() > this.settings.timeout) {
        // Check for nearby players
        const nearbyPlayers = this.map.getPlayersInRange(
            this.settings.position, 
            1000
        );
        
        if (nearbyPlayers.length === 0) {
            this.settings.timeout = new Date().getTime() + (this.settings.timer * 1000);
            this.createEntity();
        }
    }
}
```

**Respawn Process:**
1. Resource is collected/exhausted
2. `collected()` sets timeout timestamp
3. `tick()` checks every 60 seconds (1 minute)
4. If timeout expired AND no players nearby:
   - Create new resource entity
   - Reset timeout
5. If players nearby:
   - Skip respawn this cycle
   - Check again next cycle

### Respawn Timers

**Standard Resources:**
- Default: 120 seconds (2 minutes)
- Can be configured per resource type
- Longer timers for rarer resources

**Rare Resources:**
- May have longer respawn timers
- Prevents over-farming
- Maintains resource scarcity

## Collection Process

### Collection Flow

**1. Start Gathering:**
```typescript
public startGathering(payload: string) {
    const gatheringSpot = this.map.foliage.get(payload);
    if (gatheringSpot) {
        this.gatherableSpot = gatheringSpot;
        this.gatherableInteract = gatheringSpot.entityRespawned;
    }
}
```

**2. Collect Resource:**
```typescript
public async collect() {
    if (this.gatherableInteract) {
        await this.gatherableInteract.collect(this);
        
        if (this.gatherableInteract.tick <= 0) {
            this.map.setFoliageAsCollected(this.gatherableInteract.foliageId);
            this.gatherableSpot.collected();
            this.gatherableInteract = null;
        }
    }
}
```

**3. Collection Logic:**
- Check if resource has ticks remaining
- Check if player has optional tool equipped
- Calculate effective skill (base skill + tool bonus)
- Select resource table based on effective skill
- Apply tool bonuses (yield, rare chance, speed)
- Generate random resource amount with bonuses
- Consume tick(s)
- Add resources to inventory
- Update client with remaining ticks
- Gain skill experience (if below max)
- Reduce tool durability (if tool equipped)

### Collection Validation

**Distance Check:**
- Player must be within 1000 units of resource
- Prevents remote collection exploits
- Validates position on each collection

**Tool Check (Optional):**
- Tool is optional, not required
- If tool equipped: Applies all tool bonuses
- If no tool: Uses base skill and no bonuses
- Broken tools still provide bonuses (at 50% effectiveness)

**Tick Check:**
- Resource must have ticks remaining
- Exhausted resources cannot be collected
- Client notified when resource exhausted

## Skill Experience Gain

### Experience from Gathering

**Skill Gain Conditions:**
- Skill must be below `maxSkillGain` threshold
- Prevents infinite skill grinding
- Encourages progression to new resources

**Gain Amount:**
- Base: 3 XP per collection
- May vary by resource type
- Higher-tier resources may grant more XP

**Max Skill Gain Examples:**
- TreeSpot: Max skill gain at level 5
- StoneSpot: Max skill gain at level 4
- CooperSpot: Max skill gain at level 5
- SilverSpot: Max skill gain at level 6
- GoldSpot: Max skill gain at level 8
- DarkSpot: Max skill gain at level 10
- MithrilSpot: Max skill gain at level 11
- HeavenlySpot: Max skill gain at level 13

**Implementation:**
```typescript
if (playerSkill < this.maxSkillGain) {
    player.gainSkillExperiencie(this.skill);
}
```

## Resource Spot Management

### Spot Registration

**Static Spots:**
- Defined in map data
- Loaded on map initialization
- Persistent across server restarts

**Dynamic Spots:**
- Can be created at runtime
- RMT spots added dynamically
- Temporary spots for events

### Spot Storage

**Map Structure:**
```typescript
public foliage: Map<string, Gatherable> = new Map<string, Gatherable>();
```

**Key Format:**
- Position-based: `${x}${y}${z}`
- Mesh-based: `meshIndex`
- Unique identifier per spot

### Spot Serialization

**Save Format:**
```typescript
// Format: "locRef,foliageId,enable,tick|..."
const data = `${locRef},${foliageId},${enable},${tick}`;
```

**Components:**
- `locRef`: Location reference (`${x}${y}${z}`)
- `foliageId`: Resource instance ID
- `enable`: Whether resource is available (1 or 0)
- `tick`: Remaining ticks

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum GatherableType {
    Tree,
    Stone,
    Bush,
    BigStone,
    IronSpot,
    CooperSpot,
    SilverSpot,
    GoldSpot,
    Coal,
    DarkSpot,
    MithrilSpot,
    HeavenlySpot,
}

pub struct ToolAttributes {
    pub skill_bonus: u8,           // Effective skill level increase
    pub rare_chance_bonus: f32,    // Percentage increase to rare item chance
    pub yield_bonus: f32,          // Percentage increase to collection yield
    pub speed_bonus: f32,          // Percentage reduction to collection time
    pub durability: u32,           // Current durability
    pub max_durability: u32,       // Maximum durability
}

pub struct GatherableResource {
    pub resource_type: GatherableType,
    pub skill: SkillName,
    pub optional_tool_type: Option<EquipamentType>,  // Optional tool type
    pub tick: u8,
    pub max_skill_gain: u8,
    pub index_great_level: u8,
    pub consume_all_ticks: bool,
    pub resource_per_level: HashMap<u8, Vec<ResourceDrop>>,
}

pub struct ResourceDrop {
    pub item: ItemType,
    pub chance: u32,  // Weighted chance
    pub max: Option<u8>,  // Maximum yield
}

pub struct Gatherable {
    pub settings: GatherableSettings,
    pub entity_respawned: Option<GatherableResource>,
    pub timeout: SystemTime,
}

pub struct GatherableSettings {
    pub map: String,
    pub position: Vector3,
    pub timer: u64,  // Respawn timer in seconds
    pub respawn_on_start: bool,
    pub entities: Vec<GatherableType>,
    pub foliage_id: String,
}
```

### Collection Logic

```rust
impl GatherableResource {
    pub async fn collect(
        &mut self,
        player: &mut Player,
    ) -> Result<CollectionResult, CollectionError> {
        // Validate collection
        if self.tick == 0 {
            return Err(CollectionError::Exhausted);
        }
        
        // Check distance
        let distance = self.settings.position.distance_to(player.position);
        if distance > 1000.0 {
            return Err(CollectionError::TooFar);
        }
        
        // Get base skill level
        let base_skill = player.get_skill_value(self.skill);
        let base_skill_level = base_skill.max(1.0) as u8;
        
        // Check for optional tool
        let tool = if let Some(tool_type) = self.optional_tool_type {
            player.get_tool(tool_type)
        } else {
            None
        };
        
        // Calculate effective skill (base + tool bonus)
        let effective_skill = if let Some(ref tool) = tool {
            let tool_bonus = if tool.durability > 0 {
                tool.skill_bonus
            } else {
                // Broken tools provide 50% effectiveness
                (tool.skill_bonus / 2).max(0)
            };
            (base_skill_level + tool_bonus).min(20) // Cap at reasonable level
        } else {
            base_skill_level
        };
        
        // Select resource table based on effective skill
        let resource_table = self.get_resource_table(effective_skill);
        
        // Select random resource
        let resource_drop = self.select_random_resource(&resource_table);
        
        // Calculate base yield
        let max_yield = resource_drop.max.unwrap_or(effective_skill as u8);
        let base_yield = thread_rng().gen_range(1..=max_yield);
        
        // Apply tool yield bonus (if tool equipped)
        let yield_bonus = if let Some(ref tool) = tool {
            let tool_bonus = if tool.durability > 0 {
                tool.yield_bonus
            } else {
                tool.yield_bonus * 0.5 // Broken tools: 50% effectiveness
            };
            (base_yield as f32 * (tool_bonus / 100.0)) as u8
        } else {
            0
        };
        
        let total_yield = base_yield + yield_bonus;
        
        // Calculate rare item chance (if applicable)
        let rare_chance = if let Some(ref tool) = tool {
            let tool_bonus = if tool.durability > 0 {
                tool.rare_chance_bonus
            } else {
                tool.rare_chance_bonus * 0.5 // Broken tools: 50% effectiveness
            };
            // Apply rare chance bonus to resource selection
            // (Implementation depends on resource table structure)
            tool_bonus
        } else {
            0.0
        };
        
        // Consume tick
        self.tick -= 1;
        
        // Add to inventory
        player.inventory.add_item(resource_drop.item, total_yield);
        
        // Gain skill experience (if below max)
        if base_skill < self.max_skill_gain as f32 {
            player.gain_skill_experience(self.skill, 3);
        }
        
        // Reduce tool durability (if tool equipped)
        if let Some(tool_type) = self.optional_tool_type {
            if let Some(_) = tool {
                player.reduce_tool_durability(tool_type);
            }
        }
        
        // Calculate collection time (for client)
        let collection_time = if let Some(ref tool) = tool {
            let speed_bonus = if tool.durability > 0 {
                tool.speed_bonus
            } else {
                tool.speed_bonus * 0.5 // Broken tools: 50% effectiveness
            };
            3000.0 * (1.0 - speed_bonus / 100.0) // Base 3 seconds
        } else {
            3000.0 // Base collection time
        };
        
        Ok(CollectionResult {
            item: resource_drop.item,
            amount: total_yield,
            remaining_ticks: self.tick,
            collection_time: collection_time as u32,
        })
    }
    
    fn get_resource_table(&self, skill_level: u8) -> &Vec<ResourceDrop> {
        // Find closest skill level table
        for level in (1..=skill_level).rev() {
            if let Some(table) = self.resource_per_level.get(&level) {
                return table;
            }
        }
        // Fallback to great level
        self.resource_per_level
            .get(&self.index_great_level)
            .unwrap_or(&self.resource_per_level[&1])
    }
    
    fn select_random_resource(&self, table: &Vec<ResourceDrop>) -> &ResourceDrop {
        let total_chance: u32 = table.iter().map(|r| r.chance).sum();
        let mut random = thread_rng().gen_range(0..total_chance);
        
        for resource in table {
            if random < resource.chance {
                return resource;
            }
            random -= resource.chance;
        }
        
        table.last().unwrap()
    }
}
```

### Respawn System

```rust
impl Gatherable {
    pub fn tick(&mut self, map: &Map) {
        if self.entity_respawned.is_some() {
            return; // Already spawned
        }
        
        if SystemTime::now() < self.timeout {
            return; // Not time to respawn yet
        }
        
        // Check for nearby players
        let nearby_players = map.get_players_in_range(
            self.settings.position,
            1000.0,
        );
        
        if !nearby_players.is_empty() {
            return; // Players nearby, don't respawn
        }
        
        // Respawn resource
        self.create_entity();
        self.timeout = SystemTime::now() + 
            Duration::from_secs(self.settings.timer);
    }
    
    fn create_entity(&mut self) {
        let random_type = self.settings.entities
            [thread_rng().gen_range(0..self.settings.entities.len())];
        
        self.entity_respawned = Some(
            GatherableResource::create(random_type, 0)
        );
    }
}
```

### Performance Optimizations

1. **Spatial Indexing**: Use spatial hash or grid for fast player proximity checks
2. **Batch Processing**: Process multiple respawns in batches
3. **Lazy Evaluation**: Only check respawn when timeout expired
4. **Caching**: Cache resource tables per skill level
5. **Lock-Free Structures**: Use atomic operations for tick counting

## Integration with Other Systems

### Inventory System

- Collected resources automatically added to inventory
- Stackable items combine automatically
- Inventory weight affects player mobility

### Equipment System

- Tools are optional but provide significant bonuses
- Tools crafted by blacksmiths using rare materials
- Tool durability decreases with use
- Broken tools provide reduced bonuses (50% effectiveness)
- Tool bonuses affect effective skill, yield, rare chance, and speed

### Skill System

- Gathering skills gain experience from collection
- Skill level affects resource yield and quality
- Higher skills unlock better resources
- Tools provide effective skill level increase

### Crafting System

- Tools are crafted by blacksmiths (Blacksmithing skill)
- Require rare minerals as primary materials
- Require gemstones as secondary materials
- Higher tier tools require rarer materials
- Tool crafting creates valuable items for gathering economy
- Tools can be repaired by blacksmiths

### Economy System

- Collected resources can be sold
- Rarer resources have higher value
- Resource scarcity affects market prices
- Tools are valuable crafted items
- Tool crafting creates demand for rare minerals and gemstones
- Tool repair creates ongoing demand for materials

## Testing Strategy

### Unit Tests

- Test collection yield calculation
- Test skill-based resource table selection
- Test equipment bonus application
- Test tick consumption
- Test respawn timing and proximity checks

### Integration Tests

- Test full collection flow
- Test skill experience gain
- Test tool durability reduction
- Test inventory addition
- Test resource exhaustion

### Balance Tests

- Test resource yield rates across skill levels
- Test respawn timer effectiveness
- Test player proximity blocking
- Test RMT spot differences
- Test rare resource scarcity

