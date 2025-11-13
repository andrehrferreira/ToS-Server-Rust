# Bulk Orders System

## Overview

Bulk Orders allow players and NPCs to create crafting orders that other players can fulfill. The creator specifies materials, rewards, and payment. Producers can accept orders and receive payment upon completion.

## Player-Created Bulk Orders

### Order Creation

**Process:**
1. Player creates order with:
   - Item to be crafted
   - Required materials
   - Payment amount (Gold)
   - Reward items (optional)
   - Quantity needed

2. Payment and rewards are locked in escrow
3. Order is visible to all players
4. If no one accepts within 5 days, everything is refunded

### Order Structure

```rust
pub struct BulkOrder {
    pub id: OrderId,
    pub creator_id: PlayerId,
    pub producer_id: Option<PlayerId>,  // Set when accepted
    pub item_to_craft: ItemRef,
    pub required_materials: Vec<MaterialRequirement>,
    pub payment_gold: u64,
    pub reward_items: Vec<Item>,
    pub quantity: u32,
    pub status: BulkOrderStatus,
    pub created_at: DateTime<Utc>,
    pub accepted_at: Option<DateTime<Utc>>,
    pub expires_at: DateTime<Utc>,  // 5 days from creation
    pub production_deadline: Option<DateTime<Utc>>,  // 30 min from acceptance
    pub penalty_pool: u64,  // Accumulated penalties from failed attempts
}

pub struct MaterialRequirement {
    pub item_ref: ItemRef,
    pub quantity: u32,
}

pub enum BulkOrderStatus {
    Open,  // Waiting for producer
    InProduction { producer_id: PlayerId, deadline: DateTime<Utc> },
    Completed,
    Expired,  // 5 days passed, no one accepted
    Failed,  // Producer failed to deliver on time
}
```

### Creating a Bulk Order

```rust
impl BulkOrderSystem {
    pub async fn create_order(
        &self,
        creator: &mut Player,
        item_ref: ItemRef,
        materials: Vec<MaterialRequirement>,
        payment_gold: u64,
        reward_items: Vec<Item>,
        quantity: u32,
    ) -> Result<OrderId> {
        // 1. Verify creator has payment and rewards
        if creator.get_gold() < payment_gold {
            return Err(BulkOrderError::InsufficientGold);
        }
        
        // Verify reward items are in inventory
        for reward_item in &reward_items {
            if !creator.inventory.has_item(reward_item.ref()) {
                return Err(BulkOrderError::RewardItemNotFound);
            }
        }
        
        // 2. Lock payment and rewards in escrow
        creator.remove_gold(payment_gold).await?;
        for reward_item in &reward_items {
            creator.inventory.remove_item(reward_item.ref()).await?;
        }
        
        // 3. Create order
        let order = BulkOrder {
            id: self.generate_order_id(),
            creator_id: creator.id,
            producer_id: None,
            item_to_craft: item_ref,
            required_materials: materials,
            payment_gold,
            reward_items,
            quantity,
            status: BulkOrderStatus::Open,
            created_at: Utc::now(),
            accepted_at: None,
            expires_at: Utc::now() + Duration::days(5),
            production_deadline: None,
            penalty_pool: 0,
        };
        
        // 4. Store order
        self.db.insert_bulk_order(order.clone()).await?;
        
        Ok(order.id)
    }
}
```

### Accepting a Bulk Order

```rust
impl BulkOrderSystem {
    pub async fn accept_order(
        &self,
        producer: &mut Player,
        order_id: OrderId,
    ) -> Result<()> {
        // 1. Get order
        let mut order = self.db.get_bulk_order(order_id).await?
            .ok_or(BulkOrderError::OrderNotFound)?;
        
        // 2. Verify order is open
        if !matches!(order.status, BulkOrderStatus::Open) {
            return Err(BulkOrderError::OrderNotAvailable);
        }
        
        // 3. Verify order hasn't expired
        if Utc::now() > order.expires_at {
            order.status = BulkOrderStatus::Expired;
            self.db.update_bulk_order(order).await?;
            return Err(BulkOrderError::OrderExpired);
        }
        
        // 4. Calculate penalty fee (20% of total order value)
        let total_value = order.payment_gold + self.calculate_rewards_value(&order.reward_items);
        let penalty_fee = (total_value as f64 * 0.20) as u64;
        
        // 5. Verify producer has enough gold for penalty
        if producer.get_gold() < penalty_fee {
            return Err(BulkOrderError::InsufficientGoldForPenalty);
        }
        
        // 6. Lock penalty fee
        producer.remove_gold(penalty_fee).await?;
        
        // 7. Update order status
        order.producer_id = Some(producer.id);
        order.status = BulkOrderStatus::InProduction {
            producer_id: producer.id,
            deadline: Utc::now() + Duration::minutes(30),
        };
        order.accepted_at = Some(Utc::now());
        order.production_deadline = Some(Utc::now() + Duration::minutes(30));
        order.penalty_pool += penalty_fee;
        
        self.db.update_bulk_order(order).await?;
        
        Ok(())
    }
}
```

### Completing a Bulk Order

```rust
impl BulkOrderSystem {
    pub async fn complete_order(
        &self,
        producer: &mut Player,
        order_id: OrderId,
        crafted_items: Vec<Item>,
    ) -> Result<()> {
        // 1. Get order
        let mut order = self.db.get_bulk_order(order_id).await?
            .ok_or(BulkOrderError::OrderNotFound)?;
        
        // 2. Verify producer is the one who accepted
        if order.producer_id != Some(producer.id) {
            return Err(BulkOrderError::NotYourOrder);
        }
        
        // 3. Verify order is in production
        if !matches!(order.status, BulkOrderStatus::InProduction { .. }) {
            return Err(BulkOrderError::OrderNotInProduction);
        }
        
        // 4. Verify deadline hasn't passed
        if let Some(deadline) = order.production_deadline {
            if Utc::now() > deadline {
                // Order failed - penalty is burned, order goes back to open
                return self.handle_failed_order(order_id).await;
            }
        }
        
        // 5. Verify crafted items match requirements
        if !self.verify_crafted_items(&crafted_items, &order) {
            return Err(BulkOrderError::ItemsDoNotMatch);
        }
        
        // 6. Remove crafted items from producer
        for item in &crafted_items {
            producer.inventory.remove_item(item.ref()).await?;
        }
        
        // 7. Give crafted items to creator
        let creator = self.get_player_mut(order.creator_id).await?;
        for item in crafted_items {
            creator.inventory.add_item(item).await?;
        }
        
        // 8. Give payment and rewards to producer
        producer.add_gold(order.payment_gold).await?;
        for reward_item in &order.reward_items {
            producer.inventory.add_item(reward_item.clone()).await?;
        }
        
        // 9. Refund penalty fee to producer (successful completion)
        // Note: Penalty pool stays for future failed attempts
        
        // 10. Update order status
        order.status = BulkOrderStatus::Completed;
        self.db.update_bulk_order(order).await?;
        
        Ok(())
    }
    
    async fn handle_failed_order(&self, order_id: OrderId) -> Result<()> {
        let mut order = self.db.get_bulk_order(order_id).await?
            .ok_or(BulkOrderError::OrderNotFound)?;
        
        // Penalty is added to penalty pool (burned)
        // Order goes back to open status
        order.producer_id = None;
        order.status = BulkOrderStatus::Open;
        order.accepted_at = None;
        order.production_deadline = None;
        // penalty_pool stays increased
        
        // Log gold burn (penalty)
        self.gold_burn_system
            .burn_gold(
                order.penalty_pool,  // All accumulated penalties
                GoldBurnSource::BulkOrderPenalty,
                order.producer_id,
            )
            .await?;
        
        self.db.update_bulk_order(order).await?;
        
        Ok(())
    }
}
```

### Order Expiration (5 Days)

```rust
impl BulkOrderSystem {
    pub async fn check_expired_orders(&self) -> Result<()> {
        let expired_orders = self.db.get_expired_orders().await?;
        
        for mut order in expired_orders {
            // Refund everything to creator
            let creator = self.get_player_mut(order.creator_id).await?;
            
            creator.add_gold(order.payment_gold).await?;
            for reward_item in &order.reward_items {
                creator.inventory.add_item(reward_item.clone()).await?;
            }
            
            order.status = BulkOrderStatus::Expired;
            self.db.update_bulk_order(order).await?;
        }
        
        Ok(())
    }
}
```

---

## NPC Bulk Orders

### Overview

NPCs with crafting stations can create bulk orders that players can accept. These orders have a 1-day deadline and no penalty fee, making them more accessible.

### NPC Order Structure

```rust
pub struct NpcBulkOrder {
    pub id: OrderId,
    pub npc_id: NpcId,
    pub item_to_craft: ItemRef,
    pub required_materials: Vec<MaterialRequirement>,
    pub payment_gold: u64,
    pub reward_items: Vec<Item>,
    pub quantity: u32,
    pub status: NpcBulkOrderStatus,
    pub created_at: DateTime<Utc>,
    pub accepted_at: Option<DateTime<Utc>>,
    pub expires_at: DateTime<Utc>,  // 1 day from creation
    pub production_deadline: Option<DateTime<Utc>>,  // 1 day from acceptance
}

pub enum NpcBulkOrderStatus {
    Open,
    InProduction { producer_id: PlayerId, deadline: DateTime<Utc> },
    Completed,
    Expired,
    Failed,
}
```

### Accepting NPC Orders

```rust
impl NpcBulkOrderSystem {
    pub async fn accept_npc_order(
        &self,
        producer: &mut Player,
        order_id: OrderId,
    ) -> Result<()> {
        // Similar to player bulk orders but:
        // - No penalty fee required
        // - 1 day deadline instead of 30 minutes
        // - Payment comes from NPC (system)
        
        let mut order = self.db.get_npc_bulk_order(order_id).await?
            .ok_or(BulkOrderError::OrderNotFound)?;
        
        if !matches!(order.status, NpcBulkOrderStatus::Open) {
            return Err(BulkOrderError::OrderNotAvailable);
        }
        
        if Utc::now() > order.expires_at {
            order.status = NpcBulkOrderStatus::Expired;
            self.db.update_npc_bulk_order(order).await?;
            return Err(BulkOrderError::OrderExpired);
        }
        
        order.producer_id = Some(producer.id);
        order.status = NpcBulkOrderStatus::InProduction {
            producer_id: producer.id,
            deadline: Utc::now() + Duration::days(1),
        };
        order.accepted_at = Some(Utc::now());
        order.production_deadline = Some(Utc::now() + Duration::days(1));
        
        self.db.update_npc_bulk_order(order).await?;
        
        Ok(())
    }
}
```

## Database Schema

```sql
CREATE TABLE bulk_orders (
    id BIGSERIAL PRIMARY KEY,
    creator_id BIGINT NOT NULL,
    producer_id BIGINT,
    item_to_craft VARCHAR(100) NOT NULL,
    required_materials JSONB NOT NULL,
    payment_gold BIGINT NOT NULL,
    reward_items JSONB NOT NULL,
    quantity INTEGER NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    accepted_at TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    production_deadline TIMESTAMP,
    penalty_pool BIGINT NOT NULL DEFAULT 0,
    INDEX idx_status (status),
    INDEX idx_expires_at (expires_at)
);

CREATE TABLE npc_bulk_orders (
    id BIGSERIAL PRIMARY KEY,
    npc_id BIGINT NOT NULL,
    producer_id BIGINT,
    item_to_craft VARCHAR(100) NOT NULL,
    required_materials JSONB NOT NULL,
    payment_gold BIGINT NOT NULL,
    reward_items JSONB NOT NULL,
    quantity INTEGER NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    accepted_at TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    production_deadline TIMESTAMP,
    INDEX idx_status (status),
    INDEX idx_expires_at (expires_at)
);
```

## Maintenance Tasks

**Every Hour:**
- Check for expired bulk orders (both player and NPC)
- Process failed orders (deadline passed)

## See Also

- [GOLD_BURN.md](./GOLD_BURN.md) - Gold burn system for penalties
- [CRAFTING.md](../items/CRAFTING.md) - Crafting system documentation

