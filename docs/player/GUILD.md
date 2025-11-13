# Guild System Documentation

## Overview

The guild system allows players to form permanent organizations for cooperative gameplay, territorial control, and competitive content. Guilds provide social structure, shared resources, territorial warfare, and specialized features for members. The system includes hierarchy management, resource contribution, shared storage, squad organization, and guild-versus-guild (GvG) warfare.

## Key Features

- **Guild Creation**: Create guilds with gold cost
- **Member Hierarchy**: Multiple ranks with different permissions
- **Custom Flags**: Customizable guild flags displayed next to player names
- **Team Integration**: Integration with TEAM system for events
- **Pet/Mount Marking**: Guild flags on pets and mounts
- **Resource Contribution**: Automatic contribution from drops for guild improvements
- **Shared Storage**: Guild chests crafted with carpentry, access-controlled
- **Squad System**: Organize members into squads for events and GvG
- **Recruitment System**: Application window for joining guilds
- **Guild Discovery**: List, search, and invite system for guilds
- **Guild Wars**: Declare war on other guilds
- **Affiliation Types**: Casual, PvP, GvG filtering
- **Castle Domination**: Control and manage castles
- **Guild Vendors**: Specialized vendors and crafting with member discounts
- **Guild Auction House**: Exclusive auction house for guild members

## Guild Creation

### Creation Process

**Requirements:**
- Player must not be in a guild
- Gold cost (configurable, e.g., 100,000 gold)
- Unique guild name
- Guild flag design

**Process:**
1. Player initiates guild creation
2. System validates requirements (gold, name uniqueness)
3. Gold deducted from player
4. Guild created with player as Owner
5. Guild registered in system
6. Player becomes first member

**Implementation:**
```typescript
public static async createGuild(
    player: Player, 
    guildName: string, 
    flag: string
): Promise<Guild> {
    // Validate requirements
    if (player.guild) {
        throw new Error("Player is already in a guild");
    }
    
    if (player.gold < GUILD_CREATION_COST) {
        throw new Error("Insufficient gold");
    }
    
    if (Guilds.hasGuildByName(guildName)) {
        throw new Error("Guild name already exists");
    }
    
    // Deduct gold
    player.gold -= GUILD_CREATION_COST;
    player.save();
    
    // Create guild
    const guild = new Guild(
        GUID.NewID(),
        player.characterId,
        guildName,
        flag,
        100,  // Max members (default)
        1,    // Level (default)
        "[]"  // Requests (empty)
    );
    
    // Add owner as first member
    guild.Members.push({
        Id: player.characterId,
        Name: player.name,
        Tag: player.hashtag,
        Plevel: GuildAccessLevel.Owner
    });
    
    player.guild = guild;
    player.save();
    
    // Save guild to database
    await guild.update(player.socket.services.guildService);
    
    // Cache guild
    Guilds.CachedGuilds.set(guild.Id, guild);
    Guilds.CachedGuildsByName.set(guild.Name, guild);
    
    return guild;
}
```

## Member Hierarchy

### Access Levels

**Hierarchy (Lowest to Highest):**

1. **Member** (`GuildAccessLevel.Member`)
   - Basic member
   - Can view guild info
   - Can use guild features (vendors, auction house)
   - Cannot manage members or settings

2. **Recruiter** (`GuildAccessLevel.Recruter`)
   - Can invite new members
   - Can accept/deny join requests
   - Can view member list
   - Cannot remove members or change settings

3. **Team Leader** (`GuildAccessLevel.TeamLeader`)
   - Can create and manage squads
   - Can assign squad leaders
   - Can organize team events
   - Cannot manage guild settings

4. **Raid Master** (`GuildAccessLevel.RaidMaster`)
   - Can organize raids
   - Can manage raid groups
   - Can coordinate large-scale events
   - Cannot manage guild settings

5. **Staff** (`GuildAccessLevel.Staff`)
   - Can remove members (except Owner/SubLeader)
   - Can accept/deny requests
   - Can manage member ranks (below Staff)
   - Can manage guild chests
   - Cannot transfer ownership or disband guild

6. **Sub Leader** (`GuildAccessLevel.SubLeader`)
   - All Staff permissions
   - Can manage Staff and below
   - Can manage guild settings (except ownership)
   - Can declare/end guild wars
   - Cannot transfer ownership or disband guild

7. **Owner** (`GuildAccessLevel.Owner`)
   - Full permissions
   - Can transfer ownership
   - Can disband guild
   - Can manage all members and settings
   - Cannot be removed except by leaving

### Permission Matrix

| Action | Member | Recruiter | Team Leader | Raid Master | Staff | Sub Leader | Owner |
|--------|--------|-----------|-------------|-------------|-------|------------|-------|
| View Guild Info | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Use Guild Vendors | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Use Guild Auction | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Invite Members | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Accept/Deny Requests | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Create Squads | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| Manage Squads | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| Organize Raids | | | | ✓ | ✓ | ✓ | ✓ |
| Remove Members | | | | | ✓ | ✓ | ✓ |
| Manage Ranks | | | | | ✓ | ✓ | ✓ |
| Manage Chests | | | | | ✓ | ✓ | ✓ |
| Manage Settings | | | | | | ✓ | ✓ |
| Declare War | | | | | | ✓ | ✓ |
| Transfer Ownership | | | | | | | ✓ |
| Disband Guild | | | | | | | ✓ |

### Rank Management

**Promote Member:**
```typescript
public async promoteMember(
    actor: Player, 
    targetCharacterId: string, 
    newRank: GuildAccessLevel
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.Staff, actor)) {
        return false; // Insufficient permissions
    }
    
    const targetMember = this.Members.find(m => m.Id === targetCharacterId);
    if (!targetMember) {
        return false; // Member not found
    }
    
    // Cannot promote to Owner (use transferOwnership)
    if (newRank === GuildAccessLevel.Owner) {
        return false;
    }
    
    // Cannot promote above own rank (except Owner)
    if (actor.guildMember.Plevel < GuildAccessLevel.Owner) {
        if (newRank >= actor.guildMember.Plevel) {
            return false;
        }
    }
    
    targetMember.Plevel = newRank;
    await this.update(actor.socket.services.guildService);
    
    const targetPlayer = Player.getPlayerByTag(targetMember.Tag);
    if (targetPlayer) {
        packetSystemMessage.sendDirectSocket(
            targetPlayer.socket, 
            `You have been promoted to ${GuildAccessLevel[newRank]}`
        );
    }
    
    return true;
}
```

**Demote Member:**
```typescript
public async demoteMember(
    actor: Player, 
    targetCharacterId: string, 
    newRank: GuildAccessLevel
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.Staff, actor)) {
        return false;
    }
    
    const targetMember = this.Members.find(m => m.Id === targetCharacterId);
    if (!targetMember) {
        return false;
    }
    
    // Cannot demote Owner
    if (targetMember.Plevel === GuildAccessLevel.Owner) {
        return false;
    }
    
    // Cannot demote below Member
    if (newRank < GuildAccessLevel.Member) {
        return false;
    }
    
    targetMember.Plevel = newRank;
    await this.update(actor.socket.services.guildService);
    
    return true;
}
```

## Guild Flag System

### Flag Customization

**Flag Components:**
- **Base Design**: Background pattern/color
- **Symbol**: Central symbol/emblem
- **Colors**: Primary and secondary colors
- **Pattern**: Overlay pattern

**Flag Display:**
- Displayed next to player name
- Displayed on pets and mounts
- Displayed on guild buildings
- Displayed in guild list
- Displayed in GvG battles

**Flag Storage:**
- Flag data stored as string/JSON
- Can be updated by Owner/SubLeader
- Changes require gold cost (optional)

**Implementation:**
```typescript
public async updateFlag(
    actor: Player, 
    flagData: string
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.SubLeader, actor)) {
        return false;
    }
    
    // Validate flag data
    if (!this.validateFlagData(flagData)) {
        return false;
    }
    
    this.Flag = flagData;
    await this.update(actor.socket.services.guildService);
    
    // Broadcast flag update to all members
    this.broadcastFlagUpdate();
    
    return true;
}

private validateFlagData(flagData: string): boolean {
    // Validate flag format, size, etc.
    // Prevent inappropriate content
    return true;
}
```

## Team Integration

### TEAM System Integration

**Guild Events:**
- Guilds can participate in TEAM events
- Guild members form teams for events
- Guild performance tracked
- Rewards distributed to guild

**Team Formation:**
- Team Leaders can form teams from guild members
- Teams can be pre-formed before events
- Team composition saved for events

**Event Coordination:**
- Guild-wide event announcements
- Team assignments for events
- Performance tracking per team
- Rewards distribution

## Pet and Mount Marking

### Guild Flag on Pets/Mounts

**Display:**
- Guild flag displayed on pet/mount model
- Flag appears as banner or emblem
- Visible to all players
- Updates when player changes guild

**Implementation:**
```typescript
public updatePetGuildFlag(pet: Pet): void {
    if (pet.owner.guild) {
        pet.guildFlag = pet.owner.guild.Flag;
        pet.guildName = pet.owner.guild.Name;
    } else {
        pet.guildFlag = null;
        pet.guildName = null;
    }
    
    // Broadcast pet update to nearby players
    packetPetUpdate.sendNearby(pet);
}
```

## Resource Contribution System

### Automatic Contribution

**Contribution Mechanism:**
- Percentage of drops automatically contributed
- Configurable contribution rate (e.g., 5-10%)
- Contributed resources go to guild treasury
- Used for guild improvements

**Contribution Uses:**
- Upgrade guild castles
- Upgrade guild NPCs
- Build guild structures
- Maintain guild facilities
- Fund guild events

**Contribution Settings:**
- Owner/SubLeader can set contribution rate
- Minimum contribution rate (e.g., 5%)
- Maximum contribution rate (e.g., 20%)
- Members can see contribution rate

**Implementation:**
```typescript
public processDropContribution(
    player: Player, 
    dropValue: number
): number {
    if (!player.guild) {
        return 0; // No guild, no contribution
    }
    
    const contributionRate = player.guild.ContributionRate; // e.g., 0.10 (10%)
    const contribution = Math.floor(dropValue * contributionRate);
    
    if (contribution > 0) {
        player.guild.Treasury += contribution;
        player.guild.TotalContributions += contribution;
        
        // Track player's personal contribution
        const member = player.guild.Members.find(m => m.Id === player.characterId);
        if (member) {
            member.PersonalContribution += contribution;
        }
        
        // Save guild
        player.guild.update(player.socket.services.guildService);
        
        // Notify player (optional)
        packetSystemMessage.sendDirectSocket(
            player.socket, 
            `Guild contribution: ${contribution} gold`
        );
    }
    
    return contribution;
}
```

## Guild Chest System

### Shared Storage

**Chest Creation:**
- Crafted using Carpentry skill
- Requires materials and skill level
- Placed in player houses or guild castles
- Owner controls access permissions

**Access Control:**
- Owner can add/remove access permissions
- Permissions per rank (e.g., Staff+ can access)
- Individual member permissions
- Read/Write permissions separate

**Chest Types:**
- **Personal Guild Chest**: Owner controls access
- **Guild Treasury Chest**: Requires Staff+ rank
- **Squad Chest**: Squad members only
- **Public Guild Chest**: All members can access

**Implementation:**
```typescript
class GuildChest {
    id: string;
    ownerId: string;
    guildId: string;
    location: Vector3;
    accessLevel: GuildAccessLevel;  // Minimum rank required
    allowedMembers: string[];       // Specific member IDs
    items: Map<number, Item>;      // Stored items
    
    canAccess(player: Player): boolean {
        if (!player.guild || player.guild.Id !== this.guildId) {
            return false; // Not in same guild
        }
        
        // Check rank
        const member = player.guild.Members.find(m => m.Id === player.characterId);
        if (member && member.Plevel >= this.accessLevel) {
            return true;
        }
        
        // Check individual permission
        if (this.allowedMembers.includes(player.characterId)) {
            return true;
        }
        
        return false;
    }
    
    addAccess(player: Player, targetCharacterId: string): boolean {
        if (player.characterId !== this.ownerId) {
            return false; // Not owner
        }
        
        if (!this.allowedMembers.includes(targetCharacterId)) {
            this.allowedMembers.push(targetCharacterId);
            return true;
        }
        
        return false;
    }
    
    removeAccess(player: Player, targetCharacterId: string): boolean {
        if (player.characterId !== this.ownerId) {
            return false; // Not owner
        }
        
        const index = this.allowedMembers.indexOf(targetCharacterId);
        if (index !== -1) {
            this.allowedMembers.splice(index, 1);
            return true;
        }
        
        return false;
    }
}
```

## Squad System

### Squad Organization

**Squad Purpose:**
- Organize members for events
- GvG battle formations
- Raid organization
- Team coordination

**Squad Structure:**
- Each squad has independent leader
- Squad leader can be any rank
- Squad leader manages squad members
- Squads can have different purposes

**Squad Features:**
- Create/delete squads
- Assign squad leaders
- Add/remove squad members
- Squad-specific chat channel
- Squad-specific chests
- Squad performance tracking

**Implementation:**
```typescript
class GuildSquad {
    id: string;
    guildId: string;
    name: string;
    leaderId: string;
    members: string[];
    purpose: SquadPurpose;  // Event, GvG, Raid, etc.
    maxMembers: number;
    
    static create(
        guild: Guild, 
        creator: Player, 
        name: string, 
        purpose: SquadPurpose
    ): GuildSquad {
        if (!guild.hasPlevel(GuildAccessLevel.TeamLeader, creator)) {
            throw new Error("Insufficient permissions");
        }
        
        const squad = new GuildSquad();
        squad.id = GUID.NewID();
        squad.guildId = guild.Id;
        squad.name = name;
        squad.leaderId = creator.characterId;
        squad.members = [creator.characterId];
        squad.purpose = purpose;
        squad.maxMembers = (purpose === SquadPurpose.GvG) ? 50 : 20;
        
        guild.Squads.push(squad);
        guild.update(creator.socket.services.guildService);
        
        return squad;
    }
    
    assignLeader(newLeaderId: string, actor: Player): boolean {
        if (!this.guild.hasPlevel(GuildAccessLevel.TeamLeader, actor)) {
            return false;
        }
        
        if (!this.members.includes(newLeaderId)) {
            return false; // Not a squad member
        }
        
        this.leaderId = newLeaderId;
        this.guild.update(actor.socket.services.guildService);
        
        return true;
    }
    
    addMember(memberId: string, actor: Player): boolean {
        if (this.leaderId !== actor.characterId && 
            !this.guild.hasPlevel(GuildAccessLevel.TeamLeader, actor)) {
            return false; // Not squad leader or team leader
        }
        
        if (this.members.length >= this.maxMembers) {
            return false; // Squad full
        }
        
        if (this.members.includes(memberId)) {
            return false; // Already member
        }
        
        this.members.push(memberId);
        this.guild.update(actor.socket.services.guildService);
        
        return true;
    }
    
    removeMember(memberId: string, actor: Player): boolean {
        if (this.leaderId !== actor.characterId && 
            !this.guild.hasPlevel(GuildAccessLevel.TeamLeader, actor)) {
            return false;
        }
        
        const index = this.members.indexOf(memberId);
        if (index === -1) {
            return false;
        }
        
        this.members.splice(index, 1);
        this.guild.update(actor.socket.services.guildService);
        
        return true;
    }
}

enum SquadPurpose {
    Event = "event",
    GvG = "gvg",
    Raid = "raid",
    PvP = "pvp",
    General = "general"
}
```

## Recruitment System

### Application Window

**Application Process:**
1. Player searches/browses guilds
2. Player sends application to guild
3. Application appears in guild's application window
4. Staff+ members can review applications
5. Staff+ members accept/deny applications

**Application Window Features:**
- List of pending applications
- Player information (name, level, class)
- Application message (optional)
- Accept/Deny buttons
- Filter by date/level/class

**Implementation:**
```typescript
class GuildApplication {
    id: string;
    guildId: string;
    playerId: string;
    playerName: string;
    playerLevel: number;
    playerClass: string;
    message: string;
    timestamp: Date;
    status: ApplicationStatus;
}

enum ApplicationStatus {
    Pending = "pending",
    Accepted = "accepted",
    Denied = "denied"
}

public async sendApplication(
    player: Player, 
    guildId: string, 
    message?: string
): Promise<boolean> {
    if (player.guild) {
        return false; // Already in guild
    }
    
    const guild = Guilds.getGuild(guildId);
    if (!guild) {
        return false; // Guild not found
    }
    
    if (guild.Members.length >= guild.MaxMembers) {
        return false; // Guild full
    }
    
    // Check if already applied
    const existingRequest = guild.Requests.find(r => r.Id === player.characterId);
    if (existingRequest) {
        return false; // Already applied
    }
    
    // Add application
    guild.Requests.push({
        Id: player.characterId,
        Name: player.name,
        Tag: player.hashtag,
        Message: message || ""
    });
    
    await guild.update(player.socket.services.guildService);
    
    packetSystemMessage.sendDirectSocket(
        player.socket, 
        `Application sent to "${guild.Name}"`
    );
    
    // Notify Staff+ members
    guild.notifyStaffOfApplication(player);
    
    return true;
}

public async acceptApplication(
    actor: Player, 
    applicationId: string
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.Recruter, actor)) {
        return false;
    }
    
    const requestIndex = this.Requests.findIndex(r => r.Id === applicationId);
    if (requestIndex === -1) {
        return false; // Application not found
    }
    
    if (this.Members.length >= this.MaxMembers) {
        return false; // Guild full
    }
    
    const [request] = this.Requests.splice(requestIndex, 1);
    const newMember = Player.getPlayerByTag(request.Tag);
    
    if (newMember) {
        this.Members.push({
            Id: request.Id,
            Name: request.Name,
            Tag: request.Tag,
            Plevel: GuildAccessLevel.Member
        });
        
        newMember.guild = this;
        newMember.save();
        
        packetSystemMessage.sendDirectSocket(
            newMember.socket, 
            `You have been accepted into "${this.Name}"`
        );
        
        // Broadcast to guild members
        this.broadcastMemberJoin(newMember);
    }
    
    await this.update(actor.socket.services.guildService);
    
    return true;
}
```

## Guild Discovery

### List, Search, and Invite

**Guild List:**
- List all public guilds
- Pagination support
- Sort by name, level, members, etc.
- Filter by affiliation type

**Guild Search:**
- Search by name
- Search by tag
- Search by owner name
- Advanced filters (level, members, affiliation)

**Invite System:**
- Recruiters+ can invite players
- Invitation sent to target player
- Player can accept/decline
- Invitation expires after time limit

**Network Packets:**
```typescript
// List guilds
interface GuildListPacket {
    guilds: {
        id: string;
        name: string;
        level: number;
        members: number;
        maxMembers: number;
        flag: string;
        affiliation: GuildAffiliation;
    }[];
    page: number;
    totalPages: number;
}

// Search guilds
interface GuildSearchPacket {
    query: string;
    filters: {
        affiliation?: GuildAffiliation;
        minLevel?: number;
        maxMembers?: number;
    };
}

// Invite player
interface GuildInvitePacket {
    guildId: string;
    guildName: string;
    inviterName: string;
    message?: string;
}
```

**Implementation:**
```typescript
public static getGuildList(
    page: number = 1, 
    pageSize: number = 20,
    filters?: GuildFilters
): GuildListPacket {
    let guildList: any[] = [];
    
    Guilds.CachedGuilds.forEach((guild, id) => {
        // Apply filters
        if (filters) {
            if (filters.affiliation && guild.Affiliation !== filters.affiliation) {
                return;
            }
            if (filters.minLevel && guild.Level < filters.minLevel) {
                return;
            }
            if (filters.maxMembers && guild.Members.length > filters.maxMembers) {
                return;
            }
        }
        
        guildList.push({
            id: id,
            name: guild.Name,
            level: guild.Level,
            members: guild.Members.length,
            maxMembers: guild.MaxMembers,
            flag: guild.Flag,
            affiliation: guild.Affiliation
        });
    });
    
    // Sort and paginate
    guildList.sort((a, b) => a.name.localeCompare(b.name));
    
    const startIndex = (page - 1) * pageSize;
    const endIndex = startIndex + pageSize;
    const paginatedList = guildList.slice(startIndex, endIndex);
    
    return {
        guilds: paginatedList,
        page: page,
        totalPages: Math.ceil(guildList.length / pageSize)
    };
}

public async invitePlayer(
    actor: Player, 
    targetCharacterId: string, 
    message?: string
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.Recruter, actor)) {
        return false;
    }
    
    if (this.Members.length >= this.MaxMembers) {
        return false; // Guild full
    }
    
    const targetPlayer = Player.players.get(targetCharacterId);
    if (!targetPlayer) {
        return false; // Player not found
    }
    
    if (targetPlayer.guild) {
        return false; // Already in guild
    }
    
    // Send invitation
    packetGuildInvite.send(targetPlayer, {
        guildId: this.Id,
        guildName: this.Name,
        inviterName: actor.name,
        message: message
    });
    
    return true;
}
```

## Guild Wars

### War Declaration

**War System:**
- Guilds can declare war on other guilds
- War status affects PvP rules
- War duration and conditions
- War rewards and penalties

**War Declaration:**
- Owner/SubLeader can declare war
- Target guild receives declaration
- War starts after acceptance or auto-start timer
- War can be ended by mutual agreement or victory

**War Features:**
- PvP enabled between warring guilds
- War score tracking
- War objectives (castles, territories)
- War rewards for victories

**Implementation:**
```typescript
class GuildWar {
    id: string;
    attackerGuildId: string;
    defenderGuildId: string;
    status: WarStatus;
    startDate: Date;
    endDate?: Date;
    attackerScore: number;
    defenderScore: number;
    objectives: WarObjective[];
}

enum WarStatus {
    Declared = "declared",
    Active = "active",
    Ended = "ended",
    Cancelled = "cancelled"
}

public async declareWar(
    actor: Player, 
    targetGuildId: string
): Promise<boolean> {
    if (!this.hasPlevel(GuildAccessLevel.SubLeader, actor)) {
        return false;
    }
    
    const targetGuild = Guilds.getGuild(targetGuildId);
    if (!targetGuild) {
        return false; // Target guild not found
    }
    
    // Check if already at war
    if (this.isAtWarWith(targetGuildId)) {
        return false; // Already at war
    }
    
    // Create war declaration
    const war = new GuildWar();
    war.id = GUID.NewID();
    war.attackerGuildId = this.Id;
    war.defenderGuildId = targetGuildId;
    war.status = WarStatus.Declared;
    war.startDate = new Date();
    
    // Notify target guild
    targetGuild.notifyWarDeclaration(this);
    
    // Save war
    await WarRepository.saveWar(war);
    
    return true;
}

public isAtWarWith(guildId: string): boolean {
    // Check active wars
    return this.ActiveWars.some(war => 
        (war.attackerGuildId === guildId || war.defenderGuildId === guildId) &&
        war.status === WarStatus.Active
    );
}
```

## Guild Affiliation Types

### Affiliation Filtering

**Affiliation Types:**
- **Casual**: Social guild, relaxed gameplay
- **PvP**: Focused on player-versus-player content
- **GvG**: Focused on guild-versus-guild warfare
- **PvE**: Focused on player-versus-environment content
- **Mixed**: Combination of multiple focuses

**Affiliation Benefits:**
- Helps players find suitable guilds
- Affects guild matching in events
- Can affect war declaration rules
- Displayed in guild list

**Implementation:**
```typescript
enum GuildAffiliation {
    Casual = "casual",
    PvP = "pvp",
    GvG = "gvg",
    PvE = "pve",
    Mixed = "mixed"
}

public setAffiliation(
    actor: Player, 
    affiliation: GuildAffiliation
): boolean {
    if (!this.hasPlevel(GuildAccessLevel.SubLeader, actor)) {
        return false;
    }
    
    this.Affiliation = affiliation;
    await this.update(actor.socket.services.guildService);
    
    return true;
}
```

## Castle Domination

### Castle Control

**Castle System:**
- Guilds can control castles
- Castles provide benefits
- Castles can be attacked and captured
- Castle maintenance and upgrades

**Castle Benefits:**
- Resource generation
- Strategic positions
- Guild facilities
- Defensive advantages

**Castle Management:**
- Owner/SubLeader can manage castles
- Assign castle managers
- Upgrade castle facilities
- Set castle defenses

**Implementation:**
```typescript
class GuildCastle {
    id: string;
    guildId: string;
    name: string;
    level: number;
    facilities: CastleFacility[];
    defenses: CastleDefense[];
    resourceGeneration: ResourceGeneration;
    lastAttack?: Date;
}

public async captureCastle(
    guild: Guild, 
    castleId: string
): Promise<boolean> {
    const castle = CastleManager.getCastle(castleId);
    if (!castle) {
        return false;
    }
    
    // Check if guild can capture (war, siege, etc.)
    if (!this.canCaptureCastle(guild, castle)) {
        return false;
    }
    
    // Transfer ownership
    const previousOwner = castle.guildId;
    castle.guildId = guild.Id;
    
    // Notify previous owner
    if (previousOwner) {
        const previousGuild = Guilds.getGuild(previousOwner);
        previousGuild?.notifyCastleLost(castle);
    }
    
    // Notify new owner
    guild.notifyCastleCaptured(castle);
    
    // Save castle
    await CastleManager.saveCastle(castle);
    
    return true;
}
```

## Guild Vendors and Crafting

### Specialized Services

**Guild Vendors:**
- Vendors placed in guild castles
- Sell items to guild members
- Discounted prices for members
- Special items available

**Guild Crafting:**
- Specialized crafting stations
- Guild-only recipes
- Reduced crafting costs for members
- Exclusive items

**Vendor Management:**
- Owner/SubLeader can place vendors
- Set vendor inventory
- Set member discount rates
- Manage vendor locations

**Implementation:**
```typescript
class GuildVendor {
    id: string;
    guildId: string;
    name: string;
    location: Vector3;
    inventory: VendorItem[];
    memberDiscount: number;  // Percentage (e.g., 0.10 = 10% off)
    
    getPrice(item: Item, buyer: Player): number {
        const basePrice = item.basePrice;
        
        if (buyer.guild?.Id === this.guildId) {
            // Member discount
            return Math.floor(basePrice * (1 - this.memberDiscount));
        }
        
        return basePrice;
    }
}

class GuildCraftingStation {
    id: string;
    guildId: string;
    stationType: CraftingStationType;
    location: Vector3;
    recipes: Recipe[];
    memberCostReduction: number;  // Percentage reduction
    
    getCraftingCost(recipe: Recipe, crafter: Player): number {
        const baseCost = recipe.cost;
        
        if (crafter.guild?.Id === this.guildId) {
            // Member cost reduction
            return Math.floor(baseCost * (1 - this.memberCostReduction));
        }
        
        return baseCost;
    }
}
```

## Guild Auction House

### Exclusive Marketplace

**Guild Auction House:**
- Auction house exclusive to guild members
- Members can list items
- Members can bid on items
- Guild takes commission (optional)

**Features:**
- Item listing
- Bidding system
- Buyout option
- Item history
- Member-only access

**Implementation:**
```typescript
class GuildAuctionHouse {
    guildId: string;
    listings: AuctionListing[];
    
    listItem(
        seller: Player, 
        item: Item, 
        startingBid: number, 
        buyoutPrice?: number
    ): boolean {
        if (seller.guild?.Id !== this.guildId) {
            return false; // Not a guild member
        }
        
        const listing = new AuctionListing();
        listing.id = GUID.NewID();
        listing.sellerId = seller.characterId;
        listing.item = item;
        listing.startingBid = startingBid;
        listing.buyoutPrice = buyoutPrice;
        listing.currentBid = startingBid;
        listing.endTime = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days
        
        this.listings.push(listing);
        
        // Broadcast to guild members
        this.broadcastNewListing(listing);
        
        return true;
    }
    
    placeBid(bidder: Player, listingId: string, bidAmount: number): boolean {
        if (bidder.guild?.Id !== this.guildId) {
            return false; // Not a guild member
        }
        
        const listing = this.listings.find(l => l.id === listingId);
        if (!listing) {
            return false; // Listing not found
        }
        
        if (bidAmount <= listing.currentBid) {
            return false; // Bid too low
        }
        
        // Refund previous bidder
        if (listing.currentBidderId) {
            const previousBidder = Player.players.get(listing.currentBidderId);
            if (previousBidder) {
                previousBidder.gold += listing.currentBid;
            }
        }
        
        // Deduct gold from new bidder
        bidder.gold -= bidAmount;
        bidder.save();
        
        listing.currentBid = bidAmount;
        listing.currentBidderId = bidder.characterId;
        
        // Broadcast bid update
        this.broadcastBidUpdate(listing);
        
        return true;
    }
}
```

## Guild Persistence

### Save and Load

**Guild Data Stored:**
- Guild ID, name, flag
- Owner and members
- Member ranks and permissions
- Guild level and settings
- Treasury and contributions
- Squads and applications
- Active wars
- Castle ownership
- Vendor and crafting stations

**Implementation:**
```typescript
public async save(): Promise<void> {
    const guildData = {
        id: this.Id,
        owner: this.Owner,
        guildName: this.Name,
        flag: this.Flag,
        maxMembers: this.MaxMembers,
        level: this.Level,
        members: JSON.stringify(this.Members.map(m => ({
            id: m.Id,
            name: m.Name,
            plevel: m.Plevel,
            tag: m.Tag
        }))),
        requests: JSON.stringify(this.Requests),
        treasury: this.Treasury,
        contributionRate: this.ContributionRate,
        affiliation: this.Affiliation,
        squads: JSON.stringify(this.Squads),
        activeWars: JSON.stringify(this.ActiveWars)
    };
    
    await this.guildService.updateGuild(guildData);
}

public static async loadGuild(guildId: string): Promise<Guild | null> {
    const guildData = await Repository.getGuild(guildId);
    if (!guildData) {
        return null;
    }
    
    const guild = Guilds.fromDatabase(guildData);
    return guild;
}
```

## Network Packets

### Guild Packets

**Guild Data Packet:**
```typescript
interface GuildDataPacket {
    id: string;
    name: string;
    flag: string;
    level: number;
    members: GuildMember[];
    requests?: GuildApplication[];
    isOwner: boolean;
    memberRank: GuildAccessLevel;
    treasury: number;
    contributionRate: number;
    affiliation: GuildAffiliation;
    squads: GuildSquad[];
}
```

**Guild List Packet:**
```typescript
interface GuildListPacket {
    guilds: {
        id: string;
        name: string;
        level: number;
        members: number;
        maxMembers: number;
        flag: string;
        affiliation: GuildAffiliation;
    }[];
    page: number;
    totalPages: number;
}
```

**Guild Invite Packet:**
```typescript
interface GuildInvitePacket {
    guildId: string;
    guildName: string;
    inviterName: string;
    message?: string;
}
```

## Performance Considerations

### Optimization Strategies

1. **Caching**: Cache guild data in memory
2. **Lazy Loading**: Load member data on demand
3. **Batch Updates**: Batch guild updates
4. **Event-Driven**: Use events for guild updates
5. **Database Indexing**: Index guild lookups

## Integration with Other Systems

### Combat System
- Guild wars affect PvP rules
- Guild buffs in combat
- Shared damage in guild groups

### Economy System
- Guild treasury management
- Contribution from drops
- Guild auction house
- Vendor discounts

### Territory System
- Castle control
- Territory warfare
- Resource generation

### Event System
- Guild participation in events
- Team formation
- Performance tracking

## Summary

The guild system provides:

- **Complete Hierarchy**: 7 ranks with different permissions
- **Custom Flags**: Displayed on players, pets, mounts
- **Resource Management**: Contribution system and shared storage
- **Organization Tools**: Squads for events and GvG
- **Recruitment**: Application and invite system
- **Warfare**: Guild wars and castle domination
- **Services**: Vendors, crafting, auction house
- **Discovery**: List, search, and invite features

The system ensures comprehensive guild management with social, economic, and competitive features for long-term guild gameplay.

