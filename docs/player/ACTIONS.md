# Actions System Documentation

## Overview

The actions system is the core combat and ability system that handles all player skills, spells, and abilities. Actions are modular, extensible abilities that can be targeted, area-based, projectile-based, or self-targeted. The system includes complex damage calculation, skill requirements, cooldowns, resource costs, and dynamic effect scaling based on skill levels.

## Key Features

- **Multiple Action Types**: Projectile, Target, TargetSelf, Area, DirectionalCamera
- **Resource Costs**: Mana and Stamina costs
- **Cooldown System**: Per-action cooldown management
- **Cast Time**: Pre-cast and cast time mechanics
- **Dice System**: D&D-style dice notation for damage (e.g., "2D6")
- **Skill Scaling**: Damage scales with skill level
- **Weapon Amplification**: Some actions benefit from specific weapon types
- **Dynamic Loading**: Actions loaded from YAML configuration
- **Lifecycle Hooks**: OnEquip, OnUnequip, effect hooks
- **Area Effects**: Radius-based area targeting
- **Projectile System**: Projectile-based actions
- **Healing Actions**: Specialized healing action base class
- **Summon Actions**: Specialized summon action base class

## Action Types

### Action Type Enum

```typescript
enum ActionType {
    None,              // No specific type
    Projectile,        // Projectile-based action
    Target,            // Target entity action
    TargetSelf,        // Self-targeted action
    Area,              // Area-of-effect action
    DirectionalCamera  // Directional action based on camera
}
```

### Action Target Kind (C#)

```csharp
enum ActionTargetKind {
    None,
    Self,
    Target,
    Area,
    Projectile,
    Directional
}
```

## Base Action Structure

### BaseAction Class

```typescript
abstract class BaseAction {
    // Required properties
    abstract id: number;
    abstract name: string;
    abstract namespace: string;
    abstract costType: ActionCostType;
    abstract cost: number;
    
    // Action properties
    type: ActionType;
    damage?: Dices;
    damageType?: DamageType;
    skill?: SkillName;
    skillRequeriment?: number;
    damagePerLevel?: Map<number, Dices>;
    weaponAmplify: WeaponType;
    radius?: number;
    preCastTime: number;
    castSay: string;
    
    // Lifecycle methods
    precast(owner: Entity): void;
    exec(owner: Entity, target: Entity | Vector3): void;
    effectOnCast(owner: Entity): void;
    removeEffect(owner: Entity): void;
    effectOnTakeDamage(owner: Entity, causer: Entity, damage: number, damageType: DamageType): number;
    effectOnHit(owner: Entity, causer: Entity, damage: number, damageType: DamageType): number;
}
```

### Action Properties (C#)

```csharp
abstract class Action {
    // Costs
    int ManaCost;
    int StaminaCost;
    
    // Timing
    float CastTime;
    float Cooldown;
    TimeSpan CooldownTimeout;
    float CreateTime;
    
    // Requirements
    SkillName Skill;
    int MinSkill;
    int MaxSkillGain;
    
    // Damage
    float FactorDamage;
    
    // Range
    float MaxRange;
    float MinRange;
    float Radius;
    
    // Flags
    bool IsSpell;
    bool UseMounted;
    bool CanExecute;
    bool Interruptible;
    bool EnterWarmode;
    
    // Owner
    CreatureEntity Owner;
}
```

## Action Registration

### Action Registry

**TypeScript:**
```typescript
class Actions {
    public static actionsIndex: BaseAction[] = [];
    public static actionsByName: Map<string, BaseAction> = new Map();
    
    public static addAction(action: BaseAction): void {
        Actions.actionsIndex[action.id] = action;
        Actions.actionsByName.set(action.name, action);
        Actions.actionsByName.set(action.namespace, action);
        Actions.actionsByName.set(action.namespace.toLowerCase(), action);
    }
    
    public static findActionById(id: number): BaseAction | null {
        return Actions.actionsIndex[id] || null;
    }
    
    public static findActionByName(name: string): BaseAction | null {
        return Actions.actionsByName.has(name.toLowerCase()) 
            ? Actions.actionsByName.get(name.toLowerCase()) 
            : null;
    }
}
```

**C# (Auto-registration):**
```csharp
static Action() {
    foreach (Type t in Namespace.GetByNamespace("GameServer.Actions")) {
        if (Activator.CreateInstance(t) is Action action) {
            Actions.Add(action.Name, action);
            action.InitializePrefab();
        }
    }
}
```

## Action Instance Creation

### Create Instance

**TypeScript:**
- Actions registered directly
- Instances created from registered actions

**C#:**
```csharp
public static Action CreateInstance(CreatureEntity owner, String name) {
    Action prefab;
    
    if (Actions.TryGetValue(name, out prefab)) {
        Action skill = prefab.Instantiate(owner);
        
        // Copy properties from prefab
        skill.ManaCost = prefab.ManaCost;
        skill.Cooldown = prefab.Cooldown;
        skill.CastTime = prefab.CastTime;
        skill.CreateTime = prefab.CreateTime;
        skill.ActionId = prefab.ActionId;
        skill.StaminaCost = prefab.StaminaCost;
        skill.MinSkill = prefab.MinSkill;
        skill.MaxSkillGain = prefab.MaxSkillGain;
        skill.FactorDamage = prefab.FactorDamage;
        
        return skill;
    }
    
    return null;
}
```

## Resource Costs

### Cost Types

**ActionCostType Enum:**
```typescript
enum ActionCostType {
    None,      // No cost
    Mana,      // Mana cost
    Stamina,   // Stamina cost
    Health,    // Health cost (rare)
    Gold       // Gold cost (rare)
}
```

### Cost Management

**Cost Validation:**
- Check if player has enough resource
- Deduct cost on cast
- Fail if insufficient resources

**Default Costs:**
- ManaCost: 0 (default)
- StaminaCost: 0 (default)
- Can be overridden per action

## Cooldown System

### Cooldown Management

**Cooldown Properties:**
- `Cooldown`: Base cooldown duration (seconds)
- `CooldownTimeout`: Timestamp when cooldown expires
- `OnCooldown`: Boolean check if action is on cooldown
- `CooldownLeft`: Remaining cooldown time

**Cooldown Check:**
```csharp
public bool OnCooldown {
    get {
        return CooldownTimeout > Scene.CurrentTime;
    }
}

public float CooldownLeft {
    get {
        return (float)(CooldownTimeout - Scene.CurrentTime).TotalSeconds;
    }
}
```

**Cooldown Application:**
- Set `CooldownTimeout` when action is executed
- Action cannot be used while on cooldown
- Cooldown displayed to player

## Cast Time System

### Cast Time Properties

**Cast Time Components:**
- `CastTime`: Time to cast action (seconds)
- `PreCastTime`: Time before cast starts (seconds)
- `CreateTime`: Time to create effect (seconds, default 0.4s)

**Cast Process:**
1. Pre-cast phase (preCastTime)
2. Cast phase (castTime)
3. Create phase (createTime)
4. Execution phase

**Interruptible:**
- Some actions can be interrupted during cast
- Movement or damage can interrupt
- `Interruptible` property controls this

## Dice System

### Dice Notation

**Format:**
- Format: `"XdY"` where X = number of dice, Y = sides per die
- Examples: `"1D6"`, `"2D10"`, `"3D4"`
- `Dices.None` = no dice roll

**Dice Rolling:**
```typescript
public rollDice(dice: Dices): number {
    if (dice === Dices.None) return 0;
    
    const match = dice.match(/(\d+)D(\d+)/);
    if (!match) return 0;
    
    const numDices = parseInt(match[1], 10);
    const numSides = parseInt(match[2], 10);
    
    let total = 0;
    for (let i = 0; i < numDices; i++) {
        total += Math.floor(Math.random() * numSides) + 1;
    }
    
    return total;
}
```

**Examples:**
- `"1D6"`: Roll 1 die with 6 sides (1-6)
- `"2D10"`: Roll 2 dice with 10 sides each (2-20)
- `"3D4"`: Roll 3 dice with 4 sides each (3-12)

## Damage Calculation

### Effect Value Calculation

**TypeScript Implementation:**
```typescript
public getEffectValue(owner: Entity): number {
    // Get modifiers (skill bonus + weapon bonus)
    const mods = this.getMods(owner, this.skill, this.weaponAmplify);
    
    // Get skill level
    const skillValue = owner.getSkillValue(this.skill);
    
    // Get dice for skill level
    let effectDice = this.damage;
    if (this.damagePerLevel.size > 0) {
        effectDice = this.damagePerLevel[skillValue] || this.damage;
    }
    
    // Roll dice and add modifiers
    let effect = this.rollDice(effectDice) + mods;
    
    return effect;
}
```

### Damage Calculation (C#)

**Complex Damage Formula:**
```csharp
public int CalculateDamage(SkillName skill, int stat, float multiplyFactor = 1, bool spell = false) {
    int baseDamage = 0;
    int totalDamage = 0;
    
    // Apply FactorDamage if > 1
    if (FactorDamage > 1) {
        multiplyFactor = FactorDamage;
    }
    
    if (multiplyFactor < 1) {
        multiplyFactor = 1;
    }
    
    // Calculate base damage
    if (spell) {
        // Spell damage: 70% of weapon damage (or 50% if not staff)
        baseDamage = (player.CheckWeaponType(WeaponType.Staff) 
            ? player.GetWeaponDamage() 
            : (int)(player.GetWeaponDamage() / 2));
        baseDamage = (int)(baseDamage * 0.7);
    } else {
        // Physical damage: 180% of weapon damage
        baseDamage = player.GetWeaponDamage();
        baseDamage = (int)(baseDamage * 1.8);
        
        // Two-handed weapons consume stamina
        if (player.Equipments.Weapon?.Type == WeaponType.TwoHandedSword ||
            player.Equipments.Weapon?.Type == WeaponType.TwoHandedAxe ||
            player.Equipments.Weapon?.Type == WeaponType.TwoHandedMace) {
            if (player.Stamina > 1) {
                player.AddStamina(-5);
            }
        }
    }
    
    if (baseDamage <= 0) {
        baseDamage = 1;
    }
    
    // Calculate skill/stat factor
    int factorSkillStat = 0;
    if (player.InEvent && player.Scene.Name == "BattleCastle") {
        factorSkillStat = 300; // Event bonus
    } else {
        factorSkillStat = (100 + Owner.GetSkillLevel(skill) + stat);
    }
    
    // Final damage calculation
    totalDamage = (int)(((baseDamage * factorSkillStat) / 100) * multiplyFactor);
    
    if (totalDamage < 1) {
        totalDamage = 1;
    }
    
    return totalDamage;
}
```

### Modifiers

**Skill and Weapon Modifiers:**
```typescript
public getMods(owner: Entity, skill: SkillName, weaponAmplify: WeaponType = WeaponType.None): number {
    // Skill modifier
    const skillMod = owner.getSkillBonus(skill);
    
    // Weapon modifier (if weapon type matches)
    const weaponMod = (owner.getWeaponType() === weaponAmplify) 
        ? owner.getWeaponBaseDamage() 
        : 0;
    
    return skillMod + weaponMod;
}
```

## Skill Requirements

### Skill Requirements

**Properties:**
- `skill`: Skill name required
- `skillRequeriment` / `MinSkill`: Minimum skill level required
- `MaxSkillGain`: Maximum skill level that can gain experience

**Requirement Check:**
```typescript
public canExecute(player: Player): boolean {
    if (this.skill === SkillName.None) {
        return true; // No skill requirement
    }
    
    const skillLevel = player.getSkillValue(this.skill);
    return skillLevel >= this.skillRequeriment;
}
```

**Execution Failed Feedback:**
```csharp
public virtual string GetExecutionFailedFeedback(CharacterEntity entity) {
    if (MinSkill > 0 && Skill != SkillName.Count) {
        return $"<color=red>You need a Staff and {MinSkill} of {Skill} to use this ability</color>";
    } else {
        return "<color=red>You don't meet the requirements</color>.";
    }
}
```

## Skill Experience Gain

### Experience System

**Gain Conditions:**
- Action must have associated skill
- Player's skill level must be below `MaxSkillGain`
- Action must be executed successfully

**Implementation:**
```typescript
public gainSkillExperience(owner: Entity): void {
    if (owner instanceof Player) {
        (owner as Player).gainSkillExperience(this.skill);
    }
}
```

**C# Implementation:**
```csharp
public void AddSkillExperience() {
    if (Owner is CharacterEntity player) {
        if (player.GetSkillLevel(Skill) < MaxSkillGain) {
            player.AddSkillExperience(Skill);
        }
    }
}
```

## Damage Per Level

### Scaling System

**Damage Scaling:**
- Actions can have different damage dice per skill level
- `damagePerLevel`: Map of skill level to dice
- Falls back to base damage if level not found

**Example:**
```typescript
damagePerLevel = new Map([
    [1, Dices.parse("1D6")],   // Level 1: 1D6
    [5, Dices.parse("2D6")],  // Level 5: 2D6
    [10, Dices.parse("3D6")]  // Level 10: 3D6
]);
```

**Usage:**
```typescript
const skillValue = owner.getSkillValue(this.skill);
let effectDice = this.damagePerLevel.get(skillValue) || this.damage;
```

## Weapon Amplification

### Weapon Type Bonus

**Concept:**
- Some actions benefit from specific weapon types
- Weapon damage added to effect value
- Only applies if equipped weapon matches

**Implementation:**
```typescript
public weaponAmplify: WeaponType = WeaponType.None;

public getMods(owner: Entity, skill: SkillName, weaponAmplify: WeaponType): number {
    const weaponMod = (owner.getWeaponType() === weaponAmplify) 
        ? owner.getWeaponBaseDamage() 
        : 0;
    return weaponMod;
}
```

**Examples:**
- Sword skills benefit from swords
- Staff skills benefit from staffs
- Unarmed skills don't benefit from weapons

## Range System

### Range Properties

**Range Components:**
- `MaxRange`: Maximum range for action
- `MinRange`: Minimum range (default 0)
- `UseableRange`: Usable range (default MaxRange)
- `Radius`: Area effect radius

**Range Validation:**
```csharp
public bool IsInRange(Vector2 target) {
    float distance = Vector2.Distance(Owner.Position, target);
    return distance >= MinRange && distance <= MaxRange;
}
```

## Area Effects

### Area Targeting

**Get Enemies in Radius:**
```typescript
public getEnemiesInRadius(caster: Entity, position: Vector3, radius: number): Array<Entity> {
    let enemies = new Array<Entity>();
    
    caster.areaOfInterece.forEach((entity) => {
        if (
            entity.team.IsEnemyOf(caster.team) && 
            position.distanceBetween(entity.transform.position) <= radius &&
            !entity.isDead
        ) {
            enemies.push(entity);
        }
    });
    
    return enemies;
}
```

**Get Allies in Radius:**
```typescript
public getAllyInRadius(
    caster: Entity, 
    position: Vector3, 
    radius: number, 
    includeSelf: boolean = false
): Array<Entity> {
    let allys = new Array<Entity>();
    
    caster.areaOfInterece.forEach((entity) => {
        if (
            entity.team.IsAllyOf(caster.team) && 
            position.distanceBetween(entity.transform.position) <= radius &&
            !entity.isDead
        ) {
            allys.push(entity);
        }
    });
    
    if (includeSelf) {
        if (
            position.distanceBetween(caster.transform.position) <= radius &&
            !caster.isDead
        ) {
            allys.push(caster);
        }
    }
    
    return allys;
}
```

## Action Execution

### Execute Methods

**TypeScript:**
```typescript
public exec(owner: Entity, target: Entity | Vector3): void {
    this.gainSkillExperience(owner);
    // Action-specific logic in subclasses
}
```

**C#:**
```csharp
public virtual Task Execute(Vector2 target, CancellationToken cancellationToken = default) {
    throw new NotSupportedException();
}

public virtual Task Execute(CreatureEntity target, CancellationToken cancellationToken = default) {
    throw new NotSupportedException();
}
```

### Execution Flow

**Complete Flow:**
1. **Pre-validation**: Check requirements, costs, cooldown
2. **Pre-cast**: Pre-cast phase (preCastTime)
3. **Cast**: Cast phase (castTime)
4. **Create**: Create phase (createTime)
5. **Execute**: Action execution
6. **Post-execution**: Apply cooldown, consume costs, gain experience

## Lifecycle Hooks

### Action Hooks

**Pre-cast Hook:**
```typescript
public precast(owner: Entity): void {
    if (this.castSay) {
        owner.say(this.castSay, this.colorByDamageType());
    }
}
```

**On Cast Hook:**
```typescript
public effectOnCast(owner: Entity): void {
    // Apply effects when action is cast
    // Example: Buffs, visual effects, etc.
}
```

**Remove Effect Hook:**
```typescript
public removeEffect(owner: Entity): void {
    // Remove effects when action ends
    // Example: Remove buffs, cleanup
}
```

**On Take Damage Hook:**
```typescript
public effectOnTakeDamage(
    owner: Entity, 
    causer: Entity, 
    damage: number, 
    damageType: DamageType
): number {
    // Modify incoming damage
    // Return modified damage
    return damage;
}
```

**On Hit Hook:**
```typescript
public effectOnHit(
    owner: Entity, 
    causer: Entity, 
    damage: number, 
    damageType: DamageType
): number {
    // Modify outgoing damage
    // Return modified damage
    return damage;
}
```

**On Equip/Unequip (C#):**
```csharp
public virtual void OnEquip() {
    // Called when action is equipped
}

public virtual void OnUnequip() {
    // Called when action is unequipped
}
```

## Specialized Action Types

### BaseHealAction

**Healing Actions:**
```typescript
abstract class BaseHealAction extends BaseAction {
    public exec(owner: Entity, target: Entity): void {
        if (target) {
            this.gainSkillExperience(owner);
            const modInt = owner.bonusDamageMod(StatusType.Int);
            const healAmount = Math.round(this.getEffectValue(owner) + modInt);
            target.heal(owner, healAmount);
        }
    }
}
```

**Healing Features:**
- Uses Intelligence modifier
- Effect value is healing amount
- Can target self or allies
- Gains skill experience

### BaseSummonAction

**Summon Actions:**
```typescript
abstract class BaseSummonAction extends BaseAction {
    public lifetime: number = 60; // Seconds
    
    public createSummon(owner: Player, creature: Summon, position: Vector3): void {
        creature.transform.position = position;
        creature.startLifeTime(this.lifetime);
        owner.map.joinMap(creature);
    }
}
```

**Summon Features:**
- Creates summon creature
- Summon has lifetime
- Summon added to map
- Position set on creation

## Projectile System

### Projectile Actions

**Projectile Properties:**
- Type: `ActionType.Projectile`
- Projectile travels to target
- Can collide with entities
- Collision handling

**On Projectile Collision:**
```csharp
public virtual void OnProjectileCollidesWithCreature(CreatureEntity other) {
    // Handle projectile collision
    // Apply damage, effects, etc.
}
```

## Action Interval Execution

### Interval-Based Actions

**Execute Action at Intervals:**
```typescript
public execActionInterval(ticks: number, delay: number, cb: Function): Promise<void> {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < ticks; i++) {
            setTimeout((_this) => cb.call(_this), delay * i, this);
        }
        setTimeout(resolve, ticks * delay);
    });
}
```

**Use Cases:**
- Damage over time effects
- Channeled abilities
- Repeated effects
- Tick-based damage

## Damage Types

### Damage Type Enum

**Damage Types:**
- `Physic`: Physical damage
- `Fire`: Fire damage
- `Cold`: Cold/Ice damage
- `Poison`: Poison damage
- `Energy`: Energy/Magic damage
- `Light`: Light/Holy damage
- `Dark`: Dark/Shadow damage

### Color by Damage Type

**Visual Feedback:**
```typescript
public colorByDamageType(): string {
    switch (this.damageType) {
        case DamageType.Fire: return "255,42,29,255";      // Red
        case DamageType.Cold: return "67,87,255,255";     // Blue
        case DamageType.Poison: return "84,255,76,255";   // Green
        case DamageType.Energy: return "82,247,255,255";   // Cyan
        case DamageType.Light: return "255,239,124,255";  // Yellow
        case DamageType.Dark: return "57,12,88,255";       // Purple
        default: return "255,255,255,255";                // White
    }
}
```

## Dynamic Configuration Loading

### YAML Configuration

**Loading from YAML:**
```csharp
public static async void Reload() {
    string yml = HttpGetter.GetWithRetry(Config.Api + "/gamedata/abilities/public.yml");
    
    if (yml != "") {
        var deserializer = new DeserializerBuilder()
            .IgnoreUnmatchedProperties()
            .Build();
        
        Dictionary<string, object>[] actions = deserializer
            .Deserialize<Dictionary<string, object>[]>(reader);
        
        foreach (Dictionary<string, object> properties in actions) {
            if (Actions.TryGetValue((string)properties["name"], out Action actionPrefab)) {
                // Update properties from YAML
                if (properties.TryGetValue("cooldown", out object result)) {
                    actionPrefab.Cooldown = float.Parse((string)result);
                }
                if (properties.TryGetValue("manacost", out object result)) {
                    actionPrefab.ManaCost = int.Parse((string)result);
                }
                if (properties.TryGetValue("stamcost", out object result)) {
                    actionPrefab.StaminaCost = int.Parse((string)result);
                }
                // ... more properties
            }
        }
    }
}
```

**YAML Format:**
```yaml
- name: "Fireball"
  cooldown: 5.0
  manacost: 20
  stamcost: 0
  reqskill: 5
  maxskillgain: 10
  factordamage: 1.2
```

## Actionbar Integration

### IActionbarObject Interface

**Actionbar Support:**
```csharp
void IActionbarObject.Send(PlayerController pc, uint index) {
    // Send action to actionbar
    buffer.Put(ServerPacketType.SetActionbarAction);
    buffer.PutVar(index);
    buffer.PutSymbol(Name);
    
    // Send cooldown if on cooldown
    if (OnCooldown) {
        buffer.Put(ServerPacketType.ActionCasted);
        buffer.PutSymbol(Name);
        buffer.Put(CooldownLeft);
    }
}
```

## Action Flags

### Action Properties

**IsSpell:**
- Indicates if action is a spell
- Affects damage calculation
- May require staff

**UseMounted:**
- Can be used while mounted
- Default: false

**CanExecute:**
- Whether action can be executed
- Can be overridden for conditions
- Default: true

**Interruptible:**
- Whether cast can be interrupted
- Movement or damage can interrupt
- Default: false

**EnterWarmode:**
- Whether action enters warmode
- Default: true

## Factor Damage

### Damage Multiplier

**FactorDamage:**
- Multiplier for damage calculation
- Applied to final damage
- Default: 1.0
- Can be > 1 for bonus damage

**Usage:**
```csharp
if (FactorDamage > 1) {
    multiplyFactor = FactorDamage;
}
totalDamage = (int)(((baseDamage * factorSkillStat) / 100) * multiplyFactor);
```

## Action Execution Validation

### Pre-Execution Checks

**Validation Steps:**
1. Check if action exists
2. Check if owner is valid
3. Check if on cooldown
4. Check resource costs (mana/stamina)
5. Check skill requirements
6. Check range (if targeted)
7. Check if can execute

**Implementation:**
```typescript
public canExecute(player: Player, target?: Entity | Vector3): boolean {
    // Check cooldown
    if (this.isOnCooldown(player)) {
        return false;
    }
    
    // Check costs
    if (this.costType === ActionCostType.Mana && player.mana < this.cost) {
        return false;
    }
    if (this.costType === ActionCostType.Stamina && player.stamina < this.cost) {
        return false;
    }
    
    // Check skill requirement
    if (this.skill !== SkillName.None) {
        const skillLevel = player.getSkillValue(this.skill);
        if (skillLevel < this.skillRequeriment) {
            return false;
        }
    }
    
    // Check range (if targeted)
    if (target instanceof Entity) {
        const distance = player.transform.position.distanceTo(target.transform.position);
        if (distance > this.maxRange || distance < this.minRange) {
            return false;
        }
    }
    
    return true;
}
```

## Action Examples

### Simple Damage Action

```typescript
class FireballAction extends BaseAction {
    id = 1;
    name = "Fireball";
    namespace = "fireball";
    costType = ActionCostType.Mana;
    cost = 20;
    
    type = ActionType.Projectile;
    damage = Dices.parse("3D6");
    damageType = DamageType.Fire;
    skill = SkillName.Magic;
    skillRequeriment = 5;
    radius = 3.0;
    maxRange = 20.0;
    preCastTime = 1.0;
    castSay = "Fireball!";
    
    public exec(owner: Entity, target: Entity | Vector3): void {
        super.exec(owner, target);
        
        if (target instanceof Vector3) {
            const enemies = this.getEnemiesInRadius(owner, target, this.radius);
            const damage = this.getEffectValue(owner);
            
            enemies.forEach(enemy => {
                enemy.takeDamage(owner, damage, this.damageType);
            });
        }
    }
}
```

### Healing Action

```typescript
class HealAction extends BaseHealAction {
    id = 2;
    name = "Heal";
    namespace = "heal";
    costType = ActionCostType.Mana;
    cost = 15;
    
    type = ActionType.Target;
    damage = Dices.parse("2D8");
    skill = SkillName.Healing;
    skillRequeriment = 1;
    maxRange = 15.0;
    preCastTime = 1.5;
    castSay = "Heal!";
    
    // exec() inherited from BaseHealAction
}
```

### Summon Action

```typescript
class SummonSkeletonAction extends BaseSummonAction {
    id = 3;
    name = "Summon Skeleton";
    namespace = "summon_skeleton";
    costType = ActionCostType.Mana;
    cost = 30;
    
    type = ActionType.TargetSelf;
    skill = SkillName.Necromancy;
    skillRequeriment = 10;
    lifetime = 120; // 2 minutes
    
    public exec(owner: Entity, target: Entity | Vector3): void {
        super.exec(owner, target);
        
        if (owner instanceof Player) {
            const position = owner.transform.position;
            const skeleton = new SkeletonSummon();
            this.createSummon(owner, skeleton, position);
        }
    }
}
```

## Performance Considerations

### Optimization Strategies

**1. Action Caching:**
- Actions cached in registry
- Fast lookup by ID or name
- No repeated instantiation

**2. Damage Calculation Caching:**
- Cache skill values
- Cache modifiers
- Reduce redundant calculations

**3. Area Query Optimization:**
- Use spatial indexing for area queries
- Cache area of interest
- Limit radius checks

**4. Cooldown Management:**
- Efficient cooldown checking
- Batch cooldown updates
- Minimal timestamp comparisons

## Integration with Other Systems

### Combat System
- Actions trigger combat
- Damage calculation
- Status effects application

### Skill System
- Actions gain skill experience
- Skill requirements
- Skill-based scaling

### Equipment System
- Weapon amplification
- Weapon type checks
- Equipment bonuses

### Status Effect System
- Actions apply status effects
- Effect hooks for modifications
- Buff/debuff integration

### Animation System
- ActionId for animations
- Cast animations
- Effect animations

## Testing Considerations

### Unit Tests
- Action registration
- Damage calculation
- Cooldown management
- Cost validation
- Skill requirement checks

### Integration Tests
- Full action execution flow
- Area effect targeting
- Projectile collision
- Healing application
- Summon creation

### Balance Tests
- Damage scaling across levels
- Resource cost balance
- Cooldown balance
- Range balance
- Skill requirement balance

## Summary

The actions system provides:

- **Modular Design**: Extensible action system with base classes
- **Multiple Types**: Projectile, Target, Area, Self, Directional
- **Resource Management**: Mana and Stamina costs
- **Cooldown System**: Per-action cooldown management
- **Skill Integration**: Requirements and experience gain
- **Damage Scaling**: Dice-based damage with skill scaling
- **Weapon Amplification**: Weapon type bonuses
- **Area Effects**: Radius-based targeting
- **Lifecycle Hooks**: Pre-cast, on-cast, effect hooks
- **Dynamic Loading**: YAML configuration support
- **Specialized Types**: Healing and Summon actions

The system ensures flexible, balanced, and extensible ability system for combat and gameplay.

