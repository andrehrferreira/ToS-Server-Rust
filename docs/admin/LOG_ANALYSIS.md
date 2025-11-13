# Log Analysis System

## Overview

The log analysis system provides comprehensive log viewing and analysis capabilities for administrators to monitor game activity, security incidents, and system events.

## Log Viewing

**Features:**
- View security logs
- View activity logs
- View error logs
- Search logs
- Filter by date, player, event type
- Export logs

**API Endpoints:**
- `GET /logs/security` - Get security logs
- `GET /logs/activity` - Get activity logs
- `GET /logs/errors` - Get error logs
- `GET /logs/search` - Search logs
- `POST /logs/export` - Export logs

**Implementation:**
```rust
pub struct LogViewer {
    log_storage: LogStorage,
}

pub struct LogQuery {
    pub log_type: LogType,
    pub start_date: Option<DateTime<Utc>>,
    pub end_date: Option<DateTime<Utc>>,
    pub player_id: Option<CharacterId>,
    pub event_type: Option<EventType>,
    pub server_id: Option<ServerId>,
    pub limit: Option<usize>,
}

#[derive(Debug, Clone)]
pub enum LogType {
    Security,
    Activity,
    Error,
    Chat,
    Transaction,
}
```

## Log Types

### Security Logs

- Security violations
- Bot detection
- Hack detection
- Ban/unban actions
- Admin actions

### Activity Logs

- Player kills
- Item drops/loots
- Level ups
- Gold transactions
- Guild activities

### Error Logs

- Server errors
- Database errors
- Network errors
- Application errors

## Log Search

**Search Capabilities:**
- Search by player ID
- Search by event type
- Search by date range
- Search by server
- Full-text search

## Log Export

**Export Formats:**
- JSON
- CSV
- Plain text
- Filtered exports

## Summary

The log analysis system provides comprehensive tools for viewing, searching, and analyzing game logs, ensuring administrators can monitor all game activity and troubleshoot issues effectively.

