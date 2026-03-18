# 开发指南（Developing）

本项目可在本地完整运行，无需手动安装 PostgreSQL。

## 部署模式（Deployment Modes）

模式定义与 CLI 行为见 `doc/DEPLOYMENT-MODES.md`。

当前实现状态：

- 规范模型：`local_trusted` 与 `authenticated`（含 `private`/`public` 暴露）

## 前置条件（Prerequisites）

- Node.js 20+
- pnpm 9+

## 启动开发（Start Dev）

在仓库根目录：

```sh
pnpm install
pnpm dev
```

会启动：

- API 服务：`http://localhost:3100`
- UI：由 API 服务以 dev middleware 模式提供（与 API 同源）

Tailscale/私网认证开发模式：

```sh
pnpm dev --tailscale-auth
```

以 `authenticated/private` 运行并将服务绑定到 `0.0.0.0` 以支持私有网络访问。

允许额外私有主机名（如自定义 Tailscale 主机名）：

```sh
pnpm paperclipai allowed-hostname dotta-macbook-pro
```

## 一键本地运行（One-Command Local Run）

首次本地安装可一条命令完成引导并运行：

```sh
pnpm paperclipai run
```

`paperclipai run` 会：

1. 若缺少配置则自动 onboard
2. 运行 `paperclipai doctor` 并开启修复
3. 检查通过后启动服务

## Docker 快速上手（无需本地 Node）（Docker Quickstart）

在 Docker 中构建并运行 Paperclip：

```sh
docker build -t paperclip-local .
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -v "$(pwd)/data/docker-paperclip:/paperclip" \
  paperclip-local
```

或使用 Compose：

```sh
docker compose -f docker-compose.quickstart.yml up --build
```

API key 配置（`OPENAI_API_KEY` / `ANTHROPIC_API_KEY`）与持久化见 `doc/DOCKER.md`。

## 开发环境数据库（自动处理）（Database in Dev）

本地开发时请勿设置 `DATABASE_URL`。服务会自动使用嵌入式 PostgreSQL，数据持久化在：

- `~/.paperclip/instances/default/db`

覆盖 home 与实例：

```sh
PAPERCLIP_HOME=/custom/path PAPERCLIP_INSTANCE_ID=dev pnpm paperclipai run
```

此模式不需要 Docker 或外部数据库。

## 开发环境存储（自动处理）（Storage in Dev）

本地开发默认使用 `local_disk` 存储，上传的图片/附件保存在：

- `~/.paperclip/instances/default/data/storage`

配置存储提供方/设置：

```sh
pnpm paperclipai configure --section storage
```

## 默认 Agent 工作空间（Default Agent Workspaces）

当本地 agent 运行没有解析到项目/会话工作空间时，Paperclip 会回退到实例根下的 agent 主工作空间：

- `~/.paperclip/instances/default/workspaces/<agent-id>`

该路径会遵循非默认配置下的 `PAPERCLIP_HOME` 与 `PAPERCLIP_INSTANCE_ID`。

## 快速健康检查（Quick Health Checks）

在另一终端：

```sh
curl http://localhost:3100/api/health
curl http://localhost:3100/api/companies
```

预期：

- `/api/health` 返回 `{"status":"ok"}`
- `/api/companies` 返回 JSON 数组

## 重置本地开发数据库（Reset Local Dev Database）

清空本地数据重新开始：

```sh
rm -rf ~/.paperclip/instances/default/db
pnpm dev
```

## 可选：使用外部 Postgres（Optional: Use External Postgres）

若设置 `DATABASE_URL`，服务将使用该连接而非嵌入式 PostgreSQL。

## 开发环境 Secrets（Secrets in Dev）

Agent 环境变量现支持 secret 引用。默认情况下密钥值经本地加密存储，agent 配置中只持久化 secret 引用。

- 默认本地密钥路径：`~/.paperclip/instances/default/secrets/master.key`
- 直接覆盖密钥：`PAPERCLIP_SECRETS_MASTER_KEY`
- 覆盖密钥文件路径：`PAPERCLIP_SECRETS_MASTER_KEY_FILE`

严格模式（建议在非本地可信环境使用）：

```sh
PAPERCLIP_SECRETS_STRICT_MODE=true
```

启用后，敏感环境变量键（如 `*_API_KEY`、`*_TOKEN`、`*_SECRET`）必须使用 secret 引用，不能内联明文。

CLI 配置支持：

- `pnpm paperclipai onboard` 会写入默认 `secrets` 配置段（`local_encrypted`、严格模式关闭、密钥文件路径），并在需要时创建本地密钥文件。
- `pnpm paperclipai configure --section secrets` 可更新提供方/严格模式/密钥路径，并在需要时创建密钥文件。
- `pnpm paperclipai doctor` 会校验 secrets 适配器配置，可用 `--repair` 创建缺失的本地密钥文件。

已有内联 env secrets 的迁移命令：

```sh
pnpm secrets:migrate-inline-env         # 预览
pnpm secrets:migrate-inline-env --apply # 执行迁移
```

## 公司删除开关（Company Deletion Toggle）

公司删除作为开发/调试能力，可在运行时关闭：

```sh
PAPERCLIP_ENABLE_COMPANY_DELETION=false
```

默认行为：

- `local_trusted`：开启
- `authenticated`：关闭

## CLI 客户端操作（CLI Client Operations）

Paperclip CLI 除安装类命令外，还包含客户端控制平面命令。

示例：

```sh
pnpm paperclipai issue list --company-id <company-id>
pnpm paperclipai issue create --company-id <company-id> --title "Investigate checkout conflict"
pnpm paperclipai issue update <issue-id> --status in_progress --comment "Started triage"
```

用上下文档案一次性设置默认值：

```sh
pnpm paperclipai context set --api-base http://localhost:3100 --company-id <company-id>
```

之后可省略重复参数：

```sh
pnpm paperclipai issue list
pnpm paperclipai dashboard get
```

完整命令参考见 `doc/CLI.md`。

## OpenClaw 邀请入驻端点（OpenClaw Invite Onboarding Endpoints）

面向 agent 的邀请入驻现提供机器可读 API 文档：

- `GET /api/invites/:token` 返回邀请摘要及入驻与 skills 索引链接
- `GET /api/invites/:token/onboarding` 返回入驻清单详情（注册端点、认领端点模板、skill 安装提示）
- `GET /api/skills/index` 列出可用 skill 文档
- `GET /api/skills/paperclip` 返回 Paperclip 心跳 skill 的 Markdown
