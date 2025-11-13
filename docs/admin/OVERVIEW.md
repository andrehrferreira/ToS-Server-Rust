# Administrative System Overview

## Introduction

The administrative system provides a comprehensive web-based dashboard for game administrators to manage all aspects of the game servers, players, and content. The system includes a unified API for configuration preloading, authentication, and real-time server management. Administrators can perform all management tasks without needing to log into the game client.

The central API server is designed with replication to support high request volumes, uses PostgreSQL for data storage with eventual consistency synchronization, and implements a comprehensive backup system using cloud storage (S3 or Digital Ocean Spaces) to ensure zero data loss.

## Key Features

- **Unified Configuration API**: Central API for all server configuration data
- **Token-Based Authentication**: Secure token system for player authentication
- **Web-Based Dashboard**: Full administrative control via web interface
- **Real-Time Server Management**: Control and monitor servers in real-time
- **Player Management**: Complete player account and character management
- **Content Management**: Create and configure items, NPCs, quests, zones, events
- **Telemetry and Monitoring**: Real-time server metrics and monitoring
- **Log Analysis**: Comprehensive log viewing and analysis
- **Interactive Maps**: Visual representation of game world state
- **Backup System**: Comprehensive backup system with cloud storage
- **Security**: Permission-based access control

## System Components

### Central API Server
- High availability with replication
- PostgreSQL database with read replicas
- Eventual consistency synchronization
- See [ARCHITECTURE.md](./ARCHITECTURE.md) for details

### Authentication System
- Account authentication
- Token generation and validation
- Character selection tokens
- See [AUTHENTICATION.md](./AUTHENTICATION.md) for details

### Configuration API
- Server startup preload
- Content replication (items, NPCs, quests, zones, events, patrols, creatures, respawns)
- Configuration management
- Unified world structure across all servers
- See [CONFIG_API.md](./CONFIG_API.md) for details

### Administrative Dashboard
- Server management
- Player management
- Content management
- Interactive maps
- See [DASHBOARD.md](./DASHBOARD.md) for details

### Backup System
- Cloud storage integration
- Periodic backups
- File transfer system
- See [BACKUP_SYSTEM.md](./BACKUP_SYSTEM.md) for details

### Real-Time Updates
- Server-Sent Events (SSE)
- Event streaming
- Live monitoring
- See [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) for details

### Security and Permissions
- Permission system
- Access control
- Admin authentication
- See [PERMISSIONS.md](./PERMISSIONS.md) for details

## Documentation Structure

- [ARCHITECTURE.md](./ARCHITECTURE.md) - Central API server architecture
- [AUTHENTICATION.md](./AUTHENTICATION.md) - Authentication and token system
- [CONFIG_API.md](./CONFIG_API.md) - Unified configuration API
- [DASHBOARD.md](./DASHBOARD.md) - Administrative dashboard overview
- [SERVER_MANAGEMENT.md](./SERVER_MANAGEMENT.md) - Server management features
- [PLAYER_MANAGEMENT.md](./PLAYER_MANAGEMENT.md) - Player management features
- [CONTENT_MANAGEMENT.md](./CONTENT_MANAGEMENT.md) - Content management features
- [INTERACTIVE_MAP.md](./INTERACTIVE_MAP.md) - Interactive map system
- [SUPPORT_SYSTEM.md](./SUPPORT_SYSTEM.md) - Support ticket system
- [REWARD_SYSTEM.md](./REWARD_SYSTEM.md) - Reward distribution system
- [LOG_ANALYSIS.md](./LOG_ANALYSIS.md) - Log analysis and viewing
- [CHAT_MONITORING.md](./CHAT_MONITORING.md) - Chat monitoring system
- [BACKUP_SYSTEM.md](./BACKUP_SYSTEM.md) - Backup and recovery system
- [PERMISSIONS.md](./PERMISSIONS.md) - Security and permissions
- [REAL_TIME_UPDATES.md](./REAL_TIME_UPDATES.md) - Real-time updates via SSE
- [PERSISTENCE.md](./PERSISTENCE.md) - Data persistence and PostgreSQL

## Content Replication

**Important:** All game servers share the same world structure. The following content is replicated to all servers:

- **Maps**: Map definitions, boundaries, and properties
- **Items**: Item definitions and properties
- **NPCs**: NPCs, dialogues, and behaviors
- **Quests**: Quest definitions, requirements, and rewards
- **Zones**: Zone configurations, boundaries, and properties
- **Events**: Automatic event configurations
- **Patrols**: Creature patrol routes
- **Creatures**: Creature definitions and properties
- **Respawns**: All respawn points (both creature and resource respawns) - each associated with a map

**Map Management:** Each game server manages multiple maps. Maps must be registered in the admin dashboard and assigned to servers. Every respawn is associated with a specific map.

**Server-Specific Data:** Each server maintains its own player data, rankings, auction house, houses, guilds, parties, and map instances (with players, creatures, events, etc.).

## Summary

The administrative system provides comprehensive tools for managing all aspects of the game through a web-based interface, ensuring efficient operations, security, and zero data loss through robust backup systems. All content changes are automatically replicated to all game servers, ensuring identical world structure while maintaining independent player economies and communities per server.

