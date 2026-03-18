# Docker 快速上手（Docker Quickstart）

在 Docker 中运行 Paperclip，无需在本地安装 Node 或 pnpm。

## 一行命令（构建并运行）（One-liner）

```sh
docker build -t paperclip-local . && \
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -v "$(pwd)/data/docker-paperclip:/paperclip" \
  paperclip-local
```

访问：`http://localhost:3100`

数据持久化：

- 嵌入式 PostgreSQL 数据
- 上传资源
- 本地 secrets 密钥
- 本地 agent 工作空间数据

以上均保存在你挂载的目录（上例为 `./data/docker-paperclip`）。

## Compose 快速上手（Compose Quickstart）

```sh
docker compose -f docker-compose.quickstart.yml up --build
```

默认：

- 宿主机端口：`3100`
- 持久化数据目录：`./data/docker-paperclip`

可选覆盖：

```sh
PAPERCLIP_PORT=3200 PAPERCLIP_DATA_DIR=./data/pc docker compose -f docker-compose.quickstart.yml up --build
```

## Docker 中的 Claude + Codex 本地适配器（Claude + Codex Local Adapters in Docker）

镜像内已预装：

- `claude`（Anthropic Claude Code CLI）
- `codex`（OpenAI Codex CLI）

若要在容器内运行本地适配器，启动容器时传入 API key：

```sh
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -e OPENAI_API_KEY=... \
  -e ANTHROPIC_API_KEY=... \
  -v "$(pwd)/data/docker-paperclip:/paperclip" \
  paperclip-local
```

说明：

- 不传 API key 时应用仍可正常启动
- Paperclip 中的适配器环境检查会提示缺少认证/CLI 等前置条件
