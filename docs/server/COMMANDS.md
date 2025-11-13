# Commands

## Overview

The Commands system provides a flexible command parsing and execution framework with permission levels. Commands are organized by access level (Player, Gamemaster, Admin) and support aliases for convenience.

## Commands Class

### Static Properties

```typescript
class Commands {
    public static commands: Map<string, CommandContainer>
    public static access: Map<string, Plevel>
}
```

### Command Container

```typescript
class CommandContainer {
    public plevel: Plevel
    public cb: Function
}
```

## Permission Levels

### Plevel Enum

```typescript
enum Plevel {
    Player = 0,
    Gamemaster = 1,
    Admin = 2
}
```

**Levels:**
- **Player**: Basic player commands
- **Gamemaster**: Game master commands
- **Admin**: Administrator commands

## Command Parsing

### Parse Command

```typescript
static parseCommand(command: string, socket: any, server: any, services: any): void
```

**Process:**
1. Remove leading "/"
2. Convert to lowercase
3. Split by spaces
4. Extract command name (first word)
5. Extract parameters (remaining words)
6. Check if command exists
7. Check permission level
8. Execute command callback

**Example:**
```
"/give item IronSword 1"
→ command: "give"
→ params: ["item", "IronSword", "1"]
```

### Permission Check

**Check Process:**
1. Get command container
2. Compare socket.plevel with command.plevel
3. If insufficient: send error message
4. If sufficient: execute callback

**Error Message:**
```
"You do not have the permission to execute this command"
```

## Command Registration

### Add Command

```typescript
static add(
    command: string, 
    plevel: Plevel, 
    callback: Function, 
    alias: string = null
): void
```

**Process:**
1. Check if command already exists
2. Create CommandContainer
3. Store in commands map
4. If alias provided: register alias

**Alias Support:**
- Optional alias parameter
- Same permission level
- Same callback function
- Convenience for users

## Command Categories

### Admin Commands

**Commands:**
- **ban**: Ban player
- **global**: Global message
- **remove**: Remove entity/item
- **respawn**: Respawn entity
- **shutdown**: Shutdown server

**Access:** Admin only

### Gamemaster Commands

**Commands:**
- **add**: Add item/entity
- **give**: Give item to player
- **go**: Teleport to location
- **goto**: Teleport to waypoint
- **kick**: Kick player
- **kill**: Kill entity
- **pull**: Pull entity to location
- **save**: Save player data
- **setallskills**: Set all skills
- **setallstats**: Set all stats
- **summon**: Summon entity

**Access:** Gamemaster and above

### Player Commands

**Commands:**
- **killme**: Kill self
- **leave**: Leave party/guild
- **online**: List online players
- **party**: Party commands
- **removeparty**: Leave party
- **trade**: Request trade
- **unstuck**: Teleport to safe location
- **whisper**: Private message

**Access:** All players

## Command Implementation

### Command Structure

**Command Function:**
```typescript
function commandName(params: string[], socket: any, server: any, services: any): void {
    // Command logic
}
```

**Parameters:**
- **params**: Array of command arguments
- **socket**: Player socket connection
- **server**: Server instance
- **services**: Service providers

### Example Commands

**Give Command:**
```typescript
Commands.add("give", Plevel.Gamemaster, (params, socket, server, services) => {
    const [type, name, amount] = params;
    // Give item logic
});
```

**Go Command:**
```typescript
Commands.add("go", Plevel.Gamemaster, (params, socket, server, services) => {
    const [mapName, waypoint] = params;
    // Teleport logic
});
```

**Killme Command:**
```typescript
Commands.add("killme", Plevel.Player, (params, socket, server, services) => {
    // Kill self logic
});
```

## Error Handling

### Command Not Found

**Error:**
```
"Command not exits"
```

**Trigger:**
- Command not in commands map
- Invalid command name
- Typo in command

### Permission Denied

**Error:**
```
"You do not have the permission to execute this command"
```

**Trigger:**
- socket.plevel < command.plevel
- Insufficient permissions
- Access level mismatch

## Command Aliases

### Alias Registration

**Process:**
1. Register main command
2. Register alias with same container
3. Both point to same callback
4. Same permission level

**Example:**
```typescript
Commands.add("teleport", Plevel.Gamemaster, callback, "tp");
// Both "teleport" and "tp" work
```

## Command Execution

### Execution Flow

1. **Parse**: Extract command and params
2. **Lookup**: Find command in map
3. **Validate**: Check permissions
4. **Execute**: Call callback function
5. **Handle**: Process result/errors

### Callback Invocation

**Invocation:**
```typescript
if (typeof cc.cb === "function") {
    cc.cb(params, socket, server, services);
}
```

**Type Check:**
- Ensures callback is function
- Prevents errors
- Safe execution

## Socket Integration

### Socket Properties

**Required Properties:**
- **plevel**: Permission level
- **character**: Player character
- **send**: Send message function

**Usage:**
- Check permission level
- Access player data
- Send responses
- Execute commands

## Server Integration

### Server Instance

**Usage:**
- Access server state
- Broadcast messages
- Manage connections
- Server operations

## Services Integration

### Service Providers

**Usage:**
- Database operations
- Entity management
- Item creation
- Map operations

## Implementation Notes

### Performance

**Optimizations:**
- Map-based lookup (O(1))
- Lowercase conversion
- Efficient parsing
- Minimal overhead

### Security

**Security Measures:**
- Permission checks
- Input validation
- Parameter sanitization
- Error handling

### Extensibility

**Adding Commands:**
1. Create command function
2. Register with Commands.add()
3. Set permission level
4. Optional alias

## Related Documentation

- [ADMIN_COMMANDS.md](./ADMIN_COMMANDS.md) - Admin command details
- [PLAYER.md](../player/PLAYER.md) - Player system
- [SECURITY.md](./SECURITY.md) - Security system

