# Client Packets Documentation

## Overview

Client packets are sent from the client to the server, representing player actions, interactions, and requests. Each packet type has a corresponding handler on the server that processes the packet and performs the appropriate game logic.

## Movement Packets

### Move

**Type:** `ClientPacketType.Move`

**Purpose:** Send player movement input

**Handler:** `MovePacketHandler`

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates position (not NaN)
- Updates player movement direction
- Server calculates movement
- Server validates movement
- Server updates position

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (!float.IsNaN(x) && !float.IsNaN(y) && player != null) {
    player.Move(new Vector2(x, y));
}
```

### MoveTo

**Type:** `ClientPacketType.MoveTo`

**Purpose:** Request movement to specific position

**Handler:** `MoveToPacketHandler`

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates position
- Initiates pathfinding
- Moves player to target
- Handles obstacles
- Updates pathfinding

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (!float.IsNaN(x) && !float.IsNaN(y) && player != null) {
    player.MoveTo(new Vector2(x, y));
}
```

### Rotate

**Type:** `ClientPacketType.Rotate`

**Purpose:** Send player rotation input

**Handler:** `RotatePacketHandler`

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates rotation target
- Checks if player can rotate
- Updates player rotation
- Rotates towards target position

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (player != null && !player.IsMovingTo && player.CanRotate == 0) {
    player.RotateTowards(new Vector2(x, y));
}
```

## Action Packets

### Cast

**Type:** `ClientPacketType.Cast`

**Purpose:** Execute action at position

**Handler:** `CastPacketHandler`

**Data:**
```
[ActionName: symbol]
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates action exists
- Checks action cooldown
- Validates resources
- Executes action at position
- Applies action effects

**Example:**
```csharp
string name = buffer.GetSymbol();
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (player.Actions.TryGetValue(name, out Action action)) {
    player.Execute(action, new Vector2(x, y));
}
```

### CastTarget

**Type:** `ClientPacketType.CastTarget`

**Purpose:** Execute action on target entity

**Handler:** `CastTargetPacketHandler`

**Data:**
```
[ActionName: symbol]
[EntityId: int]
```

**Handler Logic:**
- Validates action exists
- Validates target entity
- Checks action range
- Executes action on target
- Applies action effects

**Example:**
```csharp
string name = buffer.GetSymbol();
int id = buffer.GetInt();

if (TryGetEntityById(id, out Entity entity) && 
    player.Actions.TryGetValue(name, out Action action)) {
    player.Execute(action, entity);
}
```

### AutoAttack

**Type:** `ClientPacketType.AutoAttack`

**Purpose:** Trigger auto-attack

**Handler:** `AutoAttackPacketHandler`

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates attack position
- Executes primary action
- Handles attack cooldown
- Processes attack logic
- Calculates damage

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

player.Execute(player.PrimaryAction, new Vector2(x, y));
```

### SetActionbarAction

**Type:** `ClientPacketType.SetActionbarAction`

**Purpose:** Set actionbar slot to action

**Handler:** `SetActionbarActionHandler`

**Data:**
```
[SlotIndex: varuint]
[ActionName: symbol]
```

**Handler Logic:**
- Validates slot index
- Validates action exists
- Creates action instance
- Sets actionbar slot
- Updates actionbar

**Example:**
```csharp
uint slotIndex = buffer.GetVarUInt();
string actionName = buffer.GetSymbol();

if (slotIndex >= 1) {
    Action action = Action.CreateInstance(player, actionName);
    if (action != null) {
        player.SetActionbarObject(slotIndex, action);
    }
}
```

### SetActionbarItem

**Type:** `ClientPacketType.SetActionbarItem`

**Purpose:** Set actionbar slot to consumable item

**Handler:** `SetActionbarItemHandler`

**Data:**
```
[SlotIndex: varuint]
[ItemId: varuint]
```

**Handler Logic:**
- Validates slot index
- Validates item exists
- Checks if item is consumable
- Sets actionbar slot
- Updates actionbar

**Example:**
```csharp
uint slotIndex = buffer.GetVarUInt();
uint itemId = buffer.GetVarUInt();

if (slotIndex >= 1) {
    if (player.Inventory.TryGetValue(itemId, out Item item) && item is Consumable) {
        player.SetActionbarObject(slotIndex, new ItemReference(item, slotIndex, itemId, player.Inventory, player));
    }
}
```

## Interaction Packets

### Interact

**Type:** `ClientPacketType.Interact`

**Purpose:** Interact with entity

**Handler:** `InteractPacketHandler`

**Data:**
```
[EntityId: int]
```

**Handler Logic:**
- Validates player not dead
- Validates entity exists
- Checks interaction range
- Processes interaction
- Opens appropriate interface

**Example:**
```csharp
int id = buffer.GetInt();

if (!player.IsDead && TryGetEntityById(id, out Entity entity)) {
    player.InteractWith(entity);
}
```

### Collect

**Type:** `ClientPacketType.Collect`

**Purpose:** Start gathering resource

**Data:**
```
[ResourceId: string]
```

**Handler Logic:**
- Validates resource exists
- Checks gathering range
- Validates gathering requirements
- Starts gathering process
- Updates gathering state

**Example:**
```csharp
string resourceId = buffer.GetString();
// Start gathering
```

### Skinning

**Type:** `ClientPacketType.Skinning`

**Purpose:** Start skinning creature

**Data:**
```
[CreatureId: int]
```

**Handler Logic:**
- Validates creature exists
- Checks if creature is dead
- Validates skinning range
- Starts skinning process
- Grants loot

**Example:**
```csharp
int creatureId = buffer.GetInt();
// Start skinning
```

## Target Packets

### SetTargetEntity

**Type:** `ClientPacketType.SetTargetEntity`

**Purpose:** Set target entity

**Handler:** `SetTargetEntityHandler`

**Data:**
```
[EntityId: int]
```

**Handler Logic:**
- Validates entity exists
- Checks target range
- Sets player target
- Updates target state
- Sends target update

**Example:**
```csharp
int id = buffer.GetInt();

if (TryGetEntityById(id, out int index)) {
    ref Entry entry = ref UnreliableEntities.Values[index];
    TargetRequestCompletionSource.SetResult(entry.Entity);
}
```

### SetTargetPosition

**Type:** `ClientPacketType.SetTargetPosition`

**Purpose:** Set target position

**Handler:** `SetTargetPositionHandler`

**Data:**
```
[PositionX: float]
[PositionY: float]
```

**Handler Logic:**
- Validates position
- Sets target position
- Updates target state
- Used for ground-targeted actions

**Example:**
```csharp
float x = buffer.GetFloat();
float y = buffer.GetFloat();

if (TargetRequestCompletionSource != null) {
    TargetRequestCompletionSource.SetResult(new Vector2(x, y));
}
```

### CancelTargetEntity

**Type:** `ClientPacketType.CancelTargetEntity`

**Purpose:** Cancel target selection

**Handler:** `CancelTargetEntityHandler`

**Data:**
```
(no data)
```

**Handler Logic:**
- Cancels target request
- Clears target state
- Updates target UI

**Example:**
```csharp
if (TargetRequestCompletionSource != null) {
    TargetRequestCompletionSource.TrySetCanceled();
}
```

## Item Packets

### EquipItem

**Type:** `ClientPacketType.EquipItem`

**Purpose:** Equip item from inventory

**Handler:** `EquipItemPacketHandler`

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Checks equipment requirements
- Validates equipment slot
- Equips item
- Updates stats
- Sends equip packet

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (player != null) {
    player.EquipItem(itemId);
}
```

### UnequipItem

**Type:** `ClientPacketType.UnequipItem`

**Purpose:** Unequip item

**Handler:** `UnequipItemPacketHandler`

**Data:**
```
[EquipmentSlot: varuint]
```

**Handler Logic:**
- Validates slot
- Unequips item
- Returns to inventory
- Updates stats
- Sends unequip packet

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (player != null) {
    player.UnequipItem((EquipmentItemSlot)itemId);
}
```

### UseItem

**Type:** `ClientPacketType.UseItem`

**Purpose:** Use consumable item

**Handler:** `UseItemPacketHandler`

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Checks if item is consumable
- Validates use conditions
- Applies item effects
- Removes item if consumed

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

player.Inventory.UseItem(player, itemId);
```

### DestroyItem

**Type:** `ClientPacketType.DestroyItem`

**Purpose:** Destroy item from inventory

**Handler:** `DestroyItemPacketHandler`

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates item exists
- Removes item from inventory
- Item permanently destroyed
- No recovery possible

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

player.Inventory.TryRemove(itemId);
```

## Chat Packets

### ChatMessage

**Type:** `ClientPacketType.ChatMessage`

**Purpose:** Send chat message

**Handler:** `ChatMessagePacketHandler`

**Data:**
```
[Channel: string]
[Content: string (max 200 chars)]
```

**Handler Logic:**
- Validates message length
- Checks if command (starts with '/')
- Executes command if applicable
- Broadcasts message to channel
- Handles chat filtering

**Example:**
```csharp
string channel = buffer.GetString();
string content = buffer.GetString(200);

if (content.StartsWith('/')) {
    // Execute command
    string[] split = content.Split(' ');
    Command.ExecuteCommand(split[0].ToLower(), ctrl, split.Skip(1).ToArray());
} else {
    // Disconnect (only commands allowed in this implementation)
    ctrl.Disconnect();
}
```

## Trade Packets

### RequestTrade

**Type:** `ClientPacketType.RequestTrade`

**Purpose:** Request trade with player

**Handler:** `TradeRequestPacketHandler`

**Data:**
```
[EntityId: int]
```

**Handler Logic:**
- Validates target entity
- Checks if target is player
- Creates trade request
- Sends trade request to target
- Handles trade request response

**Example:**
```csharp
int id = buffer.GetInt();

if (TryGetEntityById(id, out Entity entity) && entity is CharacterEntity otherPlayer) {
    otherPlayer.Controller?.CreateTradeRequest(ctrl.Player);
}
```

### AcceptTrade

**Type:** `ClientPacketType.AcceptTrade`

**Purpose:** Accept trade request

**Handler:** `TradeAcceptPacketHandler`

**Data:**
```
[Accept: bool]
```

**Handler Logic:**
- Validates trade session exists
- Checks accept cooldown
- Updates trade acceptance
- Sends trade status update
- Completes trade if both accept

**Example:**
```csharp
bool value = buffer.GetBool();

if (OnTrade && CanAcceptTradeAt <= CurrentTime) {
    AcceptTrade(value);
}
```

### NotAcceptTrade

**Type:** `ClientPacketType.NotAcceptTrade`

**Purpose:** Cancel trade acceptance

**Data:**
```
(no data)
```

**Handler Logic:**
- Resets trade acceptance
- Updates trade status
- Sends trade status update

### CancelTrade

**Type:** `ClientPacketType.CancelTrade`

**Purpose:** Cancel trade session

**Handler:** `TradeClosePacketHandler`

**Data:**
```
(no data)
```

**Handler Logic:**
- Closes trade session
- Returns items to inventories
- Clears trade state
- Sends trade close packet

**Example:**
```csharp
CloseTrade();
```

### SetTradeGold

**Type:** `ClientPacketType.SetTradeGold`

**Purpose:** Set gold amount in trade

**Handler:** `SetTradeGoldHandler`

**Data:**
```
[GoldAmount: varint (positive)]
```

**Handler Logic:**
- Validates gold amount
- Checks player has gold
- Updates trade gold
- Resets trade acceptance
- Sends trade status update

**Example:**
```csharp
int value = buffer.GetVarIntPositive();

SetTradeGold(value);
```

### ToggleItemFromTrade

**Type:** `ClientPacketType.ToggleItemFromTrade`

**Purpose:** Add/remove item from trade

**Handler:** `ToggleItemFromTradePacketHandler`

**Data:**
```
[ItemId: varuint]
[Quantity: varuint]
```

**Handler Logic:**
- Validates item exists
- Checks if item in trade
- Toggles item in trade container
- Resets trade acceptance
- Updates trade container

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();
uint quantity = buffer.GetVarUInt();

if (SelfTradeContainer != null) {
    if (SelfTradeContainer.ContainsKey(itemId)) {
        SelfTradeContainer.TryRemove(itemId);
    } else if (player.Inventory.TryGetValue(itemId, out Item item)) {
        SelfTradeContainer.AddInventoryItem(itemId, item);
    }
}
```

## Vendor Packets

### BuyItem

**Type:** `ClientPacketType.BuyItem`

**Purpose:** Buy item from vendor

**Handler:** `BuyItemPacketHandler`

**Data:**
```
[ItemId: varuint]
[Quantity: varuint]
```

**Handler Logic:**
- Validates vendor interaction
- Validates item exists in vendor
- Checks player has gold
- Adds item to inventory
- Removes gold from player

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();
uint quantity = buffer.GetVarUInt();

if (DialogCreator is NonPlayerCharacter npc && npc.Definition is VendorCharacterDefinition vcd) {
    vcd.BuyItem(player, itemId, quantity);
}
```

### SellItem

**Type:** `ClientPacketType.SellItem`

**Purpose:** Sell item to vendor

**Handler:** `SellItemPacketHandler`

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates vendor interaction
- Validates item exists
- Removes item from inventory
- Adds gold to player
- Adds to buyback list

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (DialogCreator is NonPlayerCharacter npc && npc.Definition is VendorCharacterDefinition vcd) {
    Item item = player.Inventory.TryRemove(itemId);
    if (item != null) {
        vcd.SellItem(player, item);
    }
}
```

### BuybackItem

**Type:** `ClientPacketType.BuybackItem`

**Purpose:** Buy back item from vendor

**Handler:** `BuybackItemPacketHandler`

**Data:**
```
[ItemId: varuint]
```

**Handler Logic:**
- Validates vendor interaction
- Validates item in buyback
- Checks player has gold
- Adds item to inventory
- Removes gold from player
- Removes from buyback

**Example:**
```csharp
uint itemId = buffer.GetVarUInt();

if (DialogCreator is NonPlayerCharacter npc && npc.Definition is VendorCharacterDefinition vcd) {
    vcd.Buyback(player, itemId);
}
```

## Party Packets

### RequestParty

**Type:** `ClientPacketType.RequestParty`

**Purpose:** Request party with player

**Handler:** `PartyRequestPacketHandler`

**Data:**
```
[EntityId: int]
```

**Handler Logic:**
- Validates target entity
- Checks if target is player
- Creates party request
- Sends party request to target

**Example:**
```csharp
int id = buffer.GetInt();

if (TryGetEntityById(id, out Entity entity) && entity is CharacterEntity otherPlayer) {
    otherPlayer.Controller?.CreatePartyRequest(player);
}
```

### ConfirmParty

**Type:** `ClientPacketType.ConfirmParty`

**Purpose:** Accept/deny party request

**Handler:** `PartyRequestResponseHandler`

**Data:**
```
[Accept: bool]
```

**Handler Logic:**
- Validates party request exists
- Updates party request response
- Creates party if accepted
- Sends party data to members

**Example:**
```csharp
bool response = buffer.GetBool();

if (!player.IsDead && PartyCompletionSource != null) {
    PartyCompletionSource.SetResult(response);
}
```

### LeavePartyRequest

**Type:** `ClientPacketType.LeavePartyRequest`

**Purpose:** Leave current party

**Handler:** `LeavePartyRequestPacketHandler`

**Data:**
```
(no data)
```

**Handler Logic:**
- Validates player in party
- Removes player from party
- Updates party members
- Sends party updates

**Example:**
```csharp
if (player != null && !player.IsDead && !player.IsDestroyed) {
    player.LeaveParty();
}
```

## Guild Packets

### CreateGuild

**Type:** `ClientPacketType.CreateGuild`

**Purpose:** Create new guild

**Data:**
```
[GuildName: string]
[GuildFlag: string] (optional)
```

**Handler Logic:**
- Validates guild name
- Checks name availability
- Validates creation cost
- Creates guild
- Sets player as owner

### RequestGuildJoin

**Type:** `ClientPacketType.RequestGuildJoin`

**Purpose:** Request to join guild

**Data:**
```
[GuildId: string]
```

**Handler Logic:**
- Validates guild exists
- Checks guild requirements
- Creates join request
- Sends request to guild

### AcceptGuildRequest

**Type:** `ClientPacketType.AcceptGuildRequest`

**Purpose:** Accept guild join request

**Data:**
```
[PlayerId: string]
```

**Handler Logic:**
- Validates request exists
- Checks permissions
- Adds player to guild
- Updates guild members

### DenyGuildRequest

**Type:** `ClientPacketType.DenyGuildRequest`

**Purpose:** Deny guild join request

**Data:**
```
[PlayerId: string]
```

**Handler Logic:**
- Validates request exists
- Checks permissions
- Removes request
- Notifies player

## Related Documentation

- [OVERVIEW.md](./OVERVIEW.md) - Protocol overview
- [PLAYER_PACKETS.md](./PLAYER_PACKETS.md) - Player-specific packets
- [UI_PACKETS.md](./UI_PACKETS.md) - UI packets
- [CONTAINER_PACKETS.md](./CONTAINER_PACKETS.md) - Container packets
- [TRADE.md](../player/TRADE.md) - Trade system
- [PARTY.md](../player/PARTY.md) - Party system
- [GUILD.md](../player/GUILD.md) - Guild system

