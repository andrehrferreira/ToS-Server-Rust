# Humanoids System

## Overview

Humanoids are creatures that have humanoid features, including equipment slots, visual appearance, and the ability to equip armor, weapons, and accessories. The `Humanoid` class extends `Creature` and adds equipment management, stat calculation from equipment, and combat mechanics based on equipped items.

## Humanoid Class

### Base Properties

```typescript
class Humanoid extends Entity {
    public visual: string;
    
    // Equipment Slots
    public helmet: IEquipament;
    public chest: IEquipament;
    public gloves: IEquipament;
    public pants: IEquipament;
    public boots: IEquipament;
    public robe: IEquipament;
    public cloak: IEquipament;
    
    public ring01: IEquipament;
    public ring02: IEquipament;
    public necklance: IEquipament;
    public instrument: IEquipament;
    
    public mainhand: IEquipament;
    public offhand: IEquipament;
    
    public pet: IEquipament;
    public petInstance: Pet;
    public mount: IEquipament;
    
    // Gathering Tools
    public pickaxetool: IEquipament;
    public axetool: IEquipament;
    public scythetool: IEquipament;
    
    public equipaments: Array<Equipament> = new Array<Equipament>();
}
```

## Equipment System

### Equipment Types

**Armor Slots:**
- **Helmet**: Head protection
- **Chest**: Torso armor
- **Gloves**: Hand protection
- **Pants**: Leg armor
- **Boots**: Foot protection
- **Robe**: Overlay armor
- **Cloak**: Back accessory

**Accessories:**
- **Ring01**: First ring slot
- **Ring02**: Second ring slot
- **Necklace**: Neck accessory
- **Instrument**: Musical instrument

**Weapons:**
- **Mainhand**: Primary weapon
- **Offhand**: Shield or secondary weapon

**Special:**
- **Pet**: Pet item slot
- **Mount**: Mount item slot

**Tools:**
- **PickaxeTool**: Mining tool
- **AxeTool**: Woodcutting tool
- **ScytheTool**: Harvesting tool

### Equipment Interface

```typescript
interface IEquipament {
    ItemName: string;
    ItemRef: string;
}
```

## Equipping System

### Equip Method

```typescript
async equip(
    type: EquipamentType, 
    itemId: string, 
    itemRef: string, 
    ring02: boolean = false
) {
    // Validate item in inventory
    // Clear inventory slot
    // Desequip existing item in slot
    // Set new equipment
    // Handle two-handed weapon conflicts
    // Broadcast equip packet
    // Refresh equipment list
    // Recalculate stats
    // Save entity
}
```

### Equipment Conflicts

**Two-Handed Weapons:**
- Two-handed weapons unequip offhand automatically
- Offhand cannot be equipped with two-handed mainhand

**Two-Handed Weapon Types:**
- TwoHandedAxe
- TwoHandedHammer
- TwoHandedSword
- Crossbow
- Bow
- Staff
- Spear

### Desequip Method

```typescript
async desequip(
    type: EquipamentType,
    ring02: boolean = false,
    broadcast: boolean = false,
    slotId: number = -1
) {
    // Get equipment reference
    // Clear equipment slot
    // Add to inventory
    // Call onDesequip callback
    // Broadcast desequip packet
    // Refresh equipment list
    // Recalculate stats
    // Save entity
}
```

## Equipment Refresh

### Refresh Equipments List

```typescript
refreshEquipamentsList() {
    let equipaments = new Array<Equipament>();
    
    // Add all equipped items to array
    // Call onEquip for pets/mounts
    // Update equipment list
    // Recalculate stats
    // Refresh equipment weight flags
}
```

### Equipment Weight

**Equipment Weight Types:**
- **Light**: Light armor
- **Medium**: Medium armor
- **Heavy**: Heavy armor

**Weight Flags:**
- **hasMediumEquipamentPart**: Has medium armor pieces
- **hasHeavyEquipamentPart**: Has heavy armor pieces

## Stat Calculation

### Equipment Attributes

```typescript
getEquipamentsAttr(type: AttributeType): number {
    let total = 0;
    
    this.equipaments.map((equipament) => {
        if (equipament.Attrs.has(type) && 
            !equipament.Flags.hasFlag(ItemStates.Broken)) {
            total += equipament.Attrs.get(type);
        }
    });
    
    return total;
}
```

### Attribute Types

**Combat Attributes:**
- BonusStr, BonusDex, BonusInt
- BonusVig, BonusAgi, BonusLuc
- BonusDamage, BonusMagicDamage
- CriticalChance, CriticalDamage
- DamageReduction, DodgeChance

**Resistance Attributes:**
- ReflectPhysical, ReflectSpell
- LowerManaCost, FasterCasting
- CooldownReduction

**Elemental Damage:**
- FireDamage, ColdDamage
- PoisonDamage, EnergyDamage
- LightDamage, DarkDamage

**Regeneration:**
- HealthRegen, ManaRegen, StaminaRegen

**Gathering Bonuses:**
- BonusCollectsMineral
- BonusCollectsSkins
- BonusCollectsWood

### Equipment Resistance

```typescript
getEquipamentsResistence(resistenceType: ResistenceType): number {
    let total = 0;
    
    this.equipaments.map((equipament) => {
        if (!equipament?.Flags.hasFlag(ItemStates.Broken)) {
            switch(resistenceType) {
                case ResistenceType.Physical: 
                    total += equipament.Armor; 
                    break;
                case ResistenceType.Fire: 
                    total += equipament.FireResistence; 
                    break;
                // ... other resistances
            }
        }
    });
    
    return total;
}
```

### Resistance Types

- **Physical**: Armor value
- **Fire**: Fire resistance
- **Cold**: Cold resistance
- **Poison**: Poison resistance
- **Energy**: Energy resistance
- **Light**: Light resistance
- **Dark**: Dark resistance

## Stat Calculation

### Calculate Statics

```typescript
calculateStatics(): void {
    super.calculateStatics();
    
    // Bonus stats from equipment
    this.bonusStr = this.getEquipamentsAttr(AttributeType.BonusStr);
    this.bonusDex = this.getEquipamentsAttr(AttributeType.BonusDex);
    // ... other stats
    
    // Resistances from equipment
    this.physicalResistence = Math.min(
        this.physicalResistence + 
        this.getEquipamentsResistence(ResistenceType.Physical), 
        70
    );
    // ... other resistances
    
    // Regeneration from equipment
    this.lifeRegeneration = this.getEquipamentsAttr(AttributeType.HealthRegen);
    // ... other regeneration
    
    // Combat stats
    this.bonusPhysicalDamage = this.getEquipamentsAttr(AttributeType.BonusDamage);
    this.criticalChance = this.getEquipamentsAttr(AttributeType.CriticalChance);
    // ... other combat stats
    
    // Elemental damage
    this.fireDamage = this.getEquipamentsAttr(AttributeType.FireDamage);
    // ... other elemental damage
    
    // Gathering bonuses
    this.bonusCollectsMineral = this.getEquipamentsAttr(AttributeType.BonusCollectsMineral);
    // ... other gathering bonuses
}
```

## Combat System

### Auto-Attack

```typescript
checkHitAutoAttack(data: ICheckHitAutoAttack): void {
    const actor = this.map.findEntityById(data.actorId);
    
    if (!actor || !this.validateHit(data, actor)) {
        throw new Error("Invalid data hit");
    }
    
    const bonusDamage = this.getWeaponBaseDamage();
    actor.takeDamage(
        this, 
        Dices.D1D4, 
        DamageType.Physic, 
        bonusDamage
    );
}
```

### Weapon Base Damage

```typescript
getWeaponBaseDamage(): number {
    const weapon = this.mainhand ? 
        Items.getItemByRef(this.mainhand.ItemRef) : null;
    
    let baseDamage = weapon ? (weapon as Weapon).Damage : Dices.D1D4;
    let bonusDamage = 0;
    
    // Check if weapon is broken
    if ((weapon as Weapon)?.Flags.hasFlag(ItemStates.Broken)) {
        baseDamage = Dices.D1D4;
    }
    
    if (weapon) {
        bonusDamage = (weapon as Weapon).BonusDamage;
    }
    
    const bonusBySkill = this.getSkillBonusByWeaponType(this.getWeaponType());
    
    return this.rollDice(baseDamage) + bonusBySkill + bonusDamage;
}
```

### Weapon Damage Dice

```typescript
getWeaponDiceDamage(): Dices {
    let weapon = this.mainhand ? 
        Items.getItemByRef(this.mainhand.ItemRef) : null;
    
    if ((weapon as Weapon)?.Flags.hasFlag(ItemStates.Broken)) {
        weapon = null;
    }
    
    return weapon ? (weapon as Weapon).Damage : Dices.D1D4;
}
```

### Weapon Speed

```typescript
getWeaponSpeed(): number {
    const weapon = this.mainhand ? 
        Items.getItemByRef(this.mainhand.ItemRef) : null;
    
    return weapon ? (weapon as Weapon).AttackSpeed : 0;
}
```

### Weapon Type

```typescript
getWeaponType(): WeaponType {
    const weapon = this.mainhand ? 
        Items.getItemByRef(this.mainhand.ItemRef) : null;
    
    return weapon ? (weapon as Weapon).WeaponType : WeaponType.None;
}
```

## Pet and Mount Integration

### Pet Equipment

- **Pet Slot**: Equips pet item
- **Pet Instance**: Active pet entity
- **onEquip**: Called when pet equipped
- **onDesequip**: Called when pet unequipped

### Mount Equipment

- **Mount Slot**: Equips mount item
- **onEquip**: Called when mount equipped
- **onDesequip**: Called when mount unequipped

## Destruction

### Cleanup

```typescript
destroy(): void {
    super.destroy();
    
    // Desequip pet
    if (this.pet && this.pet.ItemRef) {
        (Items.getItemByRef(this.pet.ItemRef) as PetItem)?.onDesequip(this);
    }
    
    // Desequip mount
    if (this.mount && this.mount.ItemRef) {
        (Items.getItemByRef(this.mount.ItemRef) as MountItem)?.onDesequip(this);
    }
}
```

## Item States

### Broken Items

- Broken items don't contribute stats
- Broken weapons use base damage (D1D4)
- Broken items can still be equipped
- Repair required to restore functionality

### Item State Checks

- Check `ItemStates.Broken` flag
- Skip broken items in stat calculation
- Use base values if weapon broken

## Implementation Notes

### Equipment Synchronization

- Equipment changes broadcast to nearby players
- Equipment packets sent on equip/desequip
- Visual updates for equipment changes
- Stat recalculation on equipment change

### Equipment Persistence

- Equipment saved to database
- Equipment loaded on character load
- Equipment state synchronized

### Performance Considerations

- Equipment list cached
- Stat calculation optimized
- Equipment refresh batched

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Base creature system
- [PLAYER.md](./PLAYER.md) - Player entity implementation
- [ITEMS.md](../items/ITEMS.md) - Item system
- [EQUIPMENT.md](../items/EQUIPMENT.md) - Equipment items

