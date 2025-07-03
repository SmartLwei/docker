# Docker Multi-Service Setup - API Documentation

## Overview

This Docker-based multi-service setup provides three essential development services:
- **MySQL Database Server** (MySQL 5.7)
- **Redis Cache Server** (Redis 6.2)
- **SFTP File Transfer Server** (Ubuntu-based)

All services are containerized and orchestrated using Docker Compose for easy deployment and management.

## Table of Contents

1. [Quick Start](#quick-start)
2. [MySQL Service](#mysql-service)
3. [Redis Service](#redis-service)
4. [SFTP Service](#sftp-service)
5. [Docker Configuration](#docker-configuration)
6. [API Examples](#api-examples)
7. [Troubleshooting](#troubleshooting)

## Quick Start

### Prerequisites
- Docker Engine installed
- Docker Compose installed
- Ports 3306, 6379, and 2001 available

### Starting All Services
```bash
# Clone the repository
git clone <repository-url>
cd docker

# Start all services
docker-compose up -d

# View running containers
docker-compose ps

# View logs
docker-compose logs -f
```

### Stopping Services
```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## MySQL Service

### Connection Details
- **Host**: `localhost` (or Docker host IP)
- **Port**: `3306`
- **Root Password**: `ThisIsAComplicatePassWord`
- **Character Set**: `utf8mb4`
- **Collation**: `utf8mb4_unicode_ci`

### Pre-configured Database
- **Database Name**: `test`
- **User**: `test`
- **Password**: `123456`
- **Permissions**: Full access to `test` database

### API Access Examples

#### Command Line Connection
```bash
# Connect as root
mysql -h localhost -P 3306 -u root -p
# Password: ThisIsAComplicatePassWord

# Connect as test user
mysql -h localhost -P 3306 -u test -p test
# Password: 123456
```

#### Programming Language Examples

##### Python (PyMySQL)
```python
import pymysql

# Connection configuration
config = {
    'host': 'localhost',
    'port': 3306,
    'user': 'test',
    'password': '123456',
    'database': 'test',
    'charset': 'utf8mb4'
}

# Establish connection
connection = pymysql.connect(**config)
cursor = connection.cursor()

# Example query
cursor.execute("SELECT * FROM users")
users = cursor.fetchall()
print(users)

# Close connection
cursor.close()
connection.close()
```

##### Node.js (mysql2)
```javascript
const mysql = require('mysql2');

const connection = mysql.createConnection({
  host: 'localhost',
  port: 3306,
  user: 'test',
  password: '123456',
  database: 'test'
});

// Example query
connection.query('SELECT * FROM users', (error, results) => {
  if (error) throw error;
  console.log(results);
});

connection.end();
```

##### Java (JDBC)
```java
import java.sql.*;

public class MySQLExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/test";
        String username = "test";
        String password = "123456";
        
        try {
            Connection connection = DriverManager.getConnection(url, username, password);
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery("SELECT * FROM users");
            
            while (resultSet.next()) {
                System.out.println(resultSet.getString("username"));
            }
            
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### Pre-configured Tables

#### Users Table Schema
```sql
CREATE TABLE `users` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_bin NOT NULL COMMENT '用户名',
  `password` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '密码',
  `phone` varchar(20) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '手机号',
  `email` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '邮箱',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

#### Sample Data
```sql
INSERT INTO `test`.`users` 
(`id`, `username`, `password`, `phone`, `email`, `create_time`) 
VALUES 
('1', 'xiao', '123', '12345678910', '123@qq.com', '2020-04-10 01:22:07');
```

### Key Configuration Settings
- **Max Connections**: 3500
- **Max Allowed Packet**: 1024M
- **Query Cache**: Enabled (2M)
- **InnoDB Buffer Pool**: 6144M
- **Slow Query Log**: Enabled (10 seconds threshold)
- **Binary Logging**: Enabled with GTID

### Data Persistence
- **Data Directory**: `~/docker_data/mysql/data`
- **Init Scripts**: `~/docker_data/mysql/init`

---

## Redis Service

### Connection Details
- **Host**: `localhost` (or Docker host IP)
- **Port**: `6379`
- **Password**: None (authentication disabled)
- **Protocol**: Redis Protocol (RESP)

### API Access Examples

#### Command Line Connection
```bash
# Connect using redis-cli
redis-cli -h localhost -p 6379

# Test connection
redis-cli -h localhost -p 6379 ping
# Expected response: PONG
```

#### Programming Language Examples

##### Python (redis-py)
```python
import redis

# Create connection
r = redis.Redis(host='localhost', port=6379, db=0)

# Basic operations
r.set('key', 'value')
value = r.get('key')
print(value.decode('utf-8'))  # Output: value

# Hash operations
r.hset('user:1', 'name', 'John')
r.hset('user:1', 'email', 'john@example.com')
user_data = r.hgetall('user:1')
print(user_data)

# List operations
r.lpush('queue', 'task1', 'task2', 'task3')
task = r.rpop('queue')
print(task.decode('utf-8'))

# Set operations
r.sadd('tags', 'python', 'redis', 'database')
tags = r.smembers('tags')
print(tags)
```

##### Node.js (redis)
```javascript
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

client.on('error', (err) => {
  console.log('Redis Client Error', err);
});

// Basic operations
async function example() {
  await client.connect();
  
  await client.set('key', 'value');
  const value = await client.get('key');
  console.log(value); // Output: value
  
  // Hash operations
  await client.hSet('user:1', {
    name: 'John',
    email: 'john@example.com'
  });
  
  const userData = await client.hGetAll('user:1');
  console.log(userData);
  
  await client.disconnect();
}

example();
```

##### Java (Jedis)
```java
import redis.clients.jedis.Jedis;

public class RedisExample {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("localhost", 6379);
        
        // Basic operations
        jedis.set("key", "value");
        String value = jedis.get("key");
        System.out.println(value); // Output: value
        
        // Hash operations
        jedis.hset("user:1", "name", "John");
        jedis.hset("user:1", "email", "john@example.com");
        
        // List operations
        jedis.lpush("queue", "task1", "task2");
        String task = jedis.rpop("queue");
        System.out.println(task);
        
        jedis.close();
    }
}
```

### Common Redis Commands

#### String Operations
```bash
# Set and get values
SET key value
GET key
INCR counter
DECR counter
EXPIRE key 3600
```

#### Hash Operations
```bash
# Hash field operations
HSET user:1 name "John Doe"
HGET user:1 name
HGETALL user:1
HDEL user:1 email
```

#### List Operations
```bash
# Queue operations
LPUSH queue item1 item2
RPOP queue
LLEN queue
LRANGE queue 0 -1
```

#### Set Operations
```bash
# Set operations
SADD tags tag1 tag2 tag3
SMEMBERS tags
SISMEMBER tags tag1
SCARD tags
```

#### Sorted Set Operations
```bash
# Sorted set operations
ZADD leaderboard 100 player1 200 player2
ZRANGE leaderboard 0 -1 WITHSCORES
ZRANK leaderboard player1
```

### Key Configuration Settings
- **Max Memory Policy**: Not explicitly set (uses Redis defaults)
- **Persistence**: RDB snapshots enabled
- **TCP Keepalive**: 300 seconds
- **Protected Mode**: Enabled
- **Databases**: 16 (default)

---

## SFTP Service

### Connection Details
- **Host**: `localhost` (or Docker host IP)
- **Port**: `2001` (mapped from container port 22)
- **Username**: `testuser`
- **Password**: `testpasswd`
- **Protocol**: SSH/SFTP

### API Access Examples

#### Command Line Connection
```bash
# SFTP connection
sftp -P 2001 testuser@localhost
# Password: testpasswd

# SSH connection
ssh -p 2001 testuser@localhost
# Password: testpasswd
```

#### File Transfer Examples

##### Using SFTP Command Line
```bash
# Connect and transfer files
sftp -P 2001 testuser@localhost

# SFTP commands
put local_file.txt remote_file.txt    # Upload file
get remote_file.txt local_file.txt    # Download file
ls                                    # List remote files
lls                                   # List local files
mkdir remote_directory                # Create remote directory
rmdir remote_directory                # Remove remote directory
cd /path/to/directory                 # Change remote directory
lcd /local/path                       # Change local directory
```

##### Python (paramiko)
```python
import paramiko

# Create SFTP client
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    # Connect
    ssh.connect('localhost', port=2001, username='testuser', password='testpasswd')
    sftp = ssh.open_sftp()
    
    # Upload file
    sftp.put('local_file.txt', '/home/testuser/remote_file.txt')
    
    # Download file
    sftp.get('/home/testuser/remote_file.txt', 'downloaded_file.txt')
    
    # List directory
    files = sftp.listdir('/home/testuser')
    print(files)
    
    # Create directory
    sftp.mkdir('/home/testuser/new_directory')
    
except Exception as e:
    print(f"Error: {e}")
finally:
    sftp.close()
    ssh.close()
```

##### Node.js (ssh2-sftp-client)
```javascript
const Client = require('ssh2-sftp-client');
const sftp = new Client();

const config = {
  host: 'localhost',
  port: 2001,
  username: 'testuser',
  password: 'testpasswd'
};

sftp.connect(config)
  .then(() => {
    console.log('Connected to SFTP server');
    
    // Upload file
    return sftp.put('local_file.txt', '/home/testuser/remote_file.txt');
  })
  .then(() => {
    console.log('File uploaded');
    
    // Download file
    return sftp.get('/home/testuser/remote_file.txt', 'downloaded_file.txt');
  })
  .then(() => {
    console.log('File downloaded');
    
    // List directory
    return sftp.list('/home/testuser');
  })
  .then((files) => {
    console.log('Directory listing:', files);
  })
  .catch((err) => {
    console.error('Error:', err);
  })
  .finally(() => {
    sftp.end();
  });
```

##### Java (JSch)
```java
import com.jcraft.jsch.*;
import java.io.*;

public class SFTPExample {
    public static void main(String[] args) {
        JSch jsch = new JSch();
        Session session = null;
        ChannelSftp channel = null;
        
        try {
            session = jsch.getSession("testuser", "localhost", 2001);
            session.setPassword("testpasswd");
            session.setConfig("StrictHostKeyChecking", "no");
            session.connect();
            
            channel = (ChannelSftp) session.openChannel("sftp");
            channel.connect();
            
            // Upload file
            channel.put("local_file.txt", "/home/testuser/remote_file.txt");
            
            // Download file
            channel.get("/home/testuser/remote_file.txt", "downloaded_file.txt");
            
            // List directory
            Vector<ChannelSftp.LsEntry> files = channel.ls("/home/testuser");
            for (ChannelSftp.LsEntry file : files) {
                System.out.println(file.getFilename());
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (channel != null) channel.disconnect();
            if (session != null) session.disconnect();
        }
    }
}
```

### Security Configuration
- **SSH Protocol Version**: 2
- **Authentication Methods**: 
  - Password authentication: Enabled
  - Public key authentication: Enabled
  - RSA authentication: Enabled
- **Key Exchange Algorithms**: Includes legacy support for diffie-hellman-group1-sha1
- **Public Key Algorithms**: Supports ssh-rsa

### User Home Directory
- **Path**: `/home/testuser`
- **Permissions**: Full read/write access for testuser
- **SSH Keys Directory**: `/home/testuser/.ssh` (available for public key authentication)

---

## Docker Configuration

### Service Architecture

```yaml
services:
  mysql:
    image: mysql:1
    container_name: mysql
    ports: ["3306:3306"]
    
  redis:
    image: redis:1
    container_name: redis
    ports: ["6379:6379"]
    
  sftp:
    image: sftp:1
    container_name: sftp
    ports: ["2001:22"]
```

### Volume Mounts
- **MySQL Data**: `~/docker_data/mysql/data:/var/lib/mysql`
- **MySQL Init Scripts**: `~/docker_data/mysql/init:/docker-entrypoint-initdb.d/`

### Environment Variables

#### MySQL
```yaml
environment:
  - MYSQL_ROOT_PASSWORD=ThisIsAComplicatePassWord
```

### Build Context
Each service builds from its respective directory:
- MySQL: `./mysql/dockerfile`
- Redis: `./redis/dockerfile`
- SFTP: `./sftp/dockerfile`

### Network Configuration
- **Default Network**: Bridge mode
- **Port Mappings**:
  - MySQL: `3306:3306`
  - Redis: `6379:6379`
  - SFTP: `2001:22`

---

## API Examples

### Multi-Service Application Example

#### Python Application Using All Services
```python
import pymysql
import redis
import paramiko
from datetime import datetime

class MultiServiceClient:
    def __init__(self):
        # MySQL connection
        self.mysql_conn = pymysql.connect(
            host='localhost',
            port=3306,
            user='test',
            password='123456',
            database='test',
            charset='utf8mb4'
        )
        
        # Redis connection
        self.redis_conn = redis.Redis(host='localhost', port=6379, db=0)
        
        # SFTP connection
        self.ssh = paramiko.SSHClient()
        self.ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        self.ssh.connect('localhost', port=2001, username='testuser', password='testpasswd')
        self.sftp = self.ssh.open_sftp()
    
    def create_user(self, username, email, phone):
        """Create user in MySQL and cache in Redis"""
        cursor = self.mysql_conn.cursor()
        
        # Insert into MySQL
        sql = """INSERT INTO users (username, email, phone, create_time) 
                 VALUES (%s, %s, %s, %s)"""
        cursor.execute(sql, (username, email, phone, datetime.now()))
        user_id = cursor.lastrowid
        self.mysql_conn.commit()
        
        # Cache in Redis
        user_data = {
            'id': user_id,
            'username': username,
            'email': email,
            'phone': phone
        }
        self.redis_conn.hset(f'user:{user_id}', mapping=user_data)
        self.redis_conn.expire(f'user:{user_id}', 3600)  # 1 hour TTL
        
        cursor.close()
        return user_id
    
    def get_user(self, user_id):
        """Get user from cache or database"""
        # Try Redis first
        cached_user = self.redis_conn.hgetall(f'user:{user_id}')
        if cached_user:
            return {k.decode(): v.decode() for k, v in cached_user.items()}
        
        # Fallback to MySQL
        cursor = self.mysql_conn.cursor(pymysql.cursors.DictCursor)
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        user = cursor.fetchone()
        cursor.close()
        
        if user:
            # Update cache
            self.redis_conn.hset(f'user:{user_id}', mapping=user)
            self.redis_conn.expire(f'user:{user_id}', 3600)
        
        return user
    
    def upload_user_file(self, user_id, local_file_path):
        """Upload file for user via SFTP"""
        remote_path = f'/home/testuser/user_{user_id}_file.txt'
        self.sftp.put(local_file_path, remote_path)
        
        # Log file upload in Redis
        self.redis_conn.lpush(f'user:{user_id}:uploads', remote_path)
        
        return remote_path
    
    def close_connections(self):
        """Clean up all connections"""
        self.mysql_conn.close()
        self.redis_conn.close()
        self.sftp.close()
        self.ssh.close()

# Usage example
client = MultiServiceClient()

# Create a new user
user_id = client.create_user('johndoe', 'john@example.com', '1234567890')
print(f"Created user with ID: {user_id}")

# Retrieve user
user = client.get_user(user_id)
print(f"Retrieved user: {user}")

# Upload file for user
# client.upload_user_file(user_id, 'local_file.txt')

client.close_connections()
```

### Health Check Endpoints

#### Service Health Check Script
```bash
#!/bin/bash

echo "=== Docker Services Health Check ==="

# Check MySQL
echo "Checking MySQL..."
mysql -h localhost -P 3306 -u test -p123456 -e "SELECT 1" 2>/dev/null && echo "✓ MySQL is healthy" || echo "✗ MySQL connection failed"

# Check Redis
echo "Checking Redis..."
redis-cli -h localhost -p 6379 ping | grep -q "PONG" && echo "✓ Redis is healthy" || echo "✗ Redis connection failed"

# Check SFTP
echo "Checking SFTP..."
timeout 5 bash -c "</dev/tcp/localhost/2001" 2>/dev/null && echo "✓ SFTP is healthy" || echo "✗ SFTP connection failed"

echo "=== Health Check Complete ==="
```

---

## Troubleshooting

### Common Issues and Solutions

#### MySQL Issues

**Issue**: Connection refused to MySQL
```bash
# Check if container is running
docker-compose ps mysql

# Check MySQL logs
docker-compose logs mysql

# Verify port is accessible
telnet localhost 3306
```

**Issue**: Access denied for user
```bash
# Reset MySQL root password
docker-compose exec mysql mysql -u root -p
# Use password: ThisIsAComplicatePassWord
```

#### Redis Issues

**Issue**: Redis connection timeout
```bash
# Check Redis container status
docker-compose ps redis

# Test Redis connectivity
redis-cli -h localhost -p 6379 ping

# Check Redis logs
docker-compose logs redis
```

#### SFTP Issues

**Issue**: SFTP connection refused
```bash
# Check SFTP container status
docker-compose ps sftp

# Test SSH connectivity
ssh -p 2001 testuser@localhost

# Check SFTP logs
docker-compose logs sftp
```

**Issue**: Permission denied
```bash
# Check user exists in container
docker-compose exec sftp id testuser

# Reset user password
docker-compose exec sftp passwd testuser
```

### Performance Monitoring

#### Container Resource Usage
```bash
# Monitor real-time resource usage
docker stats mysql redis sftp

# Check container processes
docker-compose top
```

#### Service-Specific Monitoring

##### MySQL Performance
```sql
-- Check active connections
SHOW PROCESSLIST;

-- Check database sizes
SELECT 
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables 
GROUP BY table_schema;

-- Check slow queries
SHOW VARIABLES LIKE 'slow_query_log';
```

##### Redis Performance
```bash
# Redis info command
redis-cli -h localhost -p 6379 info

# Monitor Redis commands
redis-cli -h localhost -p 6379 monitor

# Check memory usage
redis-cli -h localhost -p 6379 info memory
```

### Backup and Recovery

#### MySQL Backup
```bash
# Create backup
docker-compose exec mysql mysqldump -u root -p test > backup.sql

# Restore backup
docker-compose exec -T mysql mysql -u root -p test < backup.sql
```

#### Redis Backup
```bash
# Create RDB snapshot
docker-compose exec redis redis-cli BGSAVE

# Copy RDB file
docker cp redis:/data/dump.rdb ./redis_backup.rdb
```

### Security Considerations

1. **Change Default Passwords**: Update all default passwords in production
2. **Network Security**: Use custom Docker networks to isolate services
3. **SSL/TLS**: Enable SSL for MySQL and Redis in production
4. **SSH Keys**: Use SSH key authentication instead of passwords for SFTP
5. **Firewall Rules**: Restrict access to service ports
6. **Regular Updates**: Keep base images updated

---

## Conclusion

This documentation covers all public APIs, functions, and components available in the Docker multi-service setup. Each service provides standard APIs and protocols:

- **MySQL**: Standard SQL interface with custom configuration
- **Redis**: Standard Redis protocol with full command support  
- **SFTP**: Standard SSH/SFTP protocol with file transfer capabilities

For additional support or feature requests, please refer to the official documentation of each service or submit issues to the project repository.