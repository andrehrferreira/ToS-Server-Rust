# Tower Defender Event Documentation

## Overview

Tower Defender is a cooperative PvE event where a team of up to 4 players defends a crystal in the center of an instanced map. The map features walls with 4 entrances, and players must defeat waves of creatures that spawn and attack only the crystal. As players progress through waves, they face increasingly difficult enemies, including mini-bosses every 5 waves and stronger bosses every 20 waves. Individual rewards accumulate as players progress through the event.

## Key Features

- **Cooperative PvE**: Team-based cooperative gameplay
- **Team Size**: Up to 4 players per team
- **Crystal Defense**: Defend central crystal from waves of enemies
- **Wave System**: Progressive waves with increasing difficulty
- **Mini-Bosses**: Mini-boss spawns every 5 waves
- **Boss Encounters**: Stronger boss spawns every 20 waves
- **Enemy Behavior**: Enemies attack only the crystal, not players
- **Progressive Rewards**: Individual rewards accumulate as players progress
- **Instanced Map**: Each event creates an isolated instance of the Tower Defender map
- **Wall Defense**: Map features walls with 4 entrances for strategic defense

## Event Flow

### Phase 1: Event Initialization

**Admin Trigger:**
- Admin starts the event via command or admin interface
- Event status set to `InProgress = true`
- Map instance created for the event

**Map Setup:**
- Instanced map created with Tower Defender layout
- Central crystal positioned in center of map
- Walls constructed around crystal with 4 entrances
- Spawn points configured for enemy waves
- Player spawn area configured
- Map boundaries and safe zones set

**Team Formation:**
- Players form teams (up to 4 players)
- Party members automatically form team
- Solo players can join existing teams or form new ones
- Teams teleported to player spawn area

### Phase 2: Pre-Event Countdown

**3-Minute Countdown:**
- Broadcast: "Tower Defender event starting in 3 minutes!"
- Countdown messages every 30 seconds:
  - "Event starts in 2 minutes 30 seconds"
  - "Event starts in 2 minutes"
  - "Event starts in 1 minute 30 seconds"
  - "Event starts in 1 minute"
  - "Event starts in 30 seconds"
  - "Event starts in 0 seconds - Prepare for the first wave!"

**Crystal Initialization:**
- Crystal spawns at center of map
- Crystal health set to maximum
- Crystal visible to all players
- Crystal health bar displayed
- Crystal status indicators shown

**Wall Setup:**
- Walls spawn around crystal
- 4 entrances configured (North, South, East, West)
- Entrance positions marked
- Wall health displayed (if destructible)

### Phase 3: Wave Phase

**Wave Start:**
- First wave begins after countdown
- Broadcast: "Wave 1 starting!"
- Enemies spawn at designated spawn points
- Enemy count and types displayed
- Wave timer starts

**Enemy Spawning:**
- Enemies spawn at wave start
- Spawn points distributed around map perimeter
- Enemy types based on wave number
- Enemy count increases with wave number
- Enemies pathfind toward crystal

**Enemy Behavior:**
- Enemies target only the crystal
- Enemies do not attack players
- Enemies move toward crystal via shortest path
- Enemies attack crystal when in range
- Enemies ignore player attacks (but can be killed)

**Wave Progression:**
- Wave completes when all enemies defeated
- Next wave starts after short delay (10 seconds)
- Wave number increments
- Difficulty increases with wave number
- Broadcast: "Wave [number] starting!"

**Wave Completion:**
- All enemies defeated
- Short break before next wave
- Rewards distributed for wave completion
- Wave statistics displayed
- Broadcast: "Wave [number] completed!"

### Phase 4: Mini-Boss Phase

**Mini-Boss Spawning:**
- Every 5 waves (waves 5, 10, 15, etc.)
- Mini-boss spawns instead of regular wave
- Broadcast: "Mini-Boss approaching!"
- Mini-boss has increased health and damage
- Mini-boss follows same behavior (attacks crystal only)

**Mini-Boss Mechanics:**
- Higher health pool than regular enemies
- Increased damage to crystal
- May have special abilities
- Requires coordinated team effort
- Defeating mini-boss awards bonus rewards

**Mini-Boss Rewards:**
- Bonus individual rewards
- Higher experience and gold
- Chance for rare items
- Progress milestone rewards

### Phase 5: Boss Phase

**Boss Spawning:**
- Every 20 waves (waves 20, 40, 60, etc.)
- Strong boss spawns
- Broadcast: "Boss approaching - Prepare for battle!"
- Boss has significantly increased health and damage
- Boss may have multiple phases
- Boss follows same behavior (attacks crystal only)

**Boss Mechanics:**
- Very high health pool
- High damage to crystal
- May spawn additional enemies
- May have area attacks (that don't target players)
- Requires full team coordination
- Multiple phases possible

**Boss Rewards:**
- Significant bonus individual rewards
- High experience and gold
- Guaranteed rare items
- Major progress milestone rewards
- Boss-specific loot table

### Phase 6: Event End

**Defeat Condition:**
- Crystal health reaches 0
- Event ends immediately
- Broadcast: "The crystal has been destroyed! Event failed."
- Rewards distributed based on progress
- Players teleported back to safe location

**Victory Condition:**
- Players survive all waves (if wave limit exists)
- Or players defeat final boss
- Event ends successfully
- Broadcast: "Congratulations! You defended the crystal!"
- Final rewards distributed
- Players teleported back to safe location

**Event Completion:**
- Event status set to `Completed = true`
- Final statistics displayed
- Total waves completed
- Total enemies defeated
- Individual performance statistics
- Rewards distributed

**Reward Distribution:**
- Individual rewards accumulated throughout event
- Wave completion rewards
- Mini-boss bonus rewards
- Boss bonus rewards
- Final completion bonuses
- Rewards sent to player inventories

**Cleanup:**
- Players teleported back to safe location
- Map instance destroyed
- Event data cleaned up
- Resources released

## Wave System

### Wave Progression

**Wave Numbering:**
- Waves start at 1 and increment indefinitely
- No maximum wave limit (or configurable limit)
- Difficulty scales with wave number
- Enemy count increases with wave number

**Wave Difficulty:**
- Early waves: Easy enemies, low count
- Mid waves: Moderate enemies, moderate count
- Late waves: Strong enemies, high count
- Scaling formula: Base difficulty × (Wave number × multiplier)

**Wave Timing:**
- Wave starts immediately after previous wave completes
- Short delay between waves (10 seconds)
- Time limit per wave (optional, e.g., 5 minutes)
- Wave fails if time limit exceeded (crystal takes damage)

### Enemy Types

**Regular Enemies:**
- Basic creatures
- Low to moderate health
- Moderate damage to crystal
- Standard movement speed
- Spawn in groups

**Elite Enemies:**
- Stronger than regular enemies
- Higher health and damage
- Faster movement speed
- Spawn in smaller groups
- Appear in later waves

**Mini-Boss Enemies:**
- Spawn every 5 waves
- Significantly stronger than regular enemies
- High health and damage
- May have special abilities
- Single spawn per wave

**Boss Enemies:**
- Spawn every 20 waves
- Very high health and damage
- Multiple phases possible
- Special abilities and mechanics
- Single spawn per wave

## Crystal Mechanics

### Crystal Properties

**Crystal Health:**
- Maximum health: Configurable (e.g., 10,000 HP)
- Health bar displayed to all players
- Health percentage shown
- Health updates in real-time

**Crystal Defense:**
- Crystal cannot move
- Crystal cannot attack
- Crystal can be healed (if mechanic exists)
- Crystal takes damage from enemies only

**Crystal Destruction:**
- When health reaches 0, crystal is destroyed
- Event ends immediately
- Defeat condition triggered
- Rewards based on progress up to that point

### Crystal Status

**Status Indicators:**
- Health bar (green/yellow/red based on health percentage)
- Visual effects when taking damage
- Warning effects when health is low
- Status messages broadcast to players

**Health Thresholds:**
- 100-75%: Healthy (green)
- 75-50%: Damaged (yellow)
- 50-25%: Critical (orange)
- 25-0%: Critical (red)
- Warning messages at thresholds

## Map Layout

### Map Structure

**Crystal Position:**
- Central crystal in center of map
- Surrounded by walls
- 4 entrances (North, South, East, West)
- Open area around crystal for combat

**Wall System:**
- Walls around crystal perimeter
- 4 entrances for enemy access
- Walls may be destructible (optional)
- Strategic chokepoints at entrances

**Spawn Points:**
- Enemy spawn points around map perimeter
- Multiple spawn points per direction
- Spawn points distributed evenly
- Enemies spawn and pathfind to crystal

**Player Spawn Area:**
- Safe area for players
- Near crystal but protected
- Respawn point for players
- Starting position for event

## Enemy Behavior

### Enemy AI

**Targeting:**
- Enemies target only the crystal
- Enemies ignore players completely
- Enemies pathfind to crystal via shortest path
- Enemies attack crystal when in range

**Movement:**
- Enemies move toward crystal
- Pathfinding around obstacles
- Enemies use entrances to reach crystal
- Movement speed based on enemy type

**Combat:**
- Enemies attack crystal when in range
- Enemies do not attack players
- Enemies can be killed by players
- Enemies deal damage to crystal over time

**Special Behaviors:**
- Some enemies may have area attacks (crystal-focused)
- Some enemies may spawn additional enemies
- Bosses may have phase transitions
- Mini-bosses may have special abilities

## Reward System

### Individual Rewards

**Wave Completion Rewards:**
- Base rewards per wave completed
- Rewards accumulate throughout event
- Rewards scale with wave number
- Rewards sent after each wave

**Mini-Boss Rewards:**
- Bonus rewards for defeating mini-bosses
- Higher than regular wave rewards
- Rare item chances
- Experience and gold bonuses

**Boss Rewards:**
- Significant rewards for defeating bosses
- Guaranteed rare items
- High experience and gold
- Boss-specific loot table

**Progress Milestones:**
- Rewards at specific wave milestones (10, 20, 30, etc.)
- Cumulative rewards for reaching milestones
- Bonus rewards for high wave completion

### Reward Accumulation

**Reward Tracking:**
- Individual rewards tracked per player
- Rewards accumulate as event progresses
- Rewards distributed at event end
- Rewards based on participation

**Reward Types:**
- Experience points
- Gold currency
- Items (common to rare)
- Special event items
- Achievement progress

## Team Mechanics

### Team Formation

**Team Size:**
- Maximum 4 players per team
- Minimum 1 player (solo play possible)
- Party members automatically form team
- Solo players can join existing teams

**Team Coordination:**
- Players coordinate defense strategies
- Players cover different entrances
- Players focus fire on priority enemies
- Players support each other

### Player Roles

**Tank Role:**
- Focus on blocking enemies
- High health and defense
- Intercept enemies at entrances
- Protect crystal from damage

**Damage Role:**
- Focus on killing enemies quickly
- High damage output
- Clear waves efficiently
- Eliminate threats before they reach crystal

**Support Role:**
- Focus on supporting team
- Healing crystal (if mechanic exists)
- Buffing teammates
- Crowd control abilities

**Hybrid Role:**
- Balanced approach
- Adapt to situation
- Fill gaps in defense
- Versatile gameplay

## Event Configuration

### Configurable Parameters

**Team Size:**
- Maximum players: Default 4
- Minimum players: Default 1
- Configurable per event instance

**Crystal Health:**
- Maximum health: Default 10,000
- Configurable health pool
- Health scaling options

**Wave Settings:**
- Starting wave: Default 1
- Wave difficulty scaling
- Enemy count scaling
- Wave time limits

**Mini-Boss Settings:**
- Mini-boss spawn interval: Default every 5 waves
- Mini-boss difficulty scaling
- Mini-boss reward multipliers

**Boss Settings:**
- Boss spawn interval: Default every 20 waves
- Boss difficulty scaling
- Boss reward multipliers
- Boss phase mechanics

**Reward Settings:**
- Base rewards per wave
- Reward scaling with wave number
- Mini-boss reward multipliers
- Boss reward multipliers
- Milestone rewards

## Integration Points

### With Map Instance System
- Creates isolated map instance
- Uses MAPS_INSTANCE system
- Separate thread for event map
- Isolated entity lists

### With Creature System
- Spawns creatures for waves
- Creature AI for pathfinding
- Creature combat mechanics
- Boss creature mechanics

### With Reward System
- Individual reward distribution
- Progressive reward accumulation
- Wave completion rewards
- Boss defeat rewards

### With Team System
- Team formation and management
- Team-based spawn points
- Team coordination mechanics
- Team-based statistics

## Related Documentation

- [EVENT_INSTANCE.md](./EVENT_INSTANCE.md) - Event instance system
- [CHAMPIONS.md](./CHAMPIONS.md) - Champions system (similar wave-based PvE)
- [MAPS_INSTANCE.md](../server/MAPS_INSTANCE.md) - Map instances system
- [CREATURES.md](../entities/CREATURES.md) - Creature system

## See Also

- [Events Overview](../events/) - Events systems overview
- [Server README](../server/README.md) - Server documentation

