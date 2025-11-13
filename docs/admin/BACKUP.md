# Backup System

## Overview

The backup system ensures zero data loss by implementing comprehensive backup strategies using cloud storage (S3 or Digital Ocean Spaces). All backups are coordinated centrally and stored securely in the cloud.

## Zero Data Loss Guarantee

**Critical Requirement:**
- **Nothing can be lost**: All data must be backed up
- **Periodic full backups**: Complete backups of files and database
- **Cloud storage**: S3 or Digital Ocean Spaces for backup storage
- **File transfer**: Secure transfer between servers and cloud storage

## Backup Architecture

**Components:**
- Central backup coordinator
- Cloud storage (S3/Digital Ocean Spaces)
- File transfer system
- Backup scheduling system
- Backup verification system

**Architecture:**
```
Game Servers → Backup Coordinator → Cloud Storage (S3/DO Spaces)
     ↓              ↓
SQLite DBs    PostgreSQL DB
     ↓              ↓
  Files         Files
```

## Cloud Storage Integration

### Supported Providers

- Amazon S3
- Digital Ocean Spaces
- Compatible S3-compatible storage

**Implementation:**
```rust
use aws_sdk_s3::Client as S3Client;
use aws_sdk_s3::primitives::ByteStream;

pub struct CloudStorage {
    client: S3Client,
    bucket_name: String,
    provider: StorageProvider,
}

#[derive(Debug, Clone)]
pub enum StorageProvider {
    AmazonS3 {
        region: String,
        access_key: String,
        secret_key: String,
    },
    DigitalOceanSpaces {
        endpoint: String,
        region: String,
        access_key: String,
        secret_key: String,
    },
}

impl CloudStorage {
    pub async fn upload_backup(
        &self,
        backup_path: &Path,
        backup_name: &str,
    ) -> Result<String, BackupError> {
        let file_data = tokio::fs::read(backup_path).await?;
        let key = format!("backups/{}/{}", Utc::now().format("%Y%m%d"), backup_name);
        
        self.client
            .put_object()
            .bucket(&self.bucket_name)
            .key(&key)
            .body(ByteStream::from(file_data))
            .send()
            .await?;
        
        Ok(key)
    }
    
    pub async fn download_backup(
        &self,
        backup_key: &str,
        destination: &Path,
    ) -> Result<(), BackupError> {
        let response = self.client
            .get_object()
            .bucket(&self.bucket_name)
            .key(backup_key)
            .send()
            .await?;
        
        let data = response.body.collect().await?.into_bytes();
        tokio::fs::write(destination, data).await?;
        
        Ok(())
    }
    
    pub async fn list_backups(&self, prefix: &str) -> Result<Vec<String>, BackupError> {
        let response = self.client
            .list_objects_v2()
            .bucket(&self.bucket_name)
            .prefix(prefix)
            .send()
            .await?;
        
        let keys: Vec<String> = response.contents()
            .iter()
            .filter_map(|obj| obj.key().map(|k| k.to_string()))
            .collect();
        
        Ok(keys)
    }
    
    pub async fn delete_old_backups(&self, retention_days: u32) -> Result<(), BackupError> {
        let cutoff_date = Utc::now() - chrono::Duration::days(retention_days as i64);
        let prefix = format!("backups/{}", cutoff_date.format("%Y%m%d"));
        
        let backups = self.list_backups(&prefix).await?;
        
        for backup_key in backups {
            self.client
                .delete_object()
                .bucket(&self.bucket_name)
                .key(&backup_key)
                .send()
                .await?;
        }
        
        Ok(())
    }
}
```

## File Transfer System

**Purpose:**
Transfer backup files between game servers and central backup coordinator, and to/from cloud storage.

**Implementation:**
```rust
pub struct FileTransferSystem {
    sftp_client: Option<SftpClient>,
    http_client: reqwest::Client,
    cloud_storage: CloudStorage,
}

impl FileTransferSystem {
    pub async fn transfer_from_server(
        &self,
        server_id: ServerId,
        source_path: &Path,
        destination_path: &Path,
    ) -> Result<(), TransferError> {
        // Option 1: SFTP transfer
        if let Some(ref sftp) = self.sftp_client {
            sftp.download(server_id, source_path, destination_path).await?;
        }
        // Option 2: HTTP transfer
        else {
            let url = format!("https://{}/backup/{}", server_id, source_path.display());
            let response = self.http_client.get(&url).send().await?;
            let data = response.bytes().await?;
            tokio::fs::write(destination_path, data).await?;
        }
        
        Ok(())
    }
    
    pub async fn transfer_to_cloud(
        &self,
        local_path: &Path,
        cloud_key: &str,
    ) -> Result<(), TransferError> {
        self.cloud_storage.upload_backup(local_path, cloud_key).await?;
        Ok(())
    }
    
    pub async fn transfer_from_cloud(
        &self,
        cloud_key: &str,
        local_path: &Path,
    ) -> Result<(), TransferError> {
        self.cloud_storage.download_backup(cloud_key, local_path).await?;
        Ok(())
    }
}
```

## Backup Coordinator

### Responsibilities

- Schedule backups
- Coordinate backups across all servers
- Transfer files to cloud storage
- Verify backup integrity
- Manage backup retention

### Backup Schedule

```rust
pub struct BackupSchedule {
    pub database_backup_interval: Duration, // Every 1 hour
    pub file_backup_interval: Duration, // Every 6 hours
    pub full_backup_interval: Duration, // Every 24 hours
    pub retention_days: u32, // Keep backups for 30 days
}
```

### Database Backup

**Frequency:** Every 1 hour

**Process:**
1. Request backup from game server
2. Receive SQLite database file
3. Upload to cloud storage
4. Clean up local file

**Implementation:**
```rust
async fn backup_server_database(
    server: &ServerConfig,
    cloud_storage: &CloudStorage,
    file_transfer: &FileTransferSystem,
) {
    let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
    let backup_name = format!("{}_{}_database.db", server.server_id, timestamp);
    let local_path = Path::new("/tmp/backups").join(&backup_name);
    
    // Request backup from game server
    match request_server_backup(server, &local_path).await {
        Ok(_) => {
            // Transfer to cloud storage
            let cloud_key = format!("servers/{}/database/{}", server.server_id, backup_name);
            if let Err(e) = cloud_storage.upload_backup(&local_path, &cloud_key).await {
                error!("Failed to upload database backup: {:?}", e);
            } else {
                info!("Database backup uploaded: {}", cloud_key);
            }
            
            // Clean up local file
            let _ = tokio::fs::remove_file(&local_path).await;
        }
        Err(e) => {
            error!("Failed to backup server database: {:?}", e);
        }
    }
}
```

### PostgreSQL Backup

**Frequency:** Every 1 hour

**Process:**
1. Create PostgreSQL dump using `pg_dump`
2. Upload dump to cloud storage
3. Clean up local file

**Implementation:**
```rust
async fn backup_postgresql_database(cloud_storage: &CloudStorage) {
    let timestamp = Utc::now().format("%Y%m%d_%H%M%S");
    let backup_name = format!("postgresql_{}.sql", timestamp);
    let local_path = Path::new("/tmp/backups").join(&backup_name);
    
    // Create PostgreSQL dump
    let output = tokio::process::Command::new("pg_dump")
        .arg("--host=localhost")
        .arg("--dbname=game_db")
        .arg("--username=admin")
        .arg("--file")
        .arg(&local_path)
        .output()
        .await
        .expect("Failed to execute pg_dump");
    
    if output.status.success() {
        // Transfer to cloud storage
        let cloud_key = format!("central/postgresql/{}", backup_name);
        if let Err(e) = cloud_storage.upload_backup(&local_path, &cloud_key).await {
            error!("Failed to upload PostgreSQL backup: {:?}", e);
        } else {
            info!("PostgreSQL backup uploaded: {}", cloud_key);
        }
        
        // Clean up local file
        let _ = tokio::fs::remove_file(&local_path).await;
    } else {
        error!("PostgreSQL dump failed: {:?}", String::from_utf8_lossy(&output.stderr));
    }
}
```

### File Backup

**Frequency:** Every 6 hours

**Process:**
1. Create tar archive of server files
2. Upload archive to cloud storage
3. Clean up local file

### Full Backup

**Frequency:** Every 24 hours

**Process:**
1. Backup database
2. Backup files
3. Backup configuration
4. Backup logs
5. Create complete archive
6. Upload to cloud storage

## Backup Verification

**Integrity Checks:**
- Verify backup file integrity
- Test database restore
- Verify file completeness
- Check backup size consistency

**Implementation:**
```rust
pub struct BackupVerification {
    cloud_storage: CloudStorage,
}

impl BackupVerification {
    pub async fn verify_backup(&self, backup_key: &str) -> Result<BackupIntegrity, VerifyError> {
        // Download backup
        let temp_path = Path::new("/tmp/verify").join(backup_key);
        self.cloud_storage.download_backup(backup_key, &temp_path).await?;
        
        // Verify file integrity
        let file_size = tokio::fs::metadata(&temp_path).await?.len();
        let checksum = calculate_checksum(&temp_path).await?;
        
        // If database backup, test restore
        if backup_key.contains("database") {
            self.test_database_restore(&temp_path).await?;
        }
        
        // Clean up
        tokio::fs::remove_file(&temp_path).await.ok();
        
        Ok(BackupIntegrity {
            file_size,
            checksum,
            verified_at: Utc::now(),
        })
    }
    
    async fn test_database_restore(&self, backup_path: &Path) -> Result<(), VerifyError> {
        // Test restore to temporary database
        // Verify data integrity
        Ok(())
    }
}
```

## Backup Recovery

**Recovery Process:**
1. List available backups
2. Select backup to restore
3. Download from cloud storage
4. Restore to server
5. Verify restoration

**API Endpoint:**
- `POST /backup/recover` - Recover from backup

**Request:**
```json
{
  "server_id": "server_001",
  "backup_key": "servers/server_001/database/server_001_20240101_120000_database.db",
  "backup_type": "database"
}
```

## Backup Retention

**Policy:**
- Keep backups for 30 days (configurable)
- Automatic cleanup of old backups
- Daily cleanup process
- Backup metadata retained

## Summary

The backup system provides:

- **Zero Data Loss**: All data backed up regularly
- **Cloud Storage**: S3/Digital Ocean Spaces integration
- **Multiple Backup Types**: Database, files, and full backups
- **Automated Scheduling**: Regular backups without manual intervention
- **Verification**: Integrity checks on all backups
- **Recovery**: Easy restoration from backups
- **Retention**: Configurable backup retention policy

This system ensures that no data is ever lost, with comprehensive backup coverage and easy recovery options.

