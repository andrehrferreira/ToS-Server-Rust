# Support Ticket System

## Overview

The support ticket system allows players to submit support requests and administrators to respond, assign, and manage tickets through the admin dashboard.

## Ticket Management

**Features:**
- View all tickets
- Respond to tickets
- Assign tickets to admins
- Close tickets
- View ticket history

**API Endpoints:**
- `GET /support/tickets` - List all tickets
- `GET /support/tickets/{ticket_id}` - Get ticket details
- `POST /support/tickets/{ticket_id}/respond` - Respond to ticket
- `PUT /support/tickets/{ticket_id}/assign` - Assign ticket
- `POST /support/tickets/{ticket_id}/close` - Close ticket

**Implementation:**
```rust
pub struct SupportTicket {
    pub ticket_id: TicketId,
    pub player_id: CharacterId,
    pub player_name: String,
    pub subject: String,
    pub message: String,
    pub status: TicketStatus,
    pub assigned_to: Option<AdminId>,
    pub responses: Vec<TicketResponse>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

pub struct TicketResponse {
    pub response_id: ResponseId,
    pub responder_id: AdminId,
    pub responder_name: String,
    pub message: String,
    pub timestamp: DateTime<Utc>,
}
```

## Ticket Status

**Status Types:**
- **Open**: New ticket, awaiting response
- **Assigned**: Ticket assigned to admin
- **In Progress**: Admin working on ticket
- **Resolved**: Issue resolved, awaiting player confirmation
- **Closed**: Ticket closed

## Real-Time Updates

Ticket status changes are broadcast via SSE. See [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) for details.

## Summary

The support ticket system provides a comprehensive platform for players to request help and administrators to respond efficiently, ensuring good player support.

