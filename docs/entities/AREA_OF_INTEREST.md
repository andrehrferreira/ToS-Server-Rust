# Area of Interest (AOI)

## Overview

The Area of Interest (AOI) system manages entity visibility and synchronization within a configurable radius. Entities only receive updates from and send updates to entities within their AOI, optimizing network traffic and performance.

## AOI Properties

### Area of Interest Array

```typescript
public areaOfInterece: Array<Entity>
```

**Storage:**
- Array of entities currently in AOI
- Bidirectional relationships
- Updated dynamically

## AOI Radius

### Interest Radius

**Value:**
- **interestRadius**: 8000 units
- Configurable per entity
- Default: 8000

**Usage:**
- Distance calculation
- Visibility determination
- Update filtering

## Update Area of Interest

### Update Method

```typescript
updateAreaOfInterest(): void
```

**Process:**
1. Check if map exists and entity not removed
2. Create current area Set (mapIndexes)
3. Iterate all entities in map
4. Skip self
5. Calculate distance
6. Check if in interest area (distance <= radius)
7. Check if already in area
8. Add/remove entities as needed
9. Maintain bidirectional relationships

**Conditions:**
- Entity not dead
- Distance <= interestRadius
- Not already in area

### Add to AOI

```typescript
addToAreaOfInterest(entity: Entity): void
```

**Process:**
1. Add entity to areaOfInterece array
2. Send CreateEntity packet
3. Bidirectional: entity also adds this

**Packet Sending:**
- CreateEntity packet sent
- Entity becomes visible
- Initial synchronization

### Remove from AOI

```typescript
removeFromAreaOfInterest(entity: Entity): void
```

**Process:**
1. Send RemoveEntity packet
2. Filter entity from areaOfInterece array
3. Bidirectional: entity also removes this

**Packet Sending:**
- RemoveEntity packet sent
- Entity becomes invisible
- Cleanup synchronization

## Bidirectional Relationships

### Mutual AOI

**Behavior:**
- When A adds B to AOI, B also adds A
- When A removes B from AOI, B also removes A
- Maintains consistency
- Prevents one-way visibility

**Implementation:**
```typescript
if (isInInterestArea && !alreadyInArea) {
    this.addToAreaOfInterest(entity)
    entity.addToAreaOfInterest(this)
}
```

## Distance Calculation

### Distance Method

```typescript
distanceTo(other: Vector3): number
```

**Calculation:**
- Euclidean distance
- 3D space calculation
- Used for AOI checks

**Formula:**
```
distance = sqrt((x1-x2)² + (y1-y2)² + (z1-z2)²)
```

## Entity Filtering

### Get Enemies in Radius

```typescript
getEnemiesInRadius(position: Vector3, radius: number): Array<Entity>
```

**Process:**
1. Iterate areaOfInterece array
2. Check if enemy (team.IsEnemyOf())
3. Check if within radius
4. Add to enemies array
5. Return filtered list

**Usage:**
- Area effect targeting
- Combat calculations
- Skill targeting

## Update Frequency

### Update Calls

**Update Loop:**
- Called every update() cycle (1 second)
- Called every tick() cycle (300ms)
- Keeps AOI current

**Performance:**
- Efficient distance checks
- Minimal overhead
- Optimized iterations

## Dead Entity Handling

### Dead Entity Filter

**Behavior:**
- Dead entities not added to AOI
- Dead entities removed from AOI
- Prevents ghost visibility

**Check:**
```typescript
if(!entity.isDead) {
    // Add/remove logic
}
```

## Network Optimization

### Packet Reduction

**Benefits:**
- Only send to entities in AOI
- Reduces network traffic
- Improves performance

**Packets Affected:**
- CreateEntity
- UpdateEntity
- RemoveEntity
- ActionEntity
- EventEntity
- All entity packets

### Broadcast Optimization

**Broadcast Method:**
```typescript
broadcast(packet: Packet, data: any): void
```

**Process:**
1. Create Set with areaOfInterece and self
2. Send packet to all entities in Set
3. Only sends to AOI entities

## Implementation Notes

### Performance

**Optimizations:**
- Set-based lookups (O(1))
- Efficient distance calculations
- Minimal array operations
- Bidirectional maintenance

### Thread Safety

**Update Safety:**
- Called from update loop
- No concurrent modifications
- Safe array operations

### Edge Cases

**Entity Removal:**
- Handles removed entities
- Cleans up references
- Prevents memory leaks

**Map Changes:**
- AOI cleared on map change
- Rebuilt on new map
- Proper cleanup

## Related Documentation

- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [MOVEMENT.md](../core/MOVEMENT.md) - Movement system
- [NETWORK_PACKETS.md](../packets/NETWORK_PACKETS.md) - Network packets

