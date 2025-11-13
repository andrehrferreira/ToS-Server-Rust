# Client Architecture

## Overview

This document describes the complete architecture of the Unreal Engine client, including plugin structure, content organization, GameInstance customization, PlayerController functionality, and entity management systems.

## Project Structure

### Plugin Architecture

All C++ scripts are organized in a plugin that can be accessed via Blueprint when necessary. This provides a clean separation between engine code and game-specific code.

```
ProjectRoot/
├── Plugins/
│   └── ToSClient/
│       ├── Source/
│       │   └── ToSClient/
│       │       ├── Public/
│       │       │   ├── Core/
│       │       │   │   ├── ToSGameInstance.h
│       │       │   │   ├── ToSPlayerController.h
│       │       │   │   └── ToSPlayerState.h
│       │       │   ├── Network/
│       │       │   │   ├── UENetSubsystem.h
│       │       │   │   ├── UDPClient.h
│       │       │   │   └── FSecureSession.h
│       │       │   ├── Entities/
│       │       │   │   ├── ASyncEntity.h
│       │       │   │   ├── ASyncPlayer.h
│       │       │   │   ├── ASyncCreature.h
│       │       │   │   └── ASyncSummon.h
│       │       │   ├── Spells/
│       │       │   │   ├── UTOSGameplayAbility_Spell.h
│       │       │   │   ├── UTOSSpellExecutionManager.h
│       │       │   │   ├── UCastPointComponent.h
│       │       │   │   └── UAnimNotify_SpellFire.h
│       │       │   ├── GAS/
│       │       │   │   ├── ToSAbilitySystemComponent.h
│       │       │   │   ├── ToSAttributeSet.h
│       │       │   │   └── ToSGameplayEffect.h
│       │       │   └── Gathering/
│       │       │       ├── AGatherableResource.h
│       │       │       ├── ATree.h
│       │       │       ├── AOreNode.h
│       │       │       └── AHarvestablePlant.h
│       │       └── Private/
│       │           └── [Corresponding .cpp files]
│       ├── Content/
│       │   └── [Plugin-specific content]
│       └── ToSClient.Build.cs
└── Content/
    └── _Game/
        ├── Data/
        │   ├── DataTables/
        │   │   ├── DT_Spells.uasset
        │   │   ├── DT_Creatures.uasset
        │   │   ├── DT_Items.uasset
        │   │   └── DT_GatherableResources.uasset
        │   └── Config/
        │       └── ServerConfig.json
        ├── VFX/
        │   ├── Spells/
        │   │   ├── Fire/
        │   │   ├── Ice/
        │   │   ├── Lightning/
        │   │   └── Summon/
        │   ├── Projectiles/
        │   ├── Impacts/
        │   └── Conditions/
        ├── SFX/
        │   ├── Spells/
        │   ├── Combat/
        │   ├── Ambient/
        │   └── UI/
        ├── Textures/
        │   ├── Spells/
        │   ├── Items/
        │   ├── Creatures/
        │   └── UI/
        ├── Models/
        │   ├── Spells/
        │   │   ├── Projectiles/
        │   │   └── Summons/
        │   ├── Creatures/
        │   ├── Gatherable/
        │   │   ├── Trees/
        │   │   ├── OreNodes/
        │   │   └── Plants/
        │   └── Items/
        └── Systems/
            ├── Creatures/
            │   ├── Blueprints/
            │   ├── Animations/
            │   └── AI/
            └── Gathering/
                ├── Blueprints/
                └── Interactions/
```

---

## Content Organization (_Game Folder)

### Data Tables

**Location:** `Content/_Game/Data/DataTables/`

**Purpose:** Store game data that can be accessed by both C++ and Blueprint.

**Data Tables:**
- **DT_Spells**: Spell definitions, visual effects, sounds, animations
- **DT_Creatures**: Creature stats, behaviors, visual assets
- **DT_Items**: Item definitions, properties, visual assets
- **DT_GatherableResources**: Resource types, spawn rates, visual models

**Usage:**
```cpp
// C++ access
UDataTable* SpellsTable = LoadObject<UDataTable>(nullptr, TEXT("/Game/_Game/Data/DataTables/DT_Spells"));
FSpellData* SpellData = SpellsTable->FindRow<FSpellData>(FName(*FString::FromInt(SpellID)), TEXT("SpellData"));
```

### VFX (Visual Effects)

**Location:** `Content/_Game/VFX/`

**Organization:**
- **Spells/**: Particle systems for each spell type
  - Fire, Ice, Lightning, Poison, etc.
  - Summon portals and effects
- **Projectiles/**: Projectile trail and impact effects
- **Impacts/**: Impact effects for different damage types
- **Conditions/**: Visual effects for status conditions (burn, freeze, poison)

### SFX (Sound Effects)

**Location:** `Content/_Game/SFX/`

**Organization:**
- **Spells/**: Cast sounds, impact sounds, channel sounds
- **Combat/**: Weapon sounds, hit sounds, death sounds
- **Ambient/**: Environmental sounds
- **UI/**: UI interaction sounds

### Textures

**Location:** `Content/_Game/Textures/`

**Organization:**
- **Spells/**: Spell icons, UI elements
- **Items/**: Item icons, thumbnails
- **Creatures/**: Creature portraits, UI elements
- **UI/**: Interface textures

### Models (3D Assets)

**Location:** `Content/_Game/Models/`

**Organization:**
- **Spells/**: 
  - Projectiles: 3D meshes for projectiles
  - Summons: 3D models for summoned minions
- **Creatures/**: Creature meshes, skeletons
- **Gatherable/**: 
  - Trees: Tree models for gathering
  - OreNodes: Ore node models
  - Plants: Harvestable plant models
- **Items/**: 3D item models

### Dynamic Systems

**Location:** `Content/_Game/Systems/`

**Creatures:**
- Blueprints: Creature Blueprint classes
- Animations: Creature animation assets
- AI: AI behavior trees and blackboards

**Gathering:**
- Blueprints: Gatherable resource Blueprints
- Interactions: Interaction Blueprints for gathering mechanics

---

## GameInstance Customization

### ToSGameInstance

The custom GameInstance handles server configuration, connection, authentication, and provides easy access from any Blueprint similar to subsystems.

```cpp
UCLASS()
class ATOS_API UToSGameInstance : public UGameInstance
{
    GENERATED_BODY()

public:
    UToSGameInstance(const FObjectInitializer& ObjectInitializer);

    virtual void Init() override;
    virtual void Shutdown() override;

    // === SERVER CONFIGURATION ===
    
    // Server configuration structure
    USTRUCT(BlueprintType)
    struct FServerConfig
    {
        GENERATED_BODY()

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        FString ServerHost = TEXT("localhost");

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        int32 ServerPort = 7777;

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        FString AuthToken;

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        FString PlayerName;

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        bool bAutoConnect = false;

        UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Server")
        float ConnectionTimeout = 10.0f;
    };

    // Load server configuration from file
    UFUNCTION(BlueprintCallable, Category = "Server|Config")
    bool LoadServerConfig(const FString& ConfigPath);

    // Save server configuration to file
    UFUNCTION(BlueprintCallable, Category = "Server|Config")
    bool SaveServerConfig(const FString& ConfigPath);

    // Get server configuration
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Server|Config")
    FServerConfig GetServerConfig() const { return ServerConfig; }

    // Set server configuration
    UFUNCTION(BlueprintCallable, Category = "Server|Config")
    void SetServerConfig(const FServerConfig& Config);

    // === CONNECTION MANAGEMENT ===

    // Connect to server
    UFUNCTION(BlueprintCallable, Category = "Server|Connection")
    void ConnectToServer();

    // Connect to server with custom config
    UFUNCTION(BlueprintCallable, Category = "Server|Connection")
    void ConnectToServerWithConfig(const FServerConfig& Config);

    // Disconnect from server
    UFUNCTION(BlueprintCallable, Category = "Server|Connection")
    void DisconnectFromServer();

    // Check if connected
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Server|Connection")
    bool IsConnected() const;

    // Get connection status
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Server|Connection")
    EConnectionStatus GetConnectionStatus() const;

    // === AUTHENTICATION ===

    // Authenticate with server
    UFUNCTION(BlueprintCallable, Category = "Server|Auth")
    void Authenticate(const FString& Username, const FString& Password);

    // Authenticate with token
    UFUNCTION(BlueprintCallable, Category = "Server|Auth")
    void AuthenticateWithToken(const FString& Token);

    // Get authentication status
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Server|Auth")
    bool IsAuthenticated() const { return bIsAuthenticated; }

    // Get player ID after authentication
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Server|Auth")
    int32 GetPlayerID() const { return PlayerID; }

    // === NETWORK SUBSYSTEM ACCESS ===

    // Get network subsystem (easy access from Blueprint)
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Network")
    UENetSubsystem* GetNetworkSubsystem() const { return NetworkSubsystem; }

    // === EVENTS ===

    // Connection events
    DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnConnected);
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnConnected OnConnected;

    DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnDisconnected);
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnDisconnected OnDisconnected;

    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnConnectionFailed, const FString&, ErrorMessage);
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnConnectionFailed OnConnectionFailed;

    // Authentication events
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAuthenticated, int32, InPlayerID);
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnAuthenticated OnAuthenticated;

    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAuthenticationFailed, const FString&, ErrorMessage);
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnAuthenticationFailed OnAuthenticationFailed;

protected:
    // Server configuration
    UPROPERTY()
    FServerConfig ServerConfig;

    // Network subsystem reference
    UPROPERTY()
    UENetSubsystem* NetworkSubsystem;

    // Authentication state
    UPROPERTY()
    bool bIsAuthenticated = false;

    UPROPERTY()
    int32 PlayerID = -1;

    // Load config on init
    virtual void LoadConfiguration();

    // Handle connection events
    UFUNCTION()
    void HandleConnected();

    UFUNCTION()
    void HandleDisconnected();

    UFUNCTION()
    void HandleConnectionFailed(const FString& ErrorMessage);

    // Handle authentication events
    UFUNCTION()
    void HandleAuthenticated(int32 InPlayerID);

    UFUNCTION()
    void HandleAuthenticationFailed(const FString& ErrorMessage);
};
```

### GameInstance Implementation

```cpp
void UToSGameInstance::Init()
{
    Super::Init();

    // Get network subsystem
    NetworkSubsystem = GetSubsystem<UENetSubsystem>();
    if (NetworkSubsystem)
    {
        // Bind to network events
        NetworkSubsystem->OnConnected.AddDynamic(this, &UToSGameInstance::HandleConnected);
        NetworkSubsystem->OnDisconnected.AddDynamic(this, &UToSGameInstance::HandleDisconnected);
        NetworkSubsystem->OnConnectionFailed.AddDynamic(this, &UToSGameInstance::HandleConnectionFailed);
    }

    // Load server configuration
    LoadConfiguration();

    // Auto-connect if configured
    if (ServerConfig.bAutoConnect)
    {
        ConnectToServer();
    }
}

void UToSGameInstance::LoadConfiguration()
{
    // Try to load from config file
    FString ConfigPath = FPaths::ProjectConfigDir() / TEXT("ServerConfig.json");
    
    if (FPaths::FileExists(ConfigPath))
    {
        LoadServerConfig(ConfigPath);
    }
    else
    {
        // Use default configuration
        ServerConfig = FServerConfig();
    }
}

bool UToSGameInstance::LoadServerConfig(const FString& ConfigPath)
{
    FString JsonString;
    if (!FFileHelper::LoadFileToString(JsonString, *ConfigPath))
    {
        UE_LOG(LogToS, Warning, TEXT("Failed to load server config from %s"), *ConfigPath);
        return false;
    }

    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(JsonString);

    if (!FJsonSerializer::Deserialize(Reader, JsonObject) || !JsonObject.IsValid())
    {
        UE_LOG(LogToS, Warning, TEXT("Failed to parse server config JSON"));
        return false;
    }

    // Parse configuration
    if (JsonObject->HasField(TEXT("ServerHost")))
    {
        ServerConfig.ServerHost = JsonObject->GetStringField(TEXT("ServerHost"));
    }

    if (JsonObject->HasField(TEXT("ServerPort")))
    {
        ServerConfig.ServerPort = JsonObject->GetIntegerField(TEXT("ServerPort"));
    }

    if (JsonObject->HasField(TEXT("AuthToken")))
    {
        ServerConfig.AuthToken = JsonObject->GetStringField(TEXT("AuthToken"));
    }

    if (JsonObject->HasField(TEXT("PlayerName")))
    {
        ServerConfig.PlayerName = JsonObject->GetStringField(TEXT("PlayerName"));
    }

    return true;
}

void UToSGameInstance::ConnectToServer()
{
    if (!NetworkSubsystem)
    {
        UE_LOG(LogToS, Error, TEXT("NetworkSubsystem not available"));
        return;
    }

    NetworkSubsystem->SetConnectTimeout(ServerConfig.ConnectionTimeout);
    NetworkSubsystem->Connect(ServerConfig.ServerHost, ServerConfig.ServerPort);
}

void UToSGameInstance::Authenticate(const FString& Username, const FString& Password)
{
    // Send authentication packet to server
    // Implementation depends on network protocol
    // After successful authentication, server responds with PlayerID
}

void UToSGameInstance::HandleAuthenticated(int32 InPlayerID)
{
    PlayerID = InPlayerID;
    bIsAuthenticated = true;
    OnAuthenticated.Broadcast(InPlayerID);
}
```

### Blueprint Access

The GameInstance can be easily accessed from any Blueprint:

```blueprint
// In any Blueprint:
Get Game Instance -> Cast to ToS Game Instance -> Access functions
```

---

## PlayerController Customization

### ToSPlayerController

The custom PlayerController handles entity spawning, possession control (mounts, boats, transformations), and player spawning.

```cpp
UCLASS()
class ATOS_API AToSPlayerController : public APlayerController
{
    GENERATED_BODY()

public:
    AToSPlayerController(const FObjectInitializer& ObjectInitializer);

    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;

    // === ENTITY SPAWNING ===

    // Spawn entity by ID
    UFUNCTION(BlueprintCallable, Category = "Entity|Spawn")
    ASyncEntity* SpawnEntity(int32 EntityID, const FVector& Location, const FRotator& Rotation);

    // Spawn player entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Spawn")
    ASyncPlayer* SpawnPlayer(int32 PlayerID, const FVector& Location, const FRotator& Rotation);

    // Spawn creature entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Spawn")
    ASyncCreature* SpawnCreature(int32 CreatureID, const FVector& Location, const FRotator& Rotation);

    // Spawn summon entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Spawn")
    ASyncSummon* SpawnSummon(int32 SummonID, AActor* Summoner, const FVector& Location, const FRotator& Rotation);

    // Remove entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Spawn")
    void RemoveEntity(int32 EntityID);

    // Get entity by ID
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Entity")
    ASyncEntity* GetEntityByID(int32 EntityID) const;

    // === POSSESSION CONTROL ===

    // Possess entity (for mounts, boats, transformations)
    UFUNCTION(BlueprintCallable, Category = "Possession")
    void PossessEntity(ASyncEntity* Entity);

    // Unpossess current entity
    UFUNCTION(BlueprintCallable, Category = "Possession")
    void UnpossessEntity();

    // Possess mount
    UFUNCTION(BlueprintCallable, Category = "Possession")
    void PossessMount(ASyncEntity* Mount);

    // Possess boat
    UFUNCTION(BlueprintCallable, Category = "Possession")
    void PossessBoat(ASyncEntity* Boat);

    // Possess transformed creature (transformation spell)
    UFUNCTION(BlueprintCallable, Category = "Possession")
    void PossessTransformedCreature(ASyncEntity* TransformedCreature);

    // Get currently possessed entity
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Possession")
    ASyncEntity* GetPossessedEntity() const { return PossessedEntity; }

    // Check if possessing entity
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Possession")
    bool IsPossessingEntity() const { return PossessedEntity != nullptr; }

    // === PLAYER SPAWNING ===

    // Spawn player (called after authentication)
    UFUNCTION(BlueprintCallable, Category = "Player|Spawn")
    void SpawnPlayerEntity(int32 PlayerID, const FVector& SpawnLocation, const FRotator& SpawnRotation);

    // Respawn player
    UFUNCTION(BlueprintCallable, Category = "Player|Spawn")
    void RespawnPlayer(const FVector& SpawnLocation, const FRotator& SpawnRotation);

    // Get player entity
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Player")
    ASyncPlayer* GetPlayerEntity() const { return PlayerEntity; }

    // === ENTITY MANAGEMENT ===

    // Register entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Management")
    void RegisterEntity(int32 EntityID, ASyncEntity* Entity);

    // Unregister entity
    UFUNCTION(BlueprintCallable, Category = "Entity|Management")
    void UnregisterEntity(int32 EntityID);

    // Get all registered entities
    UFUNCTION(BlueprintCallable, BlueprintPure, Category = "Entity|Management")
    TArray<ASyncEntity*> GetAllEntities() const;

protected:
    // Player entity reference
    UPROPERTY()
    ASyncPlayer* PlayerEntity;

    // Currently possessed entity (mount, boat, transformed creature)
    UPROPERTY()
    ASyncEntity* PossessedEntity;

    // Entity registry (EntityID -> Entity)
    UPROPERTY()
    TMap<int32, TWeakObjectPtr<ASyncEntity>> EntityRegistry;

    // Entity spawn functions
    ASyncEntity* SpawnEntityInternal(TSubclassOf<ASyncEntity> EntityClass, int32 EntityID, 
        const FVector& Location, const FRotator& Rotation);

    // Handle possession setup
    void SetupPossession(ASyncEntity* Entity);

    // Handle possession cleanup
    void CleanupPossession();
};
```

### PlayerController Implementation

```cpp
ASyncEntity* AToSPlayerController::SpawnEntity(int32 EntityID, const FVector& Location, const FRotator& Rotation)
{
    // Determine entity class based on EntityID
    // This could query a data table or use a factory pattern
    TSubclassOf<ASyncEntity> EntityClass = GetEntityClass(EntityID);
    
    if (!EntityClass)
    {
        UE_LOG(LogToS, Warning, TEXT("Unknown entity class for ID %d"), EntityID);
        return nullptr;
    }

    return SpawnEntityInternal(EntityClass, EntityID, Location, Rotation);
}

ASyncEntity* AToSPlayerController::SpawnEntityInternal(TSubclassOf<ASyncEntity> EntityClass, int32 EntityID,
    const FVector& Location, const FRotator& Rotation)
{
    FActorSpawnParameters SpawnParams;
    SpawnParams.Owner = this;
    SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AdjustIfPossibleButAlwaysSpawn;

    ASyncEntity* Entity = GetWorld()->SpawnActor<ASyncEntity>(
        EntityClass,
        Location,
        Rotation,
        SpawnParams
    );

    if (Entity)
    {
        Entity->EntityId = EntityID;
        RegisterEntity(EntityID, Entity);
    }

    return Entity;
}

ASyncPlayer* AToSPlayerController::SpawnPlayer(int32 PlayerID, const FVector& Location, const FRotator& Rotation)
{
    ASyncPlayer* Player = Cast<ASyncPlayer>(SpawnEntity(PlayerID, Location, Rotation));
    
    if (Player)
    {
        PlayerEntity = Player;
        
        // Possess player if it's the local player
        UToSGameInstance* GameInstance = Cast<UToSGameInstance>(GetGameInstance());
        if (GameInstance && GameInstance->GetPlayerID() == PlayerID)
        {
            Possess(Player);
        }
    }

    return Player;
}

void AToSPlayerController::PossessEntity(ASyncEntity* Entity)
{
    if (!Entity)
    {
        return;
    }

    // Cleanup previous possession
    if (PossessedEntity)
    {
        CleanupPossession();
    }

    // Setup new possession
    PossessedEntity = Entity;
    SetupPossession(Entity);

    // Possess the entity
    Possess(Cast<APawn>(Entity));
}

void AToSPlayerController::PossessMount(ASyncEntity* Mount)
{
    if (!Mount || !Mount->IsA<ASyncMount>())
    {
        UE_LOG(LogToS, Warning, TEXT("Invalid mount entity"));
        return;
    }

    PossessEntity(Mount);
    
    // Mount-specific setup
    // Disable player movement, enable mount movement
    if (PlayerEntity)
    {
        PlayerEntity->SetActorHiddenInGame(true);
        PlayerEntity->SetActorEnableCollision(false);
    }
}

void AToSPlayerController::PossessBoat(ASyncEntity* Boat)
{
    if (!Boat)
    {
        return;
    }

    PossessEntity(Boat);
    
    // Boat-specific setup
    // Enable boat movement controls
}

void AToSPlayerController::PossessTransformedCreature(ASyncEntity* TransformedCreature)
{
    if (!TransformedCreature)
    {
        return;
    }

    PossessEntity(TransformedCreature);
    
    // Transformation-specific setup
    // Hide player, show transformed creature
    if (PlayerEntity)
    {
        PlayerEntity->SetActorHiddenInGame(true);
    }
}

void AToSPlayerController::UnpossessEntity()
{
    if (!PossessedEntity)
    {
        return;
    }

    CleanupPossession();
    PossessedEntity = nullptr;

    // Return control to player
    if (PlayerEntity)
    {
        Possess(PlayerEntity);
        PlayerEntity->SetActorHiddenInGame(false);
        PlayerEntity->SetActorEnableCollision(true);
    }
}

void AToSPlayerController::SetupPossession(ASyncEntity* Entity)
{
    // Setup input for possessed entity
    // Enable entity-specific controls
    // Setup camera for entity
}

void AToSPlayerController::CleanupPossession()
{
    // Cleanup input
    // Restore player controls
}

void AToSPlayerController::RegisterEntity(int32 EntityID, ASyncEntity* Entity)
{
    EntityRegistry.Add(EntityID, Entity);
}

void AToSPlayerController::UnregisterEntity(int32 EntityID)
{
    EntityRegistry.Remove(EntityID);
}

ASyncEntity* AToSPlayerController::GetEntityByID(int32 EntityID) const
{
    TWeakObjectPtr<ASyncEntity> EntityPtr = EntityRegistry.FindRef(EntityID);
    return EntityPtr.IsValid() ? EntityPtr.Get() : nullptr;
}
```

---

## Entity Spawning System

### Entity Factory

```cpp
UCLASS()
class ATOS_API UEntityFactory : public UObject
{
    GENERATED_BODY()

public:
    // Get entity class from ID
    UFUNCTION(BlueprintCallable, Category = "Entity|Factory")
    static TSubclassOf<ASyncEntity> GetEntityClass(int32 EntityID);

    // Get entity class from data table
    UFUNCTION(BlueprintCallable, Category = "Entity|Factory")
    static TSubclassOf<ASyncEntity> GetEntityClassFromDataTable(int32 EntityID, UDataTable* EntityDataTable);

protected:
    // Entity class cache
    static TMap<int32, TSubclassOf<ASyncEntity>> EntityClassCache;
};
```

---

## Blueprint Integration

### C++ to Blueprint Exposure

All critical functions are exposed to Blueprint using `UFUNCTION(BlueprintCallable)` or `UFUNCTION(BlueprintImplementableEvent)`:

**BlueprintCallable:** Functions that can be called from Blueprint
**BlueprintPure:** Functions that return values without side effects
**BlueprintImplementableEvent:** Functions implemented in Blueprint, called from C++
**BlueprintNativeEvent:** Functions with C++ default implementation, can be overridden in Blueprint

### Example Blueprint Usage

```blueprint
// In any Blueprint:
Event BeginPlay
    -> Get Game Instance
    -> Cast to ToS Game Instance
    -> Connect To Server
    -> On Authenticated (Event)
        -> Get Player Controller
        -> Cast to ToS Player Controller
        -> Spawn Player Entity
```

---

## Configuration Persistence

### Server Configuration File

**Location:** `Config/ServerConfig.json`

**Format:**
```json
{
    "ServerHost": "localhost",
    "ServerPort": 7777,
    "AuthToken": "player_auth_token_here",
    "PlayerName": "PlayerName",
    "AutoConnect": true,
    "ConnectionTimeout": 10.0
}
```

**Loading:**
- Loaded automatically on GameInstance Init
- Can be manually loaded/saved via Blueprint or C++
- Persists between game sessions

---

## See Also

- [ENTITY_SYNC.md](./ENTITY_SYNC.md) - Entity synchronization system
- [SPELLS.md](./SPELLS.md) - Spells system architecture
- [INPUT.md](./INPUT.md) - Input handling
- [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) - Player controller details

