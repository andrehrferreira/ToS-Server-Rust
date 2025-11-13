# Entity Synchronization Architecture

## Overview

The client-server synchronization system is responsible for maintaining consistent entity state between the Rust server and Unreal Engine client. All entities (players, monsters, NPCs, mounts, pets) derive from `Pawn` and `Character` classes in Unreal Engine, sending position updates to the server and receiving authoritative updates from the server. This document describes the synchronization architecture, interpolation system, animation synchronization, and optimization strategies.

## System Requirements

### Functional Requirements

1. **Entity Synchronization**: All entities (players, monsters, NPCs, mounts, pets) must be synchronized
2. **Position Interpolation**: Smooth position interpolation between server updates
3. **Animation Synchronization**: Synchronize animation states between server and client
4. **Quantization**: Use quantized positions to reduce bandwidth
5. **Delta Compression**: Only send changed data to reduce bandwidth
6. **Authoritative Server**: Server is the source of truth for all entity positions

### Performance Requirements

1. **Update Rate**: 60 Hz server tick rate (16.67ms per frame)
2. **Interpolation Smoothness**: Smooth interpolation without jitter
3. **Animation Responsiveness**: Animations must update within 1-2 frames
4. **Bandwidth Efficiency**: Minimize packet size through quantization and delta compression
5. **Latency**: < 50ms (p99) for position updates

## Current Implementation (Unreal Engine Plugin)

### Class Hierarchy

**Unreal Engine Classes:**
```
ACharacter (Unreal base class)
└── ASyncEntity (base sync entity)
    └── ASyncPlayer (player-controlled entity)
    └── ASyncCreature (monster/NPC entity)
    └── ASyncMount (mount entity)
    └── ASyncPet (pet entity)
```

**Key Classes:**
- **ASyncEntity**: Base class for all synchronized entities
- **ASyncPlayer**: Player-controlled entity with input handling
- **ATOSPlayerController**: Player controller that manages entity synchronization
- **UENetSubsystem**: Network subsystem that handles UDP communication

### ASyncEntity Class

**Class Structure:**
```cpp
UCLASS()
class ASyncEntity : public ACharacter
{
    GENERATED_BODY()
    
public:
    // Entity ID
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Entity)
    int32 EntityId;
    
    // Target position and rotation (for interpolation)
    UPROPERTY(BlueprintReadWrite, Category = "Network")
    FVector TargetLocation;
    
    UPROPERTY(BlueprintReadWrite, Category = "Network")
    FRotator TargetRotation;
    
    // Entity flags
    UPROPERTY(BlueprintReadWrite, Category = "Network")
    EEntityState EntityFlags = EEntityState::None;
    
    // Animation state
    int32 AnimationState = 0;
    
    // Local control flag (true for player's own entity)
    bool LocalControl = false;
    
    // Network update methods
    void UpdateAnimationFromNetwork(FVector Velocity, uint32 Animation, bool IsFalling);
    void UpdateFromQuantizedNetwork(int16 QuantizedX, int16 QuantizedY, int16 QuantizedZ,
        int16 QuadrantX, int16 QuadrantY, float Yaw, FVector Velocity, uint32 Animation, bool IsFalling);
    
protected:
    virtual void Tick(float DeltaTime) override;
    UENetSubsystem* NetSubsystem = nullptr;
    FTimerHandle NetSyncTimerHandle;
};
```

### ASyncPlayer Class

**Class Structure:**
```cpp
UCLASS()
class ASyncPlayer : public ASyncEntity
{
    GENERATED_BODY()
    
public:
    // Camera components
    USpringArmComponent* CameraBoom;
    UCameraComponent* FollowCamera;
    
    // Input actions
    UInputMappingContext* DefaultMappingContext;
    UInputAction* JumpAction;
    UInputAction* MoveAction;
    UInputAction* LookAction;
    
    // Network sync
    void SendSyncToServer();
    uint32 LastSyncHash = 0;
    
protected:
    virtual void SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) override;
    void Move(const FInputActionValue& Value);
    void Look(const FInputActionValue& Value);
};
```

### Network Packet Structure

**SyncEntityQuantizedPacket (Client → Server):**
```
- QuantizedX: int16 (2 bytes) - Quantized X position
- QuantizedY: int16 (2 bytes) - Quantized Y position
- QuantizedZ: int16 (2 bytes) - Quantized Z position
- QuadrantX: int16 (2 bytes) - World quadrant X
- QuadrantY: int16 (2 bytes) - World quadrant Y
- Yaw: float (4 bytes) - Rotation Yaw
- Velocity: FVector (12 bytes) - Movement velocity
- AnimationState: uint16 (2 bytes) - Animation ID
- IsFalling: bool (1 byte) - Falling state
Total: 26 bytes
```

**UpdateEntityQuantizedPacket (Server → Client):**
```
- EntityId: uint32 (4 bytes) - Entity ID
- QuantizedX: int16 (2 bytes) - Quantized X position
- QuantizedY: int16 (2 bytes) - Quantized Y position
- QuantizedZ: int16 (2 bytes) - Quantized Z position
- QuadrantX: int16 (2 bytes) - World quadrant X
- QuadrantY: int16 (2 bytes) - World quadrant Y
- Yaw: float (4 bytes) - Rotation Yaw
- Velocity: FVector (12 bytes) - Movement velocity
- AnimationState: uint16 (2 bytes) - Animation ID
- Flags: uint32 (4 bytes) - Entity flags
Total: 33 bytes
```

### Position Interpolation (Current Implementation)

**Current Interpolation (Basic):**
```cpp
void ASyncEntity::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    
    if (!LocalControl) {
        // Adaptive interpolation speed based on distance
        float BaseInterpSpeed = 10.0f;
        float DistanceToTarget = FVector::Distance(GetActorLocation(), TargetLocation);
        float AdaptiveInterpSpeed = FMath::Clamp(
            BaseInterpSpeed * (DistanceToTarget / 100.0f),
            BaseInterpSpeed, 
            BaseInterpSpeed * 3.0f
        );
        
        // Interpolate position
        FVector NewLocation = FMath::VInterpTo(
            GetActorLocation(), 
            TargetLocation, 
            DeltaTime, 
            AdaptiveInterpSpeed
        );
        SetActorLocation(NewLocation);
        
        // Interpolate rotation
        FRotator NewRotation = FMath::RInterpTo(
            GetActorRotation(), 
            TargetRotation, 
            DeltaTime, 
            BaseInterpSpeed
        );
        SetActorRotation(NewRotation);
    }
}
```

**Problems with Current Implementation:**
1. **No Smooth Interpolation**: Uses basic `VInterpTo` without considering network latency
2. **No Prediction**: Does not predict entity movement based on velocity
3. **Jitter**: Entities may jitter when receiving frequent updates
4. **No Extrapolation**: Entities stop moving when no update is received
5. **Animation Desync**: Animations are not properly synchronized with position

### Animation Synchronization (Current Implementation)

**Current Animation Update:**
```cpp
void ASyncEntity::UpdateAnimationFromNetwork(FVector Velocity, uint32 Animation, bool IsFalling)
{
    if (UCharacterMovementComponent* Movement = GetCharacterMovement())
    {
        Movement->Velocity = Velocity;
        Movement->RequestDirectMove(Velocity.GetSafeNormal() * Movement->GetMaxSpeed(), false);
        Movement->SetMovementMode(IsFalling ? MOVE_Falling : MOVE_Walking);
    }
    
    AnimationState = static_cast<int32>(Animation);
    SetSpeed(Velocity.Size());
}
```

**Problems with Current Implementation:**
1. **No Animation Blending**: Animations change abruptly without blending
2. **No Animation Prediction**: Animations don't predict based on velocity
3. **No State Machine**: No proper animation state machine
4. **Desync Issues**: Animations may desync from server state
5. **No Swimming Animation**: Swimming state not properly handled

## Rust Implementation Strategy

### 1. Entity State Management

**Entity State Component:**
```rust
// Entity state component
#[derive(Component, Clone)]
pub struct EntityState {
    pub position: Vector3,
    pub rotation: Quaternion,
    pub velocity: Vector3,
    pub animation_state: AnimationState,
    pub flags: EntityFlags,
    pub is_falling: bool,
    pub is_swimming: bool,
}

// Entity snapshot for client synchronization
pub struct EntitySnapshot {
    pub entity_id: EntityId,
    pub timestamp: u64,              // Server tick timestamp
    pub position: Vector3,
    pub rotation: Quaternion,
    pub velocity: Vector3,
    pub animation_state: AnimationState,
    pub flags: EntityFlags,
    pub is_falling: bool,
    pub is_swimming: bool,
}

// Entity update packet (server → client)
pub struct EntityUpdatePacket {
    pub entity_id: EntityId,
    pub timestamp: u64,
    pub position: QuantizedPosition,
    pub rotation: QuantizedRotation,
    pub velocity: QuantizedVelocity,
    pub animation_state: AnimationState,
    pub flags: EntityFlags,
    pub delta_flags: EntityDeltaFlags,
}

// Entity delta flags (what changed)
#[derive(Clone, Copy)]
pub struct EntityDeltaFlags {
    pub position: bool,
    pub rotation: bool,
    pub velocity: bool,
    pub animation_state: bool,
    pub flags: bool,
}
```

### 2. Quantized Position System

**Quantized Position:**
```rust
// Quantized position (reduced bandwidth)
pub struct QuantizedPosition {
    pub quantized_x: i16,        // Quantized X (-32768 to 32767)
    pub quantized_y: i16,        // Quantized Y (-32768 to 32767)
    pub quantized_z: i16,        // Quantized Z (-32768 to 32767)
    pub quadrant_x: i16,         // World quadrant X
    pub quadrant_y: i16,         // World quadrant Y
}

// Quantization constants
pub const QUANTIZATION_SCALE: f32 = 100.0;      // 1 unit = 1cm
pub const QUADRANT_SIZE: f32 = 25600.0 * 4.0;   // Quadrant size (102,400 units)

impl QuantizedPosition {
    // Quantize world position to quantized position
    pub fn quantize(world_position: Vector3, world_origin: Vector3) -> Self {
        let relative_position = world_position - world_origin;
        
        // Calculate quadrant
        let quadrant_x = (relative_position.x / QUADRANT_SIZE) as i16;
        let quadrant_y = (relative_position.y / QUADRANT_SIZE) as i16;
        
        // Calculate quantized position within quadrant
        let quadrant_offset_x = relative_position.x - (quadrant_x as f32 * QUADRANT_SIZE);
        let quadrant_offset_y = relative_position.y - (quadrant_y as f32 * QUADRANT_SIZE);
        
        let quantized_x = (quadrant_offset_x / QUANTIZATION_SCALE) as i16;
        let quantized_y = (quadrant_offset_y / QUANTIZATION_SCALE) as i16;
        let quantized_z = (relative_position.z / QUANTIZATION_SCALE) as i16;
        
        Self {
            quantized_x,
            quantized_y,
            quantized_z,
            quadrant_x,
            quadrant_y,
        }
    }
    
    // Dequantize quantized position to world position
    pub fn dequantize(&self, world_origin: Vector3) -> Vector3 {
        let quadrant_x = self.quadrant_x as f32 * QUADRANT_SIZE;
        let quadrant_y = self.quadrant_y as f32 * QUADRANT_SIZE;
        
        let world_x = world_origin.x + quadrant_x + (self.quantized_x as f32 * QUANTIZATION_SCALE);
        let world_y = world_origin.y + quadrant_y + (self.quantized_y as f32 * QUANTIZATION_SCALE);
        let world_z = world_origin.z + (self.quantized_z as f32 * QUANTIZATION_SCALE);
        
        Vector3::new(world_x, world_y, world_z)
    }
}
```

### 3. Smooth Position Interpolation

**Client-Side Interpolation:**
```rust
// Client-side interpolation buffer
pub struct InterpolationBuffer {
    snapshots: VecDeque<EntitySnapshot>,
    max_snapshots: usize,
    interpolation_delay: f32,        // Delay for interpolation (e.g., 100ms)
}

impl InterpolationBuffer {
    // Add new snapshot
    pub fn add_snapshot(&mut self, snapshot: EntitySnapshot) {
        self.snapshots.push_back(snapshot);
        
        // Remove old snapshots
        while self.snapshots.len() > self.max_snapshots {
            self.snapshots.pop_front();
        }
    }
    
    // Get interpolated position at current time
    pub fn get_interpolated_position(&self, current_time: f32) -> Option<Vector3> {
        if self.snapshots.len() < 2 {
            return None;
        }
        
        // Calculate target time (current time - interpolation delay)
        let target_time = current_time - self.interpolation_delay;
        
        // Find two snapshots to interpolate between
        let mut older_snapshot = None;
        let mut newer_snapshot = None;
        
        for i in 0..self.snapshots.len() - 1 {
            let snap1 = &self.snapshots[i];
            let snap2 = &self.snapshots[i + 1];
            
            if snap1.timestamp as f32 <= target_time && snap2.timestamp as f32 >= target_time {
                older_snapshot = Some(snap1);
                newer_snapshot = Some(snap2);
                break;
            }
        }
        
        if let (Some(older), Some(newer)) = (older_snapshot, newer_snapshot) {
            // Calculate interpolation factor (0.0 to 1.0)
            let time_diff = (newer.timestamp - older.timestamp) as f32;
            if time_diff > 0.0 {
                let t = (target_time - older.timestamp as f32) / time_diff;
                let t = t.clamp(0.0, 1.0);
                
                // Interpolate position
                let position = older.position.lerp(newer.position, t);
                return Some(position);
            }
        }
        
        // Fallback: Use latest snapshot
        self.snapshots.back().map(|snap| snap.position)
    }
    
    // Get extrapolated position (predict future position)
    pub fn get_extrapolated_position(&self, current_time: f32) -> Option<Vector3> {
        if let Some(latest_snapshot) = self.snapshots.back() {
            // Calculate time since latest snapshot
            let time_since_snapshot = current_time - (latest_snapshot.timestamp as f32);
            
            // Extrapolate based on velocity
            let extrapolated_position = latest_snapshot.position + 
                (latest_snapshot.velocity * time_since_snapshot);
            
            Some(extrapolated_position)
        } else {
            None
        }
    }
}
```

### 4. Client-Side Prediction

**Client-Side Prediction:**
```rust
// Client-side prediction state
pub struct PredictionState {
    pub predicted_position: Vector3,
    pub predicted_rotation: Quaternion,
    pub predicted_velocity: Vector3,
    pub server_position: Vector3,
    pub server_rotation: Quaternion,
    pub server_velocity: Vector3,
    pub last_server_update: f32,
}

impl PredictionState {
    // Update prediction based on input
    pub fn update_prediction(&mut self, input: MovementInput, delta_time: f32) {
        // Apply input to predicted position
        let acceleration = input.direction * input.speed;
        self.predicted_velocity = self.predicted_velocity + (acceleration * delta_time);
        self.predicted_velocity = self.predicted_velocity * (1.0 - input.friction * delta_time);
        self.predicted_position = self.predicted_position + (self.predicted_velocity * delta_time);
        
        // Update rotation
        if input.rotation_change.length() > 0.0 {
            self.predicted_rotation = self.predicted_rotation * 
                Quaternion::from_euler(input.rotation_change * delta_time);
        }
    }
    
    // Reconcile with server state (server reconciliation)
    pub fn reconcile(&mut self, server_snapshot: EntitySnapshot) {
        // Calculate error between predicted and server position
        let position_error = (self.predicted_position - server_snapshot.position).length();
        
        // If error is too large, snap to server position
        if position_error > 1.0 { // 1 unit threshold
            self.predicted_position = server_snapshot.position;
            self.predicted_rotation = server_snapshot.rotation;
            self.predicted_velocity = server_snapshot.velocity;
        } else {
            // Smoothly correct prediction
            let correction_factor = 0.1; // 10% correction per frame
            self.predicted_position = self.predicted_position.lerp(
                server_snapshot.position, 
                correction_factor
            );
            self.predicted_rotation = self.predicted_rotation.slerp(
                server_snapshot.rotation, 
                correction_factor
            );
            self.predicted_velocity = self.predicted_velocity.lerp(
                server_snapshot.velocity, 
                correction_factor
            );
        }
        
        // Update server state
        self.server_position = server_snapshot.position;
        self.server_rotation = server_snapshot.rotation;
        self.server_velocity = server_snapshot.velocity;
        self.last_server_update = server_snapshot.timestamp as f32;
    }
}
```

### 5. Animation Synchronization

**Animation State Management:**
```rust
// Animation state
#[derive(Clone, Copy, PartialEq, Eq)]
pub enum AnimationState {
    Idle = 0,
    Walking = 1,
    Running = 2,
    Jumping = 3,
    Falling = 4,
    Swimming = 5,
    Attacking = 6,
    Casting = 7,
    // ... other states
}

// Animation synchronization component
#[derive(Component)]
pub struct AnimationSync {
    pub current_animation: AnimationState,
    pub target_animation: AnimationState,
    pub animation_blend_time: f32,
    pub animation_blend_timer: f32,
    pub animation_speed: f32,
}

// Animation synchronization system
pub fn animation_sync_system(
    mut animation_syncs: Query<&mut AnimationSync>,
    entity_states: Query<&EntityState>,
    time: Res<Time>,
) {
    for (mut animation_sync, entity_state) in animation_syncs.iter_mut().zip(entity_states.iter()) {
        // Determine target animation based on entity state
        let target_animation = if entity_state.is_swimming {
            AnimationState::Swimming
        } else if entity_state.is_falling {
            AnimationState::Falling
        } else if entity_state.velocity.length() > 500.0 {
            AnimationState::Running
        } else if entity_state.velocity.length() > 100.0 {
            AnimationState::Walking
        } else {
            AnimationState::Idle
        };
        
        // Update target animation
        if target_animation != animation_sync.target_animation {
            animation_sync.target_animation = target_animation;
            animation_sync.animation_blend_timer = 0.0;
        }
        
        // Blend animation
        if animation_sync.current_animation != animation_sync.target_animation {
            animation_sync.animation_blend_timer += time.delta_seconds();
            
            if animation_sync.animation_blend_timer >= animation_sync.animation_blend_time {
                // Animation blend complete
                animation_sync.current_animation = animation_sync.target_animation;
            } else {
                // Animation blending in progress
                let blend_factor = animation_sync.animation_blend_timer / animation_sync.animation_blend_time;
                // Send blend factor to animation system
            }
        }
        
        // Update animation speed based on velocity
        animation_sync.animation_speed = entity_state.velocity.length() / 500.0; // Normalize to run speed
    }
}
```

### 6. Unreal Engine Plugin Integration

**C++ Plugin Structure:**
```cpp
// Entity synchronization manager (Unreal C++)
UCLASS()
class UEntitySyncManager : public UObject
{
    GENERATED_BODY()
    
public:
    // Initialize synchronization
    UFUNCTION(BlueprintCallable, Category = "Network")
    void InitializeSync(ASyncEntity* Entity, int32 EntityId, bool IsLocalControl);
    
    // Update entity from server snapshot
    UFUNCTION(BlueprintCallable, Category = "Network")
    void UpdateFromServerSnapshot(const FEntitySnapshot& Snapshot);
    
    // Get interpolated position
    UFUNCTION(BlueprintCallable, Category = "Network")
    FVector GetInterpolatedPosition(float CurrentTime) const;
    
    // Update animation state
    UFUNCTION(BlueprintCallable, Category = "Network")
    void UpdateAnimationState(AnimationState NewState, float BlendTime);
    
private:
    // Interpolation buffer
    TArray<FEntitySnapshot> SnapshotBuffer;
    float InterpolationDelay = 0.1f; // 100ms delay
    
    // Animation sync
    AnimationState CurrentAnimation;
    AnimationState TargetAnimation;
    float AnimationBlendTime;
    float AnimationBlendTimer;
    
    // Entity reference
    TWeakObjectPtr<ASyncEntity> EntityRef;
};
```

**Entity Update Implementation:**
```cpp
void UEntitySyncManager::UpdateFromServerSnapshot(const FEntitySnapshot& Snapshot)
{
    // Add snapshot to buffer
    SnapshotBuffer.Add(Snapshot);
    
    // Remove old snapshots (keep last 10)
    if (SnapshotBuffer.Num() > 10)
    {
        SnapshotBuffer.RemoveAt(0);
    }
    
    // Update target animation
    UpdateAnimationState(Snapshot.AnimationState, 0.2f);
    
    // Update entity flags
    if (ASyncEntity* Entity = EntityRef.Get())
    {
        Entity->SetFlags(Snapshot.Flags);
    }
}

FVector UEntitySyncManager::GetInterpolatedPosition(float CurrentTime) const
{
    if (SnapshotBuffer.Num() < 2)
    {
        // Not enough snapshots, use latest or extrapolate
        if (SnapshotBuffer.Num() > 0)
        {
            const FEntitySnapshot& Latest = SnapshotBuffer[SnapshotBuffer.Num() - 1];
            float TimeSinceSnapshot = CurrentTime - Latest.Timestamp;
            
            // Extrapolate based on velocity
            return Latest.Position + (Latest.Velocity * TimeSinceSnapshot);
        }
        return FVector::ZeroVector;
    }
    
    // Calculate target time (current time - interpolation delay)
    float TargetTime = CurrentTime - InterpolationDelay;
    
    // Find two snapshots to interpolate between
    for (int32 i = 0; i < SnapshotBuffer.Num() - 1; i++)
    {
        const FEntitySnapshot& Snapshot1 = SnapshotBuffer[i];
        const FEntitySnapshot& Snapshot2 = SnapshotBuffer[i + 1];
        
        if (Snapshot1.Timestamp <= TargetTime && Snapshot2.Timestamp >= TargetTime)
        {
            // Calculate interpolation factor
            float TimeDiff = Snapshot2.Timestamp - Snapshot1.Timestamp;
            if (TimeDiff > 0.0f)
            {
                float t = (TargetTime - Snapshot1.Timestamp) / TimeDiff;
                t = FMath::Clamp(t, 0.0f, 1.0f);
                
                // Interpolate position
                return FMath::Lerp(Snapshot1.Position, Snapshot2.Position, t);
            }
        }
    }
    
    // Fallback: Use latest snapshot
    return SnapshotBuffer[SnapshotBuffer.Num() - 1].Position;
}

void UEntitySyncManager::UpdateAnimationState(AnimationState NewState, float BlendTime)
{
    if (NewState != TargetAnimation)
    {
        TargetAnimation = NewState;
        AnimationBlendTimer = 0.0f;
        AnimationBlendTime = BlendTime;
    }
}
```

### 7. Entity Tick Update

**Entity Tick with Interpolation:**
```cpp
void ASyncEntity::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    
    if (!LocalControl)
    {
        // Get current time
        float CurrentTime = GetWorld()->GetTimeSeconds();
        
        // Get interpolated position from sync manager
        if (SyncManager)
        {
            FVector InterpolatedPosition = SyncManager->GetInterpolatedPosition(CurrentTime);
            
            // Smoothly move to interpolated position
            FVector CurrentLocation = GetActorLocation();
            FVector NewLocation = FMath::VInterpTo(
                CurrentLocation,
                InterpolatedPosition,
                DeltaTime,
                InterpolationSpeed
            );
            SetActorLocation(NewLocation);
            
            // Update rotation
            FRotator TargetRotation = SyncManager->GetTargetRotation();
            FRotator NewRotation = FMath::RInterpTo(
                GetActorRotation(),
                TargetRotation,
                DeltaTime,
                RotationInterpSpeed
            );
            SetActorRotation(NewRotation);
            
            // Update animation
            SyncManager->UpdateAnimation(DeltaTime);
        }
    }
    else
    {
        // Local control: Send position to server
        // (handled by ASyncPlayer::SendSyncToServer)
    }
}
```

### 8. Animation State Machine

**Animation State Machine:**
```cpp
// Animation state machine for entity
UCLASS()
class UEntityAnimationStateMachine : public UObject
{
    GENERATED_BODY()
    
public:
    // Animation states
    UENUM(BlueprintType)
    enum class EAnimationState
    {
        Idle,
        Walking,
        Running,
        Jumping,
        Falling,
        Swimming,
        Attacking,
        Casting,
    };
    
    // Update animation state
    UFUNCTION(BlueprintCallable, Category = "Animation")
    void UpdateAnimationState(EAnimationState NewState, float BlendTime = 0.2f);
    
    // Get current animation
    UFUNCTION(BlueprintCallable, Category = "Animation")
    EAnimationState GetCurrentAnimation() const { return CurrentAnimation; }
    
private:
    EAnimationState CurrentAnimation = EAnimationState::Idle;
    EAnimationState TargetAnimation = EAnimationState::Idle;
    float AnimationBlendTime = 0.2f;
    float AnimationBlendTimer = 0.0f;
    
    // Animation montages
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Animation")
    TMap<EAnimationState, UAnimMontage*> AnimationMontages;
    
    // Animation blend spaces
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Animation")
    TMap<EAnimationState, UBlendSpace*> AnimationBlendSpaces;
};

void UEntityAnimationStateMachine::UpdateAnimationState(EAnimationState NewState, float BlendTime)
{
    if (NewState != TargetAnimation)
    {
        TargetAnimation = NewState;
        AnimationBlendTimer = 0.0f;
        AnimationBlendTime = BlendTime;
        
        // Play animation montage
        if (UAnimMontage* Montage = AnimationMontages.FindRef(NewState))
        {
            if (ACharacter* Character = Cast<ACharacter>(GetOuter()))
            {
                if (UAnimInstance* AnimInstance = Character->GetMesh()->GetAnimInstance())
                {
                    // Blend animation
                    if (AnimationBlendTime > 0.0f)
                    {
                        AnimInstance->Montage_PlayWithBlendIn(
                            Montage,
                            BlendTime,
                            EMontagePlayReturnType::MontageLength,
                            0.0f,
                            false
                        );
                    }
                    else
                    {
                        AnimInstance->Montage_Play(Montage);
                    }
                }
            }
        }
    }
}
```

### 9. Delta Compression

**Delta Compression System:**
```rust
// Delta compression for entity updates
pub struct EntityDeltaCompressor {
    previous_states: HashMap<EntityId, EntitySnapshot>,
}

impl EntityDeltaCompressor {
    // Calculate delta between current and previous state
    pub fn calculate_delta(
        &mut self,
        entity_id: EntityId,
        current_snapshot: EntitySnapshot,
    ) -> EntityDelta {
        if let Some(previous_snapshot) = self.previous_states.get(&entity_id) {
            // Calculate delta flags
            let mut delta_flags = EntityDeltaFlags::default();
            let mut delta = EntityDelta::default();
            
            // Position delta
            let position_diff = (current_snapshot.position - previous_snapshot.position).length();
            if position_diff > 0.01 {
                delta_flags.position = true;
                delta.position = Some(current_snapshot.position - previous_snapshot.position);
            }
            
            // Rotation delta
            let rotation_diff = current_snapshot.rotation.angle_to(previous_snapshot.rotation);
            if rotation_diff > 0.01 {
                delta_flags.rotation = true;
                delta.rotation = Some(current_snapshot.rotation * previous_snapshot.rotation.inverse());
            }
            
            // Velocity delta
            let velocity_diff = (current_snapshot.velocity - previous_snapshot.velocity).length();
            if velocity_diff > 0.1 {
                delta_flags.velocity = true;
                delta.velocity = Some(current_snapshot.velocity - previous_snapshot.velocity);
            }
            
            // Animation state delta
            if current_snapshot.animation_state != previous_snapshot.animation_state {
                delta_flags.animation_state = true;
                delta.animation_state = Some(current_snapshot.animation_state);
            }
            
            // Flags delta
            if current_snapshot.flags != previous_snapshot.flags {
                delta_flags.flags = true;
                delta.flags = Some(current_snapshot.flags);
            }
            
            delta.delta_flags = delta_flags;
            delta
        } else {
            // First snapshot, send full update
            EntityDelta {
                delta_flags: EntityDeltaFlags::all(),
                position: Some(current_snapshot.position),
                rotation: Some(current_snapshot.rotation),
                velocity: Some(current_snapshot.velocity),
                animation_state: Some(current_snapshot.animation_state),
                flags: Some(current_snapshot.flags),
            }
        }
    }
    
    // Update previous state
    pub fn update_previous_state(&mut self, entity_id: EntityId, snapshot: EntitySnapshot) {
        self.previous_states.insert(entity_id, snapshot);
    }
}
```

### 10. Network Replication System

**Network Replication:**
```rust
// Network replication system
pub struct NetworkReplicationSystem {
    delta_compressor: EntityDeltaCompressor,
    send_buffer: Vec<EntityUpdatePacket>,
    send_rate: f32,                  // Updates per second (e.g., 60 Hz)
    last_send_time: f32,
}

impl NetworkReplicationSystem {
    // Update replication system
    pub fn update(&mut self, entities: &Query<&EntityState>, current_time: f32, delta_time: f32) {
        // Check if it's time to send updates
        let time_since_last_send = current_time - self.last_send_time;
        let send_interval = 1.0 / self.send_rate;
        
        if time_since_last_send >= send_interval {
            // Send updates for all entities
            for (entity_id, entity_state) in entities.iter() {
                let snapshot = EntitySnapshot {
                    entity_id,
                    timestamp: (current_time * 1000.0) as u64, // Convert to milliseconds
                    position: entity_state.position,
                    rotation: entity_state.rotation,
                    velocity: entity_state.velocity,
                    animation_state: entity_state.animation_state,
                    flags: entity_state.flags,
                    is_falling: entity_state.is_falling,
                    is_swimming: entity_state.is_swimming,
                };
                
                // Calculate delta
                let delta = self.delta_compressor.calculate_delta(entity_id, snapshot.clone());
                
                // Only send if something changed
                if delta.delta_flags.any() {
                    // Create update packet
                    let packet = EntityUpdatePacket {
                        entity_id,
                        timestamp: snapshot.timestamp,
                        position: QuantizedPosition::quantize(snapshot.position, world_origin),
                        rotation: QuantizedRotation::quantize(snapshot.rotation),
                        velocity: QuantizedVelocity::quantize(snapshot.velocity),
                        animation_state: snapshot.animation_state,
                        flags: snapshot.flags,
                        delta_flags: delta.delta_flags,
                    };
                    
                    self.send_buffer.push(packet);
                }
                
                // Update previous state
                self.delta_compressor.update_previous_state(entity_id, snapshot);
            }
            
            // Send buffered packets
            self.send_packets();
            
            // Reset send time
            self.last_send_time = current_time;
        }
    }
    
    // Send packets to clients
    fn send_packets(&mut self) {
        // Group packets by client (AOI)
        // Send to each client in AOI
        // Clear send buffer
        self.send_buffer.clear();
    }
}
```

## Unreal Engine Integration

### 1. Plugin Structure

**Plugin Directory Structure:**
```
Unreal/Plugins/ToS_Network/
├── Source/
│   ├── ToS_Network/
│   │   ├── Public/
│   │   │   ├── Entities/
│   │   │   │   ├── SyncEntity.h
│   │   │   │   ├── SyncPlayer.h
│   │   │   │   ├── SyncCreature.h
│   │   │   │   ├── SyncMount.h
│   │   │   │   └── SyncPet.h
│   │   │   ├── Network/
│   │   │   │   ├── ENetSubsystem.h
│   │   │   │   ├── UDPClient.h
│   │   │   │   └── UFlatBuffer.h
│   │   │   ├── Controllers/
│   │   │   │   └── TOSPlayerController.h
│   │   │   └── Packets/
│   │   │       ├── UpdateEntityQuantizedPacket.h
│   │   │       └── SyncEntityQuantizedPacket.h
│   │   └── Private/
│   │       └── ... (implementation files)
│   └── ToS_Network.Build.cs
└── ToS_Network.uplugin
```

### 2. Entity Classes

**SyncEntity Base Class:**
```cpp
// SyncEntity.h
UCLASS()
class ASyncEntity : public ACharacter
{
    GENERATED_BODY()
    
public:
    ASyncEntity();
    
    // Entity properties
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Entity)
    int32 EntityId;
    
    // Synchronization
    UPROPERTY(BlueprintReadWrite, Category = "Network")
    UEntitySyncManager* SyncManager;
    
    // Network update
    UFUNCTION(BlueprintCallable, Category = "Network")
    void UpdateFromServerSnapshot(const FEntitySnapshot& Snapshot);
    
protected:
    virtual void Tick(float DeltaTime) override;
    virtual void BeginPlay() override;
    
    // Interpolation
    float InterpolationSpeed = 10.0f;
    float RotationInterpSpeed = 10.0f;
    
    // Network subsystem
    UPROPERTY()
    UENetSubsystem* NetSubsystem = nullptr;
};
```

**SyncPlayer Class:**
```cpp
// SyncPlayer.h
UCLASS()
class ASyncPlayer : public ASyncEntity
{
    GENERATED_BODY()
    
public:
    ASyncPlayer();
    
    // Input handling
    virtual void SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) override;
    void Move(const FInputActionValue& Value);
    void Look(const FInputActionValue& Value);
    
    // Network sync
    void SendSyncToServer();
    
protected:
    virtual void BeginPlay() override;
    
    // Camera
    USpringArmComponent* CameraBoom;
    UCameraComponent* FollowCamera;
    
    // Input
    UInputMappingContext* DefaultMappingContext;
    UInputAction* JumpAction;
    UInputAction* MoveAction;
    UInputAction* LookAction;
    
    // Sync timer
    FTimerHandle NetSyncTimerHandle;
    float SyncInterval = 0.1f; // 10 Hz client send rate
};
```

### 3. Smooth Interpolation Implementation

**Improved Interpolation:**
```cpp
// EntitySyncManager.cpp
void UEntitySyncManager::UpdateInterpolation(float DeltaTime)
{
    if (SnapshotBuffer.Num() < 2)
    {
        return;
    }
    
    // Get current time
    float CurrentTime = GetWorld()->GetTimeSeconds();
    float TargetTime = CurrentTime - InterpolationDelay;
    
    // Find snapshots to interpolate between
    FEntitySnapshot* OlderSnapshot = nullptr;
    FEntitySnapshot* NewerSnapshot = nullptr;
    
    for (int32 i = 0; i < SnapshotBuffer.Num() - 1; i++)
    {
        FEntitySnapshot& Snap1 = SnapshotBuffer[i];
        FEntitySnapshot& Snap2 = SnapshotBuffer[i + 1];
        
        if (Snap1.Timestamp <= TargetTime && Snap2.Timestamp >= TargetTime)
        {
            OlderSnapshot = &Snap1;
            NewerSnapshot = &Snap2;
            break;
        }
    }
    
    if (OlderSnapshot && NewerSnapshot)
    {
        // Calculate interpolation factor
        float TimeDiff = NewerSnapshot->Timestamp - OlderSnapshot->Timestamp;
        if (TimeDiff > 0.0f)
        {
            float t = (TargetTime - OlderSnapshot->Timestamp) / TimeDiff;
            t = FMath::Clamp(t, 0.0f, 1.0f);
            
            // Interpolate position (with easing)
            FVector InterpolatedPosition = FMath::Lerp(
                OlderSnapshot->Position,
                NewerSnapshot->Position,
                FMath::SmoothStep(0.0f, 1.0f, t) // Smooth easing
            );
            
            // Interpolate rotation (spherical linear interpolation)
            FQuat InterpolatedRotation = FQuat::Slerp(
                OlderSnapshot->Rotation.Quaternion(),
                NewerSnapshot->Rotation.Quaternion(),
                t
            );
            
            // Update entity
            if (ASyncEntity* Entity = EntityRef.Get())
            {
                Entity->SetActorLocation(InterpolatedPosition);
                Entity->SetActorRotation(InterpolatedRotation.Rotator());
            }
        }
    }
    else if (NewerSnapshot)
    {
        // Extrapolate based on velocity
        float TimeSinceSnapshot = CurrentTime - NewerSnapshot->Timestamp;
        FVector ExtrapolatedPosition = NewerSnapshot->Position + 
            (NewerSnapshot->Velocity * TimeSinceSnapshot);
        
        if (ASyncEntity* Entity = EntityRef.Get())
        {
            Entity->SetActorLocation(ExtrapolatedPosition);
        }
    }
}
```

### 4. Animation Synchronization Implementation

**Animation Synchronization:**
```cpp
// EntitySyncManager.cpp
void UEntitySyncManager::UpdateAnimation(float DeltaTime)
{
    if (CurrentAnimation != TargetAnimation)
    {
        AnimationBlendTimer += DeltaTime;
        
        if (AnimationBlendTimer >= AnimationBlendTime)
        {
            // Animation blend complete
            CurrentAnimation = TargetAnimation;
            PlayAnimation(TargetAnimation);
        }
        else
        {
            // Animation blending in progress
            float BlendFactor = AnimationBlendTimer / AnimationBlendTime;
            BlendAnimations(CurrentAnimation, TargetAnimation, BlendFactor);
        }
    }
    
    // Update animation speed based on velocity
    if (ASyncEntity* Entity = EntityRef.Get())
    {
        if (UCharacterMovementComponent* Movement = Entity->GetCharacterMovement())
        {
            float Speed = Movement->Velocity.Size();
            float NormalizedSpeed = Speed / Movement->GetMaxSpeed();
            
            // Update animation playback rate
            if (UAnimInstance* AnimInstance = Entity->GetMesh()->GetAnimInstance())
            {
                AnimInstance->SetPlayRate(NormalizedSpeed);
            }
        }
    }
}

void UEntitySyncManager::PlayAnimation(AnimationState State)
{
    if (ASyncEntity* Entity = EntityRef.Get())
    {
        if (UAnimInstance* AnimInstance = Entity->GetMesh()->GetAnimInstance())
        {
            // Get animation montage for state
            UAnimMontage* Montage = GetAnimationMontage(State);
            if (Montage)
            {
                AnimInstance->Montage_Play(Montage);
            }
        }
    }
}

void UEntitySyncManager::BlendAnimations(AnimationState From, AnimationState To, float BlendFactor)
{
    if (ASyncEntity* Entity = EntityRef.Get())
    {
        if (UAnimInstance* AnimInstance = Entity->GetMesh()->GetAnimInstance())
        {
            // Get animation montages
            UAnimMontage* FromMontage = GetAnimationMontage(From);
            UAnimMontage* ToMontage = GetAnimationMontage(To);
            
            if (FromMontage && ToMontage)
            {
                // Blend animations
                AnimInstance->Montage_PlayWithBlendIn(
                    ToMontage,
                    AnimationBlendTime,
                    EMontagePlayReturnType::MontageLength,
                    0.0f,
                    false
                );
            }
        }
    }
}
```

## Rust Server Implementation

### 1. Entity Update System

**Entity Update System:**
```rust
// Entity update system
pub fn entity_update_system(
    mut entities: Query<(&mut EntityState, &Position, &Rotation, &Velocity)>,
    time: Res<Time>,
) {
    for (mut entity_state, position, rotation, velocity) in entities.iter_mut() {
        // Update entity state
        entity_state.position = position.0;
        entity_state.rotation = rotation.0;
        entity_state.velocity = velocity.0;
        
        // Update animation state based on velocity
        entity_state.animation_state = if entity_state.is_swimming {
            AnimationState::Swimming
        } else if entity_state.is_falling {
            AnimationState::Falling
        } else if velocity.0.length() > 500.0 {
            AnimationState::Running
        } else if velocity.0.length() > 100.0 {
            AnimationState::Walking
        } else {
            AnimationState::Idle
        };
    }
}
```

### 2. Network Replication System

**Network Replication:**
```rust
// Network replication system
pub struct NetworkReplicationSystem {
    delta_compressor: EntityDeltaCompressor,
    replication_rate: f32,           // Updates per second (e.g., 60 Hz)
    last_replication_time: f32,
}

impl NetworkReplicationSystem {
    // Replicate entities to clients
    pub fn replicate_entities(
        &mut self,
        entities: &Query<&EntityState>,
        aoi_system: &AOISystem,
        network_manager: &mut NetworkManager,
        current_time: f32,
    ) {
        // Check if it's time to replicate
        let time_since_last_replication = current_time - self.last_replication_time;
        let replication_interval = 1.0 / self.replication_rate;
        
        if time_since_last_replication >= replication_interval {
            // Replicate entities to clients in AOI
            for (entity_id, entity_state) in entities.iter() {
                // Get clients in AOI
                let clients_in_aoi = aoi_system.get_clients_in_aoi(entity_id);
                
                // Create snapshot
                let snapshot = EntitySnapshot {
                    entity_id,
                    timestamp: (current_time * 1000.0) as u64,
                    position: entity_state.position,
                    rotation: entity_state.rotation,
                    velocity: entity_state.velocity,
                    animation_state: entity_state.animation_state,
                    flags: entity_state.flags,
                    is_falling: entity_state.is_falling,
                    is_swimming: entity_state.is_swimming,
                };
                
                // Calculate delta
                let delta = self.delta_compressor.calculate_delta(entity_id, snapshot.clone());
                
                // Only send if something changed
                if delta.delta_flags.any() {
                    // Create update packet
                    let packet = EntityUpdatePacket {
                        entity_id,
                        timestamp: snapshot.timestamp,
                        position: QuantizedPosition::quantize(snapshot.position, world_origin),
                        rotation: QuantizedRotation::quantize(snapshot.rotation),
                        velocity: QuantizedVelocity::quantize(snapshot.velocity),
                        animation_state: snapshot.animation_state,
                        flags: snapshot.flags,
                        delta_flags: delta.delta_flags,
                    };
                    
                    // Send to clients in AOI
                    for client_id in clients_in_aoi {
                        network_manager.send_packet(client_id, &packet);
                    }
                }
                
                // Update previous state
                self.delta_compressor.update_previous_state(entity_id, snapshot);
            }
            
            // Update replication time
            self.last_replication_time = current_time;
        }
    }
}
```

## Performance Optimization

### 1. Adaptive Update Rate

**Adaptive Update Rate:**
```rust
// Adaptive update rate based on distance
pub struct AdaptiveUpdateRate {
    base_update_rate: f32,           // Base update rate (e.g., 60 Hz)
    distance_thresholds: Vec<f32>,   // Distance thresholds
    update_rate_multipliers: Vec<f32>, // Update rate multipliers
}

impl AdaptiveUpdateRate {
    // Get update rate based on distance
    pub fn get_update_rate(&self, distance: f32) -> f32 {
        for (i, threshold) in self.distance_thresholds.iter().enumerate() {
            if distance <= *threshold {
                return self.base_update_rate * self.update_rate_multipliers[i];
            }
        }
        
        // Default: Lowest update rate
        self.base_update_rate * self.update_rate_multipliers.last().unwrap_or(&1.0)
    }
}
```

### 2. Priority-Based Updates

**Priority-Based Updates:**
```rust
// Entity update priority
pub enum EntityUpdatePriority {
    High,    // Player, important NPCs (60 Hz)
    Medium,  // Monsters, NPCs (30 Hz)
    Low,     // Distant entities (10 Hz)
}

// Priority-based update system
pub fn priority_update_system(
    entities: &Query<&EntityState>,
    priorities: &Query<&EntityUpdatePriority>,
    aoi_system: &AOISystem,
) {
    for (entity_id, priority) in entities.iter().zip(priorities.iter()) {
        // Get update rate based on priority
        let update_rate = match priority {
            EntityUpdatePriority::High => 60.0,
            EntityUpdatePriority::Medium => 30.0,
            EntityUpdatePriority::Low => 10.0,
        };
        
        // Update entity based on priority
        // ...
    }
}
```

### 3. Packet Batching

**Packet Batching:**
```rust
// Packet batching system
pub struct PacketBatcher {
    batch_size: usize,
    batch_timeout: f32,
    batches: HashMap<ClientId, Vec<EntityUpdatePacket>>,
    batch_timers: HashMap<ClientId, f32>,
}

impl PacketBatcher {
    // Add packet to batch
    pub fn add_packet(&mut self, client_id: ClientId, packet: EntityUpdatePacket) {
        let batch = self.batches.entry(client_id).or_insert_with(Vec::new);
        batch.push(packet);
        
        // Send batch if full
        if batch.len() >= self.batch_size {
            self.send_batch(client_id);
        }
    }
    
    // Send batch
    fn send_batch(&mut self, client_id: ClientId) {
        if let Some(batch) = self.batches.remove(&client_id) {
            // Create batched packet
            let batched_packet = BatchedEntityUpdatePacket {
                packets: batch,
            };
            
            // Send batched packet
            network_manager.send_packet(client_id, &batched_packet);
        }
    }
}
```

## Migration Strategy

### Phase 1: Core Synchronization
1. Implement entity synchronization system
2. Implement quantized position system
3. Implement basic interpolation
4. Implement delta compression

### Phase 2: Smooth Interpolation
1. Implement interpolation buffer
2. Implement smooth interpolation with easing
3. Implement extrapolation for missing updates
4. Implement client-side prediction

### Phase 3: Animation Synchronization
1. Implement animation state machine
2. Implement animation blending
3. Implement animation speed synchronization
4. Implement swimming animation support

### Phase 4: Optimization
1. Implement adaptive update rate
2. Implement priority-based updates
3. Implement packet batching
4. Profile and optimize hot paths

## Performance Targets

### Interpolation
- **Interpolation Smoothness**: No visible jitter
- **Interpolation Delay**: < 100ms
- **Extrapolation Accuracy**: < 0.1 units error

### Animation Synchronization
- **Animation Blending Time**: < 200ms
- **Animation Update Latency**: < 50ms
- **Animation State Sync**: 100% accuracy

### Network Bandwidth
- **Packet Size**: < 50 bytes per entity update
- **Update Rate**: 60 Hz for high priority, 30 Hz for medium, 10 Hz for low
- **Bandwidth Usage**: < 100 KB/s per client

## Conclusion

The entity synchronization system is critical for maintaining consistent entity state between the Rust server and Unreal Engine client. The Rust implementation should use:

- **Quantized Position System**: Reduce bandwidth through quantization
- **Smooth Interpolation**: Buffer-based interpolation with extrapolation
- **Animation Synchronization**: State machine with blending
- **Delta Compression**: Only send changed data
- **Adaptive Update Rate**: Optimize bandwidth based on distance and priority
- **Packet Batching**: Reduce packet overhead

This approach provides smooth, responsive entity synchronization while minimizing bandwidth usage and maintaining server authority.

