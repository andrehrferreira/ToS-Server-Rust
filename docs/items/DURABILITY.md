# Durability and Repair System Documentation

## Overview

The Durability and Repair system manages equipment degradation over time and provides multiple repair options for players. Equipment loses durability through combat damage, usage, and gathering activities. Players can repair items using gold, with different repair methods offering varying costs and effectiveness. Players with appropriate crafting skills can repair items without durability loss, and can even increase maximum durability with the right materials. NPC artisans provide premium repair services at higher costs.

## Key Features

- **Durability Degradation**: Equipment loses durability through combat, usage, and gathering
- **Multiple Repair Methods**: Self-repair with gold, skill-based repair, NPC artisan repair
- **Skill-Based Repair**: Players with crafting skills can repair without durability loss
- **Durability Enhancement**: Can increase maximum durability with proper materials
- **Progressive Durability Loss**: Repairs without skill progressively reduce maximum durability
- **NPC Artisan Services**: Premium repair services at higher costs
- **Broken Item State**: Items with 0 durability are marked as broken

## Durability System

### Durability Properties

**Equipment Durability:**
```typescript
export abstract class Equipament extends Item {
    public MaxDurability: number = 25;  // Maximum durability
    public Durability: number = 25;     // Current durability
}
```

**Durability Range:**
- **Base Durability**: 25 (starting point)
- **Minimum Durability**: 0 (broken state)
- **Maximum Durability**: 450 (with all bonuses)

### Durability Calculation

**Base Durability:**
- Starts at 25 for all equipment
- Modified by tier bonuses
- Modified by rarity bonuses
- Modified by exceptional flag (+50)
- Capped at 450 maximum

**Durability Bonuses:**

**Tier Bonuses:**
- T2: +10 durability
- T3: +25 durability
- T4: +50 durability
- T5: +75 durability
- T6: +100 durability
- T7: +150 durability
- T8: +200 durability

**Rarity Bonuses:**
- Uncommon: +20 durability
- Rare: +50 durability
- Magic: +100 durability
- Legendary: +100 durability
- Unique: +300 durability

**Exceptional Flag:**
- +50 durability
- +1 armor/resistances

**Final Calculation:**
```typescript
let baseDurability = 25;

// Tier bonus
baseDurability += tierBonus;

// Rarity bonus
baseDurability += rarityBonus;

// Exceptional flag
if (exceptional) {
    baseDurability += 50;
}

// Cap at maximum
this.MaxDurability = Math.min(baseDurability, 450);
this.Durability = this.MaxDurability;
```

## Durability Loss

### Loss Mechanisms

**1. Combat Damage:**
- Equipment loses durability when taking damage
- Chance-based reduction (10% chance per damage instance)
- Armor pieces lose durability when hit
- Weapons lose durability when attacking

**2. Usage:**
- Equipment loses durability through normal use
- Tools lose durability when gathering
- Weapons lose durability when attacking
- Armor loses durability when taking damage

**3. Gathering Activities:**
- Tools lose durability with each collection
- Mining: Pickaxe durability loss
- Lumberjack: Axe durability loss
- Herbalism: Scythe durability loss

### Durability Reduction Implementation

**TypeScript:**
```typescript
public static async reduceDurability(ref: string, owner: Player) {
    const item = Items.getItemByRef(ref);
    
    if (item instanceof Equipament) {
        // 10% chance to reduce durability
        const chanceReduce = Random.MinMaxInt(1, 10);
        
        if (chanceReduce === 1) {
            const newDurability = Math.max(item.Durability - 1, 0);
            item.setDurability(newDurability, item.MaxDurability);
            
            // Mark as broken if durability reaches 0
            if (newDurability <= 0) {
                item.Flags.addFlag(ItemStates.Broken);
            }
        }
    }
}
```

**Durability Loss Rates:**
- **Combat**: 10% chance per damage instance
- **Gathering**: 100% chance per collection (tools)
- **Usage**: 10% chance per use

### Broken Item State

**Broken Item Properties:**
- Durability = 0
- Flag: `ItemStates.Broken`
- GoldCost = 1 (minimal value)
- Cannot be used effectively
- Still provides reduced bonuses (50% effectiveness for tools)

**Broken Item Effects:**
- Equipment: Reduced effectiveness
- Tools: 50% bonus effectiveness
- Cannot be equipped (optional restriction)
- Must be repaired before full use

## Repair System

### Repair Methods

**1. Self-Repair with Gold:**
- Player repairs item using gold
- Cost based on missing durability
- Without skill: Progressive durability loss
- With skill: No durability loss

**2. Skill-Based Repair:**
- Requires appropriate crafting skill
- No durability loss when repairing
- Can increase maximum durability
- Requires materials for enhancement

**3. NPC Artisan Repair:**
- Premium repair service
- Higher cost than self-repair
- No durability loss guaranteed
- Available at specialized NPCs

### Self-Repair with Gold

**Basic Repair:**
- Cost: Based on missing durability percentage
- Formula: `cost = (missingDurability / maxDurability) * baseItemValue * repairMultiplier`
- Repair multiplier: 0.1 to 0.5 (configurable)

**Repair Without Skill:**
- Repairs current durability
- **Progressive Durability Loss**: Maximum durability reduces with each repair
- Loss formula: `maxDurability = maxDurability - (repairCount * durabilityLossPerRepair)`
- Durability loss per repair: 1-3 points (varies by item tier)
- Higher tier items lose less durability per repair

**Repair With Skill:**
- Requires appropriate crafting skill (Blacksmithing, Tailoring, etc.)
- No durability loss
- Can repair to full durability
- Can increase maximum durability with materials

**Implementation:**
```typescript
public async repairItem(itemRef: string, useSkill: boolean = false) {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return { success: false, error: "Item is not equipment" };
    }
    
    const missingDurability = item.MaxDurability - item.Durability;
    const repairCost = this.calculateRepairCost(item, missingDurability);
    
    // Check if player has enough gold
    if (this.getGoldCoins() < repairCost) {
        return { success: false, error: "Insufficient gold" };
    }
    
    // Check if player has required skill
    const hasSkill = useSkill && this.hasRepairSkill(item);
    
    if (hasSkill) {
        // Skill-based repair: No durability loss
        item.Durability = item.MaxDurability;
    } else {
        // Basic repair: Progressive durability loss
        item.Durability = item.MaxDurability;
        
        // Reduce maximum durability
        const durabilityLoss = this.calculateDurabilityLoss(item);
        item.MaxDurability = Math.max(
            item.MaxDurability - durabilityLoss,
            item.Durability
        );
    }
    
    // Remove gold
    await this.removeGoldCoins(repairCost);
    
    // Remove broken flag if repaired
    if (item.Durability > 0) {
        item.Flags.removeFlag(ItemStates.Broken);
    }
    
    return { success: true, newDurability: item.Durability, newMaxDurability: item.MaxDurability };
}
```

### Skill-Based Repair

**Required Skills:**
- **Weapons/Armor**: Blacksmithing skill
- **Cloth Armor**: Tailoring skill
- **Tools**: Blacksmithing skill
- **Jewelry**: Jewelry skill

**Skill Requirements:**
- Minimum skill level: Varies by item tier
- Higher tier items require higher skill levels
- Formula: `requiredSkill = itemTier * 2`

**Skill-Based Repair Benefits:**
- **No Durability Loss**: Maximum durability remains unchanged
- **Full Repair**: Can repair to full maximum durability
- **Durability Enhancement**: Can increase maximum durability with materials

**Durability Enhancement:**
- Requires rare materials (ores, gemstones)
- Increases maximum durability permanently
- Enhancement amount depends on materials used
- Formula: `newMaxDurability = currentMaxDurability + enhancementAmount`

**Enhancement Materials:**
- **Common Enhancement**: Basic ores (+10-25 durability)
- **Uncommon Enhancement**: Uncommon ores + gemstones (+25-50 durability)
- **Rare Enhancement**: Rare ores + rare gemstones (+50-100 durability)
- **Very Rare Enhancement**: Very rare ores + very rare gemstones (+100-200 durability)

**Implementation:**
```typescript
public async enhanceDurability(
    itemRef: string,
    materials: { materialType: string, quantity: number }[]
) {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return { success: false, error: "Item is not equipment" };
    }
    
    // Check if player has required skill
    const requiredSkill = this.getRequiredRepairSkill(item);
    const playerSkill = this.getSkillValue(requiredSkill);
    
    if (playerSkill < requiredSkill) {
        return { success: false, error: "Insufficient skill level" };
    }
    
    // Calculate enhancement amount from materials
    let enhancementAmount = 0;
    
    for (const material of materials) {
        const materialEnhancement = this.getMaterialEnhancementValue(
            material.materialType,
            material.quantity
        );
        enhancementAmount += materialEnhancement;
    }
    
    // Check if player has materials
    const hasMaterials = await this.checkMaterials(materials);
    if (!hasMaterials) {
        return { success: false, error: "Missing required materials" };
    }
    
    // Remove materials
    await this.removeMaterials(materials);
    
    // Increase maximum durability
    const newMaxDurability = Math.min(
        item.MaxDurability + enhancementAmount,
        450 // Cap at maximum
    );
    
    item.MaxDurability = newMaxDurability;
    
    // If current durability is at max, update it too
    if (item.Durability === item.MaxDurability) {
        item.Durability = newMaxDurability;
    }
    
    return {
        success: true,
        newMaxDurability: item.MaxDurability,
        enhancementAmount: enhancementAmount
    };
}
```

### Progressive Durability Loss

**Mechanism:**
- Each repair without skill reduces maximum durability
- Higher tier items lose less durability per repair
- Lower tier items lose more durability per repair

**Durability Loss Per Repair:**
- **T0-T1**: -3 durability per repair
- **T2-T3**: -2 durability per repair
- **T4-T5**: -2 durability per repair
- **T6-T7**: -1 durability per repair
- **T8**: -1 durability per repair

**Example:**
```
Item: Iron Sword (T3)
Max Durability: 100
Current Durability: 50
Missing: 50

Repair 1 (without skill):
- Repairs to 100
- Max Durability: 100 - 2 = 98

Repair 2 (without skill):
- Repairs to 98
- Max Durability: 98 - 2 = 96

Repair 3 (without skill):
- Repairs to 96
- Max Durability: 96 - 2 = 94
```

**Prevention:**
- Use skill-based repair to prevent durability loss
- Use NPC artisan repair (premium cost, no loss)
- Enhance durability to compensate for losses

### NPC Artisan Repair

**NPC Artisan Services:**
- Specialized NPCs in towns/cities
- Provide premium repair services
- Higher cost than self-repair
- Guaranteed no durability loss

**Repair Cost:**
- Base cost: 2-5x self-repair cost
- Formula: `npcCost = selfRepairCost * npcMultiplier`
- NPC multiplier: 2.0 to 5.0 (varies by NPC)

**Benefits:**
- **No Durability Loss**: Maximum durability remains unchanged
- **Full Repair**: Repairs to full maximum durability
- **No Skill Required**: Available to all players
- **Reliable**: Always successful

**NPC Types:**
- **Blacksmith**: Repairs weapons and metal armor
- **Tailor**: Repairs cloth armor
- **Jeweler**: Repairs jewelry and accessories
- **Tool Specialist**: Repairs tools

**Implementation:**
```typescript
public async npcRepair(itemRef: string, npcType: string) {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return { success: false, error: "Item is not equipment" };
    }
    
    // Check if NPC can repair this item type
    const canRepair = this.canNpcRepair(npcType, item);
    if (!canRepair) {
        return { success: false, error: "NPC cannot repair this item type" };
    }
    
    const missingDurability = item.MaxDurability - item.Durability;
    const baseRepairCost = this.calculateRepairCost(item, missingDurability);
    const npcMultiplier = this.getNpcMultiplier(npcType);
    const npcCost = baseRepairCost * npcMultiplier;
    
    // Check if player has enough gold
    if (this.getGoldCoins() < npcCost) {
        return { success: false, error: "Insufficient gold" };
    }
    
    // Remove gold
    await this.removeGoldCoins(npcCost);
    
    // Repair to full (no durability loss)
    item.Durability = item.MaxDurability;
    
    // Remove broken flag
    if (item.Durability > 0) {
        item.Flags.removeFlag(ItemStates.Broken);
    }
    
    return {
        success: true,
        newDurability: item.Durability,
        cost: npcCost
    };
}
```

## Repair Cost Calculation

### Cost Formula

**Base Cost:**
```typescript
function calculateRepairCost(item: Equipament, missingDurability: number): number {
    const durabilityPercentage = missingDurability / item.MaxDurability;
    const baseItemValue = item.GoldCost;
    const repairMultiplier = 0.2; // 20% of item value per full repair
    
    return Math.ceil(baseItemValue * durabilityPercentage * repairMultiplier);
}
```

**Cost Factors:**
- **Item Value**: Higher value items cost more to repair
- **Missing Durability**: More missing durability = higher cost
- **Repair Method**: NPC repair costs 2-5x more
- **Item Tier**: Higher tier items may have different multipliers

**Example Calculations:**

**Low Tier Item:**
- Item: Iron Sword (T3)
- Base Value: 100 gold
- Max Durability: 100
- Current Durability: 50
- Missing: 50 (50%)
- Self-Repair Cost: 100 * 0.5 * 0.2 = 10 gold
- NPC Repair Cost: 10 * 3.0 = 30 gold

**High Tier Item:**
- Item: Mithril Sword (T7)
- Base Value: 1000 gold
- Max Durability: 300
- Current Durability: 150
- Missing: 150 (50%)
- Self-Repair Cost: 1000 * 0.5 * 0.2 = 100 gold
- NPC Repair Cost: 100 * 3.0 = 300 gold

## Repair Skill Requirements

### Skill Mapping

**Weapon Repair:**
- **Skill**: Blacksmithing
- **T0-T2**: Skill level 1+
- **T3-T4**: Skill level 3+
- **T5-T6**: Skill level 5+
- **T7-T8**: Skill level 7+

**Armor Repair:**
- **Metal Armor**: Blacksmithing (same as weapons)
- **Cloth Armor**: Tailoring
- **Leather Armor**: Tailoring or Blacksmithing

**Tool Repair:**
- **Pickaxe/Axe**: Blacksmithing
- **Scythe**: Blacksmithing or Tailoring

**Jewelry Repair:**
- **Skill**: Jewelry
- **All Tiers**: Skill level 2+

### Skill Check

**Implementation:**
```typescript
function hasRepairSkill(player: Player, item: Equipament): boolean {
    const requiredSkill = getRequiredRepairSkill(item);
    const playerSkill = player.getSkillValue(requiredSkill);
    const requiredLevel = getRequiredSkillLevel(item.Tier);
    
    return playerSkill >= requiredLevel;
}

function getRequiredRepairSkill(item: Equipament): SkillName {
    if (item instanceof Weapon || item instanceof Tool) {
        return SkillName.Blacksmithing;
    }
    
    if (item instanceof Armor) {
        if (item.MaterialType === MaterialType.Cloth) {
            return SkillName.Tailoring;
        }
        return SkillName.Blacksmithing;
    }
    
    if (item instanceof Jewelry) {
        return SkillName.Jewelry;
    }
    
    return SkillName.Blacksmithing; // Default
}

function getRequiredSkillLevel(tier: EquipamentTier): number {
    switch (tier) {
        case EquipamentTier.T0:
        case EquipamentTier.T1:
        case EquipamentTier.T2:
            return 1;
        case EquipamentTier.T3:
        case EquipamentTier.T4:
            return 3;
        case EquipamentTier.T5:
        case EquipamentTier.T6:
            return 5;
        case EquipamentTier.T7:
        case EquipamentTier.T8:
            return 7;
        default:
            return 1;
    }
}
```

## Durability Enhancement Materials

### Material Types

**Common Enhancement Materials:**
- Iron Ore: +10 durability per 5 ore
- Copper Ore: +10 durability per 5 ore
- Stone: +5 durability per 10 stone

**Uncommon Enhancement Materials:**
- Silver Ore: +25 durability per 3 ore
- Gold Ore: +25 durability per 3 ore
- Emerald: +15 durability per gem
- Common Gemstones: +10 durability per gem

**Rare Enhancement Materials:**
- Dark Ore: +50 durability per 2 ore
- Mithril Ore: +50 durability per 2 ore
- Diamond: +30 durability per gem
- Ruby: +30 durability per gem
- Ametist: +30 durability per gem

**Very Rare Enhancement Materials:**
- Heavenly Ore: +100 durability per ore
- Sunstone: +50 durability per gem
- Legendary Gemstones: +75 durability per gem

### Enhancement Calculation

**Formula:**
```typescript
function calculateEnhancement(materials: Material[]): number {
    let totalEnhancement = 0;
    
    for (const material of materials) {
        const enhancementPerUnit = getEnhancementValue(material.type);
        totalEnhancement += enhancementPerUnit * material.quantity;
    }
    
    // Cap enhancement per repair
    return Math.min(totalEnhancement, 200);
}

function getEnhancementValue(materialType: string): number {
    switch (materialType) {
        case "IronOre": return 2;      // 5 ore = 10 durability
        case "CopperOre": return 2;
        case "SilverOre": return 8.33;  // 3 ore = 25 durability
        case "GoldOre": return 8.33;
        case "DarkOre": return 25;     // 2 ore = 50 durability
        case "MithrilOre": return 25;
        case "HeavenlyOre": return 100; // 1 ore = 100 durability
        case "Emerald": return 15;
        case "Diamond": return 30;
        case "Ruby": return 30;
        case "Ametist": return 30;
        case "Sunstone": return 50;
        default: return 0;
    }
}
```

## Rust Implementation Considerations

### Data Structures

```rust
pub struct Durability {
    pub current: u32,
    pub max: u32,
}

impl Durability {
    pub fn new(max: u32) -> Self {
        Self {
            current: max,
            max,
        }
    }
    
    pub fn reduce(&mut self, amount: u32) {
        self.current = self.current.saturating_sub(amount);
    }
    
    pub fn is_broken(&self) -> bool {
        self.current == 0
    }
    
    pub fn repair(&mut self, amount: u32, preserve_max: bool) {
        if preserve_max {
            self.current = self.max.min(self.current + amount);
        } else {
            // Progressive durability loss
            let repair_amount = amount.min(self.max - self.current);
            self.current += repair_amount;
            self.max = self.max.saturating_sub(1); // Lose 1 max per repair
        }
    }
    
    pub fn enhance(&mut self, amount: u32) {
        self.max = (self.max + amount).min(450); // Cap at 450
        self.current = self.max.min(self.current + amount);
    }
}

pub enum RepairMethod {
    SelfRepair { use_skill: bool },
    NpcRepair { npc_type: String },
    SkillRepair { skill: SkillName, materials: Vec<Material> },
}

pub struct RepairResult {
    pub success: bool,
    pub new_durability: u32,
    pub new_max_durability: u32,
    pub cost: u32,
    pub error: Option<String>,
}
```

### Repair Implementation

```rust
impl Player {
    pub async fn repair_item(
        &mut self,
        item_ref: &str,
        method: RepairMethod,
    ) -> Result<RepairResult, RepairError> {
        let item = self.get_item(item_ref)?;
        
        if !item.is_equipment() {
            return Err(RepairError::NotEquipment);
        }
        
        let equipment = item.as_equipment()?;
        let missing_durability = equipment.durability.max - equipment.durability.current;
        
        match method {
            RepairMethod::SelfRepair { use_skill } => {
                self.self_repair(equipment, missing_durability, use_skill).await
            }
            RepairMethod::NpcRepair { npc_type } => {
                self.npc_repair(equipment, missing_durability, npc_type).await
            }
            RepairMethod::SkillRepair { skill, materials } => {
                self.skill_repair(equipment, skill, materials).await
            }
        }
    }
    
    async fn self_repair(
        &mut self,
        equipment: &mut Equipment,
        missing: u32,
        use_skill: bool,
    ) -> Result<RepairResult, RepairError> {
        let cost = self.calculate_repair_cost(equipment, missing);
        
        if self.gold < cost {
            return Err(RepairError::InsufficientGold);
        }
        
        let has_skill = if use_skill {
            self.has_repair_skill(equipment)
        } else {
            false
        };
        
        if has_skill {
            // Skill-based repair: No durability loss
            equipment.durability.current = equipment.durability.max;
        } else {
            // Basic repair: Progressive durability loss
            equipment.durability.current = equipment.durability.max;
            let durability_loss = self.calculate_durability_loss(equipment.tier);
            equipment.durability.max = equipment.durability.max.saturating_sub(durability_loss);
        }
        
        self.gold -= cost;
        
        Ok(RepairResult {
            success: true,
            new_durability: equipment.durability.current,
            new_max_durability: equipment.durability.max,
            cost,
            error: None,
        })
    }
    
    async fn npc_repair(
        &mut self,
        equipment: &mut Equipment,
        missing: u32,
        npc_type: &str,
    ) -> Result<RepairResult, RepairError> {
        let base_cost = self.calculate_repair_cost(equipment, missing);
        let multiplier = self.get_npc_multiplier(npc_type);
        let cost = (base_cost as f32 * multiplier) as u32;
        
        if self.gold < cost {
            return Err(RepairError::InsufficientGold);
        }
        
        // NPC repair: No durability loss
        equipment.durability.current = equipment.durability.max;
        
        self.gold -= cost;
        
        Ok(RepairResult {
            success: true,
            new_durability: equipment.durability.current,
            new_max_durability: equipment.durability.max,
            cost,
            error: None,
        })
    }
    
    async fn skill_repair(
        &mut self,
        equipment: &mut Equipment,
        skill: SkillName,
        materials: Vec<Material>,
    ) -> Result<RepairResult, RepairError> {
        let required_skill = self.get_required_repair_skill(equipment);
        let player_skill = self.get_skill_value(required_skill);
        
        if player_skill < self.get_required_skill_level(equipment.tier) {
            return Err(RepairError::InsufficientSkill);
        }
        
        // Check materials
        if !self.has_materials(&materials) {
            return Err(RepairError::MissingMaterials);
        }
        
        // Calculate enhancement
        let enhancement = self.calculate_enhancement(&materials);
        
        // Remove materials
        self.remove_materials(&materials).await?;
        
        // Repair and enhance
        equipment.durability.current = equipment.durability.max;
        equipment.durability.enhance(enhancement);
        
        Ok(RepairResult {
            success: true,
            new_durability: equipment.durability.current,
            new_max_durability: equipment.durability.max,
            cost: 0, // No gold cost for skill repair
            error: None,
        })
    }
    
    fn calculate_durability_loss(&self, tier: EquipamentTier) -> u32 {
        match tier {
            EquipamentTier::T0 | EquipamentTier::T1 => 3,
            EquipamentTier::T2 | EquipamentTier::T3 => 2,
            EquipamentTier::T4 | EquipamentTier::T5 => 2,
            EquipamentTier::T6 | EquipamentTier::T7 => 1,
            EquipamentTier::T8 => 1,
        }
    }
}
```

## Integration with Other Systems

### Equipment System

- Durability affects equipment effectiveness
- Broken items have reduced functionality
- Durability is part of equipment serialization

### Crafting System

- Crafting skills enable skill-based repair
- Materials used for durability enhancement
- Repair creates demand for materials

### Economy System

- Repair costs affect gold economy
- NPC repair creates gold sink
- Material enhancement creates material demand

### Gathering System

- Tools lose durability during gathering
- Tool repair is essential for gatherers
- Tool enhancement improves gathering efficiency

## Testing Strategy

### Unit Tests

- Test durability reduction mechanics
- Test repair cost calculation
- Test skill-based repair
- Test progressive durability loss
- Test durability enhancement
- Test broken item state

### Integration Tests

- Test full repair flow
- Test NPC repair interaction
- Test material enhancement
- Test skill requirements
- Test gold cost deduction

### Balance Tests

- Test repair cost balance
- Test durability loss rates
- Test enhancement material values
- Test NPC cost multipliers
- Test skill requirement progression

