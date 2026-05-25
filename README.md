# AI-Interview（智面）— AI 模拟面试平台

基于大模型的 AI 面试模拟平台。用户上传 PDF 简历，AI 自动解析简历内容并生成针对性面试题目，通过多轮对话模拟真实面试场景，实时流式输出评语，最终生成综合评估报告。

## 项目结构

```
ai-interview/
├── ai-interview-backend/       # 后端（FastAPI + Docker 部署）
├── ai-interview-frontend/      # 用户端前端（Vue 3 + Vite）
├── ai-interview-admin/         # 后台管理前端（Vue 3 + Vite）
├── 部署文档.md                  # 详细部署文档
└── README.md                   # 本文件
```

## 功能特性

- 📄 简历上传与 AI 智能解析（PDF → 结构化数据）
- 🎤 AI 模拟面试（多轮对话，逐题问答）
- 📊 实时评分与综合评估报告
- 🔄 SSE 流式输出（AI 回答逐字推送）
- 🎯 根据目标岗位（实习/正式）自动调整面试难度
- 👤 用户注册登录（JWT 认证）
- 🛠 后台管理系统（用户管理、面试记录查看、数据统计）

## 技术栈

| 模块 | 技术 |
|------|------|
| 后端框架 | FastAPI（异步） |
| 前端框架 | Vue 3 + Vite |
| AI 模型 | DeepSeek API（兼容 OpenAI SDK） |
| 数据库 | PostgreSQL 16 |
| 缓存/消息队列 | Redis 7 |
| 异步任务 | Celery（Worker + Beat） |
| ORM | SQLAlchemy 2.0（异步 + asyncpg） |
| 容器化 | Docker + Docker Compose |

## 快速开始

### 环境要求

- Docker + Docker Compose V2（后端部署）
- Node.js 18+（前端开发）
- DeepSeek API Key（[申请地址](https://platform.deepseek.com/)）

### 后端启动

```bash
cd ai-interview-backend

# 复制并配置环境变量
cp .env.example .env
# 编辑 .env，填入 DEEPSEEK_API_KEY

# 构建并启动所有服务
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build

# 数据库迁移
docker exec -it ai-interview-app alembic upgrade head

# 创建管理员账号
docker exec -it ai-interview-app python scripts/create_first_admin.py

# 验证服务
curl http://localhost:8006/api/v1/config/health
```

### 前端启动

```bash
# 用户端（端口 3000）
cd ai-interview-frontend
npm install
npm run dev

# 后台管理（端口 3001）— 新开终端
cd ai-interview-admin
npm install
npm run dev
```

### 访问地址

| 服务 | 地址 |
|------|------|
| 用户端 | http://localhost:3000 |
| 后台管理 | http://localhost:3001 |
| 后端 API 文档（开发环境） | http://localhost:8006 |

### 默认账号

| 角色 | 邮箱 | 密码 |
|------|------|------|
| 后台管理员 | admin@ai-interview.com | ai-interview&admin |

> 普通用户通过前端注册页面自行注册

## 后端架构

```
ai-interview-backend/app/
├── api/                # API 路由层（Client / Backoffice）
├── services/           # 业务逻辑层
├── models/             # 数据模型层（SQLAlchemy ORM）
├── schemas/            # 请求/响应数据结构（Pydantic）
├── core/               # 核心配置（安全、日志、Celery）
├── db/                 # 数据库连接与会话管理
├── exceptions/         # 分层异常体系
├── utils/              # 工具函数
└── route/              # 路由注册中心
```

## 详细部署

完整的部署步骤请参考 [部署文档.md](./部署文档.md)。
