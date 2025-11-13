# Conditions System Documentation

## Overview

The Conditions system manages temporary states and effects applied to entities, such as damage over time (DoT) effects, healing over time (HoT) effects, buffs, and debuffs. This system is crucial for implementing status effects, combat mechanics, and temporary character modifications.

## Architecture

### Condition Types

Conditions are categorized into several types based on their effects:

**Damage Over Time (DoT):**
- `Burning`: Fire damage over time
- `Bleeding`: Physical damage over time
- `Poisoned`: Poison damage over time
- `Electrified`: Lightning damage over time

**Healing Over Time (HoT):**
- `Healing`: Health regeneration over time

**Status Effects:**
- `Stunned`: Entity cannot act
- `Frozen`: Entity cannot move or act
- `Feared`: Entity runs away from source
- `Slowed`: Entity movement speed reduced
- `Snared`: Entity movement speed greatly reduced
- `Chilling`: Entity is chilled (slowed)

**Special Conditions:**
- `PlayerKiller`: Entity is marked as a player killer (C# version)

### Condition Structure

**Core Properties:**
- `Type`: Condition type (Burning, Poisoned, etc.)
- `Dealer`: Entity that applied the condition
- `Lifetime`: Duration of the condition in milliseconds
- `Value`: Damage/healing value (dice roll)
- `DamageBonus`: Additional damage bonus

**Buff/Debuff Properties:**
- `Type`: Buff/debuff type (HeavenlyProtection, etc.)
- `Dealer`: Entity that applied the buff/debuff
- `Lifetime`: Duration of the buff/debuff in milliseconds
- `Action`: Action associated with the buff/debuff

## Condition Types

### Version 1: C# Server3 (ConditionType)

```csharp
public enum ConditionType
{
    None,
    Burning,
    Bleeding,
    Electrified,
    Chilling,
    Poisoned,
    Healing,
    Stunned,
    PlayerKiller,
    Slowed,
    Snared,
    Frozen,
    Feared
}
```

### Version 2: TypeScript (ConditionType)

```typescript
export enum ConditionType {
    None,
    Burning,
    Bleeding,
    Electrified,
    Chilling,
    Poisoned,
    Healing,
    Stunned,
    Slowed,
    Snared,
    Frozen,
    Feared
}
```

## Condition Implementation

### Condition Class

**TypeScript Implementation:**
```typescript
export class Condition {
    public Dealer: Entity;
    public Lifetime: number;
    public Type: ConditionType;
    public Value: Dices;
    public DamageBonus: number = 0;

    constructor(type: ConditionType, lifetime: number, dealer: Entity, value: Dices = Dices.None, damageBonus: number = 0) {
        this.Type = type;
        this.Lifetime = lifetime * 1000; // Convert to milliseconds
        this.Dealer = dealer;
        this.Value = value;
        this.DamageBonus = damageBonus;
    }

    public apply(c: Entity) {
        switch(this.Type) {
            case ConditionType.Burning: c.states.addFlag(EntityStates.Burning); break;
            case ConditionType.Bleeding: c.states.addFlag(EntityStates.Bleeding); break;
            case ConditionType.Poisoned: c.states.addFlag(EntityStates.Poisoned); break;
            case ConditionType.Slowed: break;
            case ConditionType.Frozen: c.states.addFlag(EntityStates.Frozen); break;
            case ConditionType.Stunned: c.states.addFlag(EntityStates.Stunned); break;
            case ConditionType.Feared: c.states.addFlag(EntityStates.Feared); break;
        }
    }

    public refresh(c: Entity, dealer: Entity, newLifetime: number, newValue: Dices) {
        switch(this.Type) {
            case ConditionType.Burning:
            case ConditionType.Bleeding:
            case ConditionType.Poisoned:
                this.Lifetime = newLifetime + (this.Lifetime % 1);
                this.Value = newValue;
                break;
        }
    }

    public remove(c: Entity) {
        switch(this.Type) {
            case ConditionType.Burning: c.states.removeFlag(EntityStates.Burning); break;
            case ConditionType.Bleeding: c.states.removeFlag(EntityStates.Bleeding); break;
            case ConditionType.Poisoned: c.states.removeFlag(EntityStates.Poisoned); break;
            case ConditionType.Frozen: c.states.removeFlag(EntityStates.Frozen); break;
            case ConditionType.Stunned: c.states.removeFlag(EntityStates.Stunned); break;
            case ConditionType.Feared: c.states.removeFlag(EntityStates.Feared); break;
        }
    }

    public update(c: Entity) {
        let previousLifetime = this.Lifetime;
        this.Lifetime -= 1000; // Decrease by 1 second

        switch(this.Type) {
            case ConditionType.Burning: 
                if(Math.trunc(this.Lifetime) < Math.trunc(previousLifetime))
                    c.takeDamage(this.Dealer, this.Value, DamageType.Fire, this.DamageBonus);                
                break;
            case ConditionType.Poisoned: 
                if(Math.trunc(this.Lifetime) < Math.trunc(previousLifetime))
                    c.takeDamage(this.Dealer, this.Value, DamageType.Poison, this.DamageBonus);                
                break;
            case ConditionType.Bleeding: 
                if(Math.trunc(this.Lifetime) < Math.trunc(previousLifetime))
                    c.takeDamage(this.Dealer, this.Value, DamageType.Bleed, this.DamageBonus);                
                break;
            case ConditionType.Healing:
                if(Math.trunc(this.Lifetime) < Math.trunc(previousLifetime))
                    c.heal(this.Dealer, c.rollDice(this.Value));
                break;
        }
    }
}
```

## Buff/Debuff Implementation

### BuffDebuff Class

**TypeScript Implementation:**
```typescript
export class BuffDebuff {
    public Type: BuffDebuffStates;
    public Dealer: Entity;
    public Lifetime: number;
    public Action: BaseAction;

    constructor(type: BuffDebuffStates, lifetime: number, dealer: Entity, action: BaseAction) {
        this.Type = type;
        this.Lifetime = lifetime * 1000; // Convert to milliseconds
        this.Dealer = dealer;
        this.Action = action;
    }

    public apply(c: Entity) {
        c.buffsDebuffsState.addFlag(this.Type);
    }

    public refresh(c: Entity, newLifetime: number) {
        this.Lifetime = newLifetime + (this.Lifetime % 1);
    }

    public remove(c: Entity) {
        c.buffsDebuffsState.removeFlag(this.Type);
    }

    public update(c: Entity) {
        this.Lifetime -= 1000; // Decrease by 1 second
    }
}
```

## Condition Lifecycle

### 1. Application

**Process:**
1. Create condition with type, lifetime, dealer, and value
2. Call `apply()` method to apply condition to entity
3. Update entity flags (Burning, Poisoned, etc.)
4. Add condition to entity's condition list
5. Broadcast condition application to clients

**Example:**
```typescript
const condition = new Condition(
    ConditionType.Burning,
    5.0, // 5 seconds
    dealer,
    Dices.D6, // Damage dice
    10 // Damage bonus
);
condition.apply(entity);
entity.conditions.push(condition);
```

### 2. Update

**Process:**
1. Update condition lifetime (decrease by delta time)
2. Apply damage/healing based on condition type
3. Check if condition has expired
4. Remove condition if lifetime <= 0

**Update Frequency:**
- **Server**: Every tick (60 Hz)
- **Client**: Every frame (60+ Hz)

**Example:**
```typescript
for (const condition of entity.conditions) {
    condition.update(entity);
    
    if (condition.Lifetime <= 0) {
        condition.remove(entity);
        entity.conditions.remove(condition);
    }
}
```

### 3. Refresh

**Process:**
1. Check if condition can be refreshed (Burning, Bleeding, Poisoned)
2. Update condition lifetime (add remaining time)
3. Update condition value (new damage dice)
4. Keep condition active

**Example:**
```typescript
if (condition.Type === ConditionType.Burning) {
    condition.refresh(entity, dealer, 5.0, Dices.D8);
}
```

### 4. Removal

**Process:**
1. Call `remove()` method to remove condition from entity
2. Update entity flags (remove Burning, Poisoned, etc.)
3. Remove condition from entity's condition list
4. Broadcast condition removal to clients

**Example:**
```typescript
condition.remove(entity);
entity.conditions.remove(condition);
```

## Condition Effects

### Damage Over Time (DoT)

**Burning:**
- **Damage Type**: Fire
- **Effect**: Fire damage over time
- **Flag**: `EntityStates.Burning`
- **Update**: Damage every second
- **Stackable**: Yes (refreshable)

**Bleeding:**
- **Damage Type**: Physical (Bleed)
- **Effect**: Physical damage over time
- **Flag**: `EntityStates.Bleeding`
- **Update**: Damage every second
- **Stackable**: Yes (refreshable)

**Poisoned:**
- **Damage Type**: Poison
- **Effect**: Poison damage over time
- **Flag**: `EntityStates.Poisoned`
- **Update**: Damage every second
- **Stackable**: Yes (refreshable)

**Electrified:**
- **Damage Type**: Lightning
- **Effect**: Lightning damage over time
- **Flag**: None (future implementation)
- **Update**: Damage every second
- **Stackable**: Yes (refreshable)

### Healing Over Time (HoT)

**Healing:**
- **Healing Type**: Health regeneration
- **Effect**: Health regeneration over time
- **Flag**: None
- **Update**: Heal every second
- **Stackable**: Yes (refreshable)

### Status Effects

**Stunned:**
- **Effect**: Entity cannot act
- **Flag**: `EntityStates.Stunned`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**Frozen:**
- **Effect**: Entity cannot move or act
- **Flag**: `EntityStates.Frozen`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**Feared:**
- **Effect**: Entity runs away from source
- **Flag**: `EntityStates.Feared`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**Slowed:**
- **Effect**: Entity movement speed reduced
- **Flag**: None
- **Update**: No damage/healing
- **Stackable**: Yes (refreshable)

**Snared:**
- **Effect**: Entity movement speed greatly reduced
- **Flag**: None
- **Update**: No damage/healing
- **Stackable**: Yes (refreshable)

**Chilling:**
- **Effect**: Entity is chilled (slowed)
- **Flag**: `CreatureFlags.Chilled` (C# version)
- **Update**: No damage/healing
- **Stackable**: Yes (refreshable)

## Buff/Debuff Effects

### Buffs

**HeavenlyProtection:**
- **Effect**: Provides protection from damage
- **Flag**: `BuffDebuffStates.HeavenlyProtection`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**EchoOfTheDeath:**
- **Effect**: Increases damage output
- **Flag**: `BuffDebuffStates.EchoOfTheDeath`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**ShatteringImpact:**
- **Effect**: Provides shattering impact damage
- **Flag**: `BuffDebuffStates.ShatteringImpact`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

**ThunderAura:**
- **Effect**: Provides thunder aura damage
- **Flag**: `BuffDebuffStates.ThunderAura`
- **Update**: No damage/healing
- **Stackable**: No (replace existing)

### Debuffs

**ManaBurn:**
- **Effect**: Burns mana over time
- **Flag**: `BuffsDebuffsFlags.ManaBurn` (C# version)
- **Update**: Mana drain every second
- **Stackable**: Yes (refreshable)

## Condition Management

### Entity Condition List

**Structure:**
- List of active conditions per entity
- Condition lookup by type
- Condition expiration tracking
- Condition refresh management

**Implementation:**
```typescript
class Entity {
    public conditions: Condition[] = [];
    public buffsDebuffs: BuffDebuff[] = [];

    public addCondition(condition: Condition) {
        // Check if condition already exists
        const existing = this.conditions.find(c => c.Type === condition.Type);
        
        if (existing) {
            // Refresh existing condition
            existing.refresh(this, condition.Dealer, condition.Lifetime / 1000, condition.Value);
        } else {
            // Add new condition
            condition.apply(this);
            this.conditions.push(condition);
        }
    }

    public removeCondition(type: ConditionType) {
        const condition = this.conditions.find(c => c.Type === type);
        if (condition) {
            condition.remove(this);
            this.conditions.remove(condition);
        }
    }

    public updateConditions(deltaTime: number) {
        for (const condition of this.conditions) {
            condition.update(this);
            
            if (condition.Lifetime <= 0) {
                this.removeCondition(condition.Type);
            }
        }
    }
}
```

### Condition Stacking

**Stackable Conditions:**
- **Damage Over Time**: Burning, Bleeding, Poisoned (refreshable)
- **Healing Over Time**: Healing (refreshable)
- **Movement Effects**: Slowed, Snared (refreshable)

**Non-Stackable Conditions:**
- **Status Effects**: Stunned, Frozen, Feared (replace existing)
- **Buffs/Debuffs**: Most buffs/debuffs (replace existing)

**Stacking Rules:**
1. **Refreshable**: Update lifetime and value
2. **Replaceable**: Remove existing, add new
3. **Stackable**: Add multiple instances (future implementation)

## Serialization

### Network Protocol

**Condition Packet Structure:**
```
Condition Packet:
- ConditionType: byte (1 byte)
- Lifetime: float (4 bytes)
- DealerId: uint32 (4 bytes)
- Value: Dices (1 byte)
- DamageBonus: int16 (2 bytes)
```

**Buff/Debuff Packet Structure:**
```
BuffDebuff Packet:
- BuffDebuffType: byte (1 byte)
- Lifetime: float (4 bytes)
- DealerId: uint32 (4 bytes)
- ActionId: uint32 (4 bytes)
```

### Serialization Format

**Write:**
```csharp
buffer.Put((byte)condition.Type);
buffer.Put(condition.Lifetime);
buffer.Put(condition.Dealer.Id);
buffer.Put((byte)condition.Value);
buffer.Put((short)condition.DamageBonus);
```

**Read:**
```csharp
ConditionType type = (ConditionType)buffer.GetByte();
float lifetime = buffer.GetFloat();
uint dealerId = buffer.GetUInt();
Dices value = (Dices)buffer.GetByte();
short damageBonus = buffer.GetShort();
```

## Performance Considerations

### Optimization

**Condition Updates:**
- Update conditions in batches
- Use efficient data structures (HashMap, Vec)
- Cache frequently accessed conditions
- Optimize condition lookup

**Memory Management:**
- Reuse condition objects (object pooling)
- Minimize condition allocations
- Efficient condition storage
- Garbage collection optimization

**Network Bandwidth:**
- Delta compression for condition updates
- Only send changed conditions
- Batch condition updates
- Compress condition data

### Rust Implementation

**Recommended Approach:**
```rust
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum ConditionType {
    None,
    Burning,
    Bleeding,
    Electrified,
    Chilling,
    Poisoned,
    Healing,
    Stunned,
    Slowed,
    Snared,
    Frozen,
    Feared,
}

pub struct Condition {
    pub condition_type: ConditionType,
    pub dealer_id: EntityId,
    pub lifetime: f32, // in seconds
    pub value: Dices,
    pub damage_bonus: i16,
}

impl Condition {
    pub fn new(
        condition_type: ConditionType,
        lifetime: f32,
        dealer_id: EntityId,
        value: Dices,
        damage_bonus: i16,
    ) -> Self {
        Self {
            condition_type,
            dealer_id,
            lifetime,
            value,
            damage_bonus,
        }
    }

    pub fn apply(&self, entity: &mut Entity) {
        match self.condition_type {
            ConditionType::Burning => entity.flags.add_flag(EntityFlag::Burning),
            ConditionType::Bleeding => entity.flags.add_flag(EntityFlag::Bleeding),
            ConditionType::Poisoned => entity.flags.add_flag(EntityFlag::Poisoned),
            ConditionType::Frozen => entity.flags.add_flag(EntityFlag::Frozen),
            ConditionType::Stunned => entity.flags.add_flag(EntityFlag::Stunned),
            ConditionType::Feared => entity.flags.add_flag(EntityFlag::Feared),
            _ => {}
        }
    }

    pub fn remove(&self, entity: &mut Entity) {
        match self.condition_type {
            ConditionType::Burning => entity.flags.remove_flag(EntityFlag::Burning),
            ConditionType::Bleeding => entity.flags.remove_flag(EntityFlag::Bleeding),
            ConditionType::Poisoned => entity.flags.remove_flag(EntityFlag::Poisoned),
            ConditionType::Frozen => entity.flags.remove_flag(EntityFlag::Frozen),
            ConditionType::Stunned => entity.flags.remove_flag(EntityFlag::Stunned),
            ConditionType::Feared => entity.flags.remove_flag(EntityFlag::Feared),
            _ => {}
        }
    }

    pub fn update(&mut self, entity: &mut Entity, delta_time: f32) {
        self.lifetime -= delta_time;

        if self.lifetime <= 0.0 {
            return;
        }

        // Apply damage/healing every second
        if (self.lifetime as u32) < ((self.lifetime + delta_time) as u32) {
            match self.condition_type {
                ConditionType::Burning => {
                    entity.take_damage(self.dealer_id, self.value, DamageType::Fire, self.damage_bonus);
                }
                ConditionType::Poisoned => {
                    entity.take_damage(self.dealer_id, self.value, DamageType::Poison, self.damage_bonus);
                }
                ConditionType::Bleeding => {
                    entity.take_damage(self.dealer_id, self.value, DamageType::Bleed, self.damage_bonus);
                }
                ConditionType::Healing => {
                    entity.heal(self.dealer_id, entity.roll_dice(self.value));
                }
                _ => {}
            }
        }
    }
}
```

## Usage Examples

### Applying a Condition

```typescript
// TypeScript
const condition = new Condition(
    ConditionType.Burning,
    5.0, // 5 seconds
    dealer,
    Dices.D6, // Damage dice
    10 // Damage bonus
);
entity.addCondition(condition);
```

```rust
// Rust
let condition = Condition::new(
    ConditionType::Burning,
    5.0, // 5 seconds
    dealer_id,
    Dices::D6, // Damage dice
    10, // Damage bonus
);
condition.apply(&mut entity);
entity.conditions.push(condition);
```

### Updating Conditions

```typescript
// TypeScript
entity.updateConditions(deltaTime);
```

```rust
// Rust
for condition in &mut entity.conditions {
    condition.update(&mut entity, delta_time);
    
    if condition.lifetime <= 0.0 {
        condition.remove(&mut entity);
        // Remove from conditions list
    }
}
```

### Removing a Condition

```typescript
// TypeScript
entity.removeCondition(ConditionType.Burning);
```

```rust
// Rust
if let Some(index) = entity.conditions.iter().position(|c| c.condition_type == ConditionType::Burning) {
    entity.conditions[index].remove(&mut entity);
    entity.conditions.remove(index);
}
```

## Best Practices

### Condition Management

1. **Server Authority**: Server is authoritative for all condition states
2. **Validation**: Validate condition applications on the server
3. **Synchronization**: Synchronize conditions between server and client
4. **Expiration**: Properly handle condition expiration
5. **Stacking**: Implement proper condition stacking rules

### Performance Optimization

1. **Batch Updates**: Update conditions in batches
2. **Efficient Data Structures**: Use efficient data structures for condition storage
3. **Object Pooling**: Reuse condition objects to reduce allocations
4. **Cache Optimization**: Cache frequently accessed conditions
5. **Network Optimization**: Use delta compression for condition updates

### Error Prevention

1. **Type Safety**: Use type-safe condition operations
2. **Validation**: Validate condition values before applying
3. **Bounds Checking**: Check condition lifetime bounds
4. **State Consistency**: Ensure condition state consistency
5. **Error Recovery**: Implement error recovery for invalid conditions

## Conclusion

The Conditions system provides a flexible and efficient way to manage temporary states and effects on entities. By using a structured approach with condition types, lifetimes, and effects, the system enables:

- **Damage Over Time**: Fire, poison, bleeding effects
- **Healing Over Time**: Health regeneration effects
- **Status Effects**: Stun, freeze, fear effects
- **Buffs/Debuffs**: Temporary character modifications
- **Efficient Management**: Optimized condition updates and expiration

The Rust implementation will leverage type safety and efficient data structures to provide a safe and performant condition system while maintaining the flexibility of the current implementation.
