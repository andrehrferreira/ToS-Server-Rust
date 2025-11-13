# Administrator Commands Documentation

## Overview

The administrator command system provides Game Masters (GMs) and Administrators with powerful tools to manage the game world, players, and server operations. Commands are organized by permission level, ensuring that only authorized staff can use administrative functions.

## Permission Levels

### Permission Level Hierarchy

1. **Player** (`Plevel.Player`)
   - Regular players
   - Basic commands only

2. **Game Master** (`Plevel.GameMaster`)
   - GMs have access to player management and world manipulation
   - Can manage players, items, creatures, and teleportation

3. **Administrator** (`Plevel.Administrator`)
   - Highest level of access
   - Can manage server operations and world structure
   - Can create/remove respawns and shutdown server

## Game Master Commands

### Player Management

#### `/ban <playerTag> <reason>`

**Permission:** Game Master  
**Description:** Bans a player's account from the game.

**Parameters:**
- `playerTag`: Player's tag name or character name
- `reason`: Reason for the ban (required)

**Usage:**
```
/ban PlayerName Cheating detected
/ban TestUser Exploiting game mechanics
```

**Functionality:**
- Bans the player's account
- Disconnects the player immediately
- Records ban reason and admin who issued it
- Player cannot log in until ban is lifted

**Implementation Notes:**
- Requires valid player tag name
- Reason is mandatory
- Ban is permanent until manually removed
- Logs ban action with admin information

---

#### `/kick <playerTag>`

**Permission:** Game Master  
**Description:** Disconnects a player from the server.

**Parameters:**
- `playerTag`: Player's tag name or character name

**Usage:**
```
/kick PlayerName
/kick TestUser
```

**Functionality:**
- Immediately disconnects the player
- Player can reconnect immediately
- No ban is applied
- Useful for removing problematic players temporarily

**Implementation Notes:**
- Requires valid player tag name
- Player must be online
- Action is logged

---

#### `/goto <playerTag>` (Alias: `/gt`)

**Permission:** Game Master  
**Description:** Teleports the GM to a player's current location.

**Parameters:**
- `playerTag`: Player's tag name or character name

**Usage:**
```
/goto PlayerName
/gt TestUser
```

**Functionality:**
- Instantly teleports GM to player's position
- Works across maps
- Useful for investigating player issues

**Implementation Notes:**
- Requires valid player tag name
- Player must be online
- Works even if player is in different map

---

#### `/pull <playerTag>`

**Permission:** Game Master  
**Description:** Teleports a player to the GM's current location.

**Parameters:**
- `playerTag`: Player's tag name or character name

**Usage:**
```
/pull PlayerName
/pull TestUser
```

**Functionality:**
- Teleports player to GM's current position
- Works across maps
- Saves player position to database
- Useful for moving stuck players

**Implementation Notes:**
- Requires valid player tag name
- Player must be online
- Player position is saved after teleportation
- Works even if player is in different map

---

### Item Management

#### `/add <itemName> [amount]`

**Permission:** Game Master  
**Description:** Adds an item to the GM's inventory.

**Parameters:**
- `itemName`: Name of the item to add
- `amount`: Optional quantity (default: 1)

**Usage:**
```
/add GoldCoin
/add HealthPotion 10
/add Sword_Steel
```

**Functionality:**
- Creates item and adds to GM's inventory
- Supports equipment, consumables, and special items
- Items are properly serialized with properties
- Tooltips are sent for equipment and special items

**Implementation Notes:**
- Item name must exist in item database
- Amount must be positive integer
- Equipment items include full properties
- GM's character is saved after adding item

---

#### `/give <itemName> [amount]`

**Permission:** Game Master  
**Description:** Gives an item to the currently targeted player.

**Parameters:**
- `itemName`: Name of the item to give
- `amount`: Optional quantity (default: 1)

**Usage:**
```
/give GoldCoin
/give HealthPotion 5
/give Sword_Steel
```

**Functionality:**
- Creates item and gives to targeted player
- Requires a valid target (player)
- Both GM and target receive notification
- Supports all item types

**Implementation Notes:**
- Requires valid target actor (player)
- Item name must exist in item database
- Target player receives notification
- Target player's character is saved

---

### World Management

#### `/go <mapName> [waypoint]`

**Permission:** Game Master  
**Description:** Teleports the GM to a specific map and waypoint.

**Parameters:**
- `mapName`: Name of the destination map
- `waypoint`: Optional waypoint name (default: "Start")

**Usage:**
```
/go Tutorial
/go Dungeon_Level1 Entrance
/go City_Main Square
```

**Functionality:**
- Teleports GM to specified map
- Can specify waypoint for precise location
- Useful for map testing and navigation
- Works with any valid map name

**Implementation Notes:**
- Map name must exist
- Waypoint defaults to "Start" if not specified
- Invalid map names return error message

---

#### `/summon <creatureName> [amount]`

**Permission:** Game Master  
**Description:** Summons creatures at the GM's location.

**Parameters:**
- `creatureName`: Name of the creature to summon
- `amount`: Optional number of creatures (default: 1)

**Usage:**
```
/summon Goblin
/summon Dragon 3
/summon Orc_Warrior 5
```

**Functionality:**
- Creates creatures near GM's position
- Creatures spawn in random positions around GM (radius up to 2000 units)
- Multiple creatures can be summoned at once
- Creatures join the current map

**Implementation Notes:**
- Creature name must exist in entity database
- Creatures spawn in random positions around GM
- Creatures are added to GM's current map
- Invalid creature names return error

---

#### `/kill [target]`

**Permission:** Game Master  
**Description:** Kills the currently targeted entity.

**Parameters:**
- `target`: Optional target (uses current target if not specified)

**Usage:**
```
/kill
```

**Functionality:**
- Kills the targeted entity
- Requires a valid target
- Useful for testing death mechanics
- Can kill players or creatures

**Implementation Notes:**
- Requires valid target actor
- Target must be selected before using command
- Triggers death mechanics normally

---

### Character Management

#### `/setallstats <value>`

**Permission:** Game Master  
**Description:** Sets all character attributes to the specified value.

**Parameters:**
- `value`: Value to set all stats to (must be > 0)

**Usage:**
```
/setallstats 100
/setallstats 200
```

**Functionality:**
- Sets STR, DEX, INT, VIG, AGI, LUC to specified value
- Recalculates stats and resources
- Restores HP, Mana, Stamina
- Saves character to database

**Implementation Notes:**
- Value must be positive integer
- All stats are set to same value
- Character stats are recalculated
- Character is saved after modification

---

#### `/setallskills <level>`

**Permission:** Game Master  
**Description:** Sets all skills to the specified level.

**Parameters:**
- `level`: Skill level to set all skills to (must be > 0)

**Usage:**
```
/setallskills 50
/setallskills 100
```

**Functionality:**
- Sets all skills to specified level
- Calculates total experience required for level
- Updates skill information
- Saves character to database

**Implementation Notes:**
- Level must be positive integer
- All skills are set to same level
- Skill information is updated and sent to client
- Character is saved after modification

---

### Server Management

#### `/save`

**Permission:** Game Master  
**Description:** Saves all online players to the database.

**Parameters:** None

**Usage:**
```
/save
```

**Functionality:**
- Saves all online players
- Saves both in-memory and database
- Notifies all players that they were saved
- Useful before server maintenance

**Implementation Notes:**
- Affects all online players
- Players receive notification message
- Useful for ensuring data persistence

---

## Administrator Commands

### World Structure Management

#### `/respawn <creatureName>`

**Permission:** Administrator  
**Description:** Creates a respawn point for a creature at the GM's location.

**Parameters:**
- `creatureName`: Name of the creature to create respawn for

**Usage:**
```
/respawn Goblin
/respawn Dragon
/respawn Orc_Warrior
```

**Functionality:**
- Creates respawn point at GM's current position
- Uses creature's custom respawn list if available
- Respawn point persists until removed
- Useful for populating areas with creatures

**Implementation Notes:**
- Creature name must exist in entity database
- Respawn is created at GM's current position
- Uses creature's respawn list if it has one
- Respawn persists until manually removed

---

#### `/remove [target]`

**Permission:** Administrator  
**Description:** Removes a respawn point for the targeted creature.

**Parameters:**
- `target`: Optional target (uses current target if not specified)

**Usage:**
```
/remove
```

**Functionality:**
- Removes respawn point for targeted creature
- Requires valid target with respawn component
- Useful for cleaning up unwanted respawns

**Implementation Notes:**
- Requires valid target actor
- Target must have respawn component
- Respawn is permanently removed
- Useful for map editing

---

### Server Operations

#### `/shutdown`

**Permission:** Administrator  
**Description:** Shuts down the server with a countdown timer.

**Parameters:** None

**Usage:**
```
/shutdown
```

**Functionality:**
- Starts 60-second countdown
- Notifies all players of shutdown
- Updates countdown every 10 seconds (or every second in last 10 seconds)
- Disconnects all players when timer reaches 0
- Server exits after 10 seconds

**Implementation Notes:**
- 60-second countdown by default
- All players receive notifications
- Players are disconnected when timer reaches 0
- Server process exits after cleanup period
- Critical command - use with caution

---

## Player Commands

### Communication

#### `/global <message>` (Alias: `/g`)

**Permission:** Player  
**Description:** Sends a message to all players on the server.

**Parameters:**
- `message`: Message to broadcast

**Usage:**
```
/global Hello everyone!
/g Server maintenance in 10 minutes
```

**Functionality:**
- Broadcasts message to all online players
- Message appears in system message and special message channels
- Useful for announcements

---

#### `/whisper <playerTag> <message>` (Alias: `/w`)

**Permission:** Player  
**Description:** Sends a private message to another player.

**Parameters:**
- `playerTag`: Target player's tag name
- `message`: Message to send

**Usage:**
```
/whisper PlayerName Hello!
/w TestUser Can you help me?
```

**Functionality:**
- Sends private message to target player
- Both players see the message
- Message appears in whisper channel

---

### Party Management

#### `/party [target]` (Alias: `/p`)

**Permission:** Player  
**Description:** Requests to join a party with the targeted player.

**Parameters:**
- `target`: Optional target (uses current target if not specified)

**Usage:**
```
/party
/p
```

**Functionality:**
- Sends party join request to targeted player
- Requires valid player target
- Target must accept request

---

#### `/removeparty <playerTag>` (Alias: `/w`)

**Permission:** Player  
**Description:** Removes a player from your party (party leader only).

**Parameters:**
- `playerTag`: Player's tag name to remove

**Usage:**
```
/removeparty PlayerName
```

**Functionality:**
- Removes player from party
- Only party leader can use
- Both players receive notification

---

#### `/leave`

**Permission:** Player  
**Description:** Leaves the current party.

**Parameters:** None

**Usage:**
```
/leave
```

**Functionality:**
- Player leaves current party
- Only works if player is in a party

---

### Trading

#### `/trade [target]` (Alias: `/t`)

**Permission:** Player  
**Description:** Requests a trade with the targeted player.

**Parameters:**
- `target`: Optional target (uses current target if not specified)

**Usage:**
```
/trade
/t
```

**Functionality:**
- Sends trade request to targeted player
- Requires valid player target
- Target must accept request

---

### Utility

#### `/online`

**Permission:** Player  
**Description:** Shows the number of players currently online.

**Parameters:** None

**Usage:**
```
/online
```

**Functionality:**
- Displays count of online players
- Refreshes online player list
- Shows current server population

---

#### `/killme`

**Permission:** Player  
**Description:** Kills your own character.

**Parameters:** None

**Usage:**
```
/killme
```

**Functionality:**
- Sets player HP to 0
- Triggers death mechanics
- Useful for respawning or testing

---

#### `/unstuck`

**Permission:** Player  
**Description:** Teleports player to tutorial map if stuck or dead.

**Parameters:** None

**Usage:**
```
/unstuck
```

**Functionality:**
- If dead, revives player with 100 HP
- Teleports player to tutorial map ("1_Tutorial")
- Saves character to database
- Useful for getting unstuck from glitches

---

## Command System Architecture

### Command Registration

Commands are registered using the `Commands.add()` function:

```typescript
Commands.add(
    commandName: string,
    permissionLevel: Plevel,
    handler: (params: string[], socket: any, server: any, services: any) => void,
    alias?: string
)
```

### Permission Checking

Commands automatically check permission level before execution:
- Player commands: Available to all players
- Game Master commands: Require GM permissions
- Administrator commands: Require Admin permissions

### Error Handling

All commands include try-catch blocks:
- Errors are caught and logged
- User-friendly error messages are sent to command sender
- Prevents server crashes from command errors

### Command Parameters

- Parameters are passed as string array
- First parameter is `params[0]`
- Multiple parameters: `params.slice(1).join(" ")`
- Optional parameters checked with `params.length > 1`

## Best Practices

### For Game Masters

1. **Always provide reasons for bans**
   - Helps track player behavior
   - Useful for appeals and reviews

2. **Use `/save` before major operations**
   - Ensures data persistence
   - Prevents data loss

3. **Verify targets before using commands**
   - Check player names carefully
   - Confirm target selection

4. **Log important actions**
   - Helps track GM activity
   - Useful for accountability

### For Administrators

1. **Use `/shutdown` responsibly**
   - Warn players in advance
   - Use during maintenance windows
   - Ensure all players are notified

2. **Manage respawns carefully**
   - Balance creature populations
   - Test respawn rates
   - Remove unwanted respawns

3. **Monitor server health**
   - Check player counts regularly
   - Monitor command usage
   - Review logs periodically

## Security Considerations

### Permission Enforcement

- Commands check permission level before execution
- Invalid permissions result in command failure
- Permission levels are verified server-side

### Input Validation

- All parameters are validated
- Invalid inputs return error messages
- Prevents command injection attacks

### Logging

- Important actions are logged
- Ban/kick actions include admin information
- Helps track administrative actions

## Integration with Other Systems

### Player System

- Commands interact with Player entities
- Character data is saved after modifications
- Player state is properly managed

### Map System

- Teleportation commands use map system
- Respawn commands interact with map respawns
- World management commands affect map state

### Item System

- Item commands use ItemsService
- Proper item creation and serialization
- Equipment properties are maintained

### Network System

- Commands send packets to clients
- System messages notify players
- Special messages for important events

## Testing Considerations

### Unit Tests

- Test each command with valid inputs
- Test error cases (invalid players, items, etc.)
- Test permission level enforcement

### Integration Tests

- Test command interactions with game systems
- Test command effects on player state
- Test command logging and error handling

### Security Tests

- Test permission enforcement
- Test input validation
- Test command injection prevention

## Summary

The administrator command system provides:

- **Player Management**: Ban, kick, teleport, pull players
- **Item Management**: Add items, give items to players
- **World Management**: Teleport, summon creatures, manage respawns
- **Character Management**: Modify stats and skills
- **Server Management**: Save players, shutdown server
- **Player Commands**: Communication, party, trade, utility commands

**Key Features:**
- Permission-based access control
- Comprehensive error handling
- Player-friendly error messages
- Integration with all game systems
- Logging and accountability

This system ensures that Game Masters and Administrators have the tools they need to manage the game world effectively while maintaining security and accountability.

