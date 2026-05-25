# AI-Interview 项目分析与学习指南

## 一、项目整体分析

### 1.1 项目是什么？

这是一个 **AI 模拟面试平台**，核心流程为：

```
用户上传简历(PDF) → AI解析简历 → AI生成面试题 → 多轮问答模拟面试 → AI评分 → 生成报告
```

### 1.2 项目由哪几部分组成？

| 子项目 | 作用 | 技术 |
|--------|------|------|
| `ai-interview-backend/` | 后端 API 服务 | Python + FastAPI |
| `ai-interview-frontend/` | 用户端网页 | Vue 3 + Vite |
| `ai-interview-admin/` | 管理后台网页 | Vue 3 + Vite |

### 1.3 后端架构（最核心）

```
app/
├── api/              → 接口入口（接收请求，返回响应）
│   ├── client/v1/    → 用户端接口（注册登录、上传简历、面试）
│   └── backoffice/v1/→ 管理端接口（管理用户、查看面试记录）
├── services/         → 业务逻辑（真正干活的地方）
│   ├── client/       → 用户端逻辑
│   │   ├── ai_service.py        → ⭐ AI核心：调用DeepSeek大模型
│   │   ├── interview_service.py → ⭐ 面试逻辑：开始面试、提交答案、生成报告
│   │   ├── resume_service.py    → 简历上传与解析
│   │   └── auth.py              → 用户注册登录
│   └── common/       → 通用服务（邮件、Redis）
├── models/           → 数据库表定义（ORM模型）
│   ├── user.py       → 用户表
│   ├── interview.py  → 面试记录表
│   ├── resume.py     → 简历表
│   └── ...
├── schemas/          → 请求/响应的数据格式定义
├── core/             → 核心配置
│   ├── config.py     → 环境变量配置
│   └── security.py   → JWT认证（Token生成/验证）
├── route/            → 路由注册中心
├── db/               → 数据库连接
└── exceptions/       → 异常处理
```

### 1.4 核心业务流程

#### 面试流程（最重要的功能链）：

```
1. 用户上传PDF简历
   → resume_service.py 提取文本
   → ai_service.py 调用AI解析简历为结构化JSON

2. 用户发起面试
   → interview_service.py 创建面试会话
   → ai_service.py 根据简历+岗位生成面试题

3. 逐题问答
   → 用户回答一题
   → ai_service.py 评估答案（流式SSE输出评语）
   → 进入下一题

4. 面试结束
   → ai_service.py 生成综合评估报告
   → 保存到数据库
```

### 1.5 前端页面结构

**用户端（frontend）页面：**
| 页面 | 功能 |
|------|------|
| Login.vue | 登录 |
| Register.vue | 注册 |
| Dashboard.vue | 用户主页（查看面试记录） |
| ResumeUpload.vue | 上传简历 |
| Interview.vue | 面试对话页（核心页面） |
| Report.vue | 面试报告页 |
| Profile.vue | 个人资料 |

**管理端（admin）页面：**
| 页面 | 功能 |
|------|------|
| Login.vue | 管理员登录 |
| Dashboard.vue | 数据统计 |
| Users.vue | 用户管理 |
| Interviews.vue | 面试记录列表 |
| InterviewDetail.vue | 面试详情 |

---

## 二、技术栈说明（给基础较差的同学）

### 2.1 后端技术

| 技术 | 是什么 | 在项目中的作用 |
|------|--------|--------------|
| **Python** | 编程语言 | 后端全部用Python写 |
| **FastAPI** | Web框架 | 处理HTTP请求，类似Flask但更快 |
| **SQLAlchemy** | ORM库 | 用Python代码操作数据库，不用写SQL |
| **PostgreSQL** | 关系数据库 | 存储用户、面试、简历等数据 |
| **Redis** | 缓存数据库 | 存验证码、缓存、消息队列 |
| **Celery** | 任务队列 | 异步执行耗时任务（如发邮件） |
| **Docker** | 容器化 | 一键部署所有服务 |
| **JWT** | 认证方式 | 用户登录后发Token，后续请求带Token验证身份 |
| **DeepSeek API** | AI大模型 | 解析简历、出题、评分（核心） |

### 2.2 前端技术

| 技术 | 是什么 | 在项目中的作用 |
|------|--------|--------------|
| **Vue 3** | 前端框架 | 构建页面和交互 |
| **Vite** | 构建工具 | 快速开发和打包 |
| **Pinia** | 状态管理 | 管理全局数据（如登录状态） |
| **Axios** | HTTP库 | 前端调用后端API |
| **Vue Router** | 路由 | 页面切换 |

---

## 三、学习路径建议

### 阶段一：先能看懂代码（1-2周）

**目标：** 读懂项目代码，理解每个文件在做什么

**学习内容：**

1. **Python 基础**（如果不熟）
   - 变量、函数、类
   - async/await 异步语法（本项目大量使用）
   - 推荐：[Python官方教程](https://docs.python.org/zh-cn/3/tutorial/)

2. **FastAPI 入门**
   - 什么是API、路由、请求、响应
   - 跟着官方教程写一个简单接口
   - 推荐：[FastAPI官方文档（有中文）](https://fastapi.tiangolo.com/zh/)

3. **Vue 3 基础**
   - 组件、模板语法、事件处理
   - 推荐：[Vue官方教程](https://cn.vuejs.org/tutorial/)

**实操建议：**
- 先看 `app/api/client/v1/config.py`（最简单的接口，只有健康检查）
- 再看 `app/api/client/v1/auth.py`（注册登录接口，逻辑清晰）
- 然后看 `app/services/client/auth.py`（对应的业务逻辑）

### 阶段二：理解核心功能（1-2周）

**目标：** 完整理解面试流程

**学习顺序：**

1. **简历上传** → `resume.py`（API） → `resume_service.py`（逻辑）
2. **AI 调用** → `ai_service.py`（重点看 `parse_resume`、`generate_questions`）
3. **面试对话** → `interview.py`（API） → `interview_service.py`（逻辑）
4. **SSE流式输出** → 搜索 `EventSourceResponse`，理解实时推送

**需要额外学习：**
- 什么是REST API
- 什么是JWT认证
- 什么是SSE（Server-Sent Events）

### 阶段三：能动手改代码（2-4周）

**目标：** 能修改现有功能、添加新功能

**练习任务（由易到难）：**

1. ✅ 修改一个API的返回格式
2. ✅ 给面试报告增加一个新字段
3. ✅ 修改AI的Prompt，优化出题质量
4. ✅ 新增一个"面试历史"接口
5. ✅ 前端增加一个新页面

**需要额外学习：**
- SQLAlchemy ORM（数据库操作）
- Alembic（数据库迁移）
- Docker + Docker Compose

### 阶段四：独立开发（持续）

**目标：** 能独立设计和实现新模块

**进阶学习：**
- 系统设计（数据库表设计、API设计）
- 性能优化（连接池、缓存、异步）
- 部署运维（Docker、Nginx、CI/CD）

---

## 四、建议的学习方式

### 4.1 阅读代码的顺序

```
1. main.py                    → 程序入口
2. app/route/route.py         → 了解应用如何启动
3. app/route/router_registry.py → 了解所有路由（接口列表）
4. app/core/config.py         → 了解配置项
5. app/models/*.py            → 了解数据库有哪些表
6. app/api/client/v1/auth.py  → 看一个完整接口
7. app/services/client/auth.py → 看对应的业务逻辑
8. app/services/client/ai_service.py → 看AI核心代码（最有意思的部分）
```

### 4.2 实际操作建议

1. **先把项目跑起来**（参考 `部署文档.md`）
2. **用 Postman/curl 调用接口**，感受数据流动
3. **加 print/log 语句**，观察代码执行顺序
4. **尝试小修改**，看效果（比如改AI的Prompt）
5. **画流程图**，帮助理解复杂逻辑

### 4.3 免费学习资源

| 主题 | 推荐资源 |
|------|----------|
| Python基础 | [廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400) |
| FastAPI | [FastAPI官方文档（中文）](https://fastapi.tiangolo.com/zh/) |
| Vue 3 | [Vue.js官方教程](https://cn.vuejs.org/tutorial/) |
| Git | [Pro Git中文版](https://git-scm.com/book/zh/v2) |
| Docker | [Docker从入门到实践](https://yeasy.gitbook.io/docker_practice/) |
| 数据库 | [PostgreSQL中文文档](http://www.postgres.cn/docs/16/) |

---

## 五、项目亮点总结（面试/写简历时可说）

1. **全栈项目**：前后端分离 + 管理后台，完整闭环
2. **AI 应用**：大模型API调用、Prompt工程、结构化输出
3. **企业级架构**：三层架构、统一响应、分层异常、路由注册中心
4. **异步全链路**：FastAPI + SQLAlchemy async + asyncpg，高并发处理
5. **SSE 流式输出**：AI评语实时推送，提升用户体验
6. **Docker 容器化**：一条命令启动全部服务
7. **Celery 异步任务**：邮件等耗时操作不阻塞
8. **JWT 无状态认证**：安全且高效
