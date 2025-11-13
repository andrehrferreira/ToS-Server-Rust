# Teams System Documentation

## Overview

The Teams system manages relationships between entities, determining whether entities are allies or enemies, and whether they can attack each other. This system is crucial for implementing PvP (Player vs Player), PvE (Player vs Environment), and team-based gameplay mechanics.

## Architecture

### Team Types

Teams are categorized into several types based on their nature:

**Entity Teams:**
- `None`: No team (neutral entities)
- `Monsters`: Hostile creatures
- `NPCs`: Non-player characters (friendly)
- `Players`: Player-controlled entities
- `Tower`: Defensive structures
- `Guards`: Guard entities (protect players)
- `Provocation`: Provoked entities (attack specific targets)
- `Team1`: Team 1 (for team-based gameplay)
- `Team2`: Team 2 (for team-based gameplay)
- `Pet`: Pet entities (follow owner's team)

### Team Structure

**Core Properties:**
- `Kind`: Team type (Monsters, Players, etc.)
- `Self`: Entity that owns the team
- `TeamOwner`: Owner of the team (for parties/guilds)

**Relationships:**
- `IsAllyOf(other)`: Check if another team is an ally
- `IsEnemyOf(other)`: Check if another team is an enemy
- `CanPartyWith(other)`: Check if can form a party with another team
- `InPartyWith(player)`: Check if in a party with a player

## Team Types

### Version 1: C# Server3 (TeamKind)

```csharp
public enum TeamKind
{
    None,
    Monsters,
    NPCs,
    Players,
    Tower,
    Guards,
    Provocation,
    Team1,
    Team2,
    Pet
}
```

### Version 2: TypeScript (TeamKind)

```typescript
export enum TeamKind {
    None,
    Monsters,
    NPCs,
    Players,
    Tower,
    Guards,
    Provocation,
    Team1,
    Team2,
    Pet
}
```

## Team Relationships

### Ally Relationships

**Monsters:**
- **Allies**: Other monsters
- **Enemies**: Players (if not invulnerable), Guards, Provocation

**NPCs:**
- **Allies**: All entities (friendly to everyone)
- **Enemies**: None (neutral)

**Players:**
- **Allies**: Self, same team owner (party/guild)
- **Enemies**: Monsters, Provocation, other players (under certain conditions), Guards (if negative karma)

**Tower:**
- **Allies**: Other towers, Players
- **Enemies**: All entities (defensive structure)

**Guards:**
- **Allies**: Players (if not negative karma), NPCs, other guards
- **Enemies**: Players (if negative karma), Monsters

**Provocation:**
- **Allies**: None (hostile to everyone)
- **Enemies**: All entities

**Team1:**
- **Allies**: Other Team1 entities
- **Enemies**: Team2 entities (if not invulnerable)

**Team2:**
- **Allies**: Other Team2 entities
- **Enemies**: Team1 entities (if not invulnerable)

**Pet:**
- **Allies**: Owner's team, same team owner
- **Enemies**: Owner's enemies

### Enemy Relationships

**Monsters:**
- **Enemies**: Players (if not invulnerable), Guards, Provocation
- **Conditions**: Check `Invulnerable` flag

**Players:**
- **Enemies**: 
  - None (neutral entities)
  - Monsters
  - Provocation
  - Other players (if duel zone enemy or player enemy)
  - Guards (if negative karma)
- **Conditions**: 
  - Duel zone: Both in duel zone, not invulnerable, different team owner
  - Player enemy: Not invulnerable, not in safe zone, different team owner

**Tower:**
- **Enemies**: All entities (defensive structure)

**Guards:**
- **Enemies**: Players (if negative karma), Monsters
- **Conditions**: Check `NegativeKarma` flag

**Provocation:**
- **Enemies**: All entities (None, Monsters, Provocation, Players)

**Team1:**
- **Enemies**: Team2 entities (if not invulnerable)
- **Conditions**: Check `Invulnerable` flag

**Team2:**
- **Enemies**: Team1 entities (if not invulnerable)
- **Conditions**: Check `Invulnerable` flag

## Team Implementation

### Team Class

**TypeScript Implementation:**
```typescript
export class Team {
    public kind: TeamKind;
    public self: Entity;

    constructor(kind: TeamKind, self: Entity) {
        this.kind = kind;
        this.self = self;
    }

    public IsEnemyOf(other: Team): boolean {
        switch(this.kind) {
            case TeamKind.Monsters: 
                return ((other.kind === TeamKind.Players) && 
                       other.self.states.dontHasFlag(EntityStates.Invulnerable)) || 
                       other.kind === TeamKind.Guards || 
                       other.kind === TeamKind.Provocation;
            
            case TeamKind.Players: 
                return other.kind === TeamKind.None || 
                       other.kind === TeamKind.Monsters || 
                       other.kind === TeamKind.Provocation || 
                       (other.kind === TeamKind.Players && 
                        (other.self != this.self && 
                         (this.isDuelZoneEnemy(other) || this.isPlayerEnemy(other)))) || 
                       other.kind === TeamKind.Guards && 
                       this.self.states.hasFlag(EntityStates.NegativeKarma);
            
            case TeamKind.Tower: 
                return true;
            
            case TeamKind.Guards: 
                return ((other.kind === TeamKind.Players) && 
                       (other.self.states.hasFlag(EntityStates.NegativeKarma) ?? false)) || 
                       other.kind === TeamKind.Monsters;
            
            case TeamKind.Provocation: 
                return other.kind === TeamKind.None || 
                       other.kind === TeamKind.Monsters || 
                       other.kind === TeamKind.Provocation || 
                       other.kind === TeamKind.Players;
            
            case TeamKind.Team1: 
                return other.kind == TeamKind.Team2 && 
                       other.self.states.dontHasFlag(EntityStates.Invulnerable);
            
            case TeamKind.Team2: 
                return other.kind == TeamKind.Team1 && 
                       other.self.states.dontHasFlag(EntityStates.Invulnerable);
            
            default: 
                return false;
        }
    }

    public IsAllyOf(other: Team): boolean {
        switch(this.kind) {
            case TeamKind.Monsters: 
                return other.kind === TeamKind.Monsters;
            
            case TeamKind.NPCs: 
                return true;
            
            case TeamKind.Players: 
                return other.self.id === this.self.id || 
                       (other.self.teamOwner === this.self.teamOwner && 
                        other.self.teamOwner !== null && 
                        this.self.teamOwner !== null);
            
            case TeamKind.Guards: 
                return (other.kind !== TeamKind.Players) || 
                       other.self.states.dontHasFlag(EntityStates.NegativeKarma);
            
            case TeamKind.Provocation: 
                return false;
            
            case TeamKind.Tower: 
                return other.kind == TeamKind.Tower || 
                       other.kind == TeamKind.Players;
            
            case TeamKind.Team1: 
                return other.kind == TeamKind.Team1;
            
            case TeamKind.Team2: 
                return other.kind == TeamKind.Team2;
            
            case TeamKind.Pet: 
                return this.self.teamOwner === other.self.teamOwner || 
                       (this.self as Pet).owner?.characterId == other.self.characterId;
            
            default: 
                return false;
        }
    }

    public isDuelZoneEnemy(pt: Team): boolean {
        return this.self.states.hasFlag(EntityStates.DuelZone) && 
               pt.self.states.hasFlag(EntityStates.DuelZone) && 
               pt.self.states.dontHasFlag(EntityStates.Invulnerable) &&
               this.self.teamOwner !== pt.self.teamOwner;
    }

    public isPlayerEnemy(pt: Team): boolean {
        return pt.self.states.dontHasFlag(EntityStates.Invulnerable) &&
               pt.self.states.dontHasFlag(EntityStates.SafeZone) &&
               this.self.states.dontHasFlag(EntityStates.SafeZone) &&
               this.self.teamOwner !== pt.self.teamOwner;
    }
}
```

**C# Implementation (Version 1):**
```csharp
public struct Team
{
    public TeamKind Kind;
    public CharacterEntity Self;
    public CreatureEntity SelfCreature;

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public bool IsAllyOf(Team other)
    {
        switch (Kind)
        {
            case TeamKind.Monsters:
                return other.Kind == TeamKind.Monsters;
            case TeamKind.NPCs:
                return true;
            case TeamKind.Players:
                return other.Self == Self || (other.Self?.TeamOwner == Self.TeamOwner);
            case TeamKind.Guards:
                return other.Kind != TeamKind.Players || 
                       other.Self.Flags.DontHasFlag(CreatureFlags.NegativeKarma);
            case TeamKind.Provocation:
                return false;
            case TeamKind.Tower:
                return other.Kind == TeamKind.Tower || other.Kind == TeamKind.Players;
            case TeamKind.Team1:
                return other.Kind == TeamKind.Team1;
            case TeamKind.Team2:
                return other.Kind == TeamKind.Team2;
            default:
                return false;
        }
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    private bool IsDuelZoneEnemy(Team pt)
    {
        return (Self.Flags & CreatureFlags.DuelZone) == CreatureFlags.DuelZone && 
               (pt.Self.Flags & CreatureFlags.DuelZone) == CreatureFlags.DuelZone && 
               (pt.Self.Flags & CreatureFlags.Invulnerable) != CreatureFlags.Invulnerable && 
               (pt.Self.TeamOwner != Self.TeamOwner);
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    private bool IsPlayerEnemy(Team pt)
    {
        return ((pt.Self.Flags & CreatureFlags.Invulnerable) != CreatureFlags.Invulnerable && 
                (Self.TeamOwner != pt.Self.TeamOwner)
                && (Self.IsPlayerKillOn || pt.Self.IsPK)
                && (((Self.Flags & CreatureFlags.SafeZone) != CreatureFlags.SafeZone && 
                     (pt.Self.Flags & CreatureFlags.SafeZone) != CreatureFlags.SafeZone)));
    }

    public bool IsEnemyOf(Team other)
    {
        switch (Kind)
        {
            case TeamKind.Monsters:
                return ((other.Kind == TeamKind.Players) && 
                       (other.Self.Flags & CreatureFlags.Invulnerable) != CreatureFlags.Invulnerable) || 
                       other.Kind == TeamKind.Guards || 
                       other.Kind == TeamKind.Provocation;
            case TeamKind.Players:
                return other.Kind == TeamKind.None || 
                       other.Kind == TeamKind.Monsters || 
                       other.Kind == TeamKind.Provocation || 
                       (other.Kind == TeamKind.Players && 
                        (other.Self != Self && 
                         (IsDuelZoneEnemy(other) || IsPlayerEnemy(other)))) || 
                       other.Kind == TeamKind.Guards && 
                       Self.Flags.HasFlag(CreatureFlags.NegativeKarma);
            case TeamKind.Tower:
                return true;
            case TeamKind.Guards:
                return ((other.Kind == TeamKind.Players) && 
                       (other.Self?.Flags.HasFlag(CreatureFlags.NegativeKarma) ?? false)) || 
                       other.Kind == TeamKind.Monsters;
            case TeamKind.Provocation:
                return other.Kind == TeamKind.None || 
                       other.Kind == TeamKind.Monsters || 
                       other.Kind == TeamKind.Provocation || 
                       other.Kind == TeamKind.Players;
            case TeamKind.Team1:
                return other.Kind == TeamKind.Team2 && 
                       (other.Self?.Flags.HasFlag(CreatureFlags.Invulnerable) ?? false) != CreatureFlags.Invulnerable;
            case TeamKind.Team2:
                return other.Kind == TeamKind.Team1 && 
                       (other.Self?.Flags.HasFlag(CreatureFlags.Invulnerable) ?? false) != CreatureFlags.Invulnerable;
            default:
                return false;
        }
    }
}
```

## Team Logic

### Player vs Player (PvP)

**Duel Zone:**
- **Condition**: Both players in duel zone
- **Check**: `DuelZone` flag on both entities
- **Requirement**: Not invulnerable, different team owner
- **Result**: Can attack each other

**Player Enemy:**
- **Condition**: Player vs Player combat
- **Check**: Not invulnerable, not in safe zone, different team owner
- **Requirement**: War mode enabled, level 50+, player killer status
- **Result**: Can attack each other

**Safe Zone:**
- **Condition**: Entity in safe zone
- **Check**: `SafeZone` flag
- **Result**: Cannot attack or be attacked

### Player vs Environment (PvE)

**Monsters:**
- **Condition**: Player vs Monster
- **Check**: Monster team, player not invulnerable
- **Result**: Can attack each other

**NPCs:**
- **Condition**: Player vs NPC
- **Check**: NPC team (friendly)
- **Result**: Cannot attack (allies)

**Guards:**
- **Condition**: Player vs Guard
- **Check**: Player negative karma
- **Result**: Guards attack players with negative karma

### Team-Based Gameplay

**Team1 vs Team2:**
- **Condition**: Team-based combat
- **Check**: Different team (Team1 vs Team2)
- **Requirement**: Not invulnerable
- **Result**: Can attack each other

**Same Team:**
- **Condition**: Same team
- **Check**: Same team kind and team owner
- **Result**: Cannot attack (allies)

## Team Flags Integration

### Entity States

Teams use entity flags to determine relationships:

**Invulnerable:**
- **Flag**: `EntityStates.Invulnerable`
- **Effect**: Cannot be attacked
- **Usage**: Protected entities, admin mode

**SafeZone:**
- **Flag**: `EntityStates.SafeZone`
- **Effect**: Cannot attack or be attacked
- **Usage**: Safe areas, cities

**DuelZone:**
- **Flag**: `EntityStates.DuelZone`
- **Effect**: Can attack other players in duel zone
- **Usage**: PvP areas, arenas

**NegativeKarma:**
- **Flag**: `EntityStates.NegativeKarma`
- **Effect**: Marked as player killer
- **Usage**: Guards attack, PvP restrictions

**Warmode:**
- **Flag**: `EntityStates.Warmode`
- **Effect**: PvP enabled
- **Usage**: Player vs Player combat

## Serialization

### Network Protocol

**Team Packet Structure:**
```
Team Packet:
- TeamKind: byte (1 byte)
- EntityId: uint32 (4 bytes)
- TeamOwner: uint32 (4 bytes, optional)
```

### Serialization Format

**Write:**
```csharp
buffer.Put((byte)team.Kind);
buffer.Put(team.Self.Id);
buffer.Put(team.Self.TeamOwner ?? 0);
```

**Read:**
```csharp
TeamKind kind = (TeamKind)buffer.GetByte();
uint entityId = buffer.GetUInt();
uint teamOwner = buffer.GetUInt();
```

## Performance Considerations

### Optimization

**Team Lookups:**
- Cache team relationships
- Use efficient data structures (HashMap)
- Optimize team checks
- Batch team updates

**Relationship Checks:**
- Inline team check functions
- Compile-time optimization
- Efficient flag checks
- Cache enemy/ally lists

**Memory Management:**
- Reuse team objects
- Minimize team allocations
- Efficient team storage
- Garbage collection optimization

### Rust Implementation

**Recommended Approach:**
```rust
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum TeamKind {
    None,
    Monsters,
    NPCs,
    Players,
    Tower,
    Guards,
    Provocation,
    Team1,
    Team2,
    Pet,
}

pub struct Team {
    pub kind: TeamKind,
    pub self_id: EntityId,
    pub team_owner: Option<EntityId>,
}

impl Team {
    pub fn new(kind: TeamKind, self_id: EntityId, team_owner: Option<EntityId>) -> Self {
        Self {
            kind,
            self_id,
            team_owner,
        }
    }

    #[inline]
    pub fn is_ally_of(&self, other: &Team, self_entity: &Entity, other_entity: &Entity) -> bool {
        match self.kind {
            TeamKind::Monsters => other.kind == TeamKind::Monsters,
            TeamKind::NPCs => true,
            TeamKind::Players => {
                self.self_id == other.self_id ||
                (self.team_owner == other.team_owner && 
                 self.team_owner.is_some() && 
                 other.team_owner.is_some())
            }
            TeamKind::Guards => {
                other.kind != TeamKind::Players ||
                !other_entity.flags.has_flag(EntityFlag::NegativeKarma)
            }
            TeamKind::Provocation => false,
            TeamKind::Tower => {
                other.kind == TeamKind::Tower || 
                other.kind == TeamKind::Players
            }
            TeamKind::Team1 => other.kind == TeamKind::Team1,
            TeamKind::Team2 => other.kind == TeamKind::Team2,
            TeamKind::Pet => {
                self.team_owner == other.team_owner ||
                // Check pet owner relationship
                false // Implement pet owner check
            }
            _ => false,
        }
    }

    #[inline]
    pub fn is_enemy_of(&self, other: &Team, self_entity: &Entity, other_entity: &Entity) -> bool {
        match self.kind {
            TeamKind::Monsters => {
                (other.kind == TeamKind::Players && 
                 !other_entity.flags.has_flag(EntityFlag::Invulnerable)) ||
                other.kind == TeamKind::Guards ||
                other.kind == TeamKind::Provocation
            }
            TeamKind::Players => {
                other.kind == TeamKind::None ||
                other.kind == TeamKind::Monsters ||
                other.kind == TeamKind::Provocation ||
                (other.kind == TeamKind::Players && 
                 other.self_id != self.self_id && 
                 (self.is_duel_zone_enemy(other, self_entity, other_entity) ||
                  self.is_player_enemy(other, self_entity, other_entity))) ||
                (other.kind == TeamKind::Guards && 
                 self_entity.flags.has_flag(EntityFlag::NegativeKarma))
            }
            TeamKind::Tower => true,
            TeamKind::Guards => {
                (other.kind == TeamKind::Players && 
                 other_entity.flags.has_flag(EntityFlag::NegativeKarma)) ||
                other.kind == TeamKind::Monsters
            }
            TeamKind::Provocation => {
                other.kind == TeamKind::None ||
                other.kind == TeamKind::Monsters ||
                other.kind == TeamKind::Provocation ||
                other.kind == TeamKind::Players
            }
            TeamKind::Team1 => {
                other.kind == TeamKind::Team2 &&
                !other_entity.flags.has_flag(EntityFlag::Invulnerable)
            }
            TeamKind::Team2 => {
                other.kind == TeamKind::Team1 &&
                !other_entity.flags.has_flag(EntityFlag::Invulnerable)
            }
            _ => false,
        }
    }

    #[inline]
    fn is_duel_zone_enemy(&self, other: &Team, self_entity: &Entity, other_entity: &Entity) -> bool {
        self_entity.flags.has_flag(EntityFlag::DuelZone) &&
        other_entity.flags.has_flag(EntityFlag::DuelZone) &&
        !other_entity.flags.has_flag(EntityFlag::Invulnerable) &&
        self.team_owner != other.team_owner
    }

    #[inline]
    fn is_player_enemy(&self, other: &Team, self_entity: &Entity, other_entity: &Entity) -> bool {
        !other_entity.flags.has_flag(EntityFlag::Invulnerable) &&
        !other_entity.flags.has_flag(EntityFlag::SafeZone) &&
        !self_entity.flags.has_flag(EntityFlag::SafeZone) &&
        self.team_owner != other.team_owner
    }
}
```

## Usage Examples

### Creating a Team

```typescript
// TypeScript
const team = new Team(TeamKind.Players, entity);
```

```rust
// Rust
let team = Team::new(TeamKind::Players, entity_id, Some(team_owner_id));
```

### Checking if Enemy

```typescript
// TypeScript
if (team.IsEnemyOf(otherTeam)) {
    // Can attack
}
```

```rust
// Rust
if team.is_enemy_of(&other_team, &self_entity, &other_entity) {
    // Can attack
}
```

### Checking if Ally

```typescript
// TypeScript
if (team.IsAllyOf(otherTeam)) {
    // Cannot attack
}
```

```rust
// Rust
if team.is_ally_of(&other_team, &self_entity, &other_entity) {
    // Cannot attack
}
```

### Team-Based Combat

```typescript
// TypeScript
function canAttack(attacker: Team, target: Team): boolean {
    return attacker.IsEnemyOf(target) && 
           !target.self.states.hasFlag(EntityStates.Invulnerable) &&
           !target.self.states.hasFlag(EntityStates.SafeZone);
}
```

```rust
// Rust
fn can_attack(attacker: &Team, target: &Team, attacker_entity: &Entity, target_entity: &Entity) -> bool {
    attacker.is_enemy_of(target, attacker_entity, target_entity) &&
    !target_entity.flags.has_flag(EntityFlag::Invulnerable) &&
    !target_entity.flags.has_flag(EntityFlag::SafeZone)
}
```

## Best Practices

### Team Management

1. **Server Authority**: Server is authoritative for all team relationships
2. **Validation**: Validate team relationships on the server
3. **Synchronization**: Synchronize teams between server and client
4. **Flag Integration**: Use entity flags to determine team relationships
5. **Performance**: Cache team relationships for performance

### Performance Optimization

1. **Inline Functions**: Use inline functions for team checks
2. **Compile-Time Optimization**: Use compile-time optimization for team logic
3. **Efficient Data Structures**: Use efficient data structures for team storage
4. **Cache Relationships**: Cache enemy/ally lists
5. **Batch Updates**: Batch team updates for efficiency

### Error Prevention

1. **Type Safety**: Use type-safe team operations
2. **Validation**: Validate team relationships before applying
3. **Bounds Checking**: Check team bounds to prevent errors
4. **State Consistency**: Ensure team state consistency
5. **Error Recovery**: Implement error recovery for invalid teams

## Team Relationships Matrix

### Ally Matrix

| Team | Monsters | NPCs | Players | Tower | Guards | Provocation | Team1 | Team2 | Pet |
|------|----------|------|---------|-------|--------|-------------|-------|-------|-----|
| Monsters | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| NPCs | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Players | ❌ | ✅ | ✅* | ✅ | ✅** | ❌ | ❌ | ❌ | ✅*** |
| Tower | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Guards | ❌ | ✅ | ✅** | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Provocation | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Team1 | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Team2 | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Pet | ❌ | ✅ | ✅*** | ✅ | ✅ | ❌ | ❌ | ❌ | ✅*** |

\* Same team owner (party/guild)  
\** Not negative karma  
\*** Same team owner or pet owner

### Enemy Matrix

| Team | Monsters | NPCs | Players | Tower | Guards | Provocation | Team1 | Team2 | Pet |
|------|----------|------|---------|-------|--------|-------------|-------|-------|-----|
| Monsters | ❌ | ❌ | ✅* | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| NPCs | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Players | ✅ | ❌ | ✅** | ❌ | ✅*** | ✅ | ❌ | ❌ | ❌ |
| Tower | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Guards | ✅ | ❌ | ✅*** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Provocation | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Team1 | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅* | ❌ |
| Team2 | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅* | ❌ | ❌ |
| Pet | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

\* Not invulnerable  
\** Duel zone or player enemy  
\*** Negative karma

## Conclusion

The Teams system provides a flexible and efficient way to manage relationships between entities, enabling:

- **PvP Combat**: Player vs Player combat with duel zones and safe zones
- **PvE Combat**: Player vs Environment combat with monsters and NPCs
- **Team-Based Gameplay**: Team1 vs Team2 for team-based combat
- **Social Relationships**: Party and guild relationships
- **Guard System**: Guards protect players from negative karma players
- **Pet System**: Pets follow owner's team relationships

The Rust implementation will leverage type safety and efficient data structures to provide a safe and performant team system while maintaining the flexibility of the current implementation.
