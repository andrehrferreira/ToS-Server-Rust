# Runes System Documentation

## Overview

The runes system allows players to enhance equipment by inserting magical runes into item slots. Runes are dropped from creatures and provide resistance bonuses. Equipment can drop with slots or have slots added using Magic Stones. Each equipment type has specific slot limits, and accessories cannot have slots.

**Note:** All resistance values are multiplied by 10 to provide a greater sense of progression, consistent with the damage and status systems.

## Key Features

- **Slot System**: Equipment can have slots for runes
- **Natural Slots**: Items can drop with slots already present
- **Magic Stone Slots**: Slots can be added using Magic Stones
- **Rune Drops**: Runes drop from creatures and provide resistance bonuses
- **Multiple Levels**: Runes have multiple levels (1-10) with increasing bonuses
- **Maximum Resistance**: Highest level runes provide up to 15% resistance
- **Equipment Limits**: Different equipment types have different slot limits
- **Single Rune Limit**: Cannot use 2 runes on the same equipment
- **Slot Restrictions**: Accessories cannot have slots

## Slot System

### Slot Limits by Equipment Type

**Two-Handed Weapons:**
- **Maximum Slots**: 3
- **Common Range**: 0-2 slots
- **Rare Range**: 2-3 slots

**Shields and Offhands:**
- **Maximum Slots**: 2
- **Common Range**: 0-1 slots
- **Rare Range**: 1-2 slots

**Armor (All Types):**
- **Maximum Slots**: 6
- **Common Range**: 0-3 slots
- **Rare Range**: 3-5 slots
- **Very Rare Range**: 4-6 slots

**One-Handed Weapons:**
- **Maximum Slots**: 1
- **Common Range**: 0-1 slots

**Other Equipment (Tools, etc.):**
- **Maximum Slots**: 1
- **Common Range**: 0-1 slots

**Accessories:**
- **Maximum Slots**: 0 (cannot have slots)
- **Note**: Rings, necklaces, and other accessories cannot have slots

### Slot Acquisition

**Natural Slots:**
- Items can drop with slots already present
- Slot count is determined by item rarity and tier
- Higher rarity items have higher chance of multiple slots
- Slot count is rolled when item is generated

**Magic Stone Slots:**
- Slots can be added using Magic Stones (see MAGIC_STONES.md)
- Maximum slot limit still applies (cannot exceed equipment type limit)
- Adding slots consumes Magic Stones
- Slots added via Magic Stones are permanent

**Slot Probability:**
- **Common Items**: 10% chance for 1 slot
- **Uncommon Items**: 25% chance for 1 slot, 5% chance for 2 slots
- **Rare Items**: 40% chance for 1 slot, 15% chance for 2 slots, 3% chance for 3 slots
- **Magic Items**: 60% chance for 1 slot, 30% chance for 2 slots, 10% chance for 3 slots
- **Legendary Items**: 80% chance for 1 slot, 50% chance for 2 slots, 25% chance for 3 slots, 10% chance for 4+ slots
- **Unique Items**: Fixed slot count (defined per item)

## Runes

### Rune Types

Runes provide resistance bonuses and are categorized by resistance type:

**Physical Resistance Rune:**
- **Effect**: Increases physical resistance
- **Drop Source**: Physical creatures, armored enemies
- **Visual**: Gray/silver runic symbol

**Fire Resistance Rune:**
- **Effect**: Increases fire resistance
- **Drop Source**: Fire elementals, volcanic creatures
- **Visual**: Red/orange runic symbol

**Cold Resistance Rune:**
- **Effect**: Increases cold resistance
- **Drop Source**: Ice elementals, frozen creatures
- **Visual**: Blue/white runic symbol

**Poison Resistance Rune:**
- **Effect**: Increases poison resistance
- **Drop Source**: Poisonous creatures, toxic enemies
- **Visual**: Green/purple runic symbol

**Energy Resistance Rune:**
- **Effect**: Increases energy resistance
- **Drop Source**: Lightning elementals, electric creatures
- **Visual**: Yellow/white runic symbol

**Light Resistance Rune:**
- **Effect**: Increases light resistance
- **Drop Source**: Holy creatures, light-based enemies
- **Visual**: White/golden runic symbol

**Dark Resistance Rune:**
- **Effect**: Increases dark resistance
- **Drop Source**: Shadow creatures, dark-based enemies
- **Visual**: Black/purple runic symbol

### Rune Levels

Runes have multiple levels (1-10) with increasing resistance bonuses:

**Level 1 Rune:**
- **Resistance Bonus**: +1% (multiplied by 10 = +10)
- **Drop Rate**: Common (60% of rune drops)
- **Value**: Low

**Level 2 Rune:**
- **Resistance Bonus**: +1.5% (multiplied by 10 = +15)
- **Drop Rate**: Common (25% of rune drops)
- **Value**: Low-Medium

**Level 3 Rune:**
- **Resistance Bonus**: +2% (multiplied by 10 = +20)
- **Drop Rate**: Uncommon (10% of rune drops)
- **Value**: Medium

**Level 4 Rune:**
- **Resistance Bonus**: +2.5% (multiplied by 10 = +25)
- **Drop Rate**: Uncommon (3% of rune drops)
- **Value**: Medium-High

**Level 5 Rune:**
- **Resistance Bonus**: +3% (multiplied by 10 = +30)
- **Drop Rate**: Rare (1% of rune drops)
- **Value**: High

**Level 6 Rune:**
- **Resistance Bonus**: +4% (multiplied by 10 = +40)
- **Drop Rate**: Rare (0.5% of rune drops)
- **Value**: High

**Level 7 Rune:**
- **Resistance Bonus**: +5% (multiplied by 10 = +50)
- **Drop Rate**: Rare (0.2% of rune drops)
- **Value**: Very High

**Level 8 Rune:**
- **Resistance Bonus**: +6.5% (multiplied by 10 = +65)
- **Drop Rate**: Very Rare (0.1% of rune drops)
- **Value**: Very High

**Level 9 Rune:**
- **Resistance Bonus**: +8.5% (multiplied by 10 = +85)
- **Drop Rate**: Very Rare (0.05% of rune drops)
- **Value**: Extremely High

**Level 10 Rune:**
- **Resistance Bonus**: +15% (multiplied by 10 = +150)
- **Drop Rate**: Extremely Rare (0.01% of rune drops)
- **Value**: Maximum

### Rune Level Progression

**Resistance Bonus Formula:**
```
Level 1:  +1.0%  (+10)
Level 2:  +1.5%  (+15)
Level 3:  +2.0%  (+20)
Level 4:  +2.5%  (+25)
Level 5:  +3.0%  (+30)
Level 6:  +4.0%  (+40)
Level 7:  +5.0%  (+50)
Level 8:  +6.5%  (+65)
Level 9:  +8.5%  (+85)
Level 10: +15.0% (+150)
```

**Progression Pattern:**
- Levels 1-5: Linear progression (+0.5% per level)
- Levels 6-7: Larger jumps (+1% per level)
- Levels 8-9: Exponential growth (+1.5% and +2% respectively)
- Level 10: Maximum bonus (+15%, significant jump)

### Rune Restrictions

**Equipment Restrictions:**
- **Single Rune Limit**: Cannot use 2 runes on the same equipment
- **Slot Usage**: Each slot can contain one rune
- **Mixed Usage**: Can use 1 rune + 1 card on same equipment (different slots, see CARDS.md)

**Example Valid Configurations:**
- Armor with 6 slots: 1 rune ✓ (1 rune max, rest empty)
- Armor with 6 slots: 1 rune + 1 card ✓ (max 1 rune, max 1 card)
- Armor with 6 slots: 6 runes ✗ (cannot use 2 runes on same equipment)
- Armor with 6 slots: 2 runes ✗ (cannot use 2 runes)

## Rune Insertion and Removal

### Inserting Runes

**Requirements:**
- Item must have an empty slot
- Player must have the rune in inventory
- Equipment must not already have a rune inserted (cannot use 2 runes on same equipment)
- No additional materials required (runes are self-contained)

**Process:**
1. Player selects item with empty slot
2. Player selects rune from inventory
3. System checks if equipment already has a rune (if yes, insertion fails)
4. System checks if rune type matches slot (if slot has type restriction)
5. Rune is inserted into slot
6. Resistance bonus is immediately applied
7. Rune is consumed (removed from inventory)

**Slot Restrictions:**
- Some items may have slot type restrictions (e.g., "Fire Rune Only")
- Most items accept any rune type
- Restrictions are item-specific and rare

### Removing Runes

**Requirements:**
- Item must have a rune in slot
- No materials required for removal
- Rune is destroyed on removal (cannot be recovered)

**Process:**
1. Player selects item with runed slot
2. Player selects "Remove Rune" option
3. System removes rune from slot
4. Resistance bonus is removed
5. Slot becomes empty (can be reused)
6. Rune is destroyed (not returned to inventory)

**Note:** Runes are consumed on insertion and destroyed on removal. This creates material sink and encourages farming for better runes.

### Upgrading Runes

**Rune Upgrade System:**
- Lower level runes can be combined to create higher level runes
- Requires multiple runes of same type and level
- Upgrade consumes runes and creates one higher level rune

**Upgrade Requirements:**
- **Level 1 → 2**: 3x Level 1 runes
- **Level 2 → 3**: 3x Level 2 runes
- **Level 3 → 4**: 4x Level 3 runes
- **Level 4 → 5**: 4x Level 4 runes
- **Level 5 → 6**: 5x Level 5 runes
- **Level 6 → 7**: 5x Level 6 runes
- **Level 7 → 8**: 6x Level 7 runes
- **Level 8 → 9**: 6x Level 8 runes
- **Level 9 → 10**: 8x Level 9 runes

**Upgrade Process:**
1. Player selects rune upgrade option
2. Player provides required runes
3. System consumes runes
4. System creates one higher level rune
5. Upgrade is instant (no failure chance)

## Resistance Calculation

### Total Resistance Formula

**Base Resistance:**
```
Base Resistance = Attribute-Based Resistance + Equipment Resistance
```

**Rune Resistance:**
```
Rune Resistance = Sum of all rune bonuses in equipped items
```

**Total Resistance:**
```
Total Resistance = Base Resistance + Rune Resistance
```

**Resistance Cap:**
- Maximum resistance cap: 70% (from RESISTANCES.md)
- Rune resistance counts toward this cap
- Example: Base 50% + Runes 25% = 75% → Capped at 70%

### Example Calculations

**Example 1: Single High-Level Rune**
- Base Fire Resistance: 40%
- Level 10 Fire Resistance Rune: +15%
- Total: 40% + 15% = 55% Fire Resistance

**Example 2: Multiple Equipment Pieces**
- Base Physical Resistance: 30%
- Level 5 Physical Rune in Armor: +3%
- Level 6 Physical Rune in Shield: +4%
- Total: 30% + 3% + 4% = 37% Physical Resistance

**Example 3: Full Armor Set**
- Armor with 1 slot, filled with Level 10 Fire Resistance Rune
- Base Fire Resistance: 20%
- Rune Bonus: 15%
- Total: 20% + 15% = 35% Fire Resistance

## Visual Effects

### Rune Visuals

**Slot Indicators:**
- Empty slots: Gray outline on item
- Filled slots: Glowing runic symbol matching rune type
- Multiple slots: Multiple symbols visible

**Rune Level Visuals:**
- Higher level runes have brighter/more intense glow
- Level 10 runes have special particle effects
- Visual intensity scales with rune level

**Resistance Type Colors:**
- Physical: Gray/Silver
- Fire: Red/Orange
- Cold: Blue/White
- Poison: Green/Purple
- Energy: Yellow/White
- Light: White/Golden
- Dark: Black/Purple

## Drop Rates and Economy

### Rune Drop Rates

**General Drop Rates:**
- **Common Creatures**: 5% chance to drop Level 1-2 runes
- **Uncommon Creatures**: 10% chance to drop Level 1-3 runes
- **Rare Creatures**: 15% chance to drop Level 1-5 runes
- **Elite Creatures**: 25% chance to drop Level 1-7 runes
- **Bosses**: 50% chance to drop Level 1-9 runes, 5% chance for Level 10

**Level Distribution:**
- Level 1: 60% of rune drops
- Level 2: 25% of rune drops
- Level 3: 10% of rune drops
- Level 4: 3% of rune drops
- Level 5: 1% of rune drops
- Level 6: 0.5% of rune drops
- Level 7: 0.2% of rune drops
- Level 8: 0.1% of rune drops
- Level 9: 0.05% of rune drops
- Level 10: 0.01% of rune drops

### Rune Value

**Value Formula:**
```
Rune Value = Base Value × Level Multiplier × Rarity Multiplier
```

**Base Values:**
- Level 1: 50 gold
- Level 2: 100 gold
- Level 3: 200 gold
- Level 4: 400 gold
- Level 5: 800 gold
- Level 6: 1,500 gold
- Level 7: 3,000 gold
- Level 8: 6,000 gold
- Level 9: 12,000 gold
- Level 10: 50,000 gold

**Rarity Multipliers:**
- Common: 1.0x
- Uncommon: 1.2x
- Rare: 1.5x
- Very Rare: 2.0x
- Extremely Rare: 5.0x

## Rust Implementation Considerations

### Data Structures

**Slot Component:**
```rust
#[derive(Component, Clone)]
pub struct ItemSlots {
    pub max_slots: u8,
    pub used_slots: u8,
    pub slots: Vec<Option<SlotContent>>,
}

#[derive(Clone)]
pub enum SlotContent {
    Rune(Rune),
    Card(Card), // See CARDS.md
}

#[derive(Clone, Copy)]
pub struct Rune {
    pub rune_type: RuneType,
    pub level: u8, // 1-10
    pub resistance_bonus: f32, // Percentage (1.0 = 1%)
}

#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum RuneType {
    Physical,
    Fire,
    Cold,
    Poison,
    Energy,
    Light,
    Dark,
}

impl Rune {
    pub fn new(rune_type: RuneType, level: u8) -> Self {
        let resistance_bonus = match level {
            1 => 1.0,
            2 => 1.5,
            3 => 2.0,
            4 => 2.5,
            5 => 3.0,
            6 => 4.0,
            7 => 5.0,
            8 => 6.5,
            9 => 8.5,
            10 => 15.0,
            _ => 0.0,
        };
        
        Rune {
            rune_type,
            level,
            resistance_bonus,
        }
    }
}
```

### Slot Management

**Slot Functions:**
```rust
impl ItemSlots {
    pub fn new(max_slots: u8) -> Self {
        ItemSlots {
            max_slots,
            used_slots: 0,
            slots: vec![None; max_slots as usize],
        }
    }
    
    pub fn has_rune(&self) -> bool {
        self.slots.iter().any(|slot| {
            matches!(slot, Some(SlotContent::Rune(_)))
        })
    }
    
    pub fn insert_rune(&mut self, slot_index: usize, rune: Rune) -> Result<(), String> {
        if slot_index >= self.slots.len() {
            return Err("Invalid slot index".to_string());
        }
        
        if self.slots[slot_index].is_some() {
            return Err("Slot already occupied".to_string());
        }
        
        if self.has_rune() {
            return Err("Cannot use 2 runes on same equipment".to_string());
        }
        
        self.slots[slot_index] = Some(SlotContent::Rune(rune));
        self.used_slots += 1;
        Ok(())
    }
    
    pub fn remove_rune(&mut self, slot_index: usize) -> Result<(), String> {
        if slot_index >= self.slots.len() {
            return Err("Invalid slot index".to_string());
        }
        
        match &self.slots[slot_index] {
            Some(SlotContent::Rune(_)) => {
                self.slots[slot_index] = None;
                self.used_slots -= 1;
                Ok(())
            },
            Some(SlotContent::Card(_)) => {
                Err("Slot contains card, not rune".to_string())
            },
            None => Err("Slot is empty".to_string()),
        }
    }
    
    pub fn calculate_total_resistance(&self, resistance_type: RuneType) -> f32 {
        let mut total = 0.0;
        
        for slot in &self.slots {
            if let Some(SlotContent::Rune(rune)) = slot {
                if rune.rune_type == resistance_type {
                    total += rune.resistance_bonus;
                }
            }
        }
        
        total
    }
}
```

### Equipment Slot Limits

**Slot Limit Function:**
```rust
pub fn get_max_slots(equipment_type: EquipmentType) -> u8 {
    match equipment_type {
        EquipmentType::TwoHandedWeapon => 3,
        EquipmentType::Shield | EquipmentType::Offhand => 2,
        EquipmentType::Helmet 
        | EquipmentType::Chest 
        | EquipmentType::Legs 
        | EquipmentType::Boots 
        | EquipmentType::Gloves => 6,
        EquipmentType::OneHandedWeapon 
        | EquipmentType::Tool => 1,
        EquipmentType::Ring 
        | EquipmentType::Necklace 
        | EquipmentType::Accessory => 0, // No slots for accessories
        _ => 0,
    }
}
```

## Integration with Other Systems

### Resistance System

**Integration:**
- Rune resistance bonuses are added to base resistance
- Total resistance includes rune bonuses
- Resistance cap (70%) applies to total including runes
- See [RESISTANCES.md](../core/RESISTANCES.md) for resistance calculations

### Equipment System

**Integration:**
- Slots are part of equipment items
- Slots can be added via Magic Stones
- Runes modify equipment resistance values
- Enchanted items can also have runes (both systems work together)

### Magic Stones System

**Integration:**
- Magic Stones can add slots to items
- Slot addition is permanent
- Maximum slot limits still apply
- See [MAGIC_STONES.md](./MAGIC_STONES.md) for Magic Stone details

### Cards System

**Integration:**
- Runes and cards share the same slot system
- Cannot use 2 runes or 2 cards on same equipment
- Can use 1 rune + 1 card on same equipment (different slots)
- See [CARDS.md](./CARDS.md) for card system details

### Drop System

**Integration:**
- Runes drop from creatures
- Drop rates vary by creature rarity
- Higher level runes are rarer drops
- Rune type matches creature's damage/resistance type

## Summary

The runes system:

- Provides slot-based enhancement system for equipment
- Allows resistance bonuses through runes
- Supports multiple rune levels (1-10) with increasing bonuses
- Maximum 15% resistance per Level 10 rune
- Equipment-specific slot limits (weapons: 1-3, armor: up to 6, accessories: 0)
- Natural slot drops and Magic Stone slot addition
- Single rune limit per equipment (cannot use 2 runes)
- Rune upgrade system for combining lower level runes
- Visual effects matching rune type and level
- Integration with resistance, equipment, Magic Stones, and Cards systems

This system creates depth in equipment customization, allowing players to specialize in specific resistances through strategic rune placement while maintaining balance through slot limits and rune rarity.
