# Entity System Architecture

## Overview

The entity system is the core of the MMORPG server, managing all game objects (players, monsters, NPCs, projectiles, items, etc.). The system must handle thousands of entities simultaneously with minimal performance overhead. This document describes the entity hierarchy, performance considerations, and optimization strategies for the Rust implementation.

## Entity Hierarchy

### TypeScript Implementation

The TypeScript version uses a class-based inheritance hierarchy:

```
Entity (base class)
├── Humanoid (equipment, inventory)
│   └── Player (quests, guild, party, crafting)
├── Creature (AI, targeting, combat)
│   ├── Monster (monster-specific AI)
│   ├── Pet (pet-specific behavior)
│   └── Summon (lifetime management)
```

**Key Classes:**

- **Entity**: Base class with common properties (position, rotation, stats, conditions, buffs/debuffs, AOI, team)
- **Humanoid**: Extends Entity, adds equipment system (helmet, chest, gloves, etc.), inventory, pet, mount
- **Creature**: Extends Entity, adds AI states, targeting, combat, actions, taming
- **Player**: Extends Humanoid, adds player-specific features (quests, guild, party, crafting, vendor, friends)
- **Pet**: Extends Creature, adds pet-specific behavior (following master, passive combat)
- **Summon**: Extends Creature, adds lifetime management

### C# Implementation

The C# version uses a more complex inheritance hierarchy:

```
Entity (base class)
├── CreatureEntity (stats, conditions, flags, damage)
│   ├── MonsterBase (monster states, skills, targeting)
│   │   └── Monster (monster-specific implementation)
│   ├── CharacterEntity (player-specific features)
│   └── MinionEntity (master, combat states)
│       └── MountEntity (mount-specific behavior)
├── ProjectileEntity (projectiles with various behaviors)
├── ReliableEntity (reliable entity base)
│   └── StashEntity (stash container)
└── AutoDestroyEntity (auto-destroy on leave AOI)
```

**Key Classes:**

- **Entity**: Base class with position, rotation, bounding box, collision, AOI callbacks
- **CreatureEntity**: Extends Entity, adds health/mana/stamina, conditions, flags, damage calculation, stats
- **MonsterBase**: Extends CreatureEntity, adds monster states (Idle, Patrol, InCombat, etc.), skills, targeting
- **CharacterEntity**: Extends CreatureEntity, adds player features (inventory, actionbar, guild, party, quests)
- **MinionEntity**: Extends MonsterBase, adds master relationship, combat states
- **MountEntity**: Extends MinionEntity, adds mount-specific behavior
- **ProjectileEntity**: Extends Entity, adds projectile behavior (arrow, explosive, piercing, etc.)

## Performance Concerns

### Object-Oriented Inheritance Overhead

#### 1. Virtual Method Table (VTable) Overhead

**Problem:**
- Each class with virtual methods has a vtable pointer (8 bytes on 64-bit systems)
- Method calls through vtable add indirect call overhead (~5-10 cycles)
- Deep inheritance hierarchies increase vtable size and indirection

**Impact:**
- With 10,000 entities, vtable pointers consume ~80 KB
- Virtual method calls add ~50-100 ns per call
- Deep hierarchies (Entity → CreatureEntity → MonsterBase → Monster) add multiple indirections

**Example:**
```csharp
// C# - Virtual method call
public virtual void Update()
{
    // Virtual call overhead
    base.Update(); // Indirect call through vtable
}

// Multiple levels of indirection:
// Entity.Update() → CreatureEntity.Update() → MonsterBase.Update() → Monster.Update()
```

#### 2. Memory Layout and Cache Misses

**Problem:**
- Inheritance creates object layouts with scattered data
- Base class data may be far from derived class data
- Cache lines (64 bytes) may not contain all frequently accessed fields
- Pointer chasing through inheritance hierarchy causes cache misses

**Impact:**
- Cache miss penalty: ~100-300 cycles (3-10 ns)
- With 10,000 entities, cache misses can cause significant performance degradation
- Object layout may not be optimal for hot path access patterns

**Example:**
```typescript
// TypeScript - Scattered data access
class Entity {
    id: string;              // Offset 0
    position: Vector3;       // Offset 8
    // ... base class fields
}

class Creature extends Entity {
    health: number;          // Offset 200+ (after base class)
    mana: number;            // Offset 208+
    // ... derived class fields
}

// Accessing health requires loading different cache lines
// Entity data (cache line 1) → Creature data (cache line 2)
```

#### 3. Memory Allocation Overhead

**Problem:**
- Each entity is a separate heap allocation
- Allocation overhead: ~16-32 bytes per object (metadata, alignment)
- Fragmentation: Objects scattered across memory
- GC pressure: Frequent allocations trigger garbage collection

**Impact:**
- With 10,000 entities, allocation overhead: ~160-320 KB
- GC pauses: 10-50 ms every few seconds (unacceptable for 60 FPS)
- Memory fragmentation: Reduces cache locality

**Example:**
```typescript
// TypeScript - Heap allocation
const entity = new Entity();           // Heap allocation #1
const creature = new Creature();       // Heap allocation #2
const player = new Player();           // Heap allocation #3

// Each allocation:
// - Metadata: 16-32 bytes
// - Alignment: 8-16 bytes padding
// - Fragmentation: Scattered in memory
```

#### 4. Frequent Instantiation Overhead

**Problem:**
- Entities created/destroyed frequently (projectiles, temporary effects)
- Constructor overhead: Initialization of all base classes
- Virtual method setup: Vtable initialization
- Memory allocation: Heap allocation for each entity

**Impact:**
- Projectile creation: ~100-500 ns per projectile
- With 100 projectiles/second, overhead: ~10-50 μs per second
- Constructor chaining: Entity() → CreatureEntity() → ProjectileEntity()

**Example:**
```csharp
// C# - Constructor chaining
public ProjectileEntity()
{
    // Calls Entity() constructor
    // Calls CreatureEntity() constructor
    // Initializes vtable
    // Allocates memory
    // Sets up event handlers
}
```

## Rust Implementation Strategies

### 1. Composition Over Inheritance (ECS Pattern)

**Strategy:**
- Use Entity Component System (ECS) architecture
- Entities are simple IDs (u32 or u64)
- Components are plain data structures (no inheritance)
- Systems process components independently

**Benefits:**
- No vtable overhead (components are plain structs)
- Better cache locality (components stored in contiguous arrays)
- Flexible composition (entities can have any combination of components)
- Easy to add/remove components at runtime

**Example:**
```rust
// Rust - ECS approach
#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
    z: f32,
}

#[derive(Component)]
struct Health {
    current: u32,
    max: u32,
}

#[derive(Component)]
struct Mana {
    current: u32,
    max: u32,
}

#[derive(Component)]
struct Stats {
    str: u32,
    dex: u32,
    int: u32,
    vig: u32,
    agi: u32,
    luc: u32,
}

// Entity is just an ID
type EntityId = u32;

// Components stored in separate arrays (cache-friendly)
// Position components: [Position; 10000]
// Health components: [Health; 10000]
// Mana components: [Mana; 10000]
```

### 2. SoA (Structure of Arrays) Layout

**Strategy:**
- Store components in separate arrays (SoA layout)
- Each component type has its own array
- Entities index into component arrays

**Benefits:**
- Cache-friendly: Processing all positions loads contiguous memory
- SIMD-friendly: Can use SIMD instructions on arrays
- Better memory access patterns: Sequential access instead of random access

**Example:**
```rust
// Rust - SoA layout
struct EntityStorage {
    positions: Vec<Position>,      // Cache line: [Pos0, Pos1, Pos2, Pos3, ...]
    healths: Vec<Health>,          // Cache line: [Health0, Health1, Health2, ...]
    manas: Vec<Mana>,              // Cache line: [Mana0, Mana1, Mana2, ...]
    stats: Vec<Stats>,             // Cache line: [Stats0, Stats1, Stats2, ...]
}

// Processing all positions:
for pos in &mut storage.positions {
    // Sequential memory access (cache-friendly)
    pos.x += velocity.x;
    pos.y += velocity.y;
    pos.z += velocity.z;
}
```

### 3. Zero-Allocation Entity Pooling

**Strategy:**
- Pre-allocate entity pools for each entity type
- Reuse entities instead of allocating/deallocating
- Use indices instead of pointers (avoid pointer chasing)

**Benefits:**
- No allocation overhead: Entities reused from pool
- No fragmentation: Contiguous memory layout
- Predictable performance: No GC pauses

**Example:**
```rust
// Rust - Entity pooling
struct EntityPool<T> {
    entities: Vec<Option<T>>,      // Pre-allocated entities
    free_indices: Vec<usize>,      // Free entity indices
}

impl<T> EntityPool<T> {
    fn allocate(&mut self) -> Option<usize> {
        // Reuse free entity (zero allocation)
        self.free_indices.pop()
    }
    
    fn deallocate(&mut self, index: usize) {
        // Mark entity as free (zero deallocation)
        self.entities[index] = None;
        self.free_indices.push(index);
    }
}
```

### 4. Flat Data Structures

**Strategy:**
- Use flat data structures instead of nested objects
- Avoid pointer indirection
- Store data inline when possible

**Benefits:**
- Better cache locality: All data in one structure
- No pointer chasing: Direct memory access
- Smaller memory footprint: No pointer overhead

**Example:**
```rust
// Rust - Flat data structure
struct Entity {
    // Base properties
    id: EntityId,
    position: Vector3,
    rotation: Quaternion,
    
    // Stats (inline, no pointer)
    health: u32,
    max_health: u32,
    mana: u32,
    max_mana: u32,
    stamina: u32,
    max_stamina: u32,
    
    // Flags (bitwise, compact)
    flags: EntityFlags,
    buff_flags: BuffFlags,
    
    // Conditions (small array, inline)
    conditions: [Option<Condition>; 10],
    
    // Team (inline enum, no pointer)
    team: TeamKind,
    
    // Entity type (discriminated union)
    entity_type: EntityType,
}

// EntityType discriminates between entity variants
enum EntityType {
    Player(PlayerData),
    Monster(MonsterData),
    Pet(PetData),
    Summon(SummonData),
    Projectile(ProjectileData),
}
```

### 5. Tagged Union for Entity Types

**Strategy:**
- Use Rust enums (tagged unions) for entity type discrimination
- Store type-specific data inline
- Use pattern matching for type-specific logic

**Benefits:**
- Type safety: Compile-time type checking
- No vtable: Enum discriminant is just a byte
- Inline data: Type-specific data stored in same structure
- Pattern matching: Efficient type checking

**Example:**
```rust
// Rust - Tagged union for entity types
enum EntityData {
    Player {
        account_id: AccountId,
        inventory: InventoryId,
        quests: QuestList,
        guild: Option<GuildId>,
        party: Option<PartyId>,
    },
    Monster {
        monster_type: MonsterType,
        ai_state: AIState,
        target: Option<EntityId>,
        skills: Vec<MonsterSkill>,
    },
    Pet {
        owner: EntityId,
        pet_type: PetType,
        lifetime: Option<Duration>,
    },
    Summon {
        owner: EntityId,
        lifetime: Duration,
        summon_type: SummonType,
    },
    Projectile {
        owner: EntityId,
        damage: u32,
        damage_type: DamageType,
        velocity: Vector3,
        lifetime: Duration,
    },
}

struct Entity {
    id: EntityId,
    position: Vector3,
    rotation: Quaternion,
    health: u32,
    max_health: u32,
    // ... common fields
    data: EntityData,  // Type-specific data (inline)
}
```

### 6. Component-Based Equipment System

**Strategy:**
- Use components for equipment instead of inheritance
- Equipment slots as separate components
- Equipment effects as separate components

**Benefits:**
- Flexible: Entities can have any equipment combination
- Cache-friendly: Equipment stored in separate arrays
- Easy to query: "All entities with helmet equipped"

**Example:**
```rust
// Rust - Component-based equipment
#[derive(Component)]
struct Equipment {
    helmet: Option<ItemId>,
    chest: Option<ItemId>,
    gloves: Option<ItemId>,
    pants: Option<ItemId>,
    boots: Option<ItemId>,
    mainhand: Option<ItemId>,
    offhand: Option<ItemId>,
    // ... other slots
}

#[derive(Component)]
struct EquipmentEffects {
    bonus_str: i32,
    bonus_dex: i32,
    bonus_int: i32,
    bonus_vig: i32,
    bonus_agi: i32,
    bonus_luc: i32,
    physical_resistance: i32,
    fire_resistance: i32,
    // ... other effects
}

// Entity can have equipment component
// Equipment effects calculated and stored as separate component
```

### 7. System-Based Processing

**Strategy:**
- Separate data (components) from logic (systems)
- Systems process components independently
- Parallel processing: Systems can run in parallel

**Benefits:**
- Cache-friendly: Systems process components sequentially
- Parallelizable: Independent systems can run in parallel
- Testable: Systems are pure functions

**Example:**
```rust
// Rust - System-based processing
fn update_positions(
    positions: &mut [Position],
    velocities: &[Velocity],
    delta_time: f32,
) {
    // Process all positions (cache-friendly)
    for (pos, vel) in positions.iter_mut().zip(velocities) {
        pos.x += vel.x * delta_time;
        pos.y += vel.y * delta_time;
        pos.z += vel.z * delta_time;
    }
}

fn update_health(
    healths: &mut [Health],
    conditions: &[Conditions],
    delta_time: f32,
) {
    // Process all health (cache-friendly)
    for (health, conditions) in healths.iter_mut().zip(conditions) {
        for condition in conditions {
            match condition.condition_type {
                ConditionType::Burning => {
                    health.current -= condition.value;
                }
                ConditionType::Healing => {
                    health.current += condition.value;
                }
                // ... other conditions
            }
        }
    }
}

// Systems can run in parallel
rayon::join(
    || update_positions(&mut positions, &velocities, delta_time),
    || update_health(&mut healths, &conditions, delta_time),
);
```

## Recommended Rust Architecture

### Entity Component System (ECS)

**Structure:**
```
Entity (u32 ID)
├── Components (stored in separate arrays)
│   ├── Position
│   ├── Rotation
│   ├── Health
│   ├── Mana
│   ├── Stats
│   ├── Flags
│   ├── Conditions
│   ├── Buffs
│   ├── Team
│   ├── Equipment (optional)
│   ├── Inventory (optional)
│   ├── PlayerData (optional)
│   ├── MonsterData (optional)
│   ├── PetData (optional)
│   └── ProjectileData (optional)
└── Systems (process components)
    ├── MovementSystem
    ├── CombatSystem
    ├── AISystem
    ├── ConditionSystem
    ├── BuffSystem
    └── EquipmentSystem
```

### Implementation Example

```rust
// Entity ID
type EntityId = u32;

// Components
#[derive(Component, Clone)]
struct Position {
    x: f32,
    y: f32,
    z: f32,
}

#[derive(Component, Clone)]
struct Rotation {
    yaw: f32,
    pitch: f32,
    roll: f32,
}

#[derive(Component)]
struct Health {
    current: u32,
    max: u32,
    byte: u8,  // Quantized health for network
}

#[derive(Component)]
struct Mana {
    current: u32,
    max: u32,
    byte: u8,  // Quantized mana for network
}

#[derive(Component)]
struct Stats {
    str: u32,
    dex: u32,
    int: u32,
    vig: u32,
    agi: u32,
    luc: u32,
}

#[derive(Component)]
struct EntityFlags {
    flags: u32,  // Bitwise flags
}

#[derive(Component)]
struct Conditions {
    conditions: [Option<Condition>; 10],
    count: usize,
}

#[derive(Component)]
struct Buffs {
    buffs: [Option<Buff>; 20],
    count: usize,
}

#[derive(Component)]
struct Team {
    team: TeamKind,
    owner: Option<EntityId>,
}

// Type-specific components
#[derive(Component)]
struct PlayerData {
    account_id: AccountId,
    character_id: CharacterId,
    inventory_id: InventoryId,
    guild_id: Option<GuildId>,
    party_id: Option<PartyId>,
    quests: QuestList,
}

#[derive(Component)]
struct MonsterData {
    monster_type: MonsterType,
    ai_state: AIState,
    target: Option<EntityId>,
    skills: Vec<MonsterSkill>,
}

#[derive(Component)]
struct PetData {
    owner: EntityId,
    pet_type: PetType,
}

#[derive(Component)]
struct ProjectileData {
    owner: EntityId,
    damage: u32,
    damage_type: DamageType,
    velocity: Vector3,
    lifetime: Duration,
}

// ECS World
struct World {
    entities: EntityPool,
    positions: ComponentStorage<Position>,
    rotations: ComponentStorage<Rotation>,
    healths: ComponentStorage<Health>,
    manas: ComponentStorage<Mana>,
    stats: ComponentStorage<Stats>,
    flags: ComponentStorage<EntityFlags>,
    conditions: ComponentStorage<Conditions>,
    buffs: ComponentStorage<Buffs>,
    teams: ComponentStorage<Team>,
    player_data: ComponentStorage<PlayerData>,
    monster_data: ComponentStorage<MonsterData>,
    pet_data: ComponentStorage<PetData>,
    projectile_data: ComponentStorage<ProjectileData>,
}

// Systems
fn movement_system(world: &mut World, delta_time: f32) {
    // Process all entities with position and velocity
    for (entity, pos, vel) in world.query::<(&mut Position, &Velocity)>() {
        pos.x += vel.x * delta_time;
        pos.y += vel.y * delta_time;
        pos.z += vel.z * delta_time;
    }
}

fn health_system(world: &mut World, delta_time: f32) {
    // Process all entities with health and conditions
    for (entity, health, conditions) in world.query::<(&mut Health, &Conditions)>() {
        for condition in &conditions.conditions {
            if let Some(cond) = condition {
                match cond.condition_type {
                    ConditionType::Burning => {
                        health.current = health.current.saturating_sub(cond.value);
                    }
                    ConditionType::Healing => {
                        health.current = (health.current + cond.value).min(health.max);
                    }
                    // ... other conditions
                }
            }
        }
        // Update quantized health byte
        health.byte = ((health.current * 255) / health.max.max(1)) as u8;
    }
}
```

## Performance Comparison

### Inheritance vs ECS

**Inheritance (TypeScript/C#):**
- Virtual method calls: ~5-10 cycles per call
- Cache misses: ~100-300 cycles per miss
- Memory allocation: ~16-32 bytes overhead per entity
- GC pressure: 10-50 ms pauses every few seconds

**ECS (Rust):**
- Direct function calls: ~1-2 cycles per call
- Cache hits: Sequential memory access
- Zero allocation: Pre-allocated pools
- No GC: Predictable performance

### Estimated Performance Gains

**With 10,000 entities:**
- Virtual method overhead: ~50-100 ns per call → **Eliminated**
- Cache misses: ~100-300 cycles per miss → **Reduced by 80-90%**
- Memory allocation: ~160-320 KB overhead → **Eliminated**
- GC pauses: 10-50 ms every few seconds → **Eliminated**

**Overall performance improvement:**
- Entity update: **2-5x faster**
- Memory usage: **20-30% reduction**
- Cache efficiency: **80-90% improvement**
- Predictable performance: **No GC pauses**

## Migration Strategy

### Phase 1: Core ECS Infrastructure
1. Implement basic ECS framework (entities, components, systems)
2. Implement component storage (SoA layout)
3. Implement entity pooling
4. Migrate base entity properties (position, rotation, health, mana)

### Phase 2: Entity Types
1. Migrate player entities (PlayerData component)
2. Migrate monster entities (MonsterData component)
3. Migrate pet entities (PetData component)
4. Migrate projectile entities (ProjectileData component)

### Phase 3: Systems
1. Migrate movement system
2. Migrate combat system
3. Migrate AI system
4. Migrate condition system
5. Migrate buff system

### Phase 4: Advanced Features
1. Migrate equipment system (Equipment component)
2. Migrate inventory system (Inventory component)
3. Migrate quest system (QuestList component)
4. Migrate guild/party system (GuildId, PartyId components)

## Conclusion

The Rust implementation should use an Entity Component System (ECS) architecture instead of inheritance-based object-oriented design. This approach provides:

- **Better Performance**: No vtable overhead, better cache locality, zero allocation
- **Better Scalability**: Components stored in separate arrays, parallel processing
- **Better Flexibility**: Entities can have any combination of components
- **Better Maintainability**: Systems are pure functions, easy to test and modify

The ECS pattern is well-suited for high-performance game servers and aligns with Rust's zero-cost abstractions philosophy.

