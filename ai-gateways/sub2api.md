# sub2api

> Repo: https://github.com/Wei-Shaw/sub2api/tree/main
> Category: `ai-gateways`
> Last checked: 2026-05-18
> Commit checked: `f5bd25bea045e728846b38bf18080ffa48d133c6`

## 1. 一句话总结
Sub2API 是一个面向 AI 订阅额度分发的开源 API 网关/SaaS 平台。它把 Claude、OpenAI/Codex、Gemini、Antigravity 等上游账号或 API Key 组成账号池，对外发放统一的 API Key，并负责鉴权、调度、计费、限流、用量记录和 Web 管理后台。

## 2. 项目目的与核心问题
- 目标用户：需要自建 AI 中转/网关的团队、共享订阅额度的小团队、AI API 服务运营者、需要统一管理多平台账号池的开发者。
- 解决的问题：多个 AI 订阅账号难以统一分配、计费、限流、监控和授权；Claude Code、Codex CLI、Gemini CLI 等客户端需要尽量保持原生接入体验。
- 为什么值得关注：它不是一个简单反代脚本，而是完整平台：后端、前端、数据库 schema、Redis 调度缓存、支付系统、账号监控、模型映射、部署脚本和 CI 都在仓库中。
- 重要风险：README 明确提示使用本项目可能违反部分上游服务条款，账号封禁、服务中断等风险由使用者自行承担；生产使用前必须评估合规与授权边界。

## 3. 核心能力
- 多平台账号管理：支持 Anthropic、OpenAI、Gemini、Antigravity 等平台，账号类型覆盖 OAuth、setup-token、API Key、上游透传、Bedrock、Google service account。
- 用户 API Key 分发：用户通过 Sub2API 生成 `sk-` 风格 Key，管理员可按用户、分组、额度、IP、过期时间管理。
- 兼容协议入口：提供 `/v1/messages`、`/v1/responses`、`/v1/chat/completions`、`/v1beta`、`/backend-api/codex`、`/antigravity/v1` 等路由。
- 智能调度：按分组平台、模型映射、账号状态、优先级、负载、并发、RPM、窗口成本、粘性会话选择上游账号。
- 精确计费：记录输入/输出/cache/image token，按模型价格、分组倍率、账号倍率、订阅或余额模式计费。
- 管理后台：Vue 前端提供用户、API Key、账号、分组、渠道、支付、用量、风控、运维监控、系统设置等页面。
- 内置支付：支持 EasyPay、支付宝、微信、Stripe、Airwallex 等支付/订单相关能力。
- 部署友好：支持一键二进制安装、Docker Compose auto setup、源码编译并把前端嵌入 Go 二进制。

## 4. 实现思路 / 架构拆解
- 关键模块：
  - `backend/cmd/server/`：Go 服务入口，处理 setup、auto setup、版本信息和主服务启动。
  - `backend/internal/server/routes/`：HTTP 路由注册，按 auth/user/admin/payment/gateway 拆分。
  - `backend/internal/handler/`：HTTP handler 层，解析请求、处理响应、记录错误和桥接各类 API。
  - `backend/internal/service/`：核心业务逻辑，包括网关调度、OpenAI/Codex、Gemini、Antigravity、计费、账号、并发、订阅、支付等。
  - `backend/ent/schema/`：Ent ORM 数据模型，核心实体包括 `Account`、`Group`、`APIKey`、`User`、`UsageLog`、`PaymentOrder` 等。
  - `backend/internal/pkg/`：平台协议与兼容工具，如 `apicompat`、`openai`、`geminicli`、`antigravity`、TLS 指纹、代理、日志等。
  - `frontend/src/`：Vue 3 管理台和用户台，使用 Vite、Pinia、Vue Router、TailwindCSS、axios。
  - `deploy/`：Docker Compose、systemd、安装脚本、配置样例和数据管理进程说明。
- 核心流程：
  1. 用户或客户端带 Sub2API API Key 请求兼容端点。
  2. 中间件做请求体限制、request ID、API Key 鉴权、订阅/分组检查。
  3. Handler 预解析请求体，提取 model、stream、metadata.user_id、messages 等。
  4. GatewayService 根据分组平台、强制平台、模型路由和混合调度规则确定候选账号。
  5. 调度器检查账号是否可调度：状态、模型支持、配额、窗口成本、RPM、并发槽位、会话数量。
  6. 对候选账号按模型路由、粘性会话、优先级、负载率、最后使用时间选择。
  7. 转发到上游，必要时做协议转换、模型映射、header 过滤、流式处理。
  8. 记录 usage log，计算费用，扣余额/订阅额度，并更新账号状态与统计。
- 依赖/生态：
  - 后端：Go、Gin、Ent、Wire、PostgreSQL、Redis、Viper、JWT、Stripe/Alipay/WxPay/Airwallex SDK、testcontainers。
  - 前端：Vue 3、Vite、TypeScript、Pinia、Vue Router、TailwindCSS、Vitest。
  - 部署：Docker Compose、systemd、Caddy/Nginx 反向代理、可选 datamanagementd。
- 设计亮点：
  - 把“订阅账号池”抽象为可调度资源，而不是只做静态转发。
  - Gateway 请求只解析一次，`ParsedRequest` 在 handler/service 中复用，减少大请求体重复解析。
  - 粘性会话优先使用 `metadata.user_id` 中的 session 信息，其次 cache_control 内容 hash，最后回退到消息摘要。
  - 调度不是单一轮询：会结合模型路由、粘性会话、负载、并发槽位和等待队列。
  - 支持 OpenAI/Codex 与 Anthropic Messages 的互通映射，例如把 Claude 系列请求映射到 GPT/Codex 模型。

## 5. 安装与快速开始
### Docker Compose 一键部署
```bash
mkdir -p sub2api-deploy
cd sub2api-deploy

curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/docker-deploy.sh | bash
docker compose -f docker-compose.local.yml up -d
docker compose -f docker-compose.local.yml logs -f sub2api
```

访问：

```text
http://你的服务器IP:8080
```

如果没有显式设置 `ADMIN_PASSWORD`，首次管理员密码在日志中查看：

```bash
docker compose -f docker-compose.local.yml logs sub2api | grep "admin password"
```

### 二进制安装
```bash
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/install.sh | sudo bash
sudo systemctl start sub2api
sudo systemctl enable sub2api
```

然后浏览器打开 `http://服务器IP:8080` 完成 setup wizard。

### 源码编译
```bash
git clone https://github.com/Wei-Shaw/sub2api.git
cd sub2api

cd frontend
pnpm install
pnpm run build

cd ../backend
go build -tags embed -o sub2api ./cmd/server
cp ../deploy/config.example.yaml ./config.yaml
./sub2api
```

### 关键配置
生产部署至少需要关注：

```bash
POSTGRES_PASSWORD=...
JWT_SECRET=...
TOTP_ENCRYPTION_KEY=...
ADMIN_EMAIL=...
ADMIN_PASSWORD=...
SERVER_PORT=8080
TZ=Asia/Shanghai
```

反向代理给 Codex CLI 等客户端使用时，Nginx 需要开启：

```nginx
underscores_in_headers on;
```

否则 `session_id` 这类带下划线的请求头可能被丢弃，影响粘性会话。

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：
  - 作为客户端调用 Sub2API 提供的 Anthropic-compatible `/v1/messages`。
  - 研究/修改 Sub2API 仓库时，让 Claude Code 负责前后端功能开发、配置项扩展、部署脚本调整。
- 作为客户端的典型配置：
```bash
export ANTHROPIC_BASE_URL="http://localhost:8080"
export ANTHROPIC_AUTH_TOKEN="sk-你的Sub2API-Key"
```

Antigravity 专用端点可以按 README 示例：

```bash
export ANTHROPIC_BASE_URL="http://localhost:8080/antigravity"
export ANTHROPIC_AUTH_TOKEN="sk-你的Sub2API-Key"
```

- 推荐工作流：
  1. 管理后台添加上游 Anthropic/OpenAI/Gemini/Antigravity 账号。
  2. 创建分组，配置平台、倍率、订阅模式、模型路由和是否限制 Claude Code 客户端。
  3. 给用户创建 API Key 并绑定分组。
  4. Claude Code 使用 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_AUTH_TOKEN` 指向 Sub2API。
  5. 通过管理后台看 usage log、账号状态、RPM/并发和错误日志。
- 注意事项：
  - Claude Code 的计划模式在 Antigravity 路径下 README 提到有已知问题，可能需要手动 `Shift + Tab` 退出 Plan Mode。
  - 如果上游模型更新快于 Sub2API 默认映射，需要在账号或分组中补模型映射。
  - 生产前要确认上游账号授权方式、代理/IP、TLS 指纹和服务条款风险。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：
  - 作为 OpenAI/Codex-compatible 客户端，通过 Sub2API 的 `/v1/responses`、`/v1/chat/completions` 或 `/backend-api/codex/responses` 访问上游。
  - 作为代码代理研究和维护这个仓库，尤其适合检查路由、调度、计费、前端 API 对齐和测试覆盖。
- 作为 OpenAI-compatible 客户端的常见配置思路：
```bash
export OPENAI_API_KEY="sk-你的Sub2API-Key"
export OPENAI_BASE_URL="http://localhost:8080/v1"
```

如果使用会向 `/backend-api/codex/responses` 发请求的 Codex 客户端或兼容实现，Sub2API 已注册 `/backend-api/codex` 路由别名。反向代理必须保留 `session_id`、`conversation_id`、`x-codex-turn-state` 等相关 header。

- 推荐工作流：
  - 用 Codex 阅读 `backend/internal/server/routes/gateway.go` 先理解入口。
  - 再读 `backend/internal/service/gateway_service.go` 和 `openai_gateway_service.go` 理解账号选择与 Codex Responses 转发。
  - 改动协议兼容时补 `backend/internal/pkg/apicompat`、`backend/internal/service/*_test.go` 和前端类型/API 测试。
- 注意事项：
  - Codex/OpenAI 路径里有 OAuth/API Key 两套上游模式，行为不完全一样。
  - `gateway.force_codex_cli` 会把所有 `/openai/v1/responses` 按 Codex CLI 处理，配置里明确提示要谨慎。
  - 如果启用图片桥接、WS mode、OpenAI passthrough header 等配置，要同步验证客户端行为和计费记录。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 小团队共享订阅额度 | 管理员录入多个 OAuth/API Key 账号，按分组给用户发 Key | 统一入口、用量统计、余额/订阅控制 |
| Claude Code/Codex 统一网关 | 设置兼容 base URL 和 Sub2API Key | 原生客户端尽量无感接入 |
| 多账号负载均衡 | 配置账号优先级、并发、RPM、模型路由、粘性会话 | 降低单账号打满或频繁切换上下文的风险 |
| API 服务商业化 | 开启支付、充值、订阅计划、优惠码、风控 | 自助充值和用量计费平台 |
| 上游模型迁移 | 在分组/账号层配置模型映射 | 客户端模型名不变，上游模型可替换 |
| 运维排障 | 查看 dashboard、ops、usage、request errors、account status | 定位账号限流、过载、计费异常或路由错误 |

## 9. 适合 / 不适合
### 适合
- 需要把多个 AI 账号/订阅统一成一个内部 API 网关。
- 需要用户、余额、订阅、支付、用量记录和后台管理能力。
- 需要兼容 Claude Code、Codex CLI、Gemini CLI 等 coding agent 客户端。
- 对自部署、数据库、Redis、反向代理、安全配置和上游风控有维护能力。

### 不适合
- 只是个人临时转发一个 API Key，不需要账号池、计费和后台。
- 无法承担上游账号服务条款、封号或合规风险的场景。
- 不愿维护 PostgreSQL、Redis、备份、升级、密钥轮换和日志审计。
- 需要官方 SLA 或合规认证的企业级 API 网关；这个项目是开源自部署工具，不是官方托管服务。

## 10. 同类工具定位
- 类似工具：LiteLLM Proxy、one-api/new-api、OpenAI-compatible reverse proxy、Kong/Envoy + 自定义插件、各类 Claude/OpenAI/Gemini relay 平台。
- 相比 LiteLLM：Sub2API 更偏“订阅账号池分发 + SaaS 后台 + coding agent 兼容”，不是通用 LLM router/library。
- 相比 one-api/new-api：Sub2API 对 Claude Code、Codex、Gemini、Antigravity、粘性会话、OAuth 账号池和模型映射做了更强的专门化。
- 相比通用网关：Sub2API 内置用户、计费、支付、用量、账号调度和协议兼容，不需要从 API Gateway 原语开始拼。
- 相比限制：项目复杂度高，协议适配跟上游变化强绑定；上游客户端或模型协议变更时需要持续维护。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README_CN.md` | 中文总览、在线 demo、核心功能、部署方式、Claude Code/Antigravity 示例和免责声明。 |
| `DEV_GUIDE.md` | 开发环境、CI、本地测试命令和常见坑。 |
| `deploy/README.md` | Docker/二进制部署说明、auto setup、迁移和环境变量。 |
| `deploy/config.example.yaml` | 完整配置样例，覆盖 server、security、gateway、billing、scheduler、Gemini OAuth 等。 |
| `deploy/docker-compose.local.yml` | 推荐 Docker 部署，使用本地目录保存应用、Postgres、Redis 数据。 |
| `backend/cmd/server/main.go` | 服务入口，处理 setup wizard、Docker auto setup 和主服务启动。 |
| `backend/internal/server/routes/gateway.go` | Claude/OpenAI/Gemini/Codex/Antigravity 兼容网关路由。 |
| `backend/internal/server/routes/admin.go` | 管理后台 API 路由，覆盖用户、分组、账号、支付、监控、风控等。 |
| `backend/internal/service/gateway_service.go` | Anthropic/Gemini/Antigravity 侧核心网关、粘性会话、账号调度和用量记录逻辑。 |
| `backend/internal/service/openai_gateway_service.go` | OpenAI/Codex Responses、Chat Completions、WebSocket、header 兼容和 Codex 用量处理。 |
| `backend/internal/service/billing_service.go` | 模型价格、token 成本和实际扣费计算。 |
| `backend/internal/service/concurrency_service.go` | Redis-backed 用户/账号并发槽位和等待队列。 |
| `backend/internal/domain/constants.go` | 平台、账号类型、订阅类型、默认模型映射常量。 |
| `backend/ent/schema/account.go` | 上游账号实体，包含平台、凭证、调度状态、并发、优先级、限流、过载等字段。 |
| `backend/ent/schema/group.go` | 分组实体，包含平台、倍率、订阅限制、模型路由、Claude Code 限制、OpenAI Messages dispatch 等。 |
| `backend/ent/schema/api_key.go` | 用户 API Key、额度、窗口限流、IP 黑白名单和分组绑定。 |
| `backend/ent/schema/usage_log.go` | 请求用量、模型、token、费用、耗时、图片生成、计费模式等日志字段。 |
| `frontend/src/router/index.ts` | 前端页面路由，区分 setup、public、user、admin 页面。 |
| `frontend/src/api/client.ts` | axios client、JWT 注入、语言/时区参数、token refresh 和错误处理。 |
| `.github/workflows/backend-ci.yml` | 后端单测、集成测试、前端 typecheck/Vitest、golangci-lint。 |
| `.github/workflows/security-scan.yml` | govulncheck 和 pnpm audit 安全扫描。 |

## 12. 学习与掌控建议
1. 先用 Docker Compose 本地跑起来，完成 setup、创建管理员、添加一个测试账号、创建分组和 API Key。
2. 先只接一个平台，例如 Anthropic-compatible `/v1/messages`，跑通后再研究 OpenAI/Codex、Gemini、Antigravity。
3. 读代码按这个顺序：`routes/gateway.go` -> `handler/gateway_handler.go` / `openai_gateway_handler.go` -> `service/gateway_service.go` -> `ent/schema/*`。
4. 调度相关重点看粘性会话、模型路由、并发槽位、RPM、窗口成本和 `mixed_scheduling`，这些决定线上稳定性。
5. 上生产前必须做备份、密钥固定、Nginx/Caddy header 验证、数据库连接池调优、Redis 持久化、日志脱敏和合规评估。
6. 如果用于 Claude Code/Codex，务必验证 plan mode、streaming、Responses API、WebSocket、图片生成、长上下文和 retry 行为。
7. 修改 Ent schema 后要运行 `go generate ./ent` 和 `go generate ./cmd/server`，并提交生成代码。
8. PR 前按项目要求运行后端 unit/integration、前端 lint/typecheck/critical Vitest，以及必要的安全扫描。

## 13. Sources
- https://github.com/Wei-Shaw/sub2api/tree/main
- https://github.com/Wei-Shaw/sub2api/blob/main/README_CN.md
- https://github.com/Wei-Shaw/sub2api/blob/main/DEV_GUIDE.md
- https://github.com/Wei-Shaw/sub2api/blob/main/deploy/README.md
- https://github.com/Wei-Shaw/sub2api/blob/main/deploy/config.example.yaml
- https://github.com/Wei-Shaw/sub2api/blob/main/deploy/docker-compose.local.yml
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/cmd/server/main.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/internal/server/routes/gateway.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/internal/service/gateway_service.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/internal/service/openai_gateway_service.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/internal/service/billing_service.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/internal/service/concurrency_service.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/ent/schema/account.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/ent/schema/group.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/ent/schema/api_key.go
- https://github.com/Wei-Shaw/sub2api/blob/main/backend/ent/schema/usage_log.go
- https://github.com/Wei-Shaw/sub2api/blob/main/frontend/src/router/index.ts
- https://github.com/Wei-Shaw/sub2api/blob/main/frontend/src/api/client.ts
