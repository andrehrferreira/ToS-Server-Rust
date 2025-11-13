# Equipment System Documentation

## Overview

The equipment system manages all wearable items that can be equipped by humanoids (players, NPCs, humanoid creatures). Equipment items have visual representation on the client, alter the humanoid's appearance, and drop from creatures (except for players, which have special drop rules). Each equipment type occupies specific inventory slots and provides attribute bonuses when equipped.

**Note:** Equipment items are separate from consumables, resources, and other item types. They provide permanent stat bonuses and visual customization for characters.

## Key Features

- **9 Equipment Types**: Armor, Boots, Gloves, Helmet, Mainhand, Offhand, Belt, Ring (2 slots), Necklace
- **Resource Types**: All equipment has a resource type (Metal/Mineral, Leather, Cloth) that affects attributes and stat requirements
- **Build Segmentation**: Resource types segment builds without hard class restrictions, enabling hybrid builds
- **Visual Representation**: All equipment has graphical representation on client
- **Appearance Alteration**: Equipment changes humanoid appearance when equipped
- **Attribute Bonuses**: Equipment provides stat bonuses when equipped
- **Drop System**: Equipment drops from creatures (players have special drop rules)
- **Inventory Slots**: Each equipment type has specific inventory slot
- **Visual Skins**: Cosmetic equipment that only changes appearance (no stats)
- **Dye System**: Visual skins can be colored with dye tubs
- **Tools**: Optional equipment that provides gathering bonuses

## Resource Types

### Overview

Equipment items are crafted from different resource types based on equipment category:

**Armor and Weapons:**
- **Metal/Mineral**: Heavy, durable equipment
- **Leather**: Balanced, mobile equipment
- **Cloth**: Light, magical equipment

**Accessories (Rings and Necklaces):**
- **Bone**: Organic, carved accessories
- **Metal**: Metallic, polished accessories

The resource type determines:
- Which attributes the equipment can have
- Which stat requirements are needed to equip it
- The build focus and playstyle it supports

**Key Concept:**
Resource types create natural build segmentation without hard class restrictions. Players can mix different resource types to create hybrid builds that differ from traditional class archetypes.

### Metal/Mineral Equipment

**Description:**
Equipment crafted from metals and minerals (iron, steel, mithril, etc.). Heavy and durable, focused on physical defense and strength.

**Primary Stat Requirements:**
- **STR** (Strength): Primary requirement
- **CON** (Constitution): Secondary requirement

**Attribute Focus:**
- **Physical Defense**: High physical resistance, armor bonuses
- **HP Bonuses**: Significant HP increases
- **STR Bonuses**: Strength attribute bonuses
- **CON Bonuses**: Constitution attribute bonuses
- **Weight Capacity**: Increased weight capacity
- **Physical Damage**: Physical damage bonuses (weapons)

**Build Focus:**
- Warriors, Tanks
- Physical DPS builds
- High HP builds
- Heavy armor users

**Attribute Restrictions:**
- Cannot have high DEX bonuses
- Cannot have high INT bonuses
- Limited magic damage bonuses
- Limited evasion bonuses

**Example Equipment:**
- Plate Armor (Metal)
- Steel Boots (Metal)
- Iron Gauntlets (Metal)
- Metal Helmet (Metal)
- Steel Sword (Metal)
- Metal Shield (Metal)

### Leather Equipment

**Description:**
Equipment crafted from leather and hides. Balanced between defense and mobility, focused on dexterity and agility.

**Primary Stat Requirements:**
- **DEX** (Dexterity): Primary requirement
- **STR** (Strength): Secondary requirement

**Attribute Focus:**
- **Evasion**: High evasion bonuses
- **DEX Bonuses**: Dexterity attribute bonuses
- **Movement Speed**: Movement speed bonuses
- **Stamina**: Stamina bonuses
- **Attack Speed**: Attack speed bonuses
- **Ranged Damage**: Ranged damage bonuses (weapons)

**Build Focus:**
- Rangers, Rogues
- Evasion builds
- Mobility-focused builds
- Ranged combat builds

**Attribute Restrictions:**
- Cannot have very high physical resistance
- Cannot have very high HP bonuses
- Limited magic bonuses
- Limited INT bonuses

**Example Equipment:**
- Leather Armor (Leather)
- Leather Boots (Leather)
- Leather Gloves (Leather)
- Leather Cap (Leather)
- Leather Bow (Leather)
- Leather Quiver (Leather)

### Cloth Equipment

**Description:**
Equipment crafted from cloth and fabric. Light and flexible, focused on magic and intelligence.

**Primary Stat Requirements:**
- **INT** (Intelligence): Primary requirement
- **DEX** (Dexterity): Secondary requirement (light weight)

**Attribute Focus:**
- **Magic Damage**: High magic damage bonuses
- **Mana**: Significant mana bonuses
- **INT Bonuses**: Intelligence attribute bonuses
- **Elemental Resistances**: Elemental resistance bonuses
- **Spell Damage**: Spell damage bonuses
- **Mana Regeneration**: Mana regeneration bonuses

**Build Focus:**
- Mages, Wizards
- Magic DPS builds
- Support builds
- Spellcaster builds

**Attribute Restrictions:**
- Cannot have high physical resistance
- Cannot have high HP bonuses
- Limited STR bonuses
- Limited CON bonuses

**Example Equipment:**
- Robe (Cloth)
- Cloth Boots (Cloth)
- Cloth Gloves (Cloth)
- Cloth Hood (Cloth)
- Staff (Cloth/Magic)
- Spellbook (Cloth)

### Bone Equipment (Accessories Only)

**Description:**
Accessories crafted from bones and organic materials. Used exclusively for rings and necklaces. Provides flexible attribute bonuses.

**Primary Stat Requirements:**
- Flexible requirements (can require DEX, INT, STR, CON, or LCK)
- Usually minimal stat requirements
- Lower requirements than armor/weapons

**Attribute Focus:**
- **LCK Bonuses**: High luck bonuses
- **CHA Bonuses**: Charisma bonuses
- **Flexible Bonuses**: Can provide any attribute bonuses
- **DEX/INT Bonuses**: Common for bone accessories
- **Utility Bonuses**: Various utility bonuses

**Build Focus:**
- All builds (flexible)
- Luck-focused builds
- Utility builds
- Hybrid builds

**Attribute Restrictions:**
- No restrictions (can have any attribute bonuses)
- Generally lower bonuses than armor/weapons
- Focus on utility and secondary stats

**Example Equipment:**
- Bone Ring (Bone)
- Bone Necklace (Bone)
- Carved Bone Amulet (Bone)
- Organic Ring (Bone)

## Equipment Types

### Armor

**Slot**: Chest/Armor slot

**Description:**
Armor is the primary body protection equipment, covering the torso and providing significant defensive bonuses.

**Visual Representation:**
- Covers torso, chest, and upper body
- Visible on character model
- Can be light, medium, or heavy armor types
- Different visual styles per armor type

**Attribute Bonuses:**
- Primary: Physical Resistance, HP
- Secondary: STR, CON bonuses
- Can provide elemental resistances
- May have weight/durability modifiers

**Equipment Restrictions:**
- Cannot equip multiple armors
- May have stat requirements (STR, CON)
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules (see Player Drop System)

### Boots

**Slot**: Feet/Boots slot

**Description:**
Boots protect the feet and lower legs, providing mobility and defensive bonuses.

**Visual Representation:**
- Covers feet and lower legs
- Visible on character model
- Different styles (boots, shoes, sandals)
- Can be light or heavy

**Attribute Bonuses:**
- **Metal**: Movement Speed, Stamina, STR, CON bonuses
- **Leather**: Movement Speed, Stamina, DEX, Evasion bonuses
- **Cloth**: Movement Speed, Mana, INT bonuses

**Stat Requirements:**
- **Metal**: Moderate STR, CON
- **Leather**: High DEX, moderate STR
- **Cloth**: Moderate INT, DEX

**Equipment Restrictions:**
- Cannot equip multiple boots
- Stat requirements vary by resource type
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Gloves

**Slot**: Hands/Gloves slot

**Description:**
Gloves protect the hands and provide bonuses to manual dexterity and crafting.

**Visual Representation:**
- Covers hands and wrists
- Visible on character model
- Different styles (gloves, gauntlets, mittens)
- Can be light or heavy

**Attribute Bonuses:**
- **Metal**: Attack Speed, STR, CON, Crafting bonuses
- **Leather**: Attack Speed, DEX, LCK, Gathering bonuses
- **Cloth**: Attack Speed, INT, Crafting bonuses, Skill bonuses

**Stat Requirements:**
- **Metal**: Moderate STR, CON
- **Leather**: High DEX, moderate STR
- **Cloth**: Moderate INT, DEX

**Equipment Restrictions:**
- Cannot equip multiple gloves
- Stat requirements vary by resource type
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Helmet

**Slot**: Head/Helmet slot

**Description:**
Helmets protect the head and provide defensive bonuses and sometimes vision bonuses.

**Visual Representation:**
- Covers head
- Visible on character model
- Different styles (helmet, cap, hood, crown)
- Can hide/show hair/face
- Can be light or heavy

**Attribute Bonuses:**
- **Metal**: Physical Resistance, HP, STR, CON bonuses
- **Leather**: Evasion, DEX, Stamina bonuses
- **Cloth**: Mana, INT, Magic Damage, Elemental Resistances bonuses

**Stat Requirements:**
- **Metal**: High STR, moderate CON
- **Leather**: High DEX, moderate STR
- **Cloth**: High INT, moderate DEX

**Equipment Restrictions:**
- Cannot equip multiple helmets
- Stat requirements vary by resource type
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Mainhand

**Slot**: Main hand/Weapon slot

**Description:**
Mainhand equipment is the primary weapon or tool held in the character's main hand.

**Visual Representation:**
- Held in right hand (or main hand)
- Visible on character model
- Different weapon types (sword, axe, mace, bow, staff, etc.)
- Can be one-handed or two-handed

**Attribute Bonuses:**
- **Metal**: Physical Damage, STR, CON bonuses
- **Leather**: Ranged Damage, DEX, Evasion bonuses
- **Cloth**: Magic Damage, INT, Mana bonuses, Elemental Damage

**Stat Requirements:**
- **Metal**: High STR, moderate CON
- **Leather**: High DEX, moderate STR
- **Cloth**: High INT, moderate DEX

**Equipment Restrictions:**
- Cannot equip multiple mainhand items
- Stat requirements vary by resource type
- May have level requirements
- Two-handed weapons prevent offhand equipment

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Offhand

**Slot**: Off hand/Shield slot

**Description:**
Offhand equipment is held in the character's off hand, typically shields or secondary weapons.

**Visual Representation:**
- Held in left hand (or off hand)
- Visible on character model
- Different types (shield, offhand weapon, tome, etc.)
- Can be small or large

**Attribute Bonuses:**
- **Metal**: Block Chance, Physical Resistance, STR, CON bonuses
- **Leather**: Evasion, DEX, Movement Speed bonuses
- **Cloth**: Mana, INT, Magic Barrier, Elemental Resistances bonuses

**Stat Requirements:**
- **Metal**: High STR, moderate CON
- **Leather**: High DEX, moderate STR
- **Cloth**: High INT, moderate DEX

**Equipment Restrictions:**
- Cannot equip multiple offhand items
- Cannot equip if using two-handed mainhand
- Stat requirements vary by resource type
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Belt

**Slot**: Waist/Belt slot

**Description:**
Belts are worn around the waist and provide utility bonuses and weight capacity.

**Visual Representation:**
- Worn around waist
- Visible on character model
- Different styles (belt, sash, chain)
- Can be simple or ornate

**Attribute Bonuses:**
- **Metal**: Weight Capacity, HP, STR, CON bonuses
- **Leather**: Weight Capacity, DEX, Stamina bonuses
- **Cloth**: Weight Capacity, Mana, INT bonuses

**Stat Requirements:**
- **Metal**: Moderate STR, CON
- **Leather**: Moderate DEX, STR
- **Cloth**: Moderate INT, DEX

**Equipment Restrictions:**
- Cannot equip multiple belts
- Stat requirements vary by resource type
- May have level requirements

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Ring (Slot 1 and Slot 2)

**Slot**: Ring slots (2 separate slots)

**Description:**
Rings are worn on fingers and provide various attribute bonuses. Players can equip two rings simultaneously. Rings are always crafted from bones or metals.

**Visual Representation:**
- Worn on fingers (may not be visible on model)
- Can be visible as hand accessories
- Different styles (ring, band, signet)
- Can be simple or ornate
- Bone rings: Organic, carved appearance
- Metal rings: Metallic, polished appearance

**Resource Types:**
- **Bone**: Crafted from bones and organic materials
- **Metal**: Crafted from metals and minerals

**Attribute Bonuses:**
- **Bone**: LCK, CHA, DEX, INT bonuses, can provide any attribute bonuses
- **Metal**: STR, CON, HP, Physical Resistance bonuses, can provide any attribute bonuses
- Both types can provide LCK, CHA bonuses
- Both types can provide various utility bonuses

**Stat Requirements:**
- **Bone**: Usually minimal stat requirements (low DEX, INT, or LCK)
- **Metal**: Usually minimal stat requirements (low STR, CON)
- Generally lower requirements than armor/weapons

**Equipment Restrictions:**
- Can equip up to 2 rings (Ring Slot 1 and Ring Slot 2)
- Cannot equip same ring in both slots
- May have level requirements
- Stat requirements vary by resource type (Bone or Metal)

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

### Necklace

**Slot**: Neck/Necklace slot

**Description:**
Necklaces are worn around the neck and provide various attribute bonuses, often focused on magic or utility. Necklaces are always crafted from bones or metals.

**Visual Representation:**
- Worn around neck
- Visible on character model
- Different styles (necklace, pendant, amulet, chain)
- Can be simple or ornate
- Bone necklaces: Organic, carved appearance, often with teeth/bones
- Metal necklaces: Metallic, polished appearance, often with gems

**Resource Types:**
- **Bone**: Crafted from bones and organic materials
- **Metal**: Crafted from metals and minerals

**Attribute Bonuses:**
- **Bone**: LCK, CHA, DEX, INT, Mana bonuses, can provide any attribute bonuses
- **Metal**: STR, CON, HP, Physical Resistance bonuses, can provide any attribute bonuses
- Both types can provide LCK, CHA bonuses
- Both types can provide various utility bonuses

**Stat Requirements:**
- **Bone**: Usually minimal stat requirements (low DEX, INT, or LCK)
- **Metal**: Usually minimal stat requirements (low STR, CON)
- Generally lower requirements than armor/weapons

**Equipment Restrictions:**
- Cannot equip multiple necklaces
- May have level requirements
- Stat requirements vary by resource type (Bone or Metal)

**Drop Rules:**
- Drops from creatures
- Players have special drop rules

## Visual Skins (Cosmetic Equipment)

### Overview

Visual skins are cosmetic equipment items that alter the character's appearance without providing any attribute bonuses. They are purely visual and can be dyed with dye tubs.

**Key Features:**
- **No Attributes**: Visual skins provide zero stat bonuses
- **Appearance Only**: Only change visual appearance
- **Dye System**: Can be colored with dye tubs
- **Overlay System**: Visual skins overlay on top of regular equipment
- **Optional**: Players can choose to use visual skins or not

### Visual Skin Types

**Armor Skins:**
- Cosmetic armor appearances
- Overlay on regular armor
- Can be dyed
- No stat bonuses

**Boot Skins:**
- Cosmetic boot appearances
- Overlay on regular boots
- Can be dyed
- No stat bonuses

**Glove Skins:**
- Cosmetic glove appearances
- Overlay on regular gloves
- Can be dyed
- No stat bonuses

**Helmet Skins:**
- Cosmetic helmet appearances
- Overlay on regular helmet
- Can be dyed
- No stat bonuses

**Weapon Skins:**
- Cosmetic weapon appearances
- Overlay on regular weapons
- Can be dyed
- No stat bonuses

**Shield Skins:**
- Cosmetic shield appearances
- Overlay on regular shields
- Can be dyed
- No stat bonuses

### Dye System

**Dye Tubs:**
- Items used to change color of visual skins
- Can be applied to any visual skin
- Multiple colors available
- Dye is consumed on use

**Dye Application:**
1. Player selects visual skin
2. Player selects dye tub
3. System applies color to visual skin
4. Dye tub is consumed
5. Visual skin color is updated

**Dye Colors:**
- Primary colors (Red, Blue, Yellow, Green, etc.)
- Secondary colors (Orange, Purple, Pink, etc.)
- Metallic colors (Gold, Silver, Bronze, etc.)
- Special colors (Rainbow, Glow, etc.)

**Dye Restrictions:**
- Can only dye visual skins
- Cannot dye regular equipment
- Some visual skins may have color restrictions
- Dye effect is permanent until re-dyed

## Tools

### Overview

Tools are optional equipment items that provide gathering bonuses. They do not occupy regular equipment slots and can be equipped alongside regular equipment.

**Key Features:**
- **Optional**: Tools are not required for gathering
- **Gathering Bonuses**: Provide bonuses to gathering activities
- **No Visual**: Tools may not have visual representation (or minimal)
- **Stackable Effects**: Can stack with other gathering bonuses
- **Skill Bonuses**: Can provide skill bonuses for gathering

### Tool Types

**Mining Tools:**
- Pickaxes, drills, etc.
- Increase mineral gathering yield
- Reduce mining time
- Reduce mining skill requirements
- Increase mining skill effectiveness

**Woodcutting Tools:**
- Axes, saws, etc.
- Increase wood gathering yield
- Reduce woodcutting time
- Reduce woodcutting skill requirements
- Increase woodcutting skill effectiveness

**Skinning Tools:**
- Knives, skinning blades, etc.
- Increase skin gathering yield
- Reduce skinning time
- Reduce skinning skill requirements
- Increase skinning skill effectiveness

**Fishing Tools:**
- Fishing rods, nets, etc.
- Increase fish gathering yield
- Reduce fishing time
- Reduce fishing skill requirements
- Increase fishing skill effectiveness

**Herbalism Tools:**
- Scythes, harvesting tools, etc.
- Increase herb gathering yield
- Reduce harvesting time
- Reduce herbalism skill requirements
- Increase herbalism skill effectiveness

### Tool Bonuses

**Yield Bonuses:**
- Increase amount of resources gathered
- Percentage-based (e.g., +10% yield)
- Stacks with other yield bonuses
- Applies to all gathering activities

**Time Reduction:**
- Reduce time required for gathering
- Percentage-based (e.g., -15% gathering time)
- Stacks with other time reductions
- Minimum time cap (cannot reduce below minimum)

**Skill Requirement Reduction:**
- Reduce skill level required for gathering
- Flat reduction (e.g., -5 skill requirement)
- Allows gathering higher-tier resources earlier
- Stacks with other skill reductions

**Skill Effectiveness:**
- Increase effectiveness of gathering skills
- Percentage-based (e.g., +20% skill effectiveness)
- Stacks with other skill bonuses
- Applies to all gathering skills

## Equipment Slots

### Slot System

**Inventory Slots:**
- Each equipment type has a dedicated inventory slot
- Equipment can only be placed in its designated slot
- Slots are visually distinct in inventory UI
- Cannot place wrong equipment type in slot

**Slot Layout:**
```
[Helmet]
[Necklace]
[Armor]
[Belt]
[Gloves]        [Ring 1]    [Ring 2]
[Boots]
[Mainhand]      [Offhand]
```

**Slot Restrictions:**
- Each slot can hold only one item
- Ring slots: Can hold 2 rings (separate slots)
- Two-handed weapons: Block offhand slot
- Visual skins: Overlay slots (separate from equipment slots)

## Build Segmentation Through Resource Types

### Traditional Builds

**Pure Metal Build:**
- All equipment: Metal/Mineral
- High STR and CON requirements
- Focus: Physical defense, HP, physical damage
- Playstyle: Tank, Warrior, Physical DPS

**Pure Leather Build:**
- All equipment: Leather
- High DEX requirements
- Focus: Evasion, mobility, ranged damage
- Playstyle: Ranger, Rogue, Evasion tank

**Pure Cloth Build:**
- All equipment: Cloth
- High INT requirements
- Focus: Magic damage, mana, elemental damage
- Playstyle: Mage, Wizard, Spellcaster

### Hybrid Builds

**Metal + Leather Hybrid:**
- Mix of Metal and Leather equipment
- Requirements: Moderate STR, CON, DEX
- Focus: Balanced physical defense and mobility
- Playstyle: Versatile warrior, mobile tank

**Leather + Cloth Hybrid:**
- Mix of Leather and Cloth equipment
- Requirements: Moderate DEX, INT
- Focus: Mobile mage, ranged spellcaster
- Playstyle: Battle mage, mobile caster

**Metal + Cloth Hybrid:**
- Mix of Metal and Cloth equipment
- Requirements: Moderate STR, CON, INT
- Focus: Tanky mage, defensive spellcaster
- Playstyle: Battle mage, tank caster

**Balanced Hybrid:**
- Mix of all three resource types
- Requirements: Balanced stats
- Focus: Versatile, adaptable
- Playstyle: Jack of all trades, flexible builds

### Build Flexibility

**No Hard Restrictions:**
- Players can mix resource types freely
- No class system prevents mixing
- Stat requirements guide but don't restrict
- Build diversity through resource type combinations

**Natural Segmentation:**
- Resource types create natural build paths
- Stat requirements encourage specialization
- But allow flexibility for hybrid builds
- Enables unique build combinations

## Attribute Bonuses

### Bonus Calculation

**Total Attributes:**
```
Total Attribute = Base Attribute + Equipment Bonuses + Visual Skin Bonuses (0) + Tool Bonuses
```

**Equipment Bonuses:**
- Each equipped item provides attribute bonuses
- Bonuses are additive
- All equipment bonuses stack
- Visual skins provide 0 bonuses

**Example:**
```
Base STR: 50
Armor STR Bonus: +10
Boots STR Bonus: +2
Gloves STR Bonus: +3
Helmet STR Bonus: +5
Mainhand STR Bonus: +15
Belt STR Bonus: +1
Ring 1 STR Bonus: +4
Ring 2 STR Bonus: +3
Necklace STR Bonus: +2

Total STR: 50 + 10 + 2 + 3 + 5 + 15 + 1 + 4 + 3 + 2 = 95
```

### Bonus Types

**Core Attributes:**
- STR (Strength)
- DEX (Dexterity)
- INT (Intelligence)
- LCK (Luck)
- CON (Constitution)
- CHA (Charisma)

**Resource Bonuses:**
- HP (Health Points)
- Mana
- Stamina

**Defensive Bonuses:**
- Physical Resistance
- Elemental Resistances (Fire, Cold, Poison, Energy, Light, Dark)
- Evasion
- Block Chance
- Armor

**Offensive Bonuses:**
- Physical Damage
- Magic Damage
- Elemental Damage
- Critical Chance
- Critical Damage

**Utility Bonuses:**
- Movement Speed
- Attack Speed
- Weight Capacity
- Gathering Bonuses (from tools)

## Drop System

### Creature Drops

**Drop Rules:**
- Equipment drops from creatures
- Drop rate based on creature level and rarity
- Equipment level matches creature level
- Equipment rarity matches creature rarity

**Drop Mechanics:**
- Equipment appears in creature's loot table
- Drop chance varies by equipment type
- Higher level creatures drop better equipment
- Rare creatures drop rare equipment

### Player Drop Rules

**Special Rules:**
- Players have special drop rules (different from creatures)
- May have reduced drop rates
- May have different drop tables
- May have PvP-specific drop rules
- May have death penalty drops

**Death Penalties:**
- May drop equipped equipment on death
- May drop inventory equipment
- May have insurance system
- May have blessed items (no drop)

## Visual Representation

### Client Rendering

**Equipment Models:**
- Each equipment has 3D model
- Models are loaded from CDN
- Models are cached locally
- Models match equipment type and tier

**Appearance System:**
- Equipment appearance changes character model
- Visual skins overlay on equipment
- Dye colors affect visual appearance
- Appearance updates when equipment changes

**Animation:**
- Equipment affects character animations
- Weapons affect attack animations
- Armor affects movement animations
- Visual skins maintain base animations

## Rust Implementation Considerations

### Data Structures

**Equipment Component:**
```rust
#[derive(Component, Clone)]
pub struct Equipment {
    pub armor: Option<ItemId>,
    pub boots: Option<ItemId>,
    pub gloves: Option<ItemId>,
    pub helmet: Option<ItemId>,
    pub mainhand: Option<ItemId>,
    pub offhand: Option<ItemId>,
    pub belt: Option<ItemId>,
    pub ring1: Option<ItemId>,
    pub ring2: Option<ItemId>,
    pub necklace: Option<ItemId>,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum ResourceType {
    Metal,   // Metal/Mineral equipment (armor, weapons, accessories)
    Leather, // Leather equipment (armor, weapons)
    Cloth,   // Cloth equipment (armor, weapons)
    Bone,    // Bone equipment (accessories only: rings, necklaces)
}

#[derive(Component, Clone)]
pub struct VisualSkins {
    pub armor_skin: Option<ItemId>,
    pub boots_skin: Option<ItemId>,
    pub gloves_skin: Option<ItemId>,
    pub helmet_skin: Option<ItemId>,
    pub mainhand_skin: Option<ItemId>,
    pub offhand_skin: Option<ItemId>,
    pub dye_color: Option<DyeColor>,
}

#[derive(Component, Clone)]
pub struct Tools {
    pub mining_tool: Option<ItemId>,
    pub woodcutting_tool: Option<ItemId>,
    pub skinning_tool: Option<ItemId>,
    pub fishing_tool: Option<ItemId>,
    pub herbalism_tool: Option<ItemId>,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum EquipmentSlot {
    Armor,
    Boots,
    Gloves,
    Helmet,
    Mainhand,
    Offhand,
    Belt,
    Ring1,
    Ring2,
    Necklace,
}
```

### Equipment Management

**Equip Function:**
```rust
impl Equipment {
    pub fn can_equip(&self, item: &Item, player_stats: &PlayerStats) -> Result<(), String> {
        // Check resource type stat requirements
        match item.resource_type {
            ResourceType::Metal => {
                if player_stats.strength < item.required_str {
                    return Err(format!("Requires {} STR", item.required_str));
                }
                if player_stats.constitution < item.required_con {
                    return Err(format!("Requires {} CON", item.required_con));
                }
            },
            ResourceType::Leather => {
                if player_stats.dexterity < item.required_dex {
                    return Err(format!("Requires {} DEX", item.required_dex));
                }
                if player_stats.strength < item.required_str {
                    return Err(format!("Requires {} STR", item.required_str));
                }
            },
            ResourceType::Cloth => {
                if player_stats.intelligence < item.required_int {
                    return Err(format!("Requires {} INT", item.required_int));
                }
                if player_stats.dexterity < item.required_dex {
                    return Err(format!("Requires {} DEX", item.required_dex));
                }
            },
            ResourceType::Bone => {
                // Bone accessories have flexible requirements
                // Check whichever stat requirement is present
                if item.required_dex > 0 && player_stats.dexterity < item.required_dex {
                    return Err(format!("Requires {} DEX", item.required_dex));
                }
                if item.required_int > 0 && player_stats.intelligence < item.required_int {
                    return Err(format!("Requires {} INT", item.required_int));
                }
                if item.required_str > 0 && player_stats.strength < item.required_str {
                    return Err(format!("Requires {} STR", item.required_str));
                }
                if item.required_con > 0 && player_stats.constitution < item.required_con {
                    return Err(format!("Requires {} CON", item.required_con));
                }
            },
        }
        
        Ok(())
    }
    
    pub fn equip(&mut self, slot: EquipmentSlot, item_id: ItemId) -> Result<Option<ItemId>, String> {
        let current_item = match slot {
            EquipmentSlot::Armor => &mut self.armor,
            EquipmentSlot::Boots => &mut self.boots,
            EquipmentSlot::Gloves => &mut self.gloves,
            EquipmentSlot::Helmet => &mut self.helmet,
            EquipmentSlot::Mainhand => &mut self.mainhand,
            EquipmentSlot::Offhand => &mut self.offhand,
            EquipmentSlot::Belt => &mut self.belt,
            EquipmentSlot::Ring1 => &mut self.ring1,
            EquipmentSlot::Ring2 => &mut self.ring2,
            EquipmentSlot::Necklace => &mut self.necklace,
        };
        
        // Check if two-handed weapon and offhand slot
        if slot == EquipmentSlot::Offhand {
            if let Some(mainhand_id) = &self.mainhand {
                // Check if mainhand is two-handed
                if is_two_handed(mainhand_id) {
                    return Err("Cannot equip offhand with two-handed weapon".to_string());
                }
            }
        }
        
        // Check if mainhand is two-handed and trying to equip offhand
        if slot == EquipmentSlot::Mainhand {
            if is_two_handed(item_id) {
                if self.offhand.is_some() {
                    return Err("Cannot equip two-handed weapon with offhand".to_string());
                }
            }
        }
        
        let previous_item = current_item.replace(item_id);
        Ok(previous_item)
    }
    
    pub fn unequip(&mut self, slot: EquipmentSlot) -> Result<Option<ItemId>, String> {
        let current_item = match slot {
            EquipmentSlot::Armor => &mut self.armor,
            EquipmentSlot::Boots => &mut self.boots,
            EquipmentSlot::Gloves => &mut self.gloves,
            EquipmentSlot::Helmet => &mut self.helmet,
            EquipmentSlot::Mainhand => &mut self.mainhand,
            EquipmentSlot::Offhand => &mut self.offhand,
            EquipmentSlot::Belt => &mut self.belt,
            EquipmentSlot::Ring1 => &mut self.ring1,
            EquipmentSlot::Ring2 => &mut self.ring2,
            EquipmentSlot::Necklace => &mut self.necklace,
        };
        
        Ok(current_item.take())
    }
    
    pub fn calculate_total_bonuses(&self, items: &ItemDatabase) -> AttributeBonuses {
        let mut bonuses = AttributeBonuses::default();
        
        // Calculate bonuses from all equipped items
        if let Some(armor_id) = &self.armor {
            bonuses.add(items.get_bonuses(armor_id));
        }
        if let Some(boots_id) = &self.boots {
            bonuses.add(items.get_bonuses(boots_id));
        }
        if let Some(gloves_id) = &self.gloves {
            bonuses.add(items.get_bonuses(gloves_id));
        }
        if let Some(helmet_id) = &self.helmet {
            bonuses.add(items.get_bonuses(helmet_id));
        }
        if let Some(mainhand_id) = &self.mainhand {
            bonuses.add(items.get_bonuses(mainhand_id));
        }
        if let Some(offhand_id) = &self.offhand {
            bonuses.add(items.get_bonuses(offhand_id));
        }
        if let Some(belt_id) = &self.belt {
            bonuses.add(items.get_bonuses(belt_id));
        }
        if let Some(ring1_id) = &self.ring1 {
            bonuses.add(items.get_bonuses(ring1_id));
        }
        if let Some(ring2_id) = &self.ring2 {
            bonuses.add(items.get_bonuses(ring2_id));
        }
        if let Some(necklace_id) = &self.necklace {
            bonuses.add(items.get_bonuses(necklace_id));
        }
        
        bonuses
    }
}
```

## Integration with Other Systems

### Status System

**Integration:**
- Equipment bonuses are added to base attributes
- Total attributes = Base + Equipment + Other bonuses
- Equipment changes trigger attribute recalculation
- See [STATUS.md](../core/STATUS.md) for attribute calculations

### Item System

**Integration:**
- Equipment items follow item system rules
- Equipment has durability, rarity, tier
- Equipment can be enchanted, have runes/cards
- See [ITEMS.md](./ITEMS.md) for item system details

### Drop System

**Integration:**
- Equipment drops from creatures
- Players have special drop rules
- Equipment appears in loot tables
- Drop rates vary by equipment type and rarity

### Visual System

**Integration:**
- Equipment models loaded from CDN
- Visual skins overlay on equipment
- Dye system affects visual appearance
- Appearance updates on equipment change

### Gathering System

**Integration:**
- Tools provide gathering bonuses
- Tool bonuses stack with other bonuses
- Tools affect gathering yield, time, skill requirements
- See [GATHERING.md](../player/GATHERING.md) for gathering details

## Summary

The equipment system:

- Provides 9 equipment types (Armor, Boots, Gloves, Helmet, Mainhand, Offhand, Belt, Ring x2, Necklace)
- All equipment has visual representation and alters character appearance
- Equipment provides attribute bonuses when equipped
- Visual skins provide cosmetic appearance changes (no stats)
- Dye system allows coloring visual skins
- Tools provide optional gathering bonuses
- Equipment drops from creatures (players have special rules)
- Each equipment type has dedicated inventory slot
- Equipment bonuses stack additively
- Two-handed weapons prevent offhand equipment
- Ring slots allow 2 rings simultaneously

This system creates depth in character customization and progression, allowing players to equip items that enhance their stats while maintaining visual customization through skins and dyes.

