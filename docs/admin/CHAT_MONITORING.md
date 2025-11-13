# Chat Monitoring System

## Overview

The chat monitoring system allows administrators to view all chat messages including public channels and private whispers, ensuring proper moderation and security.

## Chat Monitoring Features

**Features:**
- View all chat channels
- View whispers (private messages)
- Search chat history
- Filter by player, channel, time
- Real-time chat updates

**API Endpoints:**
- `GET /chat/channels` - Get all chat channels
- `GET /chat/whispers` - Get all whispers
- `GET /chat/history` - Get chat history
- `GET /chat/search` - Search chat messages
- `GET /events/chat` - Real-time chat updates (SSE)

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

#[derive(Debug, Clone)]
pub enum ChatChannel {
    Global,
    Local,
    Guild,
    Party,
    Whisper,
    System,
}
```

## Chat Channels

**Channel Types:**
- **Global**: Server-wide chat
- **Local**: Area-based chat
- **Guild**: Guild-only chat
- **Party**: Party-only chat
- **Whisper**: Private messages
- **System**: System messages

## Real-Time Monitoring

Chat messages are streamed in real-time via SSE. See [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) for details.

## Privacy Considerations

**Access Control:**
- Only administrators with appropriate permissions can view chat
- Chat logs are stored securely
- Access is logged for audit purposes

## Summary

The chat monitoring system provides administrators with complete visibility into all chat communications, ensuring proper moderation and security while maintaining player privacy through access controls.

