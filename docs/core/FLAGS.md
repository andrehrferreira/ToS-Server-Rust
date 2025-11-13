# Flags System Documentation

## Overview

The Flags system is a bitwise state management system used to track entity states, buffs/debuffs, and item states. This system is implemented on both the server and client side to ensure consistency and enable efficient state synchronization.

## Architecture

### Bitwise Representation

Flags use bitwise operations (bit flags) to efficiently store multiple boolean states in a single integer value. Each flag occupies a single bit, allowing up to 32 flags in a single `uint32` (or 64 flags in a `uint64`).

**Benefits:**
- **Compact Storage**: Multiple states stored in a single integer
- **Fast Operations**: Bitwise operations are extremely fast (O(1))
- **Efficient Serialization**: Single integer value for network transmission
- **Memory Efficient**: Minimal memory overhead
- **Easy Comparison**: Quick state checks with bitwise AND operations

### Flag Operations

**Core Operations:**
- `AddFlag(flag)`: Set a flag using bitwise OR (`flags |= flag`)
- `RemoveFlag(flag)`: Clear a flag using bitwise AND with NOT (`flags &= ~flag`)
- `HasFlag(flag)`: Check if a flag is set using bitwise AND (`(flags & flag) == flag`)
- `DontHasFlag(flag)`: Check if a flag is not set (`(flags & flag) != flag`)

**Implementation:**
- **C#**: Extension methods with `[MethodImpl(MethodImplOptions.AggressiveInlining)]` for performance
- **TypeScript**: Class-based implementation with methods
- **Rust**: Will use trait-based implementation with inline functions

## Entity States (EntityStates)

Entity states represent the current state of an entity (player, NPC, monster, etc.).

### Flag Definitions

#### Version 1: C# Server3 (CreatureFlags)

```csharp
public enum CreatureFlags : uint
{
    None = 0,
    Moving = 1 << 0,          // Entity is moving
    Chilled = 1 << 1,         // Entity is chilled (slowed)
    Stunned = 1 << 2,         // Entity is stunned
    Dead = 1 << 3,            // Entity is dead
    CanRotate = 1 << 4,       // Entity can rotate
    Ally = 1 << 5,            // Entity is an ally
    Enemy = 1 << 6,           // Entity is an enemy
    NegativeKarma = 1 << 7,   // Entity has negative karma
    Burning = 1 << 8,         // Entity is burning
    Bleeding = 1 << 9,        // Entity is bleeding
    Poisoned = 1 << 10,       // Entity is poisoned
    ArcaneShield = 1 << 11,   // Entity has arcane shield
    Knockback = 1 << 12,      // Entity is being knocked back
    Stealth = 1 << 13,        // Entity is in stealth mode
    SafeZone = 1 << 14,       // Entity is in safe zone
    DuelZone = 1 << 15,       // Entity is in duel zone
    Mounted = 1 << 18,        // Entity is mounted
    Feared = 1 << 19,         // Entity is feared
    Invulnerable = 1 << 22,   // Entity is invulnerable
    Warmode = 1 << 23,        // Entity is in war mode
    FireSpray = 1 << 24,      // Entity has fire spray effect
    Vampirism = 1 << 25,      // Entity has vampirism effect
    Frozen = 1 << 26,         // Entity is frozen
    Casting = 1 << 27,        // Entity is casting a spell
    Untargettable = 1 << 28,  // Entity cannot be targeted
    Frenzy = 1 << 29,         // Entity is in frenzy mode
    Admin = 1 << 30,          // Entity is an admin
}
```

#### Version 2: TypeScript (EntityStates)

```typescript
export enum EntityStates {
    None = 0,
    Stunned = 1 << 1,         // Entity is stunned
    Dead = 1 << 2,            // Entity is dead
    Ally = 1 << 3,            // Entity is an ally
    Enemy = 1 << 4,           // Entity is an enemy
    NegativeKarma = 1 << 5,   // Entity has negative karma
    Burning = 1 << 6,         // Entity is burning
    Bleeding = 1 << 7,        // Entity is bleeding
    Poisoned = 1 << 8,        // Entity is poisoned
    Party = 1 << 9,           // Entity is in a party
    Guild = 1 << 10,          // Entity is in a guild
    Stealth = 1 << 11,        // Entity is in stealth mode
    Mounted = 1 << 12,        // Entity is mounted
    Feared = 1 << 13,         // Entity is feared
    Invulnerable = 1 << 14,   // Entity is invulnerable
    Warmode = 1 << 15,        // Entity is in war mode
    Frozen = 1 << 16,         // Entity is frozen
    Untargettable = 1 << 17,  // Entity cannot be targeted
    Frenzy = 1 << 18,         // Entity is in frenzy mode
    DuelZone = 1 << 19,       // Entity is in duel zone
    SafeZone = 1 << 20,       // Entity is in safe zone
    Admin = 1 << 21,          // Entity is an admin
    Pet = 1 << 22,            // Entity is a pet
}
```

### State Categories

**Movement States:**
- `Moving`: Entity is currently moving
- `Mounted`: Entity is mounted on a creature
- `Frozen`: Entity is frozen and cannot move
- `Stunned`: Entity is stunned and cannot act

**Combat States:**
- `Dead`: Entity is dead
- `Invulnerable`: Entity cannot take damage
- `Untargettable`: Entity cannot be targeted
- `Warmode`: Entity is in war mode (PvP enabled)
- `Frenzy`: Entity is in frenzy mode (increased damage)

**Status Effects:**
- `Burning`: Entity is taking fire damage over time
- `Bleeding`: Entity is taking bleed damage over time
- `Poisoned`: Entity is taking poison damage over time
- `Feared`: Entity is feared and runs away
- `Chilled`: Entity is chilled (slowed)

**Social States:**
- `Ally`: Entity is an ally
- `Enemy`: Entity is an enemy
- `Party`: Entity is in a party
- `Guild`: Entity is in a guild
- `NegativeKarma`: Entity has negative karma (player killer)

**Zone States:**
- `SafeZone`: Entity is in a safe zone (no PvP)
- `DuelZone`: Entity is in a duel zone (PvP allowed)
- `Stealth`: Entity is in stealth mode

**Special States:**
- `Admin`: Entity is an admin
- `Pet`: Entity is a pet
- `Casting`: Entity is casting a spell
- `CanRotate`: Entity can rotate

## Buff/Debuff States (BuffsDebuffsFlags)

Buff/debuff states represent temporary effects applied to entities.

### Flag Definitions

#### Version 1: C# Server3 (BuffsDebuffsFlags)

```csharp
public enum BuffsDebuffsFlags : uint
{
    None = 0,
    BlessingofNature = 1 << 0,    // Blessing of nature buff
    BlessingofAnimal = 1 << 1,    // Blessing of animal buff
    BlessingofCheetah = 1 << 2,   // Blessing of cheetah buff
    ManaBurn = 1 << 3,            // Mana burn debuff
}
```

#### Version 2: TypeScript (BuffDebuffStates)

```typescript
export enum BuffDebuffStates {
    None = 0,
    HeavenlyProtection = 1 << 0,  // Heavenly protection buff
    EchoOfTheDeath = 1 << 1,      // Echo of the death buff
    ShatteringImpact = 1 << 2,    // Shattering impact buff
    ShatteringImpactEffect = 1 << 3, // Shattering impact effect
    ThunderAura = 1 << 4,         // Thunder aura buff
}
```

### Buff/Debuff Categories

**Protection Buffs:**
- `HeavenlyProtection`: Provides protection from damage
- `BlessingofNature`: Provides nature-based protection
- `BlessingofAnimal`: Provides animal-based protection

**Speed Buffs:**
- `BlessingofCheetah`: Increases movement speed

**Damage Buffs:**
- `EchoOfTheDeath`: Increases damage output
- `ShatteringImpact`: Provides shattering impact damage
- `ThunderAura`: Provides thunder aura damage

**Debuffs:**
- `ManaBurn`: Burns mana over time

## Item States (ItemStates)

Item states represent the current state of items.

### Flag Definitions

#### Version 2: TypeScript (ItemStates)

```typescript
export enum ItemStates {
    None = 0,
    Blessed = 1 << 0,         // Item is blessed (cannot be dropped)
    Insured = 1 << 1,         // Item is insured (protected from loss)
    Exceptional = 1 << 2,     // Item is exceptional quality
    SpellChanneling = 1 << 3, // Item has spell channeling
    Broken = 1 << 4,          // Item is broken
}
```

### Item State Categories

**Protection States:**
- `Blessed`: Item cannot be dropped or lost
- `Insured`: Item is protected from loss on death

**Quality States:**
- `Exceptional`: Item is of exceptional quality
- `Broken`: Item is broken and cannot be used

**Special States:**
- `SpellChanneling`: Item has spell channeling capability

## Implementation

### Server-Side Implementation

**C# Implementation (Version 1):**
```csharp
public static void AddFlag(this ref CreatureFlags flags, CreatureFlags flag)
{
    flags |= flag;
}

public static void RemoveFlag(this ref CreatureFlags flags, CreatureFlags flag)
{
    flags &= ~flag;
}

public static bool HasFlag(this ref CreatureFlags flags, CreatureFlags flag)
{
    return (flags & flag) == flag;
}

public static bool DontHasFlag(this ref CreatureFlags flags, CreatureFlags flag)
{
    return (flags & flag) != flag;
}
```

**TypeScript Implementation (Version 2):**
```typescript
export class StateFlags {
    private flags: number;

    constructor(initialFlags: number = 0) {
        this.flags = initialFlags;
    }

    addFlag(flag: number): void {
        this.flags |= flag;
    }

    removeFlag(flag: number): void {
        this.flags &= ~flag;
    }

    hasFlag(flag: number): boolean {
        return (this.flags & flag) === flag;
    }

    dontHasFlag(flag: number): boolean {
        return !((this.flags & flag) === flag);
    }

    getCurrentFlags(): number {
        return this.flags;
    }
}
```

### Client-Side Implementation

The client must implement the same flag system to ensure consistency:

**Requirements:**
- Same flag definitions as server
- Same bitwise operations
- Same serialization format
- Synchronized state updates

**Implementation:**
- Use the same enum definitions
- Implement the same flag operations
- Sync flags from server updates
- Apply flags to entity rendering

## Serialization

### Network Protocol

Flags are serialized as a single `uint32` value:

```
Flag Packet Structure:
- Flags: uint32 (4 bytes) - Little Endian
```

### Serialization Format

**Write:**
```csharp
buffer.Put((uint)flags);
```

**Read:**
```csharp
uint flags = buffer.GetUInt();
```

### Compression

Flags can be compressed using bitwise operations:
- **VarInt Encoding**: Use VarInt for flags with few bits set
- **Delta Compression**: Only send changed flags
- **Bitwise Operations**: Efficient serialization with bit manipulation

## State Synchronization

### Server Authority

The server is authoritative for all flag states:
- Server determines flag changes
- Server validates flag transitions
- Server broadcasts flag updates
- Client applies flags from server

### Update Frequency

**Full Sync:**
- Entity creation (spawn)
- Entity state changes
- Significant flag changes

**Delta Sync:**
- Only changed flags
- Reduced bandwidth
- Faster updates

### Update Triggers

**Immediate Updates:**
- Combat state changes (Dead, Stunned)
- Status effect changes (Burning, Poisoned)
- Zone changes (SafeZone, DuelZone)

**Periodic Updates:**
- Movement states (Moving)
- Social states (Party, Guild)
- Buff/debuff states

## Performance Considerations

### Optimization

**Bitwise Operations:**
- Use inline functions for flag operations
- Compile-time optimization for flag checks
- Efficient bit manipulation

**Memory Usage:**
- Single integer per entity for flags
- Minimal memory overhead
- Efficient cache usage

**Network Bandwidth:**
- Single integer per flag update
- Delta compression for changed flags
- Reduced bandwidth usage

### Rust Implementation

**Recommended Approach:**
```rust
#[repr(C)]
#[derive(Clone, Copy, PartialEq, Eq)]
pub struct EntityFlags(u32);

impl EntityFlags {
    #[inline]
    pub fn add_flag(&mut self, flag: EntityFlag) {
        self.0 |= flag as u32;
    }

    #[inline]
    pub fn remove_flag(&mut self, flag: EntityFlag) {
        self.0 &= !(flag as u32);
    }

    #[inline]
    pub fn has_flag(&self, flag: EntityFlag) -> bool {
        (self.0 & flag as u32) == flag as u32
    }

    #[inline]
    pub fn dont_has_flag(&self, flag: EntityFlag) -> bool {
        (self.0 & flag as u32) != flag as u32
    }
}

#[repr(u32)]
#[derive(Clone, Copy, PartialEq, Eq)]
pub enum EntityFlag {
    None = 0,
    Stunned = 1 << 1,
    Dead = 1 << 2,
    // ... other flags
}
```

**Benefits:**
- Type-safe flag operations
- Zero-cost abstractions
- Compile-time optimization
- Memory-efficient representation

## Usage Examples

### Adding a Flag

```csharp
// C#
entity.Flags.AddFlag(CreatureFlags.Burning);
```

```typescript
// TypeScript
entity.states.addFlag(EntityStates.Burning);
```

```rust
// Rust
entity.flags.add_flag(EntityFlag::Burning);
```

### Removing a Flag

```csharp
// C#
entity.Flags.RemoveFlag(CreatureFlags.Burning);
```

```typescript
// TypeScript
entity.states.removeFlag(EntityStates.Burning);
```

```rust
// Rust
entity.flags.remove_flag(EntityFlag::Burning);
```

### Checking a Flag

```csharp
// C#
if (entity.Flags.HasFlag(CreatureFlags.Burning))
{
    // Entity is burning
}
```

```typescript
// TypeScript
if (entity.states.hasFlag(EntityStates.Burning)) {
    // Entity is burning
}
```

```rust
// Rust
if entity.flags.has_flag(EntityFlag::Burning) {
    // Entity is burning
}
```

### Multiple Flag Checks

```csharp
// C#
if (entity.Flags.HasFlag(CreatureFlags.Burning) && 
    entity.Flags.HasFlag(CreatureFlags.Poisoned))
{
    // Entity is burning and poisoned
}
```

```typescript
// TypeScript
if (entity.states.hasFlag(EntityStates.Burning) && 
    entity.states.hasFlag(EntityStates.Poisoned)) {
    // Entity is burning and poisoned
}
```

```rust
// Rust
if entity.flags.has_flag(EntityFlag::Burning) && 
   entity.flags.has_flag(EntityFlag::Poisoned) {
    // Entity is burning and poisoned
}
```

## Best Practices

### Flag Management

1. **Use Bitwise Operations**: Always use bitwise operations for flag management
2. **Validate Flag Transitions**: Validate flag state transitions on the server
3. **Synchronize Flags**: Keep server and client flags synchronized
4. **Use Delta Compression**: Only send changed flags to reduce bandwidth
5. **Cache Flag Checks**: Cache frequently used flag checks

### Performance Optimization

1. **Inline Functions**: Use inline functions for flag operations
2. **Compile-Time Optimization**: Use compile-time optimization for flag checks
3. **Efficient Bit Manipulation**: Use efficient bit manipulation techniques
4. **Memory Alignment**: Ensure proper memory alignment for flags
5. **Cache-Friendly**: Design flag operations to be cache-friendly

### Error Prevention

1. **Type Safety**: Use type-safe flag operations
2. **Validation**: Validate flag values before applying
3. **Bounds Checking**: Check flag bounds to prevent overflow
4. **State Consistency**: Ensure flag state consistency
5. **Error Recovery**: Implement error recovery for invalid flags

## Conclusion

The Flags system provides an efficient and scalable way to manage entity states, buffs/debuffs, and item states. By using bitwise operations, the system achieves:

- **Compact Storage**: Multiple states in a single integer
- **Fast Operations**: O(1) flag operations
- **Efficient Serialization**: Single integer for network transmission
- **Memory Efficient**: Minimal memory overhead
- **Easy Synchronization**: Simple state synchronization between server and client

The Rust implementation will leverage type safety and zero-cost abstractions to provide a safe and efficient flag system while maintaining the performance benefits of bitwise operations.
