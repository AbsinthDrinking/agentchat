# Docker 部署

## 前置条件

- Docker 20.10+
- Docker Compose 2.0+
- 可用内存 ≥ 4GB

## 快速启动

```bash
# 1. 配置（必须）
vim docker/docker_config.yaml

# 2. 启动
cd docker
docker-compose up --build -d
```

## 服务端口

| 服务 | 地址 |
|------|------|
| 前端 | `http://localhost:8090` |
| 后端 API | `http://localhost:7860` |
| Swagger | `http://localhost:7860/docs` |
| MySQL | `localhost:3306` |
| Redis | `localhost:6379` |

## 配置文件

核心配置在 `docker/docker_config.yaml`，Docker 环境下数据库主机名需使用服务名（`mysql`、`redis`）：

```yaml
mysql:
  endpoint: "mysql+pymysql://root:password@mysql:3306/agentchat"
  async_endpoint: "mysql+aiomysql://root:password@mysql:3306/agentchat"

redis:
  endpoint: "redis://redis:6379"

multi_models:
  conversation_model:
    api_key: "your-api-key"
    base_url: "https://api.openai.com/v1"
    model_name: "gpt-4"
  embedding:
    api_key: "your-api-key"
    base_url: "https://api.openai.com/v1"
    model_name: "text-embedding-3-small"
```

## 常用命令

```bash
# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f backend

# 重启某服务
docker-compose restart backend

# 停止全部
docker-compose down
```

## 故障排查

**服务启动失败**
```bash
docker-compose logs backend    # 查看日志
docker-compose build --no-cache backend  # 无缓存重建
```

**端口冲突**
```bash
lsof -i :7860
# 修改 docker-compose.yml 端口映射
```

**数据库连接失败**
```bash
docker-compose ps mysql
docker-compose exec mysql mysql -u root -p
docker-compose down -v && docker-compose up -d  # 重置数据
```
