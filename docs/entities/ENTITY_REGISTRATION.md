# Entity Registration

## Overview

The Entity Registration system manages entity class registration and instantiation. Entities are registered by name and can be created dynamically using factory methods.

## Entity Registry

### Entities Map

```typescript
public static Entities: Map<string, { new (): Entity }>
```

**Storage:**
- Maps entity name to entity class
- Case-insensitive lookup
- Used for entity creation

**Key Format:**
- Entity name (e.g., "Goblin", "Player")
- Lowercase version also stored
- Multiple keys per class

## Registration Methods

### Add Entity Base

```typescript
static addEntityBase(refs: string[] | string, clas: any): void
```

**Process:**
1. Check if refs is array or string
2. If array: iterate and register each
3. If string: register single name
4. Store class in Entities map
5. Also store lowercase version

**Array Support:**
- Multiple names per class
- Aliases support
- Flexible naming

**Example:**
```typescript
Entity.addEntityBase(["Goblin", "GoblinWarrior"], GoblinClass)
// Both "Goblin" and "GoblinWarrior" work
```

### Get Entity Base

```typescript
static getEntityBase(ref: string): { new (): Entity } | null
```

**Process:**
1. Check if Entities map has ref
2. Return class or null
3. Case-insensitive lookup

**Usage:**
- Entity creation
- Validation
- Lookup

## Entity Creation

### Create Method

```typescript
static create(entityName: string): Entity | null
```

**Process:**
1. Check if entity name exists in Entities map
2. Get entity class
3. Create new instance
4. Handle multiple visual support
5. Call init() method
6. Return entity or null

**Multiple Visual Support:**
- Checks multipleVisual array
- Randomly selects visual if available
- Sets customVisual property

**Initialization:**
- Calls entity.init()
- Allows custom setup
- Prepares entity for use

### Creation Example

```typescript
const entity = Entity.create("Goblin")
if (entity) {
    // Entity created successfully
    entity.transform = position
    map.joinMap(entity)
}
```

## Multiple Visual Support

### Multiple Visual Array

**Property:**
- **multipleVisual**: Array<string>
- Visual names/variants
- Random selection

**Selection:**
- Random.ArrRandom() used
- Equal probability
- Sets customVisual

**Usage:**
- Visual variety
- Random appearance
- Customization

## Entity Naming

### Name Registration

**Single Name:**
- One name per class
- Direct mapping
- Simple lookup

**Multiple Names:**
- Array of names
- All map to same class
- Aliases support

**Case Insensitivity:**
- Lowercase stored
- Case-insensitive lookup
- User-friendly

## Registration Examples

### Basic Registration

```typescript
Entity.addEntityBase("Goblin", Goblin)
Entity.addEntityBase("Orc", Orc)
```

### Multiple Names

```typescript
Entity.addEntityBase(["Skeleton", "SkeletonWarrior"], Skeleton)
```

### Case Handling

```typescript
Entity.addEntityBase("Goblin", Goblin)
// "Goblin" and "goblin" both work
```

## Entity Lookup

### Lookup Process

1. **Input**: Entity name string
2. **Check**: Entities map lookup
3. **Return**: Entity class or null
4. **Create**: Instantiate if found

### Lookup Failure

**Null Return:**
- Entity not found
- Invalid name
- Not registered

**Error Handling:**
- Null checks required
- Graceful failure
- Safe operations

## Initialization

### Init Method

**Entity Init:**
- Called after creation
- Custom setup
- Property initialization

**Override:**
- Subclasses override init()
- Custom initialization
- Setup logic

## Implementation Notes

### Performance

**Optimizations:**
- Map-based lookup (O(1))
- Efficient registration
- Minimal overhead

### Thread Safety

**Registration:**
- Static map
- Registration at startup
- No concurrent modifications

### Extensibility

**Adding Entities:**
1. Create entity class
2. Register with addEntityBase()
3. Use Entity.create()
4. Ready for use

## Related Documentation

- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [CREATURES.md](./CREATURES.md) - Creature system
- [OVERVIEW.md](./OVERVIEW.md) - Entity overview

