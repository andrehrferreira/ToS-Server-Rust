# Auction House System

## Overview

The auction house provides a complete marketplace where players can buy and sell items. It supports both instant buyout and auction-style bidding. The system includes comprehensive search, filtering, and anti-duplication protection.

## Features

**Core Features:**
- List items for sale (buyout price)
- List items for auction (bidding)
- Cancel listings (with fees)
- Buy items instantly (buyout)
- Place bids on auctions
- Receive gold/currency when item sells
- Search by item name
- Filter by category
- Filter by price range
- Filter by rarity
- Filter by tier
- Filter by seller
- Sort by price, time, rarity, etc.

**Anti-Duplication Protection:**
- Item verification on listing
- Item locked in escrow during listing
- Transaction validation on purchase
- Atomic operations for buy/sell
- Duplicate detection algorithms

## Listing Types

**1. Buyout Listing:**
- Fixed price sale
- Instant purchase available
- Listing fee: 1% of listing price (minimum 10 gold)
- Commission: 5% of sale price (paid by seller)

**2. Auction Listing:**
- Starting bid price
- Optional buyout price
- Auction duration: 24 hours, 48 hours, or 7 days
- Minimum bid increment: 5% of current bid
- Listing fee: 1% of starting bid (minimum 10 gold)
- Commission: 5% of final bid (paid by seller)

## Listing Process

```rust
pub struct AuctionHouseListing {
    pub id: ListingId,
    pub seller_id: PlayerId,
    pub item: Item,  // Item in escrow
    pub listing_type: ListingType,
    pub currency: Currency,  // Gold or Magic Stone type
    pub starting_bid: Option<u64>,  // For auctions
    pub buyout_price: Option<u64>,  // For buyout or auction buyout
    pub current_bid: Option<u64>,  // For auctions
    pub current_bidder: Option<PlayerId>,  // For auctions
    pub created_at: DateTime<Utc>,
    pub expires_at: DateTime<Utc>,
    pub status: ListingStatus,
    pub category: ItemCategory,
    pub search_tags: Vec<String>,
}

pub enum ListingType {
    Buyout { price: u64 },
    Auction { 
        starting_bid: u64,
        buyout_price: Option<u64>,
        duration_hours: u32,
    },
}

pub enum Currency {
    Gold,
    MagicStone(MagicStoneType),
}

pub enum ListingStatus {
    Active,
    Sold,
    Cancelled,
    Expired,
}
```

## Listing an Item

```rust
impl AuctionHouse {
    pub async fn list_item(
        &self,
        seller: &mut Player,
        item_ref: ItemRef,
        listing_type: ListingType,
        currency: Currency,
    ) -> Result<ListingId> {
        // 1. Verify item exists and is in seller's inventory
        let item = seller.inventory.get_item(item_ref)?;
        
        // 2. Calculate listing fee
        let listing_fee = self.calculate_listing_fee(&listing_type)?;
        
        // 3. Verify seller has enough currency for fee
        match currency {
            Currency::Gold => {
                if seller.get_gold() < listing_fee {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                seller.remove_gold(listing_fee).await?;
            }
            Currency::MagicStone(stone_type) => {
                // Check magic stone balance
                if !seller.has_magic_stone(stone_type, listing_fee) {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                seller.remove_magic_stone(stone_type, listing_fee).await?;
            }
        }
        
        // 4. Remove item from inventory (escrow)
        seller.inventory.remove_item(item_ref).await?;
        
        // 5. Create listing
        let listing = AuctionHouseListing {
            id: self.generate_listing_id(),
            seller_id: seller.id,
            item: item.clone(),
            listing_type,
            currency,
            starting_bid: None,
            buyout_price: None,
            current_bid: None,
            current_bidder: None,
            created_at: Utc::now(),
            expires_at: self.calculate_expiry(&listing_type),
            status: ListingStatus::Active,
            category: item.category(),
            search_tags: item.generate_search_tags(),
        };
        
        // 6. Store listing
        self.db.insert_listing(listing.clone()).await?;
        
        // 7. Log gold burn (listing fee)
        self.gold_burn_system
            .burn_gold(
                listing_fee,
                GoldBurnSource::AuctionHouseFee,
                Some(seller.id),
            )
            .await?;
        
        Ok(listing.id)
    }
    
    fn calculate_listing_fee(&self, listing_type: &ListingType) -> Result<u64> {
        let base_price = match listing_type {
            ListingType::Buyout { price } => *price,
            ListingType::Auction { starting_bid, .. } => *starting_bid,
        };
        
        let fee = (base_price as f64 * 0.01) as u64; // 1% fee
        Ok(fee.max(10)) // Minimum 10 gold
    }
}
```

## Buying an Item (Buyout)

```rust
impl AuctionHouse {
    pub async fn buy_item(
        &self,
        buyer: &mut Player,
        listing_id: ListingId,
    ) -> Result<()> {
        // 1. Get listing
        let mut listing = self.db.get_listing(listing_id).await?
            .ok_or(AuctionHouseError::ListingNotFound)?;
        
        // 2. Verify listing is active and buyout available
        if listing.status != ListingStatus::Active {
            return Err(AuctionHouseError::ListingNotActive);
        }
        
        let buyout_price = listing.buyout_price
            .ok_or(AuctionHouseError::NoBuyoutAvailable)?;
        
        // 3. Verify buyer has enough currency
        match listing.currency {
            Currency::Gold => {
                if buyer.get_gold() < buyout_price {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                buyer.remove_gold(buyout_price).await?;
            }
            Currency::MagicStone(stone_type) => {
                if !buyer.has_magic_stone(stone_type, buyout_price) {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                buyer.remove_magic_stone(stone_type, buyout_price).await?;
            }
        }
        
        // 4. Calculate commission (5% of sale)
        let commission = (buyout_price as f64 * 0.05) as u64;
        let seller_receives = buyout_price - commission;
        
        // 5. Give item to buyer
        buyer.inventory.add_item(listing.item.clone()).await?;
        
        // 6. Give currency to seller
        let seller = self.get_player_mut(listing.seller_id).await?;
        match listing.currency {
            Currency::Gold => {
                seller.add_gold(seller_receives).await?;
            }
            Currency::MagicStone(stone_type) => {
                seller.add_magic_stone(stone_type, seller_receives).await?;
            }
        }
        
        // 7. Update listing status
        listing.status = ListingStatus::Sold;
        self.db.update_listing(listing.clone()).await?;
        
        // 8. Log gold burn (commission)
        self.gold_burn_system
            .burn_gold(
                commission,
                GoldBurnSource::AuctionHouseCommission,
                Some(listing.seller_id),
            )
            .await?;
        
        Ok(())
    }
}
```

## Auction Bidding

```rust
impl AuctionHouse {
    pub async fn place_bid(
        &self,
        bidder: &mut Player,
        listing_id: ListingId,
        bid_amount: u64,
    ) -> Result<()> {
        // 1. Get listing
        let mut listing = self.db.get_listing(listing_id).await?
            .ok_or(AuctionHouseError::ListingNotFound)?;
        
        // 2. Verify listing is auction type
        let auction_info = match &listing.listing_type {
            ListingType::Auction { .. } => {},
            ListingType::Buyout { .. } => {
                return Err(AuctionHouseError::NotAnAuction);
            }
        };
        
        // 3. Verify auction hasn't expired
        if Utc::now() > listing.expires_at {
            listing.status = ListingStatus::Expired;
            self.db.update_listing(listing).await?;
            return Err(AuctionHouseError::AuctionExpired);
        }
        
        // 4. Calculate minimum bid
        let min_bid = listing.current_bid
            .map(|bid| (bid as f64 * 1.05) as u64) // 5% increment
            .unwrap_or(listing.starting_bid.unwrap_or(0));
        
        if bid_amount < min_bid {
            return Err(AuctionHouseError::BidTooLow);
        }
        
        // 5. Refund previous bidder if exists
        if let Some(previous_bidder_id) = listing.current_bidder {
            let previous_bidder = self.get_player_mut(previous_bidder_id).await?;
            let refund_amount = listing.current_bid.unwrap();
            
            match listing.currency {
                Currency::Gold => {
                    previous_bidder.add_gold(refund_amount).await?;
                }
                Currency::MagicStone(stone_type) => {
                    previous_bidder.add_magic_stone(stone_type, refund_amount).await?;
                }
            }
        }
        
        // 6. Deduct bid from bidder
        match listing.currency {
            Currency::Gold => {
                if bidder.get_gold() < bid_amount {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                bidder.remove_gold(bid_amount).await?;
            }
            Currency::MagicStone(stone_type) => {
                if !bidder.has_magic_stone(stone_type, bid_amount) {
                    return Err(AuctionHouseError::InsufficientFunds);
                }
                bidder.remove_magic_stone(stone_type, bid_amount).await?;
            }
        }
        
        // 7. Update listing
        listing.current_bid = Some(bid_amount);
        listing.current_bidder = Some(bidder.id);
        self.db.update_listing(listing.clone()).await?;
        
        // 8. Check if buyout was reached
        if let Some(buyout_price) = listing.buyout_price {
            if bid_amount >= buyout_price {
                // Automatically complete sale
                return self.complete_auction(listing_id).await;
            }
        }
        
        // 9. Broadcast bid update to interested players
        self.broadcast_bid_update(listing_id, bid_amount, bidder.id).await?;
        
        Ok(())
    }
    
    pub async fn complete_auction(&self, listing_id: ListingId) -> Result<()> {
        // Similar to buy_item but uses current_bid instead of buyout_price
        // Refund logic for previous bidders already handled
        // ...
    }
}
```

## Search and Filtering

```rust
pub struct AuctionHouseSearch {
    pub query: Option<String>,  // Item name search
    pub category: Option<ItemCategory>,
    pub min_price: Option<u64>,
    pub max_price: Option<u64>,
    pub rarity: Option<ItemRarity>,
    pub tier: Option<ItemTier>,
    pub seller_id: Option<PlayerId>,
    pub currency: Option<Currency>,
    pub listing_type: Option<ListingTypeFilter>,
    pub sort_by: SortOption,
    pub sort_order: SortOrder,
    pub page: u32,
    pub page_size: u32,
}

pub enum ListingTypeFilter {
    BuyoutOnly,
    AuctionOnly,
    Both,
}

pub enum SortOption {
    Price,
    Time,
    Rarity,
    Tier,
    Name,
}

pub enum SortOrder {
    Ascending,
    Descending,
}

impl AuctionHouse {
    pub async fn search(&self, search: AuctionHouseSearch) -> Result<Vec<AuctionHouseListing>> {
        self.db.search_listings(search).await
    }
}
```

## Cancelling a Listing

```rust
impl AuctionHouse {
    pub async fn cancel_listing(
        &self,
        player: &mut Player,
        listing_id: ListingId,
    ) -> Result<()> {
        // 1. Get listing
        let mut listing = self.db.get_listing(listing_id).await?
            .ok_or(AuctionHouseError::ListingNotFound)?;
        
        // 2. Verify player is the seller
        if listing.seller_id != player.id {
            return Err(AuctionHouseError::NotYourListing);
        }
        
        // 3. Verify listing is still active
        if listing.status != ListingStatus::Active {
            return Err(AuctionHouseError::ListingNotActive);
        }
        
        // 4. Refund any bids (for auctions)
        if let Some(bidder_id) = listing.current_bidder {
            let bidder = self.get_player_mut(bidder_id).await?;
            let refund_amount = listing.current_bid.unwrap();
            
            match listing.currency {
                Currency::Gold => {
                    bidder.add_gold(refund_amount).await?;
                }
                Currency::MagicStone(stone_type) => {
                    bidder.add_magic_stone(stone_type, refund_amount).await?;
                }
            }
        }
        
        // 5. Return item to seller
        player.inventory.add_item(listing.item.clone()).await?;
        
        // 6. Update listing status
        listing.status = ListingStatus::Cancelled;
        self.db.update_listing(listing).await?;
        
        // Note: Listing fee is not refunded (already burned)
        
        Ok(())
    }
}
```

## System Operations

The auction house also supports system operations for economic rebalancing:

```rust
impl AuctionHouse {
    // System can purchase items (for resource rebalancing)
    pub async fn buy_item_system(&self, listing_id: ListingId) -> Result<()> {
        // Similar to buy_item but gold goes to system account
        // System account gold is burned periodically
    }
    
    // System can list items (for market liquidity)
    pub async fn list_item_system(&self, item: Item, price: u64) -> Result<ListingId> {
        // System lists item, proceeds go to system account
    }
    
    pub async fn get_market_depth(&self) -> Result<MarketDepth> {
        // Get market statistics for liquidity calculation
    }
    
    pub async fn count_listings_by_category(&self, category: ItemCategory) -> Result<usize> {
        // Count listings in a category
    }
    
    pub async fn count_listings_by_item(&self, item_ref: ItemRef) -> Result<u32> {
        // Count listings of a specific item
    }
}
```

## Database Schema

```sql
CREATE TABLE auction_house_listings (
    id BIGSERIAL PRIMARY KEY,
    seller_id BIGINT NOT NULL,
    item_data JSONB NOT NULL,
    listing_type VARCHAR(20) NOT NULL,
    currency VARCHAR(20) NOT NULL,
    starting_bid BIGINT,
    buyout_price BIGINT,
    current_bid BIGINT,
    current_bidder_id BIGINT,
    created_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    status VARCHAR(20) NOT NULL,
    category VARCHAR(50),
    search_tags TEXT[],
    INDEX idx_status (status),
    INDEX idx_expires_at (expires_at),
    INDEX idx_category (category)
);
```

## Maintenance Tasks

**Every Hour:**
- Check for expired auction listings
- Auto-complete expired auctions (award to highest bidder)

## See Also

- [GOLD_BURN.md](./GOLD_BURN.md) - Gold burn system for fees and commissions
- [ECONOMIC_REBALANCING.md](./ECONOMIC_REBALANCING.md) - Economic rebalancing integration
- [MAGIC_STONE_MARKET.md](./MAGIC_STONE_MARKET.md) - Magic stone currency support

