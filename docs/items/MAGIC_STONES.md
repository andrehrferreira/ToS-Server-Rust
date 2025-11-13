# Magic Stones System Documentation

## Overview

Magic Stones are rare, droppable items from elite monsters and bosses that serve to transform and enhance equipment rarity and attributes, similar to Path of Exile 2's currency system. These stones have guaranteed effects (no RNG on use) but are extremely rare drops, making them valuable commodities. Magic Stones can also be used as trade currency in the auction house, creating a secondary economy around these rare items.

## Key Features

- **Rare Drops**: Dropped by elite monsters and bosses with low probability
- **Guaranteed Effects**: All stone effects are 100% successful (no RNG on use)
- **Rarity Transformation**: Transform items between rarity tiers
- **Attribute Enhancement**: Add attributes, randomize prefixes/suffixes, increase attribute tiers
- **Equipment Enhancement**: Add card slots, increase equipment tier
- **Trade Currency**: Can be used as currency in auction house
- **No Luck Required**: Effects always succeed, rarity is in obtaining the stones

## Magic Stone Types

### 1. Ascension Shard (Common → Uncommon)

**Name**: Ascension Shard  
**Effect**: Transforms Common items to Uncommon rarity and adds 1 random attribute  
**Target**: Common items only  
**Result**: 
- Rarity: Common → Uncommon
- Adds 1 random attribute
- Item gains rarity bonus (durability, value, etc.)

**Usage:**
```typescript
public useAscensionShard(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Common) {
        return false; // Only works on Common items
    }
    
    item.Rarity = ItemRarity.Uncommon;
    item.generateRandomAttribute(1); // Add 1 attribute
    item.updateGoldCost(); // Recalculate value
    
    return true;
}
```

### 2. Essence Fragment (Uncommon Enhancement)

**Name**: Essence Fragment  
**Effect**: Adds +1 attribute to Uncommon items  
**Target**: Uncommon items only  
**Result**:
- Adds 1 random attribute
- Does not change rarity
- Increases item value

**Usage:**
```typescript
public useEssenceFragment(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Uncommon) {
        return false; // Only works on Uncommon items
    }
    
    item.generateRandomAttribute(1); // Add 1 attribute
    item.updateGoldCost();
    
    return true;
}
```

### 3. Metamorphic Core (Uncommon → Rare)

**Name**: Metamorphic Core  
**Effect**: Transforms Uncommon items to Rare rarity  
**Target**: Uncommon items only  
**Result**:
- Rarity: Uncommon → Rare
- Gains rarity bonuses (durability, value, etc.)
- Maintains existing attributes

**Usage:**
```typescript
public useMetamorphicCore(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Uncommon) {
        return false; // Only works on Uncommon items
    }
    
    item.Rarity = ItemRarity.Rare;
    item.updateGoldCost();
    
    return true;
}
```

### 4. Primal Essence (Rare Enhancement)

**Name**: Primal Essence  
**Effect**: Adds +1 attribute to Rare items  
**Target**: Rare items only  
**Result**:
- Adds 1 random attribute
- Does not change rarity
- Increases item value

**Usage:**
```typescript
public usePrimalEssence(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Rare) {
        return false; // Only works on Rare items
    }
    
    item.generateRandomAttribute(1); // Add 1 attribute
    item.updateGoldCost();
    
    return true;
}
```

### 5. Arcane Catalyst (Rare → Magic)

**Name**: Arcane Catalyst  
**Effect**: Transforms Rare items to Magic rarity  
**Target**: Rare items only  
**Result**:
- Rarity: Rare → Magic
- Gains magic rarity bonuses
- Maintains existing attributes

**Usage:**
```typescript
public useArcaneCatalyst(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Rare) {
        return false; // Only works on Rare items
    }
    
    item.Rarity = ItemRarity.Magic;
    item.updateGoldCost();
    
    return true;
}
```

### 6. Enchanted Essence (Magic Enhancement)

**Name**: Enchanted Essence  
**Effect**: Adds +1 attribute to Magic items  
**Target**: Magic items only  
**Result**:
- Adds 1 random attribute
- Does not change rarity
- Increases item value

**Usage:**
```typescript
public useEnchantedEssence(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Magic) {
        return false; // Only works on Magic items
    }
    
    item.generateRandomAttribute(1); // Add 1 attribute
    item.updateGoldCost();
    
    return true;
}
```

### 7. Divine Spark (Magic → Legendary)

**Name**: Divine Spark  
**Effect**: Transforms Magic items to Legendary rarity  
**Target**: Magic items only  
**Result**:
- Rarity: Magic → Legendary
- Gains legendary rarity bonuses
- Maintains existing attributes

**Usage:**
```typescript
public useDivineSpark(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Magic) {
        return false; // Only works on Magic items
    }
    
    item.Rarity = ItemRarity.Legendary;
    item.updateGoldCost();
    
    return true;
}
```

### 8. Primordial Fragment (Common → Unique)

**Name**: Primordial Fragment  
**Effect**: Extremely rare chance to transform Common items to Unique rarity  
**Target**: Common items only  
**Result**:
- Rarity: Common → Unique
- Gains unique rarity bonuses
- Adds unique properties
- Very rare drop (lowest probability)

**Usage:**
```typescript
public usePrimordialFragment(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (item.Rarity !== ItemRarity.Common) {
        return false; // Only works on Common items
    }
    
    item.Rarity = ItemRarity.Unique;
    item.generateUniqueProperties(); // Add unique properties
    item.updateGoldCost();
    
    return true;
}
```

**Drop Probability**: Lowest of all stones (extremely rare)

### 9. Vortex Shard (Prefix Randomization)

**Name**: Vortex Shard  
**Effect**: Randomizes one prefix on the item  
**Target**: Items with prefixes  
**Result**:
- Randomly selects one existing prefix
- Replaces it with a random prefix of same tier
- Maintains other attributes

**Usage:**
```typescript
public useVortexShard(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false;
    }
    
    const prefixes = item.getPrefixes();
    if (prefixes.length === 0) {
        return false; // No prefixes to randomize
    }
    
    // Select random prefix
    const randomIndex = Random.MinMaxInt(0, prefixes.length - 1);
    const prefixToReplace = prefixes[randomIndex];
    
    // Generate new random prefix of same tier
    const newPrefix = this.generateRandomPrefix(prefixToReplace.tier);
    item.replacePrefix(randomIndex, newPrefix);
    item.updateGoldCost();
    
    return true;
}
```

### 10. Flux Crystal (Suffix Randomization)

**Name**: Flux Crystal  
**Effect**: Randomizes one suffix on the item  
**Target**: Items with suffixes  
**Result**:
- Randomly selects one existing suffix
- Replaces it with a random suffix of same tier
- Maintains other attributes

**Usage:**
```typescript
public useFluxCrystal(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false;
    }
    
    const suffixes = item.getSuffixes();
    if (suffixes.length === 0) {
        return false; // No suffixes to randomize
    }
    
    // Select random suffix
    const randomIndex = Random.MinMaxInt(0, suffixes.length - 1);
    const suffixToReplace = suffixes[randomIndex];
    
    // Generate new random suffix of same tier
    const newSuffix = this.generateRandomSuffix(suffixToReplace.tier);
    item.replaceSuffix(randomIndex, newSuffix);
    item.updateGoldCost();
    
    return true;
}
```

### 11. Apex Gem (Attribute Tier Enhancement)

**Name**: Apex Gem  
**Effect**: Increases the tier of one random attribute  
**Target**: Items with attributes  
**Result**:
- Randomly selects one attribute
- Increases its tier by 1 (if not at max)
- Increases attribute value

**Usage:**
```typescript
public useApexGem(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false;
    }
    
    const attributes = item.getAttributes();
    if (attributes.length === 0) {
        return false; // No attributes to upgrade
    }
    
    // Select random attribute
    const randomIndex = Random.MinMaxInt(0, attributes.length - 1);
    const attribute = attributes[randomIndex];
    
    // Increase tier (if not at max)
    if (attribute.tier < MaxAttributeTier) {
        attribute.tier++;
        attribute.value = this.calculateAttributeValue(attribute.type, attribute.tier);
        item.updateGoldCost();
        return true;
    }
    
    return false; // Already at max tier
}
```

### 12. Transcendence Orb (Equipment Tier Enhancement)

**Name**: Transcendence Orb  
**Effect**: Increases the equipment tier by 1  
**Target**: Equipment items  
**Result**:
- Equipment tier increases by 1 (if not at max)
- Gains tier bonuses (durability, stat requirements, etc.)
- Increases item value

**Usage:**
```typescript
public useTranscendenceOrb(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false; // Only works on equipment
    }
    
    if (item.Tier >= EquipamentTier.T8) {
        return false; // Already at max tier
    }
    
    // Increase tier
    const currentTier = item.Tier;
    item.Tier = this.getNextTier(currentTier);
    
    // Apply tier bonuses
    item.applyTierBonuses();
    item.updateGoldCost();
    
    return true;
}
```

**Tier Progression:**
- T0 → T1
- T1 → T2
- ...
- T7 → T8 (max)

### 13. Chaos Essence (Wild Attribute)

**Name**: Chaos Essence  
**Effect**: Adds one random attribute of any tier  
**Target**: All equipment (except Unique items)  
**Result**:
- Adds 1 random attribute
- Attribute tier is random (can be any tier)
- Does not work on Unique items

**Usage:**
```typescript
public useChaosEssence(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false; // Only works on equipment
    }
    
    if (item.Rarity === ItemRarity.Unique) {
        return false; // Does not work on Unique items
    }
    
    // Generate random attribute of random tier
    const randomTier = Random.MinMaxInt(1, MaxAttributeTier);
    const randomAttribute = this.generateRandomAttribute(randomTier);
    
    item.addAttribute(randomAttribute);
    item.updateGoldCost();
    
    return true;
}
```

**Restrictions:**
- Cannot be used on Unique items
- Works on all other rarities
- Attribute tier is completely random

### 14. Receptacle Stone (Card Slot Addition)

**Name**: Receptacle Stone  
**Effect**: Adds one card slot to equipment/weapon  
**Target**: Equipment and weapons  
**Result**:
- Adds 1 card slot
- Increases `CardSlots` property
- Allows insertion of additional cards

**Usage:**
```typescript
public useReceptacleStone(itemRef: string): boolean {
    const item = Items.getItemByRef(itemRef);
    
    if (!(item instanceof Equipament)) {
        return false; // Only works on equipment
    }
    
    // Check if item can have more slots
    if (item.CardSlots >= item.MaxSlots) {
        return false; // Already at max slots
    }
    
    // Add card slot
    item.CardSlots++;
    item.Name = `${item.Name} (${item.CardSlots})`; // Update name
    item.updateGoldCost();
    
    return true;
}
```

**Slot Limits:**
- Items have `MaxSlots` property (varies by tier)
- Cannot exceed maximum slots
- Each slot allows one card insertion

## Drop System

### Drop Sources

**Elite Monsters:**
- Elite monsters have a chance to drop Magic Stones
- Drop probability: Very low (0.1% - 1% per kill)
- Each elite monster can drop any stone type
- Higher level elites have slightly better drop rates

**Boss Monsters:**
- Boss monsters have higher chance to drop Magic Stones
- Drop probability: Low (1% - 5% per kill)
- Bosses can drop multiple stone types
- Higher level bosses drop rarer stones more frequently

**Drop Probability Table:**

| Stone Type | Elite Drop Rate | Boss Drop Rate |
|------------|----------------|----------------|
| Ascension Shard | 0.5% | 2% |
| Essence Fragment | 0.3% | 1.5% |
| Metamorphic Core | 0.3% | 1.5% |
| Primal Essence | 0.2% | 1% |
| Arcane Catalyst | 0.15% | 0.8% |
| Enchanted Essence | 0.1% | 0.5% |
| Divine Spark | 0.05% | 0.3% |
| Primordial Fragment | 0.01% | 0.1% |
| Vortex Shard | 0.2% | 1% |
| Flux Crystal | 0.2% | 1% |
| Apex Gem | 0.15% | 0.8% |
| Transcendence Orb | 0.1% | 0.5% |
| Chaos Essence | 0.1% | 0.5% |
| Receptacle Stone | 0.15% | 0.8% |

### Drop Implementation

**TypeScript:**
```typescript
public dropMagicStone(): MagicStoneType | null {
    const isElite = this.isElite();
    const isBoss = this.isBoss();
    
    if (!isElite && !isBoss) {
        return null; // Only elites and bosses drop stones
    }
    
    const dropChance = isBoss ? this.getBossDropChance() : this.getEliteDropChance();
    const random = Math.random();
    
    if (random > dropChance) {
        return null; // No drop
    }
    
    // Select random stone type based on drop rates
    const stoneType = this.selectRandomStoneType(isBoss);
    return stoneType;
}

private selectRandomStoneType(isBoss: boolean): MagicStoneType {
    const dropTable = isBoss ? BOSS_DROP_TABLE : ELITE_DROP_TABLE;
    const totalWeight = dropTable.reduce((sum, entry) => sum + entry.weight, 0);
    let random = Math.random() * totalWeight;
    
    for (const entry of dropTable) {
        random -= entry.weight;
        if (random <= 0) {
            return entry.stoneType;
        }
    }
    
    return dropTable[0].stoneType; // Fallback
}
```

## Stone Usage System

### Usage Requirements

**Item Requirements:**
- Item must be equipment (weapon, armor, accessory)
- Item must meet stone-specific requirements (rarity, etc.)
- Item cannot be broken (durability > 0)

**Player Requirements:**
- Stone must be in inventory
- Item must be in inventory or equipped
- No skill requirements (stones work for all players)

### Usage Process

**General Flow:**
1. Player selects item to enhance
2. Player selects Magic Stone to use
3. System validates requirements
4. Stone effect is applied (guaranteed success)
5. Stone is consumed (removed from inventory)
6. Item is updated and saved

**Implementation:**
```typescript
public async useMagicStone(
    stoneType: MagicStoneType,
    itemRef: string
): Promise<{ success: boolean, error?: string }> {
    // Get stone from inventory
    const stone = this.inventory.getItemByNamespace(`MagicStone_${stoneType}`);
    if (!stone) {
        return { success: false, error: "Stone not found in inventory" };
    }
    
    // Get item
    const item = Items.getItemByRef(itemRef);
    if (!item) {
        return { success: false, error: "Item not found" };
    }
    
    // Validate requirements
    const validation = this.validateStoneUsage(stoneType, item);
    if (!validation.valid) {
        return { success: false, error: validation.error };
    }
    
    // Apply stone effect
    const result = this.applyStoneEffect(stoneType, item);
    
    if (result.success) {
        // Remove stone from inventory
        await this.inventory.removeItem(stone.Ref);
        
        // Update item
        item.updateGoldCost();
        this.save();
        this.saveToDatabase();
    }
    
    return result;
}
```

## Stone as Trade Currency

### Auction House Currency

**Magic Stones as Currency:**
- Magic Stones can be listed on auction house
- Players can bid/buyout using Magic Stones
- Creates secondary economy around stones
- Stone value varies by rarity and usefulness

**Stone Value Hierarchy:**
1. **Highest Value**: Primordial Fragment, Divine Spark, Transcendence Orb
2. **High Value**: Arcane Catalyst, Chaos Essence, Receptacle Stone
3. **Medium Value**: Apex Gem, Vortex Shard, Flux Crystal
4. **Lower Value**: Ascension Shard, Essence Fragment, Metamorphic Core

**Auction House Integration:**
```typescript
public async listItemOnAuctionHouse(
    itemRef: string,
    startingBid: number,
    buyout: number,
    currency: "Gold" | MagicStoneType
) {
    // List item with stone as currency option
    if (currency !== "Gold") {
        // Validate stone currency
        const stoneValue = this.getStoneValue(currency);
        // Convert stone value to equivalent gold for display
        const equivalentGold = stoneValue * this.getStoneToGoldRate(currency);
    }
    
    // Create auction listing
    await this.auctionHouse.createListing({
        itemRef,
        startingBid,
        buyout,
        currency
    });
}
```

### Stone Trading

**Direct Trade:**
- Players can trade Magic Stones directly
- Stone-for-stone trades
- Stone-for-item trades
- Stone-for-gold trades

**Trade Value:**
- Stone values fluctuate based on demand
- Rarer stones have higher value
- More useful stones have higher value
- Market-driven pricing

## Stone Rarity and Value

### Rarity Classification

**Common Stones** (More frequent drops):
- Ascension Shard
- Essence Fragment
- Metamorphic Core

**Uncommon Stones** (Moderate drops):
- Primal Essence
- Vortex Shard
- Flux Crystal

**Rare Stones** (Less frequent drops):
- Arcane Catalyst
- Enchanted Essence
- Apex Gem
- Receptacle Stone
- Chaos Essence

**Very Rare Stones** (Very infrequent drops):
- Divine Spark
- Transcendence Orb

**Ultra Rare Stones** (Extremely rare drops):
- Primordial Fragment

### Value Factors

**1. Rarity:**
- Rarer stones = higher value
- Drop probability inversely correlates with value

**2. Usefulness:**
- More useful effects = higher value
- Transformation stones (rarity upgrades) = high value
- Enhancement stones = moderate value

**3. Demand:**
- Popular stones = higher value
- Meta-dependent value
- Market-driven pricing

## Rust Implementation Considerations

### Data Structures

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum MagicStoneType {
    AscensionShard,          // Common → Uncommon + 1 attr
    EssenceFragment,        // +1 attr (Uncommon)
    MetamorphicCore,        // Uncommon → Rare
    PrimalEssence,          // +1 attr (Rare)
    ArcaneCatalyst,        // Rare → Magic
    EnchantedEssence,      // +1 attr (Magic)
    DivineSpark,           // Magic → Legendary
    PrimordialFragment,    // Common → Unique (very rare)
    VortexShard,           // Randomize prefix
    FluxCrystal,           // Randomize suffix
    ApexGem,               // +1 tier to attribute
    TranscendenceOrb,      // +1 tier to equipment
    ChaosEssence,          // +1 random attr (any tier)
    ReceptacleStone,       // +1 card slot
}

pub struct MagicStone {
    pub stone_type: MagicStoneType,
    pub item_ref: String,  // Reference to stone item
}

impl MagicStone {
    pub fn use_on_item(
        &self,
        item: &mut Equipment,
    ) -> Result<StoneResult, StoneError> {
        match self.stone_type {
            MagicStoneType::AscensionShard => {
                self.apply_ascension_shard(item)
            }
            MagicStoneType::EssenceFragment => {
                self.apply_essence_fragment(item)
            }
            // ... other stone types
            _ => Err(StoneError::InvalidStoneType),
        }
    }
    
    fn apply_ascension_shard(
        &self,
        item: &mut Equipment,
    ) -> Result<StoneResult, StoneError> {
        if item.rarity != ItemRarity::Common {
            return Err(StoneError::InvalidRarity);
        }
        
        item.rarity = ItemRarity::Uncommon;
        item.add_random_attribute(1);
        item.update_gold_cost();
        
        Ok(StoneResult::Success {
            new_rarity: item.rarity,
            attributes_added: 1,
        })
    }
    
    // ... other stone application methods
}

pub struct StoneDropTable {
    pub stone_type: MagicStoneType,
    pub elite_weight: u32,
    pub boss_weight: u32,
}

impl Monster {
    pub fn drop_magic_stone(&self) -> Option<MagicStoneType> {
        if !self.is_elite() && !self.is_boss() {
            return None;
        }
        
        let drop_chance = if self.is_boss() {
            self.get_boss_drop_chance()
        } else {
            self.get_elite_drop_chance()
        };
        
        if thread_rng().gen::<f32>() > drop_chance {
            return None;
        }
        
        let drop_table = if self.is_boss() {
            BOSS_STONE_DROP_TABLE
        } else {
            ELITE_STONE_DROP_TABLE
        };
        
        Some(self.select_random_stone(drop_table))
    }
}
```

### Performance Optimizations

1. **Stone Lookup**: Use enum-based lookup instead of string matching
2. **Effect Caching**: Cache stone effect calculations
3. **Batch Processing**: Process multiple stone uses in batches
4. **Lazy Evaluation**: Only calculate effects when stone is used

## Integration with Other Systems

### Item System

- Stones modify item properties (rarity, attributes, tier)
- Item value recalculated after stone use
- Item serialization includes stone modifications

### Auction House System

- Stones can be used as currency
- Stone value conversion to gold
- Stone-for-item trades

### Economy System

- Stones create secondary economy
- Stone value affects gold economy
- Stone trading creates market dynamics

### Drop System

- Elite and boss monsters drop stones
- Drop rates affect stone availability
- Stone rarity affects economy

## Testing Strategy

### Unit Tests

- Test each stone type effect
- Test stone usage validation
- Test rarity transformations
- Test attribute additions
- Test tier increases

### Integration Tests

- Test stone drop from elites/bosses
- Test stone usage on items
- Test auction house stone currency
- Test stone trading

### Balance Tests

- Test drop rate balance
- Test stone value balance
- Test economy impact
- Test stone availability

