# Player Management

## Overview

Player management provides complete control over player accounts and characters, including account management, character management, inventory management, and chat monitoring.

## Account Management

**Features:**
- View account details
- Ban/unban accounts
- View account characters
- View account history
- Modify account settings

**API Endpoints:**
- `GET /players/accounts/{account_id}` - Get account details
- `POST /players/accounts/{account_id}/ban` - Ban account
- `POST /players/accounts/{account_id}/unban` - Unban account
- `GET /players/accounts/{account_id}/characters` - Get account characters
- `GET /players/accounts/{account_id}/history` - Get account history

## Character Management

**Features:**
- View character details
- View character inventory
- View character bank
- Remove items
- Transfer items
- Modify character stats
- Teleport character

**API Endpoints:**
- `GET /players/characters/{character_id}` - Get character details
- `GET /players/characters/{character_id}/inventory` - Get inventory
- `GET /players/characters/{character_id}/bank` - Get bank contents
- `DELETE /players/characters/{character_id}/items/{item_id}` - Remove item
- `POST /players/characters/{character_id}/items/transfer` - Transfer item
- `PUT /players/characters/{character_id}/stats` - Modify stats
- `POST /players/characters/{character_id}/teleport` - Teleport character

## Chat Monitoring

**Features:**
- View all chat channels
- View whispers (private messages)
- Search chat history
- Filter by player, channel, time

**API Endpoints:**
- `GET /chat/channels` - Get all chat channels
- `GET /chat/whispers` - Get all whispers
- `GET /chat/history` - Get chat history
- `GET /chat/search` - Search chat messages

**Implementation:**
```rust
pub struct ChatMonitoring {
    chat_logs: Arc<RwLock<Vec<ChatMessage>>>,
}

pub struct ChatMessage {
    pub message_id: MessageId,
    pub channel: ChatChannel,
    pub sender_id: CharacterId,
    pub sender_name: String,
    pub receiver_id: Option<CharacterId>, // For whispers
    pub message: String,
    pub timestamp: DateTime<Utc>,
    pub server_id: ServerId,
}
```

## Summary

Player management provides comprehensive tools for managing player accounts and characters, ensuring administrators have complete visibility and control over player data and activities.

