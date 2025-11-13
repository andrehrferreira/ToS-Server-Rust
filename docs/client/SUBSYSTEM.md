# Unreal Engine Subsystem Architecture

## Overview

The Unreal Engine plugin uses a `GameInstanceSubsystem` architecture that provides global, reactive access to network data throughout the game. This architecture enables efficient development in Blueprints and allows UI systems to reactively update based on real-time network events. The subsystem is accessible from any part of the game, including Blueprints, C++ classes, and UI widgets.

## Architecture Components

**Key Components:**
1. **UENetSubsystem**: GameInstanceSubsystem that manages UDP communication
2. **UTOSGameInstance**: GameInstance that coordinates network and entity management
3. **ATOSPlayerController**: PlayerController that manages entity synchronization
4. **UDPClient**: Low-level UDP client with encryption and compression
5. **FSecureSession**: Secure session manager for encryption and handshake

### UENetSubsystem (GameInstanceSubsystem)

**Class Definition:**
```cpp
UCLASS(DisplayName = "ENetSubSystem")
class UENetSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()
    
public:
    // === CONNECTION MANAGEMENT ===
    UFUNCTION(BlueprintCallable, Category = "UDP")
    bool Connect(const FString& Host, int32 Port);
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    void Disconnect();
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    bool IsConnected() const;
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    bool IsConnecting() const;
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    EConnectionStatus GetConnectionStatus() const;
    
    // === CONNECTION CONFIGURATION ===
    UFUNCTION(BlueprintCallable, Category = "UDP")
    void SetConnectTimeout(float Seconds);
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    void SetRetryInterval(float Seconds);
    
    UFUNCTION(BlueprintCallable, Category = "UDP")
    void SetRetryEnabled(bool bEnabled);
    
    // === ENTITY SYNCHRONIZATION ===
    void SendEntitySync(FVector Position, FRotator Rotation, int32 AnimID, FVector Velocity, bool IsFalling) const;
    void SendEntitySyncQuantized(FVector Position, FRotator Rotation, int32 AnimID, FVector Velocity, bool IsFalling) const;
    
    // === NETWORK EVENTS (BlueprintAssignable) ===
    UPROPERTY(BlueprintAssignable, Category = "UDP")
    FOnConnect OnConnect;
    
    UPROPERTY(BlueprintAssignable, Category = "UDP")
    FOnDisconnected OnDisconnected;
    
    UPROPERTY(BlueprintAssignable, Category = "UDP")
    FOnConnectDenied OnConnectDenied;
    
    UPROPERTY(BlueprintAssignable, Category = "UDP")
    FOnUDPConnectionError OnConnectionError;
    
    // === ENTITY EVENTS (BlueprintAssignable) ===
    UPROPERTY(BlueprintAssignable, meta = (DisplayName = "OnCreateEntity"), Category = "UDP")
    FCreateEntityHandler OnCreateEntity;
    
    UPROPERTY(BlueprintAssignable, meta = (DisplayName = "OnUpdateEntity"), Category = "UDP")
    FUpdateEntityHandler OnUpdateEntity;
    
    UPROPERTY(BlueprintAssignable, meta = (DisplayName = "OnRemoveEntity"), Category = "UDP")
    FRemoveEntityHandler OnRemoveEntity;
    
    UPROPERTY(BlueprintAssignable, meta = (DisplayName = "OnUpdateEntityQuantized"), Category = "UDP")
    FUpdateEntityQuantizedHandler OnUpdateEntityQuantized;
    
    UPROPERTY(BlueprintAssignable, meta = (DisplayName = "OnDeltaUpdate"), Category = "UDP")
    FDeltaUpdateHandler OnDeltaUpdate;
    
private:
    TUniquePtr<class UDPClient> UdpClient;
    FTSTicker::FDelegateHandle TickHandle;
    bool bIsConnected = false;
    bool bIsConnecting = false;
    EConnectionStatus ConnectionStatus = EConnectionStatus::Disconnected;
};
```

### System of Delegates/Events

**Delegate Types:**
```cpp
// Connection Events
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnDisconnected);
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnUDPConnectionError);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnConnect, const int32&, ClientID);
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnConnectDenied);

// Entity Events
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FCreateEntityHandler, int32, EntityId, FVector, Positon, FRotator, Rotator, int32, Flags);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FUpdateEntityHandler, FUpdateEntityPacket, Data);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FRemoveEntityHandler, int32, EntityId);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FUpdateEntityQuantizedHandler, FUpdateEntityQuantizedPacket, Data);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FDeltaUpdateHandler, FDeltaUpdateData, Data);
```

**Event Broadcasting:**
```cpp
void UENetSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    UdpClient = MakeUnique<UDPClient>();
    
    // Setup UDP client callbacks
    UdpClient->OnDataReceive = [this](UFlatBuffer* Buffer)
    {
        // Process packets and broadcast events
        switch (ServerPacketType) {
            case EServerPackets::CreateEntity:
                OnCreateEntity.Broadcast(EntityId, Position, Rotator, Flags);
                break;
            case EServerPackets::UpdateEntity:
                OnUpdateEntity.Broadcast(UpdateEntityPacket);
                break;
            case EServerPackets::UpdateEntityQuantized:
                OnUpdateEntityQuantized.Broadcast(UpdateEntityQuantizedPacket);
                break;
            case EServerPackets::RemoveEntity:
                OnRemoveEntity.Broadcast(EntityId);
                break;
            // ... other packet types
        }
    };
    
    UdpClient->OnConnect = [this](int32 clientId)
    {
        OnConnect.Broadcast(clientId);
    };
    
    UdpClient->OnDisconnect = [this]()
    {
        OnDisconnected.Broadcast();
    };
}
```

### UTOSGameInstance (GameInstance)

**Class Definition:**
```cpp
UCLASS()
class UTOSGameInstance : public UGameInstance
{
    GENERATED_BODY()
    
public:
    // === NETWORK CONFIGURATION ===
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Network")
    bool ServerAutoConnect = true;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Network")
    FString ServerIP = TEXT("127.0.0.1");
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Network")
    int32 ServerPort = 3565;
    
    // === SECURITY CONFIGURATION ===
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Security")
    bool bEnableEndToEndEncryption = true;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Security")
    bool bEnableIntegrityCheck = true;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Security")
    bool bEnableLZ4Compression = true;
    
    // === PERFORMANCE CONFIGURATION ===
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Performance")
    int32 SendRateHz = 20;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Performance")
    int32 MaxPacketSize = 1200;
    
    // === ENTITY CONFIGURATION ===
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Entities")
    TSubclassOf<ASyncEntity> EntityClass;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Entities")
    TSubclassOf<ASyncPlayer> PlayerClass;
    
    // === EVENT HANDLERS ===
    UFUNCTION()
    void HandleCreateEntity(int32 EntityId, FVector Positon, FRotator Rotator, int32 Flags);
    
    UFUNCTION()
    void HandleUpdateEntity(FUpdateEntityPacket data);
    
    UFUNCTION()
    void HandleUpdateEntityQuantized(FUpdateEntityQuantizedPacket data);
    
    UFUNCTION()
    void HandleRemoveEntity(int32 EntityId);
    
    UFUNCTION()
    void HandleDeltaUpdate(FDeltaUpdateData data);
    
private:
    UPROPERTY()
    ATOSPlayerController* PlayerController = nullptr;
};
```

**Initialization:**
```cpp
void UTOSGameInstance::Init()
{
    Super::Init();
    
    // Load default configuration
    LoadDefaultConfiguration();
    
    // Setup network event handlers
    if (UENetSubsystem* Socket = GetSubsystem<UENetSubsystem>())
    {
        Socket->OnCreateEntity.AddDynamic(this, &UTOSGameInstance::HandleCreateEntity);
        Socket->OnUpdateEntity.AddDynamic(this, &UTOSGameInstance::HandleUpdateEntity);
        Socket->OnUpdateEntityQuantized.AddDynamic(this, &UTOSGameInstance::HandleUpdateEntityQuantized);
        Socket->OnRemoveEntity.AddDynamic(this, &UTOSGameInstance::HandleRemoveEntity);
        Socket->OnDeltaUpdate.AddDynamic(this, &UTOSGameInstance::HandleDeltaUpdate);
    }
}

void UTOSGameInstance::OnStart()
{
    Super::OnStart();
    
    // Auto-connect to server if enabled
    if (UENetSubsystem* Socket = GetSubsystem<UENetSubsystem>())
    {
        if (ServerAutoConnect && !Socket->IsConnected())
        {
            Socket->Connect(ServerIP, ServerPort);
        }
    }
}
```

**Event Handlers:**
```cpp
void UTOSGameInstance::HandleCreateEntity(int32 EntityId, FVector Positon, FRotator Rotator, int32 Flags)
{
    if (!PlayerController)
        return;
    
    // Execute on game thread
    AsyncTask(ENamedThreads::GameThread, [this, EntityId, Positon, Rotator, Flags]()
    {
        if (PlayerController)
        {
            PlayerController->HandleCreateEntity(EntityId, Positon, Rotator, Flags);
        }
    });
}

void UTOSGameInstance::HandleUpdateEntityQuantized(FUpdateEntityQuantizedPacket data)
{
    if (!PlayerController)
        return;
    
    // Execute on game thread
    AsyncTask(ENamedThreads::GameThread, [this, data]()
    {
        if (PlayerController)
        {
            PlayerController->HandleUpdateEntityQuantized(data);
        }
    });
}
```

### ATOSPlayerController (PlayerController)

**Class Definition:**
```cpp
UCLASS()
class ATOSPlayerController : public APlayerController
{
    GENERATED_BODY()
    
public:
    // === ENTITY MANAGEMENT ===
    UPROPERTY(BlueprintReadOnly, Category = "Entities")
    TMap<int32, ASyncEntity*> SpawnedEntities;
    
    UFUNCTION(BlueprintCallable, Category = "Entities")
    ASyncEntity* GetEntityById(int32 Id);
    
    // === ENTITY EVENT HANDLERS ===
    UFUNCTION()
    void HandleCreateEntity(int32 EntityId, FVector Positon, FRotator Rotator, int32 Flags);
    
    UFUNCTION()
    void HandleUpdateEntity(FUpdateEntityPacket data);
    
    UFUNCTION()
    void HandleUpdateEntityQuantized(FUpdateEntityQuantizedPacket data);
    
    UFUNCTION()
    void HandleRemoveEntity(int32 EntityId);
    
    UFUNCTION()
    void HandleDeltaUpdate(FDeltaUpdateData data);
    
private:
    bool bIsReadyToSync = false;
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Entities")
    TSubclassOf<ASyncEntity> EntityClass;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Entities")
    TSubclassOf<ASyncPlayer> PlayerClass;
};
```

**Entity Management:**
```cpp
void ATOSPlayerController::HandleCreateEntity(int32 EntityId, FVector Position, FRotator Rotator, int32 Flags)
{
    if (!bIsReadyToSync || SpawnedEntities.Contains(EntityId))
        return;
    
    UWorld* World = GetWorld();
    if (!World || !EntityClass)
        return;
    
    // Spawn entity
    FActorSpawnParameters SpawnParams;
    SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AdjustIfPossibleButAlwaysSpawn;
    
    ASyncEntity* Entity = World->SpawnActor<ASyncEntity>(EntityClass, Position, Rotator, SpawnParams);
    
    if (Entity)
    {
        Entity->EntityId = EntityId;
        Entity->SetFlags(static_cast<EEntityState>(Flags));
        Entity->SetActorLocation(Position);
        Entity->SetActorRotation(Rotator);
        Entity->TargetLocation = Position;
        Entity->TargetRotation = Rotator;
        
        SpawnedEntities.Add(EntityId, Entity);
    }
}

void ATOSPlayerController::HandleUpdateEntityQuantized(FUpdateEntityQuantizedPacket data)
{
    if (!bIsReadyToSync)
        return;
    
    // Dequantize position
    const float Scale = 100.0f;
    const float QuadrantSize = 25600.0f * 4;
    
    float WorldX = (data.QuadrantX * QuadrantSize) + (data.QuantizedX * Scale);
    float WorldY = (data.QuadrantY * QuadrantSize) + (data.QuantizedY * Scale);
    float WorldZ = data.QuantizedZ * Scale;
    
    FVector WorldPosition = FVector(WorldX, WorldY, WorldZ);
    FRotator WorldRotation = FRotator(0.0f, data.Yaw, 0.0f);
    
    // Update entity
    if (ASyncEntity** Found = SpawnedEntities.Find(data.EntityId))
    {
        ASyncEntity* Entity = *Found;
        if (Entity)
        {
            bool IsFalling = (data.Flags & 1) != 0;
            Entity->UpdateFromQuantizedNetwork(
                data.QuantizedX, data.QuantizedY, data.QuantizedZ,
                data.QuadrantX, data.QuadrantY, data.Yaw,
                data.Velocity, static_cast<uint32>(data.AnimationState), IsFalling
            );
        }
    }
    else if (EntityClass)
    {
        // Spawn entity if it doesn't exist
        UWorld* World = GetWorld();
        if (World)
        {
            ASyncEntity* NewEntity = World->SpawnActor<ASyncEntity>(EntityClass, WorldPosition, WorldRotation);
            if (NewEntity)
            {
                NewEntity->EntityId = data.EntityId;
                SpawnedEntities.Add(data.EntityId, NewEntity);
            }
        }
    }
}
```

### Threading Architecture

**Packet Polling Thread:**
```cpp
class FPacketPollRunnable : public FRunnable
{
public:
    FPacketPollRunnable(UDPClient* InClient) : Client(InClient), bStop(false) {}
    
    virtual uint32 Run() override
    {
        while (!bStop)
        {
            if (Client)
            {
                Client->PollIncomingPackets();
                Client->UpdateReliablePackets();
                Client->ProcessReliableQueue();
                Client->ProcessUnreliableQueue();
            }
            
            FPlatformProcess::Sleep(0.001f); // 1ms sleep
        }
        
        return 0;
    }
    
    void Stop() { bStop = true; }
    
private:
    UDPClient* Client;
    bool bStop;
};

void UDPClient::StartPacketPollThread()
{
    StopPacketPollThread();
    PacketPollRunnable = new FPacketPollRunnable(this);
    PacketPollThread = FRunnableThread::Create(PacketPollRunnable, TEXT("UDPClientPacketPollThread"));
}
```

**Thread-Safe Event Broadcasting:**
```cpp
void UDPClient::PollIncomingPackets()
{
    // Poll packets in background thread
    while (Socket->HasPendingData(PendingDataSize))
    {
        // Receive packet
        TArray<uint8> ReceivedData;
        Socket->RecvFrom(ReceivedData.GetData(), ReceivedData.Num(), BytesRead, *Sender);
        
        // Process packet
        if (bIsEncryptedPacket)
        {
            ProcessEncryptedPacket(Buffer, BytesRead, Header);
        }
        else
        {
            // Queue for game thread processing
            UnreliableEventQueue.Enqueue(Plaintext);
        }
    }
}

void UDPClient::ProcessUnreliableQueue()
{
    TArray<uint8> Data;
    while (UnreliableEventQueue.Dequeue(Data))
    {
        if (OnDataReceive)
        {
            // Create buffer and process on game thread
            UFlatBuffer* Buffer = UFlatBuffer::CreateFlatBuffer(Data.Num());
            Buffer->CopyFromMemory(Data.GetData(), Data.Num());
            
            // Process on game thread
            AsyncTask(ENamedThreads::GameThread, [this, Buffer]()
            {
                if (OnDataReceive)
                {
                    OnDataReceive(Buffer);
                }
            });
        }
    }
}
```

### Handshake System

**Connection Handshake:**
```cpp
bool UDPClient::Connect(const FString& Host, int32 Port)
{
    // Generate client keys
    ClientPublicKey.SetNumUninitialized(32);
    ClientPrivateKey.SetNumUninitialized(32);
    crypto_box_keypair(ClientPublicKey.GetData(), ClientPrivateKey.GetData());
    
    // Send connection request
    TArray<uint8> Packet;
    Packet.Add(static_cast<uint8>(EPacketType::Connect));
    Packet.Append(ClientPublicKey.GetData(), ClientPublicKey.Num());
    Socket->SendTo(Packet.GetData(), Packet.Num(), BytesSent, *RemoteEndpoint);
    
    return true;
}

void UDPClient::ProcessConnectionAccepted(UFlatBuffer* Buffer)
{
    // Receive server public key and salt
    uint32 ConnectionId = Buffer->ReadUInt32();
    ServerPublicKey.SetNumUninitialized(32);
    Salt.SetNumUninitialized(16);
    
    for (int32 i = 0; i < 32; ++i)
        ServerPublicKey[i] = Buffer->ReadByte();
    
    for (int32 i = 0; i < 16; ++i)
        Salt[i] = Buffer->ReadByte();
    
    // Initialize secure session
    if (SecureSession.InitializeAsClient(ClientPrivateKey, ServerPublicKey, Salt, ConnectionId))
    {
        bEncryptionEnabled = true;
        
        // Send crypto test
        ClientTestValue = 0xA1B2C3D4;
        UFlatBuffer* TestBuffer = UFlatBuffer::CreateFlatBuffer(8);
        TestBuffer->WriteByte(static_cast<uint8>(EPacketType::CryptoTest));
        TestBuffer->WriteUInt32(ClientTestValue);
        SendLegacy(TestBuffer);
    }
}

void UDPClient::ProcessCryptoTestAck(UFlatBuffer* Buffer)
{
    uint32 value = Buffer->ReadUInt32();
    if (value == ClientTestValue)
    {
        bClientCryptoConfirmed = true;
        
        if (IsCryptoReady())
        {
            // Send reliable handshake
            UFlatBuffer* HandshakeBuffer = UFlatBuffer::CreateFlatBuffer(2);
            HandshakeBuffer->WriteByte(static_cast<uint8>(EPacketType::ReliableHandshake));
            HandshakeBuffer->WriteByte(0x01);
            SendLegacy(HandshakeBuffer);
            
            if (OnConnect)
                OnConnect(SecureSession.GetConnectionId());
        }
    }
}
```

### Blueprint Integration

**Accessing Subsystem in Blueprints:**
```cpp
// In Blueprint:
// 1. Get Game Instance
UTOSGameInstance* GameInstance = Cast<UTOSGameInstance>(GetGameInstance());

// 2. Get Subsystem
UENetSubsystem* NetSubsystem = GameInstance->GetSubsystem<UENetSubsystem>();

// 3. Bind to events
NetSubsystem->OnCreateEntity.AddDynamic(this, &UMyWidget::HandleCreateEntity);
NetSubsystem->OnUpdateEntity.AddDynamic(this, &UMyWidget::HandleUpdateEntity);
NetSubsystem->OnRemoveEntity.AddDynamic(this, &UMyWidget::HandleRemoveEntity);

// 4. Call functions
NetSubsystem->Connect(TEXT("127.0.0.1"), 3565);
bool IsConnected = NetSubsystem->IsConnected();
```

**Blueprint Event Handlers:**
```cpp
// In Blueprint Widget or Actor:
UFUNCTION(BlueprintCallable, Category = "Network")
void UMyWidget::HandleCreateEntity(int32 EntityId, FVector Position, FRotator Rotator, int32 Flags)
{
    // Update UI with new entity
    UE_LOG(LogTemp, Warning, TEXT("Entity Created: %d"), EntityId);
}

UFUNCTION(BlueprintCallable, Category = "Network")
void UMyWidget::HandleUpdateEntity(FUpdateEntityPacket Data)
{
    // Update UI with entity data
    UE_LOG(LogTemp, Warning, TEXT("Entity Updated: %d"), Data.EntityId);
}

UFUNCTION(BlueprintCallable, Category = "Network")
void UMyWidget::HandleRemoveEntity(int32 EntityId)
{
    // Remove entity from UI
    UE_LOG(LogTemp, Warning, TEXT("Entity Removed: %d"), EntityId);
}
```

### UI Integration

**Reactive UI Updates:**
```cpp
UCLASS()
class UEntityListWidget : public UUserWidget
{
    GENERATED_BODY()
    
public:
    virtual void NativeConstruct() override
    {
        Super::NativeConstruct();
        
        // Get subsystem
        if (UTOSGameInstance* GameInstance = Cast<UTOSGameInstance>(GetGameInstance()))
        {
            if (UENetSubsystem* NetSubsystem = GameInstance->GetSubsystem<UENetSubsystem>())
            {
                // Bind to events
                NetSubsystem->OnCreateEntity.AddDynamic(this, &UEntityListWidget::OnEntityCreated);
                NetSubsystem->OnUpdateEntity.AddDynamic(this, &UEntityListWidget::OnEntityUpdated);
                NetSubsystem->OnRemoveEntity.AddDynamic(this, &UEntityListWidget::OnEntityRemoved);
                NetSubsystem->OnConnect.AddDynamic(this, &UEntityListWidget::OnConnected);
                NetSubsystem->OnDisconnected.AddDynamic(this, &UEntityListWidget::OnDisconnected);
            }
        }
    }
    
    UFUNCTION()
    void OnEntityCreated(int32 EntityId, FVector Position, FRotator Rotator, int32 Flags)
    {
        // Add entity to list
        EntityIds.Add(EntityId);
        RefreshEntityList();
    }
    
    UFUNCTION()
    void OnEntityUpdated(FUpdateEntityPacket Data)
    {
        // Update entity in list
        RefreshEntityList();
    }
    
    UFUNCTION()
    void OnEntityRemoved(int32 EntityId)
    {
        // Remove entity from list
        EntityIds.Remove(EntityId);
        RefreshEntityList();
    }
    
    UFUNCTION()
    void OnConnected(int32 ClientId)
    {
        // Update connection status
        ConnectionStatusText->SetText(FText::FromString(TEXT("Connected")));
    }
    
    UFUNCTION()
    void OnDisconnected()
    {
        // Update connection status
        ConnectionStatusText->SetText(FText::FromString(TEXT("Disconnected")));
    }
    
private:
    UPROPERTY(meta = (BindWidget))
    class UTextBlock* ConnectionStatusText;
    
    UPROPERTY(meta = (BindWidget))
    class UListView* EntityListView;
    
    TArray<int32> EntityIds;
    
    void RefreshEntityList()
    {
        // Refresh UI list with current entities
        if (EntityListView)
        {
            EntityListView->ClearListItems();
            for (int32 EntityId : EntityIds)
            {
                // Add entity to list
                UEntityListItem* Item = NewObject<UEntityListItem>();
                Item->EntityId = EntityId;
                EntityListView->AddItem(Item);
            }
        }
    }
};
```

### Configuration System

**Client Configuration:**
```cpp
UCLASS()
class UClientConfig : public UObject
{
    GENERATED_BODY()
    
public:
    // Network Configuration
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Network")
    FNetworkConfig Network;
    
    // Security Configuration
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Security")
    FSecurityConfig Security;
    
    // Performance Configuration
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Performance")
    FPerformanceConfig Performance;
    
    // Logging Configuration
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Logging")
    FLoggingConfig Logging;
    
    UFUNCTION(BlueprintCallable, Category = "Configuration")
    bool ValidateConfiguration() const;
    
    UFUNCTION(BlueprintCallable, Category = "Configuration")
    static UClientConfig* GetDefaultConfig();
};

void UTOSGameInstance::LoadDefaultConfiguration()
{
    UClientConfig* DefaultConfig = UClientConfig::GetDefaultConfig();
    if (DefaultConfig)
    {
        ApplyClientConfiguration(DefaultConfig);
    }
}

void UTOSGameInstance::ApplyClientConfiguration(UClientConfig* Config)
{
    if (!Config || !Config->ValidateConfiguration())
        return;
    
    // Apply network settings
    ServerIP = Config->Network.ServerIP;
    ServerPort = Config->Network.ServerPort;
    ServerAutoConnect = Config->Network.bAutoConnect;
    
    // Apply security settings
    bEnableEndToEndEncryption = Config->Security.bEnableEndToEndEncryption;
    bEnableIntegrityCheck = Config->Security.bEnableIntegrityCheck;
    bEnableLZ4Compression = Config->Security.bEnableLZ4Compression;
    
    // Apply performance settings
    SendRateHz = Config->Performance.SendRateHz;
    MaxPacketSize = Config->Performance.MaxPacketSize;
    ReliableTimeoutMs = Config->Performance.ReliableTimeoutMs;
    MaxRetries = Config->Performance.MaxRetries;
    
    // Apply to ENetSubsystem
    if (UENetSubsystem* NetSubsystem = GetSubsystem<UENetSubsystem>())
    {
        NetSubsystem->SetConnectTimeout(Config->Network.ConnectionTimeoutSeconds);
        NetSubsystem->SetRetryInterval(Config->Network.RetryIntervalSeconds);
        NetSubsystem->SetRetryEnabled(Config->Network.bEnableRetry);
    }
}
```

### Benefits of Subsystem Architecture

**1. Global Access:**
- Accessible from any part of the game (Blueprints, C++, UI)
- No need to pass references between classes
- Singleton-like behavior without singleton drawbacks

**2. Reactive Programming:**
- Event-driven architecture with delegates
- UI updates automatically when network events occur
- No polling required for UI updates

**3. Blueprint Integration:**
- All functions exposed to Blueprints with `UFUNCTION(BlueprintCallable)`
- All events exposed with `UPROPERTY(BlueprintAssignable)`
- Easy to use in Blueprints without C++ knowledge

**4. Thread Safety:**
- Packet polling in background thread
- Events queued and processed on game thread
- No race conditions between network and game threads

**5. Modularity:**
- Separation of concerns (Network, GameInstance, PlayerController)
- Easy to extend with new event types
- Easy to test individual components

**6. Configuration:**
- Centralized configuration in GameInstance
- Easy to modify settings in editor
- Validation and default values

### Best Practices

**1. Event Binding:**
- Bind events in `NativeConstruct` or `BeginPlay`
- Unbind events in `NativeDestruct` or `EndPlay`
- Use weak references to avoid circular dependencies

**2. Thread Safety:**
- Always process network events on game thread
- Use `AsyncTask(ENamedThreads::GameThread, ...)` for thread switching
- Never access UObject pointers from background threads

**3. Error Handling:**
- Check if subsystem exists before using
- Handle connection errors gracefully
- Validate configuration before applying

**4. Performance:**
- Minimize work in event handlers
- Use object pooling for frequently created/destroyed objects
- Batch UI updates to avoid excessive redraws

**5. Debugging:**
- Use logging for network events
- Enable debug logs in development builds
- Use breakpoints in event handlers for debugging

## Conclusion

The Unreal Engine subsystem architecture provides a powerful, reactive system for network communication that enables efficient development in Blueprints and allows UI systems to reactively update based on real-time network events. The architecture is designed to be:

- **Global**: Accessible from any part of the game
- **Reactive**: Event-driven with automatic UI updates
- **Thread-Safe**: Background packet polling with game thread processing
- **Modular**: Separation of concerns for easy extension
- **Configurable**: Centralized configuration with validation

This approach enables efficient development in Blueprints and allows UI systems to reactively update based on real-time network events, making it ideal for MMORPG client development.

