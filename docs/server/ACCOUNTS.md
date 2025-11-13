# Account System Documentation

## Overview

The account system manages player authentication, account creation, and character management across multiple servers. Players can create accounts through various methods (website, Steam, Epic Games) and access multiple characters segmented by server. Each server manages its own player authentication and character data, ensuring proper segmentation and security.

## Key Features

- **Multiple Authentication Methods**: Website registration, Steam, Epic Games
- **Account Management**: Single account can access multiple characters
- **Server Segmentation**: Characters are segmented by server
- **Cross-Platform Support**: Steam and Epic Games integration
- **Character Selection**: Players select server before character selection
- **Server Authentication**: Each server handles its own player authentication
- **Future Character Transfer**: Planned but complex feature

## Authentication Methods

### Website Registration

**Description:**
Players can create accounts directly through the game's website registration system.

**Process:**
1. Player visits registration page
2. Fills in required information (email, username, password)
3. Account is created in central database
4. Player receives confirmation email
5. Account is ready for use

**Account Information:**
- Email address (unique identifier)
- Username (display name)
- Password (hashed and secured)
- Account creation date
- Account status (active, banned, suspended)

**Security:**
- Passwords are hashed using secure algorithms
- Email verification required
- Account recovery via email
- Two-factor authentication (optional)

### Steam Authentication

**Description:**
Players can link their Steam account to create or access their game account.

**Process:**
1. Player selects "Login with Steam" option
2. Redirected to Steam authentication page
3. Player authorizes game access
4. Steam account ID is linked to game account
5. Account is created or accessed

**Steam Integration:**
- Uses Steam OpenID/OAuth
- Steam account ID stored as authentication method
- Steam profile information can be accessed
- Steam achievements integration (optional)

**Account Linking:**
- Steam account ID linked to game account
- Can be linked to existing website account
- One Steam account = One game account
- Cannot unlink without support

### Epic Games Authentication

**Description:**
Players can link their Epic Games account to create or access their game account.

**Process:**
1. Player selects "Login with Epic Games" option
2. Redirected to Epic Games authentication page
3. Player authorizes game access
4. Epic Games account ID is linked to game account
5. Account is created or accessed

**Epic Games Integration:**
- Uses Epic Games OAuth
- Epic Games account ID stored as authentication method
- Epic Games profile information can be accessed
- Epic Games achievements integration (optional)

**Account Linking:**
- Epic Games account ID linked to game account
- Can be linked to existing website account
- One Epic Games account = One game account
- Cannot unlink without support

## Account Structure

### Account Data Model

```rust
pub struct Account {
    pub account_id: AccountId,
    pub email: String,
    pub username: String,
    pub password_hash: String, // Only for website accounts
    pub authentication_methods: Vec<AuthenticationMethod>,
    pub created_at: DateTime,
    pub last_login: Option<DateTime>,
    pub account_status: AccountStatus,
    pub characters: HashMap<ServerId, Vec<CharacterId>>, // Characters per server
}

#[derive(Debug, Clone)]
pub enum AuthenticationMethod {
    Website {
        email: String,
        password_hash: String,
    },
    Steam {
        steam_id: String,
        linked_at: DateTime,
    },
    EpicGames {
        epic_id: String,
        linked_at: DateTime,
    },
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum AccountStatus {
    Active,
    Banned {
        reason: String,
        banned_by: AccountId,
        banned_at: DateTime,
    },
    Suspended {
        reason: String,
        suspended_until: DateTime,
    },
    Inactive,
}
```

### Account Creation Flow

**Website Registration:**
```
1. Player submits registration form
2. Server validates email format
3. Server checks email uniqueness
4. Server hashes password
5. Account created with Website authentication method
6. Confirmation email sent
7. Account status: Active (after email verification)
```

**Steam/Epic Games:**
```
1. Player clicks "Login with [Platform]"
2. Redirected to platform OAuth
3. Player authorizes application
4. Server receives platform account ID
5. Server checks if account exists:
   - If exists: Link platform to existing account
   - If not: Create new account with platform authentication
6. Account created/accessed
```

## Server Segmentation

### Server Structure

**Description:**
Characters are segmented by server. Each server manages its own player authentication and character data.

**Server Model:**
```rust
pub struct Server {
    pub server_id: ServerId,
    pub server_name: String,
    pub server_type: ServerType,
    pub max_characters_per_account: u8,
    pub is_active: bool,
    pub maintenance_mode: bool,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ServerType {
    PvP,        // Player vs Player focused
    PvE,        // Player vs Environment focused
    Roleplay,   // Roleplay server
    Test,       // Test/Development server
}
```

### Character Segmentation

**How It Works:**
- Each account can have characters on multiple servers
- Characters are stored per server
- Server selection happens before character selection
- Each server maintains its own character database
- Characters cannot directly access other servers

**Character Storage:**
```
Account
├── Server 1
│   ├── Character 1
│   ├── Character 2
│   └── Character 3
├── Server 2
│   ├── Character 1
│   └── Character 2
└── Server 3
    └── Character 1
```

## Authentication Flow

### Client-Server Authentication Process

**Step 1: Account Authentication**
```
1. Player launches game client
2. Client shows login screen
3. Player selects authentication method:
   - Website: Enter email/password
   - Steam: Click "Login with Steam"
   - Epic Games: Click "Login with Epic Games"
4. Client sends authentication request to central auth server
5. Central auth server validates credentials
6. Returns account information and available servers
```

**Step 2: Server Selection**
```
1. Client displays list of available servers
2. Player selects desired server
3. Client requests server-specific authentication
4. Selected server validates account
5. Server returns character list for that account
```

**Step 3: Character Selection**
```
1. Client displays characters available on selected server
2. Player selects character or creates new one
3. Client requests character data from server
4. Server loads character data
5. Player enters game world
```

### Implementation Flow

```rust
// Step 1: Account Authentication
pub async fn authenticate_account(
    auth_method: AuthenticationMethod,
    credentials: AuthCredentials,
) -> Result<AccountInfo, AuthError> {
    match auth_method {
        AuthenticationMethod::Website => {
            authenticate_website(credentials.email, credentials.password).await
        }
        AuthenticationMethod::Steam => {
            authenticate_steam(credentials.steam_token).await
        }
        AuthenticationMethod::EpicGames => {
            authenticate_epic_games(credentials.epic_token).await
        }
    }
}

// Step 2: Server Authentication
pub async fn authenticate_server(
    account_id: AccountId,
    server_id: ServerId,
) -> Result<ServerAuthResult, AuthError> {
    // Validate account has access to server
    let account = get_account(account_id).await?;
    
    // Check server availability
    let server = get_server(server_id).await?;
    if !server.is_active || server.maintenance_mode {
        return Err(AuthError::ServerUnavailable);
    }
    
    // Get characters for this account on this server
    let characters = get_characters_for_account(account_id, server_id).await?;
    
    Ok(ServerAuthResult {
        server_id,
        characters,
        max_characters: server.max_characters_per_account,
    })
}

// Step 3: Character Selection
pub async fn select_character(
    account_id: AccountId,
    server_id: ServerId,
    character_id: CharacterId,
) -> Result<CharacterData, AuthError> {
    // Validate character belongs to account and server
    let character = get_character(character_id).await?;
    
    if character.account_id != account_id {
        return Err(AuthError::InvalidCharacter);
    }
    
    if character.server_id != server_id {
        return Err(AuthError::CharacterNotOnServer);
    }
    
    // Load character data
    load_character_data(character_id).await
}
```

## Character Management

### Character Creation

**Process:**
1. Player selects server
2. Player clicks "Create Character"
3. Player fills character creation form:
   - Character name (unique per server)
   - Character appearance
   - Starting class/attributes
4. Server validates character name
5. Server checks character limit per account
6. Character created and linked to account
7. Character appears in character list

**Character Limits:**
- Each server has maximum characters per account
- Default: 3-5 characters per server
- Can be increased with premium/subscription
- Limits enforced per server, not globally

### Character Deletion

**Process:**
1. Player selects character to delete
2. Player confirms deletion
3. Server checks deletion requirements:
   - Character not in use
   - No pending transactions
   - Account has permission
4. Character marked for deletion
5. Deletion delay (7 days) for recovery
6. Character permanently deleted after delay

**Deletion Safety:**
- 7-day recovery period
- Character data backed up
- Can be restored by support
- Permanent deletion after delay

## Character Transfer Between Servers

### Current Status

**Not Available:**
- Character transfer is not currently implemented
- This is a planned feature for the future
- Transfer will be complex and non-trivial

### Planned Implementation

**Complexity Factors:**
1. **Data Migration**: Character data must be migrated between servers
2. **Item Compatibility**: Items may not exist on target server
3. **Server-Specific Data**: Some data is server-specific
4. **Economy Balance**: Transferring items/gold affects economy
5. **Guild/Party Data**: Social connections are server-specific
6. **Quest Progress**: Quests may differ between servers
7. **Achievement Data**: Achievements may be server-specific

**Planned Transfer Process:**
```
1. Player requests character transfer
2. Server validates transfer eligibility
3. Character data exported from source server
4. Data validated and sanitized
5. Character imported to target server
6. Conflicts resolved (name, items, etc.)
7. Character available on target server
8. Original character removed from source server
```

**Transfer Restrictions (Planned):**
- Cooldown period between transfers
- Transfer cost (gold or premium currency)
- Level requirements
- Item restrictions (some items cannot transfer)
- Guild/party must be left before transfer
- Active quests may be reset

**Implementation Challenges:**
- Ensuring data integrity
- Handling server-specific content
- Maintaining game balance
- Preventing exploits
- Managing economy impact
- Testing across all scenarios

**Note:** This feature is planned but will require significant development time and testing. It is not a trivial feature and will be implemented carefully to ensure game balance and data integrity.

## Account Security

### Password Security

**Hashing:**
- Passwords are hashed using bcrypt or Argon2
- Salt is unique per account
- Password never stored in plain text
- Password reset requires email verification

**Password Requirements:**
- Minimum 8 characters
- Must contain uppercase, lowercase, number
- Special characters recommended
- Cannot be common passwords

### Account Protection

**Security Measures:**
- Account lockout after failed login attempts
- IP-based rate limiting
- Two-factor authentication (optional)
- Email verification for account changes
- Session management and timeout
- Secure token generation

**Account Recovery:**
- Password reset via email
- Account recovery questions (optional)
- Support ticket system
- Email verification required

### Ban and Suspension System

**Account Bans:**
- Permanent account ban
- Reason recorded
- Banned by admin recorded
- Ban timestamp recorded
- Cannot be appealed automatically

**Account Suspensions:**
- Temporary account suspension
- Suspension duration specified
- Reason recorded
- Account reactivates automatically

## Integration with Game Systems

### Character Data

**Per-Server Character Data:**
- Character stats and attributes
- Inventory and equipment
- Skills and progression
- Quest progress
- Achievement data
- Guild membership
- Party membership

**Account-Level Data:**
- Account settings
- Friend list (cross-server)
- Block list (cross-server)
- Account preferences
- Purchase history
- Subscription status

### Server Communication

**Central Auth Server:**
- Handles account authentication
- Manages account data
- Coordinates server selection
- Provides server list

**Game Servers:**
- Handle server-specific authentication
- Manage character data
- Process game logic
- Maintain server state

**Communication Flow:**
```
Client <-> Central Auth Server (Account Auth)
Client <-> Game Server (Character Selection)
Central Auth Server <-> Game Server (Account Validation)
```

## Rust Implementation Considerations

### Data Structures

```rust
pub struct Account {
    pub account_id: AccountId,
    pub email: String,
    pub username: String,
    pub authentication_methods: Vec<AuthenticationMethod>,
    pub created_at: DateTime<Utc>,
    pub last_login: Option<DateTime<Utc>>,
    pub account_status: AccountStatus,
}

pub struct Character {
    pub character_id: CharacterId,
    pub account_id: AccountId,
    pub server_id: ServerId,
    pub character_name: String,
    pub created_at: DateTime<Utc>,
    pub last_played: Option<DateTime<Utc>>,
    pub character_data: CharacterData,
}

pub struct ServerAuth {
    pub server_id: ServerId,
    pub account_id: AccountId,
    pub characters: Vec<CharacterId>,
    pub max_characters: u8,
}
```

### Authentication Services

```rust
pub struct AuthService {
    // Account authentication
    pub async fn authenticate(
        &self,
        method: AuthenticationMethod,
        credentials: AuthCredentials,
    ) -> Result<AccountInfo, AuthError>;
    
    // Server authentication
    pub async fn authenticate_server(
        &self,
        account_id: AccountId,
        server_id: ServerId,
    ) -> Result<ServerAuthResult, AuthError>;
    
    // Character selection
    pub async fn select_character(
        &self,
        account_id: AccountId,
        server_id: ServerId,
        character_id: CharacterId,
    ) -> Result<CharacterData, AuthError>;
}
```

## Testing Considerations

### Unit Tests

**Test Cases:**
- Account creation (all methods)
- Account authentication (all methods)
- Password hashing and validation
- Server authentication
- Character creation and deletion
- Character list retrieval

### Integration Tests

**Test Cases:**
- Full authentication flow
- Server selection process
- Character selection process
- Cross-platform authentication
- Account linking (Steam/Epic to existing account)
- Account security (lockout, rate limiting)

### Security Tests

**Test Cases:**
- Password security (hashing, validation)
- Authentication token security
- Session management
- Rate limiting effectiveness
- Account lockout functionality
- SQL injection prevention
- XSS prevention

## Summary

The account system:

- **Supports Multiple Authentication Methods**: Website, Steam, Epic Games
- **Manages Account Data**: Email, username, authentication methods, status
- **Segments Characters by Server**: Each server manages its own characters
- **Provides Server Selection**: Players choose server before character selection
- **Handles Server Authentication**: Each server validates account access
- **Manages Character Lists**: Characters are retrieved per server
- **Plans Character Transfer**: Complex feature planned for future
- **Ensures Security**: Password hashing, account protection, ban system

**Key Features:**
- **Flexibility**: Multiple authentication methods
- **Segmentation**: Server-based character management
- **Security**: Robust authentication and protection
- **Scalability**: Supports multiple servers and characters
- **Future-Proof**: Plans for character transfer

**Authentication Flow:**
1. Account authentication (Website/Steam/Epic)
2. Server selection
3. Server authentication
4. Character selection
5. Enter game world

This system ensures secure account management while providing flexibility for players to access multiple characters across different servers. Character transfer between servers is planned but will be a complex, carefully implemented feature to maintain game balance and data integrity.

