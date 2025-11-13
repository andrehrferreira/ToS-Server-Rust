# Projectiles System

## Overview

Projectiles are entities that represent ranged attacks, spells, arrows, and other ranged abilities. They move through the world, collide with targets, and apply damage or effects. The projectile system supports various types including arrows, explosive projectiles, piercing projectiles, and condition-applying projectiles.

## Projectile Entity Base

### Base Properties

```csharp
public class ProjectileEntity : Entity {
    public float ActualMovespeed;
    public int Damage;
    public DamageType DamageType;
    public byte FrameMovespeedByte;
    public byte FrameRotationByte;
    public ProjectileKind Kind;
    public float Lifetime;
    
    public CreatureEntity Owner;
    public float TimeSpeed = 1.0f;
    public float TotalLifetime;
    public SkillName DamageSkill;
    
    public Vector2 Velocity;
    protected Entity Other;
    
    public event TargetHitHandler OnTargetHit;
    
    private Predicate<Entity> CollisionFilter;
    private Predicate<Entity> MovementModifierOnly;
}
```

### Projectile Properties

**Movement:**
- **Velocity**: Current velocity vector
- **ActualMovespeed**: Current movement speed
- **FrameMovespeedByte**: Quantized speed for packets
- **FrameRotationByte**: Quantized rotation for packets
- **TimeSpeed**: Time speed multiplier

**Combat:**
- **Damage**: Base damage value
- **DamageType**: Type of damage
- **DamageSkill**: Skill that created projectile
- **Owner**: Entity that fired projectile

**Lifetime:**
- **Lifetime**: Remaining lifetime
- **TotalLifetime**: Maximum lifetime

**Collision:**
- **CollisionFilter**: Entities that block projectile
- **MovementModifierOnly**: Entities that modify movement
- **Other**: Collided entity

## Projectile Creation

### Create Method

```csharp
public static T Create<T>(
    Scene scene, 
    T projectile, 
    Vector2 direction, 
    int damage, 
    DamageType damageType, 
    ProjectileKind kind, 
    SkillName damageSkill, 
    float maxRange, 
    float speed, 
    float radius = 0.25f
) where T : ProjectileEntity {
    projectile.CollisionShape = Shape.NewCircle(radius);
    projectile.Position = projectile.Owner.Position;
    projectile.Rotation = direction;
    projectile.Damage = damage;
    projectile.Lifetime = maxRange;
    projectile.Speed = speed;
    projectile.DamageType = damageType;
    projectile.Kind = kind;
    projectile.Team = projectile.Owner.Team;
    projectile.TotalLifetime = projectile.Lifetime;
    projectile.Scene = scene;
    projectile.DamageSkill = damageSkill;
    
    Entity collidesWith = projectile.Integrate(
        direction, 
        projectile.Owner.Radius
    );
    
    scene.Add(projectile);
    
    if (collidesWith != null) {
        projectile.OnCollide(collidesWith);
        collidesWith.OnCollide(projectile);
    }
    
    return projectile;
}
```

### Creation Parameters

**Required:**
- **Scene**: World to spawn in
- **Direction**: Firing direction
- **Damage**: Base damage
- **DamageType**: Type of damage
- **Kind**: Projectile kind
- **MaxRange**: Maximum travel distance
- **Speed**: Movement speed

**Optional:**
- **Radius**: Collision radius (default 0.25f)
- **DamageSkill**: Skill that created it

## Projectile Types

### Arrow Projectile

```csharp
public class ArrowProjectile : ProjectileEntity {
    public int EffectsEfficiency = 100;
    public int Lifesteal = 0;
    
    public override int ApplyDamage(Entity other) {
        int damage = other.TakeDamage(this, Owner, Damage, DamageType);
        
        if (damage > 0) {
            Owner.ApplyOnHitEffectsOn(other, EffectsEfficiency);
            
            if (Lifesteal > 0) {
                Owner.AddHealth(
                    (damage * Lifesteal * (Owner.HealingScaling + 100)) / 10000
                );
            }
        }
        
        other.RemoveArmorDurability();
        return damage;
    }
}
```

**Features:**
- Standard arrow behavior
- On-hit effects application
- Lifesteal support
- Armor durability reduction

### Explosive Arrow Projectile

```csharp
public class ExplosiveArrowProjectile : ProjectileEntity {
    public int EffectsEfficiency = 100;
    public int ExplosionDamage;
    public DamageType ExplosionDamageType;
    public float ExplosionRadius;
    
    public override int ApplyDamage(Entity other) {
        int damage = other.TakeDamage(this, Owner, Damage, DamageType);
        
        if (damage > 0) {
            Owner.ApplyOnHitEffectsOn(other, EffectsEfficiency);
            
            var enemies = Scene.CheckCircle(
                this, 
                Position, 
                ExplosionRadius, 
                this.Owner.EnemyPredicate
            );
            
            foreach(Entity e in enemies) {
                e.TakeDamage(
                    this, 
                    Owner, 
                    ExplosionDamage, 
                    ExplosionDamageType, 
                    false
                );
            }
        }
        
        other.RemoveArmorDurability();
        return damage;
    }
}
```

**Features:**
- Explodes on impact
- Area damage to nearby enemies
- Separate explosion damage type
- Configurable explosion radius

### Piercing Arrow Projectile

```csharp
public class PiercingArrowProjectile : ProjectileEntity {
    public int EffectsEfficiency = 100;
    private List<Entity> damagedEnemies = new List<Entity>();
    
    public PiercingArrowProjectile() : base(
        entity => entity.BlocksProjectile && !(entity is CreatureEntity)
    ) {
    }
    
    public override int ApplyDamage(Entity other) {
        int value = base.ApplyDamage(other);
        
        if (value > 0) {
            Owner.ApplyOnHitEffectsOn(other, EffectsEfficiency);
        }
        
        other.RemoveArmorDurability();
        return value;
    }
    
    public override void Update() {
        base.Update();
        
        var enemies = Scene.CheckShape(this, this.Owner.EnemyPredicate);
        
        foreach (Entity enemy in enemies) {
            if (!damagedEnemies.Contains(enemy)) {
                damagedEnemies.Add(enemy);
                ApplyDamage(enemy);
            }
        }
    }
}
```

**Features:**
- Passes through enemies
- Damages multiple enemies
- Tracks damaged enemies
- Doesn't stop on creature hit

### Conditioning Projectile

```csharp
public class ConditioningProjectile : ProjectileEntity {
    public float ConditionLifetime;
    public ConditionType ConditionType;
    public int ConditionValue;
    
    public override int ApplyDamage(Entity other) {
        int result = base.ApplyDamage(other);
        
        if (result > 0) {
            other.ApplyCondition(new Condition(
                ConditionType, 
                ConditionLifetime, 
                Owner, 
                ConditionValue
            ));
        }
        
        return result;
    }
}
```

**Features:**
- Applies conditions on hit
- Configurable condition type
- Configurable condition value
- Configurable condition lifetime

### Knockback Projectile

```csharp
public class KnockbackProjectile : ProjectileEntity {
    public float KnockbackDuration;
    
    public override int ApplyDamage(Entity other) {
        int result = base.ApplyDamage(other);
        
        if (result > 0) {
            other.Knockback(KnockbackDuration, Rotation);
            other.ApplyCondition(new Condition(
                ConditionType.Slowed, 
                2.5f, 
                Owner, 
                60
            ));
            Owner.ApplyOnHitEffectsOn(other, 100);
        }
        
        return result;
    }
}
```

**Features:**
- Knocks back target
- Applies slow condition
- Configurable knockback duration

### Silence Projectile

```csharp
public class SilenceProjectile : ProjectileEntity {
    public SilenceProjectile() : base(
        entity => entity.BlocksProjectile && !(entity is CreatureEntity)
    ) {
    }
    
    public override void Update() {
        base.Update();
        
        foreach (Entity entity in Scene.CheckCircle(
            this, 
            Position, 
            5f, 
            this.Owner.EnemyPredicate
        )) {
            if (entity is CharacterEntity enemyPlayer) {
                enemyPlayer.SetGlobalCooldown(TimeSpan.FromSeconds(3));
            }
        }
    }
}
```

**Features:**
- Applies global cooldown
- Area effect around projectile
- Affects player characters
- Continuous effect while active

## Movement System

### Update Method

```csharp
public override void Update() {
    BoundingBox bbox = BoundingBox;
    Vector2 from = Position;
    
    Velocity = Rotation * Speed;
    
    // Apply movement modifiers
    var modifiers = Scene.CheckShapeDynamicOnly(
        this, 
        MovementModifierOnly
    );
    
    foreach (Entity entity in modifiers) {
        entity.ApplyMovementModifier(this, ref Velocity);
    }
    
    float length = Velocity.Length();
    
    // Clamp velocity
    if (length < 2f) {
        float multiplier = 2f / length;
        length = 2f;
        Velocity *= multiplier;
    } else if (length > 25f) {
        float multiplier = 25f / length;
        length = 25f;
        Velocity *= multiplier;
    }
    
    ActualMovespeed = length * TimeSpeed;
    FrameMovespeedByte = (byte)(MathF.Ceiling(ActualMovespeed * 10.2f));
    
    CollisionShape.Rotation = Velocity.GetDirection();
    FrameRotationByte = (byte)(
        (CollisionShape.Rotation.GetAngle() + MathF.PI) * 40.58448017647087f
    );
    
    Entity collidesWith = Integrate(
        Rotation, 
        ActualMovespeed * Scene.DeltaTime
    );
    
    BoundingBox = bbox;
    
    if (Position != from) {
        NotifyPositionChanged(Position - from);
    }
    
    if (collidesWith != null) {
        OnCollide(collidesWith);
    } else {
        Lifetime -= ActualMovespeed * Scene.DeltaTime;
        
        if (Lifetime <= 0) {
            Scene.Remove(this);
        }
    }
}
```

### Movement Features

**Velocity Management:**
- Calculated from rotation and speed
- Modified by movement modifiers
- Clamped between 2f and 25f
- Quantized for network packets

**Lifetime:**
- Decremented by distance traveled
- Removed when lifetime reaches 0
- Based on actual movement speed

## Collision System

### Collision Filter

**Default Filter:**
```csharp
CollisionFilter = entity => 
    entity.BlocksProjectile && 
    !entity.Untargettable && 
    Team.IsEnemyOf(entity.Team);
```

**Custom Filters:**
- Can be overridden in constructors
- Used for special projectile types
- Determines what blocks projectile

### Integrate Method

```csharp
protected Entity Integrate(Vector2 direction, float length) {
    int tierBreak = 10;
    
    while (length > float.Epsilon && tierBreak > 0) {
        --tierBreak;
        
        float stepLength = MathF.Min(
            Constants.HalfMinColliderWidth, 
            length
        );
        
        length -= stepLength;
        
        Vector2 velocity = direction * stepLength;
        
        CollisionShape.Position = CollisionShape.Position + velocity;
        BoundingBox = CollisionShape.GetCells();
        
        CollisionResult result;
        Entity other = Scene.ResolveCollision(
            this, 
            CollisionFilter, 
            out result
        );
        
        if (result.Penetration > 0 && other != null) {
            return other;
        }
    }
    
    return null;
}
```

**Integration:**
- Step-based movement
- Collision detection each step
- Returns collided entity
- Prevents tunneling

### OnCollide Method

```csharp
public override void OnCollide(Entity other) {
    if (Other == null) {
        int damage = ApplyDamage(other);
        
        if (damage != -1) {
            Other = other;
            other.AddSkillExperienceToOnTakeDamage(Owner, DamageSkill);
            OnTargetHit?.Invoke(other, damage);
            Scene.Remove(this);
            Lifetime = float.PositiveInfinity;
        }
    }
}
```

**Collision Behavior:**
- Applies damage to target
- Grants skill experience
- Triggers hit event
- Removes projectile
- Prevents multiple hits

## Damage Application

### ApplyDamage Method

```csharp
public virtual int ApplyDamage(Entity other) {
    return other.TakeDamage(this, Owner, Damage, DamageType);
}
```

**Base Behavior:**
- Applies damage to target
- Uses owner as damage causer
- Returns damage dealt
- Can be overridden for special effects

## Network Packets

### Create Packet

```csharp
public override void WriteCreatePacket(PlayerController ctrl, uint createdAt) {
    var buffer = ctrl.Connection.BeginUnreliable();
    
    buffer.Put((byte)ServerPacketType.ProjectileCreate);
    buffer.Put(Id);
    buffer.Put((byte)Kind);
    buffer.PutQuantized(CollisionShape.Position);
    buffer.Put(FrameRotationByte);
    buffer.Put(FrameMovespeedByte);
    
    ctrl.Connection.EndUnreliable();
}
```

### Update Packet

```csharp
public override void WriteUpdatePacket(PlayerController ctrl, uint sinceTick) {
    var buffer = ctrl.Connection.BeginUnreliable();
    
    buffer.Put((byte)ServerPacketType.ProjectileUpdate);
    buffer.Put(Id);
    buffer.PutQuantized(CollisionShape.Position);
    buffer.Put(FrameRotationByte);
    buffer.Put(FrameMovespeedByte);
    
    ctrl.Connection.EndUnreliable();
}
```

**Packet Features:**
- Quantized position
- Quantized rotation
- Quantized speed
- Unreliable channel

## Implementation Notes

### Performance

**Optimizations:**
- Quantized values for network
- Step-based collision
- Early termination on hit
- Efficient collision filtering

### Special Behaviors

**Piercing:**
- Doesn't stop on creature hit
- Damages multiple enemies
- Tracks damaged entities

**Explosive:**
- Area damage on impact
- Separate explosion damage
- Configurable radius

**Conditioning:**
- Applies conditions
- Configurable effects
- Various condition types

## Related Documentation

- [CREATURES.md](./CREATURES.md) - Creature system
- [ACTIONS.md](../skills/ACTIONS.md) - Action system
- [COMBAT.md](../combat/COMBAT.md) - Combat system

