# MongoDB Installation Guide

MongoDB is a free and open-source document-oriented NoSQL database. Originally developed by 10gen (now MongoDB Inc.), MongoDB uses JSON-like documents with optional schemas instead of traditional table-based relational database structure. It serves as a FOSS alternative to commercial document databases like Amazon DocumentDB, Azure Cosmos DB, or Oracle NoSQL Database, offering enterprise-grade features including horizontal scaling, replica sets, and sharding without licensing costs, with features like ACID transactions, aggregation pipelines, and full-text search.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended for production)
  - RAM: 2GB minimum (8GB+ recommended for production)
  - Storage: 10GB minimum (SSD strongly recommended for performance)
  - Network: Stable connectivity for replica sets and sharding
- **Operating System**: 
  - Linux: Any modern distribution with kernel 3.2+
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: Not officially supported by MongoDB Inc.
- **Network Requirements**:
  - Port 27017 (default MongoDB port)
  - Port 27018 (default shard port)
  - Port 27019 (default config server port)
  - Additional ports for replica set members
- **Dependencies**:
  - OpenSSL, PCRE, zlib (usually included in distributions)
  - systemd or compatible init system (Linux)
  - Root or administrative access for installation
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Create MongoDB 7.0 repository
sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo <<EOF
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/\$releasever/mongodb-org/7.0/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

# Install MongoDB
sudo yum install -y mongodb-org mongodb-org-tools mongodb-mongosh

# Enable and start service
sudo systemctl enable --now mongod

# Configure firewall
sudo firewall-cmd --permanent --add-port=27017/tcp
sudo firewall-cmd --reload

# Verify installation
mongosh --eval 'db.runCommand("connectionStatus")'
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install prerequisite packages
sudo apt install -y wget gnupg software-properties-common

# Import MongoDB GPG key
wget -qO /tmp/mongodb-server-7.0.asc https://www.mongodb.org/static/pgp/server-7.0.asc
sudo mv /tmp/mongodb-server-7.0.asc /etc/apt/trusted.gpg.d/mongodb-server-7.0.asc

# Add MongoDB repository
echo "deb [arch=amd64,arm64] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Update package index
sudo apt update

# Install MongoDB
sudo apt install -y mongodb-org mongodb-org-tools mongodb-mongosh

# Enable and start service
sudo systemctl enable --now mongod

# Configure firewall
sudo ufw allow 27017
```

### Arch Linux

```bash
# MongoDB is available in AUR
yay -S mongodb-bin mongodb-tools-bin mongosh-bin

# Alternative: Install from AUR with makepkg
git clone https://aur.archlinux.org/mongodb-bin.git
cd mongodb-bin
makepkg -si

# Create mongodb user and group
sudo useradd -r -s /sbin/nologin mongodb

# Create necessary directories
sudo mkdir -p /var/lib/mongodb /var/log/mongodb
sudo chown mongodb:mongodb /var/lib/mongodb /var/log/mongodb

# Enable and start service
sudo systemctl enable --now mongodb

# Configuration location: /etc/mongodb.conf
```

### Alpine Linux

```bash
# MongoDB is not officially supported on Alpine Linux
# Use Docker for MongoDB on Alpine:

# Install Docker
apk add --no-cache docker docker-compose

# Enable and start Docker
rc-update add docker default
rc-service docker start

# Run MongoDB container
docker run -d \
  --name mongodb \
  --restart unless-stopped \
  -p 27017:27017 \
  -v /var/lib/mongodb:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=SecurePassword123! \
  mongo:7.0

# Verify installation
docker exec mongodb mongosh --eval 'db.runCommand("connectionStatus")'
```

### openSUSE/SLES

```bash
# MongoDB is not officially packaged for openSUSE/SLES
# Use Docker or manual installation:

# Method 1: Docker installation
sudo zypper install -y docker docker-compose
sudo systemctl enable --now docker

docker run -d \
  --name mongodb \
  --restart unless-stopped \
  -p 27017:27017 \
  -v /var/lib/mongodb:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=SecurePassword123! \
  mongo:7.0

# Method 2: Manual installation from tarball
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-7.0.4.tgz
tar -xzf mongodb-linux-x86_64-rhel80-7.0.4.tgz
sudo cp mongodb-linux-x86_64-rhel80-7.0.4/bin/* /usr/local/bin/

# Create mongodb user and directories
sudo useradd -r mongodb
sudo mkdir -p /var/lib/mongodb /var/log/mongodb
sudo chown mongodb:mongodb /var/lib/mongodb /var/log/mongodb
```

### macOS

```bash
# Using Homebrew
brew tap mongodb/brew
brew install mongodb-community@7.0 mongodb-database-tools mongosh

# Start MongoDB service
brew services start mongodb/brew/mongodb-community@7.0

# Or run manually
mongod --config /usr/local/etc/mongod.conf

# Configuration location: /usr/local/etc/mongod.conf
# Alternative: /opt/homebrew/etc/mongod.conf (Apple Silicon)
```

### FreeBSD

```bash
# MongoDB is not officially supported on FreeBSD
# Use Docker or compile from source:

# Install Docker
pkg install docker
echo 'docker_enable="YES"' >> /etc/rc.conf
service docker start

# Run MongoDB container
docker run -d \
  --name mongodb \
  --restart unless-stopped \
  -p 27017:27017 \
  -v /var/lib/mongodb:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=SecurePassword123! \
  mongo:7.0
```

### Windows

```powershell
# Method 1: Using Chocolatey
choco install mongodb mongodb-shell

# Method 2: Using Scoop
scoop bucket add main
scoop install mongodb mongodb-shell

# Method 3: Manual installation
# Download from https://www.mongodb.com/download-center/community
# Run mongodb-windows-x86_64-*.msi

# Install as Windows service
"C:\Program Files\MongoDB\Server\7.0\bin\mongod.exe" --config "C:\Program Files\MongoDB\Server\7.0\bin\mongod.cfg" --install

# Start service
net start MongoDB

# Configuration location: C:\Program Files\MongoDB\Server\7.0\bin\mongod.cfg
```

## Initial Configuration

### First-Run Setup

1. **Create mongodb user** (if not created by package):
```bash
# Linux systems
sudo useradd -r -d /var/lib/mongodb -s /sbin/nologin -c "MongoDB Server" mongodb
```

2. **Default configuration locations**:
- RHEL/CentOS/Rocky/AlmaLinux: `/etc/mongod.conf`
- Debian/Ubuntu: `/etc/mongod.conf`
- Arch Linux: `/etc/mongodb.conf`
- Alpine Linux: Docker container configuration
- openSUSE/SLES: `/etc/mongod.conf` (manual installation)
- macOS: `/usr/local/etc/mongod.conf`
- FreeBSD: Docker container configuration
- Windows: `C:\Program Files\MongoDB\Server\7.0\bin\mongod.cfg`

3. **Essential settings to change**:

```yaml
# /etc/mongod.conf
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2
      journalCompressor: snappy

systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
  logRotate: rename

net:
  port: 27017
  bindIp: 127.0.0.1
  maxIncomingConnections: 200

security:
  authorization: enabled
  javascriptEnabled: false

operationProfiling:
  slowOpThresholdMs: 100

replication:
  replSetName: rs0

processManagement:
  fork: true
  pidFilePath: /var/run/mongodb/mongod.pid
```

### Testing Initial Setup

```bash
# Check if MongoDB is running
sudo systemctl status mongod

# Test connection
mongosh --eval 'db.runCommand("connectionStatus")'

# Check database status
mongosh --eval 'db.runCommand("serverStatus")'

# Check configuration
mongosh --eval 'db.runCommand("getCmdLineOpts")'

# Test basic operations
mongosh --eval 'use test; db.testCollection.insertOne({test: "document"}); db.testCollection.findOne()'
```

**WARNING:** Enable authentication and create admin users immediately after installation!

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable MongoDB to start on boot
sudo systemctl enable mongod

# Start MongoDB
sudo systemctl start mongod

# Stop MongoDB
sudo systemctl stop mongod

# Restart MongoDB
sudo systemctl restart mongod

# Reload configuration (graceful restart)
sudo systemctl reload mongod

# Check status
sudo systemctl status mongod

# View logs
sudo journalctl -u mongod -f
```

### OpenRC (Alpine Linux)

```bash
# MongoDB runs in Docker container on Alpine
docker start mongodb
docker stop mongodb
docker restart mongodb

# Check status
docker ps | grep mongodb

# View logs
docker logs -f mongodb
```

### rc.d (FreeBSD)

```bash
# MongoDB runs in Docker container on FreeBSD
service docker start

docker start mongodb
docker stop mongodb
docker restart mongodb

# Check status
docker ps | grep mongodb
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mongodb/brew/mongodb-community@7.0
brew services stop mongodb/brew/mongodb-community@7.0
brew services restart mongodb/brew/mongodb-community@7.0

# Check status
brew services list | grep mongodb

# Manual control
mongod --config /usr/local/etc/mongod.conf
```

### Windows Service Manager

```powershell
# Start MongoDB service
net start MongoDB

# Stop MongoDB service
net stop MongoDB

# Using PowerShell
Start-Service MongoDB
Stop-Service MongoDB
Restart-Service MongoDB

# Check status
Get-Service MongoDB

# View logs
Get-EventLog -LogName Application -Source MongoDB
```

## Advanced Configuration

### Replica Set Configuration

```yaml
# Replica set configuration
replication:
  replSetName: rs0

# Sharding configuration (config server)
sharding:
  clusterRole: configsvr

replication:
  replSetName: configReplSet

# Sharding configuration (shard)
sharding:
  clusterRole: shardsvr

replication:
  replSetName: shardReplSet
```

### Sharding Setup

```javascript
// Initialize config server replica set
rs.initiate({
  _id: "configReplSet",
  configsvr: true,
  members: [
    { _id: 0, host: "config1.example.com:27019" },
    { _id: 1, host: "config2.example.com:27019" },
    { _id: 2, host: "config3.example.com:27019" }
  ]
})

// Initialize shard replica sets
rs.initiate({
  _id: "shard1ReplSet",
  members: [
    { _id: 0, host: "shard1-a.example.com:27018" },
    { _id: 1, host: "shard1-b.example.com:27018" },
    { _id: 2, host: "shard1-c.example.com:27018" }
  ]
})

// Add shards to cluster (from mongos)
sh.addShard("shard1ReplSet/shard1-a.example.com:27018,shard1-b.example.com:27018,shard1-c.example.com:27018")
```

### Advanced Security Settings

```yaml
# Security configuration
security:
  authorization: enabled
  clusterAuthMode: keyFile
  keyFile: /etc/mongodb/mongodb-keyfile
  javascriptEnabled: false
  
net:
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/ssl/mongodb.pem
    CAFile: /etc/mongodb/ssl/ca.pem
    allowInvalidHostnames: false
    allowInvalidCertificates: false

auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.log
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
# /etc/nginx/sites-available/mongodb
upstream mongodb_backend {
    server 127.0.0.1:27017 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:27018 max_fails=3 fail_timeout=30s backup;
}

server {
    listen 27017;
    proxy_pass mongodb_backend;
    proxy_timeout 1s;
    proxy_responses 1;
    error_log /var/log/nginx/mongodb.log;
}
```

### HAProxy Configuration

```haproxy
# /etc/haproxy/haproxy.cfg
frontend mongodb_frontend
    bind *:27017
    mode tcp
    option tcplog
    default_backend mongodb_servers

backend mongodb_servers
    mode tcp
    balance roundrobin
    option tcp-check
    tcp-check connect
    server mongodb1 127.0.0.1:27017 check
    server mongodb2 127.0.0.1:27018 check backup
```

### Connection Pooling with mongos

```yaml
# mongos configuration
sharding:
  configDB: configReplSet/config1.example.com:27019,config2.example.com:27019,config3.example.com:27019

net:
  port: 27017
  bindIp: 0.0.0.0
  maxIncomingConnections: 1000

systemLog:
  destination: file
  path: /var/log/mongodb/mongos.log
  logAppend: true
```

## Security Configuration

### SSL/TLS Setup

```bash
# Generate SSL certificates for MongoDB
sudo mkdir -p /etc/mongodb/ssl

# Create CA certificate
sudo openssl genrsa -out /etc/mongodb/ssl/ca-key.pem 4096
sudo openssl req -new -x509 -days 3650 -key /etc/mongodb/ssl/ca-key.pem -out /etc/mongodb/ssl/ca.pem -subj "/C=US/ST=State/L=City/O=Organization/CN=MongoDB-CA"

# Create server certificate
sudo openssl genrsa -out /etc/mongodb/ssl/mongodb-key.pem 4096
sudo openssl req -new -key /etc/mongodb/ssl/mongodb-key.pem -out /etc/mongodb/ssl/mongodb-req.pem -subj "/C=US/ST=State/L=City/O=Organization/CN=mongodb.example.com"
sudo openssl x509 -req -in /etc/mongodb/ssl/mongodb-req.pem -CA /etc/mongodb/ssl/ca.pem -CAkey /etc/mongodb/ssl/ca-key.pem -CAcreateserial -out /etc/mongodb/ssl/mongodb-cert.pem -days 365

# Combine certificate and key
sudo cat /etc/mongodb/ssl/mongodb-cert.pem /etc/mongodb/ssl/mongodb-key.pem > /etc/mongodb/ssl/mongodb.pem

# Set permissions
sudo chown -R mongodb:mongodb /etc/mongodb/ssl
sudo chmod 600 /etc/mongodb/ssl/*.pem
sudo chmod 644 /etc/mongodb/ssl/ca.pem
```

### User Security and Authentication

```javascript
// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "SecureAdminPassword123!",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
    { role: "clusterAdmin", db: "admin" }
  ]
})

// Create application user
use myapp
db.createUser({
  user: "appuser",
  pwd: "SecureAppPassword123!",
  roles: [
    { role: "readWrite", db: "myapp" }
  ]
})

// Create backup user
use admin
db.createUser({
  user: "backup",
  pwd: "BackupPassword123!",
  roles: [
    { role: "backup", db: "admin" },
    { role: "clusterMonitor", db: "admin" }
  ]
})

// Create monitoring user
db.createUser({
  user: "monitor",
  pwd: "MonitorPassword123!",
  roles: [
    { role: "clusterMonitor", db: "admin" },
    { role: "read", db: "local" }
  ]
})
```

### Firewall Rules

```bash
# UFW (Ubuntu/Debian)
sudo ufw allow from 192.168.1.0/24 to any port 27017
sudo ufw reload

# firewalld (RHEL/CentOS/openSUSE)
sudo firewall-cmd --permanent --new-zone=mongodb
sudo firewall-cmd --permanent --zone=mongodb --add-source=192.168.1.0/24
sudo firewall-cmd --permanent --zone=mongodb --add-port=27017/tcp
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 27017 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4

# pf (FreeBSD)
# Add to /etc/pf.conf
pass in on $ext_if proto tcp from 192.168.1.0/24 to any port 27017

# Windows Firewall
New-NetFirewallRule -DisplayName "MongoDB" -Direction Inbound -Protocol TCP -LocalPort 27017 -RemoteAddress 192.168.1.0/24 -Action Allow
```

## Database Setup

### Database Creation and Management

```javascript
// Create application database
use myapp

// Create collections with validation
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["username", "email"],
      properties: {
        username: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+$",
          description: "must be a valid email address"
        }
      }
    }
  }
})

// Create indexes for performance
db.users.createIndex({ username: 1 }, { unique: true })
db.users.createIndex({ email: 1 }, { unique: true })
db.users.createIndex({ created_at: 1 })

// Create time-series collection (MongoDB 5.0+)
db.createCollection("logs", {
  timeseries: {
    timeField: "timestamp",
    metaField: "source",
    granularity: "minutes"
  }
})
```

### Database Optimization

```javascript
// Analyze collection statistics
db.stats()
db.users.stats()

// Check index usage
db.users.aggregate([{ $indexStats: {} }])

// Optimize queries with explain
db.users.find({ username: "john" }).explain("executionStats")

// Create compound indexes
db.orders.createIndex({ user_id: 1, created_at: -1 })

// Text search index
db.products.createIndex({ name: "text", description: "text" })
```

## Performance Optimization

### System Tuning

```bash
# MongoDB-specific system optimizations
sudo tee -a /etc/sysctl.conf <<EOF
# MongoDB optimizations
vm.swappiness = 1
vm.max_map_count = 262144
net.core.somaxconn = 4096
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_max_syn_backlog = 4096
EOF

sudo sysctl -p

# Disable Transparent Huge Pages
echo 'never' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
echo 'never' | sudo tee /sys/kernel/mm/transparent_hugepage/defrag

# Make THP disable permanent
sudo tee /etc/systemd/system/disable-thp.service <<EOF
[Unit]
Description=Disable Transparent Huge Pages (THP)
DefaultDependencies=no
After=sysinit.target local-fs.target
Before=mongod.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/enabled > /dev/null'
ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/defrag > /dev/null'

[Install]
WantedBy=basic.target
EOF

sudo systemctl enable --now disable-thp
```

### MongoDB Performance Tuning

```yaml
# High-performance MongoDB configuration
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 8  # 50% of available RAM
      journalCompressor: snappy
      directoryForIndexes: true
    collectionConfig:
      blockCompressor: snappy
    indexConfig:
      prefixCompression: true

operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100
  slowOpSampleRate: 1.0

net:
  maxIncomingConnections: 1000
  serviceExecutor: adaptive

setParameter:
  cursorTimeoutMillis: 600000
  failIndexKeyTooLong: false
  maxIndexBuildDrainBatchSize: 128
  wiredTigerConcurrentReadTransactions: 128
  wiredTigerConcurrentWriteTransactions: 128
```

### Query Optimization

```javascript
// Enable profiler for slow operations
db.setProfilingLevel(2, { slowms: 100 })

// Analyze slow queries
db.system.profile.find().limit(5).sort({ ts: -1 }).pretty()

// Index optimization
db.collection.getIndexes()
db.collection.dropIndex("index_name")

// Use aggregation pipeline optimization
db.collection.aggregate([
  { $match: { status: "active" } },
  { $sort: { created_at: -1 } },
  { $limit: 100 }
], { allowDiskUse: true })
```

## Monitoring

### Built-in Monitoring

```javascript
// Server status and statistics
db.runCommand("serverStatus")
db.runCommand("dbStats")
db.runCommand("collStats", "collection_name")

// Connection and operation monitoring
db.runCommand("currentOp")
db.runCommand("top")

// Replica set monitoring
rs.status()
rs.printReplicationInfo()
rs.printSlaveReplicationInfo()

// Sharding monitoring
sh.status()
db.printShardingStatus()
```

### External Monitoring Setup

```bash
# Install MongoDB Exporter for Prometheus
wget https://github.com/percona/mongodb_exporter/releases/download/v0.39.0/mongodb_exporter-0.39.0.linux-amd64.tar.gz
tar xzf mongodb_exporter-*.tar.gz
sudo cp mongodb_exporter /usr/local/bin/

# Create systemd service
sudo tee /etc/systemd/system/mongodb_exporter.service <<EOF
[Unit]
Description=MongoDB Exporter
After=network.target

[Service]
Type=simple
User=mongodb
Environment=MONGODB_URI="mongodb://monitor:MonitorPassword123!@localhost:27017/admin"
ExecStart=/usr/local/bin/mongodb_exporter --mongodb.uri=\$MONGODB_URI
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now mongodb_exporter
```

### Health Check Scripts

```bash
#!/bin/bash
# mongodb-health-check.sh

# Check MongoDB service
if ! systemctl is-active mongod >/dev/null 2>&1; then
    echo "CRITICAL: MongoDB service is not running"
    exit 2
fi

# Check connectivity
if ! mongosh --quiet --eval "db.runCommand('ping')" >/dev/null 2>&1; then
    echo "CRITICAL: Cannot connect to MongoDB"
    exit 2
fi

# Check replica set status (if configured)
REPLICA_STATUS=$(mongosh --quiet --eval "rs.status().ok" 2>/dev/null)
if [ "$REPLICA_STATUS" = "1" ]; then
    PRIMARY_COUNT=$(mongosh --quiet --eval "rs.status().members.filter(m => m.stateStr === 'PRIMARY').length" 2>/dev/null)
    if [ "$PRIMARY_COUNT" != "1" ]; then
        echo "WARNING: No primary or multiple primaries in replica set"
        exit 1
    fi
fi

# Check connections
CONNECTIONS=$(mongosh --quiet --eval "db.serverStatus().connections.current" 2>/dev/null)
MAX_CONNECTIONS=$(mongosh --quiet --eval "db.serverStatus().connections.available" 2>/dev/null)
CONNECTION_USAGE=$((CONNECTIONS * 100 / (CONNECTIONS + MAX_CONNECTIONS)))

if [ $CONNECTION_USAGE -gt 80 ]; then
    echo "WARNING: High connection usage: ${CONNECTION_USAGE}%"
    exit 1
fi

echo "OK: MongoDB is healthy"
exit 0
```

## 9. Backup and Restore

### Backup Procedures

```bash
#!/bin/bash
# mongodb-backup.sh

BACKUP_DIR="/backup/mongodb/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Full database backup
mongodump \
  --host="localhost:27017" \
  --username=backup \
  --password=BackupPassword123! \
  --authenticationDatabase=admin \
  --gzip \
  --out "$BACKUP_DIR"

# Oplog backup for point-in-time recovery
mongodump \
  --host="localhost:27017" \
  --username=backup \
  --password=BackupPassword123! \
  --authenticationDatabase=admin \
  --db=local \
  --collection=oplog.rs \
  --gzip \
  --out "$BACKUP_DIR/oplog"

# Configuration backup
cp -r /etc/mongod.conf "$BACKUP_DIR/"

# Compress backup
tar czf "$BACKUP_DIR.tar.gz" -C "$(dirname "$BACKUP_DIR")" "$(basename "$BACKUP_DIR")"
rm -rf "$BACKUP_DIR"

echo "Backup completed: $BACKUP_DIR.tar.gz"
```

### Restore Procedures

```bash
#!/bin/bash
# mongodb-restore.sh

BACKUP_FILE="$1"
if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup-file.tar.gz>"
    exit 1
fi

# Extract backup
BACKUP_DIR="/tmp/mongodb-restore-$(date +%s)"
mkdir -p "$BACKUP_DIR"
tar xzf "$BACKUP_FILE" -C "$BACKUP_DIR" --strip-components=1

# Stop applications using the database
echo "Stopping applications..."

# Restore database
echo "Restoring database from $BACKUP_FILE..."
mongorestore \
  --host="localhost:27017" \
  --username=admin \
  --password=SecureAdminPassword123! \
  --authenticationDatabase=admin \
  --gzip \
  --drop \
  "$BACKUP_DIR"

# Cleanup
rm -rf "$BACKUP_DIR"

echo "Restore completed"
```

### Point-in-Time Recovery

```bash
#!/bin/bash
# mongodb-pitr.sh

BACKUP_FILE="$1"
RECOVERY_TIME="$2"

if [ -z "$BACKUP_FILE" ] || [ -z "$RECOVERY_TIME" ]; then
    echo "Usage: $0 <backup-file.tar.gz> <recovery-time>"
    echo "Example: $0 backup.tar.gz '2024-01-15T14:30:00.000Z'"
    exit 1
fi

# Extract and restore base backup
BACKUP_DIR="/tmp/mongodb-pitr-$(date +%s)"
mkdir -p "$BACKUP_DIR"
tar xzf "$BACKUP_FILE" -C "$BACKUP_DIR" --strip-components=1

# Restore base backup
mongorestore \
  --host="localhost:27017" \
  --username=admin \
  --password=SecureAdminPassword123! \
  --authenticationDatabase=admin \
  --gzip \
  --drop \
  "$BACKUP_DIR"

# Apply oplog up to recovery point
mongorestore \
  --host="localhost:27017" \
  --username=admin \
  --password=SecureAdminPassword123! \
  --authenticationDatabase=admin \
  --oplogReplay \
  --oplogLimit="$(date -d "$RECOVERY_TIME" +%s):1" \
  --gzip \
  "$BACKUP_DIR/oplog"

# Cleanup
rm -rf "$BACKUP_DIR"

echo "Point-in-time recovery completed to $RECOVERY_TIME"
```

## 6. Troubleshooting

### Common Issues

1. **MongoDB won't start**:
```bash
# Check logs
sudo journalctl -u mongod -f
sudo tail -f /var/log/mongodb/mongod.log

# Check disk space
df -h /var/lib/mongodb

# Check permissions
ls -la /var/lib/mongodb

# Repair database
mongod --repair --dbpath /var/lib/mongodb
```

2. **Connection issues**:
```bash
# Check if MongoDB is listening
sudo ss -tlnp | grep :27017

# Test local connection
mongosh --eval "db.runCommand('ping')"

# Check authentication
mongosh admin --username admin

# Check bind address
mongosh --eval "db.runCommand('getCmdLineOpts')"
```

3. **Performance issues**:
```bash
# Check slow queries
mongosh --eval "db.setProfilingLevel(2, {slowms: 100})"
mongosh --eval "db.system.profile.find().sort({ts:-1}).limit(5)"

# Check index usage
mongosh --eval "db.collection.getIndexes()"

# Check server status
mongosh --eval "db.serverStatus()"
```

### Debug Mode

```bash
# Start MongoDB with verbose logging
sudo systemctl edit mongod
# Add:
[Service]
Environment="MONGOD_OPTIONS=--verbose"

sudo systemctl daemon-reload
sudo systemctl restart mongod

# Enable profiling for all operations
mongosh --eval "db.setProfilingLevel(2)"

# View debug logs
sudo tail -f /var/log/mongodb/mongod.log
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo yum check-update mongodb-org
sudo yum update mongodb-org

# Debian/Ubuntu
sudo apt update
sudo apt upgrade mongodb-org

# Arch Linux
yay -Syu mongodb-bin

# macOS
brew upgrade mongodb-community@7.0

# Docker (Alpine/FreeBSD)
docker pull mongo:7.0
docker stop mongodb
docker rm mongodb
# Re-run docker run command with new image

# Always backup before updates
./mongodb-backup.sh

# Restart after updates
sudo systemctl restart mongod
```

### Maintenance Tasks

```bash
# Weekly maintenance script
#!/bin/bash
# mongodb-maintenance.sh

# Compact collections
mongosh admin --username admin --password SecureAdminPassword123! <<EOF
use myapp
db.runCommand({compact: "collection_name"})
EOF

# Update collection statistics
mongosh admin --username admin --password SecureAdminPassword123! <<EOF
db.runCommand({planCacheClear: ""})
EOF

# Clean up old oplogs (automatically managed but can be tuned)
mongosh admin --username admin --password SecureAdminPassword123! <<EOF
use local
db.oplog.rs.find().sort({\$natural:-1}).limit(1)
EOF

echo "MongoDB maintenance completed"
```

### Health Monitoring

```bash
# Create monitoring cron job
echo "*/5 * * * * /usr/local/bin/mongodb-health-check.sh" | sudo crontab -

# Log rotation
sudo tee /etc/logrotate.d/mongodb <<EOF
/var/log/mongodb/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 mongodb mongodb
    postrotate
        /bin/kill -SIGUSR1 \$(cat /var/run/mongodb/mongod.pid 2>/dev/null) 2>/dev/null || true
    endscript
}
EOF
```

## Integration Examples

### Node.js Integration

```javascript
// Using MongoDB Node.js driver
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://appuser:SecureAppPassword123!@localhost:27017/myapp', {
  tls: true,
  tlsCertificateKeyFile: '/etc/mongodb/ssl/client.pem',
  tlsCAFile: '/etc/mongodb/ssl/ca.pem',
  authSource: 'myapp'
});

async function connect() {
  await client.connect();
  const db = client.db('myapp');
  return db;
}
```

### Python Integration

```python
# Using PyMongo
import pymongo
from pymongo import MongoClient

client = MongoClient('mongodb://appuser:SecureAppPassword123!@localhost:27017/myapp', 
                    tls=True,
                    tlsCertificateKeyFile='/etc/mongodb/ssl/client.pem',
                    tlsCAFile='/etc/mongodb/ssl/ca.pem',
                    authSource='myapp')

db = client.myapp
collection = db.users
```

### Java Integration

```java
// Using MongoDB Java driver
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoDatabase;
import com.mongodb.MongoClientSettings;
import com.mongodb.ConnectionString;

MongoClientSettings settings = MongoClientSettings.builder()
    .applyConnectionString(new ConnectionString("mongodb://appuser:SecureAppPassword123!@localhost:27017/myapp"))
    .applyToSslSettings(builder -> 
        builder.enabled(true)
               .invalidHostNameAllowed(false))
    .build();

MongoClient mongoClient = MongoClients.create(settings);
MongoDatabase database = mongoClient.getDatabase("myapp");
```

### Express.js Integration

```javascript
// Using Mongoose ODM
const mongoose = require('mongoose');

mongoose.connect('mongodb://appuser:SecureAppPassword123!@localhost:27017/myapp', {
  tls: true,
  tlsCertificateKeyFile: '/etc/mongodb/ssl/client.pem',
  tlsCAFile: '/etc/mongodb/ssl/ca.pem',
  authSource: 'myapp'
});

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  created_at: { type: Date, default: Date.now }
});

module.exports = mongoose.model('User', userSchema);
```

## Additional Resources

- [Official MongoDB Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/)
- [MongoDB Performance Best Practices](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/)
- [Replica Set Tutorial](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/)
- [Sharding Tutorial](https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/)
- [MongoDB Community Forum](https://developer.mongodb.com/community/forums/)
- [MongoDB Blog](https://www.mongodb.com/blog)

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.