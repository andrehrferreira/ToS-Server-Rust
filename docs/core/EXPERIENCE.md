# Experience System

## Overview

The Experience system manages player level progression through experience points gained from various activities including combat, crafting, gathering, and quests. Experience requirements increase with each level.

## Experience Sources

### Combat Experience

**Sources:**
- Killing creatures
- Boss defeats
- PvP combat
- Damage dealt

**Calculation:**
- Based on creature level
- Relative to player level
- Experience scaling
- Level difference factor

### Crafting Experience

**Sources:**
- Crafting items
- Skill progression
- Recipe completion
- Quality-based XP

**Calculation:**
- Based on recipe difficulty
- Skill level factor
- Quality multiplier
- Experience scaling

### Gathering Experience

**Sources:**
- Mining resources
- Lumberjacking
- Skinning
- Resource collection

**Calculation:**
- Based on resource tier
- Skill level factor
- Tool quality
- Experience scaling

### Quest Experience

**Sources:**
- Quest completion
- Objective completion
- Daily quests
- Story progression

**Calculation:**
- Quest difficulty
- Level requirements
- Fixed rewards
- Bonus experience

## Level Progression

### Experience Requirements

**Progression:**
- Exponential growth
- Level-based requirements
- Increasing difficulty
- Balanced progression

**Calculation:**
- Base experience per level
- Multiplier per level
- Total experience needed
- Level cap

### Level Up Process

**Process:**
1. Gain experience
2. Check if level up
3. Calculate new level
4. Apply level bonuses
5. Update stats
6. Notify player

**Level Bonuses:**
- Stat points
- Skill points
- Max stats increase
- New abilities

## Experience Calculation

### Combat Experience Formula

**Base Formula:**
```
experience = baseXP * levelFactor * difficultyFactor
```

**Factors:**
- Creature level
- Player level
- Level difference
- Difficulty multiplier

### Skill Experience Formula

**Base Formula:**
```
experience = baseXP * skillLevel * qualityFactor
```

**Factors:**
- Base experience
- Current skill level
- Quality achieved
- Difficulty multiplier

## Level Cap

### Maximum Level

**Cap:**
- Configurable maximum
- Balanced progression
- End-game content
- Prestige system (if exists)

**Post-Cap:**
- Experience still gained
- Prestige levels (if exists)
- Skill progression
- Alternative progression

## Experience Display

### UI Display

**Display:**
- Current experience
- Experience to next level
- Experience bar
- Percentage display

**Updates:**
- Real-time updates
- Smooth progression
- Visual feedback
- Progress tracking

## Implementation Notes

### Experience Storage

**Storage:**
- Database persistence
- Player data
- Experience history
- Level tracking

### Experience Validation

**Validation:**
- Positive values only
- Maximum limits
- Overflow protection
- Data integrity

### Performance

**Optimizations:**
- Efficient calculations
- Batch updates
- Minimal overhead
- Cached values

## Related Documentation

- [PLAYER.md](../player/PLAYER.md) - Player system
- [SKILLS.md](../player/SKILLS.md) - Skills system
- [QUESTS.md](../player/QUESTS.md) - Quest system

