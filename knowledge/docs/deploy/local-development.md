 ---
 title: 本地开发（Local Development）
 summary: 为本地开发环境启动 Paperclip
 ---

 在本地零外部依赖地运行 Paperclip。

 ## 前置条件（Prerequisites）

 - Node.js 20+
 - pnpm 9+

 ## 启动开发服务器（Start Dev Server）

 ```sh
 pnpm install
 pnpm dev
 ```

 这会启动：

 - **API 服务**：`http://localhost:3100`
 - **UI**：由 API 服务以 dev middleware 模式提供（同源）

 无需 Docker 或外部数据库。Paperclip 会自动使用嵌入式 PostgreSQL。

 ## 一键引导（One-Command Bootstrap）

 初次安装推荐使用：

 ```sh
 pnpm paperclipai run
 ```

 这条命令会：

 1. 如果缺失配置则自动 Onboard
 2. 运行 `paperclipai doctor` 并开启自动修复
 3. 检查通过后启动服务器

 ## Tailscale/私网认证开发模式（Tailscale/Private Auth Dev Mode）

 要在 `authenticated/private` 模式下运行以便网络访问：

 ```sh
 pnpm dev --tailscale-auth
 ```

 服务器会绑定到 `0.0.0.0` 以支持私有网络访问。

 允许额外的私有主机名：

 ```sh
 pnpm paperclipai allowed-hostname dotta-macbook-pro
 ```

 ## 健康检查（Health Checks）

 ```sh
 curl http://localhost:3100/api/health
 # -> {"status":"ok"}

 curl http://localhost:3100/api/companies
 # -> []
 ```

 ## 重置本地开发数据（Reset Dev Data）

 如需清空本地数据重新开始：

 ```sh
 rm -rf ~/.paperclip/instances/default/db
 pnpm dev
 ```

 ## 数据位置（Data Locations）

 | 数据 | 路径 |
 |------|------|
 | Config | `~/.paperclip/instances/default/config.json` |
 | Database | `~/.paperclip/instances/default/db` |
 | Storage | `~/.paperclip/instances/default/data/storage` |
 | Secrets key | `~/.paperclip/instances/default/secrets/master.key` |
 | Logs | `~/.paperclip/instances/default/logs` |

 可以通过环境变量覆盖：

 ```sh
 PAPERCLIP_HOME=/custom/path PAPERCLIP_INSTANCE_ID=dev pnpm paperclipai run
 ```
