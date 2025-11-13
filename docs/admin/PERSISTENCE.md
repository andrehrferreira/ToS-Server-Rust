# Data Persistence

## Overview

The central API uses PostgreSQL for data storage with eventual consistency synchronization across read replicas. All configuration data, account data, and administrative records are stored in PostgreSQL.

## PostgreSQL Database

### Schema Design

- All configuration data stored in PostgreSQL
- Normalized schema for data integrity
- Indexes for query performance
- Foreign keys for referential integrity

### Connection Pooling

- Primary pool for write operations
- Read replica pool for read operations
- Connection limits per pool
- Automatic connection recovery

**Implementation:**
```rust
pub struct DatabaseManager {
    primary_pool: PgPool,
    read_replica_pools: Vec<PgPool>,
    current_read_index: AtomicUsize,
}

impl DatabaseManager {
    pub async fn write_query(&self, query: &str) -> Result<(), DbError> {
        // Execute on primary database
        sqlx::query(query)
            .execute(&self.primary_pool)
            .await?;
        Ok(())
    }
    
    pub async fn read_query(&self, query: &str) -> Result<Vec<Row>, DbError> {
        // Execute on read replica (round-robin)
        let index = self.current_read_index.fetch_add(1, Ordering::Relaxed);
        let pool = &self.read_replica_pools[index % self.read_replica_pools.len()];
        
        sqlx::query(query)
            .fetch_all(pool)
            .await
    }
}
```

## Eventual Consistency

### Synchronization Strategy

- Write operations go to primary database
- Asynchronous replication to read replicas
- Eventual consistency (replicas may lag slightly)
- Conflict resolution for concurrent writes

**Implementation:**
```rust
pub struct ReplicationManager {
    primary_db: PgPool,
    replicas: Vec<ReplicaConfig>,
}

pub struct ReplicaConfig {
    pub replica_id: ReplicaId,
    pub connection_string: String,
    pub lag_threshold: Duration,
    pub last_sync: Instant,
}

impl ReplicationManager {
    pub async fn sync_to_replicas(&self) {
        // PostgreSQL handles replication automatically
        // Monitor replication lag
        for replica in &self.replicas {
            let lag = self.check_replication_lag(replica).await;
            if lag > replica.lag_threshold {
                warn!("Replica {} lagging: {:?}", replica.replica_id, lag);
            }
        }
    }
    
    async fn check_replication_lag(&self, replica: &ReplicaConfig) -> Duration {
        // Query replication lag from PostgreSQL
        // Returns time difference between primary and replica
        Duration::from_millis(0) // Simplified
    }
}
```

## Data Models

### Configuration Data

- Items
- NPCs
- Quests
- Zones
- Events
- Patrols
- Creatures

### Account Data

- Accounts
- Characters
- Tokens
- Permissions

### Administrative Data

- Admin accounts
- Audit logs
- Support tickets
- Reward campaigns

## Backup and Recovery

All PostgreSQL data is backed up regularly. See [BACKUP_SYSTEM.md](./BACKUP_SYSTEM.md) for backup details.

## Summary

The persistence system provides:

- **PostgreSQL Storage**: Reliable, scalable database
- **Read Replicas**: Distributed read operations
- **Eventual Consistency**: Efficient synchronization
- **Data Integrity**: Foreign keys and constraints
- **Performance**: Indexes and query optimization

This system ensures reliable data storage with high availability and performance.

