# Cards System Documentation

## Overview

Cards are extremely rare collectible items that provide powerful effects beyond resistance bonuses. Cards can be inserted into equipment slots (shared with runes), but also serve as collectibles with Steam rewards, can be used in the internal card game, and can be wagered. Cards are separate from runes and have unique restrictions and mechanics.

**Note:** Cards use the same slot system as runes. See [RUNES.md](./RUNES.md) for slot system details.

## Key Features

- **Extremely Rare Drops**: Less than 1% drop chance from all creatures
- **Boss Cards**: Even rarer drops from bosses
- **Powerful Attributes**: Can provide immunity to resistance types
- **Collectible Function**: Steam rewards for collecting cards
- **Card Game Integration**: Can be used in internal MMO card game
- **Wagering System**: Cards can be wagered/bet
- **Unique Restrictions**: Cannot use 2 cards or 2 runes on same equipment
- **Preserved on Removal**: Cards are not destroyed when removed (unlike runes)

## Card Drop Rates

### Regular Creature Drops

**Drop Rates:**
- **Drop Chance**: Less than 1% (0.5-0.9% depending on creature level)
- **Drop Source**: All creatures can drop cards
- **Card Level**: Matches creature level (scales with creature tier)

**Drop Rate Examples:**
- **Level 1-20 Creatures**: 0.5% card drop chance
- **Level 21-50 Creatures**: 0.7% card drop chance
- **Level 51-80 Creatures**: 0.9% card drop chance
- **Level 81-100 Creatures**: 0.9% card drop chance

### Boss Drops

**Boss Drop Rates:**
- **Drop Chance**: 0.1-0.3% (even rarer than regular creatures)
- **Drop Source**: Bosses and elite bosses
- **Card Level**: Higher level cards (boss-specific cards)
- **Special Cards**: Bosses can drop unique cards not available from regular creatures

**Boss Tier Drop Rates:**
- **Regular Bosses**: 0.1% card drop chance
- **Elite Bosses**: 0.2% card drop chance
- **Raid Bosses**: 0.3% card drop chance

## Card Types and Effects

### Attribute Cards

**Effect**: Increase core attributes (STR, DEX, INT, VIG, AGI, LUC)

**Bonus Range**: +5 to +50 (varies by card level and rarity)

**Examples:**
- "Warrior's Strength Card" - +25 STR
- "Rogue's Dexterity Card" - +30 DEX
- "Mage's Intelligence Card" - +35 INT
- "Tank's Vigor Card" - +40 VIG

**Rarity Distribution:**
- Common: +5 to +15 attributes
- Uncommon: +15 to +25 attributes
- Rare: +25 to +35 attributes
- Very Rare: +35 to +45 attributes
- Boss Cards: +45 to +50 attributes

### Damage Cards

**Effect**: Increase damage output (physical, magical, or elemental)

**Bonus Range**: +10% to +50% damage (varies by card level)

**Examples:**
- "Flame Master Card" - +30% Fire Damage
- "Frost Lord Card" - +25% Cold Damage
- "Thunder Strike Card" - +35% Energy Damage
- "Shadow Blade Card" - +40% Dark Damage

**Rarity Distribution:**
- Common: +10% to +20% damage
- Uncommon: +20% to +30% damage
- Rare: +30% to +40% damage
- Very Rare: +40% to +50% damage
- Boss Cards: +45% to +50% damage

### Defense Cards

**Effect**: Increase defensive stats (armor, evasion, block, energy shield)

**Bonus Range**: +5% to +30% (varies by card level)

**Examples:**
- "Guardian's Shield Card" - +20% Armor
- "Wind Dancer Card" - +25% Evasion
- "Fortress Card" - +15% Block Chance
- "Arcane Barrier Card" - +30% Energy Shield

**Rarity Distribution:**
- Common: +5% to +10% defense
- Uncommon: +10% to +15% defense
- Rare: +15% to +20% defense
- Very Rare: +20% to +25% defense
- Boss Cards: +25% to +30% defense

### Resistance Cards

**Effect**: Increase specific resistance types

**Bonus Range**: +5% to +25% resistance

**Examples:**
- "Fire Ward Card" - +15% Fire Resistance
- "Ice Barrier Card" - +20% Cold Resistance
- "Poison Immunity Card" - +25% Poison Resistance

**Rarity Distribution:**
- Common: +5% to +10% resistance
- Uncommon: +10% to +15% resistance
- Rare: +15% to +20% resistance
- Very Rare: +20% to +25% resistance
- Boss Cards: +25% resistance

### Immunity Cards

**Effect**: Grant immunity to specific resistance types (extremely rare)

**Immunity Types**: Can grant immunity to Fire, Cold, Poison, Energy, Light, Dark, or Physical damage

**Rarity**: Very rare, typically boss drops only

**Examples:**
- "Phoenix Card" - Immunity to Fire Damage
- "Frost King Card" - Immunity to Cold Damage
- "Shadow Lord Card" - Immunity to Dark Damage
- "Titan's Hide Card" - Immunity to Physical Damage

**Note**: Immunity cards are the rarest and most powerful cards. They provide complete immunity to a damage type, making them extremely valuable.

### Utility Cards

**Effect**: Provide utility effects (movement speed, gathering bonuses, luck, etc.)

**Bonus Range**: Varies by effect type

**Examples:**
- "Swift Wind Card" - +15% Movement Speed
- "Lucky Miner Card" - +20% Mineral Gathering
- "Treasure Hunter Card" - +25% Gold Find
- "Experience Boost Card" - +10% Experience Gain

**Rarity Distribution:**
- Common: +5% to +10% utility
- Uncommon: +10% to +15% utility
- Rare: +15% to +20% utility
- Very Rare: +20% to +25% utility
- Boss Cards: +25% to +30% utility

### Special Cards

**Effect**: Unique effects not covered by other categories

**Examples:**
- Experience bonus cards
- Gold find bonus cards
- Skill cooldown reduction cards
- Critical chance increase cards
- Health regeneration cards
- Mana regeneration cards

**Rarity**: Varies, some are boss-exclusive

## Card Restrictions

### Equipment Restrictions

**Single Card Limit:**
- Cannot use 2 cards on the same equipment (even in different slots)
- Each equipment can have maximum 1 card

**Single Rune Limit:**
- Cannot use 2 runes on the same equipment (see RUNES.md)
- Each equipment can have maximum 1 rune

**Mixed Usage:**
- Can use 1 card + 1 rune on same equipment (different slots)
- Example: Armor with 6 slots can have 1 card in slot 1 and 1 rune in slot 2

**Slot Usage:**
- Each slot can contain EITHER one rune OR one card, never both
- Cannot have both rune and card in same slot

### Example Valid Configurations

**Valid:**
- Armor with 6 slots: 1 card + 1 rune ✓ (1 card max, 1 rune max, rest empty)
- Armor with 6 slots: 1 card + 0 runes ✓ (1 card max)
- Armor with 6 slots: 0 cards + 1 rune ✓ (1 rune max)
- Two-Handed Weapon with 3 slots: 1 card + 1 rune ✓ (max 1 card, max 1 rune)
- Shield with 2 slots: 1 card + 1 rune ✓ (max 1 card, max 1 rune)

**Invalid:**
- Armor with 6 slots: 6 cards ✗ (cannot use 2 cards on same equipment)
- Armor with 6 slots: 6 runes ✗ (cannot use 2 runes on same equipment)
- Armor with 6 slots: 2 cards + 1 rune ✗ (cannot use 2 cards)
- Armor with 6 slots: 1 card + 2 runes ✗ (cannot use 2 runes)
- Armor with 6 slots: 1 card + 1 rune in same slot ✗ (cannot have both in same slot)

## Card Insertion and Removal

### Inserting Cards

**Requirements:**
- Item must have an empty slot
- Player must have the card in inventory
- Equipment must not already have a card inserted (cannot use 2 cards on same equipment)
- No additional materials required

**Process:**
1. Player selects item with empty slot
2. Player selects card from inventory
3. System checks if equipment already has a card (if yes, insertion fails)
4. System checks if equipment already has a rune (if yes, can still insert card in different slot)
5. Card is inserted into slot
6. Card effects are immediately applied
7. Card remains in collection (not consumed, can be removed and reused)

**Card Restrictions:**
- Cannot insert 2 cards on the same equipment (even in different slots)
- Can insert 1 card + 1 rune on same equipment (different slots)
- Cards are preserved (remain in collection for Steam/card game/wagering)

### Removing Cards

**Requirements:**
- Item must have a card in slot
- No materials required for removal
- Card is returned to inventory (not destroyed)

**Process:**
1. Player selects item with carded slot
2. Player selects "Remove Card" option
3. System removes card from slot
4. Card effects are removed
5. Slot becomes empty (can be reused)
6. Card is returned to inventory (preserved for collection)

**Note:** Cards are preserved on removal (unlike runes), allowing players to reuse cards and maintain their collection for Steam rewards, card game, and wagering.

## Collectible Function

### Steam Integration

**Steam Collectible System:**
- Cards are linked to Steam collectible system
- Collecting cards grants Steam rewards
- Rewards scale with card rarity and collection completion
- Collection achievements unlock Steam badges and rewards

**Collection Rewards:**
- **Common Cards**: Steam trading cards, profile backgrounds
- **Uncommon Cards**: Steam badges, emoticons
- **Rare Cards**: Steam profile showcases, special badges
- **Very Rare Cards**: Steam profile showcases, rare badges
- **Boss Cards**: Exclusive Steam rewards, rare profile items
- **Complete Sets**: Special Steam rewards for completing card sets

**Collection Tracking:**
- Player's card collection is tracked in-game
- Collection progress synced with Steam
- Collection achievements visible in Steam profile
- Trading cards between players affects Steam collection

**Steam Badge System:**
- Collecting cards contributes to Steam badge progress
- Different badges for different card types
- Boss card collections unlock special badges
- Complete collections unlock rare badges

## Card Game Integration

### Internal Card Game

**Card Game Overview:**
- Cards can be used in the MMO's internal card game
- Card game is a separate minigame within the MMO
- Players can build decks using collected cards
- Card game matches can be played against other players or NPCs

**Card Game Mechanics:**
- Cards have stats for the card game (separate from equipment stats)
- Card rarity affects card game power
- Boss cards are particularly powerful in card game
- Card game rewards include gold, items, and experience

**Deck Building:**
- Players can build decks from their collected cards
- Deck size and composition rules apply
- Rare cards allow for more powerful deck strategies
- Card game meta evolves with new card releases

**Card Game Modes:**
- **PvP Matches**: Play against other players
- **NPC Challenges**: Play against NPC opponents
- **Tournaments**: Competitive card game tournaments
- **Ranked Matches**: Ranked ladder system

**Card Game Rewards:**
- Gold rewards for winning matches
- Item rewards for tournament victories
- Experience rewards for card game participation
- Special rewards for achieving high ranks

## Wagering System

### Card Wagering

**Wagering Overview:**
- Cards can be wagered in various game activities
- Wagering allows players to bet cards for rewards
- Wagered cards are at risk of being lost
- Winning wagers return cards plus rewards

**Wagering Activities:**
- **PvP Matches**: Bet cards on match outcomes
- **Boss Fights**: Bet cards on boss kill success
- **Card Game Matches**: Bet cards on card game outcomes
- **Tournaments**: Bet cards on tournament winners
- **Arena Battles**: Bet cards on arena match results

**Wagering Mechanics:**
- Players can set wager amounts (number of cards)
- Higher wagers offer better rewards
- Losing wagers result in card loss
- Winning wagers return cards plus bonus rewards
- Wagering is optional (players choose to participate)

**Wagering Rewards:**
- **Winning**: Return wagered cards + bonus rewards (gold, items, experience)
- **Losing**: Lose wagered cards
- **Reward Scaling**: Higher wagers = better rewards
- **Risk/Reward**: Higher risk (more cards) = higher potential rewards

**Wagering Examples:**
- Wager 1 common card on PvP match: Win = 1 card + 100 gold, Lose = lose card
- Wager 3 rare cards on boss fight: Win = 3 cards + 500 gold + item, Lose = lose 3 cards
- Wager 1 boss card on tournament: Win = 1 card + 1000 gold + rare item, Lose = lose card

## Card Rarity and Value

### Card Rarity Levels

**Common Cards:**
- Most frequently dropped (still <1%)
- Basic effects and bonuses
- Lower value in card game
- Common Steam rewards

**Uncommon Cards:**
- Less frequent drops
- Moderate effects and bonuses
- Moderate value in card game
- Uncommon Steam rewards

**Rare Cards:**
- Very infrequent drops
- Strong effects and bonuses
- High value in card game
- Rare Steam rewards

**Very Rare Cards:**
- Extremely rare drops
- Very strong effects and bonuses
- Very high value in card game
- Very rare Steam rewards

**Boss Cards:**
- Boss-exclusive, rarest cards
- Most powerful effects (including immunities)
- Maximum value in card game
- Exclusive Steam rewards

### Card Value Factors

**Value Determinants:**
- **Rarity**: Rarer cards are more valuable
- **Effects**: More powerful effects increase value
- **Immunity Cards**: Highest value due to immunity effects
- **Boss Cards**: High value due to rarity and power
- **Collection Value**: Cards needed for collections are more valuable
- **Card Game Value**: Cards useful in card game meta are more valuable
- **Steam Value**: Cards that unlock rare Steam rewards are more valuable

**Value Examples:**
- Common Attribute Card: 500-1,000 gold
- Uncommon Damage Card: 1,000-2,500 gold
- Rare Defense Card: 2,500-5,000 gold
- Very Rare Immunity Card: 10,000-25,000 gold
- Boss Card: 25,000-100,000+ gold

### Card Trading

**Trading System:**
- Cards can be traded between players
- Trading affects Steam collection
- Card market prices fluctuate based on demand
- Rare cards command premium prices

**Trading Mechanics:**
- Direct player-to-player trading
- Auction house listings
- Card market with price history
- Steam collection updates on trade

## Card Visual Design

### Card Appearance

**Card Artwork:**
- Each card has unique artwork
- Artwork quality scales with card rarity
- Boss cards have special artwork
- Cards display their effects and stats visually

**Card Design Elements:**
- Card frame (varies by rarity)
- Artwork (unique per card)
- Effect text and icons
- Rarity indicator
- Card level/stat display

**Visual Quality:**
- Common cards: Basic artwork
- Uncommon cards: Enhanced artwork
- Rare cards: High-quality artwork
- Very Rare cards: Premium artwork
- Boss cards: Exclusive artwork

### Card Collection UI

**Collection Interface:**
- Collection interface shows all collected cards
- Cards can be viewed, sorted, and organized
- Collection progress tracked visually
- Steam integration visible in collection UI

**Collection Features:**
- Card gallery view
- Collection statistics
- Missing cards indicator
- Collection completion percentage
- Steam badge progress

## Rust Implementation Considerations

### Data Structures

**Card Component:**
```rust
#[derive(Component, Clone, Copy)]
pub struct Card {
    pub card_id: u32,
    pub card_type: CardType,
    pub rarity: CardRarity,
    pub level: u8,
    pub effects: CardEffects,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum CardType {
    Attribute,
    Damage,
    Defense,
    Resistance,
    Immunity,
    Utility,
    Special,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum CardRarity {
    Common,
    Uncommon,
    Rare,
    VeryRare,
    Boss,
}

#[derive(Clone, Copy)]
pub struct CardEffects {
    pub attribute_bonus: Option<AttributeBonus>,
    pub damage_bonus: Option<DamageBonus>,
    pub defense_bonus: Option<DefenseBonus>,
    pub resistance_bonus: Option<ResistanceBonus>,
    pub immunity: Option<ImmunityType>,
    pub utility_bonus: Option<UtilityBonus>,
    pub special_effect: Option<SpecialEffect>,
}

#[derive(Clone, Copy)]
pub enum ImmunityType {
    Fire,
    Cold,
    Poison,
    Energy,
    Light,
    Dark,
    Physical,
}
```

### Card Management

**Card Functions:**
```rust
impl ItemSlots {
    pub fn has_card(&self) -> bool {
        self.slots.iter().any(|slot| {
            matches!(slot, Some(SlotContent::Card(_)))
        })
    }
    
    pub fn insert_card(&mut self, slot_index: usize, card: Card) -> Result<(), String> {
        if slot_index >= self.slots.len() {
            return Err("Invalid slot index".to_string());
        }
        
        if self.slots[slot_index].is_some() {
            return Err("Slot already occupied".to_string());
        }
        
        if self.has_card() {
            return Err("Cannot use 2 cards on same equipment".to_string());
        }
        
        self.slots[slot_index] = Some(SlotContent::Card(card));
        self.used_slots += 1;
        Ok(())
    }
    
    pub fn remove_card(&mut self, slot_index: usize) -> Result<Card, String> {
        if slot_index >= self.slots.len() {
            return Err("Invalid slot index".to_string());
        }
        
        match &self.slots[slot_index] {
            Some(SlotContent::Card(card)) => {
                let card_copy = *card;
                self.slots[slot_index] = None;
                self.used_slots -= 1;
                Ok(card_copy) // Return card (not destroyed)
            },
            Some(SlotContent::Rune(_)) => {
                Err("Slot contains rune, not card".to_string())
            },
            None => Err("Slot is empty".to_string()),
        }
    }
}
```

## Integration with Other Systems

### Slot System

**Integration:**
- Cards use the same slot system as runes
- Cannot use 2 cards or 2 runes on same equipment
- Can use 1 card + 1 rune on same equipment
- See [RUNES.md](./RUNES.md) for slot system details

### Equipment System

**Integration:**
- Cards modify equipment attributes and effects
- Card effects are applied when card is inserted
- Card effects are removed when card is removed
- Cards can be used on enchanted items

### Steam System

**Integration:**
- Cards sync with Steam collectible system
- Collection progress tracked in Steam
- Trading affects Steam collection
- Steam rewards unlock based on collection

### Card Game System

**Integration:**
- Cards have separate stats for card game
- Card game uses collected cards
- Card game rewards include gold and items
- Card game meta affects card value

### Wagering System

**Integration:**
- Cards can be wagered in various activities
- Wagering creates risk/reward mechanics
- Winning wagers return cards plus rewards
- Losing wagers result in card loss

## Summary

The cards system:

- Provides extremely rare collectible items with powerful effects
- Less than 1% drop chance from all creatures
- Boss cards are even rarer (0.1-0.3% drop chance)
- Can provide immunity to damage types (rarest cards)
- Single card limit per equipment (cannot use 2 cards)
- Cards are preserved on removal (unlike runes)
- Steam integration for collectible rewards
- Internal card game integration
- Wagering system for risk/reward mechanics
- Integration with slot, equipment, Steam, and card game systems

This system creates depth in equipment customization and collectible gameplay, allowing players to acquire powerful cards while maintaining balance through rarity and restrictions. Cards serve multiple purposes: equipment enhancement, collectible completion, card game participation, and wagering, creating a multi-faceted system that rewards collection and strategic use.

