# Migration from Legacy to Rust

## Overview

This document explains how the legacy TypeScript and C# implementations inform the Rust implementation design. The legacy code serves as **reference material** to understand game systems, not as code to be directly ported.

## Philosophy

### Not a Port - A Redesign

The Rust implementation is **not a port** of TypeScript or C#. Instead:

1. **Learn** from legacy implementations (what works, what doesn't)
2. **Document** system requirements and game mechanics
3. **Design** optimal Rust architecture
4. **Implement** with performance and safety in mind

### What We Keep

**Game Design & Mechanics**:
- Combat formulas and calculations
- Skill systems and progression
- Quest mechanics and workflows
- Crafting recipes and systems
- Entity behaviors and AI

**Protocol Design**:
- Packet structures (optimized for Rust)
- Network flow and patterns
- Security requirements
- Synchronization approach

### What We Change

**Architecture**:
- TypeScript OOP → Rust ECS (Entity Component System)
- Class hierarchies → Component composition
- Inheritance → Trait-based behavior
- Garbage collection → Zero-allocation design

**Performance**:
- Dynamic arrays → Fixed-size pools
- Hash maps → Lock-free structures
- Async/await → Tokio async runtime
- Single-threaded → Multi-threaded work-stealing

**Memory Management**:
- GC pauses → Predictable allocation
- Reference counting → Ownership system
- Deep copies → Zero-copy operations
- Boxing → Stack allocation

## System Migration Examples

### Example 1: Entity System

**TypeScript (Legacy)**:
```typescript
class Entity {
    public id: string
    public transform: Transform
    public life: number
    // ... many properties
    
    update() {
        // Update logic
    }
}

class Player extends Entity {
    // Player-specific logic
}
```

**Rust (New Design)**:
```rust
// ECS approach
struct EntityId(u64);

// Components
struct Transform { position: Vec3, rotation: Quat }
struct Health { current: i32, max: i32 }
struct Player { account_id: u64, name: String }

// Systems process components
fn movement_system(query: Query<(&EntityId, &mut Transform)>) {
    // Process all entities with Transform
}
```

**Why the Change**:
- Better cache locality (SoA vs AoS)
- Easier parallelization
- More flexible composition
- Predictable performance

### Example 2: Network Packets

**TypeScript (Legacy)**:
```typescript
class ByteBuffer {
    buffer: Buffer
    position: number
    
    writeInt32(value: number) {
        this.buffer.writeInt32LE(value, this.position)
        this.position += 4
    }
}
```

**Rust (New Design)**:
```rust
// Zero-allocation, compile-time safety
struct ByteBuffer<'a> {
    data: &'a mut [u8],
    position: usize,
}

impl<'a> ByteBuffer<'a> {
    fn write_i32(&mut self, value: i32) -> Result<(), BufferError> {
        // Compile-time bounds checking, no allocations
        let bytes = value.to_le_bytes();
        self.data.get_mut(self.position..self.position + 4)
            .ok_or(BufferError::Overflow)?
            .copy_from_slice(&bytes);
        self.position += 4;
        Ok(())
    }
}
```

**Why the Change**:
- Zero runtime overhead
- Compile-time safety
- No allocations
- Memory safety guaranteed

### Example 3: Area of Interest

**TypeScript (Legacy)**:
```typescript
class Entity {
    areaOfInterest: Array<Entity> = []
    
    updateAreaOfInterest() {
        const radius = 8000
        this.map.entitiesIndexById.forEach(entity => {
            const distance = this.transform.position.distanceTo(entity.transform.position)
            if (distance <= radius) {
                this.addToAreaOfInterest(entity)
            }
        })
    }
}
```

**Rust (New Design)**:
```rust
// Spatial grid with lock-free queries
struct SpatialGrid {
    cells: Vec<RwLock<Vec<EntityId>>>,
    cell_size: f32,
}

impl SpatialGrid {
    // Lock-free read, parallel processing
    fn query_radius(&self, pos: Vec3, radius: f32) -> Vec<EntityId> {
        // Calculate cells in radius
        // Lock-free read from relevant cells
        // Parallel distance checks
    }
}
```

**Why the Change**:
- O(1) spatial queries vs O(n)
- Parallel processing
- Lock-free reads
- Scalable to 10k players

## Translation Guidelines

### Data Structures

| TypeScript/C# | Rust Equivalent | Notes |
|---------------|-----------------|-------|
| `Array<T>` | `Vec<T>` | Prefer pre-allocated with capacity |
| `Map<K,V>` | `HashMap<K,V>` or `DashMap<K,V>` | DashMap for concurrent access |
| `Set<T>` | `HashSet<T>` | Or use BitSet for dense IDs |
| `class` | `struct` + `impl` | Prefer composition over inheritance |
| `interface` | `trait` | Use for shared behavior |
| `enum` | `enum` | Rust enums are more powerful |

### Patterns

| TypeScript/C# Pattern | Rust Pattern | Rationale |
|-----------------------|--------------|-----------|
| Inheritance | Composition + Traits | More flexible, better performance |
| Observer pattern | Event channels | Lock-free, async-friendly |
| Singleton | `static` or `lazy_static!` | Thread-safe by default |
| Callbacks | Closures or `fn` pointers | Zero-cost abstractions |
| Async/await | Tokio async | Efficient, structured concurrency |

### Performance Patterns

**Avoid in Rust**:
- ❌ Cloning large structures
- ❌ Dynamic dispatch when avoidable
- ❌ Mutex for hot paths
- ❌ Allocations in game loop
- ❌ String concatenation in loops

**Prefer in Rust**:
- ✅ Pre-allocated pools
- ✅ Lock-free structures
- ✅ Static dispatch (generics)
- ✅ References and borrowing
- ✅ Copy types for small data

## Documentation Usage

### Reading Legacy Code Examples

When you see TypeScript/C# code in documentation:

1. **Understand the concept**: What is the system trying to achieve?
2. **Identify the data flow**: What data goes in, what comes out?
3. **Note the game logic**: Formulas, rules, mechanics
4. **Ignore implementation details**: Don't copy the structure
5. **Design Rust solution**: Think about Rust idioms and performance

### Writing New Documentation

When documenting a system:

```markdown
## System Name

### Purpose
[What problem does this solve?]

### Design (Rust Implementation)
[How it works in Rust]

### Game Logic
[Formulas, rules, independent of language]

### Legacy Reference
[TypeScript/C# example for context - NOT to copy]
\`\`\`typescript
// Legacy code for understanding only
\`\`\`

### Rust Implementation Status
- **Status**: 🔲 Planned | ⚠️ In Progress | ✅ Complete
- **Priority**: High | Medium | Low
```

## Common Pitfalls

### ❌ Don't Do

1. **Direct translation**: Don't translate TypeScript line-by-line to Rust
2. **Keep OOP patterns**: Don't recreate class hierarchies
3. **Ignore performance**: Don't accept "good enough" when Rust can do better
4. **Copy allocations**: Don't allocate when you can borrow
5. **Skip profiling**: Don't assume, measure

### ✅ Do Instead

1. **Understand requirements**: What does the system need to do?
2. **Design for Rust**: Use ECS, traits, zero-copy, etc.
3. **Optimize early**: Design for performance from the start
4. **Use borrowing**: Prefer references over ownership transfer
5. **Profile continuously**: Measure against targets

## Implementation Checklist

For each system migration:

- [ ] Read and understand legacy implementation
- [ ] Document game logic and formulas
- [ ] Document packet structures and protocols
- [ ] Design Rust architecture (ECS components, systems)
- [ ] Identify performance requirements
- [ ] Design memory layout (cache-friendly)
- [ ] Plan concurrency strategy (lock-free where possible)
- [ ] Implement Rust version
- [ ] Write tests (unit, integration, performance)
- [ ] Profile against targets
- [ ] Document Rust implementation
- [ ] Update STATUS.md

## Resources

- **Legacy Code**: `Server-LastTs/` and `Backup - ToS1/Server3/`
- **Design Docs**: `Server/docs/`
- **Rust Impl**: `Server/` (when created)
- **ROADMAP**: Phase-by-phase implementation plan
- **STATUS**: Current progress tracking

## See Also

- [VERSIONS.md](./VERSIONS.md) - Version information
- [ROADMAP.md](./server/ROADMAP.md) - Implementation roadmap
- [ARCHITECTURE.md](./server/ARCHITECTURE.md) - Rust architecture
- [STATUS.md](./server/STATUS.md) - Implementation status

