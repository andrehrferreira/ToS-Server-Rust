# Battle Castle Event Documentation

## Overview

Battle Castle is a large-scale PvP event system where two teams compete in an instanced castle map. Players are randomly assigned to either Team Red (Team1) or Team Blue (Team2) upon entering through the event portal. The event features a point-based victory system where teams earn points by eliminating enemy players, with the first team to reach 150 points or the team with the most points when the timer expires winning the event. The event includes a comprehensive ranking system tracking kills and deaths, respawn mechanics, and reward distribution based on team victory and individual performance.

## Key Features

- **Two-Team System**: Red Team (Team1) vs Blue Team (Team2)
- **Random Team Assignment**: Players are randomly assigned to teams upon portal entry
- **40 Player Maximum**: Event supports up to 40 concurrent participants
- **Point-Based Victory**: Teams earn points per kill, victory at 150 points or highest score at timer end
- **15-Minute Timer**: Battle lasts 15 minutes (900 seconds) unless victory condition is met
- **Kill/Death Ranking**: Individual performance tracking for all participants
- **Respawn System**: 30-second respawn timer with team-specific spawn points
- **Reward System**: Rewards based on team victory and individual kill performance
- **Instanced Map**: Each event creates an isolated instance of the BattleCastle map
- **Real-Time Updates**: Live scoreboard updates every 30 seconds
- **Special Rewards**: Optional special reward items can be configured per event

## Event Flow

### Phase 1: Event Initialization

**Admin Trigger:**
- Admin calls `StartBattleCastle(PlayerController ctrl, string specialReward = "")`
- Event status set to `InProgress = true`
- Special reward item configured (optional)

**Portal Creation:**
- After 30 seconds, portal spawns at position (159, 252) in the admin's current scene
- Portal uses blueprint "Portal Evento"
- Portal is configured as:
  - Event portal (`IsEventPortal = true`)
  - Instanced (`IsInstanced = true`)
  - Blocks creatures (`BlocksCreature = true`)
  - Random team assignment (`WaypointRandom = ["Team1", "Team2"]`)
  - Destination: "BattleCastle" map, "Start" waypoint

**Portal Entry:**
- Players interact with portal to enter
- Random team assignment (Team1 or Team2)
- Portal closes automatically when 40 players enter
- Portal also closes after 5 minutes regardless of player count
- Each entry increments `PortalController` counter

**Map Initialization:**
- First player entry triggers map initialization
- Scene "BattleCastle" is created/opened
- `StartMap = true` flag set
- All data structures cleared and reset

### Phase 2: Pre-Battle Countdown

**5-Minute Countdown:**
- Broadcast: "The event will start in 5 min, get ready for battle!"
- Countdown messages every minute:
  - "The battle starts in 4 minute(s)"
  - "The battle starts in 3 minute(s)"
  - "The battle starts in 2 minute(s)"
  - "The battle starts in 1 minute(s)"
  - "The battle starts in 0 minute(s)"

**Player Registration:**
- All players entering are added to `BroadcastList` (for messaging)
- All players added to `Players` dictionary (team tracking)
- Initial rankings set to 0 kills, 0 deaths
- Player join notifications sent to all participants
- Each player receives list of all current participants

### Phase 3: Battle Phase

**Battle Start:**
- Timer set to 900 seconds (15 minutes)
- `BattleCastlePackets.Start()` sent to all participants
- Broadcast: "It's duel time!"
- Stats update system begins (every 30 seconds)
- Event counter begins countdown

**Combat Mechanics:**
- Players can attack and kill enemy team members
- Each kill awards 1 point to the killer's team
- Kills tracked in `RankingKills` dictionary
- Deaths tracked in `RankingDeaths` dictionary
- Real-time ranking updates broadcast to all players

**Death and Respawn:**
- When player dies:
  1. Player marked as dead (`IsDead = true`)
  2. Status effects removed (Poisoned, Frozen)
  3. Invulnerability flag added
  4. Death count incremented
  5. Killer's kill count incremented
  6. Killer's team points incremented
  7. Rankings updated and broadcast
  8. Victory condition checked (150 points)
  9. After 1 second delay:
     - Knockdown animation executed
     - Player interrupted (actions cancelled)
     - Full resources restored (Health, Mana, Stamina)
     - Teleport to team spawn point:
       - Team1 (Red): Position (411, 206)
       - Team2 (Blue): Position (82, 355)
  10. After 30 seconds:
     - Player revived (`IsDead = false`)
     - Invulnerability removed
     - Full resources restored again
     - Player can resume combat

**Victory Conditions:**
- **Immediate Victory**: Team reaches 150 points
- **Timer Victory**: Team with most points when timer reaches 0
- **Force End**: Admin can force end event (`ForceEndEvent = true`)

### Phase 4: Event Conclusion

**Timer Expiration:**
- When timer reaches 0, 1-minute grace period begins
- Team with highest points declared winner
- Winner team name broadcast ("Red" or "Blue")
- Rewards distributed

**Reward Distribution:**
- **Winning Team**: 100 Battle Coins per player
- **Losing Team**: 10 Battle Coins per player
- **Serial Killer**: Player with most kills receives additional 100 Battle Coins
- **Special Reward**: If configured, all players receive 1 special reward item
- Rewards added to `EventInventory` (separate from regular inventory)

**Event Cleanup:**
- Broadcast: "Congrats to team [TeamName] on the win"
- `InProgress = false`
- Broadcast: "Bye bye"
- All players removed from invulnerability
- All players killed (to force exit from event map)
- 5-minute delay before final cleanup
- All data structures cleared
- Scene closed
- Event fully reset

## Data Structures

### Static State

```csharp
public static bool InProgress = false;              // Event active status
public static Entity Portal;                        // Portal entity reference
public static Dictionary<Guid, PlayerController> BroadcastList;  // All participants
public static Dictionary<Guid, string> Players;    // Player ID -> Team mapping
public static Dictionary<Guid, Guid> Kills;         // Kill tracking (unused in current implementation)
public static Dictionary<Guid, int> RankingKills;   // Player ID -> Kill count
public static Dictionary<Guid, int> RankingDeaths;  // Player ID -> Death count
public static int MaxPlayers = 40;                  // Maximum participants
public static int BattleTimer = 900;                // Timer in seconds (15 minutes)
public static int PortalController = 0;             // Players entered counter
public static bool StartMap = false;                // Map initialized flag
public static bool ForceEndEvent = false;           // Admin force end flag
public static string SpecialReward = "";            // Optional special reward item name
public static int Team1Points = 0;                  // Red team score
public static int Team2Points = 0;                  // Blue team score
private static Scene Scene;                         // Event scene reference
```

### Player Entity Flags

**Event-Specific Properties:**
- `ch.InEvent = true`: Player is participating in event
- `ch.EventTeam`: "Team1" or "Team2" (Red or Blue)
- `ch.EventInventory`: Separate inventory for event rewards

**Status Flags:**
- `CreatureFlags.Invulnerable`: Player cannot take damage (during respawn)
- `CreatureFlags.Poisoned`: Removed on death
- `CreatureFlags.Frozen`: Removed on death

## Team System

### Team Assignment

**Current Implementation (Random Distribution):**
- Upon portal entry, player randomly assigned to Team1 or Team2
- Assignment happens via portal's `WaypointRandom` property
- Teams displayed as "Red" (Team1) or "Blue" (Team2) to players
- No team balancing - purely random assignment
- **Note**: Party members are not guaranteed to be on the same team (see Required Improvements section)

**Future Implementation (Party-Aware Assignment):**
- Party members will be assigned to the same team
- Assignment logic will check party membership before random assignment
- Party-based assignment takes priority over random distribution

**Team Identification:**
- Team stored in `ch.EventTeam` property
- Team name used for point tracking
- Team name used for spawn point selection
- Team name used for victory determination

### Team Spawn Points

**Team1 (Red Team):**
- Spawn Position: (411, 206) in BattleCastle map
- Used for respawn after death

**Team2 (Blue Team):**
- Spawn Position: (82, 355) in BattleCastle map
- Used for respawn after death

## Scoring System

### Point Calculation

**Kill Points:**
- Each player kill awards 1 point to killer's team
- Points tracked in `Team1Points` or `Team2Points`
- Points incremented immediately upon kill
- Victory condition checked after each kill

**Victory Threshold:**
- First team to reach 150 points wins immediately
- Event ends regardless of remaining time
- `ForceEndEvent = true` triggered

### Ranking System

**Kill Ranking:**
- Each kill increments player's kill count in `RankingKills`
- Kill count used for "Serial Killer" reward determination
- Kill count displayed in real-time scoreboard

**Death Ranking:**
- Each death increments player's death count in `RankingDeaths`
- Death count displayed in real-time scoreboard
- Used for performance tracking

**Ranking Updates:**
- Rankings updated immediately on kill/death
- Rankings broadcast to all participants via `BattleCastlePackets.DeathInfo`
- Rankings included in 30-second stats updates

## Respawn System

### Respawn Process

**Death Sequence:**
1. Player dies (health reaches 0)
2. Status effects cleared
3. Invulnerability applied
4. Rankings updated
5. Team points updated
6. Victory condition checked

**Respawn Sequence:**
1. 1-second delay
2. Knockdown animation executed
3. All actions interrupted
4. Full resources restored (Health, Mana, Stamina)
5. Teleport to team spawn point
6. 30-second invulnerability period
7. Player revived and ready for combat

### Respawn Mechanics

**Invulnerability Period:**
- 30 seconds of invulnerability after respawn
- Player cannot take damage during this time
- Player can move and prepare for combat
- Invulnerability flag removed after 30 seconds

**Resource Restoration:**
- Health restored to maximum
- Mana restored to maximum
- Stamina restored to maximum
- Status effects cleared (Poison, Freeze, etc.)

**Spawn Point Selection:**
- Based on player's `EventTeam` property
- Team1 spawns at (411, 206)
- Team2 spawns at (82, 355)
- Spawn points are team-specific

## Reward System

### Reward Types

**Team Victory Rewards:**
- **Winning Team**: 100 Battle Coins
- **Losing Team**: 10 Battle Coins
- Rewards added to `EventInventory`

**Individual Performance Rewards:**
- **Serial Killer**: Player with most kills receives additional 100 Battle Coins
- If multiple players tie for most kills, first one found in iteration receives reward
- Reward added to `EventInventory`

**Special Rewards:**
- Optional special reward item configured at event start
- All participants receive 1 special reward item (if configured)
- Item name specified in `specialReward` parameter
- Item creation uses `Item.Create(SpecialReward, 1)`
- Errors in special reward creation are silently caught

### Reward Distribution

**Distribution Timing:**
- Rewards distributed immediately after winner determination
- Rewards added before event cleanup
- All participants receive rewards regardless of performance (except Serial Killer bonus)

**Reward Storage:**
- Rewards stored in `EventInventory` (separate from regular inventory)
- Prevents mixing event rewards with regular gameplay items
- Allows for event-specific item management

## Real-Time Updates

### Stats Updates

**Update Frequency:**
- Stats updated every 30 seconds during battle
- Updates continue until event ends or timer expires
- Updates stop if `InProgress = false` or `BattleTimer <= 0`

**Update Content:**
- Team1 player count
- Team2 player count
- Team1 points
- Team2 points
- Remaining battle timer
- Sent via `BattleCastlePackets.Summary()`

### Ranking Updates

**Update Triggers:**
- Immediate update on player kill
- Immediate update on player death
- Included in stats updates (every 30 seconds)

**Update Content:**
- Player name
- Player account ID
- Kill count
- Death count
- Sent via `BattleCastlePackets.DeathInfo()`

### Player Join Updates

**Update Triggers:**
- When new player enters event
- When player requests update (`UpdateMe()`)

**Update Content:**
- New player name
- New player account ID
- New player team
- Sent via `BattleCastlePackets.NewPlayer()`

## Network Packets

### BattleCastlePackets

**Start:**
- Sent to all participants when battle begins
- Triggers UI initialization on client
- Contains event start signal

**NewPlayer:**
- Sent when player joins event
- Contains: player name, account ID, team
- Broadcast to all existing participants
- New player receives list of all existing participants

**DeathInfo:**
- Sent on player kill/death
- Contains: player name, account ID, deaths, kills
- Updates ranking display on client
- Broadcast to all participants

**Summary:**
- Sent every 30 seconds during battle
- Contains: Team1 count, Team2 count, Team1 points, Team2 points, timer
- Updates scoreboard on client
- Broadcast to all participants

## Event Management

### Admin Commands

**Start Event:**
```csharp
BattleCastle.StartBattleCastle(PlayerController ctrl, string specialReward = "")
```
- `ctrl`: Admin player controller triggering event
- `specialReward`: Optional item name for special reward (empty string = no special reward)
- Creates portal in admin's current scene
- Initializes event system

**Force End Event:**
```csharp
BattleCastle.ForceEndEvent = true;
```
- Immediately ends event on next timer check
- Used for emergency event termination
- Event concludes normally (rewards distributed, cleanup performed)

### Event Lifecycle

**Total Duration:**
- Portal available: 5 minutes (or until 40 players enter)
- Pre-battle countdown: 5 minutes
- Battle phase: Up to 15 minutes (or until victory condition)
- Post-battle cleanup: 5 minutes
- **Total Maximum**: ~30 minutes

**Event States:**
1. **Not Started**: `InProgress = false`, no portal
2. **Portal Open**: Portal exists, players can enter
3. **Pre-Battle**: Countdown phase, players preparing
4. **In Progress**: Battle active, timer counting down
5. **Ending**: Rewards distributed, cleanup in progress
6. **Ended**: Event fully reset, ready for next event

## Integration Points

### Entity System

**CharacterEntity Integration:**
- `InEvent` flag: Marks player as event participant
- `EventTeam` property: Stores team assignment
- `EventInventory` property: Separate inventory for rewards
- `Died` event: Handles death and respawn logic

**CreatureFlags Integration:**
- `Invulnerable`: Used during respawn period
- `Poisoned`: Cleared on death
- `Frozen`: Cleared on death

### Scene System

**Scene Management:**
- Event uses instanced scene "BattleCastle"
- Scene created/opened on first player entry
- Scene closed on event end
- Scene supports multiple concurrent instances

**Teleportation:**
- Uses `TeleportToPosition()` for respawn
- Team-specific spawn coordinates
- Instant teleportation (no loading screen)

### Action System

**Knockdown Action:**
- Uses `Actions["Knockdown"]` for death animation
- Executed during respawn sequence
- Provides visual feedback for player death

### Portal System

**TeleportEntity Integration:**
- Portal uses `TeleportEntity` blueprint
- Configured as event portal
- Random team assignment via `WaypointRandom`
- Instanced portal (each event has own portal)
- **Note**: Portal team assignment logic needs to be modified to check party membership (see Required Improvements)

### Party System

**Current Integration:**
- No party-based team assignment (random assignment only)
- Party members may be split across teams

**Required Integration:**
- Check `ch.Party` or party membership on portal entry
- Assign party members to same team as existing party members in event
- Handle party entry synchronization
- Maintain team balance while keeping parties together
- Support both 5-member parties and 50-member raids

## Implementation Considerations

### TypeScript Implementation

**Event State Management:**
```typescript
class BattleCastle {
    static inProgress: boolean = false;
    static portal: Entity | null = null;
    static broadcastList: Map<Guid, PlayerController> = new Map();
    static players: Map<Guid, string> = new Map();
    static rankingKills: Map<Guid, number> = new Map();
    static rankingDeaths: Map<Guid, number> = new Map();
    static maxPlayers: number = 40;
    static battleTimer: number = 900;
    static portalController: number = 0;
    static startMap: boolean = false;
    static forceEndEvent: boolean = false;
    static specialReward: string = "";
    static team1Points: number = 0;
    static team2Points: number = 0;
    private static scene: Scene | null = null;
}
```

**Event Flow:**
- Use async/await for delays
- Use TimerQueue for scheduled tasks
- Use event emitters for player death handling
- Use Map for efficient lookups

**Respawn Logic:**
- Use setTimeout for delays
- Use teleport system for spawn point movement
- Use flag system for invulnerability
- Use action system for knockdown animation

### Performance Optimizations

**Broadcast Optimization:**
- Cache broadcast list to avoid repeated lookups
- Batch packet sends when possible
- Use efficient dictionary lookups

**Update Optimization:**
- Stats updates use recursive async function (consider using interval instead)
- Ranking updates sent immediately (no batching needed)
- Player join updates sent individually (necessary for synchronization)

**Memory Management:**
- Clear all dictionaries on event end
- Remove event handlers on player exit
- Dispose scene reference on cleanup

### Security Considerations

**Team Assignment:**
- Random assignment prevents team stacking
- No manual team selection (prevents exploitation)
- Team assignment happens server-side only

**Reward Validation:**
- Special reward item creation wrapped in try-catch
- Invalid items don't crash event
- Rewards only distributed to participants

**Event State Validation:**
- Check `InProgress` before processing actions
- Validate player is in event before applying logic
- Prevent duplicate entries

## Testing Scenarios

### Basic Event Flow

1. Admin starts event
2. Portal spawns after 30 seconds
3. Players enter portal (random team assignment)
4. First player triggers map initialization
5. 5-minute countdown begins
6. Battle starts after countdown
7. Players fight and earn points
8. Team reaches 150 points or timer expires
9. Rewards distributed
10. Event cleanup completes

### Edge Cases

**Portal Closes Early:**
- Portal closes when 40 players enter
- Portal closes after 5 minutes regardless
- Players already in event continue normally

**Player Disconnect:**
- Disconnected players remain in rankings
- Disconnected players don't receive rewards
- Event continues normally

**Tie in Serial Killer:**
- First player found with max kills receives reward
- No tie-breaking logic (first wins)

**Special Reward Error:**
- Invalid item name silently ignored
- Event continues normally
- Other rewards still distributed

**Force End:**
- Admin can force end at any time
- Event concludes normally
- Rewards based on current scores

## Required Improvements

### Party Team Assignment

**Current Behavior:**
- Players are randomly assigned to teams regardless of party membership
- Party members may end up on different teams, splitting groups

**Required Change:**
- **Party members must be assigned to the same team**
- When a party member enters the portal, check if other party members are already in the event
- If party members exist in event, assign new member to the same team as existing party members
- If no party members in event, assign randomly but ensure all party members entering together get same team
- Party-based team assignment takes priority over random assignment

**Implementation Considerations:**
- Check `ch.Party` or party membership before team assignment
- Group party members entering portal together
- Handle case where party members enter at different times
- Maintain team balance while respecting party groups
- Consider party size limits (5-member party vs 50-member raid)

**Edge Cases:**
- Party member enters alone (assign to team with other party members if any)
- Multiple party members enter simultaneously (assign all to same team)
- Party member enters after party split (assign to team with majority of party members)
- Party exceeds team capacity (split party across teams, prioritizing keeping party together)

## Future Enhancements

### Potential Improvements

**Team Balancing:**
- Balance teams based on player count
- Balance teams based on player level/gear
- Allow team switching before battle starts

**Additional Victory Conditions:**
- Capture point system
- Objective-based victory
- Multiple rounds

**Enhanced Rewards:**
- Performance-based tiered rewards
- Participation rewards
- Streak bonuses (kill streaks)

**UI Improvements:**
- Real-time kill feed
- Player status indicators
- Team composition display
- Map overview with player positions

**Event Variants:**
- Different map layouts
- Different team sizes
- Different victory conditions
- Different respawn mechanics

## Related Documentation

- [PLAYER.md](../player/PLAYER.md) - Player entity system
- [PARTY.md](../player/PARTY.md) - Party system (team assignment integration)
- [ACTIONS.md](../player/ACTIONS.md) - Action system (knockdown action)
- [TEAMS.md](../core/TEAMS.md) - Team system integration
- [TELEPORTATION.md](../server/TELEPORTATION.md) - Teleportation system
- [MAPS.md](../server/MAPS.md) - Map and scene system

