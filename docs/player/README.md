# Player Systems Documentation

## Overview

This directory contains comprehensive documentation for all player-related systems in the Tales of Shadowland MMORPG. These systems cover character progression, social interactions, economic activities, and gameplay mechanics that define the player experience.

## Quick Start

**New to player systems?** Start here:
1. [PLAYER.md](./PLAYER.md) - Core player progression and character system
2. [SKILLS.md](./SKILLS.md) - Skills system and progression
3. [RACES.md](./RACES.md) - Character races and racial bonuses

## Documentation Index

### Core Player Systems

- **[PLAYER.md](./PLAYER.md)** - Core player system
  - Level progression (Level 1-100)
  - Experience system and sources
  - Stat point allocation (5 points per level)
  - Six core attributes (Strength, Dexterity, Intelligence, Luck, Constitution, Charisma)
  - Level-based XP penalties
  - Character builds and playstyles
  - Monster XP calculation
  - Stat point distribution strategies

- **[SKILLS.md](./SKILLS.md)** - Skills system
  - 24 skills covering all gameplay aspects
  - Skill categories (Combat, Magic, Gathering, Crafting, Special)
  - Experience-based skill progression
  - Skill caps and bonuses
  - Stat integration (primary/secondary stats)
  - Skill point rewards
  - Progressive difficulty curves

- **[RACES.md](./RACES.md)** - Character races
  - Available races and their characteristics
  - Racial bonuses and abilities
  - Race-specific features
  - Race selection impact on gameplay

### Social Systems

- **[PARTY.md](./PARTY.md)** - Party system
  - Party formation and management
  - Party member roles
  - Experience sharing
  - Loot distribution
  - Party buffs and benefits
  - Party chat and communication

- **[GUILD.md](./GUILD.md)** - Guild system
  - Guild creation and management
  - Member hierarchy and permissions
  - Custom guild flags
  - Resource contribution system
  - Shared storage (guild chests)
  - Squad organization
  - Guild-versus-guild (GvG) warfare
  - Castle domination
  - Guild vendors and auction house
  - Recruitment system

- **[INTERACTION.md](./INTERACTION.md)** - Player interactions
  - Player-to-player interactions
  - Social features
  - Communication systems
  - Interaction mechanics

### Economic Systems

- **[TRADE.md](./TRADE.md)** - Trade system
  - Secure player-to-player trading
  - Two-phase acceptance mechanism
  - Item and gold exchange
  - Real-time trade updates
  - Trade validation and security
  - Trade cancellation and cleanup

- **[GATHERING.md](./GATHERING.md)** - Gathering system
  - Resource collection mechanics
  - No skill restrictions (all resources collectable)
  - Skill-based efficiency
  - Optional tool bonuses
  - Tick-based collection system
  - Resource respawn mechanics
  - RMT spots (larger resource spots)
  - Progressive rewards (double/triple collection)
  - Crafted tools system

### Progression Systems

- **[QUESTS.md](./QUESTS.md)** - Quest system
  - Multiple quest types (Collect, Kill, Delivery, Crafting)
  - Daily quests with reset functionality
  - Main story quests
  - Guild quests
  - Faction quests
  - Location-based quests

- **[SPELLS.md](./SPELLS.md)** - Player spells system
  - Scroll-based spell learning
  - Three rarity tiers (Common, Rare, Ancient)
  - Codex system with three spell trees
  - Build limitation through tree selection
  - Spell reset via NPC mage (gold cost)
  - NPC vendors and auction house acquisition
  - Server persistence of unlocked spells

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Player Systems                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Core Systems          Social Systems      Economic         │
│  ───────────          ──────────────      ───────         │
│  • Player              • Party             • Trade         │
│  • Skills              • Guild              • Gathering     │
│  • Races               • Interaction                        │
│                                                             │
│  Progression Systems                                        │
│  ──────────────────                                        │
│  • Quests                                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Key Features

### Character Progression

**Level System:**
- Progression from Level 1 to 100
- Non-linear experience curves
- Four progression phases (Early-Mid, Mid-Late, Late, End Game)
- Level-based XP penalties for lower-level content

**Stat System:**
- 5 stat points per level
- Six core attributes
- Build diversity and customization
- Stat point allocation strategies

**Skills System:**
- 24 skills across multiple categories
- Experience-based progression
- Skill caps and bonuses
- Integration with stat system

### Social Features

**Party System:**
- Cooperative gameplay
- Experience and loot sharing
- Party buffs and benefits

**Guild System:**
- Permanent organizations
- Territorial control
- Guild-versus-guild warfare
- Shared resources and storage

### Economic Activities

**Trading:**
- Secure player-to-player trading
- Item and gold exchange
- Real-time synchronization

**Gathering:**
- Universal resource collection
- Skill-based efficiency
- Tool bonuses
- Progressive rewards

### Content Progression

**Quests:**
- Multiple quest types
- Daily and story quests
- Guild and faction quests
- Location-based content

## System Integration

### Player → Skills
- Skills gain experience from player actions
- Skills can grant stat experience
- Skill levels affect gameplay effectiveness

### Player → Quests
- Quests provide experience and rewards
- Quest completion affects progression
- Story quests guide player progression

### Player → Guild
- Guild membership provides benefits
- Guild activities contribute to progression
- Guild wars and territorial control

### Player → Trade
- Trading enables item exchange
- Economic interactions between players
- Supports crafting and gathering economies

### Player → Gathering
- Gathering provides resources
- Resources used for crafting and trading
- Skill progression improves gathering efficiency

## Common Use Cases

### Starting a New Character

1. **Character Creation**
   - Select race (see [RACES.md](./RACES.md))
   - Understand racial bonuses
   - Plan character build

2. **Early Progression**
   - Level up through combat (see [PLAYER.md](./PLAYER.md))
   - Allocate stat points strategically
   - Start learning skills (see [SKILLS.md](./SKILLS.md))

3. **Questing**
   - Accept quests from NPCs (see [QUESTS.md](./QUESTS.md))
   - Complete objectives
   - Collect rewards

### Mid-Game Activities

1. **Skill Development**
   - Focus on primary skills
   - Balance combat and gathering skills
   - Optimize skill progression

2. **Social Integration**
   - Join or create a party (see [PARTY.md](./PARTY.md))
   - Join a guild (see [GUILD.md](./GUILD.md))
   - Participate in guild activities

3. **Economic Activities**
   - Gather resources (see [GATHERING.md](./GATHERING.md))
   - Trade with other players (see [TRADE.md](./TRADE.md))
   - Participate in economy

### End-Game Content

1. **Guild Warfare**
   - Participate in GvG battles
   - Control territories
   - Manage guild resources

2. **Advanced Progression**
   - Optimize stat distribution
   - Maximize skill levels
   - Complete end-game quests

## Related Documentation

### Server Systems
- [Server Architecture](../server/ARCHITECTURE.md) - Server architecture
- [Economy Systems](../server/ECONOMY.md) - Advanced economy systems
- [Entity System](../entities/OVERVIEW.md) - Entity system

### Core Systems
- [Core Systems](../core/) - Core game systems
- [Combat Systems](../skills/) - Combat and weapon skills
- [Items](../items/) - Items and equipment

### Client Systems
- [Client Architecture](../client/ARCHITECTURE.md) - Client architecture
- [Player Controller](../client/PLAYER_CONTROLLER.md) - Player controller

## Implementation Notes

### Data Persistence

All player data is persisted:
- Character progression (level, stats, skills)
- Quest progress and completion
- Guild membership and roles
- Inventory and equipment
- Trade history (if applicable)

### Server Authority

The server is authoritative for:
- Experience gain validation
- Stat point allocation validation
- Skill progression validation
- Trade completion validation
- Quest progress tracking

### Performance Considerations

- Efficient stat calculation
- Cached skill bonuses
- Optimized quest tracking
- Batch trade operations
- Efficient gathering tick processing

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Core Systems](../core/) - Core game systems

