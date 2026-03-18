 ---
 title: Docker 部署
 summary: 使用 Docker Compose 的快速启动
 ---

 使用 Docker 运行 Paperclip，而无需在本机安装 Node 或 pnpm。

 ## Compose 快速上手（推荐）

 ```sh
 docker compose -f docker-compose.quickstart.yml up --build
 ```

 然后打开浏览器访问 [http://localhost:3100](http://localhost:3100)。

 默认配置：

 - 宿主机端口：`3100`
 - 数据目录：`./data/docker-paperclip`

 可以通过环境变量覆盖：

 ```sh
 PAPERCLIP_PORT=3200 PAPERCLIP_DATA_DIR=./data/pc \
   docker compose -f docker-compose.quickstart.yml up --build
 ```

 ## 手动构建 Docker 镜像（Manual Docker Build）

 ```sh
 docker build -t paperclip-local .
 docker run --name paperclip \
   -p 3100:3100 \
   -e HOST=0.0.0.0 \
   -e PAPERCLIP_HOME=/paperclip \
   -v "$(pwd)/data/docker-paperclip:/paperclip" \
   paperclip-local
 ```

 ## 数据持久化（Data Persistence）

 所有数据都保存在挂载目录 `./data/docker-paperclip` 下：

 - 嵌入式 PostgreSQL 数据
 - 上传的附件资源
 - 本地 secrets 密钥
 - agent 工作空间数据

 ## 在 Docker 中使用 Claude 与 Codex 适配器

 Docker 镜像中预装了：

 - `claude`（Anthropic Claude Code CLI）
 - `codex`（OpenAI Codex CLI）

 通过传入 API key，可以在容器内启用本地适配器运行：

 ```sh
 docker run --name paperclip \
   -p 3100:3100 \
   -e HOST=0.0.0.0 \
   -e PAPERCLIP_HOME=/paperclip \
   -e OPENAI_API_KEY=sk-... \
   -e ANTHROPIC_API_KEY=sk-... \
   -v "$(pwd)/data/docker-paperclip:/paperclip" \
   paperclip-local
 ```

 如果不提供 API key，应用依然可以正常运行——适配器环境检查会提示缺失的前置条件。
