# Quick Reference Guide

## Service Connection Details

### MySQL Database
```bash
Host: localhost
Port: 3306
Root Password: ThisIsAComplicatePassWord
Database: test
User: test
Password: 123456

# Connect
mysql -h localhost -P 3306 -u test -p123456 test
```

### Redis Cache
```bash
Host: localhost
Port: 6379
Password: None

# Connect
redis-cli -h localhost -p 6379
redis-cli -h localhost -p 6379 ping  # Test connection
```

### SFTP Server
```bash
Host: localhost
Port: 2001
Username: testuser
Password: testpasswd

# Connect
sftp -P 2001 testuser@localhost
ssh -p 2001 testuser@localhost
```

## Docker Commands

### Start/Stop Services
```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View running containers
docker-compose ps

# View logs
docker-compose logs -f

# Restart specific service
docker-compose restart mysql
docker-compose restart redis
docker-compose restart sftp
```

### Container Management
```bash
# Access container shell
docker-compose exec mysql bash
docker-compose exec redis bash
docker-compose exec sftp bash

# View container stats
docker stats mysql redis sftp

# Check container processes
docker-compose top
```

## Database Operations

### MySQL Quick Commands
```sql
-- Show databases
SHOW DATABASES;

-- Use test database
USE test;

-- Show tables
SHOW TABLES;

-- Query users table
SELECT * FROM users;

-- Create new user
INSERT INTO users (username, email, phone) VALUES ('newuser', 'user@example.com', '1234567890');

-- Check MySQL status
SHOW STATUS LIKE 'Connections';
SHOW PROCESSLIST;
```

### Redis Quick Commands
```bash
# Basic operations
SET key value
GET key
DEL key
EXISTS key

# Hash operations
HSET user:1 name "John"
HGET user:1 name
HGETALL user:1

# List operations
LPUSH queue item1 item2
RPOP queue
LLEN queue

# Check Redis info
INFO
INFO memory
INFO stats
```

## File Transfer (SFTP)

### Common SFTP Commands
```bash
# After connecting: sftp -P 2001 testuser@localhost

# Upload file
put local_file.txt remote_file.txt

# Download file
get remote_file.txt local_file.txt

# List files
ls          # Remote files
lls         # Local files

# Change directory
cd /path/to/directory    # Remote
lcd /local/path          # Local

# Create/remove directory
mkdir new_directory
rmdir directory_name
```

## Connection Strings

### Python
```python
# MySQL (PyMySQL)
import pymysql
conn = pymysql.connect(host='localhost', port=3306, user='test', password='123456', database='test')

# Redis
import redis
r = redis.Redis(host='localhost', port=6379, db=0)

# SFTP (paramiko)
import paramiko
ssh = paramiko.SSHClient()
ssh.connect('localhost', port=2001, username='testuser', password='testpasswd')
```

### Node.js
```javascript
// MySQL
const mysql = require('mysql2');
const connection = mysql.createConnection({host: 'localhost', port: 3306, user: 'test', password: '123456', database: 'test'});

// Redis
const redis = require('redis');
const client = redis.createClient({host: 'localhost', port: 6379});

// SFTP
const Client = require('ssh2-sftp-client');
const sftp = new Client();
sftp.connect({host: 'localhost', port: 2001, username: 'testuser', password: 'testpasswd'});
```

### Java
```java
// MySQL JDBC
String url = "jdbc:mysql://localhost:3306/test";
Connection conn = DriverManager.getConnection(url, "test", "123456");

// Redis (Jedis)
Jedis jedis = new Jedis("localhost", 6379);

// SFTP (JSch)
JSch jsch = new JSch();
Session session = jsch.getSession("testuser", "localhost", 2001);
session.setPassword("testpasswd");
```

## Health Checks

### Quick Health Check Script
```bash
#!/bin/bash
echo "=== Health Check ==="

# MySQL
mysql -h localhost -P 3306 -u test -p123456 -e "SELECT 1" 2>/dev/null && echo "✓ MySQL OK" || echo "✗ MySQL FAIL"

# Redis
redis-cli -h localhost -p 6379 ping | grep -q "PONG" && echo "✓ Redis OK" || echo "✗ Redis FAIL"

# SFTP
timeout 5 bash -c "</dev/tcp/localhost/2001" 2>/dev/null && echo "✓ SFTP OK" || echo "✗ SFTP FAIL"
```

### Service Status Commands
```bash
# Check if services are running
docker-compose ps

# Check service logs for errors
docker-compose logs mysql | tail -20
docker-compose logs redis | tail -20
docker-compose logs sftp | tail -20

# Check port accessibility
telnet localhost 3306  # MySQL
telnet localhost 6379  # Redis
telnet localhost 2001  # SFTP
```

## Common Issues & Solutions

### MySQL Issues
```bash
# Connection refused
docker-compose logs mysql
docker-compose restart mysql

# Access denied
# Check password: ThisIsAComplicatePassWord (root) or 123456 (test user)

# Data directory permission
sudo chown -R 999:999 ~/docker_data/mysql/data
```

### Redis Issues
```bash
# Connection timeout
docker-compose logs redis
docker-compose restart redis

# Memory issues
redis-cli info memory
# Check maxmemory settings
```

### SFTP Issues
```bash
# Connection refused
docker-compose logs sftp
docker-compose restart sftp

# Authentication failed
# Check credentials: testuser / testpasswd

# Permission denied
docker-compose exec sftp id testuser
docker-compose exec sftp passwd testuser
```

## Data Backup

### MySQL Backup
```bash
# Create backup
docker-compose exec mysql mysqldump -u root -pThisIsAComplicatePassWord test > mysql_backup.sql

# Restore backup
docker-compose exec -T mysql mysql -u root -pThisIsAComplicatePassWord test < mysql_backup.sql
```

### Redis Backup
```bash
# Create RDB snapshot
docker-compose exec redis redis-cli BGSAVE

# Copy RDB file
docker cp redis:/data/dump.rdb ./redis_backup.rdb

# Restore RDB file
docker cp ./redis_backup.rdb redis:/data/dump.rdb
docker-compose restart redis
```

### SFTP File Management
```bash
# Copy files from container
docker cp sftp:/home/testuser/file.txt ./local_file.txt

# Copy files to container
docker cp ./local_file.txt sftp:/home/testuser/file.txt
```

## Performance Monitoring

### Resource Usage
```bash
# Real-time container stats
docker stats mysql redis sftp

# Memory usage
docker stats --format "table {{.Container}}\t{{.MemUsage}}\t{{.MemPerc}}"

# CPU usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}"
```

### Service Metrics
```bash
# MySQL performance
docker-compose exec mysql mysqladmin -u root -pThisIsAComplicatePassWord status
docker-compose exec mysql mysqladmin -u root -pThisIsAComplicatePassWord processlist

# Redis performance
docker-compose exec redis redis-cli info stats
docker-compose exec redis redis-cli info memory

# SFTP connections
docker-compose exec sftp netstat -an | grep :22
```

## Environment Setup

### First Time Setup
```bash
# 1. Create data directories
mkdir -p ~/docker_data/mysql/data
mkdir -p ~/docker_data/mysql/init

# 2. Set permissions
sudo chown -R 999:999 ~/docker_data/mysql

# 3. Build and start services
docker-compose up -d

# 4. Wait for services to initialize
sleep 30

# 5. Test connections
./health_check.sh
```

### Cleanup
```bash
# Stop and remove containers
docker-compose down

# Remove all data (destructive!)
docker-compose down -v
sudo rm -rf ~/docker_data/mysql

# Remove images
docker rmi mysql:1 redis:1 sftp:1
```

## Security Notes

⚠️ **Important Security Considerations:**

1. **Change default passwords** before production use
2. **Use environment variables** for sensitive data
3. **Enable SSL/TLS** for production deployments
4. **Restrict network access** with firewalls
5. **Use SSH keys** instead of passwords for SFTP
6. **Keep images updated** regularly

### Production Security Checklist
- [ ] Changed MySQL root password
- [ ] Changed test user password
- [ ] Enabled Redis authentication
- [ ] Changed SFTP user credentials
- [ ] Configured SSL for MySQL
- [ ] Set up Redis password
- [ ] Disabled SSH password authentication
- [ ] Configured proper firewall rules
- [ ] Set up log rotation
- [ ] Configured monitoring

## Port Reference

| Service | Internal Port | External Port | Protocol |
|---------|---------------|---------------|----------|
| MySQL   | 3306          | 3306          | TCP      |
| Redis   | 6379          | 6379          | TCP      |
| SFTP    | 22            | 2001          | TCP/SSH  |

## File Locations

### Configuration Files
- MySQL config: `mysql/mysqld.cnf`
- Redis config: `redis/redis.conf`
- Docker compose: `docker-compose.yaml`

### Data Directories
- MySQL data: `~/docker_data/mysql/data`
- MySQL init: `~/docker_data/mysql/init`
- Redis data: Container internal `/data`

### Log Files
- MySQL logs: `/var/lib/mysql/` (inside container)
- Redis logs: stdout (view with `docker-compose logs redis`)
- SFTP logs: `/var/log/auth.log` (inside container)

---

For detailed information, see the complete [API Documentation](API_DOCUMENTATION.md) and [Configuration Reference](CONFIGURATION_REFERENCE.md).