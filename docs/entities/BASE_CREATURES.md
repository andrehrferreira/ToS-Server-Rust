# Base Creature Classes

## Overview

Base creature classes provide common resistance patterns and creature type definitions for different creature categories. These base classes are extended by specific creature implementations to inherit resistance patterns.

## Base Classes

### BasePlant

```typescript
class BasePlant extends Creature {
    public override creatureType = CreatureType.Common
    
    constructor() {
        super()
        this.setFireResistence(-20)    // Weak to fire
        this.setColdResistence(20)     // Resistant to cold
        this.setPoisonResistence(100)  // Immune to poison
        this.setEnergyResistence(50)   // Resistant to energy
    }
}
```

**Resistance Pattern:**
- **Fire**: -20% (weakness)
- **Cold**: +20% (resistance)
- **Poison**: +100% (immunity)
- **Energy**: +50% (resistance)

**Usage:**
- Plant-based creatures
- Nature entities
- Vegetation monsters

**Examples:**
- Plant Monster
- Oak Tree Ent
- Mushroom Monster

### BaseUndead

```typescript
class BaseUndead extends Creature {
    public override creatureType = CreatureType.Undead
    
    constructor() {
        super()
        this.setFireResistence(20)     // Resistant to fire
        this.setColdResistence(20)     // Resistant to cold
        this.setPoisonResistence(20)   // Resistant to poison
        this.setEnergyResistence(20)   // Resistant to energy
        this.setLightResistence(-50)   // Weak to light
        this.setDarkResistence(50)     // Resistant to dark
    }
}
```

**Resistance Pattern:**
- **Fire**: +20% (resistance)
- **Cold**: +20% (resistance)
- **Poison**: +20% (resistance)
- **Energy**: +20% (resistance)
- **Light**: -50% (weakness)
- **Dark**: +50% (resistance)

**Usage:**
- Undead creatures
- Zombies, skeletons
- Death-themed entities

**Examples:**
- Skeleton
- Skeleton Mage
- Skeleton Archer
- Skeleton Knight
- Skeleton Assassin
- Mummy
- Undertaker
- Ancient Unread Flamen

### BaseDragon

```typescript
class BaseDragon extends Creature {
    public override creatureType = CreatureType.Dragon
    
    constructor() {
        super()
        this.setFireResistence(100)    // Immune to fire
        this.setColdResistence(-20)    // Weak to cold
        this.setPoisonResistence(50)   // Resistant to poison
        this.setEnergyResistence(50)   // Resistant to energy
        this.setLightResistence(50)    // Resistant to light
        this.setDarkResistence(50)     // Resistant to dark
    }
}
```

**Resistance Pattern:**
- **Fire**: +100% (immunity)
- **Cold**: -20% (weakness)
- **Poison**: +50% (resistance)
- **Energy**: +50% (resistance)
- **Light**: +50% (resistance)
- **Dark**: +50% (resistance)

**Usage:**
- Dragon creatures
- Fire dragons
- Powerful entities

### BaseWaterMonster

```typescript
class BaseWaterMonster extends Creature {
    public override creatureType = CreatureType.Common
    
    constructor() {
        super()
        this.setFireResistence(80)     // Highly resistant to fire
        this.setColdResistence(-20)    // Weak to cold
        this.setPoisonResistence(10)   // Slightly resistant to poison
        this.setEnergyResistence(-50)   // Weak to energy
    }
}
```

**Resistance Pattern:**
- **Fire**: +80% (high resistance)
- **Cold**: -20% (weakness)
- **Poison**: +10% (slight resistance)
- **Energy**: -50% (weakness)

**Usage:**
- Water-based creatures
- Aquatic monsters
- Sea entities

### BaseDemon

```typescript
class BaseDemon extends Creature {
    public override creatureType = CreatureType.Demon
    
    constructor() {
        super()
        this.setFireResistence(50)     // Resistant to fire
        this.setColdResistence(-20)    // Weak to cold
        this.setPoisonResistence(20)   // Resistant to poison
        this.setEnergyResistence(20)   // Resistant to energy
        this.setLightResistence(-70)   // Very weak to light
        this.setDarkResistence(70)     // Highly resistant to dark
    }
}
```

**Resistance Pattern:**
- **Fire**: +50% (resistance)
- **Cold**: -20% (weakness)
- **Poison**: +20% (resistance)
- **Energy**: +20% (resistance)
- **Light**: -70% (major weakness)
- **Dark**: +70% (high resistance)

**Usage:**
- Demon creatures
- Evil entities
- Dark-themed monsters

## Resistance Patterns

### Pattern Analysis

**Common Patterns:**
- Elemental weaknesses
- Type-based resistances
- Thematic consistency
- Gameplay balance

**Weakness Patterns:**
- Opposite elements (fire/cold)
- Thematic weaknesses (undead/light)
- Balance considerations

**Resistance Patterns:**
- Type-appropriate resistances
- Thematic consistency
- Challenge scaling

## Creature Type Assignment

### Type Usage

**CreatureType.Common:**
- BasePlant
- BaseWaterMonster
- Standard creatures

**CreatureType.Undead:**
- BaseUndead
- Undead entities

**CreatureType.Dragon:**
- BaseDragon
- Dragon entities

**CreatureType.Demon:**
- BaseDemon
- Demon entities

## Extension Pattern

### Usage Example

```typescript
class ForestPlant extends BasePlant {
    constructor() {
        super()
        // Inherits plant resistances
        // Additional properties
        // Custom behaviors
    }
}
```

**Benefits:**
- Inherits resistance pattern
- Consistent behavior
- Easy to extend
- Type safety

### Override Pattern

**Resistance Override:**
- Can override individual resistances
- Maintains base pattern
- Custom adjustments

**Example:**
```typescript
class FirePlant extends BasePlant {
    constructor() {
        super()
        this.setFireResistence(50)  // Override fire weakness
    }
}
```

## Implementation Notes

### Resistance Calculation

**Base Resistances:**
- Set in constructor
- Applied to creature
- Used in damage calculation
- Can be modified

### Type Consistency

**Type Assignment:**
- Set in constructor
- Used for AI behavior
- Affects targeting
- Thematic consistency

### Balance Considerations

**Weakness/Resistance Balance:**
- Prevents invincibility
- Creates strategic choices
- Encourages variety
- Maintains challenge

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Creature system
- [RESISTANCES.md](../core/RESISTANCES.md) - Resistance system
- [DAMAGE.md](../core/DAMAGE.md) - Damage calculation

