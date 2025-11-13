# Champions System Documentation

## Overview

The Champions System is a dynamic content system that creates progressive, level-based encounters at specific locations in the world. Champions spawn progressively stronger creatures as players defeat them, advancing through multiple levels until a final boss is summoned. The system features a unique progression mechanic where each level requires defeating a specific number of creatures before advancing, with the boss spawning after completing all levels. Champions are persistent world content that reset after boss defeat or timeout, providing continuous challenging encounters for players.

## Key Features

- **Progressive Level System**: 8 levels of increasing difficulty
- **Dynamic Creature Spawning**: Creatures spawn based on current level
- **Progress-Based Advancement**: Defeat required amount of creatures to advance levels
- **Boss Encounter**: Elite boss spawns after completing all 8 levels
- **Boss Mechanics**: Boss spawns additional creatures when taking physical damage
- **Auto-Reset System**: Champion resets after boss defeat or timeout
- **Persistent Instances**: Champions remain active until boss is defeated or times out
- **Multiple Champion Types**: System supports different champion types (e.g., Joker)
- **Event System**: Level and progress change events for UI updates

## Architecture

### Class Hierarchy

```
RandomContent (base class)
  └── ChampionSystem (base champion class)
       └── JokerChampion (specific implementation)
```

### Core Components

**ChampionsManager:**
- Static manager for all active champions
- Creates and manages champion instances
- Provides lookup functionality by GUID

**ChampionSystem:**
- Base class for all champions
- Defines level progression structure
- Manages creature spawning and boss mechanics

**JokerChampion:**
- Specific champion implementation
- Defines creature pools per level
- Implements level progression logic

## Champion Lifecycle

### Phase 1: Initialization

**Creation:**
- Admin or system calls `ChampionsManager.CreateNewChampion()`
- Champion created at specified position in scene
- Champion GUID assigned (currently hardcoded for Joker)
- Champion level set (default 0)
- `Initialize()` called

**Initialization Process:**
1. Level progression data configured (`CreaturesLevel` list)
2. Each level defines:
   - `Mobiles`: Array of creature names that can spawn
   - `Amount`: Number of creatures to defeat to advance
   - `MaxMobile`: Maximum concurrent creatures for this level
3. `CreaturesInstance` list cleared
4. Initial creatures spawned up to `MaxMobile` for current level
5. Champion ready for player interaction

### Phase 2: Level Progression

**Creature Spawning:**
- Creatures spawn at champion's position with random offset
- Random offset within `RadiusRespawn` (20 units default)
- Random creature selected from level's `Mobiles` array
- Creature difficulty set to `Regular`
- Creature added to `CreaturesInstance` list
- Creature's `Died` event subscribed to `MonsterDie` handler

**Creature Combat:**
- Players engage and defeat creatures
- Each creature death triggers `MonsterDie` handler
- Creature removed from `CreaturesInstance` list
- Progress incremented by 1
- If progress exceeds level's `Amount`, level advances

**Level Advancement:**
- When `Progress > CreaturesLevel[Level].Amount`:
  - Level incremented (`Level++`)
  - Progress reset to 0
  - `onChange` event fired with new level and progress
- After 10-second delay, dead creature removed from scene
- After another 10-second delay, new creature spawned to maintain `MaxMobile` count

**Respawn Logic:**
- `RespawnMoreMobiles()` called after creature death
- Checks if level < 8 and boss not summoned
- Checks if current creature count < `MaxMobile` for level
- Spawns new creature if conditions met
- Maintains creature population at level's `MaxMobile`

### Phase 3: Boss Phase

**Boss Summoning:**
- Triggered when `Level >= 8` (after completing all 8 levels)
- Level and progress reset to 0
- `BossSummoned` flag set to `true`
- Boss spawned at position offset (+5, +5) from champion position
- Boss difficulty set to `Elite`
- Boss added to scene
- Boss `Died` event subscribed to `BossDied` handler
- Boss `DamageTaken` event subscribed to `BossTakeDamage` handler

**Boss Combat:**
- Boss engages players
- When boss takes physical damage (`DamageType.Physical`):
  - Additional creature spawned at boss position
  - Creature selected from Level 6 creature pool
  - Creature difficulty set to `Regular`
  - Creature's target set to damage causer
  - Creature added to scene (not tracked in `CreaturesInstance`)
  - Provides additional challenge during boss fight

**Boss Timeout:**
- Boss has 60-minute timeout (3600 seconds)
- Timer counts down in 60-second intervals
- If boss still alive after timeout:
  - Boss removed from scene
  - `BossInstance` set to null
  - Champion resets to level 0

### Phase 4: Reset Phase

**Boss Defeat:**
- When boss dies, `BossDied` handler triggered
- Boss `Died` event unsubscribed
- `BossInstance` set to null
- All creatures in `CreaturesInstance` cleared
- 10-minute delay (600 seconds) before reset
- Champion resets to level 0

**Reset Process:**
- All creatures removed from scene
- Boss removed if still exists
- Level reset to 0
- Progress reset to 0
- `BossSummoned` flag reset to `false`
- 10-second delay
- `Initialize()` called to restart champion

## Level Progression System

### Level Structure

Each level defines:
- **Mobiles**: Array of creature names that can spawn at this level
- **Amount**: Number of creatures that must be defeated to advance
- **MaxMobile**: Maximum number of concurrent creatures for this level

### Joker Champion Levels

**Level 0:**
- Creatures: Bat, Giant Rat, Slime
- Amount: 300 creatures to defeat
- MaxMobile: 50 concurrent creatures

**Level 1:**
- Creatures: Ratkin Slave, Giant Rat
- Amount: 200 creatures to defeat
- MaxMobile: 50 concurrent creatures

**Level 2:**
- Creatures: Ratkin Slave, Ratkin Armored Gunner
- Amount: 150 creatures to defeat
- MaxMobile: 50 concurrent creatures

**Level 3:**
- Creatures: Ratkin Slave, Ratkin Armored Gunner
- Amount: 100 creatures to defeat
- MaxMobile: 40 concurrent creatures

**Level 4:**
- Creatures: Ratkin Armored Gunner, Ratkin Warrior, Ratkin Shaman
- Amount: 80 creatures to defeat
- MaxMobile: 30 concurrent creatures

**Level 5:**
- Creatures: Ratkin Armored Gunner, Ratkin Warrior, Ratkin Shaman
- Amount: 60 creatures to defeat
- MaxMobile: 30 concurrent creatures

**Level 6:**
- Creatures: Ratkin Shaman, Serpent Warrior, Cobra Snake
- Amount: 40 creatures to defeat
- MaxMobile: 20 concurrent creatures

**Level 7:**
- Creatures: Ratkin Shaman, Serpent Warrior, Cobra Snake
- Amount: 30 creatures to defeat
- MaxMobile: 20 concurrent creatures

**Level 8 (Boss Phase):**
- Boss: Joker (Elite difficulty)
- Spawned after completing Level 7
- Boss timeout: 60 minutes

### Progression Flow

```
Level 0 → Defeat 300 → Level 1
Level 1 → Defeat 200 → Level 2
Level 2 → Defeat 150 → Level 3
Level 3 → Defeat 100 → Level 4
Level 4 → Defeat 80 → Level 5
Level 5 → Defeat 60 → Level 6
Level 6 → Defeat 40 → Level 7
Level 7 → Defeat 30 → Level 8 (Boss)
```

**Total Creatures Required:** 960 creatures before boss spawns

## Data Structures

### ChampionsManager

```csharp
public static class ChampionsManager 
{
    public static Dictionary<Guid, ChampionSystem> Champions;
    
    public static void CreateNewChampion(Scene scene, Vector2 position, string name, int level = 0);
    public static ChampionSystem TryGet(Guid guid);
}
```

**Static State:**
- `Champions`: Dictionary mapping GUID to champion instance
- Provides centralized management of all active champions

### ChampionSystem

```csharp
public class ChampionSystem : RandomContent
{
    public class LevelProgress
    {
        public string[] Mobiles;      // Creature names for this level
        public int Amount;             // Creatures to defeat to advance
        public int MaxMobile;          // Max concurrent creatures
    }
    
    public virtual Guid ChampionUUID { get; set; }
    public List<LevelProgress> CreaturesLevel;
    public int Level = 0;
    public int Progress = 0;
    public bool BossSummoned = false;
    
    protected List<Monster> CreaturesInstance;
    protected Monster BossInstance;
}
```

**Properties:**
- `ChampionUUID`: Unique identifier for champion instance
- `CreaturesLevel`: List of level progression configurations
- `Level`: Current level (0-7, then boss at 8)
- `Progress`: Current progress toward next level
- `BossSummoned`: Flag indicating boss phase active
- `CreaturesInstance`: List of active creatures for current level
- `BossInstance`: Reference to boss monster

### JokerChampion

```csharp
public class JokerChampion : ChampionSystem
{
    public delegate void OnChangeEventHandler(int level, int progress);
    public event OnChangeEventHandler onChange;
    
    // Level creature pools
    string[] Level0, Level1, Level2, Level3, Level4, Level5, Level6, Level7;
}
```

**Events:**
- `onChange`: Fired when level or progress changes
- Parameters: new level, new progress
- Used for UI updates and notifications

## Creature Spawning

### Spawn Mechanics

**Position Calculation:**
- Base position: Champion's position
- Random offset: ±`RadiusRespawn` (20 units default)
- Final position: `Position + Random(-RadiusRespawn, RadiusRespawn)`

**Creature Selection:**
- Random creature selected from level's `Mobiles` array
- Uses `Random.Shared.Next(Mobiles.Length)`
- All creatures in pool have equal spawn chance

**Creature Configuration:**
- Difficulty: `MonsterDifficulty.Regular` (for regular creatures)
- Difficulty: `MonsterDifficulty.Elite` (for boss)
- Position: Random offset from spawn point
- Event handlers: `Died` event subscribed

**Spawn Conditions:**
- Level must be < 8
- Boss must not be summoned
- Current creature count < `MaxMobile` for level
- Spawned during initialization or after creature death

### Boss Spawn Mechanics

**Boss Position:**
- Offset from champion position: (+5, +5)
- Fixed offset (not random)
- Spawned at `new Vector2(Position.X + 5, Position.Y + 5)`

**Boss Configuration:**
- Name: `BossName` property (e.g., "Joker")
- Difficulty: `MonsterDifficulty.Elite`
- Event handlers: `Died` and `DamageTaken` subscribed

## Boss Mechanics

### Physical Damage Response

**Trigger:**
- Boss takes physical damage (`DamageType.Physical`)
- `BossTakeDamage` handler called

**Response:**
- Creature spawned at boss position
- Creature selected from Level 6 creature pool
- Creature difficulty: `Regular`
- Creature's target set to damage causer
- Creature added to scene immediately
- Provides additional challenge and threat

**Purpose:**
- Prevents pure physical DPS strategies
- Adds dynamic difficulty
- Rewards magic/ranged damage approaches
- Creates more engaging combat

### Boss Timeout

**Timer:**
- 60-minute (3600 seconds) timeout
- Counts down in 60-second intervals
- Checks every minute if boss still exists

**Timeout Behavior:**
- If boss still alive after timeout:
  - Boss removed from scene
  - `BossInstance` set to null
  - Champion resets to level 0
- Prevents champions from being stuck with unbeatable bosses

## Reset System

### Reset Triggers

**Boss Defeat:**
- Boss dies
- 10-minute delay
- Champion resets

**Boss Timeout:**
- Boss alive for 60 minutes
- Boss removed
- Champion resets immediately

**Manual Reset:**
- `ResetChampion()` method can be called
- All creatures removed
- Boss removed
- Level/progress reset
- 10-second delay
- Reinitialized

### Reset Process

1. All creatures in `CreaturesInstance` removed from scene
2. Boss removed if exists
3. Level set to 0
4. Progress set to 0
5. `BossSummoned` set to `false`
6. 10-second delay
7. `Initialize()` called
8. Champion restarts at level 0

## Event System

### onChange Event

**Purpose:**
- Notify systems of level/progress changes
- Update UI elements
- Trigger notifications

**Signature:**
```csharp
public delegate void OnChangeEventHandler(int level, int progress);
public event OnChangeEventHandler onChange;
```

**Firing:**
- Fired when level advances
- Fired when progress changes (if needed)
- Parameters: new level, new progress

**Usage:**
- UI updates
- Player notifications
- System integrations
- Analytics tracking

## Integration Points

### RandomContent Integration

**Inheritance:**
- `ChampionSystem` inherits from `RandomContent`
- Uses `CreateMonster()` method for creature spawning
- Uses `CreaturesInstance` and `BossInstance` from base class
- Uses `RadiusRespawn` for spawn positioning

**Properties:**
- `IsDynamic`: Always `true` for champions
- `Name`: Champion display name
- `BossName`: Boss creature name
- `Prefab`: Prefab name for champion altar/object
- `ContentId`: Content type identifier
- `RadiusRespawn`: Spawn radius (20 default)

### Scene System

**Scene Integration:**
- Champions exist in specific scenes
- Creatures added to scene via `Scene.Add()`
- Creatures removed via `Scene.Remove()`
- Uses scene's `TimerQueue` for delays
- Uses scene's random for creature selection

### Entity System

**Entity Properties:**
- `BlocksCreature`: `true` (champion blocks creature movement)
- `BlocksProjectile`: `true` (champion blocks projectiles)
- Position-based spawning
- Event-driven creature management

### Monster System

**Monster Creation:**
- Uses `MonsterDefinition.Load()` for creature definitions
- Configures monster difficulty
- Sets monster position and respawn position
- Configures monster skills and abilities
- Handles ranged vs melee monster types

## Implementation Considerations

### TypeScript Implementation

**Champion Manager:**
```typescript
class ChampionsManager {
    private static champions: Map<Guid, ChampionSystem> = new Map();
    
    static createNewChampion(
        scene: Scene, 
        position: Vector2, 
        name: string, 
        level: number = 0
    ): void {
        let champion: ChampionSystem | null = null;
        const guid = Guid.create(); // Generate proper GUID
        
        switch (name) {
            case "Joker":
                champion = new JokerChampion(guid);
                champion.position = position;
                champion.scene = scene;
                break;
        }
        
        if (champion) {
            champion.level = level;
            champion.initialize();
            this.champions.set(guid, champion);
        }
    }
    
    static tryGet(guid: Guid): ChampionSystem | null {
        return this.champions.get(guid) || null;
    }
}
```

**Champion System:**
```typescript
class ChampionSystem extends RandomContent {
    championUUID: Guid;
    creaturesLevel: LevelProgress[] = [];
    level: number = 0;
    progress: number = 0;
    bossSummoned: boolean = false;
    
    protected creaturesInstance: Monster[] = [];
    protected bossInstance: Monster | null = null;
    
    initialize(): void {
        super.initialize();
        // Configure levels
        // Spawn initial creatures
    }
    
    async resetChampion(): Promise<void> {
        // Remove all creatures
        // Remove boss
        // Reset state
        // Reinitialize
    }
    
    async respawnMoreMobiles(): Promise<void> {
        // Check conditions
        // Spawn creature if needed
    }
    
    async monsterDie(self: CreatureEntity, killedBy: Entity): Promise<void> {
        // Remove from list
        // Increment progress
        // Check level advancement
        // Respawn if needed
    }
    
    bossTakeDamage(
        self: CreatureEntity, 
        damageCauser: Entity, 
        damageType: DamageType, 
        value: number
    ): void {
        // Spawn additional creature on physical damage
    }
    
    async bossDied(self: CreatureEntity, killedBy: Entity): Promise<void> {
        // Cleanup
        // Delay
        // Reset
    }
}
```

**Joker Champion:**
```typescript
class JokerChampion extends ChampionSystem {
    onChange?: (level: number, progress: number) => void;
    
    private level0 = ["Bat", "Giant Rat", "Slime"];
    private level1 = ["Ratkin Slave", "Giant Rat"];
    // ... other levels
    
    override initialize(): void {
        super.initialize();
        
        this.creaturesLevel = [
            { mobiles: this.level0, amount: 300, maxMobile: 50 },
            { mobiles: this.level1, amount: 200, maxMobile: 50 },
            // ... other levels
        ];
        
        // Spawn initial creatures
        for (let i = 0; i < this.creaturesLevel[this.level].maxMobile; i++) {
            this.respawnMoreMobiles();
        }
    }
}
```

### Performance Optimizations

**Creature Management:**
- Limit concurrent creatures per level (`MaxMobile`)
- Remove dead creatures promptly
- Use efficient list operations
- Cache level data structures

**Event Handling:**
- Unsubscribe events on cleanup
- Use async/await for delays
- Batch operations when possible
- Avoid memory leaks from event handlers

**Spawn Optimization:**
- Random position calculation cached
- Creature definition loading cached
- Efficient random selection
- Spawn only when needed

### Security Considerations

**GUID Management:**
- Currently hardcoded GUID for Joker (needs proper generation)
- Ensure unique GUIDs for multiple champions
- Validate GUID on lookups

**Level Validation:**
- Validate level bounds (0-7)
- Validate progress values
- Prevent level skipping exploits
- Validate creature counts

**Boss Mechanics:**
- Validate boss spawn conditions
- Prevent duplicate boss spawns
- Validate timeout calculations
- Prevent reset exploits

## Configuration

### Creating New Champions

**Steps:**
1. Create new class extending `ChampionSystem`
2. Override `Name`, `BossName`, `Prefab` properties
3. Define creature pools for each level (Level0-Level7)
4. Configure `CreaturesLevel` in `Initialize()`
5. Add case to `ChampionsManager.CreateNewChampion()`
6. Implement any champion-specific mechanics

**Example Structure:**
```csharp
public class NewChampion : ChampionSystem
{
    public override string Name => "New Champion";
    public override string BossName => "New Boss";
    public override string Prefab => "ChampionAltar";
    
    string[] Level0 = { "Creature1", "Creature2" };
    // ... define all levels
    
    public override void Initialize()
    {
        base.Initialize();
        
        CreaturesLevel.Add(new LevelProgress() { 
            Mobiles = Level0, 
            Amount = 300, 
            MaxMobile = 50 
        });
        // ... add all levels
        
        // Spawn initial creatures
    }
}
```

### Level Configuration

**Amount Values:**
- Higher amounts = longer level duration
- Lower amounts = faster progression
- Balance for desired gameplay pace
- Consider creature difficulty

**MaxMobile Values:**
- Higher values = more concurrent creatures
- Lower values = less crowded encounters
- Balance for performance and difficulty
- Consider spawn radius

**Creature Selection:**
- Mix creature types for variety
- Progressively stronger creatures
- Consider creature abilities
- Balance melee/ranged/magic

## Testing Scenarios

### Basic Progression

1. Champion created at position
2. Level 0 creatures spawn
3. Players defeat creatures
4. Progress increments
5. Level advances after required amount
6. Process repeats through all levels
7. Boss spawns at level 8
8. Boss defeated or times out
9. Champion resets

### Edge Cases

**Rapid Creature Death:**
- Multiple creatures die quickly
- Progress increments correctly
- Level advances properly
- Respawn maintains population

**Boss Physical Damage:**
- Boss takes physical damage
- Additional creatures spawn
- Creatures target damage causer
- Combat remains challenging

**Boss Timeout:**
- Boss survives 60 minutes
- Boss removed automatically
- Champion resets properly
- New cycle begins

**Concurrent Champions:**
- Multiple champions active
- Each maintains own state
- No interference between champions
- Proper GUID management

**Player Disconnect:**
- Players disconnect during fight
- Champion continues normally
- Creatures remain active
- Progress unaffected

## Future Enhancements

### Potential Improvements

**Dynamic Difficulty:**
- Scale creature difficulty based on player count
- Adjust spawn rates based on activity
- Scale boss health based on participants

**Reward System:**
- Rewards for completing levels
- Special rewards for boss defeat
- Participation rewards
- Level-based reward scaling

**Multiple Boss Phases:**
- Multiple boss phases
- Boss transformations
- Progressive boss difficulty
- Boss mechanics per phase

**Champion Variants:**
- Different champion types
- Unique mechanics per champion
- Champion-specific rewards
- Champion-specific creatures

**UI Integration:**
- Level/progress display
- Boss health bar
- Creature count display
- Time remaining indicators

**Persistence:**
- Save champion state
- Resume after server restart
- Track completion statistics
- Historical data

## Related Documentation

- [ENTITIES.md](../core/ENTITIES.md) - Entity system
- [MAPS.md](../server/MAPS.md) - Scene and map system
- [ACTIONS.md](../player/ACTIONS.md) - Action system (creature abilities)
- [DAMAGE.md](../core/DAMAGE.md) - Damage system (boss damage mechanics)

