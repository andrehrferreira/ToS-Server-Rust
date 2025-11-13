# Magic Stone Market

## Overview

The Magic Stone Market is a real-time currency exchange system similar to Path of Exile 2's currency market. Players can create buy or sell orders for Magic Stones, setting their own prices. Orders remain active until matched with a counterparty.

## Features

**Core Features:**
- Create buy orders (want to buy Magic Stones with Gold)
- Create sell orders (want to sell Magic Stones for Gold)
- Real-time price quotes
- Order matching system
- Order cancellation
- Listing fees (paid in Gold)
- Price history tracking
- Market depth visualization

## Order Types

**1. Buy Order:**
- Player wants to buy Magic Stones
- Pays Gold upfront
- Sets maximum price per stone
- Order remains until matched or cancelled

**2. Sell Order:**
- Player wants to sell Magic Stones
- Pays Magic Stones upfront
- Sets minimum price per stone
- Order remains until matched or cancelled

## Order Structure

```rust
pub struct MagicStoneOrder {
    pub id: OrderId,
    pub player_id: PlayerId,
    pub stone_type: MagicStoneType,
    pub order_type: OrderType,
    pub quantity: u32,  // Amount of stones
    pub price_per_stone: u64,  // Price in gold per stone
    pub total_value: u64,  // quantity * price_per_stone
    pub filled_quantity: u32,  // How much has been filled
    pub created_at: DateTime<Utc>,
    pub expires_at: Option<DateTime<Utc>>,  // Optional expiration
    pub status: OrderStatus,
}

pub enum OrderType {
    Buy {  // Want to buy stones with gold
        gold_paid: u64,  // Gold locked in escrow
    },
    Sell {  // Want to sell stones for gold
        stones_paid: u32,  // Stones locked in escrow
    },
}

pub enum OrderStatus {
    Active,
    PartiallyFilled,
    Filled,
    Cancelled,
    Expired,
}
```

## Creating a Buy Order

```rust
impl MagicStoneMarket {
    pub async fn create_buy_order(
        &self,
        player: &mut Player,
        stone_type: MagicStoneType,
        quantity: u32,
        max_price_per_stone: u64,
    ) -> Result<OrderId> {
        // 1. Calculate total cost
        let total_cost = quantity as u64 * max_price_per_stone;
        
        // 2. Calculate listing fee (1% of total, minimum 10 gold)
        let listing_fee = (total_cost as f64 * 0.01) as u64;
        let listing_fee = listing_fee.max(10);
        
        // 3. Verify player has enough gold
        let total_needed = total_cost + listing_fee;
        if player.get_gold() < total_needed {
            return Err(MarketError::InsufficientFunds);
        }
        
        // 4. Lock gold in escrow
        player.remove_gold(total_cost).await?; // For the order
        player.remove_gold(listing_fee).await?; // For the fee
        
        // 5. Create order
        let order = MagicStoneOrder {
            id: self.generate_order_id(),
            player_id: player.id,
            stone_type,
            order_type: OrderType::Buy {
                gold_paid: total_cost,
            },
            quantity,
            price_per_stone: max_price_per_stone,
            total_value: total_cost,
            filled_quantity: 0,
            created_at: Utc::now(),
            expires_at: None, // No expiration by default
            status: OrderStatus::Active,
        };
        
        // 6. Store order
        self.db.insert_order(order.clone()).await?;
        
        // 7. Log gold burn (listing fee)
        self.gold_burn_system
            .burn_gold(
                listing_fee,
                GoldBurnSource::MagicStoneMarketFee,
                Some(player.id),
            )
            .await?;
        
        // 8. Try to match with existing sell orders
        self.try_match_order(order.id).await?;
        
        Ok(order.id)
    }
}
```

## Creating a Sell Order

```rust
impl MagicStoneMarket {
    pub async fn create_sell_order(
        &self,
        player: &mut Player,
        stone_type: MagicStoneType,
        quantity: u32,
        min_price_per_stone: u64,
    ) -> Result<OrderId> {
        // 1. Verify player has enough stones
        if !player.has_magic_stone(stone_type, quantity) {
            return Err(MarketError::InsufficientStones);
        }
        
        // 2. Calculate listing fee (1% of total value, minimum 10 gold)
        let total_value = quantity as u64 * min_price_per_stone;
        let listing_fee = (total_value as f64 * 0.01) as u64;
        let listing_fee = listing_fee.max(10);
        
        // 3. Verify player has enough gold for fee
        if player.get_gold() < listing_fee {
            return Err(MarketError::InsufficientFunds);
        }
        
        // 4. Lock stones in escrow
        player.remove_magic_stone(stone_type, quantity).await?;
        
        // 5. Pay listing fee
        player.remove_gold(listing_fee).await?;
        
        // 6. Create order
        let order = MagicStoneOrder {
            id: self.generate_order_id(),
            player_id: player.id,
            stone_type,
            order_type: OrderType::Sell {
                stones_paid: quantity,
            },
            quantity,
            price_per_stone: min_price_per_stone,
            total_value: total_value,
            filled_quantity: 0,
            created_at: Utc::now(),
            expires_at: None,
            status: OrderStatus::Active,
        };
        
        // 7. Store order
        self.db.insert_order(order.clone()).await?;
        
        // 8. Log gold burn (listing fee)
        self.gold_burn_system
            .burn_gold(
                listing_fee,
                GoldBurnSource::MagicStoneMarketFee,
                Some(player.id),
            )
            .await?;
        
        // 9. Try to match with existing buy orders
        self.try_match_order(order.id).await?;
        
        Ok(order.id)
    }
}
```

## Order Matching

```rust
impl MagicStoneMarket {
    pub async fn try_match_order(&self, order_id: OrderId) -> Result<()> {
        let order = self.db.get_order(order_id).await?
            .ok_or(MarketError::OrderNotFound)?;
        
        match order.order_type {
            OrderType::Buy { .. } => {
                // Find matching sell orders (price <= buy order price)
                let matching_sells = self.db
                    .find_sell_orders(
                        order.stone_type,
                        order.price_per_stone,  // Max price buyer is willing to pay
                    )
                    .await?;
                
                self.match_buy_with_sells(order_id, matching_sells).await?;
            }
            OrderType::Sell { .. } => {
                // Find matching buy orders (price >= sell order price)
                let matching_buys = self.db
                    .find_buy_orders(
                        order.stone_type,
                        order.price_per_stone,  // Min price seller wants
                    )
                    .await?;
                
                self.match_sell_with_buys(order_id, matching_buys).await?;
            }
        }
        
        Ok(())
    }
    
    async fn match_buy_with_sells(
        &self,
        buy_order_id: OrderId,
        sell_orders: Vec<MagicStoneOrder>,
    ) -> Result<()> {
        let mut buy_order = self.db.get_order(buy_order_id).await?
            .ok_or(MarketError::OrderNotFound)?;
        
        let mut remaining_quantity = buy_order.quantity - buy_order.filled_quantity;
        
        for mut sell_order in sell_orders {
            if remaining_quantity == 0 {
                break;
            }
            
            let available_quantity = sell_order.quantity - sell_order.filled_quantity;
            let match_quantity = remaining_quantity.min(available_quantity);
            
            // Calculate trade value (use sell order price - seller sets the price)
            let trade_value = match_quantity as u64 * sell_order.price_per_stone;
            
            // Get players
            let buyer = self.get_player_mut(buy_order.player_id).await?;
            let seller = self.get_player_mut(sell_order.player_id).await?;
            
            // Transfer stones to buyer
            buyer.add_magic_stone(buy_order.stone_type, match_quantity).await?;
            
            // Transfer gold to seller
            seller.add_gold(trade_value).await?;
            
            // Refund excess gold to buyer if buy order price was higher
            if buy_order.price_per_stone > sell_order.price_per_stone {
                let refund = (buy_order.price_per_stone - sell_order.price_per_stone) * match_quantity as u64;
                buyer.add_gold(refund).await?;
            }
            
            // Update orders
            buy_order.filled_quantity += match_quantity;
            sell_order.filled_quantity += match_quantity;
            
            if buy_order.filled_quantity >= buy_order.quantity {
                buy_order.status = OrderStatus::Filled;
            } else {
                buy_order.status = OrderStatus::PartiallyFilled;
            }
            
            if sell_order.filled_quantity >= sell_order.quantity {
                sell_order.status = OrderStatus::Filled;
            } else {
                sell_order.status = OrderStatus::PartiallyFilled;
            }
            
            self.db.update_order(buy_order.clone()).await?;
            self.db.update_order(sell_order.clone()).await?;
            
            remaining_quantity -= match_quantity;
        }
        
        Ok(())
    }
}
```

## Real-Time Price Quotes

```rust
impl MagicStoneMarket {
    pub async fn get_current_price(&self, stone_type: MagicStoneType) -> Result<PriceQuote> {
        // Get best buy and sell orders
        let best_buy = self.db.get_best_buy_order(stone_type).await?;
        let best_sell = self.db.get_best_sell_order(stone_type).await?;
        
        Ok(PriceQuote {
            stone_type,
            best_buy_price: best_buy.map(|o| o.price_per_stone),
            best_sell_price: best_sell.map(|o| o.price_per_stone),
            market_depth: self.get_market_depth(stone_type).await?,
        })
    }
    
    pub async fn get_market_depth(
        &self,
        stone_type: MagicStoneType,
    ) -> Result<MarketDepth> {
        // Get top 10 buy and sell orders
        let top_buys = self.db.get_top_buy_orders(stone_type, 10).await?;
        let top_sells = self.db.get_top_sell_orders(stone_type, 10).await?;
        
        Ok(MarketDepth {
            buy_orders: top_buys,
            sell_orders: top_sells,
        })
    }
}
```

## Cancelling Orders

```rust
impl MagicStoneMarket {
    pub async fn cancel_order(
        &self,
        player: &mut Player,
        order_id: OrderId,
    ) -> Result<()> {
        let mut order = self.db.get_order(order_id).await?
            .ok_or(MarketError::OrderNotFound)?;
        
        // Verify player owns the order
        if order.player_id != player.id {
            return Err(MarketError::NotYourOrder);
        }
        
        // Verify order is still active
        if !matches!(order.status, OrderStatus::Active | OrderStatus::PartiallyFilled) {
            return Err(MarketError::OrderNotCancellable);
        }
        
        // Refund remaining escrow
        match order.order_type {
            OrderType::Buy { gold_paid } => {
                let remaining_quantity = order.quantity - order.filled_quantity;
                let refund = remaining_quantity as u64 * order.price_per_stone;
                player.add_gold(refund).await?;
            }
            OrderType::Sell { stones_paid } => {
                let remaining_quantity = order.quantity - order.filled_quantity;
                player.add_magic_stone(order.stone_type, remaining_quantity).await?;
            }
        }
        
        // Update order status
        order.status = OrderStatus::Cancelled;
        self.db.update_order(order).await?;
        
        Ok(())
    }
}
```

## Database Schema

```sql
CREATE TABLE magic_stone_orders (
    id BIGSERIAL PRIMARY KEY,
    player_id BIGINT NOT NULL,
    stone_type VARCHAR(50) NOT NULL,
    order_type VARCHAR(10) NOT NULL,
    quantity INTEGER NOT NULL,
    price_per_stone BIGINT NOT NULL,
    total_value BIGINT NOT NULL,
    filled_quantity INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP,
    status VARCHAR(20) NOT NULL,
    INDEX idx_stone_type (stone_type),
    INDEX idx_order_type (order_type),
    INDEX idx_status (status)
);
```

## Maintenance Tasks

**Every Hour:**
- Process order matching for all active orders
- Check for expired orders and cancel them

## See Also

- [GOLD_BURN.md](./GOLD_BURN.md) - Gold burn system for listing fees
- [MAGIC_STONES.md](../items/MAGIC_STONES.md) - Magic Stones system documentation

