# Consumables System Documentation

## Overview

Consumables are items that can be used by players to provide temporary or permanent effects, restore resources, or grant rewards. All consumables inherit from the `Consumable` abstract class, which extends `Stackable`, allowing them to be stacked in inventory. The system supports various consumable types including potions, food, gift boxes, experience items, and special consumables.

## Key Features

- **Stackable**: All consumables can be stacked in inventory
- **Usage Effects**: Immediate or over-time effects
- **Cooldowns**: Some consumables have cooldowns to prevent abuse
- **Crafting**: Many consumables can be crafted (potions via Alchemy, food via Cooking)
- **Rarity System**: Consumables can have different rarities affecting potency
- **Visual Effects**: Consumables can trigger visual/audio effects on use
- **Action Integration**: Consumables can trigger actions/spells on use

## Consumable Hierarchy

```
Item (Abstract)
└── Stackable (Abstract)
    └── Resource (Abstract)
        └── Consumable (Abstract)
            ├── Potion (Abstract)
            │   ├── HealthPotion
            │   ├── ManaPotion
            │   ├── StaminaPotion
            │   ├── RejuvenationPotion
            │   └── BuffPotion
            ├── Food (Abstract)
            │   ├── BasicFood
            │   ├── BuffFood
            │   └── SpecialFood
            ├── GiftBox (Abstract)
            │   ├── RegularGiftBox
            │   ├── PremiumGiftBox
            │   └── EventGiftBox
            ├── ExperienceItem
            ├── PowerOfGods
            └── SpecialConsumable
```

---

## 1. Potions

### Overview

Potions are consumables that restore resources (health, mana, stamina) or provide buffs. They are crafted using the Alchemy skill and can have different rarities affecting their potency.

### Potion Types

**Health Potion:**
- Restores health instantly or over time
- Different tiers (Minor, Normal, Major, Greater, Superior)
- Higher tiers restore more health
- Can have instant or over-time healing
- Cooldown: 5-30 seconds depending on tier
- Use time: 0.5-1.5 seconds

**Mana Potion:**
- Restores mana instantly or over time
- Different tiers based on restoration amount
- Essential for spellcasters
- Can have instant or over-time restoration
- Cooldown: 5-30 seconds depending on tier
- Use time: 0.5-1.5 seconds

**Stamina Potion:**
- Restores stamina instantly or over time
- Useful for melee combatants
- Different tiers based on restoration amount
- Cooldown: 5-30 seconds depending on tier
- Use time: 0.5-1.5 seconds

**Rejuvenation Potion:**
- Restores both health and mana simultaneously
- More expensive but convenient
- Different tiers based on restoration amounts
- Cooldown: 10-60 seconds depending on tier
- Use time: 1-2 seconds

**Buff Potion:**
- Provides temporary stat bonuses
- Various buff types (Strength, Dexterity, Intelligence, Constitution, Luck, Charisma)
- Duration-based effects (typically 5-30 minutes)
- Can stack with other buffs (depending on type)
- Cooldown: 60-300 seconds
- Use time: 1.5-3 seconds

### Potion Properties

**Restoration Types:**
- **Flat Amount**: Restores fixed amount (e.g., 50 health)
- **Percentage**: Restores percentage of max resource (e.g., 25% of max health)
- **Over-Time**: Restores gradually over time (e.g., 10 health per second for 10 seconds)

**Rarity Effects:**
- **Common**: Basic restoration, standard cooldowns
- **Uncommon**: Slightly better restoration, slightly shorter cooldowns
- **Rare**: Good restoration, reduced cooldowns
- **Magic**: Excellent restoration, significantly reduced cooldowns
- **Legendary**: Maximum restoration, minimal cooldowns

**Usage Restrictions:**
- Can be used in combat (most potions)
- Can be used out of combat (all potions)
- Some potions have level requirements
- Cooldowns prevent spam usage

---

## 2. Food

### Overview

Food items provide temporary stat bonuses, health regeneration, or other beneficial effects. Food is crafted using the Cooking skill and can have various effects based on ingredients and cooking skill level.

### Food Types

**Basic Food:**
- Provides health regeneration over time
- Restores hunger/satiety
- Basic stat bonuses (small, +1 to +5)
- Common consumable
- No cooldown or short cooldown
- Use time: 1-2 seconds

**Buff Food:**
- Provides significant stat bonuses (+10 to +30)
- Longer duration than potions (15-60 minutes)
- Multiple stat bonuses possible
- More expensive to craft
- Cooldown: 30-60 minutes
- Use time: 2-3 seconds

**Special Food:**
- Unique effects (experience bonus, gathering bonus, gold find bonus, etc.)
- Event-specific food
- Rare ingredients required
- Powerful effects
- Cooldown: 1-24 hours
- Use time: 2-3 seconds

### Food Properties

**Health Regeneration:**
- Amount per tick (e.g., 5-20 health per tick)
- Tick interval (e.g., every 2 seconds)
- Total duration (e.g., 20-60 seconds)

**Stat Bonuses:**
- Can provide bonuses to any stat
- Multiple stats can be buffed simultaneously
- Duration typically 15-60 minutes
- Bonuses stack with equipment and other buffs

**Special Effects:**
- **Experience Bonus**: Increases XP gain (e.g., +20% XP for 1 hour)
- **Gathering Bonus**: Increases gathering success rate and yield
- **Gold Find Bonus**: Increases gold dropped from monsters
- **Movement Speed Bonus**: Increases movement speed
- **Custom Effects**: Event-specific or unique effects

**Satiety System:**
- Food restores hunger/satiety value
- Prevents starvation mechanics
- Different foods provide different satiety values

---

## 3. Gift Boxes

### Overview

Gift boxes are special consumables that contain random rewards when opened. They can contain items, gold, experience, or other rewards. Gift boxes are often obtained from events, quests, or as rewards.

### Gift Box Types

**Regular Gift Box:**
- Common rewards
- Basic items and gold
- Obtained from quests and events
- Stackable
- Guaranteed gold reward
- 1-3 random rewards from common pool
- Use time: 1 second

**Premium Gift Box:**
- Better rewards
- Higher chance for rare items
- May contain equipment, cards, or magic stones
- More valuable
- Guaranteed gold reward (higher amount)
- 2-4 random rewards from premium pool
- Use time: 2 seconds

**Event Gift Box:**
- Event-specific rewards
- Limited-time availability
- Unique items possible
- Themed rewards
- Guaranteed event-specific reward
- 2-5 random rewards from event pool
- Use time: 2 seconds

### Gift Box Properties

**Reward Pools:**
- Each gift box has one or more reward pools
- Rewards have probability weights
- Multiple rewards can be selected from each pool
- Minimum and maximum rewards per box

**Reward Types:**
- **Items**: Potions, resources, equipment
- **Gold**: Fixed or random amounts
- **Experience**: Character or skill experience
- **Magic Stones**: Rare enhancement items
- **Cards**: Collectible cards
- **Equipment**: Random equipment items
- **Custom Rewards**: Event-specific or unique rewards

**Opening Process:**
1. Player uses gift box
2. System rolls rewards from pools
3. Rewards are granted to player
4. Visual/audio effects play
5. Reward notification shown

---

## 4. Experience Items

### Overview

Experience items grant experience points directly to the player when used. These are useful for catching up, power leveling, or providing alternative progression paths.

### Experience Item Types

**Character Experience Scroll:**
- Grants character level experience
- Different amounts (100, 500, 1000, 5000, etc.)
- No cooldown
- Use time: 1 second
- Can be used in or out of combat

**Skill Experience Scroll:**
- Grants experience to specific skill
- Different amounts based on skill
- No cooldown
- Use time: 1 second
- Specifies which skill receives experience

**All Skills Experience Scroll:**
- Grants experience to all skills simultaneously
- Experience distributed evenly across all skills
- More valuable than single-skill scrolls
- No cooldown
- Use time: 1 second

### Experience Item Properties

**Experience Amounts:**
- Small: 100-500 XP
- Medium: 500-2000 XP
- Large: 2000-5000 XP
- Huge: 5000+ XP

**Usage:**
- Instant effect
- No restrictions (can use anytime)
- No cooldowns
- Visual effects on use

---

## 5. Special Consumables

### Overview

Special consumables include unique items with specific effects, such as Power of Gods, transformation items, teleportation scrolls, and other special-use items.

### Power of Gods

Power of Gods is a legendary consumable that permanently increases character attributes. See [POWER_OF_GODS.md](../core/POWER_OF_GODS.md) for detailed documentation.

**Key Features:**
- Permanently increases stat points
- Variants: +10, +20, +30, +50 attribute points
- Extremely rare drop
- Tradeable before use
- Opens attribute distribution interface
- One-time use per variant

### Transformation Items

Items that transform the player into another creature temporarily.

**Properties:**
- Transforms player appearance
- Changes stats (can increase or decrease)
- Duration: 5-60 minutes
- Can be used in or out of combat
- Visual effects on transformation
- Reverts automatically after duration

**Transformation Types:**
- **Creature Transformations**: Transform into specific creature
- **Elemental Transformations**: Transform into elemental form
- **Boss Transformations**: Transform into boss creature (very rare)

### Teleportation Scrolls

Scrolls that allow instant teleportation to specific locations.

**Teleportation Types:**
- **Specific Location**: Teleport to exact coordinates
- **Nearest Town**: Teleport to closest town
- **Home**: Teleport to player's home location
- **Guild Hall**: Teleport to guild hall (if in guild)
- **Saved Location**: Teleport to saved location

**Properties:**
- Instant teleportation
- Some scrolls require specific location to use
- Cooldown: 5-60 minutes
- Use time: 2-5 seconds (channeling)
- Visual effects during teleportation

---

## Consumable Usage System

### Usage Flow

1. **Player Initiates Use**
   - Player double-clicks consumable or uses hotkey
   - System checks if consumable can be used

2. **Validation Checks**
   - Check level requirements
   - Check stat requirements
   - Check cooldown status
   - Check combat state restrictions
   - Verify item exists in inventory

3. **Use Animation**
   - Play use animation (if use_time > 0)
   - Show visual effects
   - Play sound effects

4. **Apply Effects**
   - Execute consumable effects
   - Apply buffs, restore resources, grant rewards
   - Trigger actions/spells if applicable

5. **Consume Item**
   - Reduce stack count (if stackable)
   - Remove item if stack reaches 0
   - Update inventory

6. **Record Usage**
   - Record usage time for cooldown tracking
   - Save player state

### Cooldown System

**Cooldown Types:**
- **No Cooldown**: Can be used repeatedly (most food, experience items)
- **Short Cooldown**: 5-30 seconds (potions)
- **Medium Cooldown**: 30-300 seconds (buff potions, some food)
- **Long Cooldown**: 1-24 hours (special consumables, premium items)

**Cooldown Tracking:**
- Per-player, per-consumable tracking
- Cooldown persists across sessions
- Cooldown shown in UI
- Cannot use while on cooldown

### Usage Restrictions

**Combat Restrictions:**
- **Can Use In Combat**: Most potions, some food
- **Cannot Use In Combat**: Some consumables (gift boxes, teleportation scrolls)
- **Can Use Out Of Combat**: All consumables
- **Cannot Use Out Of Combat**: Rare (combat-only consumables)

**Level Requirements:**
- Some consumables require minimum level
- Higher level consumables provide better effects
- Level requirement shown in item tooltip

**Stat Requirements:**
- Some consumables require minimum stats
- Prevents low-level players from using high-tier items
- Stat requirement shown in item tooltip

---

## Visual and Audio Effects

### Particle Effects

**Potion Effects:**
- **Health Potion**: Green healing particles
- **Mana Potion**: Blue mana particles
- **Stamina Potion**: Yellow stamina particles
- **Buff Potion**: Golden buff particles
- **Rejuvenation Potion**: Multi-colored particles

**Food Effects:**
- **Basic Food**: Simple consumption particles
- **Buff Food**: Enhanced particles with stat colors
- **Special Food**: Unique themed particles

**Gift Box Effects:**
- **Opening**: Explosion of particles
- **Rewards**: Particles for each reward type
- **Rare Reward**: Special particles for rare items

**Experience Item Effects:**
- **Experience Gain**: Glowing particles
- **Level Up**: Celebration particles

### Sound Effects

**Potion Sounds:**
- Potion use sound
- Healing/mana restoration sound
- Buff application sound

**Food Sounds:**
- Eating sound
- Consumption sound

**Gift Box Sounds:**
- Opening sound
- Reward sound
- Rare reward sound

**Special Consumable Sounds:**
- Transformation sound
- Teleportation sound
- Power of Gods sound

---

## Integration with Actions/Spells

Consumables can trigger actions or spells when used:

**Action Integration:**
- Consumable can specify action name to trigger
- Action executes when consumable is used
- Useful for consumables that cast spells or trigger abilities

**Spell Integration:**
- Consumable can specify spell ID to cast
- Spell is cast when consumable is used
- Useful for scrolls and magical consumables

**Examples:**
- Scroll of Fireball: Triggers Fireball spell
- Scroll of Teleport: Triggers Teleportation action
- Buff Scroll: Triggers buff application action

---

## Crafting Consumables

### Potion Crafting (Alchemy)

**Process:**
- Use Alchemy skill
- Requires herbs and alchemical materials
- Skill level affects rarity chances
- Higher quality ingredients = better results
- Never fails (always succeeds with resources)
- Gold cost required

**Potion Tiers:**
- **Minor**: Low skill, common ingredients
- **Normal**: Medium skill, standard ingredients
- **Major**: High skill, good ingredients
- **Greater**: Very high skill, rare ingredients
- **Superior**: Maximum skill, best ingredients

### Food Crafting (Cooking)

**Process:**
- Use Cooking skill
- Requires food ingredients
- Skill level affects quality
- Specialized cooking tools improve results
- Never fails (always succeeds with ingredients)
- Gold cost required

**Food Quality:**
- **Basic**: Low skill, basic ingredients
- **Good**: Medium skill, standard ingredients
- **Excellent**: High skill, quality ingredients
- **Perfect**: Maximum skill, best ingredients

**Specialized Tools:**
- **Basic Cooking Pot**: Standard results
- **Master Cooking Pot**: Improved food quality
- **Artisan Cooking Pot**: Best food quality

---

## Related Documentation

- [ITEMS.md](./ITEMS.md) - Core items system
- [CRAFTING.md](./CRAFTING.md) - Crafting system
- [CONTAINERS.md](./CONTAINERS.md) - Container and inventory system
- [POWER_OF_GODS.md](../core/POWER_OF_GODS.md) - Power of Gods consumable

---

## See Also

- [Items README](./README.md) - Items systems overview
- [Player Systems](../player/) - Player systems
- [Server Economy](../server/ECONOMY.md) - Economy systems
