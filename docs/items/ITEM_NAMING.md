# Item Naming System Documentation

## Overview

The item naming system generates equipment names automatically based on the item's base name, rarity, and most important attributes. The system follows Path of Exile 2's naming convention, where item names are composed of base name + prefixes/suffixes based on rarity, creating a clear visual hierarchy and information system for players.

**Note:** Item names are generated automatically and reflect the item's properties, making it easy for players to identify item value and attributes at a glance.

## Key Features

- **Automatic Generation**: Item names are generated automatically based on attributes
- **Rarity-Based Naming**: Different naming patterns for each rarity tier
- **Three-Layer System**: Implicit, Explicit, and Enchantment mods displayed separately
- **PoE2-Inspired**: Follows Path of Exile 2 naming conventions
- **Attribute-Based**: Names reflect the most important attributes on the item
- **Visual Hierarchy**: Name display varies by rarity for quick identification

## Naming Structure

### Name Composition

**Item Name Formula:**
```
Item Name = Base Name + Prefix(es) + Suffix(es) [based on rarity]
```

**Three Layers of Mods:**
1. **Implicit (Implicit)**: Built into the base item, displayed above the line
2. **Explicit (Explicit)**: Prefixes and suffixes that can be modified
3. **Enchantment**: Enchantment bonuses (separate from prefix/suffix)

## Rarity-Based Naming

### Normal (White) Items

**Naming Pattern:**
- **Name**: Base name only
- **Example**: "Iron Ring", "Short Sword", "Leather Hood"
- **Prefixes/Suffixes**: None (no explicit mods shown in name)
- **Mod Count**: 0 explicit mods

**Display:**
```
Iron Ring
[Base stats only]
```

**Characteristics:**
- Simplest naming
- No attribute bonuses in name
- Only base item properties
- Most common item type

### Magic (Blue) Items

**Naming Pattern:**
- **Name**: Prefix and/or Suffix + Base Name
- **Mod Count**: 0-1 prefix, 0-1 suffix (maximum 2 explicit mods)
- **Name Format**: 
  - Prefix only: `[Prefix] [Base Name]`
  - Suffix only: `[Base Name] of [Suffix]`
  - Both: `[Prefix] [Base Name] of [Suffix]`

**Examples:**
- Prefix only: "Fiery Iron Ring"
- Suffix only: "Iron Ring of the Fox"
- Both: "Fiery Iron Ring of the Fox"

**Display:**
```
Fiery Iron Ring of the Fox
[Enchantment mods - if any]
────────────────────────────
[Implicit mods]
+10% Fire Resistance (Implicit)
────────────────────────────
[Explicit mods]
+25 Fire Damage (Prefix: Fiery)
+15 Dexterity (Suffix: of the Fox)
```

**Naming Rules:**
- Prefix appears BEFORE base name
- Suffix appears AFTER base name with "of" connector
- Pattern is fixed: `[Prefix] [Base] of [Suffix]`
- Names come from the actual mod names

**Prefix Examples:**
- "Fiery" (Fire Damage)
- "Frost" (Cold Damage)
- "Mighty" (Strength)
- "Swift" (Dexterity)
- "Arcane" (Intelligence)
- "Sturdy" (Armor)
- "Evasive" (Evasion)

**Suffix Examples:**
- "of the Fox" (Dexterity)
- "of the Bear" (Strength)
- "of the Owl" (Intelligence)
- "of the Tiger" (Damage)
- "of the Turtle" (Defense)
- "of the Phoenix" (Fire Resistance)
- "of the Frost" (Cold Resistance)

### Rare (Yellow/Gold) Items

**Naming Pattern:**
- **Name**: Title + Base Name (pseudo-random titles)
- **Mod Count**: 0-3 prefixes, 0-3 suffixes (maximum 6 explicit mods)
- **Name Format**: `[Title Part 1] [Title Part 2] [Base Name]`

**Examples:**
- "Doom Veil Leather Hood"
- "Gale Knuckle Iron Ring"
- "Storm Crown Steel Sword"
- "Shadow Claw Leather Boots"

**Display:**
```
Doom Veil Leather Hood
[Enchantment mods - if any]
────────────────────────────
[Implicit mods]
+10% Fire Resistance (Implicit)
────────────────────────────
[Explicit mods]
+35 Fire Damage (Prefix)
+28 Cold Damage (Prefix)
+22 Strength (Prefix)
+18 Dexterity (Suffix)
+15% Armor (Suffix)
+12% Evasion (Suffix)
```

**Naming Rules:**
- Name does NOT use actual mod names
- Uses pseudo-random titles from separate tables
- Titles are internally linked to the affixes the item has
- Two-part title system (Title Part 1 + Title Part 2)

**Title Part 1 Examples:**
- "Doom" (high damage mods)
- "Gale" (speed/evasion mods)
- "Storm" (elemental damage mods)
- "Shadow" (dark/stealth mods)
- "Frost" (cold mods)
- "Flame" (fire mods)
- "Thunder" (energy mods)
- "Void" (dark mods)
- "Divine" (light mods)
- "Primal" (physical mods)

**Title Part 2 Examples:**
- "Veil" (defensive mods)
- "Knuckle" (offensive mods)
- "Crown" (intelligence/head mods)
- "Claw" (dexterity/hand mods)
- "Fang" (damage mods)
- "Shield" (defense mods)
- "Blade" (weapon mods)
- "Guard" (armor mods)
- "Stride" (boots/movement mods)
- "Grasp" (gloves mods)

**Title Generation Logic:**
- Title Part 1: Based on primary damage/attribute type
- Title Part 2: Based on equipment type and mod focus
- Titles are selected from tables based on item's affixes
- Multiple affixes influence title selection

### Unique (Orange) Items

**Naming Pattern:**
- **Name**: Completely custom name
- **Mod Count**: Fixed mods (defined per unique item)
- **Name Format**: Custom (no pattern)

**Examples:**
- "Kaom's Heart"
- "Tabula Rasa"
- "Shadow's Embrace"
- "Dragon's Claw"

**Display:**
```
Kaom's Heart
[Unique item description]
────────────────────────────
[Fixed mods - cannot be changed]
+100 Strength
+50% Fire Resistance
+200 Maximum HP
[Unique effect description]
```

**Naming Rules:**
- Completely custom names
- No prefix/suffix system
- Names are defined per unique item
- Cannot be modified or crafted

## Three-Layer Mod System

### Layer 1: Implicit (Implicit)

**Description:**
- Mods that are "built into" the base item
- Cannot be changed through normal crafting
- Always present on the base item type

**Display:**
- Shown above the separator line in tooltip
- Typically displayed first
- Labeled as "(Implicit)"

**Examples:**
- Topaz Ring: Always has Lightning Resistance (implicit)
- Ruby Ring: Always has Fire Resistance (implicit)
- Sapphire Ring: Always has Cold Resistance (implicit)
- Leather Boots: Always has Movement Speed (implicit)

**Characteristics:**
- Do not change when rolling affixes
- Part of the base item definition
- Cannot be removed or modified
- Always present

### Layer 2: Explicit (Explicit)

**Description:**
- Prefixes and suffixes that can be modified
- These are the mods that crafting affects
- Count varies by rarity

**Display:**
- Shown below the separator line in tooltip
- Listed as explicit mods
- Can be modified through crafting

**Magic Items:**
- 0-1 prefix, 0-1 suffix
- Maximum 2 explicit mods
- Mod names appear in item name

**Rare Items:**
- 0-3 prefixes, 0-3 suffixes
- Maximum 6 explicit mods
- Mod names do NOT appear in item name (uses titles instead)

**Examples:**
- Magic: "Fiery Iron Ring" → +Fire Damage (explicit prefix)
- Rare: "Doom Veil Leather Hood" → Multiple mods (explicit), but name uses title

### Layer 3: Enchantment

**Description:**
- Bonuses from enchantments
- Separate from prefix/suffix system
- Have their own "slots" (do not count as prefix/suffix)

**Display:**
- Shown at the top of tooltip (above implicit)
- Displayed in light blue color
- Labeled as enchantment bonuses

**Examples:**
- "+3 Fire Enchantment" (shown above implicit line)
- Enchantment bonuses are separate from explicit mods
- Do not affect item naming (except visual display)

**Characteristics:**
- Fixed mods (from enchantment system)
- Do not follow normal prefix/suffix tables
- Cannot be rolled like explicit mods
- Separate from crafting system

## Name Generation Algorithm

### Step 1: Determine Rarity

**Process:**
1. Check item rarity (Normal, Magic, Rare, Unique)
2. If Unique: Use custom name (skip to end)
3. If Normal: Use base name only (skip affix naming)
4. If Magic or Rare: Continue to affix naming

### Step 2: Identify Most Important Attributes

**Attribute Priority:**
1. **Damage Attributes**: Highest damage bonus
2. **Resistance Attributes**: Highest resistance bonus
3. **Core Stats**: Highest stat bonus (STR, DEX, INT)
4. **Defense Attributes**: Highest defense bonus
5. **Utility Attributes**: Other bonuses

**Selection Logic:**
- Sort attributes by value (highest first)
- For Magic: Select top 1-2 attributes
- For Rare: Consider all attributes for title generation

### Step 3: Generate Name Based on Rarity

**Magic Items:**
```
1. Identify top attribute (prefix candidate)
2. Identify second attribute (suffix candidate, if exists)
3. Map attribute to prefix name
4. Map attribute to suffix name
5. Combine: [Prefix] [Base] of [Suffix]
```

**Rare Items:**
```
1. Analyze all attributes
2. Determine primary attribute type (damage type, stat type, etc.)
3. Select Title Part 1 based on primary attribute
4. Select Title Part 2 based on equipment type and mod focus
5. Combine: [Title Part 1] [Title Part 2] [Base]
```

### Step 4: Apply Name

**Final Name:**
- Apply generated name to item
- Store name in item data
- Display in tooltip and inventory

## Attribute to Name Mapping

### Prefix Mapping (Magic Items)

**Damage Attributes:**
- Fire Damage → "Fiery"
- Cold Damage → "Frost"
- Poison Damage → "Venomous"
- Energy Damage → "Thunder"
- Light Damage → "Radiant"
- Dark Damage → "Shadow"
- Physical Damage → "Mighty"

**Stat Attributes:**
- Strength → "Mighty"
- Dexterity → "Swift"
- Intelligence → "Arcane"
- Vigor → "Sturdy"
- Agility → "Nimble"
- Luck → "Lucky"

**Defense Attributes:**
- Armor → "Sturdy"
- Evasion → "Evasive"
- Block → "Shielded"
- Energy Shield → "Arcane"

**Resistance Attributes:**
- Fire Resistance → "Fiery"
- Cold Resistance → "Frost"
- Poison Resistance → "Venomous"
- Energy Resistance → "Thunder"
- Light Resistance → "Radiant"
- Dark Resistance → "Shadow"
- Physical Resistance → "Sturdy"

### Suffix Mapping (Magic Items)

**Stat Suffixes:**
- Strength → "of the Bear"
- Dexterity → "of the Fox"
- Intelligence → "of the Owl"
- Vigor → "of the Turtle"
- Agility → "of the Hare"
- Luck → "of Fortune"

**Damage Suffixes:**
- High Damage → "of the Tiger"
- Fire Damage → "of the Phoenix"
- Cold Damage → "of the Frost"
- Poison Damage → "of the Viper"
- Energy Damage → "of the Storm"
- Light Damage → "of the Light"
- Dark Damage → "of the Shadow"

**Defense Suffixes:**
- High Defense → "of the Turtle"
- Armor → "of the Guard"
- Evasion → "of the Wind"
- Block → "of the Shield"
- Energy Shield → "of the Barrier"

**Resistance Suffixes:**
- Fire Resistance → "of the Phoenix"
- Cold Resistance → "of the Frost"
- Poison Resistance → "of the Viper"
- Energy Resistance → "of the Storm"
- Light Resistance → "of the Light"
- Dark Resistance → "of the Shadow"
- Physical Resistance → "of the Guard"

### Title Part 1 Mapping (Rare Items)

**Based on Primary Damage Type:**
- Fire → "Flame", "Fiery", "Burning"
- Cold → "Frost", "Ice", "Frozen"
- Poison → "Venom", "Toxic", "Poison"
- Energy → "Thunder", "Storm", "Lightning"
- Light → "Divine", "Radiant", "Holy"
- Dark → "Shadow", "Void", "Dark"
- Physical → "Primal", "Mighty", "Brutal"

**Based on Primary Stat:**
- Strength → "Mighty", "Brutal", "Powerful"
- Dexterity → "Swift", "Nimble", "Agile"
- Intelligence → "Arcane", "Wise", "Mystical"
- Mixed → "Balanced", "Harmonious"

**Based on Defense Focus:**
- High Defense → "Sturdy", "Fortified", "Guardian"
- High Evasion → "Evasive", "Elusive", "Shadow"
- High Energy Shield → "Arcane", "Mystical", "Barrier"

### Title Part 2 Mapping (Rare Items)

**Based on Equipment Type:**
- Helmet → "Crown", "Veil", "Helm"
- Chest → "Plate", "Mail", "Vest"
- Legs → "Leggings", "Greaves", "Pants"
- Boots → "Stride", "Tread", "Step"
- Gloves → "Grasp", "Claw", "Grip"
- Ring → "Band", "Circle", "Ring"
- Necklace → "Chain", "Pendant", "Amulet"
- Weapon → "Blade", "Edge", "Strike"
- Shield → "Shield", "Guard", "Barrier"

**Based on Mod Focus:**
- Offensive → "Fang", "Claw", "Blade", "Strike"
- Defensive → "Shield", "Guard", "Barrier", "Ward"
- Balanced → "Veil", "Crown", "Band"

## Rust Implementation Considerations

### Data Structures

**Item Name Component:**
```rust
#[derive(Component, Clone)]
pub struct ItemName {
    pub base_name: String,
    pub prefix: Option<String>,
    pub suffix: Option<String>,
    pub title_part1: Option<String>,
    pub title_part2: Option<String>,
    pub custom_name: Option<String>, // For unique items
    pub full_name: String,
    pub rarity: ItemRarity,
}

#[derive(Clone, Copy, PartialEq, Eq)]
pub enum ItemRarity {
    Normal,
    Magic,
    Rare,
    Legendary,
    Unique,
}
```

### Name Generation Function

**Name Generation:**
```rust
impl ItemName {
    pub fn generate_name(
        base_name: &str,
        rarity: ItemRarity,
        attributes: &[ItemAttribute],
        is_unique: bool,
        custom_name: Option<&str>,
    ) -> Self {
        if is_unique {
            return ItemName {
                base_name: base_name.to_string(),
                prefix: None,
                suffix: None,
                title_part1: None,
                title_part2: None,
                custom_name: custom_name.map(|s| s.to_string()),
                full_name: custom_name.unwrap_or(base_name).to_string(),
                rarity,
            };
        }
        
        match rarity {
            ItemRarity::Normal => {
                ItemName {
                    base_name: base_name.to_string(),
                    prefix: None,
                    suffix: None,
                    title_part1: None,
                    title_part2: None,
                    custom_name: None,
                    full_name: base_name.to_string(),
                    rarity,
                }
            },
            ItemRarity::Magic => {
                let (prefix, suffix) = generate_magic_affixes(attributes);
                let full_name = format_magic_name(base_name, &prefix, &suffix);
                
                ItemName {
                    base_name: base_name.to_string(),
                    prefix,
                    suffix,
                    title_part1: None,
                    title_part2: None,
                    custom_name: None,
                    full_name,
                    rarity,
                }
            },
            ItemRarity::Rare | ItemRarity::Legendary => {
                let (title1, title2) = generate_rare_title(attributes, base_name);
                let full_name = format!("{} {} {}", title1, title2, base_name);
                
                ItemName {
                    base_name: base_name.to_string(),
                    prefix: None,
                    suffix: None,
                    title_part1: Some(title1),
                    title_part2: Some(title2),
                    custom_name: None,
                    full_name,
                    rarity,
                }
            },
            ItemRarity::Unique => {
                // Should not reach here (handled above)
                ItemName::generate_name(base_name, rarity, attributes, true, custom_name)
            },
        }
    }
    
    fn format_magic_name(base: &str, prefix: &Option<String>, suffix: &Option<String>) -> String {
        match (prefix, suffix) {
            (Some(p), Some(s)) => format!("{} {} of {}", p, base, s),
            (Some(p), None) => format!("{} {}", p, base),
            (None, Some(s)) => format!("{} of {}", base, s),
            (None, None) => base.to_string(),
        }
    }
}
```

### Attribute Analysis

**Attribute Analysis Function:**
```rust
fn generate_magic_affixes(attributes: &[ItemAttribute]) -> (Option<String>, Option<String>) {
    if attributes.is_empty() {
        return (None, None);
    }
    
    // Sort attributes by value (highest first)
    let mut sorted = attributes.to_vec();
    sorted.sort_by(|a, b| b.value.partial_cmp(&a.value).unwrap());
    
    let prefix = sorted.first().and_then(|attr| map_to_prefix(attr));
    let suffix = sorted.get(1).and_then(|attr| map_to_suffix(attr));
    
    (prefix, suffix)
}

fn generate_rare_title(attributes: &[ItemAttribute], base_name: &str) -> (String, String) {
    // Analyze all attributes to determine primary focus
    let primary_type = determine_primary_attribute_type(attributes);
    let equipment_type = determine_equipment_type_from_name(base_name);
    
    let title1 = map_to_title_part1(&primary_type);
    let title2 = map_to_title_part2(&equipment_type, &primary_type);
    
    (title1, title2)
}

fn map_to_prefix(attribute: &ItemAttribute) -> Option<String> {
    match attribute.attribute_type {
        AttributeType::FireDamage => Some("Fiery".to_string()),
        AttributeType::ColdDamage => Some("Frost".to_string()),
        AttributeType::BonusStr => Some("Mighty".to_string()),
        AttributeType::BonusDex => Some("Swift".to_string()),
        AttributeType::BonusInt => Some("Arcane".to_string()),
        // ... more mappings
        _ => None,
    }
}

fn map_to_suffix(attribute: &ItemAttribute) -> Option<String> {
    match attribute.attribute_type {
        AttributeType::BonusStr => Some("of the Bear".to_string()),
        AttributeType::BonusDex => Some("of the Fox".to_string()),
        AttributeType::BonusInt => Some("of the Owl".to_string()),
        AttributeType::FireDamage => Some("of the Phoenix".to_string()),
        // ... more mappings
        _ => None,
    }
}
```

## Display Format

### Tooltip Display

**Magic Item Tooltip:**
```
Fiery Iron Ring of the Fox
[+3 Fire Enchantment] ← Enchantment (light blue)
────────────────────────────
+10% Fire Resistance (Implicit) ← Implicit
────────────────────────────
+25 Fire Damage (Explicit) ← Explicit Prefix
+15 Dexterity (Explicit) ← Explicit Suffix
```

**Rare Item Tooltip:**
```
Doom Veil Leather Hood
[+4 Cold Enchantment] ← Enchantment (light blue)
────────────────────────────
+10% Fire Resistance (Implicit) ← Implicit
────────────────────────────
+35 Fire Damage (Explicit) ← Explicit Prefix 1
+28 Cold Damage (Explicit) ← Explicit Prefix 2
+22 Strength (Explicit) ← Explicit Prefix 3
+18 Dexterity (Explicit) ← Explicit Suffix 1
+15% Armor (Explicit) ← Explicit Suffix 2
+12% Evasion (Explicit) ← Explicit Suffix 3
```

**Unique Item Tooltip:**
```
Kaom's Heart
[Unique Item]
────────────────────────────
+100 Strength (Fixed)
+50% Fire Resistance (Fixed)
+200 Maximum HP (Fixed)
────────────────────────────
[Unique Effect Description]
```

## Integration with Other Systems

### Attribute System

**Integration:**
- Name generation uses attributes from ATTRIBUTES.md
- Most important attributes determine name
- Attribute values influence name selection
- See [ATTRIBUTES.md](./ATTRIBUTES.md) for attribute details

### Enchantment System

**Integration:**
- Enchantments appear above implicit line
- Enchantments do not affect item naming
- Enchantment bonuses displayed separately
- See [ENCHANTMENTS.md](./ENCHANTMENTS.md) for enchantment details

### Rarity System

**Integration:**
- Rarity determines naming pattern
- Rarity affects mod count
- Rarity affects name display
- See [ITEMS.md](./ITEMS.md) for rarity system

### Crafting System

**Integration:**
- Crafting modifies explicit mods
- Name updates when mods change
- Magic items: Name reflects new mods
- Rare items: Title may change based on new mods

## Summary

The item naming system:

- Generates names automatically based on base name and attributes
- Follows PoE2 naming conventions (Normal, Magic, Rare, Unique)
- Uses three-layer mod system (Implicit, Explicit, Enchantment)
- Magic items show mod names in item name
- Rare items use pseudo-random titles linked to affixes
- Unique items have completely custom names
- Visual hierarchy makes item properties clear at a glance
- Integrates with attribute, enchantment, and rarity systems

This system creates clear visual identification of items while maintaining the depth and complexity of Path of Exile 2's naming system, making it easy for players to understand item value and properties through naming alone.

