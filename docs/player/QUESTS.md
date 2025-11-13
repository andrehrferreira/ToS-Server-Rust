# Quest System Documentation

## Overview

The Quest System is a comprehensive mission system that provides players with various types of objectives and rewards. The system supports traditional quest types (kill creatures, collect items, deliver items, craft items), daily quests, main story progression quests, guild quests, faction quests, location-based quests, and NPC-specific quests. Quests track player progress, store completion status, support favorites, and provide rewards upon completion.

## Key Features

- **Multiple Quest Types**: Collect, Kill, Delivery, Crafting quests
- **Daily Quests**: Rotating daily quest pools with reset functionality
- **Main Story Quests**: Progressive storyline quests for game progression
- **Guild Quests**: Guild-specific objectives and rewards
- **Faction Quests**: Faction-based missions and reputation
- **Location Quests**: Area-specific objectives for exploration
- **NPC Quests**: Quest givers with profile-based quest assignment
- **Progress Tracking**: Real-time progress tracking for all objectives
- **Favorites System**: Players can mark quests as favorites
- **Metadata Persistence**: Quest state saved and loaded from metadata
- **Reward System**: Multiple reward types (items, gold, experience)

## Quest Types

### Collect Quest

**Type:** `QuestType.Collect`

**Objective:**
- Collect specific items in specified quantities
- Items can be gathered, looted, or obtained through various means
- Multiple items can be required in a single quest

**Properties:**
- `itemCollect`: Array of `IItemRef` (ItemName, Quantity)
- Progress tracked by item collection events
- Items consumed when quest is completed

**Example:**
```typescript
public override itemCollect: Array<IItemRef> = [
    { ItemName: "Wood", Quantity: 50 },
    { ItemName: "Stone", Quantity: 50 }
];
```

**Use Cases:**
- Gathering resources for NPCs
- Collecting rare items
- Resource gathering objectives
- Material collection for crafting

### Kill Quest

**Type:** `QuestType.KillerMobiles`

**Objective:**
- Kill specific creatures in specified quantities
- Track kills by creature name
- Progress updates on creature death

**Properties:**
- `creatureToKill`: Name of creature to kill
- `progress`: Number of creatures killed
- Progress tracked by combat system

**Example:**
```typescript
public override creatureToKill = "Giant Rat";
public override progress = 0; // Tracks kills
```

**Use Cases:**
- Monster hunting objectives
- Boss kill quests
- Faction enemy elimination
- Rare creature hunting

### Delivery Quest

**Type:** `QuestType.Delivery`

**Objective:**
- Deliver specific item to target NPC
- Item must be in player inventory
- Item consumed on delivery

**Properties:**
- `itemToDelivery`: Name of item to deliver
- Target NPC specified in quest metadata
- Item verification on delivery

**Example:**
```typescript
public override itemToDelivery = "Letter";
```

**Use Cases:**
- Item transport missions
- Message delivery
- Package delivery
- Trade route objectives

### Crafting Quest

**Type:** `QuestType.Crafting`

**Objective:**
- Craft specific items in specified quantities
- Items must be crafted by player
- Progress tracked by crafting system

**Properties:**
- `itemCrafting`: Array of `IItemRef` (ItemName, Quantity)
- Requires crafting skill
- Items consumed when quest is completed

**Example:**
```typescript
public override itemCrafting: Array<IItemRef> = [
    { ItemName: "IronSword", Quantity: 5 },
    { ItemName: "LeatherArmor", Quantity: 3 }
];
```

**Use Cases:**
- Crafting skill training
- Equipment production objectives
- Resource conversion quests
- Master craftsman challenges

## Quest Structure

### Base Quest Class

```typescript
export abstract class Quest {
    public namespace: string;           // Unique quest identifier
    public type: QuestType;             // Quest type (Collect, Kill, Delivery, Crafting)
    public name: string;                // Display name
    public description: string;         // Quest description/story
    public progress: number;            // Current progress (0-100 or count)
    public completed: boolean;          // Completion status
    public fav: boolean;                // Favorite flag
    
    // Objectives
    public itemCollect: Array<IItemRef>;     // Items to collect
    public itemCrafting: Array<IItemRef>;    // Items to craft
    public creatureToKill: string;           // Creature to kill
    public itemToDelivery: string;           // Item to deliver
    
    // Rewards
    public rewards: Array<IReward>;          // Quest rewards
}
```

### Quest Interfaces

**IItemRef:**
```typescript
export interface IItemRef {
    ItemName: string;    // Item identifier
    Quantity: number;    // Required quantity
}
```

**IReward:**
```typescript
export interface IReward {
    Item: { new (): any };   // Item class constructor
    Quantity: number;        // Reward quantity
}
```

### Quest Registration

**Static Quest Registry:**
```typescript
Quest.Quests: Map<string, { new (): any }>

// Register quest
Quest.AddQuest("QuestVillageResources", QuestVillageResources);
```

**Quest Lookup:**
- Quests accessed by namespace
- Used for quest instantiation
- Supports dynamic quest loading

## Quest Categories

### Traditional Quests

**Standard Quest Types:**
- Collect items
- Kill creatures
- Deliver items
- Craft items

**Characteristics:**
- Single objective focus
- Clear completion criteria
- Immediate rewards
- Repeatable (if designed)

**Examples:**
- "Collect 50 Wood and 50 Stone"
- "Kill 10 Giant Rats"
- "Deliver Letter to NPC"
- "Craft 5 Iron Swords"

### Main Story Quests

**Progressive Storyline:**
- Sequential quest chain
- Unlocks new areas/content
- Character progression
- Story-driven objectives

**Characteristics:**
- Linear progression
- Unlocks new quests
- Major story milestones
- Unique rewards

**Implementation:**
- Quest chains with prerequisites
- Story progression flags
- Area unlocking mechanics
- Character development

**Example Flow:**
```
Tutorial → Village Setup → First Exploration → 
Dungeon Discovery → Boss Encounter → New Area Unlock
```

### Guild Quests

**Guild-Specific Objectives:**
- Guild reputation objectives
- Guild resource collection
- Guild building projects
- Guild vs Guild objectives

**Characteristics:**
- Requires guild membership
- Guild-wide progress tracking
- Guild-specific rewards
- Guild reputation impact

**Objectives:**
- Collect resources for guild projects
- Complete guild building objectives
- Participate in guild events
- Defend guild territories

**Rewards:**
- Guild contribution points
- Guild chest access
- Guild-specific items
- Guild reputation

### Faction Quests

**Faction-Based Missions:**
- Faction reputation objectives
- Faction vs Faction objectives
- Faction territory control
- Faction resource gathering

**Characteristics:**
- Faction alignment required
- Faction reputation impact
- Faction-specific rewards
- Faction conflict objectives

**Objectives:**
- Complete faction missions
- Eliminate enemy faction members
- Control faction territories
- Gather faction resources

**Rewards:**
- Faction reputation
- Faction-specific items
- Faction vendor access
- Faction titles

### Location Quests

**Area-Specific Objectives:**
- Discover locations
- Explore areas
- Complete area objectives
- Unlock new areas

**Characteristics:**
- Location-based triggers
- Exploration rewards
- Area unlocking mechanics
- Location-specific objectives

**Objectives:**
- Visit specific locations
- Complete area challenges
- Discover hidden areas
- Explore new territories

**Rewards:**
- Area access unlocks
- Location-specific items
- Exploration experience
- Map completion rewards

### Daily Quests

**Rotating Daily Objectives:**
- Daily quest pool
- Daily reset functionality
- Multiple quests per day
- Daily reward bonuses

**Structure:**
```typescript
export class DailyQuests {
    public quests: { new (): any }[];           // Quest class constructors
    public questsProgress: Array<Quest>;        // Active quest instances
    public static QuestList: Map<number, DailyQuests>;
}
```

**Daily Quest Pool:**
- Multiple quests available daily
- Quest pool rotates (optional)
- Player selects from available quests
- Daily reset at server time

**Registration:**
```typescript
DailyQuests.AddQuest(1, new DailyQuests([
    QuestVillageResources,
    QuestFeedingTheVillage,
    QuestHelpTheCarpenter,
    // ... more quests
]));
```

**Retrieval:**
```typescript
const dailyQuests = DailyQuests.GetQuests(1, metadata);
```

**Reset Mechanism:**
- Daily reset at specified time
- Progress reset for incomplete quests
- New quest pool selection
- Daily reward bonuses reset

### NPC Quests

**NPC Profile System:**
- NPCs have quest profiles
- Quests assigned to NPCs
- NPC interaction triggers quests
- NPC-specific quest chains

**NPC Profile Structure:**
```typescript
export class NPCProfile {
    public Namespace: string;        // NPC identifier
    public Quests: Array<Quest>;     // Available quests
}
```

**Profile Registration:**
```typescript
NPCProfile.addProfile(new CookieProfile());
```

**Profile Lookup:**
```typescript
const profile = NPCProfile.getProfile("Cookie");
```

**NPC Quest Assignment:**
- NPCs can have multiple quests
- Quest availability based on conditions
- Quest chains through NPCs
- NPC reputation affects quest availability

**Example:**
```typescript
export class CookieProfile extends NPCProfile {
    public override Namespace = "Cookie";
    public override Quests = [
        new QuestApplePie(),
        // ... more quests
    ]
}
```

## Quest Progress System

### Progress Tracking

**Progress Types:**
- **Count-based**: Progress increments by count (kills, items collected)
- **Percentage-based**: Progress as percentage (0-100)
- **Boolean**: Completed/not completed

**Progress Updates:**
- Collect quests: Updated on item acquisition
- Kill quests: Updated on creature death
- Delivery quests: Updated on item delivery
- Crafting quests: Updated on item crafting

**Progress Calculation:**
```typescript
// For collect quests
progress = (collectedItems / requiredItems) * 100;

// For kill quests
progress = killsCount;

// For delivery quests
progress = itemDelivered ? 100 : 0;

// For crafting quests
progress = (craftedItems / requiredItems) * 100;
```

### Completion Detection

**Completion Criteria:**
- All objectives met
- Progress reaches 100% or required count
- All items collected/crafted/delivered
- All creatures killed

**Completion Process:**
1. Check if all objectives are met
2. Set `completed = true`
3. Distribute rewards
4. Update quest metadata
5. Trigger completion events

**Completion Events:**
- Quest completion notification
- Reward distribution
- Progress tracking update
- Quest chain progression (if applicable)

## Quest Metadata System

### Metadata Structure

**Base Metadata:**
```typescript
{
    namespace: string,      // Quest identifier
    completed: boolean,     // Completion status
    progress: number,       // Current progress
    fav: boolean           // Favorite flag
}
```

**Public Metadata (with appendPublicData):**
```typescript
{
    namespace: string,
    completed: boolean,
    progress: number,
    fav: boolean,
    name: string,
    description: string,
    type: QuestType,
    reward: Array<{ i: string, q: number }>,  // Item namespace, quantity
    itemsCollect: Array<{ i: string, q: number }>  // Item name, quantity
}
```

### Metadata Parsing

**ParseMetadata:**
- Loads quest state from saved metadata
- Restores completion status
- Restores progress
- Restores favorite flag

**GetMetadata:**
- Generates metadata for saving
- Includes base data (namespace, completed, progress, fav)
- Optionally includes public data (name, description, rewards)

**Metadata Storage:**
- Stored per player
- Persisted to database
- Loaded on player login
- Updated on quest progress/completion

### Daily Quest Metadata

**Daily Quest Metadata Structure:**
- Array of quest metadata objects
- Indexed by quest namespace
- Includes all quest states

**ParseMetadata:**
- Parses metadata array
- Creates quest instances
- Restores quest states
- Links to daily quest pool

**GenerateMetadata:**
- Generates metadata array
- Includes all quest states
- Optionally includes public data
- Can export as JSON string

## Quest Rewards

### Reward Types

**Item Rewards:**
- Equipment items
- Consumable items
- Resources
- Special items

**Gold Rewards:**
- Direct gold payment
- Varies by quest difficulty
- Scales with quest level

**Experience Rewards:**
- Skill experience
- Character experience
- Reputation experience

**Special Rewards:**
- Titles
- Access unlocks
- Reputation
- Faction standing

### Reward Distribution

**Reward Process:**
1. Quest completion detected
2. Reward list processed
3. Items added to inventory
4. Gold added to player
5. Experience granted
6. Special rewards applied

**Reward Structure:**
```typescript
public override rewards: Array<IReward> = [
    { Item: GoldCoin, Quantity: 100 },
    { Item: SmallLifePotion, Quantity: 10 },
    { Item: IronSword, Quantity: 1 }
];
```

**Multiple Rewards:**
- Quests can have multiple rewards
- All rewards distributed on completion
- Inventory space checked
- Overflow handling

## Quest Favorites System

### Favorite Functionality

**Favorite Flag:**
- `fav: boolean` property
- Player can mark quests as favorites
- Favorites displayed prominently
- Favorites persist across sessions

**Favorite Use Cases:**
- Mark important quests
- Track priority objectives
- Organize quest list
- Quick access to key quests

**Favorite Management:**
- Toggle favorite status
- Filter by favorites
- Display favorites first
- Favorite notifications

## Quest Integration

### Player Integration

**Player Quest Tracking:**
- Active quests list
- Completed quests list
- Quest progress tracking
- Quest metadata storage

**Quest Assignment:**
- NPC interaction
- Quest board interaction
- Automatic assignment (story quests)
- Daily quest selection

**Quest Completion:**
- Automatic completion detection
- Reward distribution
- Progress updates
- Completion notifications

### Combat System Integration

**Kill Quest Tracking:**
- Creature death events
- Quest progress updates
- Kill count tracking
- Completion detection

**Integration Points:**
- Creature death handler
- Quest progress update
- Completion check
- Reward distribution

### Gathering System Integration

**Collect Quest Tracking:**
- Item acquisition events
- Quest progress updates
- Item count tracking
- Completion detection

**Integration Points:**
- Gathering completion handler
- Loot acquisition handler
- Quest progress update
- Completion check

### Crafting System Integration

**Crafting Quest Tracking:**
- Item crafting events
- Quest progress updates
- Crafted item tracking
- Completion detection

**Integration Points:**
- Crafting completion handler
- Quest progress update
- Completion check
- Reward distribution

### Interaction System Integration

**Delivery Quest Tracking:**
- NPC interaction events
- Item delivery verification
- Quest completion detection
- Reward distribution

**Integration Points:**
- NPC interaction handler
- Item verification
- Quest completion check
- Reward distribution

## Quest Examples

### Collect Quest Example

```typescript
export class QuestVillageResources extends Quest {
    public override namespace = "QuestVillageResources";
    public override type = QuestType.Collect;
    public override name = "Village Resources";
    public override description = "Gather wood and stone for village construction.";
    public override progress = 0;

    public override itemCollect: Array<IItemRef> = [
        { ItemName: "Wood", Quantity: 50 },
        { ItemName: "Stone", Quantity: 50 }
    ];

    public override rewards: Array<IReward> = [
        { Item: GoldCoin, Quantity: 100 },
        { Item: Water, Quantity: 30 }
    ]
}
```

### Kill Quest Example

```typescript
export class QuestRatExtermination extends Quest {
    public override namespace = "QuestRatExtermination";
    public override type = QuestType.KillerMobiles;
    public override name = "Rat Extermination";
    public override description = "Eliminate 10 Giant Rats threatening the village.";
    public override progress = 0;
    public override creatureToKill = "Giant Rat";

    public override rewards: Array<IReward> = [
        { Item: GoldCoin, Quantity: 50 },
        { Item: RatTail, Quantity: 5 }
    ]
}
```

### Delivery Quest Example

```typescript
export class QuestMessageDelivery extends Quest {
    public override namespace = "QuestMessageDelivery";
    public override type = QuestType.Delivery;
    public override name = "Message Delivery";
    public override description = "Deliver a letter to the village elder.";
    public override progress = 0;
    public override itemToDelivery = "Letter";

    public override rewards: Array<IReward> = [
        { Item: GoldCoin, Quantity: 25 }
    ]
}
```

### Crafting Quest Example

```typescript
export class QuestBlacksmithTraining extends Quest {
    public override namespace = "QuestBlacksmithTraining";
    public override type = QuestType.Crafting;
    public override name = "Blacksmith Training";
    public override description = "Craft 5 Iron Swords to improve your skills.";
    public override progress = 0;

    public override itemCrafting: Array<IItemRef> = [
        { ItemName: "IronSword", Quantity: 5 }
    ];

    public override rewards: Array<IReward> = [
        { Item: GoldCoin, Quantity: 200 },
        { Item: SteelIngot, Quantity: 10 }
    ]
}
```

### Daily Quest Example

```typescript
DailyQuests.AddQuest(1, new DailyQuests([
    QuestVillageResources,
    QuestFeedingTheVillage,
    QuestHelpTheCarpenter,
    QuestHelpTheAlchemist,
    QuestBasicEquipment,
    QuestBasicWeapons,
    QuestJewelers,
    QuestTheAdventurer,
    QuestMoondance
]));
```

### NPC Quest Example

```typescript
export class QuestApplePie extends Quest {
    public override namespace = "QuestApplePie";
    public override type = QuestType.Collect;
    public override name = "Apple Pie";
    public override description = "Bring me 20 apples for a pie.";
    
    public override itemCollect: Array<IItemRef> = [
        { ItemName: "Apple", Quantity: 20 }
    ];

    public override rewards: Array<IReward> = [
        { Item: GoldCoin, Quantity: 10 },
        { Item: ApplePie, Quantity: 1 }
    ]
}

export class CookieProfile extends NPCProfile {
    public override Namespace = "Cookie";
    public override Quests = [
        new QuestApplePie()
    ]
}
```

## Implementation Considerations

### TypeScript Implementation

**Quest Base Class:**
```typescript
export abstract class Quest {
    public static Quests: Map<string, { new (): any }> = new Map();
    
    public namespace: string;
    public type: QuestType;
    public name: string;
    public description: string;
    public progress: number = 0;
    public completed: boolean = false;
    public fav: boolean = false;
    
    public itemCollect: Array<IItemRef> = [];
    public itemCrafting: Array<IItemRef> = [];
    public creatureToKill: string = "";
    public itemToDelivery: string = "";
    
    public rewards: Array<IReward> = [];
    
    public static AddQuest(namespace: string, quest: { new (): any }): void {
        this.Quests.set(namespace, quest);
    }
    
    public ParseMetadata(metadata: any): void {
        this.completed = metadata[this.namespace]?.completed ?? false;
        this.fav = metadata[this.namespace]?.fav ?? false;
        this.progress = metadata[this.namespace]?.progress ?? 0;
    }
    
    public GetMetadata(appendPublicData: boolean = false): any {
        const base = {
            namespace: this.namespace,
            completed: this.completed,
            progress: this.progress,
            fav: this.fav
        };
        
        if (appendPublicData) {
            return {
                ...base,
                name: this.name,
                description: this.description,
                type: this.type,
                reward: this.rewards.map(r => {
                    const item = new r.Item();
                    return { i: item.Namespace, q: r.Quantity };
                }),
                itemsCollect: this.itemCollect.map(i => ({
                    i: i.ItemName,
                    q: i.Quantity
                }))
            };
        }
        
        return base;
    }
}
```

**Daily Quest System:**
```typescript
export class DailyQuests {
    public quests: { new (): any }[];
    public questsProgress: Array<Quest> = [];
    public static QuestList: Map<number, DailyQuests> = new Map();
    
    public constructor(quests: { new (): any }[]) {
        this.quests = quests;
    }
    
    public static AddQuest(index: number, quest: DailyQuests): void {
        this.QuestList.set(index, quest);
    }
    
    public static GetQuests(index: number, metadata: any): DailyQuests | null {
        if (this.QuestList.has(index)) {
            const dailyQuests = this.QuestList.get(index)!;
            dailyQuests.ParseMetadata(metadata);
            return dailyQuests;
        }
        return null;
    }
    
    public ParseMetadata(metadata: any): void {
        this.questsProgress = [];
        const metadataParsed: any = {};
        
        if (metadata) {
            for (const key in metadata) {
                metadataParsed[metadata[key].namespace] = metadata[key];
            }
        }
        
        for (const questClass of this.quests) {
            const quest: Quest = new questClass();
            quest.ParseMetadata(metadataParsed);
            this.questsProgress.push(quest);
        }
    }
    
    public GenerateMetadata(appendPublicData: boolean = false, exportString: boolean = false): any {
        const metadata = this.questsProgress.map(q => q.GetMetadata(appendPublicData));
        return exportString ? JSON.stringify(metadata) : metadata;
    }
}
```

### Progress Tracking Implementation

**Collect Quest Progress:**
```typescript
function onItemAcquired(player: Player, item: Item, quantity: number): void {
    for (const quest of player.activeQuests) {
        if (quest.type === QuestType.Collect && !quest.completed) {
            for (const collectItem of quest.itemCollect) {
                if (collectItem.ItemName === item.Namespace) {
                    // Update progress logic
                    quest.progress += quantity;
                    checkQuestCompletion(quest);
                }
            }
        }
    }
}
```

**Kill Quest Progress:**
```typescript
function onCreatureKilled(player: Player, creature: Creature): void {
    for (const quest of player.activeQuests) {
        if (quest.type === QuestType.KillerMobiles && !quest.completed) {
            if (quest.creatureToKill === creature.Name) {
                quest.progress++;
                checkQuestCompletion(quest);
            }
        }
    }
}
```

**Crafting Quest Progress:**
```typescript
function onItemCrafted(player: Player, item: Item, quantity: number): void {
    for (const quest of player.activeQuests) {
        if (quest.type === QuestType.Crafting && !quest.completed) {
            for (const craftItem of quest.itemCrafting) {
                if (craftItem.ItemName === item.Namespace) {
                    quest.progress += quantity;
                    checkQuestCompletion(quest);
                }
            }
        }
    }
}
```

**Delivery Quest Progress:**
```typescript
function onItemDelivered(player: Player, npc: NPC, item: Item): void {
    for (const quest of player.activeQuests) {
        if (quest.type === QuestType.Delivery && !quest.completed) {
            if (quest.itemToDelivery === item.Namespace) {
                quest.progress = 100;
                quest.completed = true;
                distributeRewards(player, quest);
            }
        }
    }
}
```

### Completion Detection

```typescript
function checkQuestCompletion(quest: Quest): void {
    let completed = false;
    
    switch (quest.type) {
        case QuestType.Collect:
            completed = quest.itemCollect.every(item => {
                const collected = getCollectedQuantity(quest, item.ItemName);
                return collected >= item.Quantity;
            });
            break;
            
        case QuestType.KillerMobiles:
            // Progress is kill count, completion checked externally
            break;
            
        case QuestType.Crafting:
            completed = quest.itemCrafting.every(item => {
                const crafted = getCraftedQuantity(quest, item.ItemName);
                return crafted >= item.Quantity;
            });
            break;
            
        case QuestType.Delivery:
            completed = quest.progress === 100;
            break;
    }
    
    if (completed && !quest.completed) {
        quest.completed = true;
        distributeRewards(player, quest);
        updateQuestMetadata(player, quest);
    }
}
```

## Future Enhancements

### Potential Improvements

**Quest Chains:**
- Sequential quest dependencies
- Quest unlock system
- Story progression tracking
- Branching quest paths

**Quest Sharing:**
- Party quest sharing
- Guild quest coordination
- Quest collaboration

**Dynamic Quests:**
- Procedurally generated quests
- Dynamic objectives
- Adaptive difficulty
- Context-aware quests

**Quest Rewards:**
- Reputation rewards
- Title rewards
- Access unlocks
- Cosmetic rewards

**Quest UI:**
- Quest log interface
- Quest tracking UI
- Quest map markers
- Quest notifications

**Quest Events:**
- Time-limited quests
- Seasonal quests
- Event quests
- Special occasion quests

## Related Documentation

- [PLAYER.md](./PLAYER.md) - Player entity system
- [SKILLS.md](./SKILLS.md) - Skill system (crafting quests)
- [GATHERING.md](./GATHERING.md) - Gathering system (collect quests)
- [GUILD.md](./GUILD.md) - Guild system (guild quests)
- [INTERACTION.md](./INTERACTION.md) - Interaction system (NPC quests, delivery)

