# PostgreSQL Database Backup Service

A Docker-based DB backup solution using [tiredofit/db-backup](https://github.com/tiredofit/docker-db-backup).


> **Multiple Database Support**: Each database (DB01, DB02, etc.) can specify its own parameters. If not specified, default values will be used.


## Backup Management

### Cleanup Policy

`CLEANUP_TIME` removes backup files older than specified minutes from the same path.

If there are errors during the backup job, the cleanup process will be skipped:

```
Skipping Cleaning up old backups because there were errors in backing up
```

> **⚠️ Important**: When a database is removed from the server, its old backup files will also be deleted during cleanup. To preserve backups:
> - Manually move old backups to a different path before database removal
> - Configure separate backup paths for each database

### Individual Database Paths

For better backup isolation, configure separate paths for each database:

```bash
# Database 1
DB01_FILESYSTEM_PATH=/backup/db1
DB01_CLEANUP_TIME=10080  # 7 days

# Database 2
DB02_FILESYSTEM_PATH=/backup/db2
DB02_CLEANUP_TIME=20160  # 14 days
```

## Manual Operations

### Manual Backups

Manual backups can be performed by entering the container:

```bash
# Execute all scheduled backup tasks
docker exec -it <container-name> backup-now

# Execute specific database job (replace 01 with your DB number)
docker exec -it <container-name> backup01-now
```

> **Note**: Jobs execute sequentially without concurrency.

### Restore Operations

To restore from backup files:

```bash
# Enter container
docker exec -it <container-name> bash

# Restore command syntax
restore <filename> <db_type> <db_hostname> <db_name> <db_user> <db_pass> <db_port>

# Example
restore backup_20231201_093000.sql.zst pgsql localhost mydb postgres mypass 5432
```

## CI Settings

### GitLab CI/CD Configuration

To set up automated deployment with GitLab CI/CD:

1. **Configure Environment Variables**:
   - Set up a GitLab CI/CD variable named `COMPOSE_ENV_FILES`
   - Set the variable type to `file`
   - Add your environment file content as the value

2. **How it works**:
   - GitLab CI automatically saves the variable value to a temporary file
   - The `COMPOSE_ENV_FILES` environment variable is set to the file path (e.g., `/builds/rd/docker-db-backup.tmp/COMPOSE_ENV_FILES`)
   - Docker Compose automatically reads the environment variables from this file path
   - No additional configuration is needed in the CI YAML file
