# Player Spells System Documentation

## Overview

The Player Spells system allows players to learn and use magical abilities through a Codex system with three spell trees. Players acquire spells by using scrolls dropped from various sources, with different rarities determining where they can be obtained. The system limits player builds through the Codex tree selection, requiring strategic choices in spell selection and tree specialization.

## Key Features

- **Scroll-Based Learning**: Spells learned through consumable scroll items
- **Three Rarity Tiers**: Common (green), Rare (blue), and Ancient (orange) scrolls
- **Codex System**: Three spell trees similar to ArcheAge system
- **Build Limitation**: Codex selection limits available spells
- **Spell Trees**: Each tree contains 7 common, 4 rare, and 1 ancient spell
- **Reset System**: NPC mage allows resetting points or changing trees (gold cost)
- **Spell Persistence**: Server saves which spells player has unlocked
- **NPC Vendors**: Class-specific NPCs sell some spells for gold
- **Auction House**: Spells can be bought/sold on auction house
- **One-Time Use**: Scrolls are consumable items (single use)

## Scroll System

### Scroll Rarity Tiers

**Common Scrolls (Green):**
- Drop from any magical creature
- Most accessible spell scrolls
- Used for common spells in Codex trees
- Can be found in regular gameplay
- Lower drop rate but common source

**Rare Scrolls (Blue):**
- Drop only from boss creatures
- More powerful spells
- Used for rare spells in Codex trees
- Requires boss farming
- Moderate drop rate from bosses

**Ancient Scrolls (Orange):**
- Drop only from raid encounters
- Most powerful spells
- Used for ancient spells in Codex trees
- Requires raid participation
- Very low drop rate from raids

### Scroll Properties

**Scroll Item Type:**
- Consumable item (single use)
- One-time use per scroll
- Consumed when used to learn spell
- Cannot be reused after learning

**Scroll Identification:**
- Scroll name indicates spell name
- Scroll rarity indicates spell tier
- Scroll description shows spell details
- Scroll icon shows rarity color

**Scroll Usage:**
- Player uses scroll from inventory
- System checks if player can learn spell
- Spell unlocked if conditions met
- Scroll consumed after successful learning
- Player gains access to spell

## Codex System

### Codex Overview

**Three Spell Trees:**
- Players choose three Codex trees
- Each tree represents a spell specialization
- Trees limit available spells
- Similar to ArcheAge skill tree system
- Strategic build limitation

**Tree Selection:**
- Players select three trees at character creation or later
- Trees can be changed via NPC mage (gold cost)
- Selection affects available spells
- Cannot use spells from unselected trees
- Build diversity through tree combinations

### Spell Tree Structure

**Tree Composition:**
- Each tree contains 12 spells total:
  - 7 Common spells (green scrolls)
  - 4 Rare spells (blue scrolls)
  - 1 Ancient spell (orange scroll)
- Total of 36 spells across all trees
- Players can learn all spells in selected trees
- Cannot learn spells from unselected trees

**Spell Progression:**
- Spells can be learned in any order within tree
- No prerequisite requirements
- All spells in selected trees are available
- Learning limited by scroll availability
- No level requirements for learning

### Codex Trees

**Tree Types:**
- Three distinct spell trees available
- Each tree has unique theme and playstyle
- Trees complement different builds
- Detailed tree descriptions in future documentation
- Trees provide build diversity

**Tree Examples (to be detailed):**
- Tree 1: [To be specified]
- Tree 2: [To be specified]
- Tree 3: [To be specified]

**Tree Limitations:**
- Players limited to spells from selected trees
- Cannot mix spells from unselected trees
- Build defined by tree selection
- Strategic choice required
- Respec available via NPC mage

## Spell Learning

### Learning Process

**Scroll Usage:**
1. Player obtains scroll (drop, purchase, trade)
2. Player uses scroll from inventory
3. System validates learning conditions:
   - Player has selected tree containing spell
   - Player hasn't already learned spell
   - Scroll is valid for spell
4. Spell unlocked if conditions met
5. Scroll consumed
6. Spell added to player's spellbook
7. Player can now use spell

**Learning Conditions:**
- Must have selected Codex tree containing spell
- Cannot learn spell from unselected tree
- Cannot relearn already known spell
- Scroll must match spell rarity and name
- Player must have inventory space

**Learning Failure:**
- If conditions not met, scroll not consumed
- Error message displayed to player
- Player can try again after meeting conditions
- Scroll remains in inventory

### Spell Unlocking

**Unlock Status:**
- Server tracks which spells player has unlocked
- Unlock status persisted in database
- Unlocked spells available across sessions
- Cannot lose unlocked spells
- Permanent unlock (unless reset)

**Spell Availability:**
- Unlocked spells available in spellbook
- Can be assigned to hotkeys
- Can be used in combat
- Available regardless of equipment
- No cooldown on learning

## Spell Reset System

### Reset Mechanics

**NPC Mage:**
- Located in major cities
- Provides spell reset services
- Charges gold for services
- Two reset options available

**Reset Options:**
- **Reset Points**: Reset all learned spells (keep trees)
- **Change Trees**: Change Codex tree selection (resets spells)

**Reset Process:**
1. Player interacts with NPC mage
2. Player selects reset option
3. System displays gold cost
4. Player confirms payment
5. Reset executed:
   - All learned spells removed
   - Scrolls not refunded
   - Trees reset (if changing trees)
   - Player must relearn spells

**Reset Cost:**
- Gold cost configured per reset type
- Cost may increase with level
- Cost may increase with number of resets
- Cost displayed before confirmation
- Payment required before reset

### Reset Limitations

**Reset Restrictions:**
- Cannot reset individual spells
- Must reset all spells or change trees
- Scrolls consumed are not refunded
- Must relearn spells after reset
- Cooldown may apply (if configured)

**Reset Consequences:**
- All learned spells removed
- Must use scrolls again to relearn
- Build completely reset
- Can choose different tree combination
- Strategic decision required

## Spell Acquisition

### Drop Sources

**Common Scrolls:**
- Drop from any magical creature
- Regular monsters in world
- Dungeon creatures
- Event creatures
- Moderate drop rate

**Rare Scrolls:**
- Drop only from boss creatures
- World bosses
- Dungeon bosses
- Event bosses
- Lower drop rate

**Ancient Scrolls:**
- Drop only from raid encounters
- Raid bosses
- High-tier raid content
- Very low drop rate
- Requires raid participation

### NPC Vendors

**Class-Specific NPCs:**
- NPCs in cities sell spells
- Each class has dedicated NPC
- NPCs sell spells for gold
- Limited spell selection
- Convenient acquisition method

**Vendor Spells:**
- Some spells available from vendors
- Not all spells sold by vendors
- Gold cost for vendor spells
- Alternative to scroll drops
- Guaranteed availability

**Vendor Locations:**
- Major cities have spell vendors
- Class-specific vendors
- Easy to find and access
- Always available
- Reliable source

### Auction House

**Spell Trading:**
- Scrolls can be listed on auction house
- Players can buy/sell scrolls
- Market-driven prices
- Alternative acquisition method
- Trading between players

**Auction House Features:**
- Buyout and auction listings
- Search by spell name
- Filter by rarity
- Price comparison
- Secure trading

**Trading Benefits:**
- Access to spells without farming
- Sell extra scrolls for gold
- Market price discovery
- Player-driven economy
- Convenient acquisition

## Spell Usage

### Spell Availability

**Spellbook:**
- All learned spells in spellbook
- Organized by Codex tree
- Can be browsed and reviewed
- Spell details displayed
- Easy access to spells

**Hotkey Assignment:**
- Spells can be assigned to hotkeys
- Quick access in combat
- Customizable key bindings
- Multiple spell bars
- Easy spell switching

**Combat Usage:**
- Spells usable in combat
- Cooldowns and costs apply
- Mana/stamina requirements
- Cast times and animations
- Visual and audio effects

### Spell Restrictions

**Tree Restrictions:**
- Can only use spells from selected trees
- Cannot use spells from unselected trees
- Build limitation enforced
- Strategic choice required
- Respec needed to change

**Learning Restrictions:**
- Must learn spell before using
- Cannot use unlearned spells
- Scroll required for learning
- One-time learning per spell
- Permanent unlock

## Server Persistence

### Spell Data Storage

**Unlocked Spells:**
- Server saves which spells player has unlocked
- Stored in player database record
- Persisted across sessions
- Cannot be lost
- Permanent unlock

**Codex Tree Selection:**
- Server saves selected Codex trees
- Stored in player database record
- Persisted across sessions
- Changed only via NPC mage
- Build definition

**Spell Progress:**
- Learning status tracked per spell
- Unlock timestamps (optional)
- Learning source tracked (optional)
- Historical data maintained
- Analytics support

### Data Structure

**Player Spell Data:**
- List of unlocked spell IDs
- Selected Codex tree IDs (3 trees)
- Spell learning history (optional)
- Reset history (optional)
- Last reset timestamp (optional)

**Spell Metadata:**
- Spell ID and name
- Codex tree association
- Rarity tier
- Learning requirements
- Usage restrictions

## Integration Points

### With Item System
- Scrolls are consumable items
- Scrolls use consumable item mechanics
- Scrolls can be traded
- Scrolls can be sold
- Scrolls stored in inventory

### With Inventory System
- Scrolls stored in player inventory
- Scrolls take inventory space
- Scrolls can be moved/dropped
- Scrolls can be organized
- Inventory management required

### With Auction House
- Scrolls can be listed for sale
- Scrolls can be purchased
- Market-driven pricing
- Trading between players
- Economic integration

### With NPC System
- NPC mages provide reset services
- Class NPCs sell spells
- NPC interactions for services
- Gold transactions
- Service availability

### With Combat System
- Spells used in combat
- Spell effects and damage
- Cooldowns and costs
- Cast times and animations
- Combat integration

## Related Documentation

- [PLAYER.md](./PLAYER.md) - Core player system
- [SKILLS.md](./SKILLS.md) - Skills system
- [CONSUMABLES.md](../items/CONSUMABLES.md) - Consumable items system
- [AUCTION_HOUSE.md](../server/AUCTION_HOUSE.md) - Auction house system
- [SPELLS.md](../server/SPELLS.md) - Server-side spells system

## See Also

- [Player README](./README.md) - Player systems overview
- [Items README](../items/README.md) - Items systems overview
- [Server README](../server/README.md) - Server documentation

