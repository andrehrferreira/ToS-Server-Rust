# Entity Update Loop

## Overview

The Entity Update Loop manages periodic updates for entities including area of interest updates, condition/buff/debuff updates, regeneration, and state synchronization. Entities are updated at different intervals for optimal performance.

## Update Intervals

### Update Loop

**Interval:**
- **1 second** (1000ms)
- Main update cycle
- State synchronization

**Set Interval:**
```typescript
setInterval(this.update.bind(this), 1000)
```

### Regen Loop

**Interval:**
- **3 seconds** (3000ms)
- Regeneration cycle
- Resource recovery

**Set Interval:**
```typescript
setInterval(this.regenStats.bind(this), 3000)
```

## Update Method

### Update Process

```typescript
update(): void
```

**Process:**
1. Check if map exists and not removed
2. Update area of interest
3. If not dead:
   - Update conditions
   - Update buffs/debuffs
4. If dead:
   - Check dissolve conditions
   - Trigger destroy if ready

**Conditions:**
- Map must exist
- Entity not removed
- Proper state checks

### Area of Interest Update

**Update:**
- Called every update cycle
- Maintains visibility
- Synchronizes entities

**Process:**
- updateAreaOfInterest() called
- Entities added/removed
- Packets sent

## Condition Updates

### Condition Loop

**Process:**
1. Iterate conditions array
2. Call condition.update(entity)
3. Check lifetime
4. Remove if lifetime <= 0
5. Trigger OnConditionChanged

**Update Frequency:**
- Every update cycle (1 second)
- Per-condition updates
- Lifetime management

### Condition Removal

**Removal Process:**
1. Lifetime reaches 0
2. condition.remove(entity) called
3. Remove from conditions array
4. OnConditionChanged triggered
5. Cleanup executed

## Buff/Debuff Updates

### Buff/Debuff Loop

**Process:**
1. Iterate buffsDebuffs array
2. Call buffDebuff.update(entity)
3. Check lifetime
4. Remove if lifetime <= 0
5. Trigger OnBuffDebuffChanged

**Update Frequency:**
- Every update cycle (1 second)
- Per-buff/debuff updates
- Lifetime management

### Buff/Debuff Removal

**Removal Process:**
1. Lifetime reaches 0
2. buffDebuff.remove(entity) called
3. Remove from buffsDebuffs array
4. OnBuffDebuffChanged triggered
5. Cleanup executed

## Regeneration Loop

### Regen Stats Method

```typescript
regenStats(): void
```

**Process:**
1. Check if map exists and not removed
2. Check if not creature
3. Check if not dead
4. Calculate regen values
5. Apply regen to stats
6. Clamp to max values

**Regen Calculation:**
- Life: 1 + (lifeRegeneration / 10)
- Mana: 3 + (manaRegenegation / 10)
- Stamina: 10 + (staminaRegenegation / 10)

### Equipment Modifiers

**Medium Equipment:**
- Mana regen halved
- Affects mana recovery

**Heavy Equipment:**
- Mana regen halved
- Stamina regen not doubled
- Movement restrictions

**Sprint:**
- Stamina regen: -10
- Consumes stamina
- Movement speed bonus

### Regen Clamping

**Clamping:**
- Life: [0, maxLife]
- Mana: [0, maxMana]
- Stamina: [0, maxStamina]

**Math Operations:**
- Math.max() for minimum
- Math.min() for maximum
- Prevents overflow

## Dead Entity Updates

### Dead Entity Handling

**Process:**
1. Check if dead
2. Check skinner tick (looting)
3. Check loot count
4. Check die timeout
5. Trigger dissolve if ready

**Dissolve Conditions:**
- Skinner tick === 0
- Loot count <= 0
- Die timeout passed
- OR die timeout passed

### Dissolve Process

**Process:**
1. Send DissolveEntity packet to AOI
2. Trigger OnDetroy event
3. Remove from map
4. Call destroy()

**Packet Broadcasting:**
- Sent to areaOfInterece
- Visual dissolve effect
- Entity removal

## Tick Method

### Tick Process

```typescript
tick(tickNumber: number): void
```

**Process:**
1. Check if not removed
2. Update isDead flag
3. Update area of interest
4. Add Dead state if dead
5. Send UpdateEntity packet to AOI
6. Check lastUpdate timeout
7. Disconnect if timeout

**Update Frequency:**
- Called from map tick
- Every deltaTime (300ms)
- Per-entity updates

### Update Entity Packet

**Sending:**
- Sent to areaOfInterece
- State synchronization
- Position updates

**Content:**
- Entity state
- Position
- Rotation
- Stats

### Timeout Handling

**Last Update:**
- Tracks last interaction
- Timeout: 60 seconds
- Disconnects if exceeded

**Disconnect Process:**
1. Set removed flag
2. Remove from online players
3. Log disconnect
4. Close socket
5. Call destroy()

## Update Intensity

### Update Intensity Property

**Property:**
- **updateIntensity**: number
- Update frequency multiplier
- Configurable per entity type

**Usage:**
- Players: 1 (full updates)
- Creatures: Variable
- Optimization

## Implementation Notes

### Performance

**Optimizations:**
- Efficient condition loops
- Minimal array operations
- Optimized regen calculations
- Batch packet sending

### Thread Safety

**Update Safety:**
- Single-threaded execution
- No concurrent modifications
- Safe array operations

### Memory Management

**Cleanup:**
- Proper condition removal
- Buff/debuff cleanup
- Event unsubscription
- Reference cleanup

## Related Documentation

- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [CONDITIONS.md](../core/CONDITIONS.md) - Condition system
- [STATUS.md](../core/STATUS.md) - Status system
- [AREA_OF_INTEREST.md](./AREA_OF_INTEREST.md) - AOI system

