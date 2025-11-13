# Trade System Documentation

## Overview

The trade system allows players to safely exchange items and gold with each other. The system uses a secure two-phase acceptance mechanism where both players must accept the trade before items are exchanged. The system includes validation, cancellation, and proper cleanup to prevent exploits and ensure fair trading.

## Key Features

- **Secure Trading**: Two-phase acceptance system (both players must accept)
- **Item Exchange**: Trade items between players
- **Gold Exchange**: Trade gold between players
- **Real-time Updates**: Containers synchronized in real-time
- **Movement Validation**: Trade cancelled if players move
- **Timeout Protection**: Trade requests expire after timeout
- **Proper Cleanup**: Items returned to inventory on cancellation
- **State Validation**: Players must be alive and not destroyed

## Trade Flow

### Complete Trade Process

**1. Trade Request:**
- Player A initiates trade request with Player B
- System validates both players
- Request sent to Player B
- Request expires after timeout (10 seconds)

**2. Trade Acceptance:**
- Player B accepts trade request
- Trade session created
- Trade windows opened for both players
- Containers created and synchronized

**3. Trade Negotiation:**
- Both players add items/gold to trade
- Real-time container updates
- Both players can accept/reject trade
- Trade completes when both accept

**4. Trade Completion:**
- Items and gold exchanged
- Trade windows closed
- Players saved
- Trade session destroyed

## Trade Session Structure

### SafeTrade Class

```typescript
class SafeTrade {
    id: string;                    // Unique trade session ID
    owner: Player;                 // Player who initiated trade
    otherPlayer: Player;            // Other player in trade
    requestCharacterId: string;    // Character ID of requested player
    
    ownerContainer: Container;     // Owner's trade container
    otherContainer: Container;     // Other player's trade container
    
    ownerAccept: boolean;          // Owner's acceptance status
    otherAccept: boolean;         // Other player's acceptance status
}
```

### Trade State Management

**Static Sessions:**
- All active trades stored in `SafeTrade.tradeSessions` Map
- Key: Trade session ID
- Value: SafeTrade instance

**Session Lifecycle:**
1. Created when trade request accepted
2. Active during trade negotiation
3. Destroyed on completion or cancellation

## Trade Request System

### Request Trade

**Process:**
1. Player initiates trade request
2. System validates target player exists
3. System checks if target player is already trading
4. Request sent to target player
5. Request expires after timeout

**Implementation:**
```typescript
public requestTrade(characterId: string): boolean {
    if (!Player.players.has(characterId)) {
        return false; // Player not found
    }
    
    const player = Player.players.get(characterId);
    
    if (!player) {
        return false; // Player not found
    }
    
    if (player.trade) {
        packetSystemMessage.sendDirectSocket(
            this.owner.socket, 
            `The player is already in another trade.`
        );
        return false; // Already trading
    }
    
    this.requestCharacterId = characterId;
    packetRequestTrade.send(player, this.id, this.owner.name);
    
    return true;
}
```

### Accept Trade Request

**Process:**
1. Target player receives trade request
2. Player accepts trade request
3. Trade session validated
4. Containers created
5. Trade windows opened

**Implementation:**
```typescript
public acceptTrade(player: Player): boolean {
    if (player.characterId !== this.requestCharacterId) {
        return false; // Not the requested player
    }
    
    // Validate players
    if (player.isDead || player.isDestroyed) {
        return false; // Player invalid
    }
    
    if (this.owner.isDead || this.owner.isDestroyed) {
        return false; // Owner invalid
    }
    
    this.otherPlayer = player;
    
    // Create containers
    this.ownerContainer = new Container(this.owner);
    this.ownerContainer.addObserver(this.otherPlayer);
    Containers.set(this.ownerContainer.containerId, this.ownerContainer);
    
    this.otherContainer = new Container(this.otherPlayer);
    this.otherContainer.addObserver(this.owner);
    Containers.set(this.otherContainer.containerId, this.otherContainer);
    
    // Set trade references
    this.owner.trade = this;
    this.otherPlayer.trade = this;
    
    // Stop player movement
    this.owner.stopMovement();
    this.otherPlayer.stopMovement();
    
    // Open trade windows
    packetOpenTradeWindow.send(this.owner, {
        tradeId: this.id,
        ownerContainer: this.ownerContainer.containerId,
        otherContainer: this.otherContainer.containerId,
        isOwner: true
    });
    
    packetOpenTradeWindow.send(this.otherPlayer, {
        tradeId: this.id,
        ownerContainer: this.ownerContainer.containerId,
        otherContainer: this.otherContainer.containerId,
        isOwner: false
    });
    
    return true;
}
```

### Reject Trade Request

**Process:**
1. Target player rejects trade request
2. Owner notified
3. Trade session destroyed

**Implementation:**
```typescript
public notAccept(player: Player): void {
    if (player.characterId === this.requestCharacterId) {
        packetSystemMessage.sendDirectSocket(
            this.owner.socket, 
            `The other player does not accept starting a trade with you.`
        );
        
        SafeTrade.destroySession(this.id);
    }
}
```

## Trade Negotiation

### Adding Items

**Process:**
1. Player drags item to trade container
2. Item added to player's trade container
3. Container observer notified
4. Other player sees item update
5. Both acceptance states reset to false

**Container Observer:**
- Containers use observer pattern
- Changes in one container notify other player
- Real-time synchronization

**Acceptance Reset:**
- When items added/removed, both players' acceptance reset
- Prevents accepting outdated trade state
- Ensures both players review changes

### Adding Gold

**Process:**
1. Player sets gold amount in trade
2. System validates player has enough gold
3. Gold amount updated
4. Other player sees gold update
5. Both acceptance states reset to false

**Implementation:**
```typescript
public setTradeGold(player: Player, goldAmount: number): boolean {
    if (!this.otherPlayer) {
        return false; // Trade not active
    }
    
    // Validate player is in trade
    if (player.characterId !== this.owner.characterId && 
        player.characterId !== this.otherPlayer.characterId) {
        return false;
    }
    
    // Reset acceptance
    this.ownerAccept = false;
    this.otherAccept = false;
    
    // Validate and set gold
    if (player.characterId === this.owner.characterId) {
        const maxGold = Math.min(goldAmount, this.owner.gold);
        this.ownerTradeGold = maxGold;
    } else {
        const maxGold = Math.min(goldAmount, this.otherPlayer.gold);
        this.otherTradeGold = maxGold;
    }
    
    // Notify both players
    packetChangeTradeGold.send(this.owner, {
        ownerGold: this.ownerTradeGold,
        otherGold: this.otherTradeGold
    });
    
    packetChangeTradeGold.send(this.otherPlayer, {
        ownerGold: this.ownerTradeGold,
        otherGold: this.otherTradeGold
    });
    
    return true;
}
```

## Trade Acceptance

### Change Acceptance Status

**Process:**
1. Player toggles acceptance checkbox
2. System updates acceptance status
3. Other player notified of status change
4. If both accept: Trade completes
5. If either unaccepts: Trade resets

**Implementation:**
```typescript
public changeStatus(player: Player, status: boolean): void {
    if (player.characterId === this.owner.characterId) {
        this.ownerAccept = status;
        packetChangeStatusTrade.send(this.otherPlayer, status);
    } else if (this.otherPlayer && player.characterId === this.otherPlayer.characterId) {
        this.otherAccept = status;
        packetChangeStatusTrade.send(this.owner, status);
    }
    
    // Check if both accepted
    if (this.ownerAccept && this.otherAccept) {
        this.finishTrade();
    }
}
```

### Acceptance Delay

**Security Feature:**
- After accepting, 2-second delay before trade can complete
- Prevents accidental acceptance
- Gives time to review trade
- Can unaccept during delay

**Implementation:**
```typescript
private canCompleteTrade(): boolean {
    if (!this.ownerAccept || !this.otherAccept) {
        return false; // Not both accepted
    }
    
    // Check delay (2 seconds after last acceptance)
    const now = new Date().getTime();
    const lastAcceptanceTime = Math.max(this.ownerAcceptTime, this.otherAcceptTime);
    
    if (now - lastAcceptanceTime < 2000) {
        return false; // Delay not passed
    }
    
    return true;
}
```

## Trade Completion

### Finish Trade

**Process:**
1. Both players have accepted
2. System validates trade state
3. Items transferred between players
4. Gold transferred between players
5. Trade windows closed
6. Players saved
7. Trade session destroyed

**Validation:**
- Both players must be alive
- Both players must not be destroyed
- Both players must have enough gold
- Items must still exist in containers

**Implementation:**
```typescript
public async finishTrade(): Promise<void> {
    // Validate trade state
    if (!this.validateTradeCompletion()) {
        this.cancelTrade(this.owner);
        return;
    }
    
    // Transfer items
    await this.ownerContainer.changeOwner(this.otherPlayer).sendToInventory();
    await this.otherContainer.changeOwner(this.owner).sendToInventory();
    
    // Transfer gold
    if (this.ownerTradeGold > 0) {
        if (this.owner.gold >= this.ownerTradeGold) {
            this.owner.gold -= this.ownerTradeGold;
            this.otherPlayer.gold += this.ownerTradeGold;
        }
    }
    
    if (this.otherTradeGold > 0) {
        if (this.otherPlayer.gold >= this.otherTradeGold) {
            this.otherPlayer.gold -= this.otherTradeGold;
            this.owner.gold += this.otherTradeGold;
        }
    }
    
    // Close windows
    packetCloseWindow.send(this.otherPlayer, WindowType.Trade);
    packetCloseWindow.send(this.owner, WindowType.Trade);
    
    // Notify players
    packetSystemMessage.sendDirectSocket(
        this.otherPlayer.socket, 
        `Trade completed successfully!`
    );
    packetSystemMessage.sendDirectSocket(
        this.owner.socket, 
        `Trade completed successfully!`
    );
    
    // Save players
    this.otherPlayer.save();
    this.otherPlayer.saveToDatabase();
    this.owner.save();
    this.owner.saveToDatabase();
    
    // Destroy trade session
    SafeTrade.destroySession(this.id);
}

private validateTradeCompletion(): boolean {
    // Check players are valid
    if (this.owner.isDead || this.owner.isDestroyed) {
        return false;
    }
    
    if (this.otherPlayer.isDead || this.otherPlayer.isDestroyed) {
        return false;
    }
    
    // Check gold availability
    if (this.ownerTradeGold > 0 && this.owner.gold < this.ownerTradeGold) {
        return false;
    }
    
    if (this.otherTradeGold > 0 && this.otherPlayer.gold < this.otherTradeGold) {
        return false;
    }
    
    // Check containers still exist
    if (!this.ownerContainer || !this.otherContainer) {
        return false;
    }
    
    return true;
}
```

## Trade Cancellation

### Cancel Trade

**Process:**
1. Player cancels trade (closes window, moves, etc.)
2. Items returned to original owners
3. Gold not transferred
4. Trade windows closed
5. Players notified
6. Trade session destroyed

**Cancellation Triggers:**
- Player closes trade window
- Player moves during trade
- Player disconnects
- Trade timeout expires
- Invalid trade state detected

**Implementation:**
```typescript
public async cancelTrade(player: Player): Promise<void> {
    if (!this.otherPlayer) {
        return; // Trade not active
    }
    
    // Validate player is in trade
    if (player.characterId !== this.owner.characterId && 
        player.characterId !== this.otherPlayer.characterId) {
        return;
    }
    
    // Notify both players
    packetSystemMessage.sendDirectSocket(
        this.otherPlayer.socket, 
        `The trade was cancelled by one of the players.`
    );
    packetSystemMessage.sendDirectSocket(
        this.owner.socket, 
        `The trade was cancelled by one of the players.`
    );
    
    // Close windows
    packetCloseWindow.send(this.otherPlayer, WindowType.Trade);
    packetCloseWindow.send(this.owner, WindowType.Trade);
    
    // Return items to inventory
    if (this.ownerContainer && this.ownerContainer.count() > 0) {
        await this.ownerContainer.sendToInventory();
    }
    
    if (this.otherContainer && this.otherContainer.count() > 0) {
        await this.otherContainer.sendToInventory();
    }
    
    // Save players
    this.otherPlayer.save();
    this.owner.save();
    
    // Destroy trade session
    SafeTrade.destroySession(this.id);
}
```

## Movement Validation

### Position Change Detection

**Security Feature:**
- Trade cancelled if either player moves
- Prevents trade exploits
- Ensures players are in range
- Movement detection via position change events

**Implementation:**
```typescript
private onPlayerPositionChanged(player: Player, from: Vector3, to: Vector3): void {
    if (!this.otherPlayer) {
        return; // Trade not active
    }
    
    // Check if player is in trade
    if (player.characterId !== this.owner.characterId && 
        player.characterId !== this.otherPlayer.characterId) {
        return;
    }
    
    // Calculate distance moved
    const distance = from.distanceTo(to);
    
    if (distance > 50) { // Threshold for movement
        // Cancel trade
        this.cancelTrade(player);
    }
}
```

**Movement Prevention:**
- Players stopped when trade starts
- Movement blocked during trade
- Trade cancelled if movement detected

## Container Synchronization

### Observer Pattern

**Container Observers:**
- Each trade container has observer
- Observer is the other player
- Changes in container notify observer
- Real-time synchronization

**Implementation:**
```typescript
// Owner container observes other player
this.ownerContainer.addObserver(this.otherPlayer);

// Other container observes owner
this.otherContainer.addObserver(this.owner);
```

**Synchronization:**
- Item added: Other player sees item
- Item removed: Other player sees removal
- Item moved: Other player sees move
- Gold changed: Other player sees gold update

## Trade Validation

### Pre-Trade Validation

**Before Trade Starts:**
- Both players must exist
- Both players must be alive
- Both players must not be destroyed
- Target player must not be trading
- Players must be in range (optional)

**During Trade:**
- Players must remain alive
- Players must not be destroyed
- Players must have enough gold
- Items must exist in containers
- Players must not move significantly

### Post-Trade Validation

**After Trade Completes:**
- Items successfully transferred
- Gold successfully transferred
- Players saved to database
- Trade session cleaned up

## Network Packets

### Trade Packets

**Request Trade Packet:**
```typescript
interface RequestTradePacket {
    tradeId: string;
    requesterName: string;
}
```

**Open Trade Window Packet:**
```typescript
interface OpenTradeWindowPacket {
    tradeId: string;
    ownerContainer: string;
    otherContainer: string;
    isOwner: boolean;
}
```

**Change Status Trade Packet:**
```typescript
interface ChangeStatusTradePacket {
    accepted: boolean;
}
```

**Change Trade Gold Packet:**
```typescript
interface ChangeTradeGoldPacket {
    ownerGold: number;
    otherGold: number;
}
```

**Close Window Packet:**
```typescript
interface CloseWindowPacket {
    windowType: WindowType;
}
```

## Security Features

### Exploit Prevention

**1. Double Acceptance:**
- Both players must accept
- Acceptance can be toggled
- Trade only completes when both accept

**2. Movement Detection:**
- Trade cancelled if players move
- Prevents range exploits
- Ensures players stay in place

**3. State Validation:**
- Players must be alive
- Players must not be destroyed
- Gold must be available
- Items must exist

**4. Timeout Protection:**
- Trade requests expire after 10 seconds
- Prevents hanging requests
- Cleans up stale sessions

**5. Acceptance Delay:**
- 2-second delay after acceptance
- Prevents accidental completion
- Allows time to review

**6. Item Return:**
- Items returned on cancellation
- Prevents item loss
- Ensures fair trading

## Error Handling

### Error Scenarios

**1. Player Not Found:**
- Trade request fails
- Error message to requester

**2. Player Already Trading:**
- Trade request rejected
- Error message to requester

**3. Insufficient Gold:**
- Gold amount capped to available
- Trade can still proceed with available gold

**4. Player Moves:**
- Trade automatically cancelled
- Items returned to owners

**5. Player Disconnects:**
- Trade automatically cancelled
- Items returned to owner
- Other player notified

**6. Invalid Trade State:**
- Trade cancelled
- Items returned
- Error logged

## Performance Considerations

### Optimization Strategies

**1. Session Management:**
- Active sessions cached in memory
- Sessions cleaned up immediately after use
- Prevents memory leaks

**2. Container Updates:**
- Batch container updates
- Reduce network packets
- Optimize synchronization

**3. Validation Caching:**
- Cache validation results
- Reduce redundant checks
- Improve performance

## Integration with Other Systems

### Inventory System
- Items moved between trade containers and inventory
- Stackable item handling
- Weight management

### Container System
- Trade uses container system
- Observer pattern for synchronization
- Proper cleanup on trade end

### Player System
- Trade state stored on player
- Movement detection
- State validation

### Network System
- Trade packets sent reliably
- Proper packet ordering
- Error handling

## Testing Considerations

### Unit Tests
- Trade request/accept flow
- Item transfer validation
- Gold transfer validation
- Acceptance state management
- Cancellation flow

### Integration Tests
- Full trade flow
- Movement cancellation
- Disconnect handling
- Multiple simultaneous trades
- Container synchronization

### Security Tests
- Exploit prevention
- State validation
- Movement detection
- Timeout handling
- Item return on cancellation

## Summary

The trade system provides:

- **Secure Trading**: Two-phase acceptance system
- **Item Exchange**: Safe item transfer between players
- **Gold Exchange**: Gold trading with validation
- **Real-time Sync**: Container synchronization
- **Security**: Movement detection and state validation
- **Proper Cleanup**: Items returned on cancellation
- **Error Handling**: Comprehensive error handling
- **Performance**: Optimized session management

The system ensures fair and secure trading between players while preventing exploits and ensuring proper resource management.

