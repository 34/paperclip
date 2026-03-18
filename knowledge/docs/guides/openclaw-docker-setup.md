# 在 Docker 中运行 OpenClaw（本地开发）

如何在 Docker 容器中运行 OpenClaw，用于本地开发和测试 Paperclip OpenClaw 适配器集成。

## 前置条件（Prerequisites）

- **Docker Desktop v29+**（支持 Docker Sandbox）
- **2 GB+ 可用内存**用于构建镜像
- **API keys** 放在 `~/.secrets`（至少需要 `OPENAI_API_KEY`）

## 方式 A：Docker Sandbox（推荐）

Docker Sandbox 提供更好隔离（基于 microVM）且配置更简单，需 Docker Desktop v29+ / Docker Sandbox v0.12+。

```bash
# 1. 克隆 OpenClaw 仓库并构建镜像
git clone https://github.com/openclaw/openclaw.git /tmp/openclaw-docker
cd /tmp/openclaw-docker
docker build -t openclaw:local -f Dockerfile .

# 2. 使用构建好的镜像创建 sandbox
docker sandbox create --name openclaw -t openclaw:local shell ~/.openclaw/workspace

# 3. 允许访问 OpenAI API
docker sandbox network proxy openclaw \
  --allow-host api.openai.com \
  --allow-host localhost

# 4. 在 sandbox 内写入配置（见原文完整脚本）
# ...

# 5. 启动 gateway（从 ~/.secrets 传入 API key）
source ~/.secrets
docker sandbox exec -d \
  -e OPENAI_API_KEY="$OPENAI_API_KEY" \
  -w /app openclaw \
  node dist/index.js gateway --bind loopback --port 18789

# 6. 等待约 15 秒后验证
sleep 15
docker sandbox exec openclaw curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:18789/
# 应输出: 200

# 7. 查看状态
docker sandbox exec -e OPENAI_API_KEY="$OPENAI_API_KEY" -w /app openclaw \
  node dist/index.js status
```

### Sandbox 管理（Sandbox Management）

```bash
# 列出 sandbox
docker sandbox ls

# 进入 sandbox shell
docker sandbox exec -it openclaw bash

# 停止 sandbox（保留状态）
docker sandbox stop openclaw

# 删除 sandbox
docker sandbox rm openclaw

# 查看 sandbox 版本
docker sandbox version
```

## 方式 B：Docker Compose（备选）

在无法使用 Docker Sandbox（Docker Desktop < v29）时使用。

```bash
# 1. 克隆 OpenClaw 仓库
git clone https://github.com/openclaw/openclaw.git /tmp/openclaw-docker
cd /tmp/openclaw-docker

# 2. 构建 Docker 镜像（首次约 5–10 分钟）
docker build -t openclaw:local -f Dockerfile .

# 3–9. 创建配置目录、生成 token、创建 openclaw.json、.env，
#       在 docker-compose.yml 中添加 tmpfs（见原文），启动 gateway 等
# ...
```

仪表盘 URL 形如：`http://127.0.0.1:18789/#token=<your-token>`

### Docker Compose 管理（Docker Compose Management）

```bash
cd /tmp/openclaw-docker

# 停止
docker compose down

# 再次启动（无需重建）
docker compose up -d openclaw-gateway

# 查看日志
docker compose logs -f openclaw-gateway

# 查看状态
docker compose run --rm openclaw-cli status

# 获取仪表盘 URL
docker compose run --rm openclaw-cli dashboard --no-open
```

## 已知问题与修复（Known Issues and Fixes）

### 启动容器时 "no space left on device"

Docker Desktop 虚拟磁盘可能已满。

```bash
docker system df                   # 查看占用
docker system prune -f             # 清理已停止容器、未使用网络
docker image prune -f              # 清理悬空镜像
```

### "Unable to create fallback OpenClaw temp dir: /tmp/openclaw-1000"（仅 Compose）

容器无法写入 `/tmp`。在 `docker-compose.yml` 中为**两个**服务都添加 `tmpfs` 挂载：

```yaml
services:
  openclaw-gateway:
    tmpfs:
      - /tmp:exec,size=512M
  openclaw-cli:
    tmpfs:
      - /tmp:exec,size=512M
```

Docker Sandbox 方式不受此影响。

### 社区模板镜像中的 Node 版本不匹配

部分社区 sandbox 模板（如 `olegselajev241/openclaw-dmr:latest`）使用 Node 20，而 OpenClaw 需要 Node >=22.12.0。请使用本地构建的 `openclaw:local` 镜像作为 sandbox 模板（内含 Node 22）。

### Gateway 启动后约 15 秒才有响应

Node.js gateway 需要初始化时间，访问 `http://127.0.0.1:18789/` 前请等待约 15 秒。

### CLAUDE_AI_SESSION_KEY 警告（仅 Compose）

这些 Docker Compose 警告可忽略：
```
level=warning msg="The \"CLAUDE_AI_SESSION_KEY\" variable is not set. Defaulting to a blank string."
```

## 配置（Configuration）

配置文件：`~/.openclaw/openclaw.json`（JSON5 格式）

主要设置：
- `gateway.auth.token` — Web UI 与 API 的认证 token
- `agents.defaults.model.primary` — AI 模型（使用 `openai/gpt-5.2` 或更新）
- `env.OPENAI_API_KEY` — 引用环境变量 `OPENAI_API_KEY`（Compose 方式）

API keys 存放在 `~/.secrets`，通过环境变量传入容器。

## 参考（Reference）

- [OpenClaw Docker 文档](https://docs.openclaw.ai/install/docker)
- [OpenClaw 配置参考](https://docs.openclaw.ai/gateway/configuration-reference)
- [Docker 博客：在 Docker Sandbox 中安全运行 OpenClaw](https://www.docker.com/blog/run-openclaw-securely-in-docker-sandboxes/)
- [Docker Sandbox 文档](https://docs.docker.com/ai/sandboxes)
- [OpenAI 模型](https://platform.openai.com/docs/models) — 当前模型：gpt-5.2、gpt-5.2-chat-latest、gpt-5.2-pro
