 ---
 title: 安装与初始化命令（Setup Commands）
 summary: onboard、run、doctor 与 configure
 ---

 用于实例初始化与诊断的一组命令。

 ## `paperclipai run`

 一条命令完成引导并启动：

 ```sh
 pnpm paperclipai run
 ```

 它会执行：

 1. 如果缺少配置则自动执行初始化（auto-onboard）
 2. 运行 `paperclipai doctor` 并开启自动修复
 3. 在检查通过后启动服务器

 指定特定实例：

 ```sh
 pnpm paperclipai run --instance dev
 ```

 ## `paperclipai onboard`

 交互式首次配置：

 ```sh
 pnpm paperclipai onboard
 ```

 首次提问时提供两种模式：

 1. `Quickstart`（推荐）：本地默认配置（嵌入式数据库、无 LLM 提供方、本地磁盘存储、默认 secrets）
 2. `Advanced setup`：完整交互式配置流程

 完成 Onboard 后立即启动：

 ```sh
 pnpm paperclipai onboard --run
 ```

 非交互默认配置 + 立即启动（在服务器监听后自动打开浏览器）：

 ```sh
 pnpm paperclipai onboard --yes
 ```

 ## `paperclipai doctor`

 健康检查，支持自动修复：

 ```sh
 pnpm paperclipai doctor
 pnpm paperclipai doctor --repair
 ```

 会校验：

 - 服务器配置
 - 数据库连通性
 - Secrets 适配器配置
 - 存储配置
 - 关键文件是否缺失

 ## `paperclipai configure`

 更新配置片段：

 ```sh
 pnpm paperclipai configure --section server
 pnpm paperclipai configure --section secrets
 pnpm paperclipai configure --section storage
 ```

 ## `paperclipai env`

 展示解析后的环境配置：

 ```sh
 pnpm paperclipai env
 ```

 ## `paperclipai allowed-hostname`

 为认证/私有模式允许一个私有主机名：

 ```sh
 pnpm paperclipai allowed-hostname my-tailscale-host
 ```

 ## 本地存储路径（Local Storage Paths）

 | 数据 | 默认路径 |
 |------|----------|
 | Config | `~/.paperclip/instances/default/config.json` |
 | Database | `~/.paperclip/instances/default/db` |
 | Logs | `~/.paperclip/instances/default/logs` |
 | Storage | `~/.paperclip/instances/default/data/storage` |
 | Secrets key | `~/.paperclip/instances/default/secrets/master.key` |

 可以通过以下方式覆盖：

 ```sh
 PAPERCLIP_HOME=/custom/home PAPERCLIP_INSTANCE_ID=dev pnpm paperclipai run
 ```

 或者在任意命令上直接传 `--data-dir`：

 ```sh
 pnpm paperclipai run --data-dir ./tmp/paperclip-dev
 pnpm paperclipai doctor --data-dir ./tmp/paperclip-dev
 ```
*** End Patch
