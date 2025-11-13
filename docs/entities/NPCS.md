# NPCs (Non-Player Characters)

## Overview

NPCs are non-player characters that provide various services including quests, vendor interactions, dialogues, and world interactions. NPCs extend the Creature or Humanoid classes and have fixed positions on maps.

## NPC Types

### Quest NPCs

**Purpose:**
- Provide quests
- Quest progression
- Story elements
- World interaction

**Features:**
- Quest assignment
- Quest completion
- Dialog system
- Quest tracking

### Vendor NPCs

**Purpose:**
- Buy/sell items
- Item trading
- Resource exchange
- Economic interaction

**Features:**
- Vendor inventory
- Buy/sell interface
- Price management
- Transaction handling

### Dialog NPCs

**Purpose:**
- Story progression
- World building
- Information delivery
- Character interaction

**Features:**
- Dialog trees
- Conversation system
- Option selection
- Story elements

## NPC Properties

### Base Properties

**Common Properties:**
- Fixed position
- Map assignment
- Interaction radius
- Dialog/quest data

**Position:**
- X, Y, Z coordinates
- Fixed on map
- Loaded from database
- Persistent

## NPC Interaction

### Interaction System

**Interaction Process:**
1. Player approaches NPC
2. Interaction triggered
3. Check interaction type
4. Open appropriate window
5. Handle interaction

**Interaction Types:**
- Quest interaction
- Vendor interaction
- Dialog interaction
- Custom interaction

### Quest Interaction

**Process:**
1. Player interacts with quest NPC
2. Check available quests
3. Show quest dialog
4. Accept/decline quest
5. Update quest state

**Quest Assignment:**
- Quest availability check
- Quest requirements
- Quest assignment
- Quest tracking

### Vendor Interaction

**Process:**
1. Player interacts with vendor NPC
2. Open vendor window
3. Display vendor inventory
4. Handle buy/sell
5. Process transactions

**Vendor Inventory:**
- Item list
- Prices
- Stock management
- Transaction processing

## NPC Profiles

### Quest Profile

**Profile System:**
- NPC quest assignments
- Quest lists
- Quest availability
- Quest progression

**Example:**
```typescript
class CookieProfile {
    static quests = [QuestApplePie]
}
```

**Usage:**
- Assign quests to NPCs
- Manage quest availability
- Track quest state

## NPC Positioning

### Map Assignment

**Position Loading:**
- Loaded from database
- Fixed positions
- Map assignment
- Persistent storage

**Position Format:**
- X, Y, Z coordinates
- Rotation
- Map namespace
- Persistent

## NPC Behavior

### Static Behavior

**Behavior Type:**
- Fixed positions
- No movement
- Interaction-based
- Scripted behavior

### Dynamic Behavior

**Advanced NPCs:**
- Patrol routes
- Dynamic dialogues
- Event triggers
- Conditional behavior

## Implementation Notes

### NPC Creation

**Creation Process:**
1. Load NPC data from database
2. Create NPC entity
3. Set position
4. Assign quests/vendor data
5. Add to map

### NPC Management

**Management:**
- Map-based storage
- Database persistence
- State management
- Interaction handling

## Related Documentation

- [QUESTS.md](../player/QUESTS.md) - Quest system
- [INTERACTION.md](../player/INTERACTION.md) - Interaction system
- [VENDORS.md](./VENDORS.md) - Vendor system (if exists)

