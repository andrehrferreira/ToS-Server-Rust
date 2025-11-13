# Player Packets Documentation

## Overview

Player packets handle player-specific data synchronization, including character creation, stats updates, equipment changes, skill experience, auto-attack, and player actions. These packets manage the player's state, appearance, and interactions.

## Server to Client Packets

### CharacterList

**Type:** `ServerPacketType.CharacterList`

**Purpose:** Send list of characters for player selection

**Data:**
```
[PacketType: byte]
[CharacterListData: string (JSON)]
```

**Usage:**
- Player login
- Character selection screen
- Character list refresh
- Direct socket send

**Example:**
```typescript
packetCharacterList.send(socket, characterListJson);
```

### FullCharacter

**Type:** `ServerPacketType.FullCharacter`

**Purpose:** Send complete character data

**Data:**
```
[PacketType: byte]
[CharacterData: string (JSON)]
```

**Usage:**
- Character selection
- Character load
- Character data refresh
- Direct socket send

**Example:**
```typescript
packetFullCharacter.send(socket, characterDataJson);
```

### CreateCharacter

**Type:** `ServerPacketType.CreateCharacter`

**Purpose:** Notify client of new character entering visibility range

**Data:**
```
[PacketType: byte]
[CharacterId: id (4 bytes)]
[CharacterName: string (max 14 chars)]
[Visual: string]
[Equipment: string (JSON)]
[Flags: int32]
[PositionX: int32]
[PositionY: int32]
[PositionZ: int32]
[RotationYaw: int32]
[MaxLife: int32]
[CurrentLife: int32]
[CharacterId: string]
[BuffsDebuffsFlags: int32]
[GuildName: string]
```

**Equipment JSON Structure:**
```json
{
    "helmet": "ItemName" | null,
    "chest": "ItemName" | null,
    "gloves": "ItemName" | null,
    "pants": "ItemName" | null,
    "boots": "ItemName" | null,
    "robe": "ItemName" | null,
    "cloak": "ItemName" | null,
    "mainhand": "ItemName" | null,
    "offhand": "ItemName" | null
}
```

**Conditions:**
- Character is Humanoid
- Entity has socket
- Not self entity
- Queued for batch transmission

**Usage:**
- Player enters visibility range
- New player spawns
- Player loads in scene
- Character appearance update

**Example:**
```typescript
packetCreateCharacter.send(owner, entity);
```

### PlayerStats

**Type:** `ServerPacketType.PlayerStats`

**Purpose:** Update player life, mana, and stamina

**Data:**
```
[PacketType: byte]
[Life: int32]
[Mana: int32]
[Stamina: int32]
```

**Conditions:**
- Player has socket
- Player not removed
- Direct socket send (immediate)

**Usage:**
- Stats change
- Resource regeneration
- Resource consumption
- Health/mana/stamina updates

**Example:**
```typescript
packetPlayerStats.send(player);
```

### UpdateStats

**Type:** `ServerPacketType.UpdateStats`

**Purpose:** Update player attribute stats

**Data:**
```
[PacketType: byte]
[Strength: int32]
[Dexterity: int32]
[Intelligence: int32]
[Vigor: int32]
[Agility: int32]
[Luck: int32]
```

**Conditions:**
- Player has socket
- Player not removed
- Direct socket send

**Usage:**
- Stat point allocation
- Stat changes from items
- Stat changes from buffs
- Character progression

**Example:**
```typescript
packetUpdateStats.send(player);
```

### Equip

**Type:** `ServerPacketType.Equip`

**Purpose:** Notify equipment change

**Data:**
```
[PacketType: byte]
[CharacterId: id (4 bytes)]
[EquipmentType: byte]
[ItemId: string]
```

**Equipment Types:**
- Helmet
- Chest
- Gloves
- Pants
- Boots
- Robe
- Cloak
- MainHand
- OffHand
- Ring
- Ring2

**Conditions:**
- Entity has socket
- Player not removed
- Player not dead
- Direct socket send

**Usage:**
- Item equipped
- Equipment change
- Visual update
- Stat update

**Example:**
```typescript
packetEquip.send(owner, entity, { type: EquipmentType.Helmet, itemId: "IronHelmet" });
```

### Desequip

**Type:** `ServerPacketType.Desequip`

**Purpose:** Notify equipment removal

**Data:**
```
[PacketType: byte]
[CharacterId: id (4 bytes)]
[EquipmentType: byte]
[Ring02: bool]
```

**Conditions:**
- Entity has socket
- Player not removed
- Player not dead
- Direct socket send

**Usage:**
- Item unequipped
- Equipment removal
- Visual update
- Stat update

**Example:**
```typescript
packetDesequip.send(owner, entity, { type: EquipmentType.Helmet, ring02: false });
```

### AutoAttack

**Type:** `ServerPacketType.AutoAttack`

**Purpose:** Notify auto-attack execution

**Data:**
```
[PacketType: byte]
[CharacterId: id (4 bytes)]
```

**Conditions:**
- Entity has socket
- Player not removed
- Player not dead
- Queued for batch transmission

**Usage:**
- Auto-attack trigger
- Basic attack animation
- Combat synchronization
- Attack feedback

**Example:**
```typescript
packetAutoAttack.send(owner, entity);
```

### SkillExperience

**Type:** `ServerPacketType.UpdateSkillExperience`

**Purpose:** Update skill experience and level

**Data:**
```
[PacketType: byte]
[SkillName: byte]
[SkillValue: int32]
[SkillExperience: int32]
[SkillCap: int32]
```

**Conditions:**
- Entity has socket
- Direct socket send

**Usage:**
- Skill experience gain
- Skill level up
- Skill progress update
- Skill cap changes

**Example:**
```typescript
packetSkillExperience.send(owner, SkillName.Blacksmithing, skill);
```

### UpdateSkillInfo

**Type:** `ServerPacketType.UpdateSkillInfo`

**Purpose:** Send complete skill information

**Data:**
```
[PacketType: byte]
[SkillsData: string (JSON)]
```

**Conditions:**
- Entity exists and has socket
- Direct socket send

**Usage:**
- Initial skill load
- Skill data refresh
- Skill system update

**Example:**
```typescript
packetUpdateSkillInfo.send(owner);
```

### PlayerStatics

**Type:** `ServerPacketType.PlayerStatics`

**Purpose:** Send player statistics

**Data:**
```
[PacketType: byte]
[StaticsData: string (JSON)]
```

**Conditions:**
- Player has socket
- Player not removed
- Player not dead
- Direct socket send

**Usage:**
- Statistics display
- Achievement tracking
- Player profile
- Statistics update

**Example:**
```typescript
packetPlayerStatics.send(player);
```

### Unstuck

**Type:** `ServerPacketType.Unstuck`

**Purpose:** Teleport player to safe location

**Data:**
```
[PacketType: byte]
[PositionData: string (JSON)]
```

**Usage:**
- Player stuck
- Unstuck command
- Emergency teleport
- Direct socket send

**Example:**
```typescript
packetUnstuck.send(socket, positionJson);
```

## Client to Server Handlers

### Move

**Type:** `ClientPacketType.Move`

**Purpose:** Send player movement input

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates position (not NaN)
- Updates player movement
- Server validates movement
- Server calculates new position

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (!float.IsNaN(x) && !float.IsNaN(y) && player != null) {
    player.Move(new Vector2(x, y));
}
```

### MoveTo

**Type:** `ClientPacketType.MoveTo`

**Purpose:** Request movement to specific position

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates position
- Initiates pathfinding
- Moves player to target
- Handles pathfinding obstacles

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (!float.IsNaN(x) && !float.IsNaN(y) && player != null) {
    player.MoveTo(new Vector2(x, y));
}
```

### Rotate

**Type:** `ClientPacketType.Rotate`

**Purpose:** Send player rotation input

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates rotation
- Checks if player can rotate
- Updates player rotation
- Rotates towards target position

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (player != null && !player.IsMovingTo && player.CanRotate == 0) {
    player.RotateTowards(new Vector2(x, y));
}
```

### AutoAttack

**Type:** `ClientPacketType.AutoAttack`

**Purpose:** Trigger auto-attack

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates attack position
- Executes primary action
- Handles attack cooldown
- Processes attack logic

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

player.Execute(player.PrimaryAction, new Vector2(x, y));
```

### EquipItem

**Type:** `ClientPacketType.EquipItem`

**Purpose:** Equip item from inventory

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Checks equipment requirements
- Validates equipment slot
- Equips item
- Updates stats

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (player != null) {
    player.EquipItem(itemId);
}
```

### UnequipItem

**Type:** `ClientPacketType.UnequipItem`

**Purpose:** Unequip item

**Data:**
```
[EquipmentSlot: varuint]
```

**Handler Logic:**
- Validates slot
- Unequips item
- Returns to inventory
- Updates stats

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (player != null) {
    player.UnequipItem((EquipmentItemSlot)itemId);
}
```

### UseItem

**Type:** `ClientPacketType.UseItem`

**Purpose:** Use consumable item

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Checks if item is consumable
- Validates use conditions
- Applies item effects
- Removes item if consumed

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

player.Inventory.UseItem(player, itemId);
```

### DestroyItem

**Type:** `ClientPacketType.DestroyItem`

**Purpose:** Destroy item from inventory

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Removes item from inventory
- Item permanently destroyed
- No recovery possible

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

player.Inventory.TryRemove(itemId);
```

### StatMod

**Type:** `ClientPacketType.StatMod`

**Purpose:** Modify stat allocation

**Data:**
```
[StatName: byte]
[StatMod: byte]
```

**Handler Logic:**
- Validates stat name
- Validates mod type
- Updates stat modifier
- Recalculates stats

**Stat Names:**
- Strength
- Dexterity
- Intelligence

**Stat Mods:**
- Increase
- Decrease
- Reset

**Example:**
```csharp
StatName stat = (StatName)buffer.GetByte();
StatMod mod = (StatMod)buffer.GetByte();

switch (stat) {
    case StatName.Strength:
        player.StrengthMod = mod;
        break;
    case StatName.Dexterity:
        player.DexterityMod = mod;
        break;
    case StatName.Intelligence:
        player.IntelligenceMod = mod;
        break;
}
```

### UnmountRequest

**Type:** `ClientPacketType.UnmountRequest`

**Purpose:** Request to unmount

**Data:**
```
(no data)
```

**Handler Logic:**
- Validates player is mounted
- Unmounts player
- Sets global cooldown
- Updates movement speed

**Example:**
```csharp
if (player != null) {
    player.Unmount();
    player.SetGlobalCooldown(TimeSpan.FromSeconds(3));
}
```

## Packet Flow Examples

### Character Creation Flow

1. Client: Player creates character
2. Client: Sends character creation data
3. Server: Validates character data
4. Server: Creates character entity
5. Server: Sends `CreateCharacter` packet to nearby players
6. Client: Receives `CreateCharacter` packet
7. Client: Creates character representation
8. Client: Updates character appearance

### Equipment Change Flow

1. Client: Player drags item to equipment slot
2. Client: Sends `EquipItem` packet
3. Server: Validates equipment
4. Server: Equips item
5. Server: Updates player stats
6. Server: Sends `Equip` packet to nearby players
7. Client: Receives `Equip` packet
8. Client: Updates character appearance
9. Client: Updates equipment display

### Auto-Attack Flow

1. Client: Player holds attack button
2. Client: Sends `AutoAttack` packet repeatedly
3. Server: Validates attack
4. Server: Executes attack
5. Server: Calculates damage
6. Server: Sends `AutoAttack` packet to nearby players
7. Server: Sends damage packets if hit
8. Client: Receives `AutoAttack` packet
9. Client: Plays attack animation

## Implementation Notes

### Character ID System

**Character ID Format:**
- GUID-based identifier
- Converted to int32 for packet transmission
- Unique per character instance
- Persistent across sessions

**ID Conversion:**
```typescript
// Put ID
putId(character.characterId)  // GUID -> int32

// Get ID
const id = getId()  // int32 -> GUID
```

### Equipment Serialization

**Equipment JSON:**
- Compact JSON format
- Only equipped items included
- Null for empty slots
- Item names as strings

**Equipment Update:**
- Only changed equipment sent
- Full equipment on character creation
- Partial updates for equipment changes

### Stats System

**Attribute Stats:**
- Strength (STR)
- Dexterity (DEX)
- Intelligence (INT)
- Vigor (VIG)
- Agility (AGI)
- Luck (LUC)

**Resource Stats:**
- Life (HP)
- Mana (MP)
- Stamina (SP)

**Stat Updates:**
- Full stat update on change
- Immediate transmission
- Client-side stat calculation
- Server-side validation

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [ENTITY_PACKETS.md](./ENTITY_PACKETS.md) - Entity synchronization packets
- [CONTAINER_PACKETS.md](./CONTAINER_PACKETS.md) - Container management packets
- [PLAYER.md](../player/PLAYER.md) - Player system documentation

