# Entity Packets Documentation

## Overview

Entity packets handle synchronization of entities (players, NPCs, monsters, items) between server and clients. These packets manage entity lifecycle (creation, updates, removal), state synchronization, combat events, and visual effects.

## Server to Client Packets

### CreateEntity

**Type:** `ServerPacketType.CreateEntity`

**Purpose:** Notify client of new entity entering visibility range

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[EntityName: string]
[VisualData: string (JSON)]
[Flags: int32]
[PositionX: int32]
[PositionY: int32]
[PositionZ: int32]
[MaxLife: int32]
[CurrentLife: int32]
[Speed: int32]
[BuffsDebuffsFlags: int32]
[GuildName: string]
```

**Usage:**
- Entity enters player's visibility range
- New entity spawns
- Entity loads in scene
- Queued for batch transmission

**Example:**
```typescript
packetCreateEntity.send(owner, entity);
```

### UpdateEntity

**Type:** `ServerPacketType.UpdateEntity`

**Purpose:** Update entity state (position, health, flags)

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[Flags: int32]
[PositionX: int32]
[PositionY: int32]
[PositionZ: int32]
[CurrentLife: int32]
[BuffsDebuffsFlags: int32]
```

**Conditions:**
- Entity not dead
- Entity not removed
- Entity in different map than observer
- Queued for batch transmission

**Usage:**
- Position updates
- Health changes
- Flag changes
- Buff/debuff updates

**Example:**
```typescript
packetUpdateEntity.send(owner, entity, teleport);
```

### RemoveEntity

**Type:** `ServerPacketType.RemoveEntity`

**Purpose:** Notify client entity is leaving visibility or being destroyed

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
```

**Conditions:**
- Entity has valid mapIndex
- Direct socket send (immediate)

**Usage:**
- Entity leaves visibility range
- Entity destroyed
- Entity despawns
- Entity removed from scene

**Example:**
```typescript
packetRemoveEntity.send(owner, entity);
```

### EntityDie

**Type:** `ServerPacketType.Die`

**Purpose:** Notify client entity has died

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
```

**Conditions:**
- Entity exists and has socket
- Queued for batch transmission

**Usage:**
- Player death
- Creature death
- Death animations
- Death state updates

**Example:**
```typescript
packetEntityDie.send(owner, entity);
```

### SyncEvent

**Type:** `ServerPacketType.SyncEvent`

**Purpose:** Synchronize entity events (animations, effects)

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[EventType: byte]
```

**Conditions:**
- Entity has socket
- Entity in different map than owner
- Queued for batch transmission

**Usage:**
- Animation events
- Effect triggers
- State changes
- Visual synchronization

**Example:**
```typescript
packetEventEntity.send(owner, entity, eventType);
```

### EventReviveEntity

**Type:** `ServerPacketType.SyncEvent`

**Purpose:** Notify entity revival event

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[EventType: byte]
```

**Conditions:**
- Entity exists and has socket
- Direct socket send (immediate)

**Usage:**
- Player revival
- Resurrection effects
- Revive animations

**Example:**
```typescript
packetEventReviveEntity.send(owner, eventType);
```

### PlayMontage

**Type:** `ServerPacketType.PlayMontage`

**Purpose:** Trigger animation montage on entity

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[MontageIndex: int32]
```

**Conditions:**
- If entity != owner: entity has socket, direct send
- If entity == owner: owner has socket, direct send

**Usage:**
- Animation playback
- Action animations
- Effect animations
- Montage sequences

**Example:**
```typescript
packetPlayMontageEntity.send(owner, entity, montageIndex);
```

### Action

**Type:** `ServerPacketType.Action`

**Purpose:** Notify entity action execution

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[ActionIndex: byte]
```

**Conditions:**
- Entity has socket
- Queued for batch transmission

**Usage:**
- Action execution
- Skill casting
- Ability usage
- Action synchronization

**Example:**
```typescript
packetActionEntity.send(owner, entity, actionIndex);
```

### ActionArea

**Type:** `ServerPacketType.ActionArea`

**Purpose:** Notify area action execution

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[ActionIndex: byte]
[PositionX: int32]
[PositionY: int32]
[PositionZ: int32]
```

**Conditions:**
- Entity has socket
- Queued for batch transmission

**Usage:**
- Area effect actions
- Ground-targeted abilities
- Position-based actions
- Area damage spells

**Example:**
```typescript
packetActionAreaEntity.send(owner, entity, { index, position });
```

### SelectTarget

**Type:** `ServerPacketType.SelectTarget`

**Purpose:** Notify target selection

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[TargetId: id (4 bytes)]
```

**Conditions:**
- Entity has socket
- Queued for batch transmission

**Usage:**
- Target selection
- Target lock
- Target highlighting
- Combat targeting

**Example:**
```typescript
packetSelectTargetEntity.send(owner, entity, targetId);
```

### CancelTarget

**Type:** `ServerPacketType.CancelTarget`

**Purpose:** Notify target cancellation

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
```

**Conditions:**
- Entity has socket
- Entity != owner
- Direct socket send

**Usage:**
- Target unlock
- Target cancellation
- Target clearing

**Example:**
```typescript
packetCancelTargetEntity.send(owner, entity);
```

### TakeDamage

**Type:** `ServerPacketType.TakeDamage`

**Purpose:** Notify entity taking damage

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[CauserId: id (4 bytes)]
[Damage: int32]
[DamageType: byte]
```

**Conditions:**
- Entity has socket
- Queued for batch transmission

**Usage:**
- Damage events
- Health updates
- Damage type display
- Combat feedback

**Example:**
```typescript
packetTakeDamageEntity.send(owner, entity, {
    damage,
    damageType,
    causer
});
```

### TakeMiss

**Type:** `ServerPacketType.TakeMiss`

**Purpose:** Notify attack missed

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
```

**Conditions:**
- Entity has socket
- Queued for batch transmission

**Usage:**
- Miss feedback
- Dodge animations
- Evasion display

**Example:**
```typescript
packetTakeMissEntity.send(owner, entity);
```

### Heal

**Type:** `ServerPacketType.Heal`

**Purpose:** Notify entity healing

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
[HealValue: int32]
[CasterId: id (4 bytes)]
[HealType: byte]
```

**Conditions:**
- Entity has socket
- HealType defaults to Life if not provided
- Queued for batch transmission

**Usage:**
- Healing events
- Health restoration
- Mana/stamina restoration
- Regeneration display

**Example:**
```typescript
packetHealEntity.send(owner, entity, {
    caster,
    value,
    type
});
```

### Dissolve

**Type:** `ServerPacketType.Dissolve`

**Purpose:** Trigger entity dissolve effect (death animation)

**Data:**
```
[PacketType: byte]
[EntityId: id (4 bytes)]
```

**Conditions:**
- Entity has socket
- Direct socket send

**Usage:**
- Death dissolve effect
- Entity removal animation
- Corpse decay

**Example:**
```typescript
packetDissolveEntity.send(owner, entity);
```

## Client to Server Handlers

### EntityTickUpdate

**Type:** `ClientPacketType.EntityTickUpdate`

**Purpose:** Client acknowledges entity tick update (reliability system)

**Data:**
```
[EntityId: int]
[TickNumber: uint]
```

**Handler Logic:**
- Updates last acknowledged tick
- Removes entity from unreliable list if destroyed and acknowledged
- Used for entity reliability tracking

**Example:**
```csharp
int id = buffer.GetInt();
uint entityTickNumber = buffer.GetUInt();
// Update acknowledgment
```

## Packet Flow Examples

### Entity Creation Flow

1. Server: Entity spawns in scene
2. Server: `CreateEntity` packet queued for nearby players
3. Client: Receives `CreateEntity` packet
4. Client: Creates entity representation
5. Client: Updates entity state

### Entity Update Flow

1. Server: Entity position/state changes
2. Server: `UpdateEntity` packet queued
3. Client: Receives `UpdateEntity` packet
4. Client: Updates entity visual representation
5. Client: Interpolates movement

### Entity Death Flow

1. Server: Entity dies (health <= 0)
2. Server: `EntityDie` packet queued
3. Server: `Dissolve` packet sent (after delay)
4. Client: Receives `EntityDie` packet
5. Client: Plays death animation
6. Client: Receives `Dissolve` packet
7. Client: Plays dissolve effect
8. Server: `RemoveEntity` packet sent
9. Client: Removes entity

## Implementation Notes

### State Flags

**Entity Flags:**
- Stored as 32-bit integer
- Bit flags for various states
- Includes: movement, combat, status effects

**Buffs/Debuffs Flags:**
- Stored as 32-bit integer
- Bit flags for buff/debuff states
- Includes: poison, freeze, invulnerability, etc.

### Position Encoding

**Position Format:**
- Stored as int32 (scaled float)
- Precision: 1/1000th unit
- Range: -2,147,483.648 to 2,147,483.647

**Position Conversion:**
```typescript
// Server to packet
putInt32(Math.floor(position.x * 1000))

// Packet to client
position.x = getInt32() / 1000
```

### Entity ID System

**ID Format:**
- GUID converted to 32-bit integer
- Uses GUID mapping system
- Unique per entity instance

**ID Conversion:**
```typescript
// Put ID
putId(entity.mapIndex)  // GUID -> int32

// Get ID
const id = getId()  // int32 -> GUID
```

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player-specific packets
- [BYTEBUFFER.md](../core/BYTEBUFFER.md) - ByteBuffer serialization

