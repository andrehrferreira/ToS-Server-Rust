# Crafting System Documentation

## Overview

The crafting system allows players to create equipment and items using resources and crafting skills. The system is inspired by Ultima Online, where the same item can be crafted using different resources of varying quality. Higher quality resources increase skill requirements and XP gain, while also improving the chances of creating higher rarity items.

**Note:** Crafting never fails - players can always create items if they have the required resources. However, skill level and resource quality determine the final item quality and rarity.

## Key Features

- **No Crafting Failures**: Players can always create items if they have resources
- **Skill-Based Quality**: Low skill = common items, high skill = better rarity chances
- **Resource Variety**: Same item can be crafted with different resource types/qualities
- **Progressive XP**: Higher quality resources give more XP (up to 10% above minimum requirement)
- **Rarity System**: Chance-based system for uncommon, rare, magic, and legendary items
- **Gold Cost**: All crafting requires gold payment
- **Specialized Tools**: Tools increase rarity chances but cost more to use
- **Destruction System**: Items can be destroyed to recover partial resources
- **Recipe System**: All items have recipes listing required resources
- **Component Tracking**: Items track their component resources for destruction recovery

## Crafting Skills

### Skill Types

**Blacksmithing:**
- Used for Metal/Mineral equipment
- Creates: Armor, Boots, Gloves, Helmet, Mainhand (metal), Offhand (metal), Belt (metal), Rings (metal), Necklaces (metal)

**Leatherworking:**
- Used for Leather equipment
- Creates: Armor, Boots, Gloves, Helmet, Mainhand (leather), Offhand (leather), Belt (leather)

**Tailoring:**
- Used for Cloth equipment
- Creates: Armor, Boots, Gloves, Helmet, Mainhand (cloth), Offhand (cloth), Belt (cloth)

**Jewelry:**
- Used for accessories (Bone and Metal)
- Creates: Rings (bone and metal), Necklaces (bone and metal)

**Carpentry:**
- Used for wooden items and ranged weapons
- Creates: Staffs (for mages), Bows, Crossbows, Decorative items (for houses), Chests/Containers, Furniture
- Uses: Wood and Board resources

**Alchemy:**
- Used for potions and elixirs
- Creates: Potions, Elixirs, Consumables
- Uses: Herbs and other alchemical materials

**Cooking:**
- Used for food items
- Creates: Food items (system consumables)
- Uses: Food ingredients and resources

### Skill Requirements

**Minimum Requirements:**
- Each recipe has a minimum skill requirement
- Example: Iron Sword requires 10 Blacksmithing
- Below minimum skill: Item is always Common quality

**Maximum Requirements:**
- Maximum skill requirement for any equipment: 100
- Higher quality resources increase skill requirement
- Cap: 10% above minimum requirement for XP gain
- Example: Item with 10 minimum can go up to 11 for XP (10% cap)

**Resource Quality Scaling:**
- Same item can be made with different resource qualities
- Higher quality resources = higher skill requirement
- Higher quality resources = more XP gain
- Higher quality resources = better rarity chances

**Example Scaling:**
```
Iron Sword (Base):
- Minimum Skill: 10 Blacksmithing
- Resource: Iron Ingot
- XP Gain: Base

Steel Sword:
- Minimum Skill: 15 Blacksmithing
- Resource: Steel Ingot
- XP Gain: +25% (within 10% cap)

Gold Sword:
- Minimum Skill: 20 Blacksmithing
- Resource: Gold Ingot
- XP Gain: +50% (within 10% cap)

Mithril Sword:
- Minimum Skill: 30 Blacksmithing
- Resource: Mithril Ingot
- XP Gain: +100% (within 10% cap)

Heavenly Sword:
- Minimum Skill: 50 Blacksmithing
- Resource: Heavenly Ingot
- XP Gain: +200% (capped at 10% above minimum = 11 skill requirement for XP)
```

## Crafting Process

### Basic Crafting Flow

1. **Select Recipe**: Player selects item recipe
2. **Check Resources**: System checks if player has required resources
3. **Check Skill**: System checks if player meets skill requirement
4. **Pay Gold**: Player pays crafting gold cost
5. **Use Tool**: Player uses crafting tool (optional, specialized tools cost more)
6. **Create Item**: Item is created (never fails)
7. **Determine Quality**: System determines item rarity based on skill and resources
8. **Grant XP**: Player receives XP based on skill requirement and resource quality

### Crafting Requirements

**Resource Requirements:**
- Player must have all required resources
- Resources are consumed on crafting
- Resources are listed in item recipe

**Skill Requirements:**
- Player must meet minimum skill requirement
- Below minimum: Item is always Common
- At or above minimum: Item can be higher rarity

**Gold Requirements:**
- All crafting requires gold payment
- Gold cost varies by item type and tier
- Gold is consumed on crafting

**Tool Requirements:**
- Basic tools: Standard crafting (lower costs)
- Specialized tools: Better rarity chances (higher costs)
- Tools have durability/usage costs

## Quality and Rarity System

### Rarity Determination

**Skill-Based Rarity:**
- Below minimum skill: Always Common
- At minimum skill: Base rarity chances
- Above minimum skill: Improved rarity chances
- Maximum skill (100): Best rarity chances

**Resource Quality Impact:**
- Higher quality resources improve rarity chances
- Better resources = better base chances
- Still RNG-based (lottery system)

**Tool Impact:**
- Specialized tools improve rarity chances
- Example: Normal Forge 1% Legendary → Dwarf Forge 5% Legendary
- Tools don't guarantee rarity, just improve chances

### Rarity Chances

**Base Rarity Chances (Minimum Skill):**
- Common: 70%
- Uncommon: 20%
- Rare: 8%
- Magic: 1.5%
- Legendary: 0.5%

**At Maximum Skill (100):**
- Common: 40%
- Uncommon: 30%
- Rare: 20%
- Magic: 8%
- Legendary: 2%

**With High Quality Resources:**
- Common: 35%
- Uncommon: 30%
- Rare: 22%
- Magic: 10%
- Legendary: 3%

**With Specialized Tool:**
- Common: 30%
- Uncommon: 30%
- Rare: 25%
- Magic: 12%
- Legendary: 5% (Dwarf Forge example)

**Combined (Max Skill + High Resources + Specialized Tool):**
- Common: 25%
- Uncommon: 30%
- Rare: 25%
- Magic: 15%
- Legendary: 5%

**Note:** Even with maximum skill, high quality resources, and specialized tools, rarity is still RNG-based (lottery system).

## Resource System

### Resource Types

**Metal Resources (Ores and Ingots):**
- **Ores**: Iron Ore, Copper Ore, Silver Ore, Gold Ore, Dark Ore, Mithril Ore, Heavenly Ore
- **Ingots**: Copper Ingot, Iron Ingot, Steel Ingot, Silver Ingot, Gold Ingot, Dark Ingot, Dark Steel Ingot, Dwarf Metal Ingot, Mithril Ingot, Heavenly Ingot
- Used for: Metal equipment (Blacksmithing)

**Leather Resources (Hides and Leather):**
- **Hides**: Hides, Scaled Hides, Spined Hides, Demonic Hides, Darkness Hides, Barbed Hides, Dragon Hides, Divine Hides
- **Leather**: Leather, Hard Leather, Scaled Leather, Hard Scaled Leather, Spinned Leather, Iron Scaled Leather, Barbed Leather, Demonic Leather, Darkness Leather, Dragon Leather, Divine Leather
- Used for: Leather equipment (Leatherworking)

**Wood Resources (Woods and Boards):**
- **Woods**: Wood, Ipe Wood, Oak Wood, Maple Wood, Magic Oak Wood, Elven Wood, Ebano Wood, Rare Elven Wood, White Maple Wood
- **Boards**: Wood Board, Ipe Board, Oak Board, Maple Board, Magic Oak Board, Elven Board, Ebano Board, Rare Elven Board, White Maple Board
- Used for: Cloth equipment (Tailoring), Carpentry items (staffs, bows, crossbows, furniture, chests)

**Bone/Metal Resources (Accessories):**
- **Bone**: Crafted from bones (various bone types)
- **Metal**: Uses same metal ingots as armor/weapons
- Used for: Rings and Necklaces (Jewelry)

**Special Resources:**
- **Scales**: Drake Scale, Dragon Scale, Ancient Dragon Scale
- **Enchantment Materials**: Fire Essence, Cold Essence, Light Essence, Dark Essence, Elemental Dust, Magic Dust, Magic Essence (used for enchantments, not equipment crafting)
- **Herbs**: Various herbs and plants (used for Alchemy - potions and elixirs)
  - Examples: Black Plant With Thorns, Black Mushroom, Blood Berry, Blue Flower, Demon Mushroom, Dill, Edgy Root, Fire Flower, Garlic, Leaves, Mana Mushroom, Oil Plant, Red And Black Leaves, Red Fruit, Root, Rose, Rucola, Sulfurous Ash, White Flower, Wildrose, Yellow Flower, Yellow Fruit
- **Food Ingredients**: Various ingredients (used for Cooking - food items)

### Resource Quality Tiers

**Metal Resource Tiers:**

**Tier 1 (Basic):**
- Iron Ingot, Copper Ingot
- Minimum skill requirement
- Base XP gain
- Base rarity chances

**Tier 2 (Improved):**
- Steel Ingot, Silver Ingot
- +5 skill requirement
- +25% XP gain
- +10% rarity chance improvement

**Tier 3 (Superior):**
- Gold Ingot, Dark Ingot
- +10 skill requirement
- +50% XP gain
- +20% rarity chance improvement

**Tier 4 (Exceptional):**
- Dark Steel Ingot, Dwarf Metal Ingot, Mithril Ingot
- +20 skill requirement
- +100% XP gain
- +30% rarity chance improvement

**Tier 5 (Masterwork):**
- Heavenly Ingot
- +40 skill requirement (up to max 100)
- +200% XP gain (capped at 10% above minimum)
- +40% rarity chance improvement

**Leather Resource Tiers:**

**Tier 1 (Basic):**
- Leather, Hard Leather
- Minimum skill requirement
- Base XP gain
- Base rarity chances

**Tier 2 (Improved):**
- Scaled Leather, Hard Scaled Leather, Spinned Leather
- +5 skill requirement
- +25% XP gain
- +10% rarity chance improvement

**Tier 3 (Superior):**
- Iron Scaled Leather, Barbed Leather
- +10 skill requirement
- +50% XP gain
- +20% rarity chance improvement

**Tier 4 (Exceptional):**
- Demonic Leather, Darkness Leather, Dragon Leather
- +20 skill requirement
- +100% XP gain
- +30% rarity chance improvement

**Tier 5 (Masterwork):**
- Divine Leather
- +40 skill requirement (up to max 100)
- +200% XP gain (capped at 10% above minimum)
- +40% rarity chance improvement

**Wood/Board Resource Tiers (for Cloth):**

**Tier 1 (Basic):**
- Wood Board
- Minimum skill requirement
- Base XP gain
- Base rarity chances

**Tier 2 (Improved):**
- Ipe Board, Oak Board
- +5 skill requirement
- +25% XP gain
- +10% rarity chance improvement

**Tier 3 (Superior):**
- Maple Board, Magic Oak Board
- +10 skill requirement
- +50% XP gain
- +20% rarity chance improvement

**Tier 4 (Exceptional):**
- Elven Board, Ebano Board
- +20 skill requirement
- +100% XP gain
- +30% rarity chance improvement

**Tier 5 (Masterwork):**
- Rare Elven Board, White Maple Board
- +40 skill requirement (up to max 100)
- +200% XP gain (capped at 10% above minimum)
- +40% rarity chance improvement

### Same Item, Different Resources

**Example: Iron Sword Recipe:**
- Can be made with: Iron Ingot, Steel Ingot, Gold Ingot, Mithril Ingot, Heavenly Ingot
- Each resource has different skill requirement
- Each resource has different XP gain
- Each resource has different rarity chances
- Same recipe, different outcomes

**Recipe Flexibility:**
- Recipes accept multiple resource types
- Player chooses which resources to use
- System calculates skill requirement based on resources
- System calculates XP and rarity based on resources

## XP Gain System

### XP Calculation

**Base XP Formula:**
```
XP Gain = Base XP × (1 + Resource Quality Multiplier) × Skill Requirement Multiplier
```

**Resource Quality Multiplier:**
- Tier 1: 1.0x (base)
- Tier 2: 1.25x (+25%)
- Tier 3: 1.5x (+50%)
- Tier 4: 2.0x (+100%)
- Tier 5: 3.0x (+200%, capped at 10% above minimum)

**Skill Requirement Multiplier:**
- Based on skill requirement vs player skill
- Higher requirement = more XP
- Capped at 10% above minimum for XP purposes

**XP Cap:**
- Maximum XP gain is 10% above minimum skill requirement
- Example: Item with 10 minimum can give XP up to 11 skill equivalent
- Prevents excessive XP farming with high-tier resources

**Example XP Calculation:**
```
Iron Sword (Base):
- Minimum Skill: 10
- Resource: Iron Ingot (Tier 1)
- Base XP: 100
- XP Gain: 100 × 1.0 = 100 XP

Steel Sword:
- Minimum Skill: 15
- Resource: Steel Ingot (Tier 2)
- Base XP: 100
- XP Gain: 100 × 1.25 = 125 XP

Gold Sword:
- Minimum Skill: 20
- Resource: Gold Ingot (Tier 3)
- Base XP: 100
- XP Gain: 100 × 1.5 = 150 XP

Mithril Sword:
- Minimum Skill: 30
- Resource: Mithril Ingot (Tier 4)
- Base XP: 100
- XP Gain: 100 × 2.0 = 200 XP

Heavenly Sword:
- Minimum Skill: 50
- Resource: Heavenly Ingot (Tier 5)
- Base XP: 100
- XP Gain: 100 × 3.0 = 300 XP (capped at 10% above minimum = 11 skill equivalent)
```

## Gold Cost System

### Crafting Costs

**Base Gold Cost:**
- Varies by item type and tier
- Equipment costs more than consumables
- Higher tier items cost more

**Cost Factors:**
- Item type (armor, weapon, accessory)
- Item tier
- Resource quality (higher quality = higher cost)
- Tool usage (specialized tools cost more)

**Example Costs (varies by resource quality):**
- Iron Sword (Iron Ingot): 50 gold
- Steel Sword (Steel Ingot): 75 gold
- Gold Sword (Gold Ingot): 100 gold
- Mithril Sword (Mithril Ingot): 200 gold
- Heavenly Sword (Heavenly Ingot): 500 gold

**Tool Usage Costs:**
- Normal Forge: Base cost
- Dwarf Forge: +50% cost
- Master Forge: +100% cost

## Specialized Tools

### Tool Types

**Blacksmithing Tools:**
- Normal Forge: Base rarity chances
- Dwarf Forge: +4% Legendary chance (1% → 5%)
- Master Forge: +6% Legendary chance (1% → 7%)

**Leatherworking Tools:**
- Normal Workbench: Base rarity chances
- Master Workbench: +4% Legendary chance
- Artisan Workbench: +6% Legendary chance

**Tailoring Tools:**
- Normal Loom: Base rarity chances
- Master Loom: +4% Legendary chance
- Artisan Loom: +6% Legendary chance

**Jewelry Tools:**
- Normal Jewelry Bench: Base rarity chances
- Master Jewelry Bench: +4% Legendary chance
- Artisan Jewelry Bench: +6% Legendary chance

**Carpentry Tools:**
- Normal Workbench: Base rarity chances
- Master Workbench: +4% Legendary chance
- Artisan Workbench: +6% Legendary chance

**Alchemy Tools:**
- Normal Alchemy Table: Base rarity chances
- Master Alchemy Table: +4% Legendary chance (for high-tier elixirs)
- Artisan Alchemy Table: +6% Legendary chance

**Cooking Tools:**
- Normal Cooking Pot: Base quality
- Master Cooking Pot: Improved food quality
- Artisan Cooking Pot: Best food quality

### Tool Benefits

**Rarity Improvements:**
- Specialized tools improve all rarity chances
- Focus on Legendary chance improvement
- Still RNG-based (lottery)

**Cost Increases:**
- Specialized tools cost more to use
- Higher tier tools = higher costs
- Cost is per crafting use

**Tool Durability:**
- Tools may have durability/usage limits
- Higher tier tools may have more durability
- Tools may need repair or replacement

## Destruction and Recovery System

### Item Destruction

**Destruction Process:**
1. Player selects item to destroy
2. System checks item components
3. System calculates recovery percentage
4. Resources are partially recovered
5. Item is destroyed

**Recovery Percentage:**
- Base recovery: 50% of resources
- With skill: Up to 75% recovery
- With specialized tool: Up to 90% recovery
- Never 100% recovery (always some loss)

**Recovery Factors:**
- Player skill level
- Tool used for destruction
- Item durability/condition
- Random factor (RNG)

### Component Tracking

**Component Storage:**
- Every crafted item stores its component resources
- Components are tracked in item data
- Components are used for destruction recovery

**Component Structure:**
```rust
pub struct ItemComponents {
    pub resources: Vec<ResourceComponent>,
    pub quantities: Vec<u16>,
    pub resource_types: Vec<ResourceType>,
}

pub struct ResourceComponent {
    pub resource_id: ResourceId,
    pub quantity: u16,
    pub resource_type: ResourceType,
    pub quality_tier: u8,
}
```

**Component Recovery:**
- On destruction, components are recovered
- Recovery percentage applied to each component
- Recovered resources added to player inventory
- Lost resources are permanently destroyed

## Recipe System

### Recipe Structure

**Recipe Components:**
- Required resources (list)
- Required quantities (per resource)
- Minimum skill requirement
- Gold cost
- Item type and tier
- Acceptable resource alternatives

**Recipe Example:**
```
Iron Sword Recipe:
- Resources: Iron Ingot × 5
- Alternatives: Steel Ingot × 5, Gold Ingot × 5, Mithril Ingot × 5, Heavenly Ingot × 5
- Minimum Skill: 10 Blacksmithing (increases with resource quality)
- Gold Cost: 50 (increases with resource quality)
- Item Type: Mainhand Weapon (Metal)
- Tier: T1
- Skill Scaling: Iron (10), Steel (15), Gold (20), Mithril (30), Heavenly (50)
```

**Recipe Flexibility:**
- Recipes accept multiple resource types
- Same recipe, different resources = different outcomes
- Player chooses which resources to use
- System adapts skill requirement based on resources

### Recipe Discovery

**Recipe Sources:**
- Starting recipes (basic items)
- Quest rewards
- NPC vendors
- Discovery through experimentation
- Rare recipe drops

**Recipe Learning:**
- Recipes are learned permanently
- Once learned, always available
- Recipes can be shared/traded (optional)
- Recipe book tracks all learned recipes

## Rust Implementation Considerations

### Data Structures

**Recipe Component:**
```rust
#[derive(Clone)]
pub struct Recipe {
    pub recipe_id: u32,
    pub item_type: ItemType,
    pub item_tier: u8,
    pub minimum_skill: u16,
    pub base_gold_cost: u32,
    pub required_resources: Vec<ResourceRequirement>,
    pub resource_alternatives: Vec<Vec<ResourceRequirement>>,
}

#[derive(Clone)]
pub struct ResourceRequirement {
    pub resource_id: ResourceId,
    pub quantity: u16,
    pub resource_type: ResourceType,
    pub quality_tier: u8,
}

#[derive(Clone)]
pub struct CraftingResult {
    pub item_id: ItemId,
    pub rarity: ItemRarity,
    pub xp_gained: u32,
    pub gold_spent: u32,
    pub components: ItemComponents,
}
```

**Crafting Component:**
```rust
#[derive(Component)]
pub struct CraftingSkills {
    pub blacksmithing: u16,
    pub leatherworking: u16,
    pub tailoring: u16,
    pub jewelry: u16,
}

#[derive(Component)]
pub struct ItemComponents {
    pub resources: Vec<ResourceComponent>,
    pub crafting_skill: CraftingSkillType,
    pub skill_level_used: u16,
    pub tool_used: Option<ToolId>,
}

#[derive(Clone, Copy)]
pub struct ResourceComponent {
    pub resource_id: ResourceId,
    pub quantity: u16,
    pub resource_type: ResourceType,
    pub quality_tier: u8,
}
```

### Crafting Functions

**Crafting Function:**
```rust
pub fn craft_item(
    recipe: &Recipe,
    resources: &[ResourceRequirement],
    player_skill: u16,
    tool: Option<&Tool>,
) -> Result<CraftingResult, String> {
    // Check resources
    if !has_required_resources(resources) {
        return Err("Missing required resources".to_string());
    }
    
    // Calculate skill requirement based on resources
    let skill_requirement = calculate_skill_requirement(recipe, resources);
    
    // Check skill
    if player_skill < skill_requirement {
        // Below requirement: item is always Common
        return Ok(create_common_item(recipe, resources, tool));
    }
    
    // Pay gold cost
    let gold_cost = calculate_gold_cost(recipe, resources, tool);
    if !pay_gold(gold_cost) {
        return Err("Insufficient gold".to_string());
    }
    
    // Determine rarity
    let rarity = determine_rarity(player_skill, resources, tool);
    
    // Create item
    let item = create_item(recipe, resources, rarity);
    
    // Calculate XP
    let xp_gained = calculate_xp_gain(recipe, resources, skill_requirement);
    
    // Store components
    let components = store_components(resources, player_skill, tool);
    item.set_components(components);
    
    // Consume resources
    consume_resources(resources);
    
    Ok(CraftingResult {
        item_id: item.id,
        rarity,
        xp_gained,
        gold_spent: gold_cost,
        components,
    })
}

fn determine_rarity(
    player_skill: u16,
    resources: &[ResourceRequirement],
    tool: Option<&Tool>,
) -> ItemRarity {
    let base_chances = get_base_rarity_chances(player_skill);
    let resource_bonus = calculate_resource_rarity_bonus(resources);
    let tool_bonus = tool.map(|t| t.rarity_bonus()).unwrap_or(0.0);
    
    let final_chances = apply_bonuses(base_chances, resource_bonus, tool_bonus);
    
    roll_rarity(final_chances)
}

fn calculate_xp_gain(
    recipe: &Recipe,
    resources: &[ResourceRequirement],
    skill_requirement: u16,
) -> u32 {
    let base_xp = recipe.base_xp;
    let resource_multiplier = calculate_resource_xp_multiplier(resources);
    let skill_multiplier = calculate_skill_xp_multiplier(skill_requirement, recipe.minimum_skill);
    
    // Cap at 10% above minimum
    let capped_multiplier = skill_multiplier.min(1.1); // 10% cap
    
    (base_xp as f32 * resource_multiplier * capped_multiplier) as u32
}
```

**Destruction Function:**
```rust
pub fn destroy_item(
    item: &Item,
    player_skill: u16,
    tool: Option<&Tool>,
) -> DestructionResult {
    let components = item.get_components();
    
    // Calculate recovery percentage
    let base_recovery = 0.5; // 50%
    let skill_bonus = (player_skill as f32 / 100.0) * 0.25; // Up to +25%
    let tool_bonus = tool.map(|t| t.recovery_bonus()).unwrap_or(0.0); // Up to +15%
    
    let recovery_percentage = (base_recovery + skill_bonus + tool_bonus).min(0.9); // Max 90%
    
    // Recover resources
    let recovered_resources = components.resources.iter().map(|comp| {
        let recovered_quantity = (comp.quantity as f32 * recovery_percentage) as u16;
        RecoveredResource {
            resource_id: comp.resource_id,
            quantity: recovered_quantity,
            resource_type: comp.resource_type,
            quality_tier: comp.quality_tier,
        }
    }).collect();
    
    // Destroy item
    item.destroy();
    
    DestructionResult {
        recovered_resources,
        recovery_percentage,
        resources_lost: calculate_lost_resources(components, recovery_percentage),
    }
}
```

## Integration with Other Systems

### Skill System

**Integration:**
- Crafting skills are part of skill system
- Skills gain XP through crafting
- Skill level affects crafting quality
- See [SKILLS.md](../player/SKILLS.md) for skill system details

### Item System

**Integration:**
- Crafted items follow item system rules
- Items have components stored for destruction
- Items can be enchanted, have runes/cards
- See [ITEMS.md](./ITEMS.md) for item system details

### Resource System

**Integration:**
- Resources are consumed in crafting
- Resources are recovered on destruction
- Resource quality affects crafting outcomes
- Resources can be gathered or purchased

### Economy System

**Integration:**
- Crafting costs gold
- Crafted items have value
- Specialized tools cost more
- Crafting creates economic activity

## Summary

The crafting system:

- Never fails - players can always create items with resources
- Skill-based quality - low skill = common, high skill = better chances
- Resource variety - same item with different resources
- Progressive XP - higher quality resources give more XP (capped at 10% above minimum)
- Rarity lottery - chance-based system for uncommon, rare, magic, legendary
- Gold costs - all crafting requires gold payment
- Specialized tools - improve rarity chances but cost more
- Destruction system - items can be destroyed to recover partial resources
- Component tracking - items store their component resources
- Recipe system - all items have recipes with required resources
- Maximum skill requirement: 100 for any equipment
- Resource quality scaling: Up to 10% above minimum for XP gain

This system creates depth in crafting while maintaining accessibility - players can always craft items, but skill and resource quality determine the final outcome. The lottery system ensures that even with maximum skill, there's always excitement and value in high-quality crafted items.

