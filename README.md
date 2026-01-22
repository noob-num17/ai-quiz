# ai-quiz

LLMAgent 学习评估与巩固系统 - 基于 AI 的个性化学习智能体

## 快速开始

### 使用指南

#### 第一步：拉取 Docker 镜像

```bash
docker pull ghcr.io/noob-num17/ai-quiz:sha-aa41652
```

> **注意**：镜像 tag 可能会更新。请根据最新的 GitHub Actions 构建结果替换 `sha-aa41652`。
> 
> 查看最新 tag：
> ```bash
> # 方式1：查看 GitHub Packages
> # https://github.com/noob-num17/ai-quiz/pkgs/container/ai-quiz
>
> # 方式2：通过命令行
> docker search ghcr.io/noob-num17/ai-quiz
> ```

#### 第二步：创建 docker-compose.yml 文件

在本地项目目录下创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  # MongoDB 数据库服务
  mongodb:
    image: mongo:7.0
    container_name: learning-agent-mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password123
      MONGO_INITDB_DATABASE: learning_agent
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
      - mongodb_config:/data/configdb
    networks:
      - learning-network
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Python 应用服务
  app:
    image: ghcr.io/noob-num17/ai-quiz:sha-aa41652  # ⚠️ 更新为最新的 tag
    container_name: learning-agent-app
    environment:
      # MongoDB 连接配置
      MONGO_URI: mongodb://root:password123@mongodb:27017/learning_agent?authSource=admin
      # OpenAI 配置（需要在运行时设置）
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      OPENAI_BASE_URL: ${OPENAI_BASE_URL:-https://api.openai.com/v1/}
      # 流式输出配置
      ENABLE_STREAM: "True"
      DEBUG: "False"
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - learning-network
    restart: unless-stopped
    stdin_open: true
    tty: true

volumes:
  mongodb_data:
  mongodb_config:

networks:
  learning-network:
    driver: bridge
```

#### 第三步：运行应用

在终端中运行以下命令：

```bash
# 进入项目目录
cd /path/to/ai-quiz

# 设置环境变量（OpenAI API Key）
export OPENAI_API_KEY="your-api-key-here"
export OPENAI_BASE_URL="https://api.openai.com/v1/"  # 可选

# 启动容器组
docker compose up -d

# 进入容器
docker attach learning-agent-app

# 停止容器
docker compose down
```

### 环境变量配置

| 变量 | 说明 | 示例 |
|------|------|------|
| `OPENAI_API_KEY` | OpenAI API 密钥（必需） | `sk-...` |
| `OPENAI_BASE_URL` | OpenAI API 端点（可选） | `https://api.openai.com/v1/` |
| `OPENAI_MODEL` | 使用的Chat模型（可选） | `deepseek-ai/DeepSeek-V3.2` |
| `EMBEDDING_MODEL` | 使用的Embedding模型（可选） | `Qwen/Qwen3-Embedding-8B`|
| `MONGO_URI` | MongoDB 连接字符串 | `mongodb://root:password123@mongodb:27017/learning_agent` |


## 文档

- [系统架构设计](./doc/ARCHITECTURE.md) - 详细的架构文档和云原生组件说明
- [LLM Agent 智能体策略](./doc/AGENT_STRATEGY.md) - Prompt 模板、设计过程和工具链
- [测试说明](./test/README.md) - 测试套件和单元测试指南

---

## 项目结构

```
├── main.py                    # 应用入口
├── requirements.txt           # Python 依赖
├── Dockerfile                 # Docker 构建文件
├── docker-compose.yml         # 本地开发编排
├── models/                    # 核心模块
│   ├── agent.py              # LLMAgent 主控引擎
│   ├── question_generator.py # 题目生成器
│   ├── answer_evaluator.py   # 答案评估器
│   ├── weakness_analyzer.py  # 弱点分析器
│   ├── data_processor.py     # 数据处理器
│   ├── mongodb_client.py     # 数据库客户端
│   └── cli.py                # 命令行界面
├── test/                      # 测试套件
│   ├── test_agent.py
│   ├── test_question_generator.py
│   ├── test_answer_evaluator.py
│   ├── test_data_processor.py
│   └── test_cli_integration.py
└── doc/                       # 文档
    ├── ARCHITECTURE.md        # 架构设计
    └── AGENT_STRATEGY.md      # Agent 策略
```

---

## CI/CD 工作流

- **pytest** - 自动化测试（`.github/workflows/pytest.yml`）
- **Docker Build & Push** - 镜像构建和推送（`.github/workflows/docker-publish.yml`）

---

## 技术栈

- **LLM**: OpenAI API / DeepSeek
- **数据库**: MongoDB 7.0
- **容器化**: Docker & Docker Compose
- **Python**: 3.10+
- **测试**: pytest

---
