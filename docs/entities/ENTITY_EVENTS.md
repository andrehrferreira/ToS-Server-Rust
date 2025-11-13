# Entity Events

## Overview

The Entity Events system uses RxJS Subjects to provide reactive event handling for entity lifecycle, condition changes, and buff/debuff changes. Events allow other systems to subscribe and react to entity state changes.

## Event Subjects

### OnDie Subject

```typescript
public OnDie: Subject<Entity> = new Subject<Entity>()
```

**Purpose:**
- Triggered when entity dies
- Carries entity instance
- Used for cleanup and reactions

**Usage:**
- Respawn system subscription
- Loot system trigger
- Quest progress
- Achievement tracking

**Example:**
```typescript
entity.OnDie.subscribe({
    next: (deadEntity) => {
        // Handle death
        respawn.entityDie()
    }
})
```

### OnDestroy Subject

```typescript
public OnDetroy: Subject<Entity> = new Subject<Entity>()
```

**Purpose:**
- Triggered when entity is destroyed
- Carries entity instance
- Used for cleanup

**Usage:**
- Respawn system cleanup
- Memory cleanup
- Reference removal
- Final cleanup operations

**Example:**
```typescript
entity.OnDetroy.subscribe({
    next: (destroyedEntity) => {
        // Handle destruction
        map.removeEntity(destroyedEntity)
    }
})
```

### OnConditionChanged Subject

```typescript
public OnConditionChanged: Subject<Condition> = new Subject<Condition>()
```

**Purpose:**
- Triggered when condition changes
- Carries Condition instance
- Used for condition reactions

**Usage:**
- UI updates
- Effect applications
- Status synchronization
- Visual feedback

**Example:**
```typescript
entity.OnConditionChanged.subscribe({
    next: (condition) => {
        // Handle condition change
        updateUI(condition)
    }
})
```

### OnBuffDebuffChanged Subject

```typescript
public OnBuffDebuffChanged: Subject<BuffDebuff> = new Subject<BuffDebuff>()
```

**Purpose:**
- Triggered when buff/debuff changes
- Carries BuffDebuff instance
- Used for effect reactions

**Usage:**
- Stat recalculation
- Visual effects
- UI updates
- Effect synchronization

**Example:**
```typescript
entity.OnBuffDebuffChanged.subscribe({
    next: (buffDebuff) => {
        // Handle buff/debuff change
        recalculateStats()
    }
})
```

## Event Subscription

### Subscription Pattern

**RxJS Pattern:**
- Uses RxJS Subject
- Observable pattern
- Multiple subscribers

**Subscription:**
```typescript
const subscription = entity.OnDie.subscribe({
    next: (entity) => {
        // Handle event
    }
})
```

### Unsubscription

**Cleanup:**
- Store subscription reference
- Unsubscribe when done
- Prevents memory leaks

**Example:**
```typescript
const subscription = entity.OnDie.subscribe({...})
// Later...
subscription.unsubscribe()
```

## Event Triggers

### Death Trigger

**Trigger Location:**
- die() method
- Life <= 0
- Kill command

**Process:**
1. Entity dies
2. OnDie.next(entity) called
3. Subscribers notified
4. Reactions execute

### Destroy Trigger

**Trigger Location:**
- destroy() method
- Entity removal
- Map cleanup

**Process:**
1. Entity destroyed
2. OnDetroy.next(entity) called
3. Subscribers notified
4. Cleanup executes

### Condition Change Trigger

**Trigger Location:**
- applyCondition()
- removeCondition()
- Condition updates

**Process:**
1. Condition changes
2. OnConditionChanged.next(condition) called
3. Subscribers notified
4. Reactions execute

### Buff/Debuff Change Trigger

**Trigger Location:**
- applyBuffDebuff()
- removeBuffDebuff()
- Buff/debuff updates

**Process:**
1. Buff/debuff changes
2. OnBuffDebuffChanged.next(buffDebuff) called
3. Subscribers notified
4. Reactions execute

## Integration Examples

### Respawn Integration

```typescript
newEntity.OnDie.subscribe({
    next: (entity) => entity.respawn.entityDie()
})

newEntity.OnDetroy.subscribe({
    next: (entity) => entity.respawn.entityDie()
})
```

**Behavior:**
- Death triggers respawn timer
- Destruction triggers respawn timer
- Automatic respawn management

### Boss Summon Integration

```typescript
newEntity.OnDie.subscribe({
    next: (entity) => boss.summonDie(entity)
})
```

**Behavior:**
- Summon death tracked
- Removed from summons array
- Cleanup executed

### Target Integration

```typescript
this.targetOnDie = targetActor.OnDie.subscribe({
    next: (entity) => {
        this.targetActor = null
    }
})
```

**Behavior:**
- Target death clears target
- Unsubscribes automatically
- Cleanup on death

## Event Flow

### Death Event Flow

1. Entity takes damage
2. Life reaches 0
3. die() method called
4. OnDie.next(entity) triggered
5. Subscribers notified
6. Respawn system reacts
7. Loot system reacts
8. Quest system reacts

### Destroy Event Flow

1. Entity removed
2. destroy() method called
3. OnDetroy.next(entity) triggered
4. Subscribers notified
5. Map cleanup reacts
6. Memory cleanup reacts
7. Reference removal

## Implementation Notes

### Performance

**Optimizations:**
- RxJS efficient
- Minimal overhead
- Event-driven architecture

### Memory Management

**Subscription Cleanup:**
- Unsubscribe when done
- Prevent memory leaks
- Proper cleanup

### Thread Safety

**Event Threading:**
- Synchronous execution
- No race conditions
- Safe operations

## Related Documentation

- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [CONDITIONS.md](../core/CONDITIONS.md) - Condition system
- [STATUS.md](../core/STATUS.md) - Status system

