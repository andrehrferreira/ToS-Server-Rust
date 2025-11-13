# Lottery System

## Overview

The lottery system runs weekly with a prize pool funded by 1% of all gold burned during the week. Players can purchase tickets for 100 gold each, with 50% going to the prize pool and 50% being burned.

## Lottery Mechanics

**Weekly Cycle:**
- Lottery starts every Monday at 00:00:00
- Prize pool: 1% of total gold burned in previous week
- Prize pool updates daily at 03:00:00
- Lottery ends Sunday at 23:59:59
- Winner drawn automatically at end of week

**Ticket System:**
- Ticket price: 100 gold
- 50 gold → Prize pool
- 50 gold → Burned (logged in gold burn system)

## Lottery Structure

```rust
pub struct Lottery {
    pub week_id: u64,
    pub week_start: DateTime<Utc>,
    pub week_end: DateTime<Utc>,
    pub prize_pool: u64,
    pub base_prize: u64,  // 1% of burned gold from previous week
    pub ticket_contributions: u64,  // 50% of ticket sales
    pub tickets_sold: u32,
    pub winner_id: Option<PlayerId>,
    pub status: LotteryStatus,
    pub last_update: DateTime<Utc>,  // Last daily update
}

pub enum LotteryStatus {
    Active,
    Drawing,
    Completed,
}

pub struct LotteryTicket {
    pub id: TicketId,
    pub player_id: PlayerId,
    pub week_id: u64,
    pub purchased_at: DateTime<Utc>,
}
```

## Purchasing Tickets

```rust
impl LotterySystem {
    pub async fn purchase_ticket(
        &self,
        player: &mut Player,
        quantity: u32,
    ) -> Result<Vec<TicketId>> {
        // 1. Calculate cost
        let cost_per_ticket = 100;
        let total_cost = cost_per_ticket * quantity as u64;
        
        // 2. Verify player has enough gold
        if player.get_gold() < total_cost {
            return Err(LotteryError::InsufficientGold);
        }
        
        // 3. Remove gold
        player.remove_gold(total_cost).await?;
        
        // 4. Calculate prize pool contribution (50%)
        let prize_contribution = total_cost / 2;
        let burn_amount = total_cost / 2;
        
        // 5. Get current lottery
        let mut lottery = self.get_current_lottery().await?;
        
        // 6. Create tickets
        let mut ticket_ids = Vec::new();
        for _ in 0..quantity {
            let ticket = LotteryTicket {
                id: self.generate_ticket_id(),
                player_id: player.id,
                week_id: lottery.week_id,
                purchased_at: Utc::now(),
            };
            
            self.db.insert_ticket(ticket.clone()).await?;
            ticket_ids.push(ticket.id);
        }
        
        // 7. Update lottery
        lottery.prize_pool += prize_contribution;
        lottery.ticket_contributions += prize_contribution;
        lottery.tickets_sold += quantity;
        self.db.update_lottery(lottery).await?;
        
        // 8. Log gold burn (50% of ticket price)
        self.gold_burn_system
            .burn_gold(
                burn_amount,
                GoldBurnSource::LotteryTicket,
                Some(player.id),
            )
            .await?;
        
        Ok(ticket_ids)
    }
}
```

## Daily Prize Pool Update

```rust
impl LotterySystem {
    pub async fn daily_update(&self) -> Result<()> {
        // Runs at 03:00:00 daily
        
        // 1. Get current lottery
        let mut lottery = self.get_current_lottery().await?;
        
        // 2. Calculate base prize from previous week's gold burn
        let previous_week_start = lottery.week_start - Duration::days(7);
        let total_burned = self.gold_burn_system
            .get_weekly_total_for_date(previous_week_start)
            .await?;
        
        let base_prize = (total_burned as f64 * 0.01) as u64; // 1% of burned
        
        // 3. Update lottery
        lottery.base_prize = base_prize;
        lottery.prize_pool = base_prize + lottery.ticket_contributions;
        lottery.last_update = Utc::now();
        
        self.db.update_lottery(lottery).await?;
        
        Ok(())
    }
}
```

## Drawing Winner

```rust
impl LotterySystem {
    pub async fn draw_winner(&self) -> Result<()> {
        // Runs at end of week (Sunday 23:59:59)
        
        // 1. Get current lottery
        let mut lottery = self.get_current_lottery().await?;
        
        // 2. Get all tickets for this week
        let tickets = self.db.get_tickets_for_week(lottery.week_id).await?;
        
        if tickets.is_empty() {
            // No tickets sold, lottery cancelled
            lottery.status = LotteryStatus::Completed;
            self.db.update_lottery(lottery).await?;
            return Ok(());
        }
        
        // 3. Random selection
        let winner_ticket = self.random_select_ticket(&tickets)?;
        let winner_id = winner_ticket.player_id;
        
        // 4. Award prize
        let winner = self.get_player_mut(winner_id).await?;
        winner.add_gold(lottery.prize_pool).await?;
        
        // 5. Update lottery
        lottery.winner_id = Some(winner_id);
        lottery.status = LotteryStatus::Completed;
        self.db.update_lottery(lottery).await?;
        
        // 6. Create new lottery for next week
        self.create_new_lottery().await?;
        
        Ok(())
    }
    
    fn random_select_ticket(&self, tickets: &[LotteryTicket]) -> Result<&LotteryTicket> {
        use rand::Rng;
        let mut rng = rand::thread_rng();
        let index = rng.gen_range(0..tickets.len());
        Ok(&tickets[index])
    }
    
    async fn create_new_lottery(&self) -> Result<()> {
        let now = Utc::now();
        let week_start = self.get_week_start(now);
        let week_end = week_start + Duration::days(7) - Duration::seconds(1);
        
        let lottery = Lottery {
            week_id: self.generate_week_id(),
            week_start,
            week_end,
            prize_pool: 0,
            base_prize: 0,
            ticket_contributions: 0,
            tickets_sold: 0,
            winner_id: None,
            status: LotteryStatus::Active,
            last_update: now,
        };
        
        self.db.insert_lottery(lottery).await?;
        
        Ok(())
    }
}
```

## Database Schema

```sql
CREATE TABLE lottery (
    week_id BIGSERIAL PRIMARY KEY,
    week_start TIMESTAMP NOT NULL,
    week_end TIMESTAMP NOT NULL,
    prize_pool BIGINT NOT NULL,
    base_prize BIGINT NOT NULL,
    ticket_contributions BIGINT NOT NULL DEFAULT 0,
    tickets_sold INTEGER NOT NULL DEFAULT 0,
    winner_id BIGINT,
    status VARCHAR(20) NOT NULL,
    last_update TIMESTAMP NOT NULL
);

CREATE TABLE lottery_tickets (
    id BIGSERIAL PRIMARY KEY,
    player_id BIGINT NOT NULL,
    week_id BIGINT NOT NULL,
    purchased_at TIMESTAMP NOT NULL,
    INDEX idx_week_id (week_id),
    INDEX idx_player_id (player_id)
);
```

## Scheduled Tasks

**Sunday 23:59:59:**
- Draw lottery winner
- Create new lottery for next week
- Calculate lottery base prize from previous week's burns

**Monday 00:00:00:**
- Start new lottery week
- Reset lottery ticket sales

**Daily 03:00:00:**
- Update lottery prize pool with latest gold burn data

## See Also

- [GOLD_BURN.md](./GOLD_BURN.md) - Gold burn system that funds the lottery
- [ECONOMIC_REBALANCING.md](./ECONOMIC_REBALANCING.md) - Economic systems

