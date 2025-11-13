# Project Context

## Purpose

The Tales of Shadowland MMORPG server is a high-performance game server designed to support 10,000 concurrent players with a 60 FPS tick rate and processing 600,000 UDP packets per second. The server is being implemented in Rust to achieve maximum performance while maintaining security and scalability. The project follows spec-driven development using OpenSpec to ensure alignment between requirements and implementation before code is written.

## Tech Stack

### Core Technologies
- **Rust** (Edition 2024, nightly 1.85+)
  - Zero-allocation design patterns
  - Lock-free data structures
  - Custom ECS framework for entity management
- **Tokio** - Async runtime for network I/O
- **Mio** - Low-level UDP socket operations (for optimization)
- **sqlx** - Async, type-safe database access
- **Custom ByteBuffer** - Zero-allocation binary serialization

### Development Tools
- **cargo-nextest** - Fast test execution
- **cargo-llvm-cov** - Code coverage (95%+ required)
- **rustfmt** - Code formatting (nightly toolchain)
- **clippy** - Linting (warnings as errors)
- **codespell** - Typo detection in code and docs

### Network Stack
- Binary UDP protocol (14-byte header)
- ChaCha20-Poly1305 encryption (AEAD)
- LZ4 compression for packets > 512 bytes
- Reliable UDP with fragmentation support
- Batch packet processing for high throughput

### Database
- **Game Server**: SQLite (via sqlx) - Embedded database for game data persistence
- **API Principal**: PostgreSQL (via sqlx) - External database for admin API and web services
- Async connection pooling
- Type-safe queries with compile-time verification

## Project Conventions

### Code Style

**Rust Formatting:**
- Use `rustfmt` with nightly toolchain: `cargo +nightly fmt --all`
- All code must pass format check before commit
- Configuration in `rustfmt.toml` or `.rustfmt.toml`

**Naming Conventions:**
- Modules: `snake_case`
- Types: `PascalCase`
- Functions: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Private fields: `snake_case` with leading underscore if unused

**Code Quality:**
- No `unwrap()` or `expect()` in production code without justification
- Use `Result<T, E>` for recoverable errors
- Use `thiserror` for custom error types
- Use `anyhow` for application-level error handling
- Document all public APIs with `///` doc comments

### Architecture Patterns

**Entity Component System (ECS):**
- Custom ECS framework for zero-allocation entity management
- Structure of Arrays (SoA) layout for cache-friendly data access
- Component-based architecture instead of inheritance
- System-based processing for game logic

**Network Architecture:**
- Binary UDP protocol with minimal overhead
- Batch packet processing (multiple packets per syscall)
- Zero-copy buffer management with pooling
- Delta compression for entity updates (60-75% bandwidth reduction)
- Area of Interest (AOI) system for efficient entity synchronization

**Serialization:**
- Custom ByteBuffer implementation
- VarInt/ZigZag encoding for variable-length integers
- Quantization system for positions and rotations
- Symbol pooling for string caching per connection
- Built-in encryption/decryption support

**Performance Patterns:**
- Zero-allocation design where possible
- Lock-free data structures using channels
- Thread-local storage for per-connection state
- SIMD optimizations where applicable
- Cache-friendly data layouts (SoA)

### Testing Strategy

**Coverage Requirements:**
- Minimum 95% code coverage (enforced by `cargo llvm-cov`)
- All tests must pass (100% pass rate required)
- Coverage check runs before every commit

**Test Types:**
- **Unit Tests**: In same file as implementation with `#[cfg(test)]`
- **Integration Tests**: In `/tests` directory
- **Async Tests**: Use `#[tokio::test]` for async test functions
- **Benchmarks**: In `/benches` directory for performance-critical code

**Test Execution:**
- Use `cargo-nextest` for faster test execution
- Run `cargo test --workspace --all-features` before commit
- Run `cargo test --doc` to verify documentation examples

**Quality Checks (MANDATORY before commit):**
```bash
cargo fmt --all -- --check           # Format check
cargo clippy --workspace --all-targets --all-features -- -D warnings  # Lint
cargo test --workspace --all-features  # All tests (100% pass)
cargo build --release                # Build verification
cargo llvm-cov --all                 # Coverage (95%+ required)
cargo audit                          # Security audit
```

### Git Workflow

**Branching Strategy:**
- Main branch: `main` (protected)
- Feature branches: `feature/description`
- Bug fixes: `fix/description`
- Breaking changes: Use OpenSpec proposals first

**Commit Conventions:**
- Use conventional commits format
- Reference OpenSpec changes when applicable
- Include test coverage in commits
- Never commit without passing quality checks

**Pre-Commit Requirements:**
1. All tests passing
2. No clippy warnings
3. Code formatted
4. Coverage meets threshold (95%+)
5. Security audit passes
6. Documentation updated if needed

**CRITICAL RULES:**
- NEVER run `git reset` or `git push` (user handles SSH authentication)
- After commit/tag, return commands needed for push/push tag
- Always use WSL Ubuntu-24.04 for terminal commands

## Domain Context

### Game Server Architecture

**Performance Targets:**
- Tick Rate: 60 FPS (16.67ms per frame)
- Concurrent Players: 10,000 simultaneous connections
- Packet Throughput: 600,000 UDP packets per second
- Latency: < 50ms (p99)
- Bandwidth Optimization: 60-75% reduction through compression and quantization

**Core Game Systems:**
- **Entity System**: ECS-based entity management with components for position, health, inventory, etc.
- **Movement System**: Grid-based collision, pathfinding (A*), movement validation (anti-cheat)
- **Combat System**: Damage calculation, resistances, status effects, buffs/debuffs
- **Item System**: Multi-tier equipment (T0-T8), rarity system, prefix/suffix modifiers, durability
- **Player System**: Level progression (1-100), 6 core attributes (STR, DEX, INT, LCK, CON, CHA), 28 skills
- **Economy System**: Gold currency, crafting, trading, auction house
- **Guild System**: Guild creation, management, guild wars
- **Quest System**: Dynamic quest generation and tracking

**Network Protocol:**
- Binary UDP protocol (no JSON/text encoding)
- 14-byte packet header (ConnectionId, Channel, Flags, Sequence)
- Three channel types: Unreliable, ReliableOrdered, ReliableUnordered
- Packet fragmentation for packets > 1200 bytes (MTU)
- Encryption: ChaCha20-Poly1305 (AEAD)
- Compression: LZ4 for packets > 512 bytes with low entropy

**Entity Synchronization:**
- Area of Interest (AOI) system for efficient updates
- Delta compression (only send changed data)
- Quantized positions and rotations
- Client-side prediction support
- Adaptive update rates based on distance

**Security Considerations:**
- End-to-end encryption (ChaCha20-Poly1305)
- Replay protection (sequence numbers)
- Integrity checks (CRC32C signatures)
- Movement validation (prevent fly hacks, wall clipping, speed hacks)
- Rate limiting for packet processing

### OpenSpec Integration

**Workflow:**
1. Create OpenSpec proposal for new features/breaking changes
2. Validate spec format with `openspec validate --strict`
3. Get approval before implementation
4. Implement according to spec requirements
5. Run quality checks (AGENT_AUTOMATION workflow)
6. Archive change after deployment

**Spec Format:**
- Requirements MUST use SHALL/MUST keywords
- Scenarios MUST use `#### Scenario:` (4 hashtags)
- GIVEN/WHEN/THEN keywords in bold
- At least one scenario per requirement
- Delta format: ADDED/MODIFIED/REMOVED/RENAMED sections

**When to Use OpenSpec:**
- ✅ New features/capabilities
- ✅ Breaking changes
- ✅ Architecture changes
- ✅ Performance/security work
- ❌ Bug fixes (restore intended behavior)
- ❌ Typos, formatting, comments
- ❌ Dependency updates (non-breaking)

## Important Constraints

### Technical Constraints

**Performance:**
- Zero-allocation design required for hot paths
- Lock-free data structures preferred
- Cache-friendly memory layouts (SoA)
- Batch processing for network operations
- SIMD optimizations where applicable

**Memory:**
- Buffer pooling for zero-allocation packet handling
- Entity pooling to reduce allocations
- Symbol pooling for string caching
- Pre-allocated data structures where possible

**Network:**
- MTU: 1200 bytes (UDP optimized)
- Minimal packet header: 14 bytes
- Bandwidth optimization: 60-75% reduction target
- Latency target: < 50ms (p99)

**Rust Edition:**
- **CRITICAL**: Always use Rust Edition 2024 (never 2021)
- Toolchain: nightly 1.85+
- Update regularly: `rustup update nightly`

**Code Quality:**
- 95%+ test coverage required
- No clippy warnings allowed (`-D warnings`)
- All tests must pass (100% pass rate)
- Documentation required for all public APIs

### Business Constraints

**Legacy Compatibility:**
- C# version exists as reference (limited to ~1,000 concurrent players)
- Must maintain protocol compatibility where possible
- Migration path from C# to Rust must be documented

**Development Process:**
- Spec-driven development (OpenSpec)
- Test-first approach (95%+ coverage)
- Quality gates before commit
- Documentation updates required for features

### Operational Constraints

**Deployment:**
- Server must support hot reloading (future requirement)
- Health check endpoints required
- Metrics and tracing for observability
- Admin REST API for server management

**Monitoring:**
- Distributed tracing support
- Metrics collection (prometheus format)
- Log aggregation
- Performance profiling tools

## External Dependencies

### Core Dependencies

**Async Runtime:**
- `tokio` - Async runtime for network I/O and task scheduling
- `tokio-util` - Utilities for Tokio (codec, time, etc.)

**Network:**
- `mio` - Low-level UDP socket operations (for optimization)
- Custom UDP implementation using raw sockets

**Database:**
- `sqlx` - Async, type-safe SQL with compile-time verification
- SQLite driver (async) - For game server data persistence
- PostgreSQL driver (async) - For API principal/admin services

**Serialization:**
- Custom ByteBuffer implementation (no external dependency)
- VarInt/ZigZag encoding (custom implementation)

**Error Handling:**
- `thiserror` - Custom error types
- `anyhow` - Application-level error handling

**Cryptography:**
- `chacha20poly1305` - AEAD encryption (ChaCha20-Poly1305)
- `rand` - Cryptographically secure random number generation

### Development Dependencies

**Testing:**
- `cargo-nextest` - Fast test execution
- `cargo-llvm-cov` - Code coverage tool
- `tokio-test` - Testing utilities for async code

**Code Quality:**
- `rustfmt` - Code formatter (nightly toolchain)
- `clippy` - Linter (warnings as errors)
- `codespell` - Typo detection

**Security:**
- `cargo-audit` - Vulnerability scanning
- `cargo-outdated` - Dependency version checking

### External Services

**CDN:**
- Item image loading (on-demand with local caching)
- Asset delivery for game content

**Monitoring (Future):**
- Prometheus-compatible metrics endpoint
- Distributed tracing (OpenTelemetry compatible)
- Health check endpoints for load balancers

**Admin API:**
- REST API for server management
- Dashboard for player management
- Configuration API for dynamic server settings

### Dependency Management Rules

**Before Adding Dependencies:**
1. Check Context7 MCP for latest version
2. Verify security advisories (`cargo audit`)
3. Review breaking changes and migration guides
4. Document version choice in `Cargo.toml` comments
5. Update CHANGELOG.md with new dependencies

**Dependency Guidelines:**
- ✅ Use latest stable versions
- ✅ Prefer well-maintained crates
- ✅ Minimize dependency count
- ✅ Use workspace dependencies for monorepos
- ❌ Don't use outdated versions without justification
- ❌ Don't add dependencies without checking latest version
