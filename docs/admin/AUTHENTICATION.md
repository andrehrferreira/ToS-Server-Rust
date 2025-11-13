# Authentication and Token System

## Overview

The authentication system handles player account authentication and generates secure tokens for game server access. Players have two tokens: a login token for account authentication and a character token for game server access.

## Token Types

### Login Token

**Purpose:** Validates player account authentication.

**Generated:** When player authenticates account (website, Steam, Epic Games)

**Valid For:** 1 hour

**Usage:** Used to select character and generate character token

### Character Token

**Purpose:** Validates player access to game server with selected character.

**Generated:** When player selects character and enters game server

**Valid For:** 24 hours

**Usage:** Used for all game server operations

**Revalidation:** Must be revalidated periodically on game server

## Authentication Flow

### Step 1: Account Authentication

**Endpoint:** `POST /auth/authenticate`

**Request:**
```json
{
  "account_id": "account_123",
  "authentication_method": "website",
  "credentials": {
    "email": "player@example.com",
    "password": "hashed_password"
  }
}
```

**Response:**
```json
{
  "login_token": "login_token_abc123",
  "token_expires_at": "2024-01-01T01:00:00Z",
  "account_info": {
    "account_id": "account_123",
    "username": "PlayerName",
    "characters": [
      {
        "character_id": "char_001",
        "server_id": "server_001",
        "character_name": "CharacterName",
        "level": 50
      }
    ]
  }
}
```

**Implementation:**
```rust
pub async fn authenticate_account(
    account_id: AccountId,
    auth_method: AuthenticationMethod,
    credentials: AuthCredentials,
) -> Result<AuthResponse, AuthError> {
    // Validate credentials
    let account = validate_credentials(account_id, auth_method, credentials).await?;
    
    // Generate login token
    let login_token = generate_login_token(account_id).await?;
    let token_expires_at = Utc::now() + chrono::Duration::hours(1);
    
    // Get account characters
    let characters = get_account_characters(account_id).await?;
    
    Ok(AuthResponse {
        login_token,
        token_expires_at,
        account_info: AccountInfo {
            account_id,
            username: account.username,
            characters,
        },
    })
}
```

### Step 2: Character Selection

**Endpoint:** `POST /auth/select-character`

**Request:**
```json
{
  "login_token": "login_token_abc123",
  "character_id": "char_001",
  "server_id": "server_001"
}
```

**Response:**
```json
{
  "character_token": "character_token_xyz789",
  "token_expires_at": "2024-01-01T02:00:00Z",
  "character_data": {
    "character_id": "char_001",
    "character_name": "CharacterName",
    "level": 50,
    "position": {"x": 1000.0, "y": 2000.0, "z": 500.0},
    "map_id": "map_001"
  }
}
```

**Implementation:**
```rust
pub async fn select_character(
    login_token: LoginToken,
    character_id: CharacterId,
    server_id: ServerId,
) -> Result<CharacterAuthResponse, AuthError> {
    // Validate login token
    let account_id = validate_login_token(login_token).await?;
    
    // Verify character belongs to account
    verify_character_ownership(account_id, character_id).await?;
    
    // Generate character token
    let character_token = generate_character_token(character_id, server_id).await?;
    let token_expires_at = Utc::now() + chrono::Duration::hours(24);
    
    // Load character data
    let character_data = load_character_data(character_id).await?;
    
    Ok(CharacterAuthResponse {
        character_token,
        token_expires_at,
        character_data,
    })
}
```

### Step 3: Token Revalidation

**Endpoint:** `POST /auth/revalidate`

**Purpose:** Revalidate tokens on game server.

**Request:**
```json
{
  "token": "character_token_xyz789",
  "token_type": "character"
}
```

**Response:**
```json
{
  "valid": true,
  "token_info": {
    "character_id": "char_001",
    "account_id": "account_123",
    "expires_at": "2024-01-01T02:00:00Z"
  }
}
```

**Implementation:**
```rust
pub async fn revalidate_token(
    token: String,
    token_type: TokenType,
) -> Result<TokenValidation, AuthError> {
    match token_type {
        TokenType::Login => {
            let account_id = validate_login_token(token).await?;
            Ok(TokenValidation {
                valid: true,
                token_info: TokenInfo::Login { account_id },
            })
        }
        TokenType::Character => {
            let (character_id, account_id) = validate_character_token(token).await?;
            Ok(TokenValidation {
                valid: true,
                token_info: TokenInfo::Character {
                    character_id,
                    account_id,
                },
            })
        }
    }
}
```

## Token Generation

### Login Token Generation

```rust
pub async fn generate_login_token(account_id: AccountId) -> Result<LoginToken, TokenError> {
    let token = generate_secure_token(32);
    let expires_at = Utc::now() + chrono::Duration::hours(1);
    
    // Store token in database
    store_token(Token {
        token: token.clone(),
        account_id,
        token_type: TokenType::Login,
        expires_at,
        created_at: Utc::now(),
    }).await?;
    
    Ok(token)
}
```

### Character Token Generation

```rust
pub async fn generate_character_token(
    character_id: CharacterId,
    server_id: ServerId,
) -> Result<CharacterToken, TokenError> {
    let token = generate_secure_token(32);
    let expires_at = Utc::now() + chrono::Duration::hours(24);
    
    // Store token in database
    store_token(Token {
        token: token.clone(),
        character_id: Some(character_id),
        server_id: Some(server_id),
        token_type: TokenType::Character,
        expires_at,
        created_at: Utc::now(),
    }).await?;
    
    Ok(token)
}
```

## Token Validation

### Login Token Validation

```rust
pub async fn validate_login_token(token: LoginToken) -> Result<AccountId, AuthError> {
    // Check token exists and is valid
    let token_data = get_token(token).await?;
    
    // Check expiration
    if token_data.expires_at < Utc::now() {
        return Err(AuthError::TokenExpired);
    }
    
    // Check token type
    if token_data.token_type != TokenType::Login {
        return Err(AuthError::InvalidTokenType);
    }
    
    Ok(token_data.account_id)
}
```

### Character Token Validation

```rust
pub async fn validate_character_token(
    token: CharacterToken,
) -> Result<(CharacterId, AccountId), AuthError> {
    // Check token exists and is valid
    let token_data = get_token(token).await?;
    
    // Check expiration
    if token_data.expires_at < Utc::now() {
        return Err(AuthError::TokenExpired);
    }
    
    // Check token type
    if token_data.token_type != TokenType::Character {
        return Err(AuthError::InvalidTokenType);
    }
    
    let character_id = token_data.character_id
        .ok_or(AuthError::InvalidToken)?;
    
    let account_id = get_character_account(character_id).await?;
    
    Ok((character_id, account_id))
}
```

## Security Considerations

### Token Storage

- Tokens stored in PostgreSQL database
- Hashed tokens for security
- Expiration tracking
- Automatic cleanup of expired tokens

### Token Rotation

- Tokens can be rotated for security
- Old tokens invalidated on rotation
- New tokens generated seamlessly

### Token Revocation

- Tokens can be revoked by admins
- Immediate invalidation
- Logged for audit purposes

## Summary

The authentication system provides:

- **Secure Token Generation**: Cryptographically secure tokens
- **Two-Token System**: Login token and character token
- **Token Validation**: Secure validation on game servers
- **Token Revalidation**: Periodic revalidation for security
- **Token Management**: Storage, expiration, and revocation

This system ensures secure player authentication while maintaining performance and usability.

