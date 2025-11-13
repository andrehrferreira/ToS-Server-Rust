# Real-Time Updates System

## Overview

The real-time updates system uses Server-Sent Events (SSE) to stream events from game servers to the admin dashboard. All game servers connect to the central API and send events as they occur, which are then broadcast to connected admin clients.

## Architecture

**Event Flow:**
```
Game Server → POST /api/v1/events/receive → Central API
                                              ↓
                                    Event Broadcast Channel
                                              ↓
                                    Admin Clients (SSE)
```

**Flow:**
1. Game server generates event (e.g., player joins, chat message, telemetry)
2. Game server sends event via POST to central API
3. Central API receives event and validates server token
4. Central API broadcasts event to all connected admin clients via SSE
5. Admin dashboard receives event and updates UI in real-time

## SSE Endpoints

**Server Event Stream:**
- `GET /events/servers` - Server status updates
- `GET /events/telemetry/{server_id}` - Server telemetry updates
- `GET /events/map/{map_id}` - Map entity updates
- `GET /events/chat` - Chat message updates
- `GET /events/tickets` - Support ticket updates
- `GET /events/players` - Player activity updates
- `GET /events/security` - Security incident updates

## Server Event Types

**Server Status Events:**
- Server started
- Server stopped
- Server restarting
- Player count changes
- Server health status

**Telemetry Events:**
- CPU usage updates
- Memory usage updates
- Network traffic updates
- Database metrics updates

**Map Events:**
- Player position updates
- Creature spawn/despawn
- NPC position updates
- Event start/end
- Waypoint updates

**Chat Events:**
- New chat message
- Whisper sent/received
- Channel join/leave

**Ticket Events:**
- New ticket created
- Ticket status changed
- Ticket response added
- Ticket assigned

**Security Events:**
- Security violation detected
- Player banned
- Bot detected
- Hack detected

## Implementation

### Server-Side Event Sending

**Game Server Implementation:**
```rust
pub struct GameServerEventSender {
    api_url: String,
    server_id: ServerId,
    event_queue: Arc<RwLock<VecDeque<ServerEvent>>>,
    sender_thread: Option<JoinHandle<()>>,
}

impl GameServerEventSender {
    pub fn new(api_url: String, server_id: ServerId) -> Self {
        Self {
            api_url,
            server_id,
            event_queue: Arc::new(RwLock::new(VecDeque::new())),
            sender_thread: None,
        }
    }
    
    pub fn start(&mut self) {
        let api_url = self.api_url.clone();
        let server_id = self.server_id.clone();
        let queue = Arc::clone(&self.event_queue);
        
        self.sender_thread = Some(tokio::spawn(async move {
            loop {
                // Process event queue
                while let Some(event) = queue.write().await.pop_front() {
                    if let Err(e) = Self::send_event_to_api(&api_url, &server_id, event).await {
                        error!("Failed to send event to API: {:?}", e);
                        // Retry logic could be added here
                    }
                }
                
                tokio::time::sleep(Duration::from_millis(100)).await;
            }
        }));
    }
    
    pub async fn send_event(&self, event: ServerEvent) {
        self.event_queue.write().await.push_back(event);
    }
    
    async fn send_event_to_api(
        api_url: &str,
        server_id: &ServerId,
        event: ServerEvent,
    ) -> Result<(), ApiError> {
        let client = reqwest::Client::new();
        
        let response = client
            .post(&format!("{}/api/v1/events/receive", api_url))
            .header("X-Server-ID", server_id.to_string())
            .header("X-Server-Token", get_server_token().await?)
            .json(&event)
            .send()
            .await?;
        
        if !response.status().is_success() {
            return Err(ApiError::HttpError(response.status()));
        }
        
        Ok(())
    }
}
```

### API-Side Event Broadcasting

**Implementation:**
```rust
use axum::response::sse::{Event, Sse};
use futures::stream::{self, Stream};

pub struct EventStream {
    event_sender: broadcast::Sender<ServerEvent>,
}

#[derive(Debug, Clone, Serialize)]
pub enum ServerEvent {
    ServerStatusUpdate {
        server_id: ServerId,
        status: ServerStatus,
        player_count: usize,
    },
    TelemetryUpdate {
        server_id: ServerId,
        telemetry: ServerTelemetry,
    },
    MapEntityUpdate {
        map_id: MapId,
        entity_id: EntityId,
        entity_type: EntityType,
        position: Position,
    },
    ChatMessage {
        server_id: ServerId,
        channel: ChatChannel,
        sender_id: CharacterId,
        sender_name: String,
        message: String,
    },
    TicketUpdate {
        ticket_id: TicketId,
        status: TicketStatus,
    },
    PlayerActivity {
        server_id: ServerId,
        character_id: CharacterId,
        activity_type: ActivityType,
    },
    SecurityIncident {
        server_id: ServerId,
        incident_type: IncidentType,
        player_id: CharacterId,
    },
}

// API-side: Receive events from game servers
pub async fn receive_server_event(
    event: ServerEvent,
    event_broadcast: broadcast::Sender<ServerEvent>,
) -> Result<(), ApiError> {
    // Broadcast event to all connected admin clients
    event_broadcast.send(event)?;
    Ok(())
}

// API-side: Stream events to admin clients
pub async fn stream_events(
    event_receiver: broadcast::Receiver<ServerEvent>,
) -> Sse<impl Stream<Item = Result<Event, axum::Error>>> {
    let stream = stream::unfold(event_receiver, |mut receiver| async move {
        match receiver.recv().await {
            Ok(event) => {
                let event = Event::default()
                    .json_data(event)
                    .map_err(|e| axum::Error::from(e))?;
                Some((Ok(event), receiver))
            }
            Err(_) => None,
        }
    });
    
    Sse::new(stream)
}
```

### Admin Client Connection

**Dashboard Implementation:**
```typescript
// Admin dashboard connects to SSE endpoint
class EventStream {
    private eventSource: EventSource;
    
    constructor(endpoint: string, adminToken: string) {
        this.eventSource = new EventSource(
            `${endpoint}?token=${adminToken}`
        );
        
        this.setupEventHandlers();
    }
    
    private setupEventHandlers() {
        // Listen for server events
        this.eventSource.addEventListener('server-status', (e) => {
            const event = JSON.parse(e.data);
            this.handleServerStatusUpdate(event);
        });
        
        this.eventSource.addEventListener('telemetry', (e) => {
            const event = JSON.parse(e.data);
            this.handleTelemetryUpdate(event);
        });
        
        this.eventSource.addEventListener('map-entity', (e) => {
            const event = JSON.parse(e.data);
            this.handleMapEntityUpdate(event);
        });
        
        this.eventSource.addEventListener('chat', (e) => {
            const event = JSON.parse(e.data);
            this.handleChatMessage(event);
        });
        
        this.eventSource.addEventListener('ticket', (e) => {
            const event = JSON.parse(e.data);
            this.handleTicketUpdate(event);
        });
        
        this.eventSource.addEventListener('security', (e) => {
            const event = JSON.parse(e.data);
            this.handleSecurityIncident(event);
        });
        
        this.eventSource.onerror = (error) => {
            console.error('SSE connection error:', error);
            // Implement reconnection logic
        };
    }
    
    private handleServerStatusUpdate(event: ServerStatusUpdate) {
        // Update server status in dashboard
        updateServerStatus(event.server_id, event.status);
    }
    
    private handleTelemetryUpdate(event: TelemetryUpdate) {
        // Update telemetry charts
        updateTelemetryChart(event.server_id, event.telemetry);
    }
    
    private handleMapEntityUpdate(event: MapEntityUpdate) {
        // Update map with entity position
        updateMapEntity(event.map_id, event.entity_id, event.position);
    }
    
    private handleChatMessage(event: ChatMessage) {
        // Add message to chat monitor
        addChatMessage(event);
    }
    
    private handleTicketUpdate(event: TicketUpdate) {
        // Update ticket status
        updateTicket(event.ticket_id, event.status);
    }
    
    private handleSecurityIncident(event: SecurityIncident) {
        // Show security alert
        showSecurityAlert(event);
    }
    
    public close() {
        this.eventSource.close();
    }
}
```

## Connection Management

### Server Connection

- Game servers maintain persistent connection to API
- Automatic reconnection on disconnect
- Heartbeat to keep connection alive
- Token validation on each event

### Client Connection

- Admin clients connect via SSE
- Automatic reconnection on disconnect
- Token-based authentication
- Filtered events based on permissions

## Usage Examples

### Game Server Sending Events

```rust
// Usage in game server
impl GameServer {
    pub async fn on_player_join(&self, player: CharacterId) {
        // Send event to API
        self.event_sender.send_event(ServerEvent::PlayerActivity {
            server_id: self.server_id.clone(),
            character_id: player,
            activity_type: ActivityType::PlayerJoined,
        }).await;
    }
    
    pub async fn on_chat_message(
        &self,
        channel: ChatChannel,
        sender: CharacterId,
        message: String,
    ) {
        // Send event to API
        self.event_sender.send_event(ServerEvent::ChatMessage {
            server_id: self.server_id.clone(),
            channel,
            sender_id: sender,
            sender_name: get_character_name(sender).await,
            message,
        }).await;
    }
    
    pub async fn on_telemetry_update(&self, telemetry: ServerTelemetry) {
        // Send telemetry update every 5 seconds
        self.event_sender.send_event(ServerEvent::TelemetryUpdate {
            server_id: self.server_id.clone(),
            telemetry,
        }).await;
    }
}
```

## Summary

The real-time updates system provides:

- **Server-Sent Events**: Efficient one-way event streaming
- **Event Broadcasting**: Central API broadcasts to all clients
- **Real-Time Updates**: Instant updates in admin dashboard
- **Multiple Event Types**: Servers, telemetry, maps, chat, tickets, security
- **Automatic Reconnection**: Resilient connections
- **Token-Based Security**: Secure event streaming

This system ensures administrators have real-time visibility into all game server activities without polling or manual refresh.

