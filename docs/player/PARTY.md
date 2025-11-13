# Party and Raid System Documentation

## Overview

The party and raid system allows players to form groups for cooperative gameplay. The system supports two types of groups: **Party** (up to 5 members) for regular group content and events, and **Raid** (up to 50 members) for large-scale content. The system includes leader controls, member management, buff/debuff sharing, and role assignment features.

## Key Features

- **Two Group Types**: Party (5 members) and Raid (50 members)
- **Leader Controls**: Advanced leadership functions for group management
- **Role Assignment**: Mark important players (tanks, healers, DPS)
- **Buff/Debuff Sharing**: Shared buffs and debuffs within the group
- **Leadership Transfer**: Transfer leadership to another member
- **Member Management**: Invite, kick, promote, and manage members
- **Status Synchronization**: Real-time health, mana, stamina, and status updates
- **Offline Detection**: Track member online/offline status
- **Persistent Groups**: Groups persist across server restarts

## Group Types

### Party (Regular Group)

**Purpose:**
- Regular group content (dungeons, quests, events)
- Small-scale cooperative gameplay
- Up to 5 members

**Use Cases:**
- Dungeon runs
- Quest groups
- Small PvP groups
- Event participation
- Social grouping

**Limitations:**
- Maximum 5 members
- No raid-specific features
- Standard buff sharing

### Raid (Large Group)

**Purpose:**
- Large-scale content (raids, world bosses, massive PvP)
- Up to 50 members
- Advanced management features

**Use Cases:**
- Raid dungeons
- World boss encounters
- Large-scale PvP battles
- Guild events
- Massive cooperative content

**Features:**
- Up to 50 members
- Sub-group management
- Raid-specific buffs
- Advanced role assignment
- Raid-wide announcements

## Group Structure

### Basic Structure

```typescript
class Party {
    id: string;                    // Unique party ID
    type: GroupType;               // Party or Raid
    owner: Player;                 // Leader/Owner
    maxMembers: number;            // 5 for Party, 50 for Raid
    members: Map<string, Player>;  // All members
    createdAt: Date;               // Creation timestamp
    settings: PartySettings;       // Group settings
}
```

### Group Types Enum

```typescript
enum GroupType {
    Party = "party",  // Up to 5 members
    Raid = "raid"     // Up to 50 members
}
```

### Member Roles

```typescript
enum MemberRole {
    Leader = "leader",           // Group leader
    CoLeader = "coleader",      // Co-leader (raid only)
    Tank = "tank",              // Tank role
    Healer = "healer",          // Healer role
    DPS = "dps",                // Damage dealer
    Support = "support",        // Support role
    Member = "member"          // Regular member
}
```

### Member Data Structure

```typescript
interface PartyMember {
    characterId: string;
    name: string;
    role: MemberRole;
    isImportant: boolean;        // Marked as important
    isLeader: boolean;
    isCoLeader: boolean;         // Raid only
    life: number;
    maxLife: number;
    mana: number;
    maxMana: number;
    stamina: number;
    maxStamina: number;
    isOnline: boolean;
    lastUpdate: Date;
    hashtag: string;
}
```

## Group Creation

### Create Party

**Process:**
1. Player initiates party creation
2. System creates new party with player as leader
3. Player is added as first member
4. Party ID is generated
5. Party is registered in system

**Implementation:**
```typescript
public static createParty(owner: Player, type: GroupType = GroupType.Party): Party {
    const party = new Party();
    party.id = GUID.NewID();
    party.type = type;
    party.owner = owner;
    party.maxMembers = (type === GroupType.Party) ? 5 : 50;
    party.members = new Map();
    party.members.set(owner.characterId, owner);
    party.createdAt = new Date();
    
    owner.party = party;
    owner.partyOwner = owner;
    owner.partyRole = MemberRole.Leader;
    
    Party.partySessions.set(party.id, party);
    party.broadcast();
    
    return party;
}
```

### Create Raid

**Process:**
1. Leader creates raid (requires special permission or item)
2. System creates raid with 50 member capacity
3. Leader can invite up to 50 members
4. Raid-specific features enabled

**Requirements:**
- May require special item or permission
- Leader must have sufficient level/rank
- May have cooldown restrictions

## Member Management

### Invite Member

**Process:**
1. Leader selects target player
2. System checks if player is available
3. Invitation sent to target player
4. Player accepts/declines invitation
5. If accepted, player joins group

**Implementation:**
```typescript
public requestMember(characterId: string): boolean {
    if (this.members.size >= this.maxMembers) {
        return false; // Group is full
    }
    
    if (Player.players.has(characterId)) {
        const player = Player.players.get(characterId);
        
        if (player && !player.party) {
            packetRequestParty.send(player, this.id, this.owner.name);
            return true;
        } else {
            packetSystemMessage.sendDirectSocket(
                this.owner.socket, 
                `The player is already in another group.`
            );
            return false;
        }
    }
    
    return false;
}
```

### Join Member

**Process:**
1. Player accepts invitation
2. System validates group capacity
3. Player added to group
4. Group data broadcasted to all members
5. Notifications sent

**Implementation:**
```typescript
public joinMember(player: Player): boolean {
    if (this.members.size >= this.maxMembers) {
        packetSystemMessage.sendDirectSocket(
            player.socket, 
            `The group is already full.`
        );
        return false;
    }
    
    this.members.set(player.characterId, player);
    player.party = this;
    player.partyOwner = this.owner;
    player.partyRole = MemberRole.Member;
    
    Player.players.set(player.characterId, player);
    player.save();
    
    packetSystemMessage.sendDirectSocket(
        player.socket, 
        `You have been accepted into the group organized by ${this.owner.name}.`
    );
    
    this.members.forEach((member) => {
        if (member.characterId !== player.characterId) {
            packetSystemMessage.sendDirectSocket(
                member.socket, 
                `Member ${player.name} joined the group.`
            );
        }
    });
    
    this.broadcast();
    return true;
}
```

### Remove Member (Kick)

**Process:**
1. Leader selects member to remove
2. System validates leader permissions
3. Member removed from group
4. Member notified
5. Group data updated

**Implementation:**
```typescript
public kickMember(leader: Player, targetCharacterId: string): boolean {
    if (leader.characterId !== this.owner.characterId) {
        return false; // Not leader
    }
    
    if (targetCharacterId === this.owner.characterId) {
        return false; // Cannot kick self
    }
    
    if (this.members.has(targetCharacterId)) {
        const targetPlayer = this.members.get(targetCharacterId);
        this.leave(targetPlayer);
        return true;
    }
    
    return false;
}
```

### Leave Group

**Process:**
1. Player initiates leave
2. Player removed from group
3. If leader leaves, leadership transferred
4. Group data updated
5. Notifications sent

**Implementation:**
```typescript
public leave(player: Player): void {
    if (!this.members.has(player.characterId)) {
        return;
    }
    
    this.members.delete(player.characterId);
    player.party = null;
    player.partyOwner = null;
    player.partyRole = null;
    player.save();
    
    Player.players.set(player.characterId, player);
    
    // Clear party data for leaving player
    packetPartyData.send(player, JSON.stringify({
        members: [],
        isLeader: false,
        force: true
    }));
    
    // Transfer leadership if leader left
    if (this.owner.characterId === player.characterId) {
        if (this.members.size > 0) {
            // Transfer to first available member
            const newLeader = Array.from(this.members.values())[0];
            this.transferLeadership(newLeader);
        } else {
            // No members left, disband party
            this.disband();
            return;
        }
    }
    
    Party.updateParty(this);
    packetSystemMessage.sendDirectSocket(
        player.socket, 
        `You left the group`
    );
    
    this.members.forEach((member) => {
        packetSystemMessage.sendDirectSocket(
            member.socket, 
            `Member ${player.name} left the group`
        );
    });
    
    this.broadcast(true);
}
```

## Leadership Management

### Transfer Leadership

**Process:**
1. Leader selects target member
2. System validates target is member
3. Leadership transferred
4. Old leader becomes regular member
5. New leader receives leader permissions
6. Group data updated

**Implementation:**
```typescript
public transferLeadership(newLeader: Player): boolean {
    if (!this.members.has(newLeader.characterId)) {
        return false; // Not a member
    }
    
    const oldLeader = this.owner;
    
    // Update old leader
    oldLeader.partyRole = MemberRole.Member;
    
    // Update new leader
    this.owner = newLeader;
    newLeader.partyRole = MemberRole.Leader;
    newLeader.partyOwner = newLeader;
    
    // Update all members' partyOwner reference
    this.members.forEach((member) => {
        member.partyOwner = newLeader;
        member.save();
    });
    
    packetSystemMessage.sendDirectSocket(
        newLeader.socket, 
        `You have been promoted as group leader`
    );
    
    packetSystemMessage.sendDirectSocket(
        oldLeader.socket, 
        `Leadership has been transferred to ${newLeader.name}`
    );
    
    this.broadcast(true);
    return true;
}
```

### Promote to Co-Leader (Raid Only)

**Process:**
1. Leader selects member
2. Member promoted to co-leader
3. Co-leader receives additional permissions
4. Group data updated

**Co-Leader Permissions:**
- Invite members
- Kick members (except leader and other co-leaders)
- Mark important players
- Manage roles
- Cannot transfer leadership or disband raid

**Implementation:**
```typescript
public promoteToCoLeader(member: Player): boolean {
    if (this.type !== GroupType.Raid) {
        return false; // Only for raids
    }
    
    if (!this.members.has(member.characterId)) {
        return false; // Not a member
    }
    
    member.partyRole = MemberRole.CoLeader;
    member.save();
    
    packetSystemMessage.sendDirectSocket(
        member.socket, 
        `You have been promoted to co-leader`
    );
    
    this.broadcast();
    return true;
}
```

## Role Assignment

### Assign Role

**Process:**
1. Leader/Co-Leader selects member
2. Selects role (Tank, Healer, DPS, Support)
3. Role assigned to member
4. Group data updated

**Roles:**
- **Tank**: Primary damage absorber
- **Healer**: Primary healer
- **DPS**: Damage dealer
- **Support**: Support/buffer

**Implementation:**
```typescript
public assignRole(member: Player, role: MemberRole): boolean {
    if (!this.members.has(member.characterId)) {
        return false;
    }
    
    // Validate role (cannot assign Leader or CoLeader this way)
    if (role === MemberRole.Leader || role === MemberRole.CoLeader) {
        return false;
    }
    
    member.partyRole = role;
    member.save();
    
    this.broadcast();
    return true;
}
```

### Mark Important Player

**Process:**
1. Leader/Co-Leader selects member
2. Toggles important flag
3. Important players highlighted in UI
4. Group data updated

**Use Cases:**
- Mark main tank
- Mark main healer
- Mark key DPS
- Mark support players

**Implementation:**
```typescript
public markImportant(member: Player, important: boolean): boolean {
    if (!this.members.has(member.characterId)) {
        return false;
    }
    
    member.partyIsImportant = important;
    member.save();
    
    this.broadcast();
    return true;
}
```

## Buff and Debuff System

### Shared Buffs

**Mechanism:**
- Buffs cast by party members can be shared
- Shared buffs apply to all party members within range
- Range: 1000 units (configurable)
- Duration: Same as original buff
- Stacking: Follows normal buff stacking rules

**Buff Sharing Rules:**
- Only beneficial buffs are shared
- Buffs with "party_share" flag are shared
- Leader can toggle buff sharing per buff type
- Co-leaders can manage buff sharing (raid)

**Implementation:**
```typescript
public shareBuff(caster: Player, buff: Buff): void {
    if (!this.members.has(caster.characterId)) {
        return; // Not a member
    }
    
    if (!buff.flags.has(BuffFlags.PartyShare)) {
        return; // Buff not shareable
    }
    
    const casterPosition = caster.transform.position;
    
    this.members.forEach((member) => {
        if (member.characterId === caster.characterId) {
            return; // Don't share to caster
        }
        
        const distance = casterPosition.distanceTo(member.transform.position);
        
        if (distance <= 1000) { // Within range
            member.applyBuff(buff, caster);
        }
    });
}
```

### Shared Debuffs

**Mechanism:**
- Debuffs applied to enemies can be shared
- Shared debuffs help coordinate attacks
- Visual indicators show shared debuffs
- Duration tracking for all members

**Debuff Sharing Rules:**
- Only harmful debuffs are shared (enemy debuffs)
- Debuffs with "party_share" flag are shared
- All members can see debuff timers
- Leader can prioritize debuff display

**Implementation:**
```typescript
public shareDebuff(target: Entity, debuff: Debuff, caster: Player): void {
    if (!this.members.has(caster.characterId)) {
        return;
    }
    
    if (!debuff.flags.has(DebuffFlags.PartyShare)) {
        return;
    }
    
    // Broadcast debuff info to all party members
    this.members.forEach((member) => {
        packetDebuffInfo.send(member, {
            targetId: target.id,
            debuffId: debuff.id,
            duration: debuff.duration,
            casterId: caster.characterId
        });
    });
}
```

### Buff Priority System

**Priority Rules:**
- Leader buffs have highest priority
- Co-leader buffs have second priority
- Important players' buffs have third priority
- Regular member buffs have lowest priority

**Conflict Resolution:**
- Higher priority buffs override lower priority
- Same priority: Longer duration wins
- Same priority and duration: Most recent wins

## Status Synchronization

### Member Status Updates

**Synchronized Data:**
- Health (life/maxLife)
- Mana (mana/maxMana)
- Stamina (stamina/maxStamina)
- Online/Offline status
- Position (optional, for proximity checks)
- Status effects (buffs/debuffs)

**Update Frequency:**
- Real-time: Health, Mana, Stamina (on change)
- Periodic: Full status sync every 2 seconds
- On-demand: When member joins/leaves

**Implementation:**
```typescript
public tick(tickNumber: number): void {
    // Full status sync every 60 ticks (2 seconds at 30 TPS)
    if (tickNumber % 60 === 0) {
        this.broadcast();
    }
    
    // Check for offline members
    this.checkOfflineMembers();
}

private checkOfflineMembers(): void {
    const now = new Date().getTime();
    const offlineThreshold = 30 * 1000; // 30 seconds
    
    this.members.forEach((member) => {
        const timeSinceUpdate = now - member.lastUpdate.getTime();
        
        if (timeSinceUpdate > offlineThreshold) {
            member.isOnline = false;
        } else {
            member.isOnline = true;
        }
    });
}
```

### Refresh Member Data

**Process:**
1. Member data updated (health, mana, etc.)
2. System refreshes member in party
3. Party data serialized for that member
4. Updated data sent to member
5. Other members receive update via broadcast

**Implementation:**
```typescript
public refreshCharacter(player: Player): void {
    if (this.owner.characterId === player.characterId) {
        this.owner = player;
    }
    
    this.members.set(player.characterId, player);
    Player.players.set(player.characterId, player);
    
    // Send updated party data after short delay
    setTimeout(() => {
        const partyData = this.serialize(player);
        packetPartyData.send(player, partyData);
    }, 2000);
}
```

## Group Settings

### Party Settings

```typescript
interface PartySettings {
    // Invitation settings
    autoAcceptInvites: boolean;      // Auto-accept invitations
    inviteLevelRange: number;        // Max level difference for invites
    
    // Loot settings
    lootMode: LootMode;              // Free-for-all, Round-robin, Need-before-greed
    lootMaster: string;              // Character ID of loot master
    
    // Buff settings
    shareBuffs: boolean;             // Enable buff sharing
    shareDebuffs: boolean;           // Enable debuff sharing
    buffRange: number;               // Buff sharing range
    
    // Raid-specific settings
    subGroups: SubGroup[];           // Raid sub-groups (raid only)
    raidAnnouncements: boolean;      // Enable raid-wide announcements
}
```

### Loot Modes

**Free-for-All:**
- Anyone can loot
- No restrictions
- Default for party

**Round-Robin:**
- Loot rotates between members
- Fair distribution
- Good for balanced groups

**Need-Before-Greed:**
- Members roll for items
- Highest roll wins
- Good for valuable items

**Master Looter:**
- Leader/designated member distributes loot
- Best for organized raids
- Prevents ninja looting

## Raid-Specific Features

### Sub-Groups

**Purpose:**
- Organize large raids into smaller groups
- Assign roles per sub-group
- Manage buffs per sub-group

**Structure:**
- Up to 10 sub-groups in raid
- 5 members per sub-group
- Each sub-group has leader
- Sub-group buffs apply to sub-group only

**Implementation:**
```typescript
interface SubGroup {
    id: number;                      // Sub-group ID (1-10)
    leader: Player;                  // Sub-group leader
    members: Player[];               // Sub-group members (max 5)
    role: string;                    // Sub-group role (tank, healer, dps)
}
```

### Raid Announcements

**Purpose:**
- Communicate with entire raid
- Important messages
- Strategy coordination

**Features:**
- Leader and co-leaders can send announcements
- Announcements displayed prominently
- Can be targeted to specific sub-groups
- Persistent until dismissed

**Implementation:**
```typescript
public sendAnnouncement(sender: Player, message: string, targetSubGroup?: number): void {
    if (this.type !== GroupType.Raid) {
        return;
    }
    
    if (sender.characterId !== this.owner.characterId && 
        sender.partyRole !== MemberRole.CoLeader) {
        return; // Not authorized
    }
    
    const announcement = {
        sender: sender.name,
        message: message,
        timestamp: new Date(),
        targetSubGroup: targetSubGroup
    };
    
    if (targetSubGroup) {
        // Send to specific sub-group
        const subGroup = this.settings.subGroups[targetSubGroup - 1];
        subGroup.members.forEach((member) => {
            packetRaidAnnouncement.send(member, announcement);
        });
    } else {
        // Send to entire raid
        this.members.forEach((member) => {
            packetRaidAnnouncement.send(member, announcement);
        });
    }
}
```

## Group Persistence

### Save Group Data

**Process:**
1. Group data serialized
2. Saved to database
3. Member references stored
4. Settings persisted

**Data Stored:**
- Group ID
- Group type
- Leader character ID
- Member character IDs
- Group settings
- Creation timestamp

**Implementation:**
```typescript
public async save(): Promise<void> {
    const groupData = {
        id: this.id,
        type: this.type,
        ownerId: this.owner.characterId,
        memberIds: Array.from(this.members.keys()),
        settings: this.settings,
        createdAt: this.createdAt
    };
    
    await Repository.saveParty(groupData);
}
```

### Load Group Data

**Process:**
1. System loads groups from database
2. Members loaded and added to group
3. Group recreated in memory
4. Members notified of group status

**Implementation:**
```typescript
public static async loadParties(): Promise<void> {
    const parties = await Repository.getParties();
    
    for (const partyData of parties) {
        const party = new Party();
        party.id = partyData.id;
        party.type = partyData.type;
        party.settings = partyData.settings;
        party.createdAt = partyData.createdAt;
        
        // Load owner
        const owner = await Player.load(partyData.ownerId);
        if (owner) {
            party.owner = owner;
            party.members.set(owner.characterId, owner);
            owner.party = party;
            owner.partyOwner = owner;
        }
        
        // Load members
        for (const memberId of partyData.memberIds) {
            if (memberId !== partyData.ownerId) {
                const member = await Player.load(memberId);
                if (member) {
                    party.members.set(member.characterId, member);
                    member.party = party;
                    member.partyOwner = owner;
                }
            }
        }
        
        Party.partySessions.set(party.id, party);
    }
}
```

## Group Disbanding

### Disband Group

**Process:**
1. Leader initiates disband
2. All members notified
3. Members removed from group
4. Group data cleared
5. Group removed from system

**Implementation:**
```typescript
public disband(): void {
    this.members.forEach((member) => {
        member.party = null;
        member.partyOwner = null;
        member.partyRole = null;
        member.save();
        
        packetPartyData.send(member, JSON.stringify({
            members: [],
            isLeader: false,
            force: true
        }));
        
        packetSystemMessage.sendDirectSocket(
            member.socket, 
            `The group has been disbanded.`
        );
    });
    
    Party.partySessions.delete(this.id);
    Repository.deleteParty(this.id);
}
```

## Network Packets

### Party Data Packet

**Structure:**
```typescript
interface PartyDataPacket {
    sessionId: string;
    members: PartyMember[];
    isLeader: boolean;
    force: boolean;  // Force update
}
```

### Request Party Packet

**Structure:**
```typescript
interface RequestPartyPacket {
    partyId: string;
    leaderName: string;
}
```

### Party Member Update Packet

**Structure:**
```typescript
interface PartyMemberUpdatePacket {
    characterId: string;
    life: number;
    maxLife: number;
    mana: number;
    maxMana: number;
    stamina: number;
    maxStamina: number;
    isOnline: boolean;
    role: MemberRole;
    isImportant: boolean;
}
```

## Performance Considerations

### Optimization Strategies

**1. Batch Updates:**
- Group multiple member updates
- Send updates in batches
- Reduce network traffic

**2. Proximity-Based Updates:**
- Only update members in same area
- Reduce unnecessary data sync
- Improve performance

**3. Caching:**
- Cache party data
- Reduce database queries
- Faster lookups

**4. Lazy Loading:**
- Load member data on demand
- Don't load offline members immediately
- Reduce memory usage

## Integration with Other Systems

### Combat System

- Shared damage in party
- Coordinated attacks
- Group buffs/debuffs
- Raid-wide damage sharing

### Loot System

- Group loot distribution
- Loot mode enforcement
- Master looter support
- Need-before-greed rolls

### Skill System

- Party-wide skill effects
- Coordinated skill usage
- Group skill bonuses

### Quest System

- Shared quest progress
- Group quest objectives
- Raid quest support

## Testing Considerations

### Unit Tests

- Group creation
- Member management
- Leadership transfer
- Role assignment
- Buff sharing

### Integration Tests

- Full group lifecycle
- Member join/leave flow
- Leadership transfer flow
- Buff sharing mechanics
- Status synchronization

### Performance Tests

- Large raid performance (50 members)
- Status update frequency
- Network packet size
- Database query performance

## Known Issues (From Previous Implementation)

### Issues Fixed

1. **Incomplete leave logic**: Fixed leadership transfer
2. **Missing buff system**: Added buff/debuff sharing
3. **No raid support**: Added raid system
4. **Limited leader controls**: Added advanced controls
5. **No role assignment**: Added role system
6. **Missing persistence**: Added save/load system

### Improvements Made

1. **Better member management**: Complete invite/kick/leave flow
2. **Leadership system**: Transfer and co-leader support
3. **Role assignment**: Tank, healer, DPS, support roles
4. **Important players**: Mark key members
5. **Buff system**: Shared buffs and debuffs
6. **Raid features**: Sub-groups and announcements
7. **Persistence**: Save/load groups

## Summary

The party and raid system provides:

- **Two group types**: Party (5 members) and Raid (50 members)
- **Advanced leadership**: Transfer, co-leaders, role assignment
- **Member management**: Invite, kick, leave, promote
- **Buff system**: Shared buffs and debuffs
- **Status sync**: Real-time member status updates
- **Raid features**: Sub-groups, announcements, advanced management
- **Persistence**: Groups saved and loaded across restarts

The system ensures smooth group gameplay with proper management tools for both small parties and large raids.

