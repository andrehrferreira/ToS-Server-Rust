# Security and Permissions System

## Overview

The security and permissions system provides fine-grained access control for administrative functions. All actions require appropriate permissions, and access is logged for audit purposes.

## Permission Levels

### Permission Hierarchy

1. **Super Admin**: Full access to all features
2. **Admin**: Server management, player management, content management
3. **Moderator**: Player management, support tickets, log viewing
4. **Support**: Support tickets, log viewing (read-only)

## Permission System

**Permission Checks:**
```rust
pub struct PermissionSystem {
    admin_permissions: Arc<RwLock<HashMap<AdminId, Vec<Permission>>>>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Permission {
    // Server Management
    RestartServers,
    ViewTelemetry,
    RecoverBackups,
    
    // Player Management
    BanAccounts,
    UnbanAccounts,
    ViewPlayerData,
    ModifyPlayerData,
    ViewChat,
    
    // Content Management
    CreateItems,
    ModifyItems,
    CreateRespawns,
    RemoveRespawns,
    CreateCreatures,
    CreateNPCs,
    
    // Support
    RespondTickets,
    AssignTickets,
    
    // Rewards
    CreateRewards,
    DistributeRewards,
    
    // Logs
    ViewLogs,
    ExportLogs,
}

pub async fn check_permission(
    admin_id: AdminId,
    permission: Permission,
) -> Result<(), PermissionError> {
    let permissions = get_admin_permissions(admin_id).await?;
    
    if permissions.contains(&permission) {
        Ok(())
    } else {
        Err(PermissionError::InsufficientPermissions)
    }
}
```

## Admin Authentication

### Endpoint: `POST /admin/auth/login`

**Request:**
```json
{
  "admin_id": "admin_001",
  "password": "hashed_password"
}
```

**Response:**
```json
{
  "admin_token": "admin_token_xyz",
  "permissions": ["RestartServers", "BanAccounts", "CreateItems"],
  "expires_at": "2024-01-01T01:00:00Z"
}
```

**Implementation:**
```rust
pub async fn authenticate_admin(
    admin_id: AdminId,
    password: String,
) -> Result<AdminAuthResponse, AuthError> {
    // Validate credentials
    let admin = validate_admin_credentials(admin_id, password).await?;
    
    // Get admin permissions
    let permissions = get_admin_permissions(admin_id).await?;
    
    // Generate admin token
    let admin_token = generate_admin_token(admin_id).await?;
    let expires_at = Utc::now() + chrono::Duration::hours(8);
    
    Ok(AdminAuthResponse {
        admin_token,
        permissions,
        expires_at,
    })
}
```

## Permission Enforcement

### API Endpoint Protection

All API endpoints check permissions before executing:

```rust
pub async fn protected_endpoint(
    admin_token: AdminToken,
    permission: Permission,
    handler: impl Fn() -> Result<Response, Error>,
) -> Result<Response, Error> {
    // Validate admin token
    let admin_id = validate_admin_token(admin_token).await?;
    
    // Check permission
    check_permission(admin_id, permission).await?;
    
    // Execute handler
    handler()
}
```

## Audit Logging

**All Actions Logged:**
- Admin ID
- Action performed
- Target (server, player, item, etc.)
- Timestamp
- Result (success/failure)

**Implementation:**
```rust
pub struct AuditLogger {
    log_storage: LogStorage,
}

impl AuditLogger {
    pub async fn log_action(
        &self,
        admin_id: AdminId,
        action: AdminAction,
        target: Option<String>,
        result: ActionResult,
    ) {
        let audit_entry = AuditEntry {
            admin_id,
            action,
            target,
            result,
            timestamp: Utc::now(),
        };
        
        self.log_storage.write_audit_entry(audit_entry).await;
    }
}
```

## Summary

The security and permissions system provides:

- **Fine-Grained Permissions**: Specific permissions for each action
- **Permission Levels**: Hierarchical permission system
- **Token-Based Auth**: Secure admin authentication
- **Access Control**: All actions require appropriate permissions
- **Audit Logging**: All actions logged for accountability

This system ensures that only authorized administrators can perform specific actions, maintaining security and accountability.

