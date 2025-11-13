# Version Information

## Current Status

### 🚀 Rust (Active Development - Target Version)
**Status**: In Development  
**Target**: Production Server  
**Goal**: 10,000 concurrent players, 600,000 packets/second, 60 FPS tick rate

This is the **primary and only supported version** going forward. All new development and documentation focus on Rust implementation.

**Architecture**:
- High-performance UDP protocol
- Zero-allocation design
- Lock-free data structures
- ECS (Entity Component System)
- Advanced security (ChaCha20-Poly1305, X25519)

**Documentation Focus**:
- All system documentation describes intended Rust implementation
- Reference implementations in TypeScript/C# used only for understanding system design
- Performance targets based on Rust capabilities

### 📚 TypeScript (Legacy - Reference Only)
**Status**: Legacy/Deprecated  
**Last Updated**: 2024  
**Purpose**: Reference implementation for understanding game systems

This version serves as **reference documentation only**. It demonstrates:
- Complete game system implementations
- Business logic and game mechanics
- Entity relationships and workflows
- Network packet structures

**Not Recommended For**:
- Production use
- New features
- Performance-critical applications

### 🗂️ C# (Legacy - Reference Only)
**Status**: Legacy/Deprecated  
**Last Updated**: 2023  
**Purpose**: Historical reference

This version is maintained only for:
- Understanding original architecture decisions
- Comparing different implementation approaches
- Historical context

**Known Limitations**:
- GC pauses affecting performance
- Limited to ~1,000 concurrent players
- Memory allocation overhead

## Documentation Strategy

### How to Read Documentation

1. **System Design**: All documentation describes the Rust implementation design
2. **Code Examples**: TypeScript/C# code shown as reference for understanding concepts
3. **Performance**: All targets and metrics are for Rust version
4. **Implementation Status**: Check `STATUS.md` for current Rust implementation progress

### Documentation Conventions

```markdown
## Implementation Status
- **Target**: Rust
- **Status**: ✅ Implemented | ⚠️ In Progress | 🔲 Planned
- **Reference**: TypeScript example for design understanding
```

### Migration Path

Legacy implementations → Design documentation → Rust implementation

1. **Analyze** legacy TypeScript/C# code
2. **Document** system design and requirements
3. **Design** Rust architecture with performance optimizations
4. **Implement** in Rust following documented design
5. **Test** against performance targets

## Version Comparison

| Feature | TypeScript | C# | Rust |
|---------|------------|----|----- |
| Status | Legacy | Legacy | Active |
| Players | ~1,000 | ~1,000 | 10,000 (target) |
| Packets/sec | ~50k | ~50k | 600k (target) |
| Tick Rate | 30 FPS | 30 FPS | 60 FPS (target) |
| Memory | GC managed | GC managed | Zero-allocation |
| Security | Basic | Basic | Advanced |
| Production | ❌ No | ❌ No | ✅ Yes (when ready) |

## See Also

- [ROADMAP.md](./server/ROADMAP.md) - Rust implementation roadmap
- [STATUS.md](./server/STATUS.md) - Current Rust implementation status
- [ARCHITECTURE.md](./server/ARCHITECTURE.md) - Rust architecture design
- [MIGRATION.md](./MIGRATION.md) - How legacy code informs Rust design

