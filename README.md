# Dream RAG — 企业级异步流式知识引擎

基于 RAG（检索增强生成）的企业级 AI 知识库系统，支持多租户文档智能检索与对话问答，提供从文档上传、切片向量化到语义检索、流式对话的完整链路。

## 技术栈

| 层级 | 技术 |
|------|------|
| 后端框架 | Java 17 / Spring Boot 3.4 / WebFlux |
| 前端 | Vue 3 / TypeScript / Vite / Naive UI / Pinia |
| 搜索引擎 | Elasticsearch 8.10 |
| 消息队列 | Apache Kafka（异步文档处理） |
| 实时通信 | WebSocket（全双工流式对话） |
| 数据库 | MySQL 8.0 / Redis 7 |
| 对象存储 | MinIO |
| AI 集成 | DeepSeek API / DashScope Embedding |
| 安全 | Spring Security + JWT |
| 构建 | Maven / pnpm |

## 系统架构

```
Vue 3 Frontend (Naive UI + Pinia)         Port 9527
        | HTTP/WebSocket
Spring Boot Backend                        Port 8081
   ├── Document Upload → Kafka → Async Processing
   ├── Elasticsearch (向量 + 关键词混合检索)
   ├── DeepSeek LLM (流式 SSE → WebSocket)
   └── Redis (对话上下文窗口)
        |
Infrastructure: MySQL / Redis / ES / Kafka / MinIO
```

## 本地启动

### 前置条件

- Java 17+
- Node.js 18+ / pnpm
- Docker Desktop

### 1. 启动基础设施

```bash
cd docs
docker compose up -d   # 启动 MySQL / Redis / ES / Kafka / MinIO
```

### 2. 配置环境变量

```bash
cp .env.example .env
# 编辑 .env，填写 DEEPSEEK_API_KEY、EMBEDDING_API_KEY 等
```

### 3. 启动后端

```bash
mvn spring-boot:run
```

### 4. 启动前端

```bash
cd frontend
pnpm install
pnpm dev
```

访问 http://localhost:9527

## 环境变量

| 变量 | 说明 |
|------|------|
| `SPRING_DATASOURCE_URL` / `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD` | MySQL 连接 |
| `SPRING_DATA_REDIS_HOST` / `SPRING_DATA_REDIS_PORT` | Redis 连接 |
| `SPRING_KAFKA_BOOTSTRAP_SERVERS` | Kafka 地址 |
| `ELASTICSEARCH_HOST` / `ELASTICSEARCH_PORT` / `ELASTICSEARCH_PASSWORD` | Elasticsearch 连接 |
| `MINIO_ENDPOINT` / `MINIO_ACCESS_KEY` / `MINIO_SECRET_KEY` | MinIO 凭证 |
| `DEEPSEEK_API_KEY` | DeepSeek LLM API Key |
| `EMBEDDING_API_KEY` | 向量化服务 API Key |
| `JWT_SECRET_KEY` | JWT 签名密钥 (Base64) |

详见 `.env.example` 和 `src/main/resources/application.yml`。

## 核心功能

- **文档智能处理**：上传 → 解析 → 文本切片 → 向量化入库全流程自动化
- **异步消息驱动**：Kafka 解耦文件处理链路，支持重试与死信队列
- **混合语义检索**：Elasticsearch 关键词 + 向量相似度混合搜索
- **WebSocket 流式对话**：全双工通信，后端接入 LLM SSE 流实现逐字输出
- **断线续传与状态管理**：Redis 维护对话上下文窗口，断线重连自动恢复
- **多租户架构**：组织标签隔离，公开/私有文档权限控制

## 关键技术实现

### 1. Kafka 异步文档处理

文件上传后由 Kafka 生产者投递到 `file-processing-topic`，消费者 (`FileProcessingConsumer`) 顺序完成文件解析、文本切片与向量化入库。消费端支持 4 次重试，失败消息路由至死信队列 (`file-processing-dlt`)。

### 2. WebSocket 流式对话

基于 Spring WebSocket 实现全双工通信，后端通过 WebFlux 的 Reactive Stream 消费 LLM SSE 流，并以 WebSocket 逐帧推送到前端。支持用户主动中断生成，后端同步取消 LLM 连接。

### 3. Redis 上下文窗口

每段对话在 Redis 中维护 20 条消息的上下文窗口，供 LLM 调用时毫秒级加载。完整对话历史持久化到 MySQL。每个流式 token 原子写入 Redis，通过状态机维持生成进度，保障弱网环境下的断线重连。

### 4. Elasticsearch 混合检索

结合 BM25 关键词检索与向量语义检索，支持文档级和段落级精确匹配。

---

## 面试官源码阅读导航

> 如果您正在评估候选人此项目的技术深度，以下路径可按优先级阅读。

### 第一优先：文档处理与异步链路

| 文件 | 关注点 |
|------|--------|
| `src/main/java/com/cty/dreamrag/service/DocumentService.java` | 文档上传、解析、管理 |
| `src/main/java/com/cty/dreamrag/consumer/FileProcessingConsumer.java` | Kafka 消费端："解析→切片→向量化"全流程 |
| `src/main/java/com/cty/dreamrag/service/ElasticsearchService.java` | ES 索引与混合检索 |
| `src/main/java/com/cty/dreamrag/config/KafkaConfig.java` | Kafka 生产者/消费者配置 |

### 第二优先：WebSocket 流式对话与状态管理

| 文件 | 关注点 |
|------|--------|
| `src/main/java/com/cty/dreamrag/handler/ChatWebSocketHandler.java` | WebSocket 消息处理器，流式推送核心 |
| `src/main/java/com/cty/dreamrag/client/DeepSeekClient.java` | DeepSeek LLM SSE 流式调用 |
| `src/main/java/com/cty/dreamrag/service/ChatGenerationStateService.java` | 生成状态机与断线续传 |
| `src/main/java/com/cty/dreamrag/service/ChatSessionRegistry.java` | 会话注册与上下文管理 |

### 第三优先：安全与架构

| 文件 | 关注点 |
|------|--------|
| `src/main/java/com/cty/dreamrag/config/SecurityConfig.java` | Spring Security + JWT 安全配置 |
| `src/main/java/com/cty/dreamrag/config/JwtAuthenticationFilter.java` | JWT 认证过滤器 |
| `src/main/java/com/cty/dreamrag/config/OrgTagAuthorizationFilter.java` | 多租户组织权限控制 |
| `src/main/java/com/cty/dreamrag/service/ConversationService.java` | 对话持久化与历史管理 |

### 第四优先：前端核心

| 文件 | 关注点 |
|------|--------|
| `frontend/src/views/` | Vue 页面组件（聊天、知识库、文档管理） |
| `frontend/src/service/` | API 调用层 |
| `frontend/src/store/` | Pinia 状态管理 |

### 代码规模

Java 139 · TypeScript 150 · Vue 94 · 总计约 480 个源文件

---

## License

MIT
