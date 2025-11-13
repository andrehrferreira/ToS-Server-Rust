# Movement and Collision System Architecture

## Overview

The movement and collision system is critical for server authority in an MMORPG. It ensures that players cannot exploit movement (fly hacks, wall clipping, speed hacks) and provides accurate collision detection for AI pathfinding, patrol routes, and environmental interactions. This document describes the grid-based collision system, movement validation, swimming system, and optimization strategies for the Rust implementation.

## System Requirements

### Security Requirements

1. **Prevent Fly Hacks**: Players cannot move to positions without valid ground collision
2. **Prevent Wall Clipping**: Players cannot pass through walls or static obstacles
3. **Prevent Speed Hacks**: Movement speed is validated server-side
4. **Prevent Teleport Hacks**: Large position changes are detected and rejected
5. **Authoritative Movement**: Server is the source of truth for all entity positions

### Functional Requirements

1. **Collision Detection**: Grid-based collision system for static and dynamic entities
2. **Pathfinding**: AI can navigate terrain and find paths around obstacles
3. **Patrol Routes**: Monsters can follow predefined patrol paths
4. **Swimming System**: Entities can enter water areas and swim (state change, animation triggers)
5. **3D Support**: System must work in 3D space (X, Y, Z coordinates)
6. **Dynamic Entities**: Support for player-built structures (houses, etc.)

## Current Implementation (C#)

### Grid-Based Collision System

The C# implementation uses a grid-based spatial partitioning system:

**Grid Structure:**
- **Cell Size**: 4.0 units (configurable)
- **Grid Array**: 2D array of cells (flattened to 1D)
- **Out-of-Bound Cells**: Dictionary for cells outside grid bounds
- **Static Entities**: Stored in cells with AABB (Axis-Aligned Bounding Box)
- **Dynamic Entities**: Stored in cells with reference to entity

**Cell Class:**
```csharp
public class Cell
{
    public readonly Vector2i Position;
    public Bag<Entity> DynamicEntities;        // Moving entities
    public QuickBag<StaticEntity> StaticEntities; // Static obstacles
    public Bag<AreaOfInterest> AreasOfInterest; // AOI observers
    
    public void Enter(Entity entity);
    public void Leave(Entity entity);
}
```

**Scene Class:**
```csharp
public partial class Scene
{
    public const float CellSize = 4.0f;
    private Cell[] Grid;                      // Main grid array
    private Dictionary<Vector2i, Cell> OutOfBoundCells; // Cells outside bounds
    private Vector2i Origin;                  // Grid origin
    private Vector2i Size;                    // Grid size
    
    public Cell GetCell(in Vector2i position);
    public void EntityBoundingBoxChanged(Entity entity, ref BoundingBox from, ref BoundingBox to);
}
```

### Shape System

The system supports two shape types:

**Shape Types:**
- **Circle**: Radius-based collision (efficient for dynamic entities)
- **Rectangle**: Oriented bounding box (OBB) for static obstacles

**Shape Structure:**
```csharp
public struct Shape
{
    public float HalfHeight;      // Rectangle height (0 for circle)
    public float HalfWidth;       // Circle radius or rectangle width
    public Vector2 Position;      // Shape center
    public Vector2 Rotation;      // Rotation direction (normalized)
    
    public bool IsCircle() => HalfHeight == 0.0f;
    public BoundingBox GetCells();           // Get grid cells this shape occupies
    public AABB GetAABB();                   // Get axis-aligned bounding box
    public bool IntersectsWith(ref Shape other); // Check intersection
    public float Raycast(in Ray ray);        // Raycast intersection
    public void ResolveCollision(ref Shape other, out CollisionResult result);
}
```

### Collision Detection

**Collision Query Methods:**
- `CheckCircle(Entity self, Vector2 position, float radius, Predicate<Entity> predicate)`: Check entities in circle
- `CheckShape(Entity self, ref Shape shape, Predicate<Entity> predicate)`: Check entities in shape
- `IntersectsCircle(Entity self, Vector2 pos, float radius, Predicate<Entity> predicate)`: Check if circle intersects
- `IntersectsShape(Entity self, ref Shape shape, Predicate<Entity> predicate)`: Check if shape intersects
- `Raycast(Vector2 from, Vector2 to, Entity self, Predicate<Entity> predicate)`: Raycast for pathfinding
- `ResolveCollision(Entity self, Predicate<Entity> predicate, out CollisionResult result)`: Resolve collision penetration

**Collision Resolution:**
```csharp
public struct CollisionResult
{
    public float Penetration;     // Penetration depth
    public Vector2 Normal;       // Collision normal (separation direction)
}
```

### Movement Validation

**Position Validation:**
```csharp
public bool IsValidPosition(Entity entity)
{
    // Check if entity is within grid bounds
    BoundingBox clusters = entity.CollisionShape.GetCells();
    for (Int32 x = clusters.From.X; x <= clusters.To.X; ++x)
        for (Int32 y = clusters.From.Y; y <= clusters.To.Y; ++y)
        {
            Cell cell = GetCell(new Vector2i(x, y));
            if (cell == null)
                return false;
        }
    
    // Check if entity collides with blocking entities
    if (IntersectsCircle(entity, entity.Position, 0.8f, Entity.BlocksCreatureOnly))
        return false;
    
    return true;
}
```

**Shape Casting (Continuous Collision Detection):**
```csharp
public Entity ShapeCast(Entity self, Vector2 direction, float length, Predicate<Entity> predicate)
{
    int tierBreak = 100;
    while (length > 0 && tierBreak > 0)
    {
        --tierBreak;
        float stepLength = MathF.Min(Constants.HalfMinColliderWidth, length);
        length -= stepLength;
        
        Vector2 velocity = direction * stepLength;
        self.Position += velocity;
        self.BoundingBox = self.CollisionShape.GetCells();
        
        CollisionResult result;
        Entity other = ResolveCollision(self, predicate, out result);
        
        if (result.Penetration > 0)
        {
            // Resolve penetration
            self.Position += (result.Normal * (result.Penetration + 0.01f));
            self.BoundingBox = self.CollisionShape.GetCells();
            return other;
        }
    }
    return null;
}
```

### Raycasting for Pathfinding

**Raycast Implementation:**
```csharp
public Entity Raycast(Vector2 from, Vector2 to, Entity self, Predicate<Entity> predicate)
{
    // Bresenham-like line algorithm for grid traversal
    int x0 = (int)(from.X <= to.X ? MathF.Floor(from.X / Scene.CellSize) : MathF.Ceiling(from.X / Scene.CellSize));
    int y0 = (int)(from.Y <= to.Y ? MathF.Floor(from.Y / Scene.CellSize) : MathF.Ceiling(from.Y / Scene.CellSize));
    int x1 = (int)(from.X >= to.X ? MathF.Floor(to.X / Scene.CellSize) : MathF.Ceiling(to.X / Scene.CellSize));
    int y1 = (int)(from.Y >= to.Y ? MathF.Floor(to.Y / Scene.CellSize) : MathF.Ceiling(to.Y / Scene.CellSize));
    
    // Traverse cells along line
    // Check entities in each cell
    // Return first intersecting entity
}
```

## TypeScript Implementation (Limited)

### Client-Side Collision (Not Recommended)

The TypeScript version did not implement server-side collision detection due to performance limitations:

**Limitations:**
- Client resolves collisions (vulnerable to hacks)
- No server-side validation
- No pathfinding for AI
- No swimming system
- Performance bottlenecks at scale

**Why Not Recommended:**
- **Security Risk**: Clients can manipulate collision data
- **Cheating**: Fly hacks, wall clipping, speed hacks possible
- **Inconsistency**: Different clients may have different collision results
- **No AI Control**: Cannot control monster pathfinding server-side

## Rust Implementation Strategy

### 1. Grid-Based Spatial Partitioning (3D Support)

**3D Grid Structure:**
```rust
// 3D grid with configurable cell size
pub struct SpatialGrid {
    cell_size: f32,              // Cell size (e.g., 4.0 units)
    origin: Vector3i,            // Grid origin
    size: Vector3i,              // Grid dimensions
    cells: Vec<Option<Cell>>,    // Flattened 3D array: cells[x + y * size_x + z * size_x * size_y]
    out_of_bounds: HashMap<Vector3i, Cell>, // Cells outside grid bounds
}

// Cell stores entities by type
pub struct Cell {
    position: Vector3i,
    static_entities: Vec<StaticEntityRef>,   // Static obstacles (walls, terrain)
    dynamic_entities: Vec<EntityId>,         // Dynamic entities (players, monsters)
    water_volumes: Vec<WaterVolume>,         // Water areas for swimming
    height_map: Option<HeightMap>,           // Terrain height data
}

// Static entity reference with AABB
pub struct StaticEntityRef {
    entity_id: EntityId,
    aabb: AABB,                  // Axis-aligned bounding box
    shape: CollisionShape,       // Collision shape (box, sphere, mesh)
}

// Water volume for swimming detection
pub struct WaterVolume {
    bounds: AABB,                // Water volume bounds
    surface_height: f32,         // Water surface height
    depth: f32,                  // Water depth
}
```

**Benefits:**
- **3D Support**: Works in 3D space (X, Y, Z)
- **Efficient Queries**: O(1) cell lookup, O(n) entity queries in cells
- **Cache-Friendly**: Cells stored in contiguous memory
- **Scalable**: Handles large worlds with millions of entities

### 2. Collision Shapes (3D)

**Collision Shape Types:**
```rust
pub enum CollisionShape {
    Sphere {
        center: Vector3,
        radius: f32,
    },
    Box {
        center: Vector3,
        half_extents: Vector3,
        rotation: Quaternion,
    },
    Capsule {
        center: Vector3,
        radius: f32,
        height: f32,
        rotation: Quaternion,
    },
    Mesh {
        vertices: Vec<Vector3>,
        indices: Vec<u32>,
        bounds: AABB,
    },
}

// Axis-aligned bounding box for broad phase
pub struct AABB {
    min: Vector3,
    max: Vector3,
}

impl AABB {
    pub fn contains(&self, point: Vector3) -> bool;
    pub fn intersects(&self, other: &AABB) -> bool;
    pub fn expand(&mut self, point: Vector3);
}
```

### 3. Movement Validation System

**Movement Validator:**
```rust
pub struct MovementValidator {
    grid: Arc<SpatialGrid>,
    max_speed: f32,              // Maximum movement speed
    max_teleport_distance: f32,  // Maximum teleport distance
    collision_margin: f32,       // Collision margin (e.g., 0.1 units)
}

impl MovementValidator {
    // Validate movement request from client
    pub fn validate_movement(
        &self,
        entity_id: EntityId,
        from: Vector3,
        to: Vector3,
        delta_time: f32,
    ) -> Result<Vector3, MovementError> {
        // 1. Check speed limit
        let distance = (to - from).length();
        let speed = distance / delta_time;
        if speed > self.max_speed {
            return Err(MovementError::SpeedExceeded { speed, max: self.max_speed });
        }
        
        // 2. Check teleport distance
        if distance > self.max_teleport_distance {
            return Err(MovementError::TeleportDistanceExceeded { distance });
        }
        
        // 3. Check ground collision (prevent fly hacks)
        if !self.has_ground_collision(to) {
            return Err(MovementError::NoGroundCollision);
        }
        
        // 4. Check wall collision (prevent wall clipping)
        if self.intersects_walls(from, to) {
            return Err(MovementError::WallCollision);
        }
        
        // 5. Resolve collision with dynamic entities
        let resolved_position = self.resolve_collision(entity_id, from, to);
        
        Ok(resolved_position)
    }
    
    // Check if position has valid ground collision
    fn has_ground_collision(&self, position: Vector3) -> bool {
        // Raycast downward to check for ground
        let ray = Ray {
            origin: position + Vector3::new(0.0, 10.0, 0.0), // Start above position
            direction: Vector3::new(0.0, -1.0, 0.0),         // Downward
            max_distance: 20.0,
        };
        
        if let Some(hit) = self.grid.raycast(&ray, |entity| entity.blocks_creature) {
            // Check if hit is close to position (within tolerance)
            let ground_height = hit.point.y;
            let height_difference = (position.y - ground_height).abs();
            height_difference <= 2.0 // Allow 2 units tolerance
        } else {
            false // No ground found
        }
    }
    
    // Check if movement path intersects walls
    fn intersects_walls(&self, from: Vector3, to: Vector3) -> bool {
        let direction = (to - from).normalize();
        let distance = (to - from).length();
        let ray = Ray {
            origin: from,
            direction,
            max_distance: distance,
        };
        
        // Raycast to check for wall collisions
        if let Some(hit) = self.grid.raycast(&ray, |entity| entity.blocks_creature) {
            // Check if hit is close to destination (allow small margin)
            let hit_distance = (hit.point - from).length();
            hit_distance < distance - self.collision_margin
        } else {
            false
        }
    }
    
    // Resolve collision with dynamic entities
    fn resolve_collision(&self, entity_id: EntityId, from: Vector3, to: Vector3) -> Vector3 {
        // Continuous collision detection (shape casting)
        let mut current_position = from;
        let direction = (to - from).normalize();
        let mut remaining_distance = (to - from).length();
        let max_iterations = 10;
        let step_size = 0.5; // Small steps for accuracy
        
        for _ in 0..max_iterations {
            if remaining_distance <= 0.0 {
                break;
            }
            
            let step_distance = remaining_distance.min(step_size);
            let next_position = current_position + direction * step_distance;
            
            // Check for collisions
            if let Some(collision) = self.grid.check_collision(entity_id, next_position) {
                // Resolve collision by moving along collision normal
                let penetration = collision.penetration;
                let normal = collision.normal;
                current_position += normal * (penetration + self.collision_margin);
                remaining_distance = 0.0; // Stop movement
            } else {
                current_position = next_position;
                remaining_distance -= step_distance;
            }
        }
        
        current_position
    }
}

pub enum MovementError {
    SpeedExceeded { speed: f32, max: f32 },
    TeleportDistanceExceeded { distance: f32 },
    NoGroundCollision,
    WallCollision,
    InvalidPosition,
}
```

### 4. Swimming System

**Water Detection and State Management:**
```rust
// Water volume component
#[derive(Component)]
pub struct WaterVolume {
    bounds: AABB,
    surface_height: f32,
    depth: f32,
    current: Option<Vector3>, // Water current (for rivers)
}

// Swimming state component
#[derive(Component)]
pub struct SwimmingState {
    is_swimming: bool,
    water_volume_id: Option<EntityId>,
    surface_height: f32,
    depth: f32,
}

// Movement system with swimming support
pub fn movement_system(
    mut positions: Query<&mut Position>,
    mut swimming_states: Query<&mut SwimmingState>,
    water_volumes: Query<&WaterVolume>,
    grid: Res<SpatialGrid>,
    time: Res<Time>,
) {
    for (entity_id, mut position, mut swimming_state) in positions.iter_mut().zip(swimming_states.iter_mut()) {
        // Check if entity is in water
        let entity_position = position.0;
        let mut in_water = false;
        let mut water_volume_id = None;
        let mut surface_height = 0.0;
        let mut depth = 0.0;
        
        // Query water volumes at entity position
        let cell = grid.get_cell(entity_position);
        for water_volume in cell.water_volumes.iter() {
            if water_volume.bounds.contains(entity_position) {
                // Check if entity is below water surface
                if entity_position.y <= water_volume.surface_height {
                    in_water = true;
                    water_volume_id = Some(water_volume.entity_id);
                    surface_height = water_volume.surface_height;
                    depth = water_volume.surface_height - entity_position.y;
                    break;
                }
            }
        }
        
        // Update swimming state
        if in_water != swimming_state.is_swimming {
            swimming_state.is_swimming = in_water;
            swimming_state.water_volume_id = water_volume_id;
            swimming_state.surface_height = surface_height;
            swimming_state.depth = depth;
            
            // Trigger animation change event
            if in_water {
                // Switch to swimming animation
                events.send(AnimationEvent::StartSwimming { entity_id });
            } else {
                // Switch to walking animation
                events.send(AnimationEvent::StopSwimming { entity_id });
            }
        }
        
        // Apply water physics (buoyancy, current)
        if in_water {
            // Apply buoyancy (float to surface)
            let buoyancy_force = (surface_height - entity_position.y) * 9.81 * 0.5;
            position.0.y += buoyancy_force * time.delta_seconds();
            
            // Apply water current (if river)
            if let Some(current) = water_volume.current {
                position.0 += current * time.delta_seconds();
            }
            
            // Clamp to water surface (prevent flying above water)
            if position.0.y > surface_height {
                position.0.y = surface_height;
            }
        }
    }
}
```

### 5. Pathfinding System

**A* Pathfinding with Grid:**
```rust
pub struct PathfindingSystem {
    grid: Arc<SpatialGrid>,
    pathfinder: AStarPathfinder,
}

impl PathfindingSystem {
    // Find path from start to goal
    pub fn find_path(
        &self,
        start: Vector3,
        goal: Vector3,
        entity_id: EntityId,
    ) -> Option<Vec<Vector3>> {
        // Convert world positions to grid coordinates
        let start_cell = self.grid.world_to_cell(start);
        let goal_cell = self.grid.world_to_cell(goal);
        
        // A* pathfinding on grid
        let path_cells = self.pathfinder.find_path(
            start_cell,
            goal_cell,
            |cell| self.is_walkable(cell, entity_id),
        )?;
        
        // Convert grid coordinates to world positions
        let path: Vec<Vector3> = path_cells
            .iter()
            .map(|cell| self.grid.cell_to_world(*cell))
            .collect();
        
        Some(path)
    }
    
    // Check if cell is walkable for entity
    fn is_walkable(&self, cell: Vector3i, entity_id: EntityId) -> bool {
        let cell_data = self.grid.get_cell(cell)?;
        
        // Check static entities (walls, obstacles)
        for static_entity in cell_data.static_entities.iter() {
            if static_entity.blocks_creature {
                return false;
            }
        }
        
        // Check height map (terrain)
        if let Some(height_map) = &cell_data.height_map {
            // Check if height difference is too steep
            let height = height_map.get_height(cell);
            // Allow walking if slope is reasonable
        }
        
        true
    }
}
```

### 6. AI Patrol System

**Patrol Route System:**
```rust
// Patrol route component
#[derive(Component)]
pub struct PatrolRoute {
    waypoints: Vec<Vector3>,
    current_waypoint_index: usize,
    patrol_speed: f32,
    wait_time: f32,
    wait_timer: f32,
}

// AI patrol system
pub fn ai_patrol_system(
    mut patrol_routes: Query<&mut PatrolRoute>,
    mut positions: Query<&mut Position>,
    mut velocities: Query<&mut Velocity>,
    pathfinding: Res<PathfindingSystem>,
    time: Res<Time>,
) {
    for (entity_id, mut patrol_route, mut position, mut velocity) in
        patrol_routes.iter_mut().zip(positions.iter_mut()).zip(velocities.iter_mut())
    {
        if patrol_route.waypoints.is_empty() {
            continue;
        }
        
        let current_waypoint = patrol_route.waypoints[patrol_route.current_waypoint_index];
        let distance_to_waypoint = (position.0 - current_waypoint).length();
        
        // Check if reached waypoint
        if distance_to_waypoint < 1.0 {
            // Wait at waypoint
            patrol_route.wait_timer += time.delta_seconds();
            if patrol_route.wait_timer >= patrol_route.wait_time {
                // Move to next waypoint
                patrol_route.current_waypoint_index =
                    (patrol_route.current_waypoint_index + 1) % patrol_route.waypoints.len();
                patrol_route.wait_timer = 0.0;
            }
            velocity.0 = Vector3::ZERO;
        } else {
            // Move towards waypoint
            let direction = (current_waypoint - position.0).normalize();
            velocity.0 = direction * patrol_route.patrol_speed;
            
            // Optional: Use pathfinding for complex terrain
            // if let Some(path) = pathfinding.find_path(position.0, current_waypoint, entity_id) {
            //     // Follow path
            // }
        }
    }
}
```

### 7. Performance Optimizations

**Spatial Hashing (Alternative to Grid):**
```rust
// Spatial hash for dynamic entities (more efficient for sparse entities)
pub struct SpatialHash {
    cell_size: f32,
    buckets: HashMap<Vector3i, Vec<EntityId>>,
}

impl SpatialHash {
    pub fn insert(&mut self, entity_id: EntityId, position: Vector3) {
        let cell = self.world_to_cell(position);
        self.buckets.entry(cell).or_insert_with(Vec::new).push(entity_id);
    }
    
    pub fn query(&self, bounds: AABB) -> Vec<EntityId> {
        let mut results = Vec::new();
        let min_cell = self.world_to_cell(bounds.min);
        let max_cell = self.world_to_cell(bounds.max);
        
        for x in min_cell.x..=max_cell.x {
            for y in min_cell.y..=max_cell.y {
                for z in min_cell.z..=max_cell.z {
                    let cell = Vector3i::new(x, y, z);
                    if let Some(entities) = self.buckets.get(&cell) {
                        results.extend(entities.iter().copied());
                    }
                }
            }
        }
        
        results
    }
}
```

**Broad Phase / Narrow Phase:**
```rust
// Broad phase: Quick AABB checks
pub fn broad_phase_collision(
    grid: &SpatialGrid,
    entity_id: EntityId,
    bounds: AABB,
) -> Vec<EntityId> {
    let mut candidates = Vec::new();
    
    // Query entities in overlapping cells
    let cells = grid.get_cells_in_bounds(bounds);
    for cell in cells {
        for static_entity in cell.static_entities.iter() {
            if static_entity.aabb.intersects(&bounds) {
                candidates.push(static_entity.entity_id);
            }
        }
        for dynamic_entity_id in cell.dynamic_entities.iter() {
            if *dynamic_entity_id != entity_id {
                candidates.push(*dynamic_entity_id);
            }
        }
    }
    
    candidates
}

// Narrow phase: Precise shape intersection
pub fn narrow_phase_collision(
    entity_shape: &CollisionShape,
    other_shape: &CollisionShape,
) -> Option<CollisionResult> {
    match (entity_shape, other_shape) {
        (CollisionShape::Sphere(s1), CollisionShape::Sphere(s2)) => {
            sphere_sphere_collision(s1, s2)
        }
        (CollisionShape::Box(b1), CollisionShape::Box(b2)) => {
            box_box_collision(b1, b2)
        }
        (CollisionShape::Sphere(s), CollisionShape::Box(b)) => {
            sphere_box_collision(s, b)
        }
        // ... other combinations
        _ => None,
    }
}
```

## Security Considerations

### 1. Movement Validation

**Server Authority:**
- All movement requests validated server-side
- Client positions are suggestions, not authoritative
- Server corrects invalid movements
- Large position changes rejected as teleport attempts

**Validation Checks:**
- Speed limit check (prevent speed hacks)
- Teleport distance check (prevent teleport hacks)
- Ground collision check (prevent fly hacks)
- Wall collision check (prevent wall clipping)
- Height validation (prevent climbing invalid surfaces)

### 2. Anti-Cheat Measures

**Detection:**
- Monitor movement patterns for anomalies
- Track speed violations
- Detect impossible position changes
- Log suspicious activities

**Response:**
- Reject invalid movement requests
- Correct positions to valid locations
- Kick players with repeated violations
- Ban players with severe violations

### 3. Rate Limiting

**Movement Rate Limits:**
- Maximum movement updates per second (e.g., 60 Hz)
- Maximum position change per update
- Cooldown on rejected movements
- Progressive penalties for violations

## Migration Strategy

### Phase 1: Core Collision System
1. Implement 3D spatial grid
2. Implement collision shapes (sphere, box, capsule)
3. Implement broad phase collision detection
4. Implement narrow phase collision detection
5. Implement collision resolution

### Phase 2: Movement Validation
1. Implement movement validator
2. Implement ground collision check
3. Implement wall collision check
4. Implement speed validation
5. Implement teleport detection

### Phase 3: Pathfinding and AI
1. Implement A* pathfinding
2. Implement patrol route system
3. Implement AI movement system
4. Integrate with collision system

### Phase 4: Swimming System
1. Implement water volume system
2. Implement swimming state management
3. Implement water physics (buoyancy, current)
4. Implement animation triggers
5. Integrate with movement system

### Phase 5: Optimization
1. Implement spatial hashing for dynamic entities
2. Optimize collision queries
3. Implement multithreading for collision detection
4. Profile and optimize hot paths

## Performance Targets

### Collision Detection
- **Query Time**: < 1ms for 1000 entities
- **Update Time**: < 5ms for 10,000 entities
- **Memory Usage**: < 100MB for large world

### Movement Validation
- **Validation Time**: < 0.1ms per entity
- **Throughput**: 10,000 validations/second
- **Latency**: < 16ms (one frame at 60 FPS)

### Pathfinding
- **Pathfinding Time**: < 10ms for complex paths
- **Path Length**: Up to 1000 waypoints
- **Concurrent Pathfinding**: 100+ paths simultaneously

## Conclusion

The movement and collision system is critical for server authority and security in an MMORPG. The Rust implementation should use:

- **3D Grid-Based Spatial Partitioning**: Efficient collision queries
- **Server-Side Movement Validation**: Prevent hacks and exploits
- **Pathfinding System**: AI navigation and patrol routes
- **Swimming System**: Water areas and state management
- **Performance Optimizations**: Spatial hashing, broad/narrow phase, multithreading

This approach provides security, performance, and scalability for a high-performance MMORPG server supporting 10,000 concurrent players.

