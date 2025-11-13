# Enchantment System Documentation

## Overview

The enchantment system allows players to enhance equipment and weapons with elemental properties through magical rituals. Enchanted items gain powerful bonuses, visual effects (VFX particles), and color changes, making them significantly more valuable than unenchanted items. The system requires specific materials, skills, and gold, with success rates and costs scaling based on enchantment level and player skill.

**Note:** All equipment and weapons can be enchanted, but each item can only have ONE enchantment at a time. Enchanted items can be disenchanted or upgraded to higher levels.

## Key Features

- **Single Enchantment Limit**: Each item can only have one enchantment
- **Visual Effects**: Enchanted items display VFX particles and color changes
- **Elemental Properties**: Enchantments enhance elemental damage, resistance, and related properties
- **Skill-Based**: Enchantment success and effectiveness scale with Enchanting skill level
- **Material Requirements**: Requires elemental essences and star dust
- **Progressive Levels**: Six enchantment levels (+1 to +6) with increasing bonuses
- **Failure Risk**: Chance to fail, losing partial resources and gold
- **High Value**: Enchanted items are significantly more valuable than unenchanted items
- **Upgrade System**: Existing enchantments can be upgraded to higher levels
- **Disenchantment**: Enchantments can be removed with specific items

## Enchantment Types

### Fire Enchantment
- **Essence Required**: Fire Essence
- **Visual Effect**: Red/orange particle effects, warm glow, possible red tint
- **Properties Enhanced**: Fire damage, fire resistance, fire-related attributes
- **Example**: Item with +20 Fire Resistance → +40 Fire Resistance at +2 level

### Cold Enchantment
- **Essence Required**: Cold Essence
- **Visual Effect**: Blue/white particle effects, frosty glow, possible blue tint
- **Properties Enhanced**: Cold damage, cold resistance, cold-related attributes
- **Example**: Item with +15 Cold Damage → +30 Cold Damage at +2 level

### Poison Enchantment
- **Essence Required**: Poison Essence
- **Visual Effect**: Green/purple particle effects, toxic glow, possible green tint
- **Properties Enhanced**: Poison damage, poison resistance, poison-related attributes
- **Example**: Item with +25 Poison Damage → +50 Poison Damage at +2 level

### Energy Enchantment
- **Essence Required**: Energy Essence
- **Visual Effect**: Yellow/white particle effects, electric sparks, possible yellow tint
- **Properties Enhanced**: Energy damage, energy resistance, energy-related attributes
- **Example**: Item with +18 Energy Damage → +36 Energy Damage at +2 level

### Light Enchantment
- **Essence Required**: Light Essence
- **Visual Effect**: White/golden particle effects, holy glow, possible golden tint
- **Properties Enhanced**: Light damage, light resistance, light-related attributes
- **Example**: Item with +22 Light Resistance → +44 Light Resistance at +2 level

### Dark Enchantment
- **Essence Required**: Dark Essence
- **Visual Effect**: Black/purple particle effects, shadowy aura, possible dark tint
- **Properties Enhanced**: Dark damage, dark resistance, dark-related attributes
- **Example**: Item with +20 Dark Damage → +40 Dark Damage at +2 level

## Enchantment Levels

### Level Progression

Enchantment levels range from +1 to +6, with each level providing increased effectiveness:

**+1 Enchantment:**
- **Effectiveness**: 20% of base property value
- **Skill Requirement**: Enchanting Skill Level 10
- **Success Rate**: 95% (if skill requirement met)
- **Material Cost**: 1x Elemental Essence, 1x Star Dust
- **Gold Cost**: 100 gold

**+2 Enchantment:**
- **Effectiveness**: 35% of base property value
- **Skill Requirement**: Enchanting Skill Level 25
- **Success Rate**: 90% (if skill requirement met)
- **Material Cost**: 2x Elemental Essence, 2x Star Dust
- **Gold Cost**: 300 gold

**+3 Enchantment:**
- **Effectiveness**: 50% of base property value
- **Skill Requirement**: Enchanting Skill Level 40
- **Success Rate**: 85% (if skill requirement met)
- **Material Cost**: 3x Elemental Essence, 3x Star Dust
- **Gold Cost**: 600 gold

**+4 Enchantment:**
- **Effectiveness**: 70% of base property value
- **Skill Requirement**: Enchanting Skill Level 60
- **Success Rate**: 75% (if skill requirement met)
- **Material Cost**: 5x Elemental Essence, 5x Star Dust
- **Gold Cost**: 1,200 gold

**+5 Enchantment:**
- **Effectiveness**: 90% of base property value
- **Skill Requirement**: Enchanting Skill Level 80
- **Success Rate**: 60% (even at max skill)
- **Material Cost**: 8x Elemental Essence, 8x Star Dust
- **Gold Cost**: 2,500 gold

**+6 Enchantment:**
- **Effectiveness**: 100% of base property value (doubles the property)
- **Skill Requirement**: Enchanting Skill Level 100
- **Success Rate**: 40% (even at max skill)
- **Material Cost**: 12x Elemental Essence, 12x Star Dust
- **Gold Cost**: 5,000 gold

## Enchantment Mechanics

### Property Multiplication

Enchantments multiply related elemental properties on the item:

**Formula:**
```
Enchanted Property Value = Base Property Value * (1 + Enchantment Effectiveness)
```

**Examples:**
- Item with +20 Fire Resistance, +2 Fire Enchantment (35%):
  - Enchanted Value: 20 * (1 + 0.35) = 20 * 1.35 = 27 Fire Resistance
  
- Item with +30 Fire Damage, +4 Fire Enchantment (70%):
  - Enchanted Value: 30 * (1 + 0.70) = 30 * 1.70 = 51 Fire Damage
  
- Item with +25 Cold Resistance, +6 Cold Enchantment (100%):
  - Enchanted Value: 25 * (1 + 1.00) = 25 * 2.00 = 50 Cold Resistance (doubled)

### Related Properties

Each enchantment type enhances multiple related properties:

**Fire Enchantment enhances:**
- Fire Damage
- Fire Resistance
- Fire-related attributes (if any)

**Cold Enchantment enhances:**
- Cold Damage
- Cold Resistance
- Cold-related attributes (if any)

**Poison Enchantment enhances:**
- Poison Damage
- Poison Resistance
- Poison-related attributes (if any)

**Energy Enchantment enhances:**
- Energy Damage
- Energy Resistance
- Energy-related attributes (if any)

**Light Enchantment enhances:**
- Light Damage
- Light Resistance
- Light-related attributes (if any)

**Dark Enchantment enhances:**
- Dark Damage
- Dark Resistance
- Dark-related attributes (if any)

### Multiple Properties

If an item has multiple properties of the same element, all are enhanced:

**Example:**
- Item has: +20 Fire Damage, +15 Fire Resistance
- Enchanted with +3 Fire Enchantment (50%):
  - Fire Damage: 20 * 1.50 = 30
  - Fire Resistance: 15 * 1.50 = 22.5 (rounded to 23)

## Enchantment Process

### Requirements

**To enchant an item, you need:**
1. **Item to Enchant**: Equipment or weapon (must not already be enchanted)
2. **Enchanting Skill**: Appropriate skill level for desired enchantment level
3. **Elemental Essence**: Type matching desired enchantment
4. **Star Dust**: Required for all enchantments
5. **Gold**: Payment for the enchantment ritual

### Enchantment Steps

**Step 1: Preparation**
- Player selects item to enchant
- Player selects enchantment type (Fire, Cold, Poison, Energy, Light, Dark)
- Player selects enchantment level (+1 to +6)
- System checks if player has required skill level
- System checks if player has required materials and gold

**Step 2: Skill Check**
- System calculates success rate based on skill level
- If skill < requirement: Success rate reduced by 5% per level below requirement
- If skill >= requirement: Uses base success rate for level

**Step 3: Resource Consumption**
- Player's materials and gold are consumed immediately
- Resources are locked until enchantment completes (success or failure)

**Step 4: Enchantment Roll**
- System rolls for success based on calculated success rate
- If successful: Enchantment is applied
- If failed: Partial resources are lost (see Failure Mechanics)

**Step 5: Visual Application**
- If successful: VFX particles and color changes are applied to item
- Item is marked as enchanted
- Enchantment properties are calculated and applied

### Success Rate Calculation

**Formula:**
```rust
fn calculate_success_rate(
    desired_level: u8,
    player_skill: u16,
    required_skill: u16,
) -> f32 {
    let base_success_rate = match desired_level {
        1 => 0.95,
        2 => 0.90,
        3 => 0.85,
        4 => 0.75,
        5 => 0.60,
        6 => 0.40,
        _ => 0.0,
    };
    
    if player_skill < required_skill {
        let skill_deficit = required_skill - player_skill;
        let penalty = skill_deficit as f32 * 0.05; // 5% per level below
        (base_success_rate - penalty).max(0.0)
    } else {
        base_success_rate
    }
}
```

**Examples:**
- +1 Enchantment, Skill 10 (required 10): 95% success rate
- +2 Enchantment, Skill 20 (required 25): 90% - (5 * 0.05) = 65% success rate
- +5 Enchantment, Skill 100 (required 80): 60% success rate (max, cannot exceed base rate)

## Failure Mechanics

### Failure Consequences

When an enchantment fails, the player loses resources:

**Resource Loss on Failure:**
- **+1 to +3**: Lose 50% of materials and gold
- **+4**: Lose 60% of materials and gold
- **+5**: Lose 70% of materials and gold
- **+6**: Lose 80% of materials and gold

**Example:**
- Attempting +5 Fire Enchantment (8x Fire Essence, 8x Star Dust, 2,500 gold)
- Failure: Lose 5.6x Fire Essence (rounded to 6), 5.6x Star Dust (rounded to 6), 1,750 gold
- Player receives: 2x Fire Essence, 2x Star Dust, 750 gold refund

### Failure Prevention

**Strategies to reduce failure risk:**
- Ensure skill level meets or exceeds requirement
- Use higher skill levels for better success rates
- Consider enchanting at lower levels first, then upgrading
- Save materials for when skill is sufficient

## Upgrade System

### Upgrading Existing Enchantments

Players can upgrade existing enchantments to higher levels:

**Requirements:**
- Item must already have an enchantment
- Player must have skill level for new enchantment level
- Player must have required materials and gold
- Enchantment type must match (cannot change type)

**Upgrade Process:**
1. Player selects enchanted item
2. Player selects upgrade (next level only, e.g., +2 → +3)
3. System calculates difference in material/gold costs
4. Player pays difference in costs
5. Success rate uses new level's base rate (not reduced for upgrade)

**Upgrade Costs:**
- Upgrade from +1 to +2: Pay difference (2-1=1 essence, 2-1=1 dust, 300-100=200 gold)
- Upgrade from +2 to +3: Pay difference (3-2=1 essence, 3-2=1 dust, 600-300=300 gold)
- Upgrade from +3 to +4: Pay difference (5-3=2 essence, 5-3=2 dust, 1200-600=600 gold)
- Upgrade from +4 to +5: Pay difference (8-5=3 essence, 8-5=3 dust, 2500-1200=1300 gold)
- Upgrade from +5 to +6: Pay difference (12-8=4 essence, 12-8=4 dust, 5000-2500=2500 gold)

**Upgrade Success Rate:**
- Uses the new level's base success rate
- No penalty for upgrading (treats as new enchantment at new level)

## Disenchantment

### Removing Enchantments

Players can remove enchantments from items using a Disenchantment Scroll:

**Requirements:**
- Item must be enchanted
- Disenchantment Scroll (consumed on use)
- Gold cost: 10% of original enchantment cost

**Disenchantment Process:**
1. Player uses Disenchantment Scroll on enchanted item
2. System removes enchantment
3. Item returns to base state (no enchantment)
4. Visual effects are removed
5. Player receives partial material refund (see below)

**Material Refund:**
- **+1 to +3**: Refund 30% of materials used
- **+4**: Refund 25% of materials used
- **+5**: Refund 20% of materials used
- **+6**: Refund 15% of materials used

**Example:**
- Disenchanting +5 Fire Enchantment (used 8x Fire Essence, 8x Star Dust)
- Refund: 1.6x Fire Essence (rounded to 2), 1.6x Star Dust (rounded to 2)
- Gold cost: 250 gold (10% of 2,500)

## Visual Effects

### Particle Effects

Each enchantment type has unique particle effects:

**Fire Enchantment:**
- Red/orange embers floating around item
- Flickering flames at edges
- Heat distortion effect

**Cold Enchantment:**
- Blue/white snowflakes floating around item
- Frost crystals forming on surface
- Cold mist effect

**Poison Enchantment:**
- Green/purple toxic bubbles
- Poisonous gas clouds
- Corrosive dripping effect

**Energy Enchantment:**
- Yellow/white electric sparks
- Lightning arcs around item
- Static electricity effect

**Light Enchantment:**
- White/golden light rays
- Holy aura surrounding item
- Divine glow effect

**Dark Enchantment:**
- Black/purple shadow tendrils
- Dark mist swirling around item
- Corrupting aura effect

### Color Changes

Enchanted items gain color tints matching their enchantment:

**Color Intensity:**
- **+1 to +2**: Subtle tint (10-20% color overlay)
- **+3 to +4**: Moderate tint (30-40% color overlay)
- **+5 to +6**: Strong tint (50-60% color overlay)

**Color Application:**
- Applied as overlay on item texture
- Preserves item's base appearance
- Intensity scales with enchantment level

## Value Impact

### Enchanted Item Value

Enchanted items are significantly more valuable:

**Value Formula:**
```
Enchanted Item Value = Base Item Value * (1 + Enchantment Value Multiplier)
```

**Enchantment Value Multipliers:**
- **+1**: 1.5x base value
- **+2**: 2.0x base value
- **+3**: 2.5x base value
- **+4**: 3.0x base value
- **+5**: 4.0x base value
- **+6**: 5.0x base value

**Additional Value Factors:**
- Visual effects add 10-20% value
- Perfect enchantment rolls add 5-10% value
- Rare enchantment combinations add 15-25% value

**Example:**
- Base item value: 1,000 gold
- +4 Fire Enchantment: 1,000 * 3.0 = 3,000 gold
- With visual effects: 3,000 * 1.15 = 3,450 gold

## Materials

### Elemental Essences

**Fire Essence:**
- Obtained from: Fire elementals, volcanic areas, fire-based crafting
- Rarity: Uncommon
- Used for: Fire enchantments

**Cold Essence:**
- Obtained from: Ice elementals, frozen areas, cold-based crafting
- Rarity: Uncommon
- Used for: Cold enchantments

**Poison Essence:**
- Obtained from: Poisonous creatures, toxic areas, poison-based crafting
- Rarity: Uncommon
- Used for: Poison enchantments

**Energy Essence:**
- Obtained from: Lightning elementals, storm areas, energy-based crafting
- Rarity: Uncommon
- Used for: Energy enchantments

**Light Essence:**
- Obtained from: Holy creatures, sacred areas, light-based crafting
- Rarity: Rare
- Used for: Light enchantments

**Dark Essence:**
- Obtained from: Shadow creatures, dark areas, dark-based crafting
- Rarity: Rare
- Used for: Dark enchantments

### Star Dust

**Star Dust:**
- Obtained from: Meteorites, star fragments, celestial crafting
- Rarity: Uncommon
- Used for: All enchantments (universal material)
- Required for: Every enchantment level

## Rust Implementation Considerations

### Data Structures

**Enchantment Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct Enchantment {
    pub enchantment_type: EnchantmentType,
    pub level: u8, // +1 to +6
    pub effectiveness: f32, // 0.20 to 1.00
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum EnchantmentType {
    Fire,
    Cold,
    Poison,
    Energy,
    Light,
    Dark,
}

impl Enchantment {
    pub fn new(enchantment_type: EnchantmentType, level: u8) -> Self {
        let effectiveness = match level {
            1 => 0.20,
            2 => 0.35,
            3 => 0.50,
            4 => 0.70,
            5 => 0.90,
            6 => 1.00,
            _ => 0.0,
        };
        
        Enchantment {
            enchantment_type,
            level,
            effectiveness,
        }
    }
    
    pub fn calculate_enhanced_value(&self, base_value: f32) -> f32 {
        base_value * (1.0 + self.effectiveness)
    }
}
```

**Enchantment Materials:**
```rust
#[derive(Clone, Copy)]
pub struct EnchantmentMaterials {
    pub fire_essence: u16,
    pub cold_essence: u16,
    pub poison_essence: u16,
    pub energy_essence: u16,
    pub light_essence: u16,
    pub dark_essence: u16,
    pub star_dust: u16,
}

pub fn get_material_cost(level: u8) -> EnchantmentMaterials {
    match level {
        1 => EnchantmentMaterials { fire_essence: 1, cold_essence: 1, poison_essence: 1, energy_essence: 1, light_essence: 1, dark_essence: 1, star_dust: 1 },
        2 => EnchantmentMaterials { fire_essence: 2, cold_essence: 2, poison_essence: 2, energy_essence: 2, light_essence: 2, dark_essence: 2, star_dust: 2 },
        3 => EnchantmentMaterials { fire_essence: 3, cold_essence: 3, poison_essence: 3, energy_essence: 3, light_essence: 3, dark_essence: 3, star_dust: 3 },
        4 => EnchantmentMaterials { fire_essence: 5, cold_essence: 5, poison_essence: 5, energy_essence: 5, light_essence: 5, dark_essence: 5, star_dust: 5 },
        5 => EnchantmentMaterials { fire_essence: 8, cold_essence: 8, poison_essence: 8, energy_essence: 8, light_essence: 8, dark_essence: 8, star_dust: 8 },
        6 => EnchantmentMaterials { fire_essence: 12, cold_essence: 12, poison_essence: 12, energy_essence: 12, light_essence: 12, dark_essence: 12, star_dust: 12 },
        _ => EnchantmentMaterials { fire_essence: 0, cold_essence: 0, poison_essence: 0, energy_essence: 0, light_essence: 0, dark_essence: 0, star_dust: 0 },
    }
}
```

### Enchantment Application

**Enchantment Function:**
```rust
pub struct EnchantmentResult {
    pub success: bool,
    pub enchantment: Option<Enchantment>,
    pub materials_lost: EnchantmentMaterials,
    pub gold_lost: u32,
}

pub fn apply_enchantment(
    item: &mut Item,
    enchantment_type: EnchantmentType,
    level: u8,
    player_skill: u16,
    player_materials: &mut EnchantmentMaterials,
    player_gold: &mut u32,
) -> EnchantmentResult {
    // Check if item already has enchantment
    if item.enchantment.is_some() {
        return EnchantmentResult {
            success: false,
            enchantment: None,
            materials_lost: EnchantmentMaterials::default(),
            gold_lost: 0,
        };
    }
    
    // Get requirements
    let required_skill = get_required_skill(level);
    let material_cost = get_material_cost(level);
    let gold_cost = get_gold_cost(level);
    
    // Check if player has required materials
    let has_materials = match enchantment_type {
        EnchantmentType::Fire => player_materials.fire_essence >= material_cost.fire_essence && player_materials.star_dust >= material_cost.star_dust,
        EnchantmentType::Cold => player_materials.cold_essence >= material_cost.cold_essence && player_materials.star_dust >= material_cost.star_dust,
        EnchantmentType::Poison => player_materials.poison_essence >= material_cost.poison_essence && player_materials.star_dust >= material_cost.star_dust,
        EnchantmentType::Energy => player_materials.energy_essence >= material_cost.energy_essence && player_materials.star_dust >= material_cost.star_dust,
        EnchantmentType::Light => player_materials.light_essence >= material_cost.light_essence && player_materials.star_dust >= material_cost.star_dust,
        EnchantmentType::Dark => player_materials.dark_essence >= material_cost.dark_essence && player_materials.star_dust >= material_cost.star_dust,
    };
    
    if !has_materials || *player_gold < gold_cost {
        return EnchantmentResult {
            success: false,
            enchantment: None,
            materials_lost: EnchantmentMaterials::default(),
            gold_lost: 0,
        };
    }
    
    // Consume materials and gold
    match enchantment_type {
        EnchantmentType::Fire => player_materials.fire_essence -= material_cost.fire_essence,
        EnchantmentType::Cold => player_materials.cold_essence -= material_cost.cold_essence,
        EnchantmentType::Poison => player_materials.poison_essence -= material_cost.poison_essence,
        EnchantmentType::Energy => player_materials.energy_essence -= material_cost.energy_essence,
        EnchantmentType::Light => player_materials.light_essence -= material_cost.light_essence,
        EnchantmentType::Dark => player_materials.dark_essence -= material_cost.dark_essence,
    }
    player_materials.star_dust -= material_cost.star_dust;
    *player_gold -= gold_cost;
    
    // Calculate success rate
    let success_rate = calculate_success_rate(level, player_skill, required_skill);
    
    // Roll for success
    let roll = rand::random::<f32>();
    if roll < success_rate {
        // Success
        let enchantment = Enchantment::new(enchantment_type, level);
        item.enchantment = Some(enchantment);
        apply_enchantment_effects(item, &enchantment);
        
        EnchantmentResult {
            success: true,
            enchantment: Some(enchantment),
            materials_lost: EnchantmentMaterials::default(),
            gold_lost: 0,
        }
    } else {
        // Failure
        let loss_percentage = get_failure_loss_percentage(level);
        let materials_lost = calculate_material_loss(material_cost, loss_percentage);
        let gold_lost = (gold_cost as f32 * loss_percentage) as u32;
        
        // Refund partial materials and gold
        refund_partial_resources(player_materials, player_gold, material_cost, gold_cost, loss_percentage, enchantment_type);
        
        EnchantmentResult {
            success: false,
            enchantment: None,
            materials_lost,
            gold_lost,
        }
    }
}
```

### Property Enhancement

**Enhancement Application:**
```rust
pub fn apply_enchantment_effects(item: &mut Item, enchantment: &Enchantment) {
    match enchantment.enchantment_type {
        EnchantmentType::Fire => {
            if let Some(fire_damage) = item.attributes.get_mut(&AttributeType::FireDamage) {
                *fire_damage = enchantment.calculate_enhanced_value(*fire_damage);
            }
            if let Some(fire_resistance) = item.attributes.get_mut(&AttributeType::FireResistance) {
                *fire_resistance = enchantment.calculate_enhanced_value(*fire_resistance);
            }
        },
        EnchantmentType::Cold => {
            if let Some(cold_damage) = item.attributes.get_mut(&AttributeType::ColdDamage) {
                *cold_damage = enchantment.calculate_enhanced_value(*cold_damage);
            }
            if let Some(cold_resistance) = item.attributes.get_mut(&AttributeType::ColdResistance) {
                *cold_resistance = enchantment.calculate_enhanced_value(*cold_resistance);
            }
        },
        // ... similar for other enchantment types
    }
    
    // Apply visual effects
    item.visual_effects = Some(create_visual_effects(enchantment));
    item.color_tint = Some(get_color_tint(enchantment));
}
```

## Integration with Other Systems

### Equipment System

**Integration:**
- Enchantments are stored on equipment items
- Enchantments affect item value significantly
- Enchantments modify item attributes
- Visual effects are rendered on equipped items

### Skill System

**Integration:**
- Enchanting skill determines success rate
- Skill level unlocks higher enchantment levels
- Skill affects resource efficiency
- Skill progression rewards players

### Economy System

**Integration:**
- Enchanted items have much higher value
- Enchantment materials are valuable commodities
- Enchantment services create economic opportunities
- Failed enchantments create material sinks

### Visual System

**Integration:**
- VFX particles are rendered on enchanted items
- Color tints are applied to item textures
- Visual effects scale with enchantment level
- Effects are visible to all players

## Summary

The enchantment system:

- Allows players to enhance equipment with elemental properties
- Requires materials, skills, and gold
- Provides visual effects and increased item value
- Has six progressive levels with increasing costs and risks
- Includes upgrade and disenchantment mechanics
- Multiplies related elemental properties
- Creates high-value items through investment
- Balances risk and reward through failure mechanics

This system creates depth in item customization, allowing players to invest in powerful enchanted equipment while maintaining balance through material costs and failure risks.

