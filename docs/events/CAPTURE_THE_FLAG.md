# Capture the Flag Event Documentation

## Overview

Capture the Flag (CTF) is a team-based PvP event where two teams compete to capture and control flags on an instanced map. Each team defends their own flag while attempting to capture the neutral flag in the center. Teams earn points by controlling flags, with the team having the most points after 15 minutes winning the event.

## Key Features

- **Two-Team System**: Two teams compete for flag control
- **Three Flags**: Each team has a defensive flag, plus one neutral flag in the center
- **Capture Mechanics**: Players must stand near flag for 30 seconds without taking damage to capture
- **Point System**: Teams earn 10 points per minute per controlled flag
- **15-Minute Duration**: Event lasts 15 minutes (900 seconds)
- **Defense Mechanics**: Teams must defend captured flags from enemy capture
- **Instanced Map**: Each event creates an isolated instance of the CTF map
- **Real-Time Scoreboard**: Live score updates showing team points and flag status

## Event Flow

### Phase 1: Event Initialization

**Admin Trigger:**
- Admin starts the event via command or admin interface
- Event status set to `InProgress = true`
- Map instance created for the event

**Map Setup:**
- Instanced map created with CTF layout
- Three flag positions configured:
  - Team 1 flag (defensive position)
  - Team 2 flag (defensive position)
  - Neutral flag (center position)
- Team spawn points configured
- Map boundaries and safe zones set

**Team Assignment:**
- Players assigned to Team 1 or Team 2
- Teams balanced as much as possible
- Party members kept together when possible
- Players teleported to team spawn points

### Phase 2: Pre-Event Countdown

**5-Minute Countdown:**
- Broadcast: "Capture the Flag event starting in 5 minutes!"
- Countdown messages every minute:
  - "Event starts in 4 minute(s)"
  - "Event starts in 3 minute(s)"
  - "Event starts in 2 minute(s)"
  - "Event starts in 1 minute(s)"
  - "Event starts in 0 minute(s) - Get ready!"

**Flag Initialization:**
- All flags spawn at their positions
- Team flags start as controlled by their respective teams
- Neutral flag starts as neutral (no team control)
- Flags visible to all players
- Flag status indicators shown

### Phase 3: Event Phase

**Event Start:**
- Timer set to 900 seconds (15 minutes)
- Broadcast: "Capture the Flag event has begun!"
- Point tracking begins
- Scoreboard updates every 30 seconds

**Flag Control System:**
- **Team Flags**: Start controlled by their respective teams
- **Neutral Flag**: Starts neutral, can be captured by either team
- **Flag Status**: Visual indicator shows which team controls each flag
- **Flag Colors**: Team colors indicate control (Red/Blue)

**Capture Mechanics:**
- Player approaches flag (within capture radius)
- Player must remain within radius for 30 seconds
- Player cannot take damage during capture attempt
- If player takes damage, capture progress resets
- Capture progress shown to player (progress bar)
- When 30 seconds complete, flag is captured by player's team
- Broadcast: "[Team Name] captured [Flag Name]!"

**Defense Mechanics:**
- Teams must defend their captured flags
- Enemy players can capture flags from opposing teams
- Same 30-second capture process applies
- Defending players can interrupt enemy capture attempts
- Killing enemy players interrupts their capture progress

**Point System:**
- Every 60 seconds (1 minute), points are calculated
- Teams earn 10 points per flag they control
- Points calculated based on flag status at that moment
- Scoreboard updated with new totals
- Broadcast: "Score update: Team 1: [points], Team 2: [points]"

**Flag Status Tracking:**
- Each flag tracks controlling team
- Flag status updated in real-time
- Flag status broadcast to all players
- Visual indicators updated on map

### Phase 4: Event End

**Victory Conditions:**
- Timer reaches 0 (15 minutes elapsed)
- Team with most points wins
- In case of tie, team with most recent flag capture wins

**Event Completion:**
- Event status set to `Completed = true`
- Final score broadcast to all players
- Winner announcement: "[Team Name] wins Capture the Flag!"
- Final scoreboard displayed
- Rewards distributed based on team victory and individual performance

**Reward Distribution:**
- Winning team receives base rewards
- Losing team receives participation rewards
- Individual performance bonuses (captures, defenses, kills)
- Rewards sent to player inventories

**Cleanup:**
- Players teleported back to safe location
- Map instance destroyed
- Event data cleaned up
- Resources released

## Flag Mechanics

### Flag Types

**Team Flags:**
- One flag per team
- Located in team's defensive area
- Starts controlled by respective team
- Can be captured by enemy team
- Team loses points if flag is captured

**Neutral Flag:**
- One flag in center of map
- Starts neutral (no team control)
- Can be captured by either team
- Provides points when controlled
- Strategic objective for both teams

### Capture Process

**Capture Requirements:**
- Player must be within capture radius (e.g., 5 units)
- Player must remain in radius for 30 seconds
- Player cannot take damage during capture
- Player must be alive
- Player must be on a team

**Capture Progress:**
- Progress bar shown to capturing player
- Progress updates every second (0-100%)
- Progress visible to nearby players
- Progress resets if player leaves radius or takes damage
- Progress resets if player dies

**Capture Completion:**
- When 30 seconds complete, flag is captured
- Flag ownership changes to capturing player's team
- Flag visual updates (color change)
- Broadcast sent to all players
- Point calculation includes new flag status

### Defense Process

**Defense Strategies:**
- Players can guard flags to prevent capture
- Killing enemy players interrupts their capture
- Multiple defenders can protect a flag
- Defenders can attack capturing enemies
- Defenders can use abilities to interrupt capture

**Interrupt Mechanics:**
- Any damage to capturing player resets progress
- Killing capturing player resets progress
- Crowd control effects interrupt capture
- Defenders can push enemies out of capture radius

## Point System

### Point Calculation

**Point Frequency:**
- Points calculated every 60 seconds (1 minute)
- Calculation based on flag status at that moment
- Points awarded: 10 points per controlled flag

**Point Examples:**
- Team controls 1 flag = 10 points per minute
- Team controls 2 flags = 20 points per minute
- Team controls 3 flags = 30 points per minute (rare, requires capturing all flags)

**Point Tracking:**
- Total points tracked per team
- Points displayed on scoreboard
- Points updated every minute
- Final points determine winner

### Scoreboard

**Scoreboard Information:**
- Team 1 points
- Team 2 points
- Time remaining
- Flag status (which team controls each flag)
- Top performers (most captures, most defenses)

**Scoreboard Updates:**
- Updated every 30 seconds
- Real-time flag status
- Point totals refreshed
- Broadcast to all players

## Team System

### Team Assignment

**Assignment Rules:**
- Players assigned to Team 1 or Team 2
- Teams balanced by player count
- Party members kept together
- Level balancing when possible

**Team Limits:**
- Maximum players per team (e.g., 20 per team)
- Minimum players per team (e.g., 4 per team)
- Event starts when minimum players reached

### Team Spawn Points

**Spawn Locations:**
- Each team has designated spawn area
- Spawn points near team's defensive flag
- Safe zone around spawn points
- Players respawn at team spawn after death

**Respawn Mechanics:**
- 30-second respawn timer
- Players respawn at team spawn point
- Invulnerability period after respawn (5 seconds)
- Full resources restored on respawn

## Map Layout

### Map Structure

**Flag Positions:**
- Team 1 flag: Defensive position (one side of map)
- Team 2 flag: Defensive position (opposite side of map)
- Neutral flag: Center position (strategic location)

**Spawn Points:**
- Team 1 spawn: Near Team 1 flag
- Team 2 spawn: Near Team 2 flag
- Spawn points in safe zones

**Map Features:**
- Open areas for combat
- Cover and obstacles for strategy
- Clear paths between flags
- Balanced map design

## Player Mechanics

### Player Actions

**Capture Attempt:**
- Approach flag within capture radius
- Stand still or move slowly within radius
- Avoid taking damage
- Complete 30-second capture timer

**Defense Actions:**
- Guard flags from enemy capture
- Attack capturing enemies
- Use abilities to interrupt capture
- Coordinate with teammates

**Combat:**
- Players can attack enemy team members
- Kills don't award points directly
- Kills help prevent flag captures
- Death results in respawn after 30 seconds

### Player Rewards

**Base Rewards:**
- Winning team: Higher base rewards
- Losing team: Lower base rewards
- Participation rewards for all players

**Performance Bonuses:**
- Flag captures: Bonus for each flag captured
- Flag defenses: Bonus for defending flags
- Kills: Bonus for eliminating enemies
- Assists: Bonus for helping captures

## Event Configuration

### Configurable Parameters

**Event Duration:**
- Default: 15 minutes (900 seconds)
- Configurable per event instance

**Point Values:**
- Points per flag per minute: Default 10
- Configurable point multipliers

**Capture Time:**
- Default: 30 seconds
- Configurable capture duration

**Capture Radius:**
- Default: 5 units
- Configurable flag capture area

**Respawn Timer:**
- Default: 30 seconds
- Configurable respawn delay

**Team Limits:**
- Maximum players per team
- Minimum players to start
- Party size limits

## Integration Points

### With Map Instance System
- Creates isolated map instance
- Uses MAPS_INSTANCE system
- Separate thread for event map
- Isolated entity lists

### With Team System
- Uses team assignment system
- Team-based spawn points
- Team-based flag ownership
- Team-based rewards

### With Combat System
- PvP combat enabled
- Damage interrupts capture
- Kills affect capture attempts
- Death and respawn mechanics

### With Reward System
- Team-based reward distribution
- Individual performance bonuses
- Reward items and currency
- Experience rewards

## Related Documentation

- [EVENT_INSTANCE.md](./EVENT_INSTANCE.md) - Event instance system
- [BATTLECASTLE.md](./BATTLECASTLE.md) - Battle Castle event (similar PvP event)
- [MAPS_INSTANCE.md](../server/MAPS_INSTANCE.md) - Map instances system
- [TEAMS.md](../core/TEAMS.md) - Team system

## See Also

- [Events Overview](../events/) - Events systems overview
- [Server README](../server/README.md) - Server documentation

