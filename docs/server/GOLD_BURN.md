# Gold Burn System

## Overview

The gold burn system tracks all gold removed from circulation and accumulates it weekly for the lottery prize pool. This system prevents inflation by removing gold from the economy through various transactions.

## Gold Burn Sources

**All gold removed from circulation is logged:**

1. **NPC Purchases**
   - Gold paid to NPCs for items
   - Gold paid for services (repair, crafting, etc.)
   - Gold paid for teleportation

2. **Taxes and Fees**
   - Auction house listing fees
   - Auction house commission (house cut)
   - Magic stone market listing fees
   - Bulk order creation fees
   - Lottery ticket purchases (50% burned)

3. **Auction House**
   - House commission percentage on sales
   - Listing fees

4. **Magic Stone Market**
   - Listing fees for buy/sell orders

5. **Bulk Orders**
   - Failed order penalties (20% of order value)
   - Timeout penalties (if order not completed)

6. **Lottery**
   - 50% of ticket purchase price

7. **Economic Rebalancing**
   - System purchases of excess resources
   - System trade proceeds

## Gold Burn Log

**Database Structure:**
```rust
pub struct GoldBurnLog {
    pub id: u64,
    pub timestamp: DateTime<Utc>,
    pub amount: u64,
    pub source: GoldBurnSource,
    pub player_id: Option<PlayerId>,
    pub transaction_id: Option<String>,
    pub week_start: DateTime<Utc>,  // Week start date for aggregation
}

pub enum GoldBurnSource {
    NpcPurchase,
    NpcService,
    AuctionHouseFee,
    AuctionHouseCommission,
    MagicStoneMarketFee,
    BulkOrderFee,
    BulkOrderPenalty,
    LotteryTicket,
    EconomicRebalancing,
    Other(String),
}
```

## Weekly Aggregation

**Process:**
1. All gold burns are logged with `week_start` timestamp
2. At end of week (Sunday 23:59:59), aggregate all burns for the week
3. Calculate lottery prize: `1% of total burned gold`
4. Remaining 99% is permanently deleted from database
5. Lottery prize pool is set for the week

**Aggregation Query:**
```sql
-- Get total burned gold for current week
SELECT SUM(amount) as total_burned
FROM gold_burn_log
WHERE week_start = CURRENT_WEEK_START;

-- Calculate lottery prize (1% of burned)
UPDATE lottery_pool
SET prize_pool = (SELECT SUM(amount) * 0.01 FROM gold_burn_log WHERE week_start = CURRENT_WEEK_START)
WHERE week_id = CURRENT_WEEK_ID;

-- Delete old burn logs (keep only current week for lottery calculation)
DELETE FROM gold_burn_log
WHERE week_start < CURRENT_WEEK_START - INTERVAL '1 week';
```

## Implementation

```rust
pub struct GoldBurnSystem {
    db: Arc<Database>,
    current_week_start: DateTime<Utc>,
}

impl GoldBurnSystem {
    pub async fn burn_gold(
        &self,
        amount: u64,
        source: GoldBurnSource,
        player_id: Option<PlayerId>,
    ) -> Result<()> {
        // Log the burn
        let log = GoldBurnLog {
            id: self.generate_id(),
            timestamp: Utc::now(),
            amount,
            source,
            player_id,
            transaction_id: None,
            week_start: self.current_week_start,
        };
        
        self.db.insert_gold_burn_log(log).await?;
        
        // Gold is already removed from player/system
        // This just tracks it for lottery calculation
        Ok(())
    }
    
    pub async fn get_weekly_total(&self) -> Result<u64> {
        self.db
            .query_gold_burn_total(self.current_week_start)
            .await
    }
    
    pub async fn get_weekly_total_for_date(&self, week_start: DateTime<Utc>) -> Result<u64> {
        self.db
            .query_gold_burn_total(week_start)
            .await
    }
    
    pub async fn calculate_lottery_prize(&self) -> Result<u64> {
        let total_burned = self.get_weekly_total().await?;
        Ok((total_burned as f64 * 0.01) as u64) // 1% of burned gold
    }
    
    pub async fn cleanup_old_logs(&self) -> Result<()> {
        let cutoff = self.current_week_start - Duration::days(7);
        self.db.delete_gold_burn_logs_before(cutoff).await?;
        Ok(())
    }
}
```

## Database Schema

```sql
CREATE TABLE gold_burn_log (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    amount BIGINT NOT NULL,
    source VARCHAR(50) NOT NULL,
    player_id BIGINT,
    transaction_id VARCHAR(100),
    week_start TIMESTAMP NOT NULL,
    INDEX idx_week_start (week_start)
);
```

## Integration

All systems that remove gold from circulation must log to the gold burn system:

```rust
// Example: NPC purchase
player.remove_gold(purchase_price).await?;
gold_burn_system.burn_gold(
    purchase_price,
    GoldBurnSource::NpcPurchase,
    Some(player.id),
).await?;
```

## See Also

- [LOTTERY.md](./LOTTERY.md) - Lottery system that uses gold burn data
- [AUCTION_HOUSE.md](./AUCTION_HOUSE.md) - Auction house fees and commissions
- [MAGIC_STONE_MARKET.md](./MAGIC_STONE_MARKET.md) - Magic stone market fees
- [BULK_ORDERS.md](./BULK_ORDERS.md) - Bulk order penalties
- [ECONOMIC_REBALANCING.md](./ECONOMIC_REBALANCING.md) - Economic rebalancing burns

