# Network Protocol Overview

## Overview

The game uses a custom binary UDP protocol for client-server communication. The protocol is designed for high-performance gameplay, supporting packet queuing, batching, reliable/unreliable delivery modes, fragmentation, and encryption. All packets use a binary format with a packet header (14 bytes) followed by serialized data using ByteBuffer.

## Key Features

- **Binary UDP Protocol**: Efficient binary serialization using ByteBuffer over UDP
- **Packet Header**: 14-byte header with connection ID, channel, flags, and sequence
- **Packet Queuing**: Automatic packet batching and queuing system
- **Duplicate Detection**: Prevents duplicate packet transmission
- **Reliable/Unreliable Channels**: Support for reliable ordered, reliable unordered, and unreliable delivery
- **Packet Batching**: Multiple packets combined into single transmission
- **Fragmentation**: Automatic fragmentation for packets > 1200 bytes (MTU)
- **Encryption**: ChaCha20-Poly1305 AEAD encryption for all payloads
- **Type Safety**: Enum-based packet type system
- **UDP Transport**: UDP-based communication with reliability layer

## Protocol Structure

### Packet Header

**Messenger Packet Structure:**
```
[Flags: 1 byte][Type: 1 byte][Sequence: 4 bytes if Reliable][Payload: variable][CRC32: 4 bytes]
```

**Total Header Size:**
- Minimum: 6 bytes (Flags + Type + CRC32)
- With Sequence: 10 bytes (Flags + Type + Sequence + CRC32)
- Payload: Variable length (ByteBuffer)

**Packet Flags (PacketFlags enum):**
- `None` (0x00): No special flags
- `Encrypted` (0x01): Packet is encrypted with ChaCha20-Poly1305
- `Rekey` (0x04): Request key exchange
- `Fragment` (0x08): Packet is a fragment
- `Compressed` (0x10): Packet payload is compressed (LZ4)

**Packet Types (PacketType enum):**
- `Connect`: Connection request
- `Ping`: Heartbeat request (server → client)
- `Pong`: Heartbeat response (client → server)
- `Reliable`: Reliable packet with sequence number (guaranteed delivery, ordered)
- `Unreliable`: Unreliable packet (best-effort delivery, no guarantees)
- `Ack`: Acknowledgment for reliable packets
- `Disconnect`: Disconnection request
- `ConnectionAccepted`: Connection accepted response
- `ConnectionDenied`: Connection denied response
- `Cookie`: Cookie for connection validation
- `RetryToken`: Retry token for connection
- `Handshake`: Handshake packet
- `HandshakeAck`: Handshake acknowledgment
- `ReliableHandshake`: Reliable channel handshake
- `Heartbeat`: Heartbeat packet
- `Error`: Error packet

### Packet Format

**Basic Packet Structure (Messenger):**
```
[Flags: 1 byte][Type: 1 byte][Sequence: 4 bytes if Reliable][Payload: variable][CRC32: 4 bytes]
```

**Packet Header Breakdown:**
- `Flags` (1 byte): PacketFlags enum (Encrypted, Rekey, Fragment, Compressed)
- `Type` (1 byte): PacketType enum (Connect, Ping, Reliable, Unreliable, etc.)
- `Sequence` (4 bytes, optional): Sequence number for reliable packets
- `Payload` (variable): Serialized packet data (ByteBuffer)
- `CRC32` (4 bytes): CRC32C checksum for integrity validation

**Queued Packet Structure (TypeScript version):**
```
[QueueType: 1 byte][Packet1][0xFE 0xFE 0xFE 0xFE][Packet2][0xFE 0xFE 0xFE 0xFE]...
```

**Note:** The TypeScript version uses a simpler queuing system with packet separators. The Rust/C# version uses the full UDP protocol with headers, channels, and fragmentation.

### UDP Protocol Details

**UDP Transport:**
- Connectionless protocol (no persistent connection)
- Best-effort delivery (packets may be lost)
- Low latency (no handshake overhead)
- High throughput (minimal overhead)

**Reliability Layer:**
- Reliable channels provide guaranteed delivery
- Acknowledgment system for reliable packets
- Retransmission for lost packets
- Sequence numbers for ordering

**Connection Management:**
- `Connection` class manages client connection state
- `ConnectionId` (uint) identifies client connection
- `ConnectionState` enum: Disconnected, Connecting, Connected, Disconnecting
- Heartbeat system (Ping/Pong) for connection health
- Automatic disconnection on timeout (30 seconds default)
- State recovery on reconnection
- Sequence numbers for reliable packet ordering
- Ping/RTT calculation for latency monitoring

**MTU and Fragmentation:**
- Maximum Transmission Unit: 1200 bytes
- Packets > 1200 bytes automatically fragmented
- Fragments reassembled on receiver
- Fragment timeout: 10 seconds

**Performance Targets:**
- 60 FPS server tick rate
- 600,000 UDP packets per second
- 10,000 concurrent players
- < 50ms latency (p99)

### Packet Types

**Packet Type System:**
- Packets are identified by `PacketType` enum
- Each packet type has a corresponding handler
- Handlers process packets based on type
- Server and client packets use same type system

**Server Packets (IPacketServer):**
- Sent from server to client
- Implement `IPacketServer` interface
- Must implement `Serialize(ref ByteBuffer buffer)` and `ToBuffer()`
- Examples: ConnectionAccepted, Ping, Entity updates, Player actions, UI updates

**Client Packets (IPacketClient):**
- Sent from client to server
- Implement `IPacketClient` interface
- Must implement `Deserialize(ref ByteBuffer buffer)`
- Examples: Pong, Movement, Actions, Interactions, Trade, Crafting

**Control Packets:**
- `Connect`: Connection request
- `Ping`: Heartbeat request
- `Pong`: Heartbeat response
- `Reliable`: Reliable packet with sequence number
- `Unreliable`: Unreliable packet (best-effort)
- `Ack`: Acknowledgment for reliable packets
- `Disconnect`: Disconnection request
- `ConnectionAccepted`: Connection accepted response
- `ConnectionDenied`: Connection denied response
- `Cookie`: Cookie for connection validation
- `RetryToken`: Retry token for connection

### ByteBuffer Serialization

**Supported Types:**
- `byte` (1 byte): Unsigned 8-bit integer
- `bool` (1 byte): Boolean (0 or 1)
- `int32` (4 bytes): Signed 32-bit integer (little-endian)
- `uint32` (4 bytes): Unsigned 32-bit integer (little-endian)
- `float` (4 bytes): 32-bit floating point (little-endian)
- `string` (4 bytes length + UTF-8 data): Variable-length UTF-8 string
- `id` (4 bytes): Entity ID (GUID converted to int)
- `vector` (12 bytes): 3 floats (x, y, z)
- `rotator` (12 bytes): 3 floats (pitch, yaw, roll)

**String Format:**
- Length prefix: 4-byte signed integer (UTF-8 byte count)
- Data: UTF-8 encoded string bytes

**ID Format:**
- GUID converted to 32-bit integer
- Uses GUID mapping system
- Supports entity identification

## Packet Queuing System

### QueueBuffer

**Purpose:**
- Batches multiple packets into single transmission
- Reduces network overhead
- Prevents duplicate packets
- Optimizes bandwidth usage

**Queue Management:**
- Per-socket queue storage
- Maximum buffer size: 512 KB
- Automatic flush on size limit
- Periodic flush via tick system

**Packet Separation:**
- Packets separated by `0xFE 0xFE 0xFE 0xFE` (4 bytes)
- Queue packet type: `ServerPacketType.Queue`
- Client splits combined packets using separator

**Duplicate Detection:**
- Hex-based duplicate checking
- Prevents same packet sent multiple times
- Compares packet hex representation

**Queue Operations:**
- `addBuffer(socketId, buffer)`: Add packet to queue
- `checkAndSend(socketId)`: Check size and send if needed
- `sendBuffers(socketId)`: Flush and send queued packets
- `tick()`: Periodic flush of all queues

### Queue Behavior

**Automatic Flush:**
- When total size >= 512 KB
- On periodic tick (every deltaTime seconds)
- Manual flush via `sendBuffers()`

**Single Packet:**
- If queue has 1 packet, send directly
- No separator needed
- Immediate transmission

**Multiple Packets:**
- Combined with separators
- Queue type byte prepended
- All packets sent in single transmission

## Packet Delivery Modes

### Direct Socket Send

**Usage:**
- Immediate delivery required
- Critical packets (login, errors)
- Single packet transmission
- No queuing

**Example:**
```typescript
// UDP socket send
udpSocket.send(buffer.getBuffer(), clientAddress, clientPort);
```

### Queued Send

**Usage:**
- Non-critical updates
- Batchable packets
- Entity updates
- UI updates

**Example:**
```typescript
QueueBuffer.addBuffer(entity.mapIndex, buffer);
```

## Packet Categories

### Entity Packets
- Entity creation, updates, removal
- Entity synchronization
- Entity events (death, revive, animations)
- See [ENTITY_PACKETS.md](./ENTITY_PACKETS.md)

### Container Packets
- Container open/close
- Item add/remove/update
- Inventory management
- See [CONTAINER_PACKETS.md](./CONTAINER_PACKETS.md)

### Player Packets
- Player actions
- Character creation
- Stats updates
- Equipment changes
- See [PLAYER_PACKETS.md](./PLAYER_PACKETS.md)

### UI Packets
- Window management
- Crafting interface
- Vendor interface
- Trade interface
- Tooltips
- See [UI_PACKETS.md](./UI_PACKETS.md)

### Network Packets
- Connection management
- Authentication
- Server list
- Map loading
- See [NETWORK_PACKETS.md](./NETWORK_PACKETS.md)

### Client Packets (Handlers)
- Movement and rotation
- Actions and casting
- Interactions
- Trade operations
- Crafting operations
- See [CLIENT_PACKETS.md](./CLIENT_PACKETS.md)

## Packet Flow

### Server to Client

1. **Packet Creation**: Server creates packet instance
2. **Data Serialization**: Data serialized to ByteBuffer
3. **Header Addition**: Packet header added (ConnectionId, Channel, Flags, Sequence)
4. **Encryption**: Payload encrypted with ChaCha20-Poly1305 (if enabled)
5. **Fragmentation**: Packet fragmented if > 1200 bytes (MTU)
6. **Queue Decision**: Direct send or queue (for batching)
7. **Transmission**: Sent via UDP socket
8. **Client Reception**: Client receives UDP datagram
9. **Decryption**: Payload decrypted (if encrypted)
10. **Reassembly**: Fragments reassembled (if fragmented)
11. **Deserialization**: Data deserialized from ByteBuffer
12. **Processing**: Client processes packet data

### Client to Server

1. **User Action**: Player performs action
2. **Packet Creation**: Client creates packet
3. **Data Serialization**: Data serialized to ByteBuffer
4. **Header Addition**: Packet header added
5. **Encryption**: Payload encrypted (if enabled)
6. **Fragmentation**: Packet fragmented if > 1200 bytes
7. **Transmission**: Sent via UDP socket
8. **Server Reception**: Server receives UDP datagram
9. **Decryption**: Payload decrypted
10. **Reassembly**: Fragments reassembled
11. **Handler Processing**: Packet handler processes data
12. **Response**: Server may send response packets

## Packet Security

### Validation

**Server-Side Validation:**
- All client packets validated
- Range checks on values
- Type validation
- Entity existence checks

**Client-Side Validation:**
- Packet structure validation
- Type checking
- Range validation
- Error handling

### Rate Limiting

**Packet Throttling:**
- Rate limits on high-frequency packets
- Prevents spam/exploits
- Per-packet-type limits
- Throttler system (commented in code, to be implemented)

## Error Handling

### Packet Errors

**Buffer Underflow:**
- Insufficient data in buffer
- Throws error on read
- Packet discarded

**Invalid Data:**
- Type mismatches
- Range violations
- Validation failures
- Error logged, packet ignored

**Network Errors (UDP-Specific):**
- Packet loss (handled by reliable channel retransmission)
- Out-of-order delivery (handled by sequence numbers)
- Duplicate packets (handled by sequence window)
- Connection loss (detected via heartbeat timeout)
- Timeout handling (reliable channel retransmission)
- Reconnection logic (new connection ID, state recovery)
- State recovery (client resync on reconnection)

## Performance Considerations

### Optimization Strategies

**Packet Batching:**
- Multiple packets combined
- Reduces network overhead
- Lower latency for updates

**Duplicate Prevention:**
- Hex-based duplicate detection
- Prevents redundant transmission
- Saves bandwidth

**Selective Updates:**
- Only send changed data
- Entity visibility checks
- Distance-based culling

**Queue Management:**
- Size-based flushing
- Time-based flushing
- Prevents buffer overflow

## Implementation Notes

### Packet Architecture

The protocol uses a handler-based architecture where packets are either **Server** (sent from server) or **Client** (sent from client), and each packet type has a corresponding handler.

### C# Implementation (Generic Server)

**Packet Interfaces:**
```csharp
// Server packets (sent from server to client)
public interface IPacketServer {
    void Serialize(ref ByteBuffer buffer);
    ByteBuffer ToBuffer();
}

// Client packets (sent from client to server)
public interface IPacketClient {
    void Deserialize(ref ByteBuffer buffer);
}
```

**Packet Handler Base:**
```csharp
public abstract class PacketHandler {
    public abstract PacketType Type { get; }
    public abstract void Consume(Messenger msg, Connection conn);
    
    // Static handler registry
    static PacketHandler[] Handlers = new PacketHandler[1024];
    
    // Handle packet routing
    public static void HandlePacket(Messenger msg, Connection conn) {
        var handler = Handlers[(int)msg.Type];
        if (handler != null) {
            msg.UnpackSecurityCompress(conn);
            handler.Consume(msg, conn);
        }
    }
}
```

**Server Packet Example:**
```csharp
public struct ConnectionAcceptedPacket : IPacketServer {
    public uint Id;
    public byte[] ServerPublicKey;
    public byte[] Salt;
    
    public void Serialize(ref ByteBuffer buffer) {
        buffer.Write(Id);
        buffer.WriteBytes(ServerPublicKey);
        buffer.WriteBytes(Salt);
    }
    
    public ByteBuffer ToBuffer() {
        var buffer = new ByteBuffer();
        Serialize(ref buffer);
        return buffer;
    }
}
```

**Client Packet Example:**
```csharp
public struct PongPacket : IPacketClient {
    public ushort SentTimestamp;
    
    public void Deserialize(ref ByteBuffer buffer) {
        SentTimestamp = buffer.Read<ushort>();
    }
    
    public static PongPacket FromBuffer(ref ByteBuffer buffer) {
        var packet = new PongPacket();
        packet.Deserialize(ref buffer);
        return packet;
    }
}
```

**Packet Handler Example:**
```csharp
public class PongHandler : PacketHandler {
    public override PacketType Type => PacketType.Pong;
    
    public override void Consume(Messenger msg, Connection conn) {
        var pongPacket = PongPacket.FromBuffer(ref msg.Payload);
        ushort nowMs = (ushort)(Stopwatch.GetTimestamp() / (Stopwatch.Frequency / 1000) % 65536);
        ushort rttMs = (ushort)((nowMs - pongPacket.SentTimestamp) & 0xFFFF);
        
        conn.Ping = rttMs;
        conn.TimeoutLeft = 30f;
    }
}
```

**Messenger (Packet Wrapper):**
```csharp
public unsafe struct Messenger : IDisposable {
    public PacketFlags Flags;      // Encryption, compression, fragment flags
    public PacketType Type;        // Packet type (Connect, Ping, Reliable, etc.)
    public uint Sequence;          // Sequence number (for reliable packets)
    public ByteBuffer Payload;     // Packet payload data
    public Address Address;        // Client address
    
    // Pack packet with header and CRC32C
    public byte* Pack() {
        // [Flags: 1 byte][Type: 1 byte][Sequence: 4 bytes if Reliable][Payload: variable][CRC32: 4 bytes]
        // Returns pointer to packed buffer
    }
    
    // Unpack packet and validate CRC32C
    public static Messenger Unpack(byte* buffer, int length, Address address) {
        // Validates CRC32C, extracts header, returns Messenger
    }
}
```

**Sending Packets:**
```csharp
// Send server packet
public static bool Send(PacketType type, Connection conn, PacketFlags flags = PacketFlags.None) {
    return GlobalSendChannel.Writer.TryWrite(new Messenger {
        Address = conn.RemoteAddress,
        Payload = new ByteBuffer(),
        Type = type,
        Sequence = conn.NextSequence,
        Flags = flags
    });
}

// Send server packet with payload
public static bool Send(PacketType type, ref ByteBuffer buffer, Connection conn, PacketFlags flags = PacketFlags.None) {
    return GlobalSendChannel.Writer.TryWrite(new Messenger {
        Address = conn.RemoteAddress,
        Payload = buffer,
        Type = type,
        Sequence = conn.NextSequence,
        Flags = flags
    });
}

// Example: Send ConnectionAccepted packet
conn.Send(PacketType.ConnectionAccepted, (new ConnectionAcceptedPacket {
    Id = conn.Id,
    ServerPublicKey = serverPub,
    Salt = salt
}).ToBuffer());
```

**Receiving Packets:**
```csharp
// Process received packet
public static unsafe void ProcessPacket(byte* buffer, int len, Address address) {
    var msg = Messenger.Unpack(buffer, len, address);
    
    switch (msg.Type) {
        case PacketType.Connect:
            HandleConnect(new Connection { RemoteAddress = address }, msg);
            break;
        case PacketType.Unreliable:
        case PacketType.Reliable:
            if (Clients.TryGetValue(address, out Connection conn))
                Task.Run(() => PacketHandler.HandlePacket(msg, conn));
            break;
    }
    
    msg.Free();
}
```

### TypeScript Implementation

**Packet Base Class:**
```typescript
export class Packet {
    public type: ServerPacketType | ClientPacketType;
    
    public send(entity: Entity, data: any = null, extra: any = null): void {
        // Default implementation
    }
}
```

**Packet Direct Socket:**
```typescript
export class PacketDirectSocket extends Packet {
    public override send(socket: any, data: any = null): void {
        if (socket) {
            socket.send(new ByteBuffer()
                .putByte(this.type)
                .putString(data)
                .getBuffer());
        }
    }
}
```

**Packet with ByteBuffer:**
```typescript
export class PacketExample extends Packet {
    public override type = ServerPacketType.Example;
    
    public override send(owner: Entity, data: any): void {
        if (owner.udpSocket) {
            const buffer = new ByteBuffer()
                .putByte(this.type)
                .putInt32(data.value)
                .putString(data.message);
            
            // Add packet header, encrypt, fragment if needed
            const finalBuffer = preparePacket(buffer, owner.connectionId, Channel.ReliableOrdered);
            owner.udpSocket.send(finalBuffer, owner.address, owner.port);
        }
    }
}
```

## Packet Handler System

### Handler Registration

**Static Handler Registry:**
- Handlers registered automatically via reflection
- Handler array indexed by PacketType
- One handler per packet type
- Handlers discovered from namespace (e.g., "Wormhole.Handlers")

**Handler Processing Flow:**
1. Packet received via UDP
2. Messenger.Unpack() extracts packet data
3. PacketHandler.HandlePacket() routes to correct handler
4. Handler.Consume() processes packet
5. Handler performs game logic
6. Response packets sent if needed

### Handler Best Practices

**Server Packet Handlers:**
- Serialize data using `IPacketServer.Serialize()`
- Use `ToBuffer()` to create ByteBuffer
- Send via `Connection.Send()` or `Server.Send()`

**Client Packet Handlers:**
- Deserialize data using `IPacketClient.Deserialize()`
- Use `FromBuffer()` static method for creation
- Access connection via `Connection` parameter
- Update connection state as needed

**Error Handling:**
- Handlers should handle exceptions gracefully
- Invalid packets should be logged and ignored
- Connection should be disconnected on critical errors

## Related Documentation

- [ENTITY_PACKETS.md](./ENTITY_PACKETS.md) - Entity synchronization packets
- [CONTAINER_PACKETS.md](./CONTAINER_PACKETS.md) - Container management packets
- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player-specific packets
- [ACTIONBAR_PACKETS.md](./ACTIONBAR_PACKETS.md) - ActionBar packets (action call, add/remove, state sync)
- [UI_PACKETS.md](./UI_PACKETS.md) - UI and interface packets
- [NETWORK_PACKETS.md](./NETWORK_PACKETS.md) - Network and connection packets
- [CLIENT_PACKETS.md](./CLIENT_PACKETS.md) - Client-to-server packet handlers
- [BYTEBUFFER.md](../core/BYTEBUFFER.md) - ByteBuffer serialization system
- [ARCHITECTURE.md](../server/ARCHITECTURE.md) - Server architecture and UDP protocol details

