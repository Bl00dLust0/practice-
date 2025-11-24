# MinIO and Scality Integration -- Complete Documentation

## 1. What is MinIO?

MinIO is an open-source, high-performance object storage server designed
for cloud‑native applications. It provides a scalable, S3‑compatible
storage solution that can run across private clouds, public clouds,
on‑premise servers, and edge environments.

### Key Advantages

#### **1. S3 API Compatibility**

MinIO is fully compatible with the Amazon S3 API, allowing applications
built for S3 to work without modification. This prevents cloud vendor
lock‑in and gives full flexibility.

#### **2. High Performance**

MinIO is optimized for ultra‑high throughput and low latency, making it
ideal for: - AI/ML workloads\
- Big data analytics\
- High‑performance storage systems

#### **3. Kubernetes Native**

Designed for modern infrastructure, MinIO integrates seamlessly with
Kubernetes environments.

#### **4. Scalability & Resilience**

-   Horizontally scalable to exabyte levels\
-   Uses erasure coding\
-   Bitrot protection\
-   High durability and availability

#### **5. Hybrid & Multi‑Cloud**

MinIO can run on any hardware, cloud provider, or edge node---providing
a consistent storage layer across environments.

------------------------------------------------------------------------

## 2. Installing MinIO on Ubuntu

MinIO has two components:

### ✔ **MinIO Server**

The storage backend (not needed if using Scality as the storage system).

### ✔ **MinIO Client (mc)**

The command‑line tool used to interact with any S3‑compatible storage,
including: - Scality RING - Scality ARTESCA - AWS S3 - MinIO Server

### A. Install MinIO Client (mc)

``` bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
```

Check version:

``` bash
mc --version
```

------------------------------------------------------------------------

## 3. Adding Scality as an S3 Endpoint

Scality provides a fully S3‑compatible API. We can connect using:

``` bash
mc alias set scality https://<scality-endpoint> <access-key> <secret-key>
```

Verify:

``` bash
mc ls scality
```

If you see a list of buckets → **connection successful**.

------------------------------------------------------------------------

## 4. Using MinIO Client to Manage Scality Storage

### ✔ List buckets

``` bash
mc ls scality
```

### ✔ Upload a file

``` bash
mc cp file.txt scality/mybucket/
```

### ✔ Download a file

``` bash
mc cp scality/mybucket/file.txt .
```

### ✔ Create a bucket

``` bash
mc mb scality/newbucket
```

------------------------------------------------------------------------

## 5. Why MinIO Client Is Necessary for Scality

Although Scality provides an S3‑compatible API, it lacks a powerful,
user‑friendly CLI tool. MinIO Client (`mc`) fills this gap.

### MinIO Client provides:

-   Easy bucket management\
-   Folder synchronization (like `rsync`)\
-   Access policies\
-   Real‑time monitoring\
-   Cleaner syntax than AWS CLI\
-   Works with **ANY** S3 endpoint

------------------------------------------------------------------------

## 6. Detailed Feature Examples

### \##### 1. Easy Bucket Management

**List buckets**

``` bash
mc ls scality
```

**Create a bucket**

``` bash
mc mb scality/project-backup
```

**Remove a bucket**

``` bash
mc rb scality/old-bucket
```

**Force remove bucket**

``` bash
mc rb --force scality/old-bucket
```

**List objects**

``` bash
mc ls scality/project-backup
```

Example output:

    [2025-11-24 10:15:30]  5MiB config.json
    [2025-11-24 10:16:01]  1MiB app.log

------------------------------------------------------------------------

### \##### 2. Folder Sync (Like rsync for S3)

This is one of the best features of `mc`.

**Sync local folder → Scality bucket**

``` bash
mc mirror /var/www/uploads scality/backup-uploads
```

**Sync + delete removed files**

``` bash
mc mirror --remove /var/www/uploads scality/backup-uploads
```

**Sync Scality → local directory**

``` bash
mc mirror scality/project-data /data/local-copy
```

Useful for: - Backups\
- Disaster recovery\
- Automated syncing

------------------------------------------------------------------------

### \##### 3. Access Policies

You can configure: - Public access\
- Read‑only access\
- Private/no access

**Make bucket public**

``` bash
mc anonymous set public scality/media
```

**Read-only access**

``` bash
mc anonymous set download scality/docs
```

**Remove all access**

``` bash
mc anonymous set none scality/docs
```

------------------------------------------------------------------------

### \##### 4. Monitoring & Info

**Bucket size**

``` bash
mc du scality/project-data
```

Output:

    5.2GiB used, 120 files

**Server info** (if Scality exposes admin API):

``` bash
mc admin info scality
```

Output:

    Uptime: 12 days
    Drives: 10
    Capacity: 50TB | Used: 12TB | Free: 38TB

**Watch live events**

``` bash
mc watch scality/project-data
```

Example:

    PUT 2025-11-24T10:20:30Z report.pdf
    PUT 2025-11-24T10:20:31Z logs.tar.gz

------------------------------------------------------------------------

## 7. Summary

MinIO Client (`mc`) is essential for Scality because:

-   Scality lacks a full‑featured CLI\
-   MinIO Client provides superior bucket/object management\
-   Folder syncing is powerful and essential for backups\
-   Access control is easy\
-   Monitoring features improve visibility\
-   Works with any S3‑compatible system

MinIO + Scality together provide a powerful, flexible, S3‑compatible
storage experience.

------------------------------------------------------------------------
