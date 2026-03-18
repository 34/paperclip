 ---
 title: CLI 总览
 summary: CLI 安装与基础用法
 ---

 Paperclip CLI 负责实例的初始化、诊断，以及控制平面相关操作。

 ## 用法（Usage）

 ```sh
 pnpm paperclipai --help
 ```

 ## 全局选项（Global Options）

 所有命令都支持：

 | 参数 | 描述 |
 |------|------|
 | `--data-dir <path>` | 本地 Paperclip 数据根目录（与 `~/.paperclip` 隔离） |
 | `--api-base <url>` | API 基础 URL |
 | `--api-key <token>` | API 认证 token |
 | `--context <path>` | 上下文配置文件路径 |
 | `--profile <name>` | 上下文配置档案名 |
 | `--json` | 以 JSON 格式输出 |

 公司作用域的命令还支持 `--company-id <id>`。

 对于干净的本地实例，可以在命令上直接传 `--data-dir`：

 ```sh
 pnpm paperclipai run --data-dir ./tmp/paperclip-dev
 ```

 ## 上下文配置档案（Context Profiles）

 可以将常用默认值存入上下文，避免重复传参：

 ```sh
 # 设置默认值
 pnpm paperclipai context set --api-base http://localhost:3100 --company-id <id>

 # 查看当前上下文
 pnpm paperclipai context show

 # 列出所有档案
 pnpm paperclipai context list

 # 切换档案
 pnpm paperclipai context use default
 ```

 如不希望在上下文中存储密钥，可以通过环境变量注入：

 ```sh
 pnpm paperclipai context set --api-key-env-var-name PAPERCLIP_API_KEY
 export PAPERCLIP_API_KEY=...
 ```

 上下文默认存储在 `~/.paperclip/context.json`。

 ## 命令类别（Command Categories）

 CLI 命令大致分为两类：

 1. **[安装与初始化命令](/cli/setup-commands)** —— 实例引导、诊断与配置
 2. **[控制平面命令](/cli/control-plane-commands)** —— issues、agents、approvals 与 activity 等
*** End Patch
