# Configuration Reference

## Overview

This document provides detailed configuration information for all services in the Docker multi-service setup. It includes parameter explanations, default values, and tuning recommendations for production environments.

## Table of Contents

1. [MySQL Configuration](#mysql-configuration)
2. [Redis Configuration](#redis-configuration)
3. [SFTP Configuration](#sftp-configuration)
4. [Docker Compose Configuration](#docker-compose-configuration)
5. [Environment Variables](#environment-variables)
6. [Performance Tuning](#performance-tuning)

---

## MySQL Configuration

### Configuration File: `mysql/mysqld.cnf`

#### Client Configuration
```ini
[client]
port=3306                          # Default MySQL port
socket = /var/run/mysqld/mysqld.sock  # Unix socket path
```

#### MySQL Client Configuration
```ini
[mysql]
no-auto-rehash                     # Disable automatic rehashing (faster startup)
auto-rehash                        # Enable automatic rehashing (overrides no-auto-rehash)
default-character-set = utf8mb4    # Default character set for client
```

#### Server Configuration
```ini
[mysqld]
###basic settings
server-id = 2                      # Unique server identifier for replication
pid-file = /var/run/mysqld/mysqld.pid    # Process ID file location
socket = /var/run/mysqld/mysqld.sock     # Unix socket path
datadir = /var/lib/mysql                 # Data directory location
symbolic-links = 0                       # Disable symbolic links (security)
character-set-server = utf8mb4           # Default server character set
sql_mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"  # SQL mode settings
default-storage-engine = INNODB          # Default storage engine
transaction_isolation = READ-COMMITTED   # Default transaction isolation level
```

#### Connection and Timeout Settings
```ini
auto_increment_offset = 1          # Auto increment offset for replication
connect_timeout = 20               # Connection timeout in seconds
max_connections = 3500             # Maximum concurrent connections
wait_timeout = 86400               # Non-interactive connection timeout (24 hours)
interactive_timeout = 7200         # Interactive connection timeout (2 hours)
log_bin_trust_function_creators = 1 # Allow non-deterministic functions in binary log
```

#### Buffer and Cache Settings
```ini
sort_buffer_size = 32M             # Sort buffer size for ORDER BY operations
join_buffer_size = 128M            # Buffer size for full joins
max_allowed_packet = 1024M         # Maximum packet size (1GB)
tmp_table_size = 2097152           # Maximum size for temporary tables in memory
read_buffer_size = 16M             # Buffer size for sequential table scans
read_rnd_buffer_size = 32M         # Buffer size for random reads
query_cache_type = 1               # Enable query cache
query_cache_size = 2M              # Query cache size
table_open_cache = 1500            # Number of cached table file descriptors
table_definition_cache = 1000      # Number of cached table definitions
thread_cache_size = 768            # Number of cached connection threads
back_log = 3000                    # Connection backlog queue size
open_files_limit = 65536           # Maximum number of open files
skip-name-resolve                  # Skip DNS lookups for faster connections
```

#### Logging Configuration
```ini
########log settings########
log-output = FILE                  # Log output destination
general_log = ON                   # Enable general query log
general_log_file = /var/lib/mysql/general.log  # General log file location
slow_query_log = ON                # Enable slow query log
slow_query_log_file = /var/lib/mysql/slowquery.log  # Slow query log location
long_query_time = 10               # Slow query threshold (10 seconds)
log_queries_not_using_indexes = OFF # Don't log queries without indexes
log_throttle_queries_not_using_indexes = 0  # No throttling for index-less queries
min_examined_row_limit = 100       # Minimum rows examined for slow log
```

#### InnoDB Storage Engine Settings
```ini
########innodb settings########
innodb_io_capacity = 4000          # I/O operations per second capability
innodb_io_capacity_max = 8000      # Maximum I/O operations per second
innodb_buffer_pool_size = 6144M    # InnoDB buffer pool size (6GB)
innodb_file_per_table = on         # Use separate file for each table
innodb_buffer_pool_instances = 20  # Number of buffer pool instances
innodb_buffer_pool_load_at_startup = 1  # Load buffer pool at startup
innodb_buffer_pool_dump_at_shutdown = 1 # Dump buffer pool at shutdown
innodb_log_file_size = 300M        # Size of each log file
innodb_log_files_in_group = 2      # Number of log files in group
innodb_log_buffer_size = 16M       # Log buffer size
innodb_undo_logs = 128             # Number of undo logs
innodb_flush_method = O_DIRECT     # File flush method (bypasses OS cache)
innodb_flush_neighbors = 1         # Flush neighboring pages
innodb_purge_threads = 4           # Number of purge threads
innodb_large_prefix = 1            # Enable large index key prefixes
innodb_thread_concurrency = 64     # Maximum concurrent threads
innodb_print_all_deadlocks = 1     # Log all deadlocks
innodb_strict_mode = 1             # Enable strict mode
innodb_sort_buffer_size = 64M      # Sort buffer size for index creation
innodb_flush_log_at_trx_commit = 1 # Flush log at transaction commit
innodb_autoextend_increment = 64   # Tablespace auto-extend increment
innodb_concurrency_tickets = 5000  # Concurrency tickets
innodb_old_blocks_time = 1000      # Time for old blocks promotion
innodb_open_files = 65536          # Maximum open files for InnoDB
innodb_stats_on_metadata = 0       # Don't update statistics on metadata access
innodb_checksum_algorithm = 0      # Checksum algorithm
innodb_data_file_path = ibdata1:12M:autoextend  # Data file configuration
```

#### Additional Settings
```ini
delayed_insert_limit = 100         # Delayed insert limit
delayed_insert_timeout = 300       # Delayed insert timeout
delayed_queue_size = 1000          # Delayed insert queue size
delay_key_write = ON               # Delay key writes for MyISAM
disconnect_on_expired_password = ON # Disconnect on expired password
div_precision_increment = 4        # Division precision increment
end_markers_in_json = OFF          # End markers in JSON
eq_range_index_dive_limit = 10     # Range index dive limit
innodb_adaptive_flushing = ON      # Adaptive flushing
innodb_adaptive_hash_index = ON    # Adaptive hash index
innodb_adaptive_max_sleep_delay = 150000  # Maximum adaptive sleep delay
innodb_autoinc_lock_mode = 1       # Auto-increment lock mode
```

### Tuning Recommendations

#### For High-Traffic Applications
```ini
max_connections = 5000             # Increase for more concurrent users
innodb_buffer_pool_size = 8192M    # Set to 70-80% of available RAM
query_cache_size = 64M             # Increase query cache for read-heavy workloads
thread_cache_size = 1000           # Increase thread cache
table_open_cache = 2000            # Increase table cache
```

#### For Memory-Constrained Environments
```ini
max_connections = 1000             # Reduce concurrent connections
innodb_buffer_pool_size = 1024M    # Reduce buffer pool size
query_cache_size = 32M             # Reduce query cache
join_buffer_size = 64M             # Reduce join buffer
sort_buffer_size = 16M             # Reduce sort buffer
```

---

## Redis Configuration

### Configuration File: `redis/redis.conf`

#### Network Configuration
```ini
bind * -::*                        # Bind to all interfaces (IPv4 and IPv6)
protected-mode yes                 # Enable protected mode
port 6379                          # Default Redis port
tcp-backlog 511                    # TCP listen backlog
timeout 0                          # Client idle timeout (0 = disabled)
tcp-keepalive 300                  # TCP keepalive interval
```

#### General Configuration
```ini
daemonize no                       # Don't run as daemon (for Docker)
pidfile /var/run/redis_6379.pid    # PID file location
loglevel notice                    # Log level (debug, verbose, notice, warning)
logfile ""                         # Log file (empty = stdout)
databases 16                       # Number of databases
always-show-logo no                # Don't show ASCII logo
set-proc-title yes                 # Modify process title
proc-title-template "{title} {listen-addr} {server-mode}"  # Process title template
```

#### Snapshotting Configuration
```ini
# Default save points (commented out in config)
# save 3600 1                      # Save after 1 hour if 1+ keys changed
# save 300 100                     # Save after 5 minutes if 100+ keys changed
# save 60 10000                    # Save after 1 minute if 10000+ keys changed

stop-writes-on-bgsave-error yes    # Stop writes if background save fails
rdbcompression yes                 # Compress RDB files
rdbchecksum yes                    # Checksum RDB files
dbfilename dump.rdb                # RDB filename
rdb-del-sync-files no              # Keep RDB files for replication
dir ./                             # Working directory
```

#### Security Configuration
```ini
# Authentication (disabled by default)
# requirepass foobared             # Set password for authentication

# Access Control Lists (ACLs)
# User management for fine-grained access control
# Example: user default on +@all ~* &* -@dangerous +acl +info +config|get
```

#### Memory Management
```ini
# maxmemory <bytes>                # Maximum memory usage limit
# maxmemory-policy noeviction      # Memory eviction policy options:
#   - noeviction: Return errors when memory limit reached
#   - allkeys-lru: Remove any key according to LRU algorithm
#   - volatile-lru: Remove volatile keys according to LRU
#   - allkeys-random: Remove random keys
#   - volatile-random: Remove random volatile keys
#   - volatile-ttl: Remove volatile keys with nearest TTL
```

#### Persistence Configuration
```ini
# Append Only File (AOF) persistence
# appendonly no                    # Enable AOF persistence
# appendfilename "appendonly.aof"  # AOF filename
# appendfsync everysec             # AOF fsync policy:
#   - always: Fsync every write (slow, safe)
#   - everysec: Fsync every second (fast, safe)
#   - no: Never fsync (fast, unsafe)
```

#### Replication Configuration
```ini
# Master-Replica replication
# replicaof <masterip> <masterport>  # Make this instance a replica
# masterauth <master-password>       # Master password
# masteruser <username>              # Master username (for ACLs)
# replica-serve-stale-data yes       # Serve stale data when disconnected
# replica-read-only yes              # Make replica read-only
```

#### Client Configuration
```ini
# maxclients 10000                 # Maximum number of clients
# timeout 0                        # Client idle timeout
```

### Performance Tuning

#### For High-Throughput Applications
```ini
tcp-backlog 2048                   # Increase TCP backlog
maxclients 20000                   # Increase maximum clients
save ""                            # Disable RDB snapshots for better performance
appendonly yes                     # Use AOF for persistence
appendfsync everysec               # Balance between safety and performance
```

#### For Memory-Optimized Workloads
```ini
maxmemory 2gb                      # Set memory limit
maxmemory-policy allkeys-lru       # Use LRU eviction
hash-max-ziplist-entries 512       # Optimize hash encoding
hash-max-ziplist-value 64          # Optimize hash values
list-max-ziplist-size -2           # Optimize list encoding
```

---

## SFTP Configuration

### Dockerfile Configuration: `sftp/dockerfile`

#### Base Image and Packages
```dockerfile
FROM ubuntu:latest                 # Base Ubuntu image
RUN apt-get update && \            # Update package lists
    apt-get install -y openssh-server && \  # Install SSH server
    apt-get install -y vim         # Install text editor
```

#### SSH Service Setup
```dockerfile
RUN mkdir /var/run/sshd           # Create SSH runtime directory
```

#### User Configuration
```dockerfile
RUN useradd -m testuser && \      # Create user with home directory
    echo "testuser:testpasswd" | chpasswd  # Set user password
```

#### SSH Configuration
```dockerfile
# SSH server configuration
RUN echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config
RUN echo 'RSAAuthentication yes' >> /etc/ssh/sshd_config
RUN echo 'PubkeyAcceptedAlgorithms +ssh-rsa' >> /etc/ssh/sshd_config
RUN echo 'AuthorizedKeysFile .ssh/authorized_keys' >> /etc/ssh/sshd_config
RUN echo 'KexAlgorithms +diffie-hellman-group1-sha1' >> /etc/ssh/sshd_config
```

#### Security Settings
```dockerfile
USER testuser                     # Switch to non-root user
RUN mkdir -p /home/testuser/.ssh   # Create SSH directory for keys
USER root                         # Switch back to root for service startup
```

#### Service Configuration
```dockerfile
EXPOSE 22                         # Expose SSH port
CMD ["/usr/sbin/sshd", "-D"]      # Start SSH daemon
```

### SSH Configuration Parameters

#### Authentication Methods
- **PubkeyAuthentication**: Enabled for SSH key authentication
- **RSAAuthentication**: Enabled for RSA key support
- **PasswordAuthentication**: Enabled by default (not explicitly disabled)

#### Key Exchange and Algorithms
- **PubkeyAcceptedAlgorithms**: Includes ssh-rsa for compatibility
- **KexAlgorithms**: Includes diffie-hellman-group1-sha1 for legacy support
- **AuthorizedKeysFile**: Standard location for authorized public keys

#### User Configuration
- **Username**: testuser
- **Password**: testpasswd
- **Home Directory**: /home/testuser
- **Shell**: Default system shell
- **SSH Directory**: /home/testuser/.ssh

### Security Hardening Recommendations

#### For Production Environments
```bash
# Disable password authentication
echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config

# Use only secure key exchange algorithms
echo 'KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group14-sha256' >> /etc/ssh/sshd_config

# Use only secure ciphers
echo 'Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr' >> /etc/ssh/sshd_config

# Use only secure MACs
echo 'MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com' >> /etc/ssh/sshd_config

# Disable root login
echo 'PermitRootLogin no' >> /etc/ssh/sshd_config

# Set maximum authentication attempts
echo 'MaxAuthTries 3' >> /etc/ssh/sshd_config

# Set client alive interval
echo 'ClientAliveInterval 300' >> /etc/ssh/sshd_config
echo 'ClientAliveCountMax 2' >> /etc/ssh/sshd_config
```

---

## Docker Compose Configuration

### File: `docker-compose.yaml`

#### Version and Services Structure
```yaml
version: '3'                       # Docker Compose file version

services:                          # Services definition
```

#### MySQL Service Configuration
```yaml
mysql:
  image: mysql:1                   # Custom image tag
  container_name: mysql            # Container name
  build: ./mysql                   # Build context
  volumes:                         # Volume mounts
    - ~/docker_data/mysql/data:/var/lib/mysql
    - ~/docker_data/mysql/init:/docker-entrypoint-initdb.d/
  environment:                     # Environment variables
    - MYSQL_ROOT_PASSWORD=ThisIsAComplicatePassWord
  ports:                           # Port mappings
    - "3306:3306"
  restart: always                  # Restart policy
  command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

#### Redis Service Configuration
```yaml
redis:
  image: redis:1                   # Custom image tag
  container_name: redis            # Container name
  build: ./redis                   # Build context
  ports:                           # Port mappings
    - "6379:6379"
  restart: always                  # Restart policy
```

#### SFTP Service Configuration
```yaml
sftp:
  image: sftp:1                    # Custom image tag
  container_name: sftp             # Container name
  build: ./sftp                    # Build context
  ports:                           # Port mappings
    - "2001:22"
  network_mode: bridge             # Network mode
  # restart: always                # Restart disabled (commented out)
```

### Volume Configuration

#### MySQL Volumes
- **Data Volume**: `~/docker_data/mysql/data:/var/lib/mysql`
  - **Purpose**: Persistent storage for MySQL data files
  - **Permissions**: Should be owned by mysql user (999:999)
  - **Backup**: Essential for data recovery

- **Init Volume**: `~/docker_data/mysql/init:/docker-entrypoint-initdb.d/`
  - **Purpose**: Initialization scripts (SQL files)
  - **Execution**: Scripts run on first container start
  - **Order**: Executed alphabetically

### Network Configuration

#### Port Mappings
```yaml
ports:
  mysql: "3306:3306"               # MySQL standard port
  redis: "6379:6379"               # Redis standard port
  sftp: "2001:22"                  # SFTP mapped to non-standard port
```

#### Network Mode
- **Default**: Bridge network for mysql and redis
- **SFTP**: Explicitly set to bridge mode
- **Communication**: Services can communicate using container names

### Container Management

#### Restart Policies
- **mysql**: `always` - Always restart on failure
- **redis**: `always` - Always restart on failure
- **sftp**: Disabled (commented out) - Manual restart only

#### Build Context
Each service builds from its respective directory:
- **MySQL**: `./mysql/` (contains dockerfile and configuration)
- **Redis**: `./redis/` (contains dockerfile and redis.conf)
- **SFTP**: `./sftp/` (contains dockerfile only)

---

## Environment Variables

### MySQL Environment Variables
```bash
MYSQL_ROOT_PASSWORD=ThisIsAComplicatePassWord  # Root user password
```

#### Additional MySQL Environment Variables (Available)
```bash
MYSQL_DATABASE=database_name       # Create database on startup
MYSQL_USER=username               # Create additional user
MYSQL_PASSWORD=password           # Password for additional user
MYSQL_ALLOW_EMPTY_PASSWORD=yes    # Allow empty root password
MYSQL_RANDOM_ROOT_PASSWORD=yes    # Generate random root password
MYSQL_ONETIME_PASSWORD=yes        # Expire root password on first login
```

### Redis Environment Variables
No specific environment variables are set for Redis in the current configuration.

#### Available Redis Environment Variables
```bash
REDIS_PASSWORD=password           # Set Redis password
REDIS_DATABASES=16                # Number of databases
REDIS_MAXMEMORY=2gb              # Maximum memory usage
REDIS_MAXMEMORY_POLICY=allkeys-lru # Memory eviction policy
```

### SFTP Environment Variables
No environment variables are configured. User credentials are set during image build.

#### Potential SFTP Environment Variables
```bash
SFTP_USERS=user1:pass1:uid:gid:dir,user2:pass2:uid:gid:dir  # Multiple users
SFTP_CHROOT_DIR=/data             # Chroot directory
```

---

## Performance Tuning

### System-Level Optimizations

#### Host System Settings
```bash
# Increase file descriptor limits
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

# Optimize kernel parameters for databases
echo "vm.swappiness = 1" >> /etc/sysctl.conf
echo "net.core.somaxconn = 65535" >> /etc/sysctl.conf
echo "net.ipv4.tcp_max_syn_backlog = 65535" >> /etc/sysctl.conf

# Apply changes
sysctl -p
```

#### Docker Resource Limits
```yaml
# Add to docker-compose.yaml services
deploy:
  resources:
    limits:
      cpus: '2.0'                  # CPU limit
      memory: 4G                   # Memory limit
    reservations:
      cpus: '1.0'                  # CPU reservation
      memory: 2G                   # Memory reservation
```

### Service-Specific Tuning

#### MySQL Performance Tuning
```ini
# For SSD storage
innodb_flush_method = O_DIRECT
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

# For HDD storage
innodb_flush_method = fsync
innodb_io_capacity = 200
innodb_io_capacity_max = 400

# For read-heavy workloads
query_cache_type = 1
query_cache_size = 128M
read_buffer_size = 32M

# For write-heavy workloads
innodb_log_file_size = 512M
innodb_log_buffer_size = 32M
innodb_flush_log_at_trx_commit = 2
```

#### Redis Performance Tuning
```ini
# For high-frequency operations
save ""                           # Disable snapshots
appendonly yes                    # Use AOF
appendfsync no                    # Best performance, least safety

# For memory efficiency
maxmemory-policy allkeys-lru
hash-max-ziplist-entries 512
list-max-ziplist-size -2
```

#### SFTP Performance Tuning
```bash
# SSH configuration optimizations
echo 'Compression yes' >> /etc/ssh/sshd_config
echo 'TCPKeepAlive yes' >> /etc/ssh/sshd_config
echo 'MaxStartups 100:30:200' >> /etc/ssh/sshd_config
echo 'MaxSessions 100' >> /etc/ssh/sshd_config
```

### Monitoring and Metrics

#### Container Monitoring
```bash
# Real-time resource usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Log monitoring
docker-compose logs -f --tail=100

# Health checks
docker-compose ps
```

#### Service-Specific Monitoring
```bash
# MySQL monitoring
docker-compose exec mysql mysqladmin -u root -p status
docker-compose exec mysql mysqladmin -u root -p processlist

# Redis monitoring
docker-compose exec redis redis-cli info stats
docker-compose exec redis redis-cli info memory

# SFTP monitoring
docker-compose exec sftp netstat -an | grep :22
docker-compose exec sftp ps aux | grep sshd
```

This configuration reference provides comprehensive details for tuning and optimizing each service according to specific requirements and environments.