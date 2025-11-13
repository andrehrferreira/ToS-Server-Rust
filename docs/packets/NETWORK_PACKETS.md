# Network Packets Documentation

## Overview

Network packets handle connection management, authentication, server list, map loading, and character creation. These packets are critical for establishing and maintaining the client-server connection.

## Server to Client Packets

### Pong

**Type:** `ServerPacketType.Pong`

**Purpose:** Heartbeat response to client ping

**Data:**
```
[PacketType: byte]
[Timestamp: ushort] (optional)
```

**Usage:**
- Heartbeat system
- Connection health check
- Latency measurement
- Connection keep-alive

**Example:**
```typescript
packetPong.send(socket);
```

### LoginToken

**Type:** `ServerPacketType.LoginToken`

**Purpose:** Send authentication token after login

**Data:**
```
[PacketType: byte]
[Token: string]
```

**Usage:**
- Authentication success
- Session token
- Login completion
- Direct socket send

**Example:**
```typescript
packetLogin.send(socket, authToken);
```

### EnterToWorld

**Type:** `ServerPacketType.EnterToWorld`

**Purpose:** Confirm player entered world

**Data:**
```
[PacketType: byte]
[WorldData: string] (optional)
```

**Usage:**
- World entry confirmation
- Initial world state
- Spawn location
- Direct socket send

**Example:**
```typescript
packetEnterToWorld.send(socket);
```

### GetServerList

**Type:** `ServerPacketType.GetServerList`

**Purpose:** Send list of available game servers

**Data:**
```
[PacketType: byte]
[ServerList: string (JSON)]
```

**Server List JSON Structure:**
```json
[
    {
        "id": "server1",
        "name": "Server 1",
        "ip": "192.168.1.1",
        "port": 3565,
        "population": 150,
        "maxPlayers": 1000,
        "status": "online"
    }
]
```

**Usage:**
- Server selection screen
- Server list refresh
- Server status update
- Direct socket send

**Example:**
```typescript
packetGetServerList.send(socket, serverListJson);
```

### LoadMapData

**Type:** `ServerPacketType.LoadMapData`

**Purpose:** Send map data for loading

**Data:**
```
[PacketType: byte]
[MapData: string (JSON)]
```

**Map Data JSON Structure:**
```json
{
    "mapId": "map_001",
    "mapName": "Starting Village",
    "waypoints": [...],
    "spawnPoints": [...],
    "zones": [...]
}
```

**Usage:**
- Map loading
- World initialization
- Map change
- Direct socket send

**Example:**
```typescript
packetMapData.send(socket, mapDataJson);
```

### CreateCharacterError

**Type:** `ServerPacketType.CreateCharacterError`

**Purpose:** Send character creation error

**Data:**
```
[PacketType: byte]
[ErrorMessage: string]
```

**Usage:**
- Character creation failure
- Validation errors
- Name conflicts
- Direct socket send

**Example:**
```typescript
packetCreateCharacterError.send(socket, "Character name already exists");
```

### CreateCharacterFinish

**Type:** `ServerPacketType.CreateCharacterFinish`

**Purpose:** Confirm character creation success

**Data:**
```
[PacketType: byte]
[CharacterData: string (JSON)]
```

**Usage:**
- Character creation success
- Character data confirmation
- Character list update
- Direct socket send

**Example:**
```typescript
packetCreateCharacterFinish.send(socket, characterDataJson);
```

### LoadLevel

**Type:** `ServerPacketType.LoadLevel`

**Purpose:** Load new level/map

**Data:**
```
[PacketType: byte]
[LevelName: string]
[Waypoint: string]
```

**Usage:**
- Level transition
- Map change
- Teleportation
- Direct socket send

**Example:**
```typescript
packetLoadLevel.send(owner, {
    levelName: "map_002",
    waypoint: "spawn_point_1"
});
```

### GoTo

**Type:** `ServerPacketType.GoTo`

**Purpose:** Teleport player to location

**Data:**
```
[PacketType: byte]
[LevelName: string]
[PositionX: int32]
[PositionY: int32]
[PositionZ: int32]
```

**Usage:**
- Teleportation
- Unstuck command
- Admin teleport
- Direct socket send

**Example:**
```typescript
packetGoTo.send(owner, {
    levelName: "map_001",
    x: 1000,
    y: 2000,
    z: 100
});
```

## Client to Server Handlers

### Ping

**Type:** `ClientPacketType.Ping`

**Purpose:** Heartbeat request to server

**Data:**
```
[Timestamp: ushort] (optional)
```

**Handler Logic:**
- Records ping timestamp
- Responds with Pong packet
- Calculates latency
- Updates connection health

**Example:**
```csharp
ushort timestamp = buffer.GetUShort();
// Respond with Pong packet
```

### Login

**Type:** `ClientPacketType.Login`

**Purpose:** Authenticate player login

**Data:**
```
[Username: string]
[Password: string] (hashed)
[Token: string] (optional)
```

**Handler Logic:**
- Validates credentials
- Checks account status
- Generates session token
- Sends LoginToken packet
- Creates connection

**Example:**
```csharp
string username = buffer.GetString();
string password = buffer.GetString();
// Validate and authenticate
```

### CharacterList

**Type:** `ClientPacketType.CharacterList`

**Purpose:** Request character list

**Data:**
```
(no data)
```

**Handler Logic:**
- Gets player characters
- Serializes character list
- Sends CharacterList packet

**Example:**
```csharp
// Get characters from database
// Send CharacterList packet
```

### FullCharacter

**Type:** `ClientPacketType.FullCharacter`

**Purpose:** Request full character data

**Data:**
```
[CharacterId: string]
```

**Handler Logic:**
- Gets character data
- Loads character state
- Serializes character
- Sends FullCharacter packet

**Example:**
```csharp
string characterId = buffer.GetString();
// Load character and send data
```

### EnterToWorld

**Type:** `ClientPacketType.EnterToWorld`

**Purpose:** Request to enter game world

**Data:**
```
[CharacterId: string]
```

**Handler Logic:**
- Validates character
- Loads character state
- Spawns character in world
- Sends EnterToWorld packet
- Sends initial world state

**Example:**
```csharp
string characterId = buffer.GetString();
// Load and spawn character
```

### CreateCharacter

**Type:** `ClientPacketType.CreateCharacter`

**Purpose:** Create new character

**Data:**
```
[CharacterName: string]
[Visual: string]
[Race: byte] (optional)
[Class: byte] (optional)
```

**Handler Logic:**
- Validates character name
- Checks name availability
- Validates visual/race/class
- Creates character
- Sends CreateCharacterFinish or CreateCharacterError

**Example:**
```csharp
string characterName = buffer.GetString();
string visual = buffer.GetString();
// Validate and create character
```

### LoginSteam

**Type:** `ClientPacketType.LoginSteam`

**Purpose:** Authenticate via Steam

**Data:**
```
[SteamId: string]
[SteamToken: string]
```

**Handler Logic:**
- Validates Steam token
- Checks Steam account
- Links to game account
- Authenticates player
- Sends LoginToken packet

**Example:**
```csharp
string steamId = buffer.GetString();
string steamToken = buffer.GetString();
// Validate Steam authentication
```

### Waipoint

**Type:** `ClientPacketType.Waipoint`

**Purpose:** Request waypoint teleportation

**Data:**
```
[WaypointId: string]
```

**Handler Logic:**
- Validates waypoint exists
- Checks waypoint access
- Validates player can use waypoint
- Teleports player
- Sends LoadLevel packet

**Example:**
```csharp
string waypointId = buffer.GetString();
// Teleport to waypoint
```

## Connection Flow

### Initial Connection Flow

1. Client: Connects to server
2. Server: Sends Cookie packet (connection validation)
3. Client: Sends Connect packet with cookie
4. Server: Validates cookie
5. Server: Sends ConnectionAccepted packet
6. Client: Sends Login packet
7. Server: Validates credentials
8. Server: Sends LoginToken packet
9. Client: Stores token for session

### Character Selection Flow

1. Client: Sends CharacterList packet
2. Server: Gets player characters
3. Server: Sends CharacterList packet
4. Client: Displays character list
5. Client: Selects character
6. Client: Sends FullCharacter packet
7. Server: Loads character data
8. Server: Sends FullCharacter packet
9. Client: Displays character details

### Enter World Flow

1. Client: Sends EnterToWorld packet
2. Server: Validates character
3. Server: Loads character state
4. Server: Spawns character
5. Server: Sends EnterToWorld packet
6. Server: Sends LoadMapData packet
7. Server: Sends initial entity updates
8. Client: Loads map
9. Client: Renders world

### Character Creation Flow

1. Client: Sends CreateCharacter packet
2. Server: Validates character name
3. Server: Checks name availability
4. Server: Validates character data
5. Server: Creates character
6. Server: Sends CreateCharacterFinish packet
7. Client: Updates character list
8. Client: Shows character creation success

## Implementation Notes

### Authentication System

**Token Generation:**
- Secure random token
- Session-based tokens
- Token expiration
- Token validation

**Steam Integration:**
- Steam ID validation
- Steam token verification
- Account linking
- Steam achievements

### Connection Management

**Connection States:**
- Disconnected
- Connecting
- Connected
- Disconnecting

**Connection Health:**
- Ping/Pong heartbeat
- Connection timeout
- Automatic disconnection
- Reconnection handling

### Map Loading

**Map Data:**
- Map ID and name
- Waypoints
- Spawn points
- Zones and areas
- Map boundaries

**Level Transitions:**
- Smooth transitions
- State preservation
- Entity synchronization
- Map preloading

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [ARCHITECTURE.md](../server/ARCHITECTURE.md) - Server architecture
- [ACCOUNTS.md](../server/ACCOUNTS.md) - Account system
- [MAPS.md](../server/MAPS.md) - Map system

