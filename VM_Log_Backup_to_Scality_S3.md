# VM Log Backup to Scality S3 (Using AWS CLI)

This document explains how to **backup VM logs to Scality S3-compatible
storage** using AWS CLI v2 on Ubuntu.
It includes **common fixes**, **best practices**, and **cron
scheduling**.

------------------------------------------------------------------------

## Requirements

-   Scality S3 credentials
    -   Access Key
    -   Secret Key
    -   Scality S3 Endpoint URL
-   Ubuntu VM (20.04 / 22.04 recommended)
-   Root or sudo access
-   Internet access (or internal access to Scality endpoint)

------------------------------------------------------------------------

## Step 1: Install AWS CLI v2 on Ubuntu

``` bash
sudo apt update
sudo apt install -y unzip curl
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws
```

Verify installation:

``` bash
aws --version
```

------------------------------------------------------------------------

## Step 2: Configure AWS CLI for Scality

### 2.1 Create AWS Profile

``` bash
aws configure --profile scality
```

Enter: - **Access Key ID** → Scality access key
- **Secret Access Key** → Scality secret key
- **Default region name** → `us-east-1`
- **Default output format** → `json`

> Region is logical for Scality but must be explicitly set.

------------------------------------------------------------------------

### 2.2 Configure Scality Endpoint (IMPORTANT)

Edit config:

``` bash
nano ~/.aws/config
```

Add:

``` ini
[profile scality]
region = us-east-1
output = json
endpoint_url = https://s3.your-scality-domain.com
s3 =
  checksum_mode = disabled
```

This disables **trailing checksum**, which is unsupported by many
Scality versions.

------------------------------------------------------------------------

### 2.3 Test Connectivity

``` bash
aws s3 ls --profile scality
```

Optional test bucket:

``` bash
aws s3 mb s3://test-bucket --profile scality
```

------------------------------------------------------------------------

## Step 3: Backup Script (Improved & Scality-Compatible)

Create script:

``` bash
nano backup-logs-to-s3.sh
```

### Script Content

``` bash
#!/bin/bash

set -e

# Configuration
LOG_DIR="/var/log"
S3_BUCKET="s3://my-vm-logs-backup"
BACKUP_NAME="vm-logs"
TEMP_DIR="/tmp/log_backup"
LOG_FILE="/var/log/s3_backup.log"
AWS_PROFILE="scality"

mkdir -p "$TEMP_DIR"

TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
ARCHIVE="${TEMP_DIR}/${BACKUP_NAME}-${TIMESTAMP}.tar.gz"

echo "[$(date)] Starting backup" >> "$LOG_FILE"

# Compress logs
tar -czf "$ARCHIVE" -C "$LOG_DIR" .

echo "[$(date)] Compression completed" >> "$LOG_FILE"

# Upload to Scality
aws s3 cp "$ARCHIVE"   "${S3_BUCKET}/${TIMESTAMP}/"   --profile "$AWS_PROFILE"   --checksum-algorithm none

echo "[$(date)] Upload successful" >> "$LOG_FILE"

# Cleanup
rm -f "$ARCHIVE"

echo "[$(date)] Backup completed successfully" >> "$LOG_FILE"
```

Make executable:

``` bash
chmod +x backup-logs-to-s3.sh
```

Run manually:

``` bash
./backup-logs-to-s3.sh
```

------------------------------------------------------------------------

## Step 4: Verify Upload

``` bash
aws s3 ls s3://my-vm-logs-backup/ --profile scality
```

------------------------------------------------------------------------

## Step 5: Schedule Daily Backup (Cron)

Edit cron:

``` bash
crontab -e
```

Run daily at **2 AM**:

``` bash
0 2 * * * /full/path/to/backup-logs-to-s3.sh
```

------------------------------------------------------------------------

## Common Errors & Fixes

### ❌ trailing checksum is not supported

**Cause:** AWS CLI v2 checksum feature
**Fix:**
- `--checksum-algorithm none` OR
- `checksum_mode = disabled` in AWS config

------------------------------------------------------------------------

### ❌ InvalidLocationConstraint

**Cause:** Missing or invalid region\
**Fix:** Use `us-east-1` explicitly

------------------------------------------------------------------------

### ❌ Could not connect to endpoint

**Cause:** Endpoint URL missing or wrong\
**Fix:** Verify `endpoint_url` in `~/.aws/config`


------------------------------------------------------------------------

## Conclusion

This setup provides a **stable, Scality-compatible, automated VM log
backup solution** using AWS CLI.

------------------------------------------------------------------------

**Author:** Anant Gadaili
**Format:** Markdown (.md)
