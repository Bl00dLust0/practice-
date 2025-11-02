# 🚀 Nextcloud VM Deployment & Configuration Guide

This document provides detailed steps to set up a multi-customer **Nextcloud** environment using **Docker**, **Docker Compose**, **MySQL**  and **NGINX reverse proxy** on AlmaLinux.

---

##

### Customer Accounts

| Customer       | Admin User | Admin Pass        | DB User      | DB Pass      |
| -------------- | ---------- | ----------------- | ------------ | ------------ |
| **Customer 1** | `admin`    | `cTJyhucdPpQ92Do` | `cust1_user` | `cust1_pass` |
| **Customer 2** | `admin`    | `YWtVFx6t3y7h2jK` | `cust2_user` | `cust2_pass` |
| **Customer 3** | `admin`    | `xoZbDtNppJ2tzig` | `cust3_user` | `cust3_pass` |
|                |            |                   |              |              |

---

## ⚙️ Step 1: Install Docker & Docker Compose on AlmaLinux

### 1️⃣ Remove old Docker versions

```bash
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest \
  docker-latest-logrotate docker-logrotate docker-engine
```

### 2️⃣ Install required packages

```bash
sudo dnf -y install dnf-plugins-core
```

### 3️⃣ Set up Docker repo

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 4️⃣ Install Docker Engine

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

### 5️⃣ Start and enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 🧩 Step 2: Docker Compose Configuration

### Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  reverse-proxy:
    image: nginx:latest
    container_name: nginx_proxy
    ports:
      - "80:80"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    depends_on:
      - nextcloud_customer1
      - nextcloud_customer2
      - nextcloud_customer3

  db:
    image: mariadb:10.6
    container_name: nextcloud_db
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    environment:
      MYSQL_ROOT_PASSWORD: x8NxbkmPBttqNU4
    volumes:
      - db_data:/var/lib/mysql

  nextcloud_customer1:
    image: nextcloud
    container_name: nextcloud_cust1
    expose:
      - "80"
    environment:
      MYSQL_PASSWORD: cust1_pass
      MYSQL_DATABASE: nextcloud_cust1
      MYSQL_USER: cust1_user
      MYSQL_HOST: db
      PHP_UPLOAD_LIMIT: 10G
      PHP_MEMORY_LIMIT: 512M
    volumes:
      - nextcloud_cust1_data:/var/www/html

  nextcloud_customer2:
    image: nextcloud
    container_name: nextcloud_cust2
    expose:
      - "80"
    environment:
      MYSQL_PASSWORD: cust2_pass
      MYSQL_DATABASE: nextcloud_cust2
      MYSQL_USER: cust2_user
      MYSQL_HOST: db
      PHP_UPLOAD_LIMIT: 10G
      PHP_MEMORY_LIMIT: 512M
    volumes:
      - nextcloud_cust2_data:/var/www/html

  nextcloud_customer3:
    image: nextcloud
    container_name: nextcloud_cust3
    expose:
      - "80"
    environment:
      MYSQL_PASSWORD: cust3_pass
      MYSQL_DATABASE: nextcloud_cust3
      MYSQL_USER: cust3_user
      MYSQL_HOST: db
      PHP_UPLOAD_LIMIT: 10G
      PHP_MEMORY_LIMIT: 512M
    volumes:
      - nextcloud_cust3_data:/var/www/html

volumes:
  db_data:
  nextcloud_cust1_data:
  nextcloud_cust2_data:
  nextcloud_cust3_data:
```

### Start all containers

```bash
docker compose up -d
```

To start only one container (e.g., Customer 1):

```bash
docker compose up -d nextcloud_customer1
```

---

## 🌐 Step 3: NGINX Configuration

Create and edit your configuration:

```bash
cd nginx/
vi default.conf
```

### Example: Reverse Proxy Configuration

```nginx
server {
    listen 80;
    server_name customer1.datahub.com.np;

    location / {
        proxy_pass http://nextcloud_customer1:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        client_max_body_size 10G;
    }
}
```

Repeat similar blocks for `customer2` and `customer3`.

---

## 🗃️ Step 4: Database Setup

Access the database container:

```bash
docker exec -it nextcloud_db bash
mysql -u root -p
```

Enter your root password, then create databases:

```sql
CREATE DATABASE nextcloud_cust1;
CREATE USER 'cust1_user'@'%' IDENTIFIED BY 'cust1_pass';
GRANT ALL PRIVILEGES ON nextcloud_cust1.* TO 'cust1_user'@'%';

CREATE DATABASE nextcloud_cust2;
CREATE USER 'cust2_user'@'%' IDENTIFIED BY 'cust2_pass';
GRANT ALL PRIVILEGES ON nextcloud_cust2.* TO 'cust2_user'@'%';

CREATE DATABASE nextcloud_cust3;
CREATE USER 'cust3_user'@'%' IDENTIFIED BY 'cust3_pass';
GRANT ALL PRIVILEGES ON nextcloud_cust3.* TO 'cust3_user'@'%';

FLUSH PRIVILEGES;
```

---

## ☁️ Step 5: Configure S3 as Primary Storage

Edit the file inside each Nextcloud container:

```bash
vi /var/www/html/config/config.php
```

Append the following block:

```php
'objectstore' => array(
  'class' => 'OC\\Files\\ObjectStore\\S3',
  'arguments' => array(
    'bucket' => 'nextcloud-data-prabhat',
    'autocreate' => true,
    'key' => 'Q9R7JYRKIXXSWL24RB',
    'secret' => '/Q6jlj=pGo2napts2xrHmQtuzwYk9+Dv71BPVQ',
    'hostname' => 's3-np1.datahub.com.np',
    'port' => 443,
    'use_ssl' => true,
    'region' => 'us-east1',
    'use_path_style' => true,
  ),
),
```

---

## 🔐 Step 6: SSL Configuration

### 1️⃣ Add SSL Certificates

Place certificate files inside:

```
/nginx/ssl/
```

### 2️⃣ Update NGINX Config

Example for Customer 1:

```nginx
server {
    listen 80;
    server_name customer1.datahub.com.np;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name customer1.datahub.com.np;

    ssl_certificate /etc/nginx/conf.d/ssl/datahub-2025certificate.crt;
    ssl_certificate_key /etc/nginx/conf.d/ssl/datahub.key;

    location / {
        proxy_pass http://nextcloud_customer1:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Repeat for each customer domain.

### 3️⃣ Update Nextcloud Config

Append inside `/var/www/html/config/config.php`:

```php
'overwrite.cli.url' => 'https://customer1.datahub.com.np',
'overwriteprotocol' => 'https',
'trusted_proxies' => array('nginx_proxy'),
'forwarded_for_headers' => array('HTTP_X_FORWARDED_FOR'),
'force_ssl' => true,
```

---

## 📦 Step 7: Update Handling

> Nextcloud disables browser-based updates in Docker environments for consistency.

To repair or apply maintenance tasks:

```bash
docker exec -it -u www-data nextcloud_cust1 bash
php occ maintenance:repair --include-expensive
```

---

## ✅ Final Configuration Example (config.php)

```php
'default_phone_region' => 'NP',
'maintenance_window_start' => 1,
'trusted_proxies' => array(
  '172.17.0.1/16',
  '172.18.0.0/16',
  '10.10.112.0/24',
),
'overwrite.cli.url' => 'https://ss-associates.datahub.com.np',
'overwriteprotocol' => 'https',
```

