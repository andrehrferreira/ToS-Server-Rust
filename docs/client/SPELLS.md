# Client-Side Spells System (GAS)

## Overview

The client-side spells system uses Unreal Engine's **Gameplay Ability System (GAS)** to handle all visual and audio aspects of spells. Every spell includes particles, animations, sounds, and various other visual effects that provide rich feedback to players.

## Architecture

### Server-Controlled Spell Execution

**Important:** The server has full authority to trigger any spell for any entity (player or creature), whether:
- **Player-initiated**: Via actionbar input
- **AI-initiated**: Via server-side AI control

The client must be able to execute spells for any entity type without distinction, using a generic spell execution system.

### Gameplay Ability System Integration

The client uses GAS to manage spell execution, visual effects, and interactions:

**Key Components:**
1. **Gameplay Abilities** - Spell definitions and execution logic
2. **Gameplay Effects** - Visual/audio effects and conditions
3. **Ability Tasks** - Async operations (projectiles, channels, etc.)
4. **Attribute Sets** - Spell-related attributes (cooldowns, costs, etc.)
5. **Ability System Component** - Manages abilities on entities
6. **Cast Points** - Invisible adjustable points on entities for projectile spawn
7. **Animation Notifies** - Triggers in animations for precise spell execution timing
8. **Summon System** - Temporary minion creation and management

---

## Spell Visual Components

Every spell includes multiple visual and audio components:

### 1. Particles

**Particle Systems:**
- Cast particles (emitted during cast time)
- Projectile particles (for projectile spells)
- Impact particles (on hit/arrival)
- Area particles (for area effects)
- Trail particles (for projectiles)
- Condition particles (burn, poison, etc.)

**Particle Types by Spell Type:**

**Projectiles:**
- Projectile mesh/particle
- Trail effect
- Impact explosion
- Elemental effects (fire, ice, lightning, etc.)

**Target Area:**
- Area indicator (preview)
- Activation particles
- Persistent area particles
- Tick particles (for DoT effects)

**Camera Direction:**
- Directional beam/line
- Wall particles (ice wall, fire wall)
- Environmental effects

**Single Target:**
- Cast particles
- Travel particles (if applicable)
- Impact particles on target
- Condition particles

**Target Self:**
- Self-cast particles
- Buff/aura particles
- Healing particles

**Summon:**
- Summon portal/rift particles
- Minion spawn effects
- Minion visual effects (animated entities)
- Minion despawn effects

**Summon:**
- Summon portal/rift particles
- Minion spawn effects
- Minion visual effects (animated entities)
- Minion despawn effects

**Area (Centered):**
- Caster-centered particles
- Expanding area particles
- Environmental effects (blizzard, earthquake)

**Summon:**
- Summon spawn particles
- Minion appearance effects
- Minion persistent particles
- Despawn particles

### 2. Animations

**Animation Types:**
- Cast animations (pre-cast, cast, post-cast)
- Projectile animations (spawn, travel, impact)
- Impact animations (on target)
- Caster animations (movement, gestures)
- Environmental animations (walls, areas)

**Animation Montages:**
- Cast montage (pre-cast + cast)
- Impact montage (on hit)
- Channel montage (for channeled spells)
- Self-cast montage (for target self)

**Animation Notifies:**
- **Spell Fire Notify**: Triggers projectile/spell creation at precise animation moment
- **Spell Impact Notify**: Triggers impact effects at specific animation frame
- **Spell End Notify**: Signals spell completion
- Used for precise timing control independent of cast time

### 3. Sounds

**Sound Types:**
- Cast sounds (start of cast)
- Channel sounds (during channel)
- Projectile sounds (travel)
- Impact sounds (on hit)
- Area sounds (activation, persistent)
- Condition sounds (burn, freeze, etc.)

**Sound Categories:**
- Spell cast (whoosh, chant, etc.)
- Elemental sounds (fire crackle, ice shatter, lightning zap)
- Impact sounds (hit, explosion, etc.)
- Environmental sounds (blizzard wind, earthquake rumble)

### 4. Visual Effects

**Additional Visual Elements:**
- Screen effects (shakes, flashes)
- Post-process effects (color grading, blur)
- Decals (impact marks, area markers)
- Mesh effects (spell meshes, projectiles)
- UI indicators (cast bars, cooldowns)

---

## Cast Points System

### Overview

Every entity (player, creature, NPC) must have **Cast Points** - invisible, manually adjustable points that define where projectiles and spell effects originate from. This allows precise control over spell spawn locations regardless of entity type or skeleton structure.

### Cast Point Component

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class ATOS_API UCastPointComponent : public USceneComponent
{
    GENERATED_BODY()

public:
    UCastPointComponent();

    // Cast point types
    UENUM(BlueprintType)
    enum class ECastPointType : uint8
    {
        HandRight,      // Right hand cast point
        HandLeft,       // Left hand cast point
        Chest,          // Chest cast point (for area spells)
        Head,           // Head cast point (for some spells)
        Weapon,         // Weapon cast point (if weapon is equipped)
        Custom          // Custom cast point
    };

    // Get cast point location
    UFUNCTION(BlueprintCallable, Category = "Cast Point")
    FVector GetCastPointLocation(ECastPointType CastPointType) const;

    // Get cast point transform
    UFUNCTION(BlueprintCallable, Category = "Cast Point")
    FTransform GetCastPointTransform(ECastPointType CastPointType) const;

    // Set cast point offset (for manual adjustment)
    UFUNCTION(BlueprintCallable, Category = "Cast Point")
    void SetCastPointOffset(ECastPointType CastPointType, const FVector& Offset);

protected:
    // Cast point socket names (attached to skeleton)
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Cast Points")
    TMap<ECastPointType, FName> CastPointSocketNames;

    // Manual offsets for fine-tuning
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Cast Points")
    TMap<ECastPointType, FVector> CastPointOffsets;

    // Visual debug spheres (editor only)
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Cast Points|Debug")
    bool bShowCastPoints = false;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Cast Points|Debug")
    float CastPointSphereRadius = 5.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Cast Points|Debug")
    FColor CastPointSphereColor = FColor::Red;

    virtual void BeginPlay() override;
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

#if WITH_EDITOR
    virtual void PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent) override;
#endif
};
```

### Cast Point Integration in Entity

```cpp
// In ASyncEntity or base entity class
UCLASS()
class ATOS_API ASyncEntity : public ACharacter
{
    GENERATED_BODY()

public:
    // Cast point component
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UCastPointComponent* CastPointComponent;

    // Get cast point location (convenience function)
    UFUNCTION(BlueprintCallable, Category = "Spell")
    FVector GetCastPointLocation(UCastPointComponent::ECastPointType CastPointType) const
    {
        if (CastPointComponent)
        {
            return CastPointComponent->GetCastPointLocation(CastPointType);
        }
        return GetActorLocation();
    }

protected:
    virtual void BeginPlay() override
    {
        Super::BeginPlay();
        
        // Create cast point component
        CastPointComponent = CreateDefaultSubobject<UCastPointComponent>(TEXT("CastPointComponent"));
        CastPointComponent->SetupAttachment(GetMesh());
    }
};
```

### Cast Point Setup

**In Blueprint/Editor:**
1. Each entity skeleton should have sockets for cast points:
   - `CastPoint_HandRight`
   - `CastPoint_HandLeft`
   - `CastPoint_Chest`
   - `CastPoint_Head`
   - `CastPoint_Weapon` (optional)

2. Cast points can be manually adjusted via offsets in the editor
3. Visual debug spheres can be enabled to see cast point locations

---

## Animation Notify System

### Overview

Animation Notifies are used to trigger spell execution at precise moments during cast animations. This allows spells to fire at the exact frame where the animation shows the spell being cast, rather than relying solely on cast time.

### Spell Fire Notify

```cpp
UCLASS()
class ATOS_API UAnimNotify_SpellFire : public UAnimNotify
{
    GENERATED_BODY()

public:
    UAnimNotify_SpellFire();

    virtual void Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation) override;

    // Spell ID to fire (set in animation editor)
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Spell")
    int32 SpellID;

    // Cast point type to use
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Spell")
    UCastPointComponent::ECastPointType CastPointType = UCastPointComponent::ECastPointType::HandRight;

    // Target data (if available from context)
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Spell")
    bool bUseContextTarget = true;

#if WITH_EDITOR
    virtual bool CanBePlaced(UAnimSequenceBase* Animation, int32 TrackIndex, int32 NotifyIndex) const override;
#endif
};
```

### Spell Fire Notify Implementation

```cpp
void UAnimNotify_SpellFire::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
    Super::Notify(MeshComp, Animation);

    AActor* Owner = MeshComp->GetOwner();
    if (!Owner)
    {
        return;
    }

    // Get ability system component
    UAbilitySystemComponent* ASC = Owner->FindComponentByClass<UAbilitySystemComponent>();
    if (!ASC)
    {
        return;
    }

    // Get spell execution manager
    UTOSSpellExecutionManager* SpellManager = Owner->FindComponentByClass<UTOSSpellExecutionManager>();
    if (!SpellManager)
    {
        return;
    }

    // Execute spell fire
    SpellManager->ExecuteSpellFire(SpellID, CastPointType, Owner);
}
```

### Spell Execution Manager

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class ATOS_API UTOSSpellExecutionManager : public UActorComponent
{
    GENERATED_BODY()

public:
    UTOSSpellExecutionManager();

    // Execute spell fire (called by animation notify or directly)
    UFUNCTION(BlueprintCallable, Category = "Spell")
    void ExecuteSpellFire(int32 SpellID, UCastPointComponent::ECastPointType CastPointType, AActor* Caster);

    // Execute spell fire with target data
    UFUNCTION(BlueprintCallable, Category = "Spell")
    void ExecuteSpellFireWithTarget(int32 SpellID, UCastPointComponent::ECastPointType CastPointType, 
        AActor* Caster, const FSpellTargetData& TargetData);

    // Generic spell execution (works for any entity type)
    UFUNCTION(BlueprintCallable, Category = "Spell")
    void ExecuteSpellGeneric(int32 SpellID, AActor* Caster, const FSpellTargetData& TargetData);

    // Get current spell context (for animation notifies)
    UFUNCTION(BlueprintCallable, Category = "Spell")
    FSpellContext GetCurrentSpellContext() const { return CurrentSpellContext; }

    // Set spell context (called before animation plays)
    UFUNCTION(BlueprintCallable, Category = "Spell")
    void SetSpellContext(const FSpellContext& Context) { CurrentSpellContext = Context; }

protected:
    // Current spell context (set before animation, used by notify)
    UPROPERTY()
    FSpellContext CurrentSpellContext;

    // Get cast point location
    FVector GetCastPointLocation(AActor* Caster, UCastPointComponent::ECastPointType CastPointType) const;

    // Spawn projectile
    void SpawnProjectile(int32 SpellID, AActor* Caster, const FVector& StartLocation, const FVector& TargetLocation);

    // Execute spell by type
    void ExecuteSpellByType(int32 SpellID, AActor* Caster, const FSpellTargetData& TargetData);
};
```

### Spell Context Structure

```cpp
USTRUCT(BlueprintType)
struct FSpellContext
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadWrite)
    int32 SpellID;

    UPROPERTY(BlueprintReadWrite)
    UCastPointComponent::ECastPointType CastPointType;

    UPROPERTY(BlueprintReadWrite)
    FSpellTargetData TargetData;

    UPROPERTY(BlueprintReadWrite)
    AActor* Caster;

    FSpellContext()
        : SpellID(0)
        , CastPointType(UCastPointComponent::ECastPointType::HandRight)
        , Caster(nullptr)
    {}
};
```

---

## Generic Spell Execution System

### Overview

The spell execution system must work generically for any entity type (player, creature, NPC) without distinction. The server can trigger spells for any entity, and the client must execute them using the same system.

### Generic Spell Execution Function

```cpp
// In UTOSSpellExecutionManager
void UTOSSpellExecutionManager::ExecuteSpellGeneric(int32 SpellID, AActor* Caster, const FSpellTargetData& TargetData)
{
    if (!Caster)
    {
        UE_LOG(LogSpell, Warning, TEXT("ExecuteSpellGeneric: Invalid caster"));
        return;
    }

    // Get spell data
    const FSpellData* SpellData = GetSpellData(SpellID);
    if (!SpellData)
    {
        UE_LOG(LogSpell, Warning, TEXT("ExecuteSpellGeneric: Invalid spell ID %d"), SpellID);
        return;
    }

    // Get cast point component
    UCastPointComponent* CastPointComp = Caster->FindComponentByClass<UCastPointComponent>();
    if (!CastPointComp)
    {
        UE_LOG(LogSpell, Warning, TEXT("ExecuteSpellGeneric: Caster %s has no CastPointComponent"), *Caster->GetName());
        return;
    }

    // Get cast point location
    UCastPointComponent::ECastPointType CastPointType = SpellData->DefaultCastPointType;
    FVector CastPointLocation = CastPointComp->GetCastPointLocation(CastPointType);

    // Execute spell based on type
    switch (SpellData->SpellType)
    {
    case ESpellType::Projectile:
        ExecuteProjectileSpell(SpellID, Caster, CastPointLocation, TargetData);
        break;

    case ESpellType::TargetArea:
        ExecuteTargetAreaSpell(SpellID, Caster, TargetData);
        break;

    case ESpellType::CameraDirection:
        ExecuteCameraDirectionSpell(SpellID, Caster, TargetData);
        break;

    case ESpellType::SingleTarget:
        ExecuteSingleTargetSpell(SpellID, Caster, TargetData);
        break;

    case ESpellType::TargetSelf:
        ExecuteTargetSelfSpell(SpellID, Caster);
        break;

    case ESpellType::AreaCentered:
        ExecuteAreaCenteredSpell(SpellID, Caster);
        break;

    case ESpellType::Summon:
        ExecuteSummonSpell(SpellID, Caster, TargetData);
        break;

    default:
        UE_LOG(LogSpell, Warning, TEXT("ExecuteSpellGeneric: Unknown spell type for spell %d"), SpellID);
        break;
    }
}

void UTOSSpellExecutionManager::ExecuteProjectileSpell(int32 SpellID, AActor* Caster, 
    const FVector& StartLocation, const FSpellTargetData& TargetData)
{
    // Get spell data
    const FSpellData* SpellData = GetSpellData(SpellID);
    if (!SpellData)
    {
        return;
    }

    // Calculate target location
    FVector TargetLocation = TargetData.TargetLocation;
    if (TargetData.TargetActor)
    {
        TargetLocation = TargetData.TargetActor->GetActorLocation();
    }

    // Spawn projectile
    SpawnProjectile(SpellID, Caster, StartLocation, TargetLocation);
}
```

### Projectile Spawn with Cast Point

```cpp
void UTOSSpellExecutionManager::SpawnProjectile(int32 SpellID, AActor* Caster, 
    const FVector& StartLocation, const FVector& TargetLocation)
{
    // Get spell data
    const FSpellData* SpellData = GetSpellData(SpellID);
    if (!SpellData)
    {
        return;
    }

    // Get projectile class
    TSubclassOf<AProjectileActor> ProjectileClass = GetProjectileClass(SpellID);
    if (!ProjectileClass)
    {
        UE_LOG(LogSpell, Warning, TEXT("SpawnProjectile: No projectile class for spell %d"), SpellID);
        return;
    }

    // Spawn projectile at cast point location
    FActorSpawnParameters SpawnParams;
    SpawnParams.Owner = Caster;
    SpawnParams.Instigator = Cast<APawn>(Caster);

    AProjectileActor* Projectile = GetWorld()->SpawnActor<AProjectileActor>(
        ProjectileClass,
        StartLocation,
        (TargetLocation - StartLocation).Rotation(),
        SpawnParams
    );

    if (Projectile)
    {
        // Configure projectile
        Projectile->SetSpeed(SpellData->ProjectileSpeed);
        Projectile->SetTargetLocation(TargetLocation);
        Projectile->SetHoming(SpellData->bHoming);
        if (SpellData->bHoming && TargetData.TargetActor)
        {
            Projectile->SetHomingTarget(TargetData.TargetActor);
        }

        // Spawn trail particles
        if (SpellData->TrailParticles)
        {
            Projectile->SetTrailParticles(SpellData->TrailParticles);
        }

        // Play travel sound
        if (SpellData->TravelSound)
        {
            Projectile->SetTravelSound(SpellData->TravelSound);
        }
    }
}
```

---

## Server-Triggered Spell Execution

### Network Spell Execution

The server can trigger spells for any entity via network messages:

```cpp
// Network message structure
USTRUCT()
struct FSpellExecutionMessage
{
    GENERATED_BODY()

    UPROPERTY()
    int32 EntityID;

    UPROPERTY()
    int32 SpellID;

    UPROPERTY()
    FSpellTargetData TargetData;

    UPROPERTY()
    UCastPointComponent::ECastPointType CastPointType;
};
```

### Client-Side Spell Execution Handler

```cpp
// In UENetSubsystem or entity sync system
UFUNCTION()
void HandleSpellExecution(const FSpellExecutionMessage& Message)
{
    // Find entity by ID
    ASyncEntity* Entity = FindEntityByID(Message.EntityID);
    if (!Entity)
    {
        UE_LOG(LogSpell, Warning, TEXT("HandleSpellExecution: Entity %d not found"), Message.EntityID);
        return;
    }

    // Get spell execution manager
    UTOSSpellExecutionManager* SpellManager = Entity->FindComponentByClass<UTOSSpellExecutionManager>();
    if (!SpellManager)
    {
        UE_LOG(LogSpell, Warning, TEXT("HandleSpellExecution: Entity %d has no SpellExecutionManager"), Message.EntityID);
        return;
    }

    // Execute spell generically (works for any entity type)
    SpellManager->ExecuteSpellGeneric(Message.SpellID, Entity, Message.TargetData);
}
```

### Animation Integration

When a spell is triggered, the system:
1. Sets the spell context in the execution manager
2. Plays the cast animation montage
3. Animation notify fires at the correct frame
4. Notify calls `ExecuteSpellFire` which uses the context
5. Projectile/spell effect is created at the cast point location

```cpp
// In spell ability activation
void UTOSGameplayAbility_Spell::ActivateAbility(
    const FGameplayAbilitySpecHandle Handle,
    const FGameplayAbilityActorInfo* ActorInfo,
    const FGameplayAbilityActivationInfo ActivationInfo,
    const FGameplayEventData* TriggerEventData)
{
    AActor* Caster = GetAvatarActorFromActorInfo();
    
    // Get spell execution manager
    UTOSSpellExecutionManager* SpellManager = Caster->FindComponentByClass<UTOSSpellExecutionManager>();
    if (SpellManager)
    {
        // Set spell context (used by animation notify)
        FSpellContext Context;
        Context.SpellID = SpellData.SpellID;
        Context.CastPointType = SpellData.DefaultCastPointType;
        Context.Caster = Caster;
        Context.TargetData = GetTargetData();
        
        SpellManager->SetSpellContext(Context);
    }

    // Play cast animation (notify will fire at correct frame)
    if (CastMontage)
    {
        PlayMontageAndWait(CastMontage);
    }
    else
    {
        // Fallback: execute immediately if no animation
        ExecuteSpellGeneric(SpellData.SpellID, Caster, GetTargetData());
    }
}
```

---

## GAS Implementation

### Gameplay Ability Base Class

```cpp
UCLASS()
class ATOS_API UTOSGameplayAbility_Spell : public UGameplayAbility
{
    GENERATED_BODY()

public:
    UTOSGameplayAbility_Spell();

    // Spell metadata
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spell")
    FSpellData SpellData;

    // Visual components
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Visual")
    UParticleSystem* CastParticles;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Visual")
    UParticleSystem* ProjectileParticles;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Visual")
    UParticleSystem* ImpactParticles;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Visual")
    UParticleSystem* AreaParticles;

    // Audio components
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Audio")
    USoundBase* CastSound;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Audio")
    USoundBase* ImpactSound;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Audio")
    USoundBase* ChannelSound;

    // Animations
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Animation")
    UAnimMontage* CastMontage;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Animation")
    UAnimMontage* ImpactMontage;

    // Visual effects
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Effects")
    TSubclassOf<ACameraShakeBase> CameraShake;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Effects")
    UMaterialParameterCollection* PostProcessCollection;

    // Spell type
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Spell")
    ESpellType SpellType;

    // Override activation
    virtual void ActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        const FGameplayEventData* TriggerEventData
    ) override;

    // Override end ability
    virtual void EndAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        bool bReplicateEndAbility,
        bool bWasCancelled
    ) override;

protected:
    // Spell execution by type
    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteProjectileSpell(const FSpellTargetData& TargetData);

    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteTargetAreaSpell(const FSpellTargetData& TargetData);

    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteCameraDirectionSpell(const FSpellTargetData& TargetData);

    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteSingleTargetSpell(const FSpellTargetData& TargetData);

    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteTargetSelfSpell();

    UFUNCTION(BlueprintImplementableEvent, Category = "Spell")
    void ExecuteAreaCenteredSpell();

    // Visual effect helpers
    UFUNCTION(BlueprintCallable, Category = "Visual")
    void SpawnCastParticles(AActor* Caster);

    UFUNCTION(BlueprintCallable, Category = "Visual")
    void SpawnImpactParticles(const FVector& Location, AActor* Target);

    UFUNCTION(BlueprintCallable, Category = "Visual")
    void PlayCastSound(AActor* Caster);

    UFUNCTION(BlueprintCallable, Category = "Visual")
    void PlayImpactSound(const FVector& Location);

    UFUNCTION(BlueprintCallable, Category = "Visual")
    void PlayCastAnimation(AActor* Caster);

    UFUNCTION(BlueprintCallable, Category = "Visual")
    void ApplyCameraShake(AActor* Caster, float Intensity = 1.0f);
};
```

### Spell Data Structure

```cpp
USTRUCT(BlueprintType)
struct FSpellData
{
    GENERATED_BODY()

    // Spell identification
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 SpellID;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString SpellName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    ESpellType SpellType;

    // Spell properties
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float CastTime;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Cooldown;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Range;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Radius;  // For area spells

    // Damage/effect properties
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    EDamageType DamageType;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TArray<FConditionData> Conditions;

    // Visual properties
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FLinearColor SpellColor;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float ParticleScale;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float SoundVolume;
};
```

### Spell Type Enum

```cpp
UENUM(BlueprintType)
enum class ESpellType : uint8
{
    None,
    Projectile,        // Projectile-based spell
    TargetArea,        // Area effect at target point
    CameraDirection,  // Directional spell along camera
    SingleTarget,      // Direct effect on target
    TargetSelf,        // Self-targeted spell
    AreaCentered       // Area centered on caster
};
```

---

## Spell Type Implementations

### 1. Projectile Spells

**Gameplay Ability Task:**
```cpp
UCLASS()
class ATOS_API UAbilityTask_ProjectileSpell : public UAbilityTask
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Ability|Tasks",
        meta = (HidePin = "OwningAbility", DefaultToSelf = "OwningAbility",
        BlueprintInternalUseOnly = "TRUE"))
    static UAbilityTask_ProjectileSpell* SpawnProjectile(
        UGameplayAbility* OwningAbility,
        FName TaskInstanceName,
        TSubclassOf<AProjectileActor> ProjectileClass,
        const FVector& StartLocation,
        const FVector& TargetLocation,
        float Speed,
        UParticleSystem* TrailParticles,
        USoundBase* TravelSound
    );

    virtual void Activate() override;

    // Delegates
    UPROPERTY(BlueprintAssignable)
    FProjectileHitDelegate OnProjectileHit;

    UPROPERTY(BlueprintAssignable)
    FProjectileHitDelegate OnProjectileMissed;

protected:
    UPROPERTY()
    AProjectileActor* SpawnedProjectile;

    UPROPERTY()
    FVector StartLoc;

    UPROPERTY()
    FVector TargetLoc;

    UPROPERTY()
    float ProjectileSpeed;

    UFUNCTION()
    void OnProjectileHitCallback(AActor* HitActor, const FVector& HitLocation);

    UFUNCTION()
    void OnProjectileMissedCallback();
};
```

**Projectile Actor:**
```cpp
UCLASS()
class ATOS_API AProjectileActor : public AActor
{
    GENERATED_BODY()

public:
    AProjectileActor();

    // Components
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UStaticMeshComponent* ProjectileMesh;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UParticleSystemComponent* TrailParticles;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UAudioComponent* TravelSound;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UProjectileMovementComponent* MovementComponent;

    // Projectile properties
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Projectile")
    float Speed;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Projectile")
    float MaxRange;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Projectile")
    bool bHoming;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Projectile")
    AActor* HomingTarget;

    // Visual effects
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual")
    UParticleSystem* ImpactParticles;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual")
    USoundBase* ImpactSound;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual")
    TSubclassOf<UCameraShakeBase> ImpactCameraShake;

    // Events
    UFUNCTION()
    void OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor,
        UPrimitiveComponent* OtherComp, FVector NormalImpulse,
        const FHitResult& Hit);

    UFUNCTION(BlueprintImplementableEvent, Category = "Projectile")
    void OnProjectileImpact(const FVector& ImpactLocation, AActor* HitActor);

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;

    FVector StartLocation;
    float TraveledDistance;
};
```

**Visual Implementation (Fireball Example):**
```cpp
// In Blueprint or C++ implementation
void UTOSGameplayAbility_Fireball::ExecuteProjectileSpell(const FSpellTargetData& TargetData)
{
    // Spawn cast particles
    SpawnCastParticles(GetAvatarActorFromActorInfo());

    // Play cast sound
    PlayCastSound(GetAvatarActorFromActorInfo());

    // Play cast animation
    PlayCastAnimation(GetAvatarActorFromActorInfo());

    // Spawn projectile task
    UAbilityTask_ProjectileSpell* ProjectileTask = UAbilityTask_ProjectileSpell::SpawnProjectile(
        this,
        FName("FireballProjectile"),
        FireballProjectileClass,
        GetAvatarActorFromActorInfo()->GetActorLocation(),
        TargetData.TargetLocation,
        1500.0f,  // Speed
        FireballTrailParticles,
        FireballTravelSound
    );

    ProjectileTask->OnProjectileHit.AddDynamic(this, &UTOSGameplayAbility_Fireball::OnProjectileHit);
    ProjectileTask->OnProjectileMissed.AddDynamic(this, &UTOSGameplayAbility_Fireball::OnProjectileMissed);
    ProjectileTask->ReadyForActivation();
}

void UTOSGameplayAbility_Fireball::OnProjectileHit(AActor* HitActor, const FVector& HitLocation)
{
    // Spawn impact particles (explosion)
    if (ImpactParticles)
    {
        UGameplayStatics::SpawnEmitterAtLocation(
            GetWorld(),
            ImpactParticles,
            HitLocation,
            FRotator::ZeroRotator,
            true
        );
    }

    // Play impact sound
    if (ImpactSound)
    {
        UGameplayStatics::PlaySoundAtLocation(
            GetWorld(),
            ImpactSound,
            HitLocation
        );
    }

    // Apply camera shake
    ApplyCameraShake(GetAvatarActorFromActorInfo(), 0.5f);

    // Apply damage/effects to hit actor
    if (HitActor)
    {
        // Apply gameplay effect for damage and burn condition
        FGameplayEffectContextHandle EffectContext = MakeEffectContext(GetCurrentAbilitySpec(), GetCurrentActorInfo());
        FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(DamageEffectClass, GetAbilityLevel());
        ApplyGameplayEffectSpecToTarget(GetCurrentAbilitySpec(), GetCurrentActorInfo(), EffectContext, SpecHandle);
    }

    EndAbility(GetCurrentAbilitySpecHandle(), GetCurrentActorInfo(), GetCurrentActivationInfo(), true, false);
}
```

### 2. Target Area Spells

**Gameplay Ability Task:**
```cpp
UCLASS()
class ATOS_API UAbilityTask_TargetAreaSpell : public UAbilityTask
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Ability|Tasks",
        meta = (HidePin = "OwningAbility", DefaultToSelf = "OwningAbility",
        BlueprintInternalUseOnly = "TRUE"))
    static UAbilityTask_TargetAreaSpell* CreateTargetArea(
        UGameplayAbility* OwningAbility,
        FName TaskInstanceName,
        const FVector& TargetLocation,
        float Radius,
        float Duration,
        UParticleSystem* AreaParticles,
        USoundBase* ActivationSound
    );

    virtual void Activate() override;

    // Delegates
    UPROPERTY(BlueprintAssignable)
    FAreaActivatedDelegate OnAreaActivated;

protected:
    UPROPERTY()
    FVector TargetLoc;

    UPROPERTY()
    float AreaRadius;

    UPROPERTY()
    float AreaDuration;

    UPROPERTY()
    UParticleSystemComponent* AreaParticleComponent;

    UFUNCTION()
    void OnAreaTick();

    FTimerHandle AreaTickTimer;
};
```

**Visual Implementation (Healing Circle Example):**
```cpp
void UTOSGameplayAbility_HealingCircle::ExecuteTargetAreaSpell(const FSpellTargetData& TargetData)
{
    // Spawn cast particles
    SpawnCastParticles(GetAvatarActorFromActorInfo());

    // Play cast sound
    PlayCastSound(GetAvatarActorFromActorInfo());

    // Create area task
    UAbilityTask_TargetAreaSpell* AreaTask = UAbilityTask_TargetAreaSpell::CreateTargetArea(
        this,
        FName("HealingCircle"),
        TargetData.TargetLocation,
        500.0f,  // Radius
        10.0f,   // Duration
        HealingCircleParticles,
        HealingCircleActivationSound
    );

    AreaTask->OnAreaActivated.AddDynamic(this, &UTOSGameplayAbility_HealingCircle::OnAreaActivated);
    AreaTask->ReadyForActivation();
}

void UTOSGameplayAbility_HealingCircle::OnAreaActivated()
{
    // Spawn persistent area particles
    // Apply healing effect to allies in area each tick
    // Play healing sound loop
}
```

### 3. Camera Direction Spells

**Gameplay Ability Task:**
```cpp
UCLASS()
class ATOS_API UAbilityTask_CameraDirectionSpell : public UAbilityTask
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Ability|Tasks",
        meta = (HidePin = "OwningAbility", DefaultToSelf = "OwningAbility",
        BlueprintInternalUseOnly = "TRUE"))
    static UAbilityTask_CameraDirectionSpell* ExecuteDirectionalSpell(
        UGameplayAbility* OwningAbility,
        FName TaskInstanceName,
        const FVector& StartLocation,
        const FVector& Direction,
        float Range,
        float Width,
        EDirectionalSpellType SpellType,
        UParticleSystem* BeamParticles,
        USoundBase* BeamSound
    );

    virtual void Activate() override;

protected:
    UPROPERTY()
    FVector StartLoc;

    UPROPERTY()
    FVector DirectionVec;

    UPROPERTY()
    float SpellRange;

    UPROPERTY()
    float SpellWidth;

    UPROPERTY()
    EDirectionalSpellType Type;
};
```

**Visual Implementation (Ice Wall Example):**
```cpp
void UTOSGameplayAbility_IceWall::ExecuteCameraDirectionSpell(const FSpellTargetData& TargetData)
{
    // Get camera direction
    APawn* Pawn = Cast<APawn>(GetAvatarActorFromActorInfo());
    FVector CameraLocation;
    FRotator CameraRotation;
    Pawn->GetActorEyesViewPoint(CameraLocation, CameraRotation);
    FVector Direction = CameraRotation.Vector();

    // Spawn cast particles
    SpawnCastParticles(GetAvatarActorFromActorInfo());

    // Create directional spell task
    UAbilityTask_CameraDirectionSpell* DirectionTask = UAbilityTask_CameraDirectionSpell::ExecuteDirectionalSpell(
        this,
        FName("IceWall"),
        CameraLocation,
        Direction,
        800.0f,  // Range
        200.0f,  // Width
        EDirectionalSpellType::Wall,
        IceWallParticles,
        IceWallSound
    );

    DirectionTask->ReadyForActivation();
}
```

### 4. Single Target Spells

**Visual Implementation (Heal Example):**
```cpp
void UTOSGameplayAbility_Heal::ExecuteSingleTargetSpell(const FSpellTargetData& TargetData)
{
    if (!TargetData.TargetActor)
    {
        EndAbility(GetCurrentAbilitySpecHandle(), GetCurrentActorInfo(), GetCurrentActivationInfo(), true, true);
        return;
    }

    // Spawn cast particles on caster
    SpawnCastParticles(GetAvatarActorFromActorInfo());

    // Play cast sound
    PlayCastSound(GetAvatarActorFromActorInfo());

    // Play cast animation
    PlayCastAnimation(GetAvatarActorFromActorInfo());

    // Spawn travel particles (optional, for visual effect)
    if (TravelParticles)
    {
        // Create particle trail from caster to target
        UGameplayStatics::SpawnEmitterAttached(
            TravelParticles,
            GetAvatarActorFromActorInfo()->GetRootComponent(),
            NAME_None,
            GetAvatarActorFromActorInfo()->GetActorLocation(),
            (TargetData.TargetActor->GetActorLocation() - GetAvatarActorFromActorInfo()->GetActorLocation()).Rotation(),
            EAttachLocation::KeepWorldPosition
        );
    }

    // Apply effect after cast time
    FTimerHandle EffectTimer;
    GetWorld()->GetTimerManager().SetTimer(EffectTimer, [this, TargetData]()
    {
        // Spawn impact particles on target
        SpawnImpactParticles(TargetData.TargetActor->GetActorLocation(), TargetData.TargetActor);

        // Play impact sound
        PlayImpactSound(TargetData.TargetActor->GetActorLocation());

        // Apply healing effect
        FGameplayEffectContextHandle EffectContext = MakeEffectContext(GetCurrentAbilitySpec(), GetCurrentActorInfo());
        FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(HealingEffectClass, GetAbilityLevel());
        ApplyGameplayEffectSpecToTarget(GetCurrentAbilitySpec(), GetCurrentActorInfo(), EffectContext, SpecHandle);

        EndAbility(GetCurrentAbilitySpecHandle(), GetCurrentActorInfo(), GetCurrentActivationInfo(), true, false);
    }, CastTime, false);
}
```

### 5. Target Self Spells

**Visual Implementation (Self-Heal Example):**
```cpp
void UTOSGameplayAbility_SelfHeal::ExecuteTargetSelfSpell()
{
    AActor* Caster = GetAvatarActorFromActorInfo();

    // Spawn self-cast particles
    if (CastParticles)
    {
        UGameplayStatics::SpawnEmitterAttached(
            CastParticles,
            Caster->GetRootComponent(),
            NAME_None,
            FVector::ZeroVector,
            FRotator::ZeroRotator,
            EAttachLocation::KeepWorldPosition
        );
    }

    // Play cast sound
    PlayCastSound(Caster);

    // Play self-cast animation
    PlayCastAnimation(Caster);

    // Apply effect after cast time
    FTimerHandle EffectTimer;
    GetWorld()->GetTimerManager().SetTimer(EffectTimer, [this, Caster]()
    {
        // Spawn healing particles
        if (HealingParticles)
        {
            UGameplayStatics::SpawnEmitterAttached(
                HealingParticles,
                Caster->GetRootComponent()
            );
        }

        // Play healing sound
        if (HealingSound)
        {
            UGameplayStatics::PlaySoundAttached(
                HealingSound,
                Caster->GetRootComponent()
            );
        }

        // Apply healing effect
        FGameplayEffectContextHandle EffectContext = MakeEffectContext(GetCurrentAbilitySpec(), GetCurrentActorInfo());
        FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(HealingEffectClass, GetAbilityLevel());
        ApplyGameplayEffectSpecToTarget(GetCurrentAbilitySpec(), GetCurrentActorInfo(), EffectContext, SpecHandle);

        EndAbility(GetCurrentAbilitySpecHandle(), GetCurrentActorInfo(), GetCurrentActivationInfo(), true, false);
    }, CastTime, false);
}
```

### 6. Area Centered Spells

**Visual Implementation (Blizzard Example):**
```cpp
void UTOSGameplayAbility_Blizzard::ExecuteAreaCenteredSpell()
{
    AActor* Caster = GetAvatarActorFromActorInfo();
    FVector CasterLocation = Caster->GetActorLocation();

    // Spawn cast particles
    SpawnCastParticles(Caster);

    // Play cast sound
    PlayCastSound(Caster);

    // Play cast animation
    PlayCastAnimation(Caster);

    // Spawn area particles (centered on caster)
    if (AreaParticles)
    {
        UParticleSystemComponent* ParticleComp = UGameplayStatics::SpawnEmitterAtLocation(
            GetWorld(),
            AreaParticles,
            CasterLocation,
            FRotator::ZeroRotator,
            true
        );

        // Scale particles to match radius
        if (ParticleComp)
        {
            ParticleComp->SetFloatParameter(FName("Radius"), Radius);
        }
    }

    // Play area sound (looping)
    if (AreaSound)
    {
        UAudioComponent* AudioComp = UGameplayStatics::SpawnSoundAtLocation(
            GetWorld(),
            AreaSound,
            CasterLocation
        );

        if (AudioComp)
        {
            AudioComp->SetBoolParameter(FName("Looping"), true);
        }
    }

    // Apply camera shake
    ApplyCameraShake(Caster, 0.3f);

    // Apply damage/effects each tick
    FTimerHandle TickTimer;
    float TickInterval = 1.0f;
    float ElapsedTime = 0.0f;
    float Duration = 10.0f;

    GetWorld()->GetTimerManager().SetTimer(TickTimer, [this, Caster, CasterLocation, &ElapsedTime, Duration, TickInterval]()
    {
        ElapsedTime += TickInterval;

        // Find enemies in radius
        TArray<AActor*> OverlappingActors;
        TArray<TEnumAsByte<EObjectTypeQuery>> ObjectTypes;
        ObjectTypes.Add(UEngineTypes::ConvertToObjectType(ECC_Pawn));

        UKismetSystemLibrary::SphereOverlapActors(
            GetWorld(),
            CasterLocation,
            Radius,
            ObjectTypes,
            AActor::StaticClass(),
            TArray<AActor*>{Caster},
            OverlappingActors
        );

        // Apply effects to enemies
        for (AActor* Actor : OverlappingActors)
        {
            // Apply damage and slow effect
            FGameplayEffectContextHandle EffectContext = MakeEffectContext(GetCurrentAbilitySpec(), GetCurrentActorInfo());
            FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(DamageEffectClass, GetAbilityLevel());
            ApplyGameplayEffectSpecToTarget(GetCurrentAbilitySpec(), GetCurrentActorInfo(), EffectContext, SpecHandle);
        }

        // Spawn tick particles
        if (TickParticles)
        {
            UGameplayStatics::SpawnEmitterAtLocation(
                GetWorld(),
                TickParticles,
                CasterLocation + FMath::VRand() * Radius,
                FRotator::ZeroRotator
            );
        }

        // End ability after duration
        if (ElapsedTime >= Duration)
        {
            EndAbility(GetCurrentAbilitySpecHandle(), GetCurrentActorInfo(), GetCurrentActivationInfo(), true, false);
        }
    }, TickInterval, true);
}
```

---

## Condition Visual Effects

### Condition Particles and Effects

Each condition type has associated visual effects:

**Burn Condition:**
```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UParticleSystem* BurnParticles;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
USoundBase* BurnSound;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UMaterialParameterCollection* BurnPostProcess;
```

**Freeze Condition:**
```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UParticleSystem* FreezeParticles;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UMaterialInstanceDynamic* FreezeMaterial;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
USoundBase* FreezeSound;
```

**Poison Condition:**
```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UParticleSystem* PoisonParticles;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
UMaterialParameterCollection* PoisonPostProcess;

UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Conditions")
USoundBase* PoisonSound;
```

---

## Spell UI Integration

### Cast Bar

```cpp
UCLASS()
class ATOS_API UTOSCastBarWidget : public UUserWidget
{
    GENERATED_BODY()

public:
    UPROPERTY(meta = (BindWidget))
    class UProgressBar* CastProgressBar;

    UPROPERTY(meta = (BindWidget))
    class UTextBlock* SpellNameText;

    UFUNCTION(BlueprintCallable)
    void StartCast(const FString& SpellName, float CastTime);

    UFUNCTION(BlueprintCallable)
    void UpdateCastProgress(float Progress);

    UFUNCTION(BlueprintCallable)
    void CancelCast();

protected:
    FTimerHandle CastTimer;
    float CurrentCastTime;
    float TotalCastTime;
};
```

### Cooldown Display

```cpp
UCLASS()
class ATOS_API UTOSCooldownWidget : public UUserWidget
{
    GENERATED_BODY()

public:
    UPROPERTY(meta = (BindWidget))
    class UImage* SpellIcon;

    UPROPERTY(meta = (BindWidget))
    class UTextBlock* CooldownText;

    UPROPERTY(meta = (BindWidget))
    class UProgressBar* CooldownProgressBar;

    UFUNCTION(BlueprintCallable)
    void SetCooldown(float CooldownDuration);

    UFUNCTION(BlueprintCallable)
    void UpdateCooldown(float RemainingTime);

protected:
    FTimerHandle CooldownTimer;
    float RemainingCooldown;
};
```

---

## Network Synchronization

### Spell Execution Sync

Spells are synchronized between server and client:

```cpp
// Server sends spell execution to clients
UFUNCTION(NetMulticast, Reliable)
void MulticastExecuteSpell(int32 SpellID, FVector TargetLocation, AActor* TargetActor);

// Client receives spell execution
void MulticastExecuteSpell_Implementation(int32 SpellID, FVector TargetLocation, AActor* TargetActor)
{
    // Find spell ability
    UTOSGameplayAbility_Spell* SpellAbility = GetSpellAbility(SpellID);
    if (SpellAbility)
    {
        // Execute visual effects
        FSpellTargetData TargetData;
        TargetData.TargetLocation = TargetLocation;
        TargetData.TargetActor = TargetActor;

        switch (SpellAbility->SpellType)
        {
        case ESpellType::Projectile:
            SpellAbility->ExecuteProjectileSpell(TargetData);
            break;
        case ESpellType::TargetArea:
            SpellAbility->ExecuteTargetAreaSpell(TargetData);
            break;
        // ... other types
        }
    }
}
```

---

## Performance Optimization

### Particle Pooling

```cpp
UCLASS()
class ATOS_API UTOSParticlePool : public UObject
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable)
    UParticleSystemComponent* GetPooledParticle(UParticleSystem* Template);

    UFUNCTION(BlueprintCallable)
    void ReturnPooledParticle(UParticleSystemComponent* Particle);

protected:
    UPROPERTY()
    TMap<UParticleSystem*, TArray<UParticleSystemComponent*>> ParticlePool;
};
```

### LOD System

```cpp
// Adjust particle quality based on distance
void UTOSGameplayAbility_Spell::SpawnCastParticles(AActor* Caster)
{
    float DistanceToCamera = FVector::Dist(Caster->GetActorLocation(), GetWorld()->GetFirstPlayerController()->GetPawn()->GetActorLocation());
    
    UParticleSystem* ParticlesToUse = CastParticles;
    
    if (DistanceToCamera > 5000.0f)
    {
        ParticlesToUse = CastParticlesLOD;  // Lower quality particles
    }
    
    UGameplayStatics::SpawnEmitterAttached(ParticlesToUse, Caster->GetRootComponent());
}
```

---

## 7. Summon Spells

### Overview

Summon spells create temporary minions that are allied to the summoner and their allies. These minions can be controllable or non-controllable, and have their own individual behaviors. They do not cause damage to allies but actively seek and damage enemies.

### Characteristics

**Minion Types:**
- **Controllable Minions**: Can be controlled by the summoner (follow, attack, defend commands)
- **Non-Controllable Minions**: Act autonomously with their own AI behaviors
- **Animated Entities**: Minions are full entities with animations, particles, and behaviors

**Minion Properties:**
- Temporary duration (lifetime)
- Allied to summoner and their allies
- Cannot damage allies (friendly fire protection)
- Individual behaviors (patrol, seek enemies, attack, etc.)
- Visual representation (animated entities with particles)

**Examples:**
- **Vortex**: Creates an energy vortex that moves randomly seeking enemies and dealing damage
- **Skeleton Warrior**: Controllable melee minion
- **Familiar**: Flying minion that follows and assists
- **Totem**: Stationary minion that provides area effects

### Summon Spell Data Structure

```cpp
USTRUCT(BlueprintType)
struct FSummonSpellData
{
    GENERATED_BODY()

    // Minion entity type/class
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSubclassOf<ASyncEntity> MinionClass;

    // Minion lifetime (seconds)
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Lifetime;

    // Number of minions to summon
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 MinionCount = 1;

    // Spawn location relative to caster
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FVector SpawnOffset = FVector::ZeroVector;

    // Spawn radius (random spawn within radius)
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float SpawnRadius = 100.0f;

    // Is minion controllable
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bControllable = false;

    // Minion behavior type
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    EMinionBehaviorType BehaviorType = EMinionBehaviorType::Aggressive;

    // Visual effects
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual")
    UParticleSystem* SpawnParticles;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Visual")
    UParticleSystem* AppearanceParticles;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Audio")
    USoundBase* SpawnSound;
};

UENUM(BlueprintType)
enum class EMinionBehaviorType : uint8
{
    Aggressive,      // Actively seeks and attacks enemies
    Defensive,       // Defends summoner and allies
    Patrol,          // Patrols area randomly
    Follow,          // Follows summoner
    Stationary,      // Stays in place (totems, etc.)
    Custom           // Custom behavior
};
```

### Summon Minion Entity

```cpp
UCLASS()
class ATOS_API ASyncSummon : public ASyncEntity
{
    GENERATED_BODY()

public:
    ASyncSummon();

    // Summoner reference
    UPROPERTY(BlueprintReadOnly, Category = "Summon")
    AActor* Summoner;

    // Lifetime timer
    UPROPERTY(BlueprintReadOnly, Category = "Summon")
    float RemainingLifetime;

    // Is controllable
    UPROPERTY(BlueprintReadOnly, Category = "Summon")
    bool bControllable;

    // Behavior type
    UPROPERTY(BlueprintReadOnly, Category = "Summon")
    EMinionBehaviorType BehaviorType;

    // Team/alignment (allied to summoner)
    UPROPERTY(BlueprintReadOnly, Category = "Summon")
    ETeamAlignment TeamAlignment;

    // Behavior component
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UMinionBehaviorComponent* BehaviorComponent;

    // Initialize summon
    UFUNCTION(BlueprintCallable, Category = "Summon")
    void InitializeSummon(AActor* InSummoner, float Lifetime, bool bInControllable, EMinionBehaviorType InBehaviorType);

    // Start lifetime countdown
    UFUNCTION(BlueprintCallable, Category = "Summon")
    void StartLifetime(float Lifetime);

    // Despawn summon
    UFUNCTION(BlueprintCallable, Category = "Summon")
    void Despawn();

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;

    // Lifetime timer handle
    FTimerHandle LifetimeTimerHandle;

    // Despawn particles
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Visual")
    UParticleSystem* DespawnParticles;

    // Despawn sound
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Audio")
    USoundBase* DespawnSound;
};
```

### Minion Behavior Component

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class ATOS_API UMinionBehaviorComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UMinionBehaviorComponent();

    // Set behavior type
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void SetBehaviorType(EMinionBehaviorType Type);

    // Update behavior (called each tick)
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void UpdateBehavior(float DeltaTime);

    // Aggressive behavior: seek and attack enemies
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void ExecuteAggressiveBehavior(float DeltaTime);

    // Defensive behavior: defend summoner
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void ExecuteDefensiveBehavior(float DeltaTime);

    // Patrol behavior: random movement
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void ExecutePatrolBehavior(float DeltaTime);

    // Follow behavior: follow summoner
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void ExecuteFollowBehavior(float DeltaTime);

    // Stationary behavior: stay in place
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    void ExecuteStationaryBehavior(float DeltaTime);

protected:
    UPROPERTY()
    EMinionBehaviorType CurrentBehaviorType;

    UPROPERTY()
    AActor* OwnerActor;

    UPROPERTY()
    AActor* Summoner;

    // Behavior state
    UPROPERTY()
    FVector PatrolTargetLocation;

    UPROPERTY()
    float LastBehaviorUpdateTime;

    // Find nearest enemy
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    AActor* FindNearestEnemy(float SearchRadius = 1000.0f) const;

    // Check if target is enemy
    UFUNCTION(BlueprintCallable, Category = "Behavior")
    bool IsEnemy(AActor* Target) const;
};
```

### Summon Spell Execution

```cpp
void UTOSSpellExecutionManager::ExecuteSummonSpell(int32 SpellID, AActor* Caster, const FSpellTargetData& TargetData)
{
    // Get spell data
    const FSpellData* SpellData = GetSpellData(SpellID);
    if (!SpellData || !SpellData->SummonData)
    {
        UE_LOG(LogSpell, Warning, TEXT("ExecuteSummonSpell: Invalid spell data for spell %d"), SpellID);
        return;
    }

    const FSummonSpellData& SummonData = *SpellData->SummonData;

    // Get cast point location
    UCastPointComponent* CastPointComp = Caster->FindComponentByClass<UCastPointComponent>();
    FVector SpawnLocation = Caster->GetActorLocation();
    
    if (CastPointComp)
    {
        SpawnLocation = CastPointComp->GetCastPointLocation(UCastPointComponent::ECastPointType::Chest);
    }

    // Apply spawn offset and random radius
    SpawnLocation += SummonData.SpawnOffset;
    
    // Spawn multiple minions if count > 1
    for (int32 i = 0; i < SummonData.MinionCount; ++i)
    {
        // Random spawn location within radius
        FVector RandomOffset = FMath::VRand() * SummonData.SpawnRadius;
        FVector FinalSpawnLocation = SpawnLocation + RandomOffset;

        // Spawn summon portal/rift particles
        if (SummonData.SpawnParticles)
        {
            UGameplayStatics::SpawnEmitterAtLocation(
                GetWorld(),
                SummonData.SpawnParticles,
                FinalSpawnLocation,
                FRotator::ZeroRotator
            );
        }

        // Play spawn sound
        if (SummonData.SpawnSound)
        {
            UGameplayStatics::PlaySoundAtLocation(
                GetWorld(),
                SummonData.SpawnSound,
                FinalSpawnLocation
            );
        }

        // Spawn minion entity
        FActorSpawnParameters SpawnParams;
        SpawnParams.Owner = Caster;
        SpawnParams.Instigator = Cast<APawn>(Caster);
        SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AdjustIfPossibleButAlwaysSpawn;

        ASyncSummon* Minion = GetWorld()->SpawnActor<ASyncSummon>(
            SummonData.MinionClass,
            FinalSpawnLocation,
            FRotator::ZeroRotator,
            SpawnParams
        );

        if (Minion)
        {
            // Initialize minion
            Minion->InitializeSummon(
                Caster,
                SummonData.Lifetime,
                SummonData.bControllable,
                SummonData.BehaviorType
            );

            // Set team alignment (allied to summoner)
            Minion->SetTeamAlignment(GetTeamAlignment(Caster));

            // Start lifetime countdown
            Minion->StartLifetime(SummonData.Lifetime);

            // Spawn appearance particles
            if (SummonData.AppearanceParticles)
            {
                UGameplayStatics::SpawnEmitterAttached(
                    SummonData.AppearanceParticles,
                    Minion->GetRootComponent()
                );
            }
        }
    }
}
```

### Vortex Example Implementation

```cpp
// Vortex minion behavior (non-controllable, aggressive)
void UMinionBehaviorComponent::ExecuteAggressiveBehavior(float DeltaTime)
{
    if (!OwnerActor)
    {
        return;
    }

    // Find nearest enemy
    AActor* NearestEnemy = FindNearestEnemy(1500.0f);

    if (NearestEnemy)
    {
        // Move towards enemy
        FVector Direction = (NearestEnemy->GetActorLocation() - OwnerActor->GetActorLocation()).GetSafeNormal();
        float MoveSpeed = 300.0f; // Vortex movement speed
        
        OwnerActor->AddMovementInput(Direction, MoveSpeed * DeltaTime);

        // Check if in range to damage
        float DistanceToEnemy = FVector::Dist(OwnerActor->GetActorLocation(), NearestEnemy->GetActorLocation());
        if (DistanceToEnemy < 200.0f) // Damage radius
        {
            // Apply damage (handled by server, but visual feedback here)
            // Damage is applied continuously while in range
        }
    }
    else
    {
        // No enemy found, move randomly
        ExecutePatrolBehavior(DeltaTime);
    }
}
```

### Minion Despawn

```cpp
void ASyncSummon::Despawn()
{
    // Spawn despawn particles
    if (DespawnParticles)
    {
        UGameplayStatics::SpawnEmitterAtLocation(
            GetWorld(),
            DespawnParticles,
            GetActorLocation(),
            FRotator::ZeroRotator
        );
    }

    // Play despawn sound
    if (DespawnSound)
    {
        UGameplayStatics::PlaySoundAtLocation(
            GetWorld(),
            DespawnSound,
            GetActorLocation()
        );
    }

    // Fade out or destroy
    // Could use timeline for smooth fade out
    Destroy();
}

void ASyncSummon::StartLifetime(float Lifetime)
{
    RemainingLifetime = Lifetime;

    // Set timer to despawn
    GetWorldTimerManager().SetTimer(
        LifetimeTimerHandle,
        this,
        &ASyncSummon::Despawn,
        Lifetime,
        false
    );
}
```

### Team Alignment and Friendly Fire Protection

```cpp
// In ASyncSummon or base entity
bool ASyncSummon::IsEnemy(AActor* Target) const
{
    if (!Target || !Summoner)
    {
        return false;
    }

    // Get target's team
    ETeamAlignment TargetTeam = GetTeamAlignment(Target);
    ETeamAlignment SummonerTeam = GetTeamAlignment(Summoner);

    // Same team = not enemy (friendly fire protection)
    if (TargetTeam == SummonerTeam)
    {
        return false;
    }

    // Different team = enemy
    return TargetTeam != TeamAlignment;
}

// Damage application (only damages enemies)
void ASyncSummon::ApplyDamageToTarget(AActor* Target, float Damage)
{
    if (!IsEnemy(Target))
    {
        // Friendly fire protection - don't damage allies
        return;
    }

    // Apply damage (server authoritative)
    // Visual feedback can be shown here
}
```

### Controllable Minions

```cpp
// For controllable minions, add command system
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class ATOS_API UMinionCommandComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    // Command types
    UENUM(BlueprintType)
    enum class EMinionCommand : uint8
    {
        Follow,      // Follow summoner
        Attack,      // Attack target
        Defend,      // Defend position
        Patrol,      // Patrol area
        Stay         // Stay in place
    };

    // Execute command
    UFUNCTION(BlueprintCallable, Category = "Command")
    void ExecuteCommand(EMinionCommand Command, AActor* Target = nullptr, FVector Location = FVector::ZeroVector);

protected:
    UPROPERTY()
    EMinionCommand CurrentCommand;

    UPROPERTY()
    AActor* CommandTarget;

    UPROPERTY()
    FVector CommandLocation;
};
```

### Summon Spell Visual Effects

**Spawn Effects:**
- Portal/rift particles at spawn location
- Energy burst particles
- Summon sound

**Minion Appearance:**
- Appearance particles on minion
- Material effects (glow, energy, etc.)
- Animation start

**Minion Persistent Effects:**
- Trail particles (for moving minions like Vortex)
- Aura particles (for stationary minions)
- Behavior-specific particles

**Despawn Effects:**
- Fade out particles
- Despawn sound
- Energy dissipation effects

---

## See Also

- [SPELLS.md](../../entities/SPELLS.md) - Server-side spells system
- [ACTIONS.md](../../entities/ACTIONS.md) - Actions system documentation
- [ENTITY_SYNC.md](./ENTITY_SYNC.md) - Entity synchronization
- [INPUT.md](./INPUT.md) - Input handling for spells

