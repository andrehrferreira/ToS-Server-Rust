# Reward System

## Overview

The reward system allows administrators to create reward campaigns and automatically distribute rewards to players based on various criteria.

## Automatic Rewards

**Features:**
- Create reward campaigns
- Schedule automatic rewards
- Target specific players or groups
- Set reward conditions
- Track reward distribution

**API Endpoints:**
- `POST /rewards/create` - Create reward campaign
- `GET /rewards/campaigns` - List all campaigns
- `POST /rewards/distribute` - Distribute rewards
- `GET /rewards/history` - Get reward history

**Implementation:**
```rust
pub struct RewardCampaign {
    pub campaign_id: CampaignId,
    pub name: String,
    pub description: String,
    pub target_players: RewardTarget,
    pub rewards: Vec<Reward>,
    pub conditions: RewardConditions,
    pub schedule: RewardSchedule,
    pub status: CampaignStatus,
    pub created_by: AdminId,
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Clone)]
pub enum RewardTarget {
    AllPlayers,
    SpecificPlayers(Vec<CharacterId>),
    LevelRange { min: u8, max: u8 },
    Server(ServerId),
    Guild(GuildId),
}

pub struct Reward {
    pub item_id: Option<ItemId>,
    pub item_type: String,
    pub quantity: u32,
    pub gold: Option<i64>,
    pub experience: Option<i64>,
}
```

## Reward Distribution

**Distribution Methods:**
- Immediate distribution
- Scheduled distribution
- Conditional distribution (based on player actions)
- Event-based distribution

## Summary

The reward system provides flexible tools for creating and distributing rewards to players, supporting various campaigns and promotional activities.

