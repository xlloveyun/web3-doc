在 Ubuntu 中使用 Docker Compose 安装 PostgreSQL 14 的完整指南如下：

### 1. 准备工作

确保已安装 Docker 和 Docker Compose：
```bash
sudo apt update
sudo apt install docker.io docker-compose-plugin
sudo systemctl enable --now docker
```

### 2. 创建项目目录
```bash
mkdir -p ~/postgres14/{data,init} && cd ~/postgres14
```

### 3. 创建 `docker-compose.yml` 文件

```bash
nano docker-compose.yml
```

粘贴以下内容：

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: postgres14
    environment:
      POSTGRES_USER: myuser       # 自定义用户名
      POSTGRES_PASSWORD: mypassword  # 自定义密码
      POSTGRES_DB: mydb          # 初始数据库名
    volumes:
      - ./data:/var/lib/postgresql/data  # 数据持久化
      - ./init:/docker-entrypoint-initdb.d  # 初始化脚本目录
    ports:
      - "5432:5432"               # 主机端口:容器端口
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### 4. (可选) 添加初始化脚本

如果需要数据库初始化时自动创建表或导入数据：
```bash
nano init/init.sql
```

示例内容：
```sql
CREATE TABLE test_yiquan (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL
);

INSERT INTO test_yiquan (username) VALUES ('admin');
```

### 5. 启动服务
```bash
docker compose up -d
```

### 6. 验证安装

检查容器状态：
```bash
docker compose ps
```

查看日志：
```bash
docker compose logs -f
```

测试连接：
```bash
sudo apt install postgresql-client  # 如果尚未安装客户端
psql -h 127.0.0.1 -U myuser -d mydb
```

### 7. 管理命令

停止服务：
```bash
docker compose down
```

完全删除（包括数据）：
```bash
docker compose down -v
```

### 8. 重要配置说明

1. **数据持久化**：
   - `./data` 目录会永久存储数据库文件
   - 即使容器删除，数据也不会丢失

2. **安全建议**：
   - 修改默认的 `myuser/mypassword`
   - 生产环境建议：
     ```yaml
     environment:
       POSTGRES_HOST_AUTH_METHOD: scram-sha-256
     ```

3. **性能调优**：
   在 `init` 目录下创建 `postgresql.conf` 文件可覆盖默认配置：
   ```bash
   nano init/postgresql.conf
   ```
   示例内容：
   ```
   shared_buffers = 256MB
   work_mem = 16MB
   ```

### 9. 备份数据

```bash
docker exec postgres14 pg_dump -U myuser -d mydb > backup.sql
```

### 10. 恢复数据

```bash
cat backup.sql | docker exec -i postgres14 psql -U myuser -d mydb
```

这种安装方式比直接 `apt install` 更干净，不会影响系统已有的 PostgreSQL 安装，且便于管理和迁移。