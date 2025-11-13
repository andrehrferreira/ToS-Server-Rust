# Economic Rebalancing System

## Overview

The economic rebalancing system automatically monitors and adjusts the game economy to maintain balance, prevent inflation, and ensure controlled scarcity of rare items. The system runs daily maintenance tasks and continuously monitors economic indicators.

## Core Principles

**Inflation Control:**
- Monitor gold drop rates
- Adjust NPC prices daily based on gold circulation
- Reduce gold drops temporarily when needed

**Resource Balance:**
- Monitor resource availability in auction house
- Purchase excess resources at low prices
- Maintain resource scarcity

**Rare Item Scarcity:**
- Limit ultra-rare items to maximum 5 units simultaneously
- Reduce drop rates when many units are for sale
- Programmed scarcity for legendary items

**Market Liquidity:**
- Monitor auction house inventory levels
- System posts items at premium prices when market is empty
- All system trade gold/currency is burned

## Economic Monitoring

**Metrics Tracked:**

```rust
pub struct EconomicMetrics {
    pub timestamp: DateTime<Utc>,
    
    // Gold circulation
    pub total_gold_in_circulation: u64,
    pub gold_dropped_last_24h: u64,
    pub gold_burned_last_24h: u64,
    pub gold_circulation_change_percent: f64,
    
    // Resource availability
    pub resources_in_auction_house: HashMap<ItemRef, ResourceAvailability>,
    
    // Rare item tracking
    pub rare_items_in_game: HashMap<ItemRef, u32>,  // Count of items in game
    pub rare_items_for_sale: HashMap<ItemRef, u32>,  // Count in auction house
    pub boss_cards_in_game: HashMap<CardId, u32>,
    
    // Market liquidity
    pub auction_house_listing_count: usize,
    pub average_listing_price: u64,
    pub market_depth: MarketDepth,
}

pub struct ResourceAvailability {
    pub total_listings: u32,
    pub average_price: u64,
    pub lowest_price: u64,
    pub abundance_threshold: u32,  // Considered abundant if listings > this
    pub is_abundant: bool,
}

pub struct MarketDepth {
    pub total_listings: usize,
    pub categories_with_listings: usize,
    pub average_price_by_category: HashMap<ItemCategory, u64>,
    pub liquidity_score: f64,  // 0.0 to 1.0
}
```

## Daily Price Adjustment (NPC Vendors)

**Process:**
1. Calculate gold circulation change from previous day
2. Adjust NPC vendor prices based on inflation rate
3. Price adjustment formula: `new_price = base_price * (1.0 + inflation_rate * adjustment_multiplier)`

**Implementation:**

```rust
pub struct PriceAdjustmentSystem {
    db: Arc<Database>,
    gold_burn_system: Arc<GoldBurnSystem>,
    npc_vendor_manager: Arc<NpcVendorManager>,
}

impl PriceAdjustmentSystem {
    pub async fn daily_price_adjustment(&self) -> Result<()> {
        // 1. Get economic metrics
        let metrics = self.calculate_economic_metrics().await?;
        
        // 2. Calculate inflation rate
        let inflation_rate = self.calculate_inflation_rate(&metrics).await?;
        
        // 3. Calculate price adjustment multiplier
        // Positive inflation = increase prices
        // Negative inflation = decrease prices (rare, but possible)
        let adjustment_multiplier = self.calculate_adjustment_multiplier(inflation_rate);
        
        // 4. Adjust all NPC vendor prices
        let vendors = self.npc_vendor_manager.get_all_vendors().await?;
        
        for mut vendor in vendors {
            for mut item in &mut vendor.inventory {
                let base_price = item.base_price;
                let new_price = (base_price as f64 * (1.0 + inflation_rate * adjustment_multiplier)) as u64;
                
                item.current_price = new_price;
                item.last_price_adjustment = Utc::now();
            }
            
            self.npc_vendor_manager.update_vendor(vendor).await?;
        }
        
        // 5. Log adjustment
        self.log_price_adjustment(inflation_rate, adjustment_multiplier).await?;
        
        Ok(())
    }
    
    fn calculate_inflation_rate(&self, metrics: &EconomicMetrics) -> f64 {
        // Inflation rate based on gold circulation change
        // Positive = inflation (too much gold)
        // Negative = deflation (too little gold)
        metrics.gold_circulation_change_percent / 100.0
    }
    
    fn calculate_adjustment_multiplier(&self, inflation_rate: f64) -> f64 {
        // Adjustment multiplier (0.1 to 0.5)
        // Higher inflation = faster price adjustment
        if inflation_rate > 0.05 {
            0.5  // High inflation, aggressive adjustment
        } else if inflation_rate > 0.02 {
            0.3  // Moderate inflation
        } else if inflation_rate > 0.0 {
            0.1  // Low inflation, gentle adjustment
        } else {
            0.0  // No adjustment needed
        }
    }
}
```

## Resource Abundance Management

**Process:**
1. Monitor all resources in auction house
2. Identify abundant resources (listings > threshold)
3. Purchase lowest-priced listings automatically
4. Burn all gold used for purchases

**Implementation:**

```rust
pub struct ResourceRebalancingSystem {
    auction_house: Arc<AuctionHouse>,
    gold_burn_system: Arc<GoldBurnSystem>,
    db: Arc<Database>,
}

impl ResourceRebalancingSystem {
    pub async fn check_resource_abundance(&self) -> Result<()> {
        // 1. Get all resource listings from auction house
        let resource_listings = self.auction_house
            .get_listings_by_category(ItemCategory::Resource)
            .await?;
        
        // 2. Group by resource type
        let mut resources_by_type: HashMap<ItemRef, Vec<AuctionHouseListing>> = HashMap::new();
        for listing in resource_listings {
            resources_by_type
                .entry(listing.item.ref())
                .or_insert_with(Vec::new)
                .push(listing);
        }
        
        // 3. Check each resource type for abundance
        for (resource_ref, listings) in resources_by_type {
            let availability = self.calculate_resource_availability(&listings);
            
            if availability.is_abundant {
                // Resource is abundant, purchase lowest-priced listings
                self.purchase_excess_resources(resource_ref, &listings).await?;
            }
        }
        
        Ok(())
    }
    
    fn calculate_resource_availability(
        &self,
        listings: &[AuctionHouseListing],
    ) -> ResourceAvailability {
        let total_listings = listings.len() as u32;
        let prices: Vec<u64> = listings.iter().map(|l| l.buyout_price.unwrap_or(0)).collect();
        let average_price = prices.iter().sum::<u64>() / prices.len() as u64;
        let lowest_price = *prices.iter().min().unwrap_or(&0);
        
        // Abundance threshold: 50+ listings of same resource
        let abundance_threshold = 50;
        let is_abundant = total_listings > abundance_threshold;
        
        ResourceAvailability {
            total_listings,
            average_price,
            lowest_price,
            abundance_threshold,
            is_abundant,
        }
    }
    
    async fn purchase_excess_resources(
        &self,
        resource_ref: ItemRef,
        listings: &[AuctionHouseListing],
    ) -> Result<()> {
        // Sort by price (lowest first)
        let mut sorted_listings = listings.to_vec();
        sorted_listings.sort_by_key(|l| l.buyout_price.unwrap_or(u64::MAX));
        
        // Purchase bottom 20% of listings (lowest priced)
        let purchase_count = (sorted_listings.len() as f64 * 0.20).ceil() as usize;
        let listings_to_purchase = &sorted_listings[..purchase_count.min(sorted_listings.len())];
        
        let mut total_gold_burned = 0u64;
        
        for listing in listings_to_purchase {
            if let Some(buyout_price) = listing.buyout_price {
                // Purchase listing (system acts as buyer)
                match self.auction_house.buy_item_system(listing.id).await {
                    Ok(_) => {
                        total_gold_burned += buyout_price;
                        
                        // Item is removed from auction house and stored in system inventory
                        // (or deleted, depending on design)
                    }
                    Err(e) => {
                        // Log error but continue
                        tracing::warn!("Failed to purchase listing {}: {:?}", listing.id, e);
                    }
                }
            }
        }
        
        // Burn all gold used for purchases
        if total_gold_burned > 0 {
            self.gold_burn_system
                .burn_gold(
                    total_gold_burned,
                    GoldBurnSource::EconomicRebalancing,
                    None,  // System action, no player
                )
                .await?;
        }
        
        Ok(())
    }
}
```

## Gold Drop Rate Adjustment

**Process:**
1. Monitor gold circulation metrics
2. If inflation is high, temporarily reduce gold drop rates
3. Reduction applies to all monster drops
4. Rate returns to normal when inflation stabilizes

**Implementation:**

```rust
pub struct GoldDropAdjustmentSystem {
    current_drop_multiplier: Arc<AtomicF64>,  // 1.0 = normal, 0.5 = 50% reduction
    gold_burn_system: Arc<GoldBurnSystem>,
}

impl GoldDropAdjustmentSystem {
    pub async fn adjust_gold_drop_rate(&self) -> Result<()> {
        // 1. Calculate current inflation
        let metrics = self.calculate_economic_metrics().await?;
        let inflation_rate = metrics.gold_circulation_change_percent / 100.0;
        
        // 2. Determine drop multiplier
        let new_multiplier = if inflation_rate > 0.10 {
            // High inflation: reduce drops by 50%
            0.5
        } else if inflation_rate > 0.05 {
            // Moderate inflation: reduce drops by 25%
            0.75
        } else if inflation_rate > 0.02 {
            // Low inflation: reduce drops by 10%
            0.9
        } else {
            // Normal: no reduction
            1.0
        };
        
        // 3. Update multiplier
        self.current_drop_multiplier.store(new_multiplier, Ordering::Relaxed);
        
        // 4. Log adjustment
        tracing::info!(
            "Gold drop rate adjusted: {:.1}% (inflation: {:.2}%)",
            new_multiplier * 100.0,
            inflation_rate * 100.0
        );
        
        Ok(())
    }
    
    pub fn get_drop_multiplier(&self) -> f64 {
        self.current_drop_multiplier.load(Ordering::Relaxed)
    }
    
    pub fn apply_to_drop(&self, base_gold_amount: u64) -> u64 {
        let multiplier = self.get_drop_multiplier();
        (base_gold_amount as f64 * multiplier) as u64
    }
}
```

## Rare Item Scarcity Control

**Ultra-Rare Item Limits:**
- Maximum 5 units of same ultra-rare item in game simultaneously
- Maximum 5 units of same boss card in game simultaneously
- Drop rate reduced when limit is reached
- Drop rate further reduced when items are for sale

**Implementation:**

```rust
pub struct RareItemScarcitySystem {
    db: Arc<Database>,
    item_tracker: Arc<ItemTracker>,
    auction_house: Arc<AuctionHouse>,
}

pub struct ItemScarcityConfig {
    pub item_ref: ItemRef,
    pub max_in_game: u32,  // Maximum 5 for ultra-rare items
    pub base_drop_rate: f64,  // Base drop rate (e.g., 0.001 = 0.1%)
    pub current_drop_rate: f64,  // Current adjusted drop rate
}

impl RareItemScarcitySystem {
    pub async fn check_rare_item_limits(&self) -> Result<()> {
        // 1. Get all ultra-rare items configuration
        let ultra_rare_items = self.get_ultra_rare_items_config().await?;
        
        for mut config in ultra_rare_items {
            // 2. Count items currently in game
            let in_game_count = self.item_tracker
                .count_items_in_game(config.item_ref)
                .await?;
            
            // 3. Count items for sale in auction house
            let for_sale_count = self.auction_house
                .count_listings_by_item(config.item_ref)
                .await?;
            
            // 4. Calculate drop rate adjustment
            let drop_rate_multiplier = self.calculate_drop_rate_multiplier(
                in_game_count,
                for_sale_count,
                config.max_in_game,
            );
            
            // 5. Update drop rate
            config.current_drop_rate = config.base_drop_rate * drop_rate_multiplier;
            
            // 6. Save configuration
            self.update_item_scarcity_config(config).await?;
            
            // 7. If limit reached, prevent new drops
            if in_game_count >= config.max_in_game {
                tracing::warn!(
                    "Ultra-rare item {} reached limit: {}/{}",
                    config.item_ref,
                    in_game_count,
                    config.max_in_game
                );
            }
        }
        
        Ok(())
    }
    
    fn calculate_drop_rate_multiplier(
        &self,
        in_game_count: u32,
        for_sale_count: u32,
        max_in_game: u32,
    ) -> f64 {
        let mut multiplier = 1.0;
        
        // Reduce drop rate as we approach limit
        let usage_ratio = in_game_count as f64 / max_in_game as f64;
        if usage_ratio >= 1.0 {
            // Limit reached, no more drops
            return 0.0;
        } else if usage_ratio >= 0.8 {
            // Near limit, reduce by 80%
            multiplier *= 0.2;
        } else if usage_ratio >= 0.6 {
            // Approaching limit, reduce by 50%
            multiplier *= 0.5;
        } else if usage_ratio >= 0.4 {
            // Getting full, reduce by 25%
            multiplier *= 0.75;
        }
        
        // Further reduce if many are for sale
        let sale_ratio = for_sale_count as f64 / max_in_game as f64;
        if sale_ratio >= 0.6 {
            // Many for sale, reduce drop rate by additional 50%
            multiplier *= 0.5;
        } else if sale_ratio >= 0.4 {
            // Some for sale, reduce by 25%
            multiplier *= 0.75;
        }
        
        multiplier
    }
    
    pub async fn can_drop_item(&self, item_ref: ItemRef) -> Result<bool> {
        let config = self.get_item_scarcity_config(item_ref).await?;
        
        if let Some(config) = config {
            let in_game_count = self.item_tracker
                .count_items_in_game(item_ref)
                .await?;
            
            // Cannot drop if limit reached
            if in_game_count >= config.max_in_game {
                return Ok(false);
            }
        }
        
        Ok(true)
    }
    
    pub async fn get_drop_rate(&self, item_ref: ItemRef) -> Result<f64> {
        let config = self.get_item_scarcity_config(item_ref).await?;
        
        if let Some(config) = config {
            Ok(config.current_drop_rate)
        } else {
            // No scarcity config, use base drop rate
            Ok(0.01)  // Default 1%
        }
    }
}
```

## Boss Card Scarcity

**Boss Card Limits:**
- Maximum 5 units of same boss card in game simultaneously
- Same scarcity system as ultra-rare items

```rust
impl RareItemScarcitySystem {
    pub async fn check_boss_card_limits(&self) -> Result<()> {
        // Similar to rare item limits but for boss cards
        let boss_cards = self.get_boss_cards_config().await?;
        
        for mut card_config in boss_cards {
            let in_game_count = self.item_tracker
                .count_cards_in_game(card_config.card_id)
                .await?;
            
            let for_sale_count = self.auction_house
                .count_card_listings(card_config.card_id)
                .await?;
            
            let drop_rate_multiplier = self.calculate_drop_rate_multiplier(
                in_game_count,
                for_sale_count,
                5,  // Max 5 boss cards
            );
            
            card_config.current_drop_rate = card_config.base_drop_rate * drop_rate_multiplier;
            self.update_card_scarcity_config(card_config).await?;
        }
        
        Ok(())
    }
}
```

## Market Liquidity Management

**Process:**
1. Monitor auction house inventory levels
2. If market is empty (low liquidity), system posts items
3. Prices set higher than recent average prices
4. All gold/currency from system sales is burned

**Implementation:**

```rust
pub struct MarketLiquiditySystem {
    auction_house: Arc<AuctionHouse>,
    gold_burn_system: Arc<GoldBurnSystem>,
    item_generator: Arc<ItemGenerator>,
    db: Arc<Database>,
}

impl MarketLiquiditySystem {
    pub async fn maintain_market_liquidity(&self) -> Result<()> {
        // 1. Calculate market liquidity score
        let liquidity_score = self.calculate_liquidity_score().await?;
        
        // 2. If liquidity is low, add items to market
        if liquidity_score < 0.3 {
            // Low liquidity, add items
            self.add_items_to_market().await?;
        }
        
        Ok(())
    }
    
    async fn calculate_liquidity_score(&self) -> Result<f64> {
        // Get market depth
        let market_depth = self.auction_house.get_market_depth().await?;
        
        // Calculate liquidity score (0.0 to 1.0)
        let total_listings = market_depth.total_listings;
        let categories_with_listings = market_depth.categories_with_listings;
        
        // Normalize scores
        let listing_score = (total_listings as f64 / 1000.0).min(1.0);  // 1000+ listings = 1.0
        let category_score = (categories_with_listings as f64 / 10.0).min(1.0);  // 10+ categories = 1.0
        
        // Combined score
        let liquidity_score = (listing_score * 0.6 + category_score * 0.4);
        
        Ok(liquidity_score)
    }
    
    async fn add_items_to_market(&self) -> Result<()> {
        // 1. Identify categories with low listings
        let market_depth = self.auction_house.get_market_depth().await?;
        
        for (category, avg_price) in &market_depth.average_price_by_category {
            let category_listings = self.auction_house
                .count_listings_by_category(*category)
                .await?;
            
            // If category has less than 10 listings, add items
            if category_listings < 10 {
                self.add_category_items_to_market(*category, *avg_price).await?;
            }
        }
        
        Ok(())
    }
    
    async fn add_category_items_to_market(
        &self,
        category: ItemCategory,
        recent_avg_price: u64,
    ) -> Result<()> {
        // 1. Generate items for this category
        let items_to_add = self.generate_market_items(category, 5).await?;  // Add 5 items
        
        for item in items_to_add {
            // 2. Calculate premium price (20% above recent average)
            let premium_price = (recent_avg_price as f64 * 1.20) as u64;
            
            // 3. List item on auction house (system as seller)
            let listing_id = self.auction_house
                .list_item_system(item, premium_price)
                .await?;
            
            // When item is purchased, gold goes to system account
            // All system account gold is burned periodically
        }
        
        Ok(())
    }
    
    pub async fn burn_system_trade_proceeds(&self) -> Result<()> {
        // Get all gold from system trades
        let system_gold = self.get_system_account_balance().await?;
        
        if system_gold > 0 {
            // Burn all system trade proceeds
            self.gold_burn_system
                .burn_gold(
                    system_gold,
                    GoldBurnSource::EconomicRebalancing,
                    None,
                )
                .await?;
            
            // Reset system account
            self.reset_system_account().await?;
        }
        
        Ok(())
    }
}
```

## Scheduled Maintenance Tasks

**Daily Tasks (00:00:00):**
1. Calculate economic metrics
2. Adjust NPC vendor prices
3. Check rare item limits and adjust drop rates
4. Check boss card limits and adjust drop rates
5. Burn system trade proceeds

**Hourly Tasks:**
1. Check resource abundance and purchase excess
2. Adjust gold drop rates if needed
3. Check market liquidity and add items if needed

**Real-Time Monitoring:**
1. Track gold drops continuously
2. Track item drops and counts
3. Track auction house listings
4. Update economic metrics

## Database Schema

```sql
-- Economic metrics history
CREATE TABLE economic_metrics_history (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    total_gold_in_circulation BIGINT NOT NULL,
    gold_dropped_24h BIGINT NOT NULL,
    gold_burned_24h BIGINT NOT NULL,
    gold_circulation_change_percent DOUBLE PRECISION NOT NULL,
    inflation_rate DOUBLE PRECISION NOT NULL,
    INDEX idx_timestamp (timestamp)
);

-- Item scarcity configuration
CREATE TABLE item_scarcity_config (
    item_ref VARCHAR(100) PRIMARY KEY,
    max_in_game INTEGER NOT NULL DEFAULT 5,
    base_drop_rate DOUBLE PRECISION NOT NULL,
    current_drop_rate DOUBLE PRECISION NOT NULL,
    last_adjustment TIMESTAMP NOT NULL
);

-- Boss card scarcity configuration
CREATE TABLE boss_card_scarcity_config (
    card_id VARCHAR(100) PRIMARY KEY,
    max_in_game INTEGER NOT NULL DEFAULT 5,
    base_drop_rate DOUBLE PRECISION NOT NULL,
    current_drop_rate DOUBLE PRECISION NOT NULL,
    last_adjustment TIMESTAMP NOT NULL
);

-- System account (for market liquidity trades)
CREATE TABLE system_account (
    id INTEGER PRIMARY KEY DEFAULT 1,
    gold_balance BIGINT NOT NULL DEFAULT 0,
    last_updated TIMESTAMP NOT NULL
);

-- Price adjustment history
CREATE TABLE price_adjustment_history (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    inflation_rate DOUBLE PRECISION NOT NULL,
    adjustment_multiplier DOUBLE PRECISION NOT NULL,
    average_price_change_percent DOUBLE PRECISION NOT NULL
);
```

## Integration with Other Systems

**Monster Drop System:**
```rust
// When monster drops gold
let base_gold = monster.calculate_gold_drop();
let adjusted_gold = gold_drop_adjustment_system.apply_to_drop(base_gold);
monster.drop_gold(adjusted_gold);
```

**Item Drop System:**
```rust
// When checking if item can drop
if let Some(drop_rate) = rare_item_scarcity_system.get_drop_rate(item_ref).await? {
    if can_drop_item(item_ref).await? && rng.gen::<f64>() < drop_rate {
        // Item can drop
    }
}
```

**Auction House:**
- System can list items (for liquidity)
- System can purchase items (for rebalancing)
- All system trades burn gold/currency

## See Also

- [GOLD_BURN.md](./GOLD_BURN.md) - Gold burn system
- [AUCTION_HOUSE.md](./AUCTION_HOUSE.md) - Auction house integration
- [GOLD.md](../core/GOLD.md) - Basic gold economy system

