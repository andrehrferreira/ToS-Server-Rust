# Advanced Economy Systems

## Overview

This document provides an overview of the advanced economy systems that control gold circulation, provide marketplace functionality, and create economic sinks to maintain currency stability. These systems work together to prevent inflation and create engaging economic gameplay.

## Systems

1. **[Gold Burn System](./GOLD_BURN.md)** - Tracks and burns gold to prevent inflation
2. **[Auction House](./AUCTION_HOUSE.md)** - Complete marketplace with buyout and auction mechanics
3. **[Magic Stone Market](./MAGIC_STONE_MARKET.md)** - Real-time currency exchange market (PoE2-style)
4. **[Bulk Orders](./BULK_ORDERS.md)** - Player and NPC-created crafting orders with rewards
5. **[Lottery System](./LOTTERY.md)** - Weekly lottery with gold burn integration
6. **[Economic Rebalancing](./ECONOMIC_REBALANCING.md)** - Automatic economic monitoring and adjustment

---

## System Overview

### Gold Burn System

The foundation of the economy, tracking all gold removed from circulation. Gold is burned through:
- NPC purchases and services
- Auction house fees and commissions
- Magic stone market fees
- Bulk order penalties
- Lottery ticket purchases (50%)
- Economic rebalancing operations

**See:** [GOLD_BURN.md](./GOLD_BURN.md)

### Auction House

Complete marketplace supporting:
- Buyout listings (instant purchase)
- Auction listings (bidding system)
- Search and filtering
- Anti-duplication protection
- Support for Gold and Magic Stones

**See:** [AUCTION_HOUSE.md](./AUCTION_HOUSE.md)

### Magic Stone Market

Real-time currency exchange similar to Path of Exile 2:
- Buy/sell orders for Magic Stones
- Order matching system
- Real-time price quotes
- Market depth visualization

**See:** [MAGIC_STONE_MARKET.md](./MAGIC_STONE_MARKET.md)

### Bulk Orders

Crafting order system:
- Player-created orders (5-day expiration, 30-min production deadline, penalty fees)
- NPC-created orders (1-day expiration, 1-day production deadline, no penalty)
- Escrow system for payment and rewards
- Penalty system for failed orders

**See:** [BULK_ORDERS.md](./BULK_ORDERS.md)

### Lottery System

Weekly lottery system:
- Prize pool: 1% of gold burned in previous week
- Ticket price: 100 gold (50% to prize pool, 50% burned)
- Daily prize pool updates
- Automatic winner selection

**See:** [LOTTERY.md](./LOTTERY.md)

### Economic Rebalancing

Automatic economic monitoring and adjustment:
- Daily NPC price adjustments based on inflation
- Resource abundance management (purchases excess resources)
- Gold drop rate adjustment (reduces drops during high inflation)
- Rare item scarcity control (limits ultra-rare items to 5 units)
- Market liquidity management (system posts items when market is empty)

**See:** [ECONOMIC_REBALANCING.md](./ECONOMIC_REBALANCING.md)

---

## Integration Points

### Gold Burn Integration

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

### Weekly Maintenance Tasks

**Sunday 23:59:59:**
- Draw lottery winner
- Create new lottery for next week
- Calculate lottery base prize from previous week's burns

**Monday 00:00:00:**
- Start new lottery week
- Reset lottery ticket sales

**Daily 00:00:00:**
- Calculate economic metrics
- Adjust NPC vendor prices
- Check rare item limits and adjust drop rates
- Check boss card limits and adjust drop rates
- Burn system trade proceeds

**Daily 03:00:00:**
- Update lottery prize pool with latest gold burn data
- Cleanup old gold burn logs (keep only current week)

**Every Hour:**
- Check for expired bulk orders
- Check for expired auction listings
- Process order matching (magic stone market)
- Check resource abundance and purchase excess
- Adjust gold drop rates if needed
- Check market liquidity and add items if needed

---

## See Also

- [GOLD.md](../core/GOLD.md) - Basic gold economy system
- [MAGIC_STONES.md](../items/MAGIC_STONES.md) - Magic Stones system
- [CRAFTING.md](../items/CRAFTING.md) - Crafting system
- [TRADE.md](../player/TRADE.md) - Player-to-player trade
