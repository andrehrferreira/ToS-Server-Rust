# Player Races System Documentation

## Overview

The player races system allows players to choose their character's race at character creation. Each race provides unique starting attributes, racial skills, and passive bonuses that affect gameplay throughout the character's progression. Races follow classic RPG conventions, offering distinct playstyles and build options.

**Note:** Race selection is permanent and cannot be changed after character creation. Racial bonuses are always active and stack with other bonuses. The system uses 6 core attributes as defined in [PLAYER.md](./PLAYER.md): STR (Strength), DEX (Dexterity), INT (Intelligence), LCK (Luck), CON (Constitution), and CHA (Charisma).

## Key Features

- **9 Playable Races**: Humans, Elves, Half-Elves, Orcs, Devas (Dark Elves), Dwarves, Half-Orcs, Goblins, Half-Humans
- **Racial Attributes**: Each race has base attribute bonuses
- **Racial Skills**: Unique skills available only to specific races
- **Racial Passives**: Permanent passive bonuses
- **Build Diversity**: Different races favor different playstyles
- **Permanent Choice**: Race cannot be changed after creation

## Playable Races

### Humans

**Description:**
Humans are the most versatile and adaptable race, excelling in all areas without specialization. They are balanced and can excel in any build or playstyle.

**Base Attribute Bonuses:**
- **STR**: +2
- **DEX**: +2
- **INT**: +2
- **LCK**: +2
- **CON**: +2
- **CHA**: +2

**Racial Skills:**
- **Adaptability**: Gain +5% experience from all sources
- **Versatility**: +1 skill point per level (instead of standard amount)
- **Human Determination**: Once per day, can reroll a failed skill check or combat roll

**Racial Passives:**
- **Jack of All Trades**: +5% to all skill effectiveness
- **Quick Learner**: +10% experience gain
- **Diplomatic**: +10% reputation gain with NPCs

**Build Focus:**
- Versatile builds
- Hybrid builds
- Any playstyle
- Balanced attributes

**Lore:**
Humans are the most common race, known for their adaptability and ambition. They excel in trade, diplomacy, and can master any skill with dedication.

### Elves

**Description:**
Elves are graceful and intelligent, with a natural affinity for magic and ranged combat. They are long-lived and have keen senses.

**Base Attribute Bonuses:**
- **STR**: +0
- **DEX**: +4
- **INT**: +4
- **LCK**: +0
- **CON**: +1
- **CHA**: +0

**Resource Impact:**
- **HP**: +50 HP (from +1 CON) + +0 HP (from +0 STR) = +50 HP total
- **Mana**: +120 Mana (from +4 INT)
- **Stamina**: +0 Stamina (from +0 STR) + +120 Stamina (from +4 DEX) + +10 Stamina (from +1 CON) = +130 Stamina total

**Racial Skills:**
- **Elven Grace**: +15% movement speed and +10% evasion
- **Nature's Blessing**: Regenerate 1% HP per second when in natural environments
- **Keen Senses**: Detect hidden enemies and traps within 20 meters

**Racial Passives:**
- **Arcane Affinity**: +15% magic damage and +10% mana regeneration
- **Ranged Mastery**: +10% damage with bows and crossbows
- **Longevity**: +20% maximum HP (elves live longer, are more resilient)

**Build Focus:**
- Mage builds
- Ranger builds
- Evasion builds
- Magic damage builds

**Lore:**
Elves are ancient and wise, with deep connections to nature and magic. They excel in archery, magic, and have exceptional dexterity.

### Half-Elves

**Description:**
Half-Elves combine the versatility of humans with the grace and magic affinity of elves. They are adaptable like humans but retain some elven traits.

**Base Attribute Bonuses:**
- **STR**: +1
- **DEX**: +3
- **INT**: +3
- **LCK**: +1
- **CON**: +2
- **CHA**: +1

**Resource Impact:**
- **HP**: +100 HP (from +2 CON) + +10 HP (from +1 STR) = +110 HP total
- **Mana**: +90 Mana (from +3 INT)
- **Stamina**: +20 Stamina (from +1 STR) + +90 Stamina (from +3 DEX) + +20 Stamina (from +2 CON) = +130 Stamina total

**Racial Skills:**
- **Dual Heritage**: Gain benefits from both human and elven racial passives (50% effectiveness)
- **Adaptive Magic**: +10% magic damage and +5% experience gain
- **Social Grace**: +15% reputation gain with both human and elven factions

**Racial Passives:**
- **Versatile**: +5% to all attributes
- **Magic Affinity**: +10% magic damage
- **Diplomatic**: +10% reputation gain

**Build Focus:**
- Hybrid mage builds
- Versatile builds
- Balanced magic/physical builds

**Lore:**
Half-Elves bridge the gap between humans and elves, often feeling like they belong to neither world. They combine human adaptability with elven grace.

### Orcs

**Description:**
Orcs are powerful warriors with exceptional strength and endurance. They excel in melee combat and physical damage.

**Base Attribute Bonuses:**
- **STR**: +5
- **DEX**: +1
- **INT**: +0
- **LCK**: +0
- **CON**: +4
- **CHA**: +0

**Resource Impact:**
- **HP**: +200 HP (from +4 CON) + +50 HP (from +5 STR) = +250 HP total
- **Mana**: +0 Mana (from +0 INT)
- **Stamina**: +100 Stamina (from +5 STR) + +30 Stamina (from +1 DEX) + +40 Stamina (from +4 CON) = +170 Stamina total

**Racial Skills:**
- **Berserker Rage**: When HP drops below 30%, gain +25% damage and +15% attack speed for 10 seconds (cooldown: 5 minutes)
- **Intimidating Presence**: Reduce enemy damage by 10% within 10 meters
- **War Cry**: Stun nearby enemies for 2 seconds (cooldown: 3 minutes)

**Racial Passives:**
- **Brutal Strength**: +20% physical damage
- **Tough Hide**: +15% physical resistance
- **Warrior's Endurance**: +25% maximum HP

**Build Focus:**
- Warrior builds
- Tank builds
- Physical DPS builds
- Melee combat builds

**Lore:**
Orcs are fierce warriors known for their strength and combat prowess. They value honor and strength above all else.

### Devas (Dark Elves)

**Description:**
Devas are dark elves with a natural affinity for shadow magic and stealth. They excel in dark magic and assassination.

**Base Attribute Bonuses:**
- **STR**: +1
- **DEX**: +4
- **INT**: +4
- **LCK**: +0
- **CON**: +1
- **CHA**: +0

**Resource Impact:**
- **HP**: +50 HP (from +1 CON) + +10 HP (from +1 STR) = +60 HP total
- **Mana**: +120 Mana (from +4 INT)
- **Stamina**: +20 Stamina (from +1 STR) + +120 Stamina (from +4 DEX) + +10 Stamina (from +1 CON) = +150 Stamina total

**Racial Skills:**
- **Shadow Step**: Teleport short distances through shadows (cooldown: 30 seconds)
- **Dark Vision**: See in complete darkness and detect hidden enemies
- **Shadow Magic**: +20% dark damage and +15% dark resistance

**Racial Passives:**
- **Shadow Affinity**: +20% dark damage and +15% dark resistance
- **Stealth Mastery**: +25% evasion and +15% movement speed in darkness
- **Assassin's Grace**: +15% critical strike chance

**Build Focus:**
- Rogue builds
- Dark mage builds
- Assassin builds
- Stealth builds

**Lore:**
Devas are dark elves who have embraced shadow magic and stealth. They excel in assassination and dark magic, often operating from the shadows.

### Dwarves

**Description:**
Dwarves are sturdy and resilient, with exceptional physical resistance and crafting skills. They excel in defense and craftsmanship.

**Base Attribute Bonuses:**
- **STR**: +3
- **DEX**: +0
- **INT**: +2
- **LCK**: +2
- **CON**: +5
- **CHA**: +1

**Resource Impact:**
- **HP**: +250 HP (from +5 CON) + +30 HP (from +3 STR) = +280 HP total
- **Mana**: +60 Mana (from +2 INT)
- **Stamina**: +60 Stamina (from +3 STR) + +0 Stamina (from +0 DEX) + +50 Stamina (from +5 CON) = +110 Stamina total

**Racial Skills:**
- **Stone Skin**: +30% physical resistance for 15 seconds (cooldown: 5 minutes)
- **Crafting Mastery**: +25% crafting success rate and +15% item quality
- **Underground Sense**: Detect minerals and hidden passages within 30 meters

**Racial Passives:**
- **Sturdy Build**: +30% physical resistance and +20% maximum HP
- **Crafting Expertise**: +20% crafting success rate
- **Mining Proficiency**: +25% mining yield

**Build Focus:**
- Tank builds
- Crafter builds
- Miner builds
- Defensive builds

**Lore:**
Dwarves are master craftsmen and miners, known for their resilience and skill with metalwork. They excel in defense and crafting.

### Half-Orcs

**Description:**
Half-Orcs combine the strength of orcs with human adaptability. They are powerful warriors with more versatility than pure orcs.

**Base Attribute Bonuses:**
- **STR**: +4
- **DEX**: +1
- **INT**: +1
- **LCK**: +1
- **CON**: +3
- **CHA**: +1

**Resource Impact:**
- **HP**: +150 HP (from +3 CON) + +40 HP (from +4 STR) = +190 HP total
- **Mana**: +30 Mana (from +1 INT)
- **Stamina**: +80 Stamina (from +4 STR) + +30 Stamina (from +1 DEX) + +30 Stamina (from +3 CON) = +140 Stamina total

**Racial Skills:**
- **Fierce Rage**: When HP drops below 40%, gain +20% damage for 8 seconds (cooldown: 4 minutes)
- **Adaptive Combat**: +10% damage with all weapon types
- **Intimidating Strike**: Critical strikes have 25% chance to stun target

**Racial Passives:**
- **Brutal Strength**: +15% physical damage
- **Tough Hide**: +10% physical resistance
- **Versatile Warrior**: +5% damage with all weapon types

**Build Focus:**
- Warrior builds
- Physical DPS builds
- Versatile combat builds

**Lore:**
Half-Orcs combine orc strength with human adaptability, making them powerful and versatile warriors.

### Goblins

**Description:**
Goblins are small, agile, and cunning. They excel in stealth, traps, and have natural luck.

**Base Attribute Bonuses:**
- **STR**: +0
- **DEX**: +3
- **INT**: +2
- **LCK**: +4
- **CON**: +1
- **CHA**: +1

**Resource Impact:**
- **HP**: +50 HP (from +1 CON) + +0 HP (from +0 STR) = +50 HP total
- **Mana**: +60 Mana (from +2 INT)
- **Stamina**: +0 Stamina (from +0 STR) + +90 Stamina (from +3 DEX) + +10 Stamina (from +1 CON) = +100 Stamina total

**Racial Skills:**
- **Sneak Attack**: When attacking from stealth, deal +50% damage
- **Trap Mastery**: +30% trap effectiveness and +20% trap detection
- **Lucky Break**: Once per hour, can reroll any failed roll

**Racial Passives:**
- **Natural Luck**: +20% luck and +15% rare item drop chance
- **Small Stature**: +25% evasion and +15% movement speed
- **Cunning**: +15% critical strike chance

**Build Focus:**
- Rogue builds
- Trap builds
- Luck-based builds
- Stealth builds

**Lore:**
Goblins are small but cunning, known for their luck and skill with traps. They excel in stealth and have a natural affinity for finding valuable items.

### Half-Humans

**Description:**
Half-Humans are versatile hybrids with balanced attributes. They combine human adaptability with traits from their other parent race.

**Base Attribute Bonuses:**
- **STR**: +2
- **DEX**: +2
- **INT**: +2
- **LCK**: +2
- **CON**: +2
- **CHA**: +2

**Racial Skills:**
- **Adaptive Nature**: Gain +5% to all attributes
- **Versatile Skills**: Can learn skills from multiple skill trees
- **Social Adaptability**: +10% reputation gain with all factions

**Racial Passives:**
- **Balanced**: +5% to all attributes
- **Quick Learner**: +10% experience gain
- **Diplomatic**: +10% reputation gain

**Build Focus:**
- Versatile builds
- Hybrid builds
- Balanced builds
- Any playstyle

**Lore:**
Half-Humans are versatile hybrids who can adapt to any situation. They combine human adaptability with traits from their other parent race.

## Race Selection

### Character Creation

**Selection Process:**
1. Player selects race during character creation
2. Race selection is permanent (cannot be changed)
3. Racial bonuses are applied immediately
4. Racial skills are unlocked at level 1

**Visual Customization:**
- Each race has unique visual options
- Customization includes: skin color, hair, facial features
- Some races have unique visual traits (pointed ears for elves, etc.)

### Race Bonuses Application

**Attribute Bonuses:**
- Applied to base attributes at character creation
- Stack with level-up attribute points
- Always active (permanent)

**Racial Skills:**
- Unlocked at level 1
- Cannot be unlearned
- Cooldowns are race-specific
- Some skills have daily/hourly limits

**Racial Passives:**
- Always active
- Stack with equipment bonuses
- Cannot be disabled
- Permanent throughout character's life

## Racial Skills Details

### Skill Cooldowns and Limits

**Daily Skills:**
- Human Determination: Once per day
- Some racial skills have daily limits

**Cooldown Skills:**
- Berserker Rage (Orc): 5 minutes cooldown
- War Cry (Orc): 3 minutes cooldown
- Stone Skin (Dwarf): 5 minutes cooldown
- Shadow Step (Deva): 30 seconds cooldown
- Fierce Rage (Half-Orc): 4 minutes cooldown
- Lucky Break (Goblin): Once per hour

**Passive Skills:**
- Most racial passives are always active
- No cooldowns or limits

### Skill Effectiveness

**Skill Scaling:**
- Racial skills scale with character level
- Some skills scale with relevant attributes
- Racial passives provide flat bonuses

**Example Scaling:**
- Elven Grace: +15% movement speed (flat) + 10% evasion (scales with DEX)
- Berserker Rage: +25% damage (flat) + 15% attack speed (flat)

## Build Recommendations by Race

### Human Builds
- **Recommended**: Any build (most versatile)
- **Best For**: New players, hybrid builds, balanced playstyles

### Elf Builds
- **Recommended**: Mage, Ranger, Evasion builds
- **Best For**: Magic damage, ranged combat, high mobility

### Half-Elf Builds
- **Recommended**: Hybrid mage, versatile builds
- **Best For**: Players who want magic with versatility

### Orc Builds
- **Recommended**: Warrior, Tank, Physical DPS
- **Best For**: Melee combat, high HP, physical damage

### Deva Builds
- **Recommended**: Rogue, Assassin, Dark Mage
- **Best For**: Stealth, dark magic, critical strikes

### Dwarf Builds
- **Recommended**: Tank, Crafter, Miner
- **Best For**: Defense, crafting, mining

### Half-Orc Builds
- **Recommended**: Warrior, Physical DPS
- **Best For**: Melee combat with versatility

### Goblin Builds
- **Recommended**: Rogue, Trap Master, Luck builds
- **Best For**: Stealth, traps, rare item farming

### Half-Human Builds
- **Recommended**: Any build (versatile)
- **Best For**: Balanced playstyles, hybrid builds

## Rust Implementation Considerations

### Data Structures

**Race Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct Race {
    pub race_type: RaceType,
    pub attribute_bonuses: AttributeBonuses,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum RaceType {
    Human,
    Elf,
    HalfElf,
    Orc,
    Deva,
    Dwarf,
    HalfOrc,
    Goblin,
    HalfHuman,
}

#[derive(Clone, Copy)]
pub struct AttributeBonuses {
    pub str: i16,
    pub dex: i16,
    pub int: i16,
    pub lck: i16,
    pub con: i16,
    pub cha: i16,
}

impl Race {
    pub fn new(race_type: RaceType) -> Self {
        let attribute_bonuses = match race_type {
            RaceType::Human => AttributeBonuses {
                str: 2,
                dex: 2,
                int: 2,
                lck: 2,
                con: 2,
                cha: 2,
            },
            RaceType::Elf => AttributeBonuses {
                str: 0,
                dex: 4,
                int: 4,
                lck: 0,
                con: 1,
                cha: 0,
            },
            RaceType::HalfElf => AttributeBonuses {
                str: 1,
                dex: 3,
                int: 3,
                lck: 1,
                con: 2,
                cha: 1,
            },
            RaceType::Orc => AttributeBonuses {
                str: 5,
                dex: 1,
                int: 0,
                lck: 0,
                con: 4,
                cha: 0,
            },
            RaceType::Deva => AttributeBonuses {
                str: 1,
                dex: 4,
                int: 4,
                lck: 0,
                con: 1,
                cha: 0,
            },
            RaceType::Dwarf => AttributeBonuses {
                str: 3,
                dex: 0,
                int: 2,
                lck: 2,
                con: 5,
                cha: 1,
            },
            RaceType::HalfOrc => AttributeBonuses {
                str: 4,
                dex: 1,
                int: 1,
                lck: 1,
                con: 3,
                cha: 1,
            },
            RaceType::Goblin => AttributeBonuses {
                str: 0,
                dex: 3,
                int: 2,
                lck: 4,
                con: 1,
                cha: 1,
            },
            RaceType::HalfHuman => AttributeBonuses {
                str: 2,
                dex: 2,
                int: 2,
                lck: 2,
                con: 2,
                cha: 2,
            },
        };
        
        Race {
            race_type,
            attribute_bonuses,
        }
    }
}
```

### Racial Skills System

**Racial Skills Component:**
```rust
#[derive(Component, Clone)]
pub struct RacialSkills {
    pub race: RaceType,
    pub active_skills: Vec<RacialSkill>,
    pub passive_bonuses: RacialPassives,
}

#[derive(Clone)]
pub struct RacialSkill {
    pub skill_id: u32,
    pub name: String,
    pub cooldown_remaining: u64,
    pub daily_uses_remaining: u8,
    pub hourly_uses_remaining: u8,
}

#[derive(Clone, Copy)]
pub struct RacialPassives {
    pub experience_bonus: f32,
    pub damage_bonus: f32,
    pub resistance_bonus: f32,
    pub movement_speed_bonus: f32,
    pub evasion_bonus: f32,
    pub crafting_bonus: f32,
    pub luck_bonus: f32,
    // ... more passive bonuses
}

impl RacialSkills {
    pub fn new(race: RaceType) -> Self {
        let (active_skills, passive_bonuses) = match race {
            RaceType::Human => create_human_skills(),
            RaceType::Elf => create_elf_skills(),
            // ... other races
            _ => (Vec::new(), RacialPassives::default()),
        };
        
        RacialSkills {
            race,
            active_skills,
            passive_bonuses,
        }
    }
    
    pub fn can_use_skill(&self, skill_id: u32) -> bool {
        if let Some(skill) = self.active_skills.iter().find(|s| s.skill_id == skill_id) {
            skill.cooldown_remaining == 0 
                && (skill.daily_uses_remaining > 0 || skill.daily_uses_remaining == 255)
                && (skill.hourly_uses_remaining > 0 || skill.hourly_uses_remaining == 255)
        } else {
            false
        }
    }
}
```

## Integration with Other Systems

### Status System

**Integration:**
- Racial attribute bonuses are added to base attributes
- Total attributes = Base + Racial + Level-up + Equipment
- See [STATUS.md](../core/STATUS.md) for attribute calculations

### Skill System

**Integration:**
- Racial skills are separate from regular skills
- Racial skills cannot be unlearned
- Some racial skills unlock additional skill tree access
- See [SKILLS.md](./SKILLS.md) for skill system details

### Equipment System

**Integration:**
- Racial bonuses stack with equipment bonuses
- Some equipment may have race-specific bonuses
- Racial passives work with all equipment

### Experience System

**Integration:**
- Racial experience bonuses stack with other bonuses
- Some races gain more experience from specific activities
- Experience bonuses affect leveling speed

## Summary

The player races system:

- Provides 9 playable races with unique characteristics
- Each race has base attribute bonuses
- Each race has unique racial skills and passives
- Race selection is permanent at character creation
- Racial bonuses are always active
- Different races favor different playstyles and builds
- Creates build diversity and replayability
- Integrates with status, skill, equipment, and experience systems

This system creates depth in character creation, allowing players to choose races that match their preferred playstyle while maintaining balance through trade-offs and specialization.

