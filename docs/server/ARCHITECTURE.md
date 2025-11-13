# Server Architecture Documentation

## Overview

This document describes the complete architecture of the Tales of Shadowland MMORPG server. The server is designed for high-performance gameplay, targeting support for 10,000 simultaneous players, 60 FPS server tick rate, and processing 600,000 UDP packets per second.

## Performance Requirements

### Server Targets
- **Tick Rate**: 60 FPS (16.67ms per frame)
- **Concurrent Players**: 10,000 simultaneous connections
- **Packet Throughput**: 600,000 UDP packets per second
- **Latency**: < 50ms (p99)
- **Bandwidth Optimization**: 60-75% reduction through compression and quantization

### Network Targets
- **Minimal Header**: 14 bytes per packet
- **MTU**: 1200 bytes (optimized for UDP)
- **Overhead Reduction**: Batch packet queuing to reduce protocol overhead
- **Compression**: LZ4 for packets > 512 bytes with low entropy
- **Delta Compression**: 60-75% bandwidth reduction for entity updates

## System Architecture

### 1. Network Layer

#### 1.1 UDP Protocol

The server uses a binary UDP protocol optimized for high-performance multiplayer gameplay. All communication is binary-encoded to minimize bandwidth and processing overhead.

**Key Features:**
- Binary protocol (no JSON/text encoding)
- Minimal packet header (14 bytes)
- Batch packet processing (multiple packets in parallel)
- Hardware-accelerated CRC32C for packet signatures
- Fragmentation support for packets > 1200 bytes

#### 1.2 Packet Header Structure

```
PacketHeader (14 bytes):
- ConnectionId: uint32 (4 bytes) - Little Endian
- Channel: byte (1 byte) - Unreliable, ReliableOrdered, ReliableUnordered
- Flags: byte (1 byte) - Encrypted, Compressed, Fragment, Acknowledgment, etc.
- Sequence: uint64 (8 bytes) - Little Endian
```

**PacketHeaderFlags:**
- `Encrypted` (0x01): Packet is encrypted with ChaCha20-Poly1305
- `AEAD_ChaCha20Poly1305` (0x02): Encryption method flag
- `Rekey` (0x04): Request key exchange
- `Fragment` (0x08): Packet is a fragment
- `Compressed` (0x10): Packet payload is compressed
- `Acknowledgment` (0x20): Packet is an ACK
- `ReliableHandshake` (0x40): Reliable channel handshake

**PacketChannel:**
- `Unreliable` (0): Best-effort delivery, no guarantees
- `ReliableOrdered` (1): Guaranteed delivery, ordered
- `ReliableUnordered` (2): Guaranteed delivery, unordered

#### 1.3 Batch Packet Processing

The server implements batch packet processing to maximize throughput:

- **Receive Batch**: Process multiple packets per system call
- **Send Batch**: Send multiple packets in parallel using syscalls
- **Zero-Copy**: Direct memory access using unsafe pointers
- **Lock-Free**: Thread-safe packet queues using Channels

**Implementation:**
- NanoSockets for low-level UDP operations
- Channel-based packet queues (unbounded, thread-safe)
- Multiple worker threads for packet processing
- Batch syscalls to reduce system call overhead

#### 1.4 Packet Fragmentation

Packets larger than 1200 bytes are automatically fragmented:

```
Fragment Header:
- FragmentId: uint16 (2 bytes)
- Offset: uint16 (2 bytes)
- TotalSize: uint32 (4 bytes)
- Payload: variable length
```

**Fragmentation Process:**
1. Check packet size against MTU (1200 bytes)
2. Split packet into fragments with headers
3. Send fragments with unique FragmentId
4. Reassemble fragments on receiver
5. Cleanup old fragments after timeout (10 seconds)

### 2. Serialization System

#### 2.1 ByteBuffer / FlatBuffer

The server uses a custom ByteBuffer/FlatBuffer implementation for zero-allocation binary serialization. This is the core component of the client-server communication system.

**See [BYTEBUFFER.md](./BYTEBUFFER.md) for detailed documentation.**

**Key Features:**
- Zero-allocation operations (direct memory management)
- Unsafe pointer operations for maximum performance
- Variable-length encoding (VarInt/ZigZag)
- Bit-level operations for boolean flags
- Position save/restore for complex serialization patterns
- Structured serialization with proper alignment
- Buffer pooling for zero-allocation buffer reuse
- Integrated encryption (AES-GCM) in production version
- Symbol pooling for string caching per connection
- Delta compression for position updates

**Serialization Methods:**
- `Write<T>()`: Write unmanaged types directly
- `WriteVarInt(int)`: Variable-length integer encoding (ZigZag)
- `WriteBytes(byte[])`: Write byte arrays
- `WriteFVector(FVector, float)`: Quantized vector serialization
- `WriteFRotator(FRotator, float)`: Quantized rotation serialization
- `WriteBits(bool[])`: Bit-level boolean serialization
- `WritePosition(Vector2)`: Delta-compressed position serialization
- `PutSymbol(string)`: Symbol-pooled string serialization

**Performance:**
- 75% size reduction with VarInt encoding
- 50-75% bandwidth reduction with quantization
- 60-75% bandwidth reduction with delta compression
- Zero garbage collection overhead
- Direct memory copy operations
- Buffer pooling for zero-allocation reuse

**Version Evolution:**

**Version 1: ByteBuffer.cs (Most Tested - C# Unsafe)**
- **Location**: `Backup - ToS1/Server3/GameServer/Shared/Networking/ByteBuffer.cs`
- **Performance**: ⭐⭐⭐⭐⭐ (Highest performance, unsafe pointers)
- **Stability**: ⭐⭐⭐ (Byte alignment issues, packet boundary issues)
- **Features**: Integrated encryption, symbol pooling, delta compression, buffer pooling
- **Problems**: Byte alignment issues, packet boundary issues, buffer overflow/underflow

**Version 2: FlatBuffer.cs (Latest - C# Struct)**
- **Location**: `Server-LastCSharp/Core/Network/FlatBuffer.cs`
- **Performance**: ⭐⭐⭐⭐ (High performance, safer than ByteBuffer.cs)
- **Stability**: ⭐⭐⭐⭐ (Better error handling, exception-based)
- **Features**: Generic Write/Read, bit packing, quantized vectors/rotators
- **Advantages**: Cleaner API, type safety, exception handling
- **Disadvantages**: No pooling, no encryption, no symbol pooling

**Version 3: UFlatBuffer.cpp (Unreal Engine C++)**
- **Location**: `Server-LastCSharp/Unreal/Source/ToS_Network/Private/Network/UFlatBuffer.cpp`
- **Performance**: ⭐⭐⭐⭐ (High performance, Unreal integration)
- **Stability**: ⭐⭐⭐⭐ (Unreal memory management, bounds checking)
- **Features**: Unreal types (FVector, FRotator, FString), Blueprint integration
- **Advantages**: Unreal integration, Blueprint support, Unreal logging
- **Disadvantages**: UObject overhead, Unreal-specific

**Version 4: bytebuffer.ts (TypeScript)**
- **Location**: `Server-LastTs/src/engine/core/bytebuffer.ts`
- **Performance**: ⭐⭐ (Slowest, JavaScript overhead)
- **Stability**: ⭐⭐⭐⭐⭐ (Most stable, no alignment issues)
- **Features**: Dynamic growth, packet splitting, type-safe reads/writes
- **Advantages**: Simplicity, stability, packet isolation, type safety
- **Disadvantages**: Performance, memory allocations, no zero-copy

**Critical Lessons Learned:**

**Byte Alignment Issues (Version 1 - C# Server3):**
- **Problem**: Direct pointer arithmetic without alignment checks caused frequent misalignment errors
- **Impact**: Packet loss, data corruption, strange client behaviors
- **Root Cause**: Different data types require different alignment (1, 2, 4, 8 bytes)
- **Solution**: Use structured serialization with proper alignment validation, use `std::ptr::read_unaligned` in Rust

**Packet Boundary Issues (Version 1 - C# Server3):**
- **Problem**: Without packet boundaries, errors cascaded through the buffer
- **Impact**: One misaligned read corrupted all subsequent reads
- **Root Cause**: No isolation between packets in the buffer
- **Solution**: Implement packet boundaries with length prefixes or markers (0xFE repeated 4 times)

**Error Recovery (Version 2 - TypeScript):**
- **Problem**: Single serialization error caused complete packet loss
- **Impact**: No mechanism to recover from partial packet corruption
- **Root Cause**: Lack of packet validation and error recovery
- **Solution**: Add packet validation (CRC32C checksums) and error recovery mechanisms

**Buffer Overflow/Underflow (All Versions):**
- **Problem**: No bounds checking in unsafe code
- **Impact**: Memory corruption, crashes, security vulnerabilities
- **Root Cause**: Direct pointer arithmetic without validation
- **Solution**: Always check bounds before read/write operations

**Thread Safety (Version 1 - C# Server3):**
- **Problem**: Shared state (`QuantizeOffsetX/Y`) not thread-safe
- **Impact**: Race conditions, incorrect quantization
- **Root Cause**: Shared state without synchronization
- **Solution**: Use thread-local storage or per-connection state

**Rust Implementation Considerations:**
- Use `#[repr(align(16))]` for explicit alignment control
- Use `std::ptr::read_unaligned` and `std::ptr::write_unaligned` for safe unaligned access
- Implement packet boundaries with length prefixes or markers
- Add alignment validation at compile time where possible
- Use Rust's type system to prevent alignment errors
- Implement packet validation (CRC32C checksums) for error detection
- Add error recovery mechanisms (retransmission, resync) for critical packets
- Use thread-local storage for quantization state
- Implement buffer pooling with `Arc<Mutex<VecDeque<ByteBuffer>>>`
- Use `Result<T, BufferError>` for error handling
- Use SIMD optimizations for memory copy operations

#### 2.2 Quantization

Position and rotation data are quantized to reduce bandwidth:

**Position Quantization:**
- Float (32-bit) → Int16 (16-bit) compression
- Configurable scale factors per map
- World origin rebasing for large worlds
- 60% bandwidth reduction per position update

**Rotation Quantization:**
- Optional Yaw-only rotation (saves 8 bytes)
- Float (32-bit) → Int16 (16-bit) compression
- Configurable precision per axis

**HP/Mana Quantization:**
- Int32 (32-bit) → Byte (8-bit) compression
- Range: 0-255 (suitable for percentage-based systems)
- Automatic clamping and validation

#### 2.3 World Origin Rebasing

Large worlds are divided into quadrants to enable position quantization:

**Quadrant System:**
- Configurable quadrant size per map
- Automatic quadrant calculation from world position
- Position relative to quadrant origin
- Int16 quantization within quadrant bounds

**Benefits:**
- Support for virtually unlimited world sizes
- 60% bandwidth reduction per position update
- Seamless coordinate conversion
- Map-specific configuration

#### 2.4 Compression

LZ4 compression is used for packets with low entropy:

**Compression Strategy:**
- Threshold: 512 bytes (only compress larger packets)
- Low entropy detection (repeated patterns)
- Fast compression/decompression (LZ4)
- Compression flag in packet header

**Performance:**
- 30-50% size reduction for compressible data
- Fast compression (< 1ms for typical packets)
- Hardware-accelerated when available

#### 2.5 Delta Compression

Entity updates use delta compression to reduce bandwidth:

**Delta Compression:**
- Track previous entity state
- Send only changed fields
- EntityDelta flags (Position, Rotation, AnimState, Flags, Velocity)
- Automatic delta calculation

**Benefits:**
- 60-75% bandwidth reduction for entity updates
- Reduced server CPU usage
- Improved client performance

### 3. Encryption and Security

#### 3.1 Key Exchange (X25519)

The server uses X25519 (Elliptic Curve Diffie-Hellman) for key exchange:

**Handshake Process:**
1. Client sends public key (32 bytes)
2. Server generates key pair and sends public key (32 bytes)
3. Both parties calculate shared secret
4. HKDF key derivation (RFC 5869)
5. Separate TX/RX keys for bidirectional encryption

**Security Properties:**
- Perfect forward secrecy (ephemeral keys)
- Industry-standard cryptography
- Fast key exchange (< 10ms)
- Stateless cookie validation

#### 3.2 Encryption (ChaCha20-Poly1305)

All payloads are encrypted with ChaCha20-Poly1305 AEAD:

**Encryption Process:**
1. Generate nonce from ConnectionId + Sequence (12 bytes)
2. Encrypt payload with ChaCha20 stream cipher
3. Authenticate with Poly1305 MAC (16 bytes)
4. Include AAD (Additional Authenticated Data) from packet header

**Security Properties:**
- Confidentiality: All payloads encrypted
- Authenticity: Poly1305 MAC ensures packet integrity
- Replay Protection: 64-position sliding window
- Forward Secrecy: Ephemeral X25519 keys

#### 3.3 Replay Protection

A 64-position sliding window prevents packet replay attacks:

**Implementation:**
- Track highest sequence number received
- Maintain bitset for last 64 sequences
- Reject packets outside window
- Automatic window advancement

**Performance:**
- O(1) lookup time
- Minimal memory overhead (8 bytes per connection)
- Efficient bitset operations

#### 3.4 Cookie Anti-Spoof

HMAC-SHA256 stateless cookies prevent DDoS amplification:

**Cookie Generation:**
1. Generate random nonce (8 bytes)
2. Include client address
3. Include timestamp
4. Sign with HMAC-SHA256 (32 bytes)
5. Send cookie to client (48 bytes total)

**Cookie Validation:**
1. Extract timestamp and nonce
2. Validate timestamp (10 second TTL)
3. Recompute HMAC-SHA256 signature
4. Compare with provided signature
5. Reject if invalid or expired

**Benefits:**
- Stateless (no server-side storage)
- DDoS protection (amplification attacks prevented)
- Fast validation (< 1ms)
- Automatic expiration

#### 3.5 Heartbeat System

Periodic heartbeat packets monitor connection health:

**Heartbeat Process:**
1. Server sends Ping packet (configurable interval, default 5s)
2. Client responds with Pong packet
3. Calculate RTT (Round-Trip Time)
4. Disconnect if no response (timeout, default 15s)

**Configuration:**
- Interval: 5000ms (configurable)
- Timeout: 15000ms (configurable)
- RTT tracking for latency monitoring
- Automatic disconnection on timeout

#### 3.6 Integrity System

Client integrity verification ensures official client usage:

**Integrity Process:**
1. Generate integrity key table per version
2. Server sends integrity challenge (index)
3. Client responds with integrity key
4. Validate key against table
5. Disconnect if invalid

**Implementation:**
- Version-based key tables
- Randomized challenge indices
- Periodic integrity checks (configurable)
- Automatic table generation per version

#### 3.7 Certificate Rotation (Rekey)

Periodic key rotation maintains security:

**Rekey Process:**
1. Monitor bytes transmitted (threshold: 1GB)
2. Monitor session duration (threshold: 60 minutes)
3. Request rekey from client
4. Perform new X25519 key exchange
5. Update encryption keys

**Benefits:**
- Long-term security (key rotation)
- Forward secrecy maintenance
- Automatic rotation triggers
- Seamless key updates

### 4. Area of Interest (AOI)

#### 4.1 Grid-Based AOI

The server uses a grid-based AOI system to optimize entity replication:

**Grid Structure:**
- Configurable cell size (default: 500x500x500 units)
- 3D grid (X, Y, Z axes)
- Configurable grid dimensions
- Interest radius per entity type

**AOI Process:**
1. Calculate entity grid cell from position
2. Update entity cell on position change
3. Query entities in interest radius
4. Filter entities by distance and type
5. Send updates only to interested players

**Benefits:**
- O(1) cell lookup
- Reduced bandwidth (only visible entities)
- Scalable to large player counts
- Configurable per entity type

#### 4.2 Distance-Based Filtering

Entities are filtered by distance and type:

**Distance Configuration:**
- Player: 100,000 units (1km)
- NPC: 50,000 units (500m)
- StaticBuild: 25,000 units (250m)
- Vehicle: 150,000 units (1.5km)
- Projectile: 200,000 units (2km)
- Item: 10,000 units (100m)
- Effect: 50,000 units (500m)

**Filtering Process:**
1. Calculate distance between entities
2. Check entity type distance thresholds
3. Include entity if within range
4. Exclude entity if outside range
5. Update AOI on position change

#### 4.3 Real-Time Updates

Players receive real-time updates for entities in their AOI:

**Update Types:**
- Entity creation (spawn)
- Entity update (position, rotation, state)
- Entity removal (despawn)
- Delta updates (changed fields only)

**Update Frequency:**
- Configurable tick rate (default: 60 Hz)
- Adaptive sync based on movement
- Periodic sync for stationary entities
- Immediate sync for moving entities

#### 4.4 Adaptive Sync

Synchronization frequency adapts based on entity state:

**Sync Rates:**
- Moving entities: Full tick rate (60 Hz)
- Stationary entities: Reduced rate (10% of normal, ~6 Hz)
- Distant entities: Reduced rate (30% of normal, ~18 Hz)
- Periodic sync: Every 5 seconds for stationary entities

**Benefits:**
- Reduced bandwidth for stationary entities
- Full precision for moving entities
- Configurable thresholds
- Automatic adaptation

### 5. Entity System

The entity system manages all game objects (players, monsters, NPCs, projectiles, items, etc.) using an Entity Component System (ECS) architecture. For detailed documentation on the entity system, see [ENTITIES.md](./ENTITIES.md).

#### 5.1 Entity States

Entities use bitwise flags for state representation:

**EntityState Flags:**
- `IsAlive` (0x01): Entity is alive
- `IsInCombat` (0x02): Entity is in combat
- `IsMoving` (0x04): Entity is moving
- `IsCasting` (0x08): Entity is casting
- `IsInvisible` (0x10): Entity is invisible
- `IsStunned` (0x20): Entity is stunned
- `IsFalling` (0x40): Entity is falling

**Benefits:**
- Compact representation (1 byte for 8 states)
- Fast state checks (bitwise operations)
- Efficient serialization
- Extensible (add new states easily)

#### 5.2 Entity Delta

Entity updates use delta compression:

**EntityDelta Flags:**
- `Position` (0x01): Position changed
- `Rotation` (0x02): Rotation changed
- `AnimState` (0x04): Animation state changed
- `Flags` (0x08): Entity flags changed
- `Velocity` (0x10): Velocity changed
- `All` (0x1F): All fields changed

**Delta Process:**
1. Track previous entity state
2. Compare current state with previous
3. Calculate delta flags
4. Serialize only changed fields
5. Update previous state

#### 5.3 Entity Types

Entities are classified by type:

**EntityType:**
- `Player`: Player-controlled entity
- `NPC`: Non-player character
- `Monster`: Hostile creature
- `Projectile`: Projectile entity
- `Environment`: Environmental entity

**Type-Specific Behavior:**
- Different AOI distances per type
- Different update frequencies per type
- Different serialization per type
- Different authority rules per type

#### 5.4 Entity Pooling

Entities use object pooling to avoid allocation:

**Pooling Strategy:**
- Pre-allocate entity pool (configurable size)
- Reuse entities from pool
- Return entities to pool on removal
- Zero allocation during gameplay

**Benefits:**
- Reduced garbage collection
- Improved performance
- Predictable memory usage
- Scalable to large entity counts

### 6. Reliable UDP System

#### 6.1 Reliable Messaging

The server implements reliable UDP for guaranteed delivery:

**Reliable Channel:**
- Guaranteed delivery (ACK required)
- Ordered delivery (sequence numbers)
- Automatic retransmission
- Congestion control

**Reliable Process:**
1. Assign sequence number to packet
2. Store packet in send queue
3. Send packet to client
4. Wait for ACK
5. Retransmit if timeout (250ms default)
6. Remove from queue on ACK

#### 6.2 Acknowledgment System

Clients acknowledge reliable packets:

**ACK Packet:**
- Sequence number (uint64)
- Automatic ACK generation
- Batch ACK support (multiple sequences)

**ACK Process:**
1. Client receives reliable packet
2. Extract sequence number
3. Send ACK packet to server
4. Server receives ACK
5. Remove packet from send queue
6. Update congestion metrics

#### 6.3 Retransmission

Packets are retransmitted if not acknowledged:

**Retransmission Strategy:**
- Timeout: 250ms (configurable)
- Max retries: 10 (configurable)
- Exponential backoff (optional)
- Congestion window adjustment

**Retransmission Process:**
1. Check packet timeout
2. Retransmit if timeout exceeded
3. Increment retry count
4. Disconnect if max retries exceeded
5. Update congestion metrics

#### 6.4 Reliable Handshake

Reliable channel requires handshake:

**Handshake Process:**
1. Client sends ReliableHandshake packet
2. Server responds with ReliableHandshake
3. Both parties establish reliable channel
4. Begin reliable packet transmission
5. Maintain sequence numbers

### 7. Fragment System

#### 7.1 Fragmentation

Packets larger than MTU are fragmented:

**Fragmentation Process:**
1. Check packet size against MTU (1200 bytes)
2. Calculate number of fragments
3. Generate unique FragmentId
4. Split packet into fragments
5. Send fragments with headers
6. Reassemble on receiver

#### 7.2 Reassembly

Fragments are reassembled on receiver:

**Reassembly Process:**
1. Receive fragment with FragmentId
2. Store fragment in reassembly buffer
3. Track received fragments (bitset)
4. Check if all fragments received
5. Reassemble packet from fragments
6. Process reassembled packet
7. Cleanup fragments after timeout (10s)

#### 7.3 Fragment Cleanup

Old fragments are cleaned up automatically:

**Cleanup Process:**
1. Track fragment creation time
2. Check fragment age periodically
3. Remove fragments older than timeout (10s)
4. Free fragment buffers
5. Update reassembly state

### 8. Authority System

#### 8.1 Server Authority

The server is fully authoritative for game state:

**Authority Rules:**
- Position: Server controls all entity positions
- Rotation: Server controls all entity rotations
- Actions: Server validates all actions
- Animations: Server controls animation triggers
- State: Server controls entity state changes

**Benefits:**
- Prevents client-side cheating
- Consistent game state
- Fair gameplay
- Secure game logic

#### 8.2 Movement Authority

Server controls all entity movement:

**Movement Process:**
1. Client sends movement input
2. Server validates movement
3. Server calculates new position
4. Server updates entity position
5. Server broadcasts position update
6. Client receives authoritative position

**Validation:**
- Speed limits (prevent speed hacks)
- Collision detection (prevent wall clipping)
- Terrain validation (prevent fly hacks)
- Ground collision check (prevent flying)
- Wall collision check (prevent passing through walls)
- Height validation (prevent climbing invalid surfaces)
- Anti-cheat checks (detect anomalies)

**Movement System:**
- Grid-based spatial partitioning (3D support)
- Continuous collision detection (shape casting)
- Raycasting for pathfinding
- Swimming system (water areas, state management)
- Pathfinding system (A* for AI navigation)
- Patrol route system (monster AI)

For detailed documentation on the movement and collision system, see [MOVEMENT.md](./MOVEMENT.md).

#### 8.3 Action Authority

Server validates all actions:

**Action Process:**
1. Client sends action request
2. Server validates action
3. Server executes action
4. Server broadcasts action result
5. Client receives authoritative result

**Validation:**
- Cooldown checks
- Resource checks
- Range checks
- State checks

#### 8.4 Delta Compression with Authority

Delta compression maintains authority:

**Delta Authority:**
- Server calculates delta
- Server sends delta to clients
- Clients apply delta to local state
- Server maintains authoritative state
- Clients interpolate between updates

**Benefits:**
- Reduced bandwidth (60-75% reduction)
- Maintained authority
- Smooth client interpolation
- Consistent game state

### 9. Buffs and Debuffs

#### 9.1 State System

Entities use bitwise flags for buffs/debuffs:

**Entity States:**
- `IsFrozen` (0x01): Entity is frozen
- `IsPoisoned` (0x02): Entity is poisoned
- `IsBurning` (0x04): Entity is burning
- `IsSlowed` (0x08): Entity is slowed
- `IsStunned` (0x10): Entity is stunned
- `IsInvisible` (0x20): Entity is invisible
- `IsInvulnerable` (0x40): Entity is invulnerable

#### 9.2 Entity State 1-to-1

Fast enemy/friend detection using EntityState:

**State Detection:**
1. Calculate EntityState flags
2. Check entity type and faction
3. Determine enemy/friend status
4. Update EntityState flags
5. Broadcast state to interested players

**Benefits:**
- O(1) enemy/friend detection
- Real-time state updates
- Efficient serialization
- Extensible state system

#### 9.3 Buff/Debuff Updates

Buffs/debuffs are updated in real-time:

**Update Process:**
1. Apply buff/debuff effect
2. Update EntityState flags
3. Calculate duration and intensity
4. Broadcast state update
5. Remove buff/debuff on expiration
6. Update EntityState flags

### 10. Packet Types

#### 10.1 Server Packets

**ServerPackets:**
- `Benchmark` (0): Benchmark test packet
- `CreateEntity` (1): Create entity packet
- `UpdateEntity` (2): Update entity packet
- `RemoveEntity` (3): Remove entity packet
- `UpdateEntityQuantized` (4): Quantized entity update
- `RekeyRequest` (5): Request key rotation
- `DeltaSync` (6): Delta synchronization packet

#### 10.2 Client Packets

**ClientPackets:**
- `SyncEntity` (0): Entity synchronization
- `SyncEntityQuantized` (1): Quantized entity sync
- `Pong` (2): Heartbeat response
- `EnterToWorld` (3): Enter world request
- `RekeyResponse` (4): Key rotation response

#### 10.3 Control Packets

**Control Packets:**
- `Connect` (0): Connection request
- `Cookie` (1): Cookie response
- `ConnectionAccepted` (2): Connection accepted
- `ConnectionDenied` (3): Connection denied
- `Ping` (4): Heartbeat request
- `Pong` (5): Heartbeat response
- `Disconnect` (6): Disconnection request
- `Ack` (7): Acknowledgment
- `Fragment` (8): Fragment packet
- `ReliableHandshake` (9): Reliable channel handshake
- `CryptoTest` (10): Encryption test
- `CryptoTestAck` (11): Encryption test acknowledgment
- `CheckIntegrity` (12): Integrity check
- `RekeyRequest` (13): Key rotation request
- `RekeyResponse` (14): Key rotation response

### 11. Rust Implementation Considerations

#### 11.1 Performance Targets

**Rust Implementation Goals:**
- 60 FPS server tick rate
- 600,000 UDP packets per second
- 10,000 concurrent players
- < 50ms latency (p99)
- Zero-allocation packet processing
- Lock-free data structures

#### 11.2 Recommended Libraries

**Network:**
- `tokio`: Async runtime for network I/O
- `socket2`: Low-level socket control
- `mio`: Metal I/O for efficient polling
- `quinn`: QUIC implementation (optional)

**Serialization:**
- `bincode`: Binary serialization
- `postcard`: Compact binary format
- Custom FlatBuffer implementation

**Cryptography:**
- `ring`: Cryptographic primitives
- `chacha20poly1305`: ChaCha20-Poly1305 AEAD
- `x25519-dalek`: X25519 key exchange

**Compression:**
- `lz4`: LZ4 compression
- `zstd`: Zstandard compression (optional)

**Data Structures:**
- `dashmap`: Concurrent hash map
- `crossbeam`: Lock-free data structures
- `parking_lot`: Efficient locks

#### 11.3 Code Patterns

**Zero-Allocation:**
- Use stack allocation where possible
- Pre-allocate buffers
- Use object pooling
- Avoid Vec allocations in hot paths

**Lock-Free:**
- Use atomic operations
- Use lock-free queues
- Use RCU (Read-Copy-Update) patterns
- Minimize locking overhead

**Async/Await:**
- Use tokio for async I/O
- Use channels for message passing
- Use futures for concurrent operations
- Avoid blocking operations

#### 11.4 Unreal Engine Integration

**C++ Client:**
- Native C++ deserializer
- Efficient packet processing
- Hardware-accelerated CRC32C
- Optimized memory management

**Plugin Integration:**
- Unreal Engine plugin for network
- Blueprint support
- C++ API for custom logic
- Automatic code generation

**Client-Server Synchronization:**
For detailed documentation on client-server synchronization, entity interpolation, animation synchronization, and Unreal Engine plugin integration, see:
- [ENTITY_SYNC.md](../client/ENTITY_SYNC.md) - Entity synchronization documentation
- [SUBSYSTEM.md](../client/SUBSYSTEM.md) - Unreal Engine subsystem architecture documentation

**Key Features:**
- Entity synchronization architecture (Unreal Engine plugin)
- Position interpolation (smooth interpolation, extrapolation)
- Animation synchronization (state machine, blending)
- Quantized position system (reduce bandwidth)
- Delta compression (only send changed data)
- Client-side prediction (reduce latency)
- Network replication system (AOI-based updates)

### 12. Configuration

#### 12.1 Server Configuration

**Network Configuration:**
- Port: 3565 (configurable)
- Max packet size: 1200 bytes
- Receive buffer: 512 KB
- Send buffer: 512 KB
- Send thread count: 1 (configurable)

**Security Configuration:**
- End-to-end encryption: Enabled
- Integrity check: Enabled
- WAF: Enabled
- Heartbeat: Enabled (5s interval)
- Heartbeat timeout: 15s

**Performance Configuration:**
- Tick rate: 60 Hz (configurable)
- Bundle timeout: 50ms
- Reliable timeout: 250ms
- Max retries: 10

**AOI Configuration:**
- Enabled: True
- Base distance: 100,000 units (1km)
- Entity distances: Configurable per type
- Adaptive sync: Enabled

#### 12.2 World Origin Rebasing Configuration

**Map Configuration:**
- Section size: 25,600 x 25,600 x 6,400 units
- Sections per component: 4 x 4 x 1
- Number of components: 8 x 8 x 4
- Scale: 100 x 100 x 100 units

**Benefits:**
- Support for large worlds
- 60% bandwidth reduction
- Configurable per map
- Seamless coordinate conversion

## Evolution and Lessons Learned

### Historical Versions

The server architecture has evolved over 4 years of development and testing, with multiple versions providing valuable insights into performance, stability, and reliability trade-offs.

#### Version 1: C# Server3 (Initial Implementation)

**Architecture:**
- **ByteBuffer System**: Linked list with object pooling
- **Per-Player Buffers**: Each player had their own buffer
- **Accumulative Send/Receive**: Packets accumulated in buffers before sending
- **Send Criteria**: Size threshold or timeout-based sending
- **Implementation**: Unsafe pointers, multiple threads, direct memory access

**Strengths:**
- **Excellent Performance**: Direct pointer manipulation provided maximum speed
- **Simple Management**: Easy to understand and expand
- **Efficient Memory**: Object pooling reduced allocations
- **High Throughput**: Multiple threads handled packet processing efficiently
- **Scalable Design**: Simple architecture allowed for easy expansion

**Critical Issues:**
- **Byte Alignment Errors**: Frequent misalignment issues when reading/writing data
- **Packet Loss**: Any serialization/deserialization error caused complete packet loss
- **Client Side Effects**: Errors in buffer operations caused strange client behaviors
- **Data Corruption**: Byte misalignment could corrupt entire packet streams
- **Debugging Difficulty**: Hard to trace and fix alignment-related bugs

**Root Causes:**
- Direct pointer arithmetic without proper alignment checks
- No packet boundaries or isolation between packets
- Lack of validation during serialization/deserialization
- Buffer position management errors
- Endianness and alignment assumptions

#### Version 2: TypeScript Implementation

**Architecture:**
- **QueueBuffer System**: Packet queuing with isolation markers
- **Packet Isolation**: Extra bytes (0xFE repeated 4 times) between packets
- **Duplicate Detection**: Packet deduplication to prevent resends
- **Size-Based Sending**: 512KB buffer size threshold
- **Timeout-Based Sending**: Periodic flush of queued packets

**Strengths:**
- **Improved Stability**: Packet isolation prevented alignment errors
- **Error Resilience**: Packet boundaries allowed recovery from errors
- **Easier Debugging**: Clear packet boundaries made issue identification easier
- **Client Compatibility**: More forgiving to serialization errors
- **Reduced Data Corruption**: Isolated packets prevented cascading errors

**Trade-offs:**
- **Performance Impact**: Slower than C# version due to extra overhead
- **Bandwidth Overhead**: Extra bytes (4 bytes per packet) increased bandwidth usage
- **Processing Overhead**: Packet isolation and validation added CPU overhead
- **Memory Usage**: Higher memory usage due to packet queuing

**Implementation Details:**
- Packet type `ServerPacketType.Queue` for batched packets
- End-of-packet marker: `0xFE` repeated 4 times
- Maximum buffer size: 512KB
- Duplicate packet detection using hex comparison
- Periodic tick-based packet flushing

#### Version 3: C# Server-LastCSharp (Current)

**Architecture:**
- **FlatBuffer System**: Zero-allocation binary serialization
- **Packet Header**: 14-byte header with connection ID, channel, flags, sequence
- **Encryption**: ChaCha20-Poly1305 AEAD encryption
- **Fragmentation**: Automatic fragmentation for packets > 1200 bytes
- **Reliable UDP**: Acknowledgment system with retransmission

**Improvements:**
- **Better Serialization**: Structured FlatBuffer with proper alignment
- **Security**: End-to-end encryption with replay protection
- **Reliability**: Reliable UDP with acknowledgment system
- **Performance**: Zero-allocation operations with unsafe pointers
- **Scalability**: AOI system for efficient entity replication

**Remaining Challenges:**
- **GC Pauses**: C# garbage collector affects performance at scale
- **Memory Allocation**: Some allocations still occur in hot paths
- **Packet Processing**: Bottleneck at high packet rates
- **Entity Updates**: Limited scalability with many players

### Key Lessons Learned

#### 1. Byte Alignment is Critical

**Problem:**
- Direct pointer arithmetic without alignment checks causes misalignment
- Different data types require different alignment (1, 2, 4, 8 bytes)
- Platform-specific alignment requirements (x86, x64, ARM)

**Solution:**
- Use structured serialization with proper alignment
- Validate alignment before reading/writing
- Use `#[repr(C)]` or `#[repr(packed)]` in Rust for explicit alignment
- Consider using `#[repr(align(N))]` for specific alignment requirements

#### 2. Packet Boundaries are Essential

**Problem:**
- Without packet boundaries, errors cascade through the buffer
- One misaligned read corrupts all subsequent reads
- Difficult to recover from serialization errors

**Solution:**
- Add packet boundaries (markers or length prefixes)
- Isolate packets to prevent error propagation
- Validate packet integrity before processing
- Use packet headers with size information

#### 3. Performance vs. Stability Trade-offs

**Problem:**
- Maximum performance (direct pointers) sacrifices stability
- Maximum stability (packet isolation) sacrifices performance
- Need to balance both requirements

**Solution:**
- Use structured serialization (FlatBuffer) for stability
- Maintain zero-allocation operations for performance
- Add validation layers that can be disabled in release builds
- Use Rust's type system to prevent alignment errors at compile time

#### 4. Error Recovery is Important

**Problem:**
- Single serialization error causes complete packet loss
- No mechanism to recover from partial packet corruption
- Client-side errors propagate to server state

**Solution:**
- Implement packet validation (CRC32C checksums)
- Add error recovery mechanisms (retransmission, resync)
- Validate packet integrity before processing
- Use reliable UDP for critical packets

#### 5. Buffer Management is Complex

**Problem:**
- Manual buffer management is error-prone
- Pool management requires careful synchronization
- Buffer lifecycle management is complex

**Solution:**
- Use Rust's ownership system for automatic buffer management
- Implement buffer pooling with proper cleanup
- Use RAII (Resource Acquisition Is Initialization) patterns
- Leverage Rust's memory safety to prevent buffer errors

### Rust Implementation Considerations

Based on the lessons learned from previous versions, the Rust implementation should:

1. **Use Structured Serialization**:
   - Leverage Rust's type system for safe serialization
   - Use `#[repr(C)]` for explicit alignment control
   - Use `#[repr(packed)]` for tightly packed structures when needed
   - Use `#[repr(align(N))]` for specific alignment requirements
   - Implement packet boundaries with length prefixes
   - Validate alignment at compile time where possible
   - Use Rust's type system to prevent alignment errors

2. **Implement Packet Isolation**:
   - Add packet boundaries (markers or length prefixes)
   - Isolate packets to prevent error propagation
   - Use packet headers with size information
   - Implement packet validation (CRC32C checksums)
   - Add packet integrity checks before processing
   - Use packet markers for error recovery (similar to TypeScript version's 0xFE markers)

3. **Balance Performance and Stability**:
   - Use zero-allocation operations where possible
   - Add validation layers that can be disabled in release builds
   - Leverage Rust's type system to prevent errors at compile time
   - Use unsafe code only when necessary and well-documented
   - Maintain structured serialization for stability
   - Use direct memory access for performance-critical paths

4. **Implement Error Recovery**:
   - Add packet validation (CRC32C checksums) for all packets
   - Implement error recovery mechanisms (retransmission, resync)
   - Validate packet integrity before processing
   - Use reliable UDP for critical packets
   - Add packet boundaries for error isolation
   - Implement packet deduplication to prevent resends

5. **Simplify Buffer Management**:
   - Use Rust's ownership system for automatic buffer management
   - Implement buffer pooling with proper cleanup
   - Use RAII patterns for resource management
   - Leverage Rust's memory safety to prevent buffer errors
   - Use `Vec<u8>` or `Box<[u8]>` for buffer management
   - Implement buffer lifecycle management with Rust's ownership

6. **Avoid Historical Issues**:
   - **Byte Alignment**: Use `#[repr(C)]` and validate alignment
   - **Packet Boundaries**: Always use length prefixes or markers
   - **Error Recovery**: Implement validation and recovery mechanisms
   - **Buffer Management**: Use Rust's ownership for automatic management
   - **Performance**: Balance performance and stability requirements

## Conclusion

This architecture provides a high-performance foundation for a large-scale MMORPG server. The design prioritizes:

1. **Performance**: Zero-allocation, lock-free, batch processing
2. **Security**: End-to-end encryption, replay protection, integrity checks
3. **Scalability**: AOI system, entity pooling, adaptive sync
4. **Bandwidth Efficiency**: Quantization, compression, delta compression
5. **Reliability**: Reliable UDP, fragmentation, heartbeat system

The Rust implementation will leverage these architectural patterns and lessons learned from previous versions to achieve the target performance metrics while maintaining security, stability, and scalability. By using Rust's type system and memory safety guarantees, we can avoid the byte alignment errors and buffer management issues that plagued earlier versions while maintaining the high performance of direct memory access.
