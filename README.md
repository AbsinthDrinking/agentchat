# AgentChat

基于大语言模型的智能对话系统，支持多模型接入、知识库检索、工具调用与 MCP 协议。

## 功能

- 多模型支持：可同时配置对话模型、工具调用模型、Embedding 模型
- 智能体系统：支持子智能体协作、工具多轮调用、任务编排
- 知识库：基于 RAG 的语义检索与问答
- MCP Server：完整支持 Model Context Protocol，支持从 OpenAPI 文档对话式生成
- 人机协同：关键节点支持人工介入确认
- 工作区：多工作区切换，应用中心管理
- 数据看板：按 Agent、模型、时间范围查看调用统计

## 技术栈

| 层级 | 技术 |
|------|------|
| 后端框架 | FastAPI (Python 3.12+) |
| 前端框架 | Vue 3.4 + Element Plus + TypeScript |
| 数据库 | MySQL 8.0 + Redis 7.0 |
| 向量库 | ChromaDB |
| 对象存储 | MinIO |
| 部署 | Docker + Docker Compose |

## 快速开始

### 环境要求

- Python 3.12+ / Node.js 18+
- MySQL 8.0+ / Redis 7.0+
- Docker 20.10+ (可选)

### 本地启动

```bash
# 后端
cd src/backend
pip install uv && uv sync
uv run uvicorn agentchat.main:app --port 7860 --reload

# 前端
cd src/frontend
npm install && npm run dev
```

### Docker 部署

```bash
cd docker
# 编辑 docker_config.yaml 填入数据库和模型配置
vim docker_config.yaml
docker-compose up --build -d
```

启动后访问：
- 前端：`http://localhost:8090`
- 后端 API：`http://localhost:7860`
- Swagger 文档：`http://localhost:7860/docs`

## 项目结构

```
AgentChat/
├── src/
│   ├── backend/agentchat/    # 后端
│   │   ├── api/              # API 路由
│   │   ├── services/         # 业务逻辑
│   │   ├── database/         # 数据库 DAO
│   │   ├── tools/            # 内置工具
│   │   ├── mcp_servers/      # MCP 服务端
│   │   └── prompts/          # Prompt 模板
│   └── frontend/src/         # 前端
│       ├── pages/            # 页面组件
│       ├── components/       # 通用组件
│       ├── apis/             # API 封装
│       └── store/            # 状态管理
├── docker/                   # Docker 部署文件
└── scripts/                  # 辅助脚本
```

## 配置

核心配置文件为 `src/backend/agentchat/config.yaml`，主要配置项：

```yaml
mysql:
  endpoint: "mysql+pymysql://root:password@localhost:3306/agentchat"

redis:
  endpoint: "redis://localhost:6379"

multi_models:
  conversation_model:
    api_key: "your-key"
    base_url: "https://api.openai.com/v1"
    model_name: "gpt-4"
  embedding:
    api_key: "your-key"
    base_url: "https://api.openai.com/v1"
    model_name: "text-embedding-3-small"

storage:
  mode: "minio"  # minio / oss
  minio:
    endpoint: "localhost:9000"
    access_key: "minioadmin"
    secret_key: "minioadmin"
```

## 许可证

[MIT License](LICENSE)
