# Core Systems Documentation

## Overview

This directory contains comprehensive documentation for the core game systems that form the foundation of the Tales of Shadowland MMORPG. These systems define fundamental game mechanics including attributes, stats, damage, movement, experience, gold, conditions, status effects, and UI synchronization.

## Quick Start

**New to core systems?** Start here:
1. [CORE.md](./CORE.md) - Core system overview
2. [ENTITIES.md](./ENTITIES.md) - Core entity system
3. [DAMAGE.md](./DAMAGE.md) - Damage calculation system

## Documentation Index

### Core Foundation Systems

- **[CORE.md](./CORE.md)** - Core system overview
  - System architecture
  - Core mechanics overview
  - System integration
  - Fundamental concepts

- **[ENTITIES.md](./ENTITIES.md)** - Core entity system
  - Entity base classes
  - Entity properties
  - Entity state management
  - Entity lifecycle
  - Core entity mechanics

### Attribute and Stat Systems

- **[SECONDARY_STATS.md](./SECONDARY_STATS.md)** - Secondary stats system
  - Derived stats calculation
  - Stat modifiers and bonuses
  - Stat caps and limits
  - Stat display and UI
  - Stat integration with attributes

- **[RESISTANCES.md](./RESISTANCES.md)** - Resistance system
  - Resistance types (Fire, Cold, Poison, Energy, Light, Dark)
  - Resistance calculation
  - Resistance penalties (zone-based)
  - Resistance caps
  - Resistance display

- **[STATUS.md](./STATUS.md)** - Status system
  - Status effect types
  - Status application and removal
  - Status duration and stacking
  - Status effects on gameplay
  - Status synchronization

- **[CONDITIONS.md](./CONDITIONS.md)** - Condition system
  - Condition types (burn, freeze, poison, stun, slow, blind, weakness)
  - Condition application
  - Condition effects and damage
  - Condition duration and removal
  - Condition stacking rules

### Combat and Damage Systems

- **[DAMAGE.md](./DAMAGE.md)** - Damage calculation system
  - Damage types (Physical, Fire, Cold, Poison, Energy, Light, Dark)
  - Damage calculation formulas
  - Damage modifiers
  - Critical hits
  - Damage reduction and mitigation
  - Damage over time (DoT)

### Movement and Positioning

- **[MOVEMENT.md](./MOVEMENT.md)** - Movement system
  - Movement mechanics
  - Position synchronization
  - Movement validation
  - Collision detection
  - Pathfinding integration
  - Movement speed modifiers

### Progression Systems

- **[EXPERIENCE.md](./EXPERIENCE.md)** - Experience system
  - Experience sources
  - Experience calculation
  - Level progression
  - Experience penalties
  - Skill experience
  - Experience sharing (party)

- **[GOLD.md](./GOLD.md)** - Gold economy system
  - Gold sources and sinks
  - Gold transactions
  - Gold storage and limits
  - Gold synchronization
  - Gold burn system integration

### Special Systems

- **[POWER_OF_GODS.md](./POWER_OF_GODS.md)** - Power of Gods system
  - Permanent stat point increases
  - Power of Gods variants (+10, +20, +30, +50)
  - Attribute distribution interface
  - Rarity and acquisition
  - One-time use per variant

- **[ANCIENT_SCROLLS.md](./ANCIENT_SCROLLS.md)** - Ancient Scrolls system
  - Scroll types and effects
  - Scroll usage mechanics
  - Scroll rarity and acquisition
  - Scroll effects on gameplay

- **[FLAGS.md](./FLAGS.md)** - Flag system
  - Flag types and purposes
  - Flag management
  - Flag synchronization
  - Flag effects on gameplay
  - Zone flags and Sacred Lands

### Social Systems

- **[TEAMS.md](./TEAMS.md)** - Team system
  - Team formation
  - Team membership
  - Team benefits
  - Team vs team combat
  - Team synchronization

### UI and Synchronization

- **[UI.md](./UI.md)** - UI synchronization system
  - UI state management
  - Client-server UI sync
  - UI update packets
  - UI element states
  - Real-time UI updates

### Low-Level Systems

- **[BYTEBUFFER.md](./BYTEBUFFER.md)** - ByteBuffer system
  - Zero-allocation binary serialization
  - Network packet encoding/decoding
  - Efficient data structures
  - Memory management
  - Performance optimization

## Key Concepts

### Attribute System

**Core Attributes:**
- Strength (STR) - Physical damage, melee combat
- Dexterity (DEX) - Accuracy, evasion, ranged combat
- Intelligence (INT) - Magic damage, mana pool
- Constitution (CON) - Health pool, physical defense
- Luck (LUK) - Critical chance, rare drops
- Charisma (CHA) - Trading, social interactions

**Attribute Distribution:**
- 5 stat points per level
- Free distribution on level up
- Power of Gods for permanent increases
- Equipment bonuses

### Damage System

**Damage Types:**
- Physical (melee/ranged weapons)
- Fire (burn damage over time)
- Cold (slow effects)
- Poison (poison damage over time)
- Energy (stun effects)
- Light (healing, buffs)
- Dark (debuffs, life drain)

**Damage Calculation:**
- Base damage from weapons/abilities
- Attribute modifiers
- Critical hit multipliers
- Resistance reduction
- Damage over time effects

### Status and Condition System

**Status Effects:**
- Buffs (temporary stat increases)
- Debuffs (temporary stat decreases)
- Healing over time
- Damage over time
- Movement modifiers
- Ability modifiers

**Conditions:**
- Burn (fire damage over time)
- Freeze (movement slow + cold damage)
- Poison (poison damage over time)
- Stun (cannot act)
- Slow (movement speed reduction)
- Blind (reduced accuracy)
- Weakness (damage reduction)

### Experience System

**Experience Sources:**
- Monster kills
- Quest completion
- Gathering resources
- Crafting items
- Exploration
- Event participation

**Experience Calculation:**
- Base XP from source
- Level difference penalties
- Party bonuses
- Event bonuses
- Skill-specific XP

### Gold System

**Gold Sources:**
- Monster drops
- Quest rewards
- Item sales (NPCs, auction house)
- Trading with players
- Event rewards

**Gold Sinks:**
- NPC purchases
- Repair costs
- Teleportation fees
- Auction house fees
- Taxes and penalties

## System Integration

### With Entity System
- Entities use core stats and attributes
- Damage system applies to entities
- Movement system controls entity positions
- Status effects modify entity properties

### With Combat System
- Damage calculation in combat
- Status effects in combat
- Resistance application
- Critical hit mechanics

### With Player System
- Player attributes and stats
- Experience and level progression
- Gold management
- Power of Gods integration

### With Network System
- Core state synchronization
- UI updates via network
- Position synchronization
- Status effect synchronization

## Related Documentation

- [Entities Systems](../entities/) - Entity-related systems
- [Player Systems](../player/) - Player-specific systems
- [Server Systems](../server/) - Server architecture
- [Items Systems](../items/) - Item and equipment systems

## See Also

- [Main README](../../README.md) - Project overview
- [Server README](../server/README.md) - Server documentation
- [Entities README](../entities/README.md) - Entities systems documentation

