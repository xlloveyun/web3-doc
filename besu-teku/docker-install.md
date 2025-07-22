以下是详细的 **Docker Compose 配置与执行指南**，涵盖 `docker-compose.yml` 编写、JWT 密钥配置、目录挂载、服务启动和验证全流程。

---

### 一、准备工作
#### 1. 创建项目目录结构
```bash
mkdir -p ~/sepolia-node/{besu_data,teku_data,config} && cd ~/sepolia-node
```
- `besu_data`: Besu 区块链数据存储
- `teku_data`: Teku 信标链数据存储
- `config`: 配置文件目录

#### 2. 生成 JWT 密钥文件
```bash
openssl rand -hex 32 > config/jwt.hex
chmod 644 config/jwt.hex  # 确保权限正确
```

---

### 二、编写 `docker-compose.yml`
```bash
nano docker-compose.yml
```
粘贴以下内容：

```yaml
version: '3.8'

services:
  # 1. Besu (执行层)
  besu:
    image: hyperledger/besu:latest
    container_name: besu_sepolia
    restart: unless-stopped
    ports:
      - "8545:8545"   # JSON-RPC
      - "8546:8546"   # WebSocket
      - "30303:30303" # P2P (TCP/UDP)
      - "30303:30303/udp"
    volumes:
      - ./besu_data:/var/lib/besu
      - ./config/jwt.hex:/jwt.hex
    command:
      - "--network=sepolia"
      - "--sync-mode=SNAP"
      - "--data-storage-format=BONSAI"
      - "--rpc-http-enabled=true"
      - "--rpc-http-host=0.0.0.0"
      - "--rpc-http-api=ETH,NET,WEB3,ADMIN,ENGINE"
      - "--engine-rpc-enabled=true"
      - "--engine-host-allowlist=*"
      - "--engine-jwt-secret=/jwt.hex"
      - "--metrics-enabled=true"  # 可选：启用指标

  # 2. Teku (共识层)
  teku:
    image: consensys/teku:latest
    container_name: teku_sepolia
    restart: unless-stopped
    depends_on:
      - besu
    ports:
      - "9000:9000"  # P2P
      - "5051:5051"  # REST API
    volumes:
      - ./teku_data:/var/lib/teku
      - ./config/jwt.hex:/jwt.hex
    command:
      - "--network=sepolia"
      - "--ee-endpoint=http://besu:8551"
      - "--ee-jwt-secret-file=/jwt.hex"
      - "--metrics-enabled=true"
      - "--data-storage-mode=PRUNE"
      - "--p2p-enabled=true"
      - "--rest-api-enabled=true"
      - "--checkpoint-sync-url=https://sepolia.beaconstate.info"
      - "--validators-proposer-default-fee-recipient=0xYOUR_ETH_ADDRESS"  # 替换为你的地址

volumes:
  besu_data:
  teku_data:
```

---

### 三、关键配置说明
#### 1. **Besu 参数**
| 参数 | 作用 |
|------|------|
| `--network=sepolia` | 连接 Sepolia 测试网 |
| `--engine-jwt-secret=/jwt.hex` | JWT 认证密钥路径 |
| `--sync-mode=X_SNAP` | 快照同步（比全同步快 5-10 倍） |

#### 2. **Teku 参数**
| 参数 | 作用 |
|------|------|
| `--ee-endpoint=http://besu:8551` | 连接 Besu 的 Engine API |
| `--checkpoint-sync-url` | 从检查点快速同步（节省数天时间） |
| `--validators-proposer-default-fee-recipient` | 设置手续费接收地址 |

---

### 四、启动与监控
#### 1. 启动服务
```bash
docker compose up -d
```

#### 2. 查看实时日志
```bash
# Besu 日志
docker logs -f besu_sepolia

# Teku 日志
docker logs -f teku_sepolia
```

#### 3. 验证同步状态
```bash
# 检查 Besu 同步
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545

# 检查 Teku 同步
curl http://localhost:5051/eth/v1/node/syncing
```

#### 4. 停止服务
```bash
docker compose down  # 保留数据
docker compose down -v  # 彻底删除数据
```

---

### 五、高级配置
#### 1. 自定义检查点同步源
修改 Teku 的 `--checkpoint-sync-url`：
```yaml
command:
      - "--checkpoint-sync-url=https://sepolia.beaconstate.info"
```

#### 2. 启用 Prometheus 监控
在 `docker-compose.yml` 中添加：
```yaml
# Besu 配置
command:
  - "--metrics-enabled=true"
  - "--metrics-host=0.0.0.0"
  - "--metrics-port=6060"

# Teku 配置
command:
  - "--metrics-enabled=true"
  - "--metrics-host=0.0.0.0"
  - "--metrics-port=8008"
```

---

### 六、故障排查
#### 1. 同步卡住
- **Besu**: 检查磁盘 IO 性能（建议 SSD）
- **Teku**: 尝试更换 `checkpoint-sync-url`

#### 2. JWT 认证失败
```bash
# 确认密钥文件内容
cat config/jwt.hex
# 确认容器内权限
docker exec -it besu_sepolia ls -la /jwt.hex
```

#### 3. 端口冲突
```bash
ss -tulnp | grep -E '8545|8546|30303|9000|5051'
# 修改 docker-compose.yml 中的端口映射
```

---

### 七、最终目录结构
```
~/sepolia-node/
├── besu_data/       # Besu 区块链数据
├── teku_data/       # Teku 信标链数据
├── config/
│   └── jwt.hex      # JWT 密钥
└── docker-compose.yml
```

通过以上步骤，您将获得一个完全配置好的 **Sepolia 测试网节点**，同时运行执行层（Besu）和共识层（Teku）。