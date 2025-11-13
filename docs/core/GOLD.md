# Gold Economy System Documentation

## Overview

Gold is the primary currency of the game, serving as the foundation of the economy. It is directly related to crafting, housing, and many other game systems. Gold can be obtained through various means including monster drops, quests, events, trade, and auction house. When a player dies, they drop a portion of the gold in their inventory. Gold can be stored in banks located in cities to prevent loss. Every item has a base value for selling to NPCs, calculated based on tier, quantity, and attributes. Rarer attributes increase item value. Administrators can also set fixed prices for items. Crafting and repair costs are proportional to item value, ensuring rarer items cost more to craft and repair.

## Key Features

- **Primary Currency**: Gold is the base currency for all economic transactions
- **Multiple Acquisition Methods**: Monster drops, quests, events, trade, auction house
- **Death Penalty**: Players drop a portion of gold on death
- **Bank Storage**: Gold can be stored in city banks to prevent loss
- **Dynamic Item Value**: Item value calculated from tier, quantity, and attributes
- **Rare Attribute Bonus**: Rarer attributes increase item value significantly
- **Fixed Prices**: Administrators can set fixed prices for items
- **Proportional Costs**: Crafting and repair costs proportional to item value

## Gold Item

### GoldCoin Item

**TypeScript:**
```typescript
export class GoldCoin extends Stackable {
    public override Namespace: string = "GoldCoin";
    public override Name: string = "Gold Coin";
    public override GoldCost: number = 1; // Base value: 1 gold = 1 gold
    public override Weight: number = 0.01; // Very light
}
```

**Properties:**
- Stackable item (can accumulate in inventory)
- Base value: 1 gold coin = 1 gold
- Very light weight (0.01 per coin)
- Can be stored in inventory or bank

## Gold Acquisition

### 1. Monster Drops

**Drop Mechanics:**
- Monsters drop gold coins when killed
- Drop amount varies by monster level and type
- Higher level monsters drop more gold
- Boss monsters drop significantly more gold

**Drop Examples:**
```typescript
// Skeleton (low level)
this.loot.dropChance(GoldCoin, 100, 5, 40); // 100% chance, 5-40 gold

// Skeleton Knight (mid level)
this.loot.dropChance(GoldCoin, 100, 100, 200); // 100% chance, 100-200 gold

// Undertaker (boss)
this.loot.dropChance(GoldCoin, 100, 200, 500); // 100% chance, 200-500 gold
```

**Drop Factors:**
- Monster level
- Monster type (normal, elite, boss)
- Area difficulty
- Random variation

### 2. Quest Rewards

**Quest Gold Rewards:**
- Quests provide gold as completion rewards
- Reward amount varies by quest difficulty
- Tutorial quests: 100-5000 gold
- Main quests: Higher rewards
- Daily quests: Moderate rewards

**Example Quest Rewards:**
```typescript
// Tutorial quests
{ Item: GoldCoin, Quantity: 100 }
{ Item: GoldCoin, Quantity: 500 }
{ Item: GoldCoin, Quantity: 1500 }
{ Item: GoldCoin, Quantity: 5000 }
```

### 3. Event Rewards

**Event Gold:**
- Special events provide gold rewards
- Event participation rewards
- Event completion bonuses
- Seasonal event rewards

### 4. Trade

**Player-to-Player Trade:**
- Direct gold transfer between players
- Trade window allows gold exchange
- Secure trade system prevents scams
- Trade fees (optional)

### 5. Auction House

**Auction House Sales:**
- Players can sell items on auction house
- Gold received when item sells
- Auction house fees (percentage of sale)
- Buyout and bidding options

### 6. NPC Sales

**Selling Items to NPCs:**
- Items can be sold to NPC vendors
- Value based on item's `GoldCost` property
- Stackable items: `totalValue = GoldCost * quantity`
- Non-stackable items: `totalValue = GoldCost`

**Implementation:**
```typescript
public async sellItem(ref: string, amount: number = 1) {
    const item = this.inventory.getItem(ref);
    
    if (item) {
        const totalGainGold = item.GoldCost * amount;
        
        if (await this.removeItem(ref, amount)) {
            await this.addItem("GoldCoin", totalGainGold, "sell");
            this.save();
            this.saveToDatabase();
        }
    }
}
```

## Gold Usage

### 1. Crafting

**Crafting Costs:**
- Crafting requires gold payment
- Cost is proportional to item value
- Formula: `craftingCost = baseItemValue * craftingMultiplier`
- Crafting multiplier: 0.5 (50% of item value)

**Implementation:**
```typescript
const baseItem = Items.createItemByClass(recipe.resultItem, this.name);
const craftingCost = (baseItem.GoldCost / 2) * amount;

if (this.getGoldCoins() < craftingCost) {
    return { success: false, error: "Insufficient gold" };
}

await this.removeGoldCoins(craftingCost);
// ... proceed with crafting
```

### 2. Repair

**Repair Costs:**
- Repair costs based on item value and missing durability
- Formula: `repairCost = (missingDurability / maxDurability) * itemValue * repairMultiplier`
- Repair multiplier: 0.1 to 0.5 (10-50% of item value per full repair)
- Higher value items cost more to repair

**See [DURABILITY.md](../items/DURABILITY.md) for detailed repair system documentation.**

### 3. Housing

**Housing Costs:**
- House purchase requires gold
- House upgrades cost gold
- Property taxes (if implemented)
- Furniture and decoration purchases

### 4. NPC Purchases

**Buying from NPCs:**
- NPCs sell items for gold
- Prices set by NPC vendor lists
- Vendor prices: `baseItem.GoldCost * vendorMultiplier`
- Vendor multiplier: Typically 3x base cost

**Implementation:**
```typescript
public async buyItem(namespace: string, amount: number) {
    const vendorItem = this.vendorList.data.find(ref => ref.ns === namespace);
    const total = vendorItem.g * amount; // Vendor price
    
    if (this.getGoldCoins() >= total) {
        await this.removeGoldCoins(total);
        // ... add item to inventory
    }
}
```

### 5. Guild Creation

**Guild Costs:**
- Creating a guild costs gold
- Default cost: 20,000 gold
- Can be configured per server

**Implementation:**
```typescript
public createGuildCost = 20000;

if (this.getGoldCoins() >= this.createGuildCost) {
    await this.removeGoldCoins(this.createGuildCost);
    // ... create guild
}
```

### 6. Other Uses

- Skill training (if implemented)
- Teleportation fees
- Service fees (NPC artisans, etc.)
- Event participation fees
- Market fees (auction house, trade)

## Death and Gold Loss

### Death Penalty

**Gold Drop on Death:**
- When a player dies, they drop a portion of gold
- Only gold in inventory is at risk
- Gold in bank is safe
- Drop percentage: Configurable (typically 10-50%)

**Drop Calculation:**
```typescript
public onDeath() {
    const goldInInventory = this.getGoldCoins();
    const dropPercentage = 0.25; // 25% drop rate
    const goldToDrop = Math.floor(goldInInventory * dropPercentage);
    
    if (goldToDrop > 0) {
        await this.removeGoldCoins(goldToDrop);
        // Create gold coin drop at death location
        this.createGoldDrop(goldToDrop);
    }
}
```

**Gold Drop Properties:**
- Dropped gold appears as GoldCoin items
- Can be picked up by any player
- Despawns after timeout (if configured)
- Visible to all players in area

### Bank Storage

**City Banks:**
- Banks located in cities/towns
- Players can deposit gold for safe storage
- Gold in bank is not lost on death
- Bank storage has limits (if configured)

**Bank Operations:**
```typescript
// Deposit gold
public async depositGold(amount: number) {
    if (this.getGoldCoins() >= amount) {
        await this.removeGoldCoins(amount);
        this.bankGold += amount;
        this.save();
    }
}

// Withdraw gold
public async withdrawGold(amount: number) {
    if (this.bankGold >= amount) {
        this.bankGold -= amount;
        await this.addItem("GoldCoin", amount, "withdraw");
        this.save();
    }
}
```

**Bank Benefits:**
- Safe storage (no death penalty)
- Prevents gold loss
- Can be accessed from any bank in any city
- Account-wide storage

## Item Value Calculation

### Base Value System

**Every Item Has GoldCost:**
- All items have a `GoldCost` property
- Base value for selling to NPCs
- Used for crafting/repair cost calculations
- Can be set manually or calculated dynamically

### Value Calculation Factors

**1. Tier-Based Value:**
- Higher tier items have higher base values
- Tier multiplier: Increases exponentially
- Formula: `baseValue = tierBaseValue * tierMultiplier`

**Tier Base Values:**
- T0: 100-300 gold
- T1: 300-600 gold
- T2: 600-1200 gold
- T3: 1200-2000 gold
- T4: 2000-3000 gold
- T5: 3000-5000 gold
- T6: 5000-8000 gold
- T7: 8000-12000 gold
- T8: 12000-20000 gold

**2. Quantity (Stackable Items):**
- Stackable items: Value multiplied by quantity
- Formula: `totalValue = GoldCost * quantity`
- Example: 100 gold coins × 50 = 5000 gold total

**3. Attribute Value:**
- Items with attributes have increased value
- Each attribute adds value based on its type and rarity
- Rarer attributes add significantly more value

**Attribute Value Calculation:**
```typescript
function calculateAttributeValue(item: Equipament): number {
    let attributeValue = 0;
    
    for (const attr of item.Attributes) {
        const baseAttrValue = getAttributeBaseValue(attr.type);
        const rarityMultiplier = getRarityMultiplier(attr.rarity);
        attributeValue += baseAttrValue * rarityMultiplier;
    }
    
    return attributeValue;
}

function getRarityMultiplier(rarity: AttributeRarity): number {
    switch (rarity) {
        case AttributeRarity.Common: return 1.0;
        case AttributeRarity.Uncommon: return 1.5;
        case AttributeRarity.Rare: return 2.5;
        case AttributeRarity.Epic: return 5.0;
        case AttributeRarity.Legendary: return 10.0;
        default: return 1.0;
    }
}
```

**4. Rarity Bonus:**
- Item rarity affects base value
- Higher rarity = higher value multiplier

**Rarity Multipliers:**
- Common: 1.0x
- Uncommon: 1.3x
- Rare: 1.5x
- Magic: 2.0x
- Legendary: 3.0x
- Unique: 5.0x

**5. Durability Factor:**
- Current durability affects sell value
- Formula: `sellValue = baseValue * (currentDurability / maxDurability)`
- Broken items (0 durability) have minimal value

### Final Value Calculation

**Complete Formula:**
```typescript
function calculateItemValue(item: Item): number {
    // Base value from tier
    let baseValue = getTierBaseValue(item.Tier);
    
    // Rarity multiplier
    baseValue *= getRarityMultiplier(item.Rarity);
    
    // Attribute value
    if (item instanceof Equipament) {
        baseValue += calculateAttributeValue(item);
    }
    
    // Quantity (for stackable items)
    if (item instanceof Stackable) {
        baseValue *= item.Amount;
    }
    
    // Durability factor (for equipment)
    if (item instanceof Equipament) {
        const durabilityFactor = item.Durability / item.MaxDurability;
        baseValue *= durabilityFactor;
    }
    
    // Administrator fixed price (if set)
    if (item.FixedPrice && item.FixedPrice > 0) {
        return item.FixedPrice;
    }
    
    return Math.floor(baseValue);
}
```

### Administrator Fixed Prices

**Fixed Price System:**
- Administrators can set fixed prices for items
- Overrides calculated value
- Useful for quest items, special items
- Can be set via admin panel/API

**Implementation:**
```typescript
export abstract class Item {
    public GoldCost: number = 0;
    public FixedPrice?: number; // Optional fixed price
    
    public getSellValue(): number {
        if (this.FixedPrice && this.FixedPrice > 0) {
            return this.FixedPrice;
        }
        return this.calculateDynamicValue();
    }
}
```

## Proportional Cost System

### Crafting Cost Proportionality

**Crafting Cost Formula:**
```typescript
const baseItem = Items.createItemByClass(recipe.resultItem, this.name);
const craftingCost = (baseItem.GoldCost / 2) * amount;
```

**Proportionality:**
- Cost is 50% of final item value
- Rarer items cost more to craft
- Ensures economic balance
- Prevents crafting exploits

**Example:**
- Common Iron Sword (T3): Base value 600 gold → Crafting cost: 300 gold
- Legendary Mithril Sword (T7): Base value 10000 gold → Crafting cost: 5000 gold

### Repair Cost Proportionality

**Repair Cost Formula:**
```typescript
const missingDurability = item.MaxDurability - item.Durability;
const durabilityPercentage = missingDurability / item.MaxDurability;
const repairCost = item.GoldCost * durabilityPercentage * repairMultiplier;
```

**Proportionality:**
- Cost proportional to item value
- Cost proportional to missing durability
- Higher value items cost more to repair
- Ensures repair costs scale with item value

**Example:**
- Common Iron Sword (600 gold, 50% missing): Repair cost = 600 × 0.5 × 0.2 = 60 gold
- Legendary Mithril Sword (10000 gold, 50% missing): Repair cost = 10000 × 0.5 × 0.2 = 1000 gold

## Gold Management

### Player Gold Methods

**Get Gold:**
```typescript
public getGoldCoins(): number {
    const goldcoin = this.inventory.getItemByNamespace("GoldCoin");
    return (goldcoin) ? goldcoin.Amount : 0;
}
```

**Remove Gold:**
```typescript
public async removeGoldCoins(amount: number): Promise<boolean> {
    const goldcoin = this.inventory.getItemByNamespace("GoldCoin");
    const slot = this.inventory.getSlotByItemNamespace("GoldCoin");
    
    if (goldcoin && goldcoin.Amount >= amount && slot !== null) {
        await this.inventory.changeAmount(slot, goldcoin.Amount - amount);
        return true;
    }
    
    return false;
}
```

**Add Gold:**
```typescript
public async addItem(baseItemName: string, amount: number = 1, context: string = "add") {
    // ... creates GoldCoin items and adds to inventory
    await this.addItem("GoldCoin", amount, context);
}
```

### Gold Storage

**Inventory Storage:**
- Gold stored as GoldCoin items in inventory
- Subject to weight (0.01 per coin)
- Subject to death penalty
- Limited by inventory capacity

**Bank Storage:**
- Gold stored in bank account
- Not subject to weight
- Not subject to death penalty
- Can be accessed from any city bank
- Account-wide storage

## Rust Implementation Considerations

### Data Structures

```rust
pub struct Gold {
    pub inventory: u64,  // Gold in inventory (as GoldCoin items)
    pub bank: u64,       // Gold in bank
}

impl Gold {
    pub fn new() -> Self {
        Self {
            inventory: 0,
            bank: 0,
        }
    }
    
    pub fn total(&self) -> u64 {
        self.inventory + self.bank
    }
    
    pub fn add_inventory(&mut self, amount: u64) {
        self.inventory += amount;
    }
    
    pub fn remove_inventory(&mut self, amount: u64) -> Result<(), GoldError> {
        if self.inventory >= amount {
            self.inventory -= amount;
            Ok(())
        } else {
            Err(GoldError::InsufficientGold)
        }
    }
    
    pub fn deposit(&mut self, amount: u64) -> Result<(), GoldError> {
        self.remove_inventory(amount)?;
        self.bank += amount;
        Ok(())
    }
    
    pub fn withdraw(&mut self, amount: u64) -> Result<(), GoldError> {
        if self.bank >= amount {
            self.bank -= amount;
            self.inventory += amount;
            Ok(())
        } else {
            Err(GoldError::InsufficientGold)
        }
    }
    
    pub fn calculate_death_loss(&self, drop_percentage: f32) -> u64 {
        (self.inventory as f32 * drop_percentage) as u64
    }
}

pub struct ItemValue {
    pub base_value: u64,
    pub tier_multiplier: f32,
    pub rarity_multiplier: f32,
    pub attribute_value: u64,
    pub durability_factor: f32,
    pub fixed_price: Option<u64>,
}

impl ItemValue {
    pub fn calculate(&self) -> u64 {
        if let Some(fixed) = self.fixed_price {
            return fixed;
        }
        
        let mut value = self.base_value as f32;
        value *= self.tier_multiplier;
        value *= self.rarity_multiplier;
        value += self.attribute_value as f32;
        value *= self.durability_factor;
        
        value as u64
    }
}
```

### Gold Operations

```rust
impl Player {
    pub fn get_gold(&self) -> u64 {
        // Get gold from GoldCoin items in inventory
        self.inventory
            .get_item_by_namespace("GoldCoin")
            .map(|item| item.amount)
            .unwrap_or(0)
    }
    
    pub async fn remove_gold(&mut self, amount: u64) -> Result<(), GoldError> {
        let gold_item = self.inventory.get_item_by_namespace("GoldCoin");
        
        if let Some(gold) = gold_item {
            if gold.amount >= amount {
                self.inventory.change_amount(gold.slot_id, gold.amount - amount).await?;
                Ok(())
            } else {
                Err(GoldError::InsufficientGold)
            }
        } else {
            Err(GoldError::InsufficientGold)
        }
    }
    
    pub async fn add_gold(&mut self, amount: u64) -> Result<(), InventoryError> {
        self.inventory.add_item("GoldCoin", amount).await
    }
    
    pub fn on_death(&mut self, drop_percentage: f32) {
        let gold_to_drop = self.calculate_death_loss(drop_percentage);
        
        if gold_to_drop > 0 {
            let _ = self.remove_gold(gold_to_drop);
            // Create gold drop at death location
            self.create_gold_drop(gold_to_drop);
        }
    }
    
    pub async fn deposit_gold(&mut self, amount: u64) -> Result<(), GoldError> {
        self.remove_gold(amount).await?;
        self.bank_gold += amount;
        self.save().await?;
        Ok(())
    }
    
    pub async fn withdraw_gold(&mut self, amount: u64) -> Result<(), GoldError> {
        if self.bank_gold >= amount {
            self.bank_gold -= amount;
            self.add_gold(amount).await?;
            self.save().await?;
            Ok(())
        } else {
            Err(GoldError::InsufficientGold)
        }
    }
}
```

## Integration with Other Systems

### Crafting System

- Crafting costs use gold
- Cost proportional to item value
- See [ITEMS.md](../items/ITEMS.md) for crafting details

### Repair System

- Repair costs use gold
- Cost proportional to item value and missing durability
- See [DURABILITY.md](../items/DURABILITY.md) for repair details

### Housing System

- House purchase uses gold
- House upgrades use gold
- Property maintenance costs gold

### Trade System

- Trade uses gold as currency
- Auction house uses gold
- Player-to-player trade uses gold

### Quest System

- Quests reward gold
- Quest completion grants gold
- Daily quests provide gold rewards

### Combat System

- Monsters drop gold
- Boss monsters drop more gold
- Death causes gold loss

## Testing Strategy

### Unit Tests

- Test gold addition/removal
- Test gold value calculation
- Test death gold loss
- Test bank deposit/withdraw
- Test item value calculation
- Test proportional cost calculation

### Integration Tests

- Test crafting cost payment
- Test repair cost payment
- Test NPC buy/sell
- Test trade gold transfer
- Test auction house gold
- Test quest gold rewards

### Balance Tests

- Test gold drop rates
- Test crafting cost balance
- Test repair cost balance
- Test item value balance
- Test economy inflation/deflation

