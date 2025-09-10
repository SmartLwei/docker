# Docker 服务部署

使用 Docker 部署 MySQL 8.0 和 Redis 7.x 服务的完整解决方案。

## 🚀 特性

- **MySQL 8.0**: 最新稳定版本，优化配置
- **Redis 7.x**: 最新版本，启用 AOF 持久化
- **健康检查**: 自动监控服务状态
- **数据持久化**: 数据卷确保数据安全
- **网络隔离**: 独立网络提高安全性
- **环境变量**: 安全的密码管理
- **安全增强**: 启用安全选项和保护模式

## 📋 环境要求

- Docker >= 20.10
- Docker Compose >= 2.0
- 至少 4GB 可用内存
- 至少 10GB 可用磁盘空间

## 🛠️ 快速开始

### 1. 克隆项目

```bash
git clone <your-repo-url>
cd docker
```

### 2. 配置环境变量

复制并编辑环境变量文件：

```bash
cp .env.example .env
# 编辑 .env 文件，设置安全的密码
```

### 3. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 4. 验证服务

**MySQL 连接测试：**
```bash
# 使用 root 用户连接
docker exec -it mysql mysql -uroot -p

# 使用 test 用户连接
docker exec -it mysql mysql -utest -p test
```

**Redis 连接测试：**
```bash
# 连接 Redis
docker exec -it redis redis-cli
# 输入密码
AUTH YourSecureRedisPassword2024!
# 测试
PING
```

## 📊 服务信息

### MySQL 8.0

- **端口**: 3306
- **数据库**: test
- **用户**: test
- **密码**: 在 .env 文件中配置
- **数据卷**: mysql_data
- **配置文件**: `./mysql/mysqld.cnf`
- **初始化脚本**: `./mysql/test.sql`

**连接信息：**
```
Host: localhost
Port: 3306
Database: test
Username: test
Password: [在.env中设置]
```

### Redis 7.x

- **端口**: 6379
- **密码**: 在 .env 文件中配置
- **数据卷**: redis_data
- **配置文件**: `./redis/redis.conf`
- **持久化**: RDB + AOF

**连接信息：**
```
Host: localhost
Port: 6379
Password: ThisIsAComplicatePassWord
```

## 🔄 更新和维护

### 更新服务

```bash
# 拉取最新镜像
docker-compose pull

# 重启服务
docker-compose up -d
```

### 备份数据

```bash
# 备份 MySQL
docker exec mysql mysqldump -uroot -p test > backup.sql

# 备份 Redis
docker exec redis redis-cli BGSAVE
```

## 📁 目录结构

```
docker/
├── docker-compose.yaml    # Docker Compose 配置
├── .env                   # 环境变量文件
├── README.md             # 说明文档
├── mysql/
│   ├── mysqld.cnf        # MySQL 配置文件
│   └── test.sql          # 初始化 SQL 脚本
└── redis/
    └── redis.conf        # Redis 配置文件
```

## ⚙️ 配置说明

### MySQL 配置优化

- 字符集：utf8mb4
- 存储引擎：InnoDB
- 缓冲池：1GB
- 连接数：1000
- 慢查询日志：启用
- 二进制日志：启用

### Redis 配置优化

- 内存限制：2GB
- 淘汰策略：allkeys-lru
- 持久化：RDB + AOF
- 密码保护：启用
- 最大连接数：10000

## 🔒 安全建议

1. 请务必修改默认密码
2. 生产环境中限制访问IP
3. 定期备份重要数据
4. 监控日志和异常访问

## 🧪 故障排除

### 常见问题

**1. 端口冲突**
```bash
# 检查端口占用情况
lsof -i :3306
lsof -i :6379
```

**2. 容器无法启动**
```bash
# 查看容器日志
docker-compose logs mysql
docker-compose logs redis
```

**3. 连接被拒绝**
- 检查服务是否正常运行
- 检查防火墙设置
- 确认密码是否正确

### 性能调优

**MySQL 性能监控：**
```sql
-- 查看连接数
SHOW STATUS LIKE 'Threads_connected';

-- 查看慢查询
SHOW STATUS LIKE 'Slow_queries';

-- 查看缓冲池状态
SHOW STATUS LIKE 'Innodb_buffer_pool%';
```

**Redis 性能监控：**
```bash
# 查看内存使用
redis-cli INFO memory

# 查看连接数
redis-cli INFO clients

# 查看命令统计
redis-cli INFO commandstats
```

## 📝 更新日志

- **v2.0**: 升级到 MySQL 8.0 和 Redis 7.x
- **v1.5**: 添加健康检查和数据持久化
- **v1.0**: 初始版本

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

MIT License
