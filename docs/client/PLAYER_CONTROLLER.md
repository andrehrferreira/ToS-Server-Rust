# Player Controller System Documentation

## Overview

The player controller system handles player movement, animation, and synchronization with the game server. The system uses Unreal Engine's Motion Matching for high-quality, thread-safe character movement, implements swimming mechanics, and provides glider-based flight for mobility. The player controller ensures smooth, responsive gameplay while maintaining accurate synchronization with the server.

## Key Features

- **Motion Matching**: Unreal Engine's Motion Matching system for character movement
- **Thread-Safe Movement**: Motion Matching is thread-safe, ensuring smooth performance
- **Target Lock System**: Lock on targets for healing or combat focus
- **Swimming System**: Underwater movement and swimming mechanics
- **Glider Flight**: Glider-based flight system for mobility (no climbing)
- **Server Synchronization**: Accurate position and state synchronization with server
- **Non-Blocking Movement**: Client movement never blocked by server validation
- **Lag Compensation**: Client-side prediction with server reconciliation
- **Pathfinding Interaction**: Automatic pathfinding to interactive objects when out of range

## Camera System

**See [CAMERA.md](./CAMERA.md) for complete camera system documentation.**

## Target Lock System

### Overview

The target lock system allows players to lock onto targets for focused combat or healing. Similar to ArcheAge's system, players can lock onto enemies or allies to maintain focus during combat.

### Target Lock Features

**Lock Modes:**
- **Enemy Lock**: Lock onto enemy for combat focus
- **Ally Lock**: Lock onto ally for healing/support
- **No Lock**: Free camera and targeting

**Target Selection:**
- **TAB Key**: Cycle through nearby targets
- **Click Selection**: Click on enemy/ally to select
- **Auto-Lock**: Optional auto-lock on nearest enemy

### Target Lock Implementation

```cpp
// Unreal Engine C++ Implementation
class UTargetLockComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    // Current locked target
    UPROPERTY(BlueprintReadOnly)
    AActor* LockedTarget;
    
    // Lock distance
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float MaxLockDistance = 5000.0f;
    
    // Target search radius
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float TargetSearchRadius = 1000.0f;
    
    // Lock functions
    UFUNCTION(BlueprintCallable)
    void LockTarget(AActor* Target);
    
    UFUNCTION(BlueprintCallable)
    void UnlockTarget();
    
    UFUNCTION(BlueprintCallable)
    void CycleNextTarget();
    
    UFUNCTION(BlueprintCallable)
    void CyclePreviousTarget();
    
    // Get nearby targets
    UFUNCTION(BlueprintCallable)
    TArray<AActor*> GetNearbyTargets(bool EnemiesOnly = true) const;
    
    // Check if target is valid
    UFUNCTION(BlueprintCallable)
    bool IsTargetValid(AActor* Target) const;
    
    // Update target lock
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
    
private:
    // Find nearest target
    AActor* FindNearestTarget(bool EnemiesOnly) const;
    
    // Validate lock distance
    bool IsTargetInRange(AActor* Target) const;
    
    // Update camera to follow target
    void UpdateCameraToTarget(float DeltaTime);
};
```

### Target Cycling

**TAB Key Functionality:**
- **Press TAB**: Cycle to next nearby target
- **Target Priority**: Enemies prioritized over allies
- **Distance Priority**: Closer targets prioritized
- **Cycle Direction**: Forward cycling through target list

**Target Selection:**
- **Click on Enemy**: Immediately lock onto clicked enemy
- **Click on Ally**: Lock onto ally for healing/support
- **Visual Feedback**: Highlighted target indicator
- **Range Indicator**: Shows if target is in range

### Target Lock Behavior

**Camera Behavior:**
- Camera follows locked target (see [CAMERA.md](./CAMERA.md) for camera details)
- Maintains zoom level during lock
- Smooth camera movement to target
- Unlocks if target moves out of range

**Combat Integration:**
- All attacks target locked enemy
- Skills automatically target locked enemy
- Healing automatically targets locked ally
- Auto-rotation towards locked target

**Lock Conditions:**
- Target must be within lock distance
- Target must be visible (not behind wall)
- Target must be valid (alive, not dead)
- Lock breaks if target dies or moves out of range

## Input Controls

### Overview

The input control system handles all player input including movement, camera, targeting, and interactions.

### Movement Controls

**See [INPUT.md](./INPUT.md) for complete movement controls documentation.**

### Camera Controls

**See [CAMERA.md](./CAMERA.md) for complete camera controls documentation.**

### Target Lock Controls

The target lock system is similar to ArcheAge's implementation, allowing players to lock onto targets for focused combat or healing.

**Target Selection:**
- **TAB**: Cycle through nearby targets (enemies prioritized, similar to ArcheAge)
- **Z**: Lock onto selected target
- **` (Backtick/Grave Accent)**: Cancel/unlock current target
- **Left Click**: Select target by clicking directly on enemy/ally
- **Mouse Wheel**: Cycle targets while locked (if multiple targets available)

**Target Lock Behavior:**
- **Auto-Follow**: Camera follows locked target (see [CAMERA.md](./CAMERA.md))
- **Auto-Rotate**: Character rotates towards locked target
- **Range Indicator**: Visual indicator shows lock range
- **Target Health Bar**: Display health bar of locked target
- **Lock for Healing**: Lock on allies to target healing abilities
- **Lock for Combat**: Lock on enemies to maintain combat focus
- **Autoattack**: Left mouse button triggers autoattack on locked target (see [INPUT.md](./INPUT.md))

### Interaction Controls

**Object Interaction:**
- **E Key**: Interact with object/NPC/chest (primary interaction key)
- **F Key**: Alternative interaction key
- **Click Interaction**: Click directly on interactive object to interact

**Pathfinding Interaction (Auto-Pathfinding):**
When a player clicks on an interactive object that is out of interaction range, the system automatically initiates pathfinding to the target location.

**Auto-Pathfinding Behavior:**
- **Automatic Pathfinding**: If interaction target is out of range, player automatically pathfinds to location
- **Range Check**: System checks if object is within interaction range (e.g., 200 units)
- **Pathfinding Range**: Maximum distance for auto-pathfinding (e.g., 5000 units)
- **Pathfinding Execution**: Player automatically moves to interaction target using navigation system
- **Interaction on Arrival**: Automatically interact when player reaches interaction range
- **Pathfinding Cancellation**: Player can cancel pathfinding by moving manually or pressing cancel key
- **Visual Feedback**: Visual indicator shows pathfinding path and target location

**Pathfinding Flow:**
1. Player clicks on interactive object (NPC, chest, item, etc.)
2. System checks if object is within interaction range
3. If out of range but within pathfinding range, start automatic pathfinding
4. Player automatically moves to target location
5. When within interaction range, automatically execute interaction
6. Player can cancel at any time by moving or pressing cancel

### Interaction System

```cpp
// Unreal Engine C++ Implementation
class UInteractionComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    // Interaction range
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float InteractionRange = 200.0f;
    
    // Pathfinding range (max distance to auto-pathfind)
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float PathfindingRange = 5000.0f;
    
    // Current interaction target
    UPROPERTY(BlueprintReadOnly)
    AActor* InteractionTarget;
    
    // Is pathfinding to interaction
    UPROPERTY(BlueprintReadOnly)
    bool bIsPathfindingToInteraction;
    
    // Interaction functions
    UFUNCTION(BlueprintCallable)
    void InteractWithTarget(AActor* Target);
    
    UFUNCTION(BlueprintCallable)
    void CancelInteraction();
    
    // Check if target is in range
    UFUNCTION(BlueprintCallable)
    bool IsTargetInRange(AActor* Target) const;
    
    // Start pathfinding to interaction
    UFUNCTION(BlueprintCallable)
    void StartPathfindingToInteraction(AActor* Target);
    
    // Update interaction
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
    
private:
    // Pathfinding component reference
    UPROPERTY()
    class UNavigationPath* CurrentPath;
    
    // Check if reached interaction range
    bool HasReachedInteractionRange(AActor* Target) const;
    
    // Execute interaction
    void ExecuteInteraction(AActor* Target);
};
```

### Pathfinding to Interaction

**Auto-Pathfinding Flow:**
1. **Player Clicks Object**: Player clicks on interactive object
2. **Range Check**: System checks if object is within interaction range
3. **Out of Range**: If out of range, check if within pathfinding range
4. **Start Pathfinding**: If within pathfinding range, start pathfinding
5. **Move to Target**: Player automatically moves to interaction target
6. **Reach Range**: When within interaction range, automatically interact
7. **Cancel**: Player can cancel by moving or pressing cancel key

**Pathfinding Behavior:**
- Uses Unreal Engine's navigation system
- Follows valid paths around obstacles
- Stops if path becomes blocked
- Cancels if target becomes invalid
- Visual feedback during pathfinding

## Motion Matching System

### Overview

Motion Matching is Unreal Engine's advanced animation system that provides:
- **High Quality**: Natural, fluid character movement
- **Thread Safety**: Thread-safe implementation for optimal performance
- **Responsive Controls**: Immediate response to player input
- **Smooth Transitions**: Seamless transitions between movement states

### Motion Matching Integration

**Unreal Engine Implementation:**
- Uses Unreal's Motion Matching plugin/system
- Character movement driven by Motion Matching
- Animation-driven movement for natural feel
- Thread-safe execution ensures no performance issues

**Movement States:**
- **Idle**: Standing still
- **Walking**: Normal ground movement
- **Running**: Fast ground movement
- **Swimming**: Underwater movement
- **Flying**: Glider flight
- **Falling**: Falling/jumping states

### Client Movement Flow

1. **Input Received**: Player provides input (keyboard/mouse)
2. **Motion Matching**: Motion Matching system processes input
3. **Immediate Movement**: Character moves immediately (non-blocking)
4. **Position Update**: Client sends position update to server
5. **Server Validation**: Server validates with lag compensation
6. **Reconciliation**: Client receives server corrections if needed
7. **Smooth Interpolation**: Client smoothly interpolates to corrected position

## Swimming System

### Overview

The swimming system provides underwater movement mechanics for players.

### Swimming Mechanics

**Swimming States:**
- **Surface Swimming**: Swimming on water surface
- **Underwater Swimming**: Swimming below water surface
- **Diving**: Descending into water
- **Emerging**: Ascending from water
- **Underwater Breathing**: Limited breath mechanics

**Swimming Controls:**
- **Forward/Backward**: Swim forward or backward
- **Left/Right**: Turn while swimming
- **Up/Down**: Ascend or descend in water
- **Sprint**: Faster swimming (consumes stamina)

### Swimming Implementation

```cpp
// Unreal Engine C++ Implementation
class USwimmingComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    // Swimming state
    UPROPERTY(BlueprintReadOnly)
    ESwimmingState CurrentSwimmingState;
    
    // Water detection
    UFUNCTION(BlueprintCallable)
    bool IsInWater() const;
    
    // Swimming speed
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float SwimmingSpeed = 300.0f;
    
    // Underwater breath
    UPROPERTY(BlueprintReadOnly)
    float BreathRemaining = 100.0f;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float MaxBreath = 100.0f;
    
    // Update swimming
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
    
private:
    // Check if character is in water
    bool CheckWaterCollision() const;
    
    // Update breath mechanics
    void UpdateBreath(float DeltaTime);
    
    // Apply swimming movement
    void ApplySwimmingMovement(float DeltaTime);
};
```

### Water Detection

**Water Detection Methods:**
- **Collision Detection**: Check collision with water volumes
- **Height Check**: Check if character is below water level
- **Volume Overlap**: Check overlap with water volumes
- **Surface Detection**: Detect water surface for transitions

### Swimming Animation

**Motion Matching Integration:**
- Swimming animations integrated into Motion Matching
- Smooth transitions between swimming states
- Natural swimming movement
- Thread-safe animation updates

## Glider Flight System

### Overview

The glider system provides flight-based mobility using gliders. Players can deploy gliders to glide through the air, providing enhanced mobility across the game world.

### Glider Mechanics

**Glider Features:**
- **Deployment**: Deploy glider from inventory
- **Flight**: Glide through the air
- **Control**: Control direction and pitch while gliding
- **Landing**: Land safely on ground
- **Stamina**: Gliding consumes stamina over time
- **Height Loss**: Gradual descent while gliding

**Glider States:**
- **Deployed**: Glider is active and player is gliding
- **Folded**: Glider is stored and inactive
- **Landing**: Glider is being landed

### Glider Implementation

```cpp
// Unreal Engine C++ Implementation
class UGliderComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    // Glider state
    UPROPERTY(BlueprintReadOnly)
    EGliderState CurrentGliderState;
    
    // Deploy glider
    UFUNCTION(BlueprintCallable)
    void DeployGlider();
    
    // Fold glider
    UFUNCTION(BlueprintCallable)
    void FoldGlider();
    
    // Glider speed
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float GliderSpeed = 500.0f;
    
    // Descent rate
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float DescentRate = 50.0f;
    
    // Stamina consumption per second
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float StaminaConsumptionPerSecond = 10.0f;
    
    // Update glider
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
    
private:
    // Apply glider movement
    void ApplyGliderMovement(float DeltaTime);
    
    // Check if can deploy glider (must be falling/airborne)
    bool CanDeployGlider() const;
    
    // Check if should auto-land (ground proximity or stamina)
    bool ShouldAutoLand() const;
};
```

### Glider Deployment

**Deployment Conditions:**
- Player must be airborne (falling or in air)
- Player must have glider in inventory
- Player must have sufficient stamina
- Player must not be in water

**Deployment Process:**
1. Check deployment conditions
2. Equip glider from inventory
3. Transition to gliding state
4. Apply glider movement physics
5. Start stamina consumption

### Glider Flight

**Flight Mechanics:**
- **Forward Movement**: Glide forward at glider speed
- **Direction Control**: Control direction with input
- **Pitch Control**: Control pitch (up/down) with input
- **Descent**: Gradual descent over time
- **Stamina**: Consumes stamina while gliding
- **Speed**: Maintains forward speed while gliding

**Flight Controls:**
- **W/S**: Pitch up/down
- **A/D**: Turn left/right
- **Space**: Increase descent rate (faster landing)
- **Shift**: Maintain altitude (consumes more stamina)

### Glider Landing

**Landing Conditions:**
- Player reaches ground
- Player runs out of stamina
- Player manually folds glider
- Player enters water

**Landing Process:**
1. Detect landing condition
2. Transition to landing state
3. Apply landing animation
4. Fold glider automatically
5. Return to normal movement

## Movement Synchronization

### Client-Server Sync

**Synchronization Flow:**
1. **Client Movement**: Player moves using Motion Matching (immediate)
2. **Position Update**: Client sends position to server
3. **Server Validation**: Server validates with lag compensation
4. **Server Response**: Server sends validation/correction
5. **Client Reconciliation**: Client reconciles if correction needed
6. **Smooth Interpolation**: Client smoothly moves to corrected position

### Position Updates

**Update Frequency:**
- **Client Sends**: Position updates sent at fixed interval (e.g., 20Hz)
- **Server Receives**: Server receives and validates updates
- **Server Responds**: Server sends corrections if needed
- **Client Applies**: Client applies corrections smoothly

**Update Format:**
```rust
pub struct PositionUpdate {
    pub player_id: CharacterId,
    pub position: Position,
    pub rotation: Rotation,
    pub velocity: Vector3,
    pub timestamp: DateTime<Utc>,
    pub movement_state: MovementState,
}
```

### State Synchronization

**Synchronized States:**
- **Position**: X, Y, Z coordinates
- **Rotation**: Character rotation
- **Velocity**: Movement velocity
- **Movement State**: Current movement state (walking, swimming, gliding, etc.)
- **Stamina**: Current stamina level
- **Health**: Current health level

**State Update Frequency:**
- **Position**: Updated frequently (20Hz)
- **Rotation**: Updated with position
- **Velocity**: Updated with position
- **Movement State**: Updated on state change
- **Stamina**: Updated periodically (5Hz)
- **Health**: Updated on change

## No Climbing System

### Design Decision

**No Climbing:**
- The game does **not** include a climbing system
- Players cannot climb walls or vertical surfaces
- Movement is limited to:
  - Ground movement (walking, running)
  - Swimming (water)
  - Glider flight (air)

**Rationale:**
- Simplifies movement system
- Focuses on glider-based mobility
- Reduces complexity in pathfinding
- Maintains consistent movement mechanics

## Client Prediction

### Prediction System

**Client-Side Prediction:**
- Client predicts movement locally
- Immediate response to input
- Server validates and corrects
- Client reconciles differences

**Prediction Flow:**
1. **Input**: Player provides input
2. **Predict**: Client predicts movement
3. **Move**: Character moves immediately
4. **Send**: Send position to server
5. **Receive**: Receive server validation
6. **Reconcile**: Reconcile if correction needed

### Reconciliation

**Reconciliation Process:**
- Client maintains prediction history
- Server sends corrections when needed
- Client smoothly interpolates to corrected position
- Prediction history updated with corrections

## Performance Considerations

### Optimization

**Motion Matching Performance:**
- Thread-safe execution
- Efficient animation blending
- Optimized for multiple characters
- Minimal CPU overhead

**Synchronization Optimization:**
- Batch position updates
- Compress network data
- Delta compression for state updates
- Prioritize critical updates

### Scalability

**Client Performance:**
- Efficient rendering for many characters
- Optimized animation system
- Network update batching
- Memory-efficient state management

## Integration with Server Systems

### Lag Compensation Integration

**Client-Side:**
- Client tracks latency
- Applies client-side prediction
- Receives server corrections
- Smoothly reconciles differences

**See [../server/LAG_COMPENSATION.md](../server/LAG_COMPENSATION.md) for details.**

### Security Integration

**Movement Validation:**
- Client sends position updates
- Server validates with lag compensation
- Server detects speed hacks
- Client receives corrections

**See [../server/SECURITY.md](../server/SECURITY.md) for details.**

### Map Integration

**Position Validation:**
- Client position validated against heightmap
- Water detection for swimming
- Ground detection for glider landing
- Boundary checks for map limits

**See [../server/MAPS.md](../server/MAPS.md) for details.**

## Testing Considerations

### Unit Tests

- Motion Matching state transitions
- Swimming mechanics
- Glider deployment and flight
- Position synchronization
- State synchronization

### Integration Tests

- Client-server position sync
- Lag compensation integration
- Swimming state transitions
- Glider flight and landing
- Movement state changes

### Performance Tests

- Motion Matching performance
- Network update performance
- Animation system performance
- Memory usage with many characters

## Related Documentation

- [INPUT.md](./INPUT.md) - Input system (movement controls, WASD)
- [CAMERA.md](./CAMERA.md) - Camera system (hybrid camera, zoom, rotation)
- [../server/LAG_COMPENSATION.md](../server/LAG_COMPENSATION.md) - Lag compensation system (client-server synchronization)
- [../server/SECURITY.md](../server/SECURITY.md) - Security system (movement validation and speed hack detection)
- [../server/MAPS.md](../server/MAPS.md) - Map system (position validation, water detection, heightmap)
- [ENTITY_SYNC.md](./ENTITY_SYNC.md) - Entity synchronization system
- [SUBSYSTEM.md](./SUBSYSTEM.md) - Client subsystem architecture

## Summary

The player controller system provides:

- **Motion Matching**: High-quality, thread-safe character movement using Unreal Engine's Motion Matching
- **Swimming System**: Complete underwater movement mechanics
- **Glider Flight**: Glider-based flight system for enhanced mobility
- **No Climbing**: Simplified movement system without climbing mechanics
- **Server Synchronization**: Accurate position and state synchronization
- **Client Prediction**: Non-blocking movement with server reconciliation
- **Smooth Gameplay**: Fluid, responsive gameplay experience

The system ensures players experience smooth, natural movement while maintaining accurate synchronization with the server and preventing exploits through proper validation.

