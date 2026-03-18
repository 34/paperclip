# CLI 参考（CLI Reference）

Paperclip CLI 现支持两类能力：

- 实例安装与诊断（`onboard`、`doctor`、`configure`、`env`、`allowed-hostname`）
- 控制平面客户端操作（issues、approvals、agents、activity、dashboard）

## 基本用法（Base Usage）

在开发中通过仓库脚本调用：

```sh
pnpm paperclipai --help
```

首次本地引导并运行：

```sh
pnpm paperclipai run
```

指定本地实例：

```sh
pnpm paperclipai run --instance dev
```

## 部署模式（Deployment Modes）

模式分类与设计意图见 `doc/DEPLOYMENT-MODES.md`。

当前 CLI 行为：

- `paperclipai onboard` 与 `paperclipai configure --section server` 在配置中设置部署模式
- 运行时可通过 `PAPERCLIP_DEPLOYMENT_MODE` 覆盖模式
- `paperclipai run` 与 `paperclipai doctor` 尚未提供直接的 `--mode` 参数

目标行为（规划中）见 `doc/DEPLOYMENT-MODES.md` 第 5 节。

允许一个已认证/私有主机名（如自定义 Tailscale DNS）：

```sh
pnpm paperclipai allowed-hostname dotta-macbook-pro
```

所有客户端命令支持：

- `--data-dir <path>`
- `--api-base <url>`
- `--api-key <token>`
- `--context <path>`
- `--profile <name>`
- `--json`

公司作用域命令还支持 `--company-id <id>`。

在任意 CLI 命令上使用 `--data-dir` 可将默认本地状态（config/context/db/logs/storage/secrets）与 `~/.paperclip` 隔离：

```sh
pnpm paperclipai run --data-dir ./tmp/paperclip-dev
pnpm paperclipai issue list --data-dir ./tmp/paperclip-dev
```

## 上下文档案（Context Profiles）

将本地默认值保存在 `~/.paperclip/context.json`：

```sh
pnpm paperclipai context set --api-base http://localhost:3100 --company-id <company-id>
pnpm paperclipai context show
pnpm paperclipai context list
pnpm paperclipai context use default
```

若不想在上下文中存密钥，可设置 `apiKeyEnvVarName` 并将 key 放在环境变量中：

```sh
pnpm paperclipai context set --api-key-env-var-name PAPERCLIP_API_KEY
export PAPERCLIP_API_KEY=...
```

## 公司命令（Company Commands）

```sh
pnpm paperclipai company list
pnpm paperclipai company get <company-id>
pnpm paperclipai company delete <company-id-or-prefix> --yes --confirm <same-id-or-prefix>
```

示例：

```sh
pnpm paperclipai company delete PAP --yes --confirm PAP
pnpm paperclipai company delete 5cbe79ee-acb3-4597-896e-7662742593cd --yes --confirm 5cbe79ee-acb3-4597-896e-7662742593cd
```

说明：

- 删除是否允许由服务端 `PAPERCLIP_ENABLE_COMPANY_DELETION` 控制
- 在 agent 认证下，公司删除按公司作用域；请使用当前公司 ID/前缀（如通过 `--company-id` 或 `PAPERCLIP_COMPANY_ID`），不能删其他公司

## Issue 命令（Issue Commands）

```sh
pnpm paperclipai issue list --company-id <company-id> [--status todo,in_progress] [--assignee-agent-id <agent-id>] [--match text]
pnpm paperclipai issue get <issue-id-or-identifier>
pnpm paperclipai issue create --company-id <company-id> --title "..." [--description "..."] [--status todo] [--priority high]
pnpm paperclipai issue update <issue-id> [--status in_progress] [--comment "..."]
pnpm paperclipai issue comment <issue-id> --body "..." [--reopen]
pnpm paperclipai issue checkout <issue-id> --agent-id <agent-id> [--expected-statuses todo,backlog,blocked]
pnpm paperclipai issue release <issue-id>
```

## Agent 命令（Agent Commands）

```sh
pnpm paperclipai agent list --company-id <company-id>
pnpm paperclipai agent get <agent-id>
```

## Approval 命令（Approval Commands）

```sh
pnpm paperclipai approval list --company-id <company-id> [--status pending]
pnpm paperclipai approval get <approval-id>
pnpm paperclipai approval create --company-id <company-id> --type hire_agent --payload '{"name":"..."}' [--issue-ids <id1,id2>]
pnpm paperclipai approval approve <approval-id> [--decision-note "..."]
pnpm paperclipai approval reject <approval-id> [--decision-note "..."]
pnpm paperclipai approval request-revision <approval-id> [--decision-note "..."]
pnpm paperclipai approval resubmit <approval-id> [--payload '{"...":"..."}']
pnpm paperclipai approval comment <approval-id> --body "..."
```

## Activity 命令（Activity Commands）

```sh
pnpm paperclipai activity list --company-id <company-id> [--agent-id <agent-id>] [--entity-type issue] [--entity-id <id>]
```

## Dashboard 命令（Dashboard Commands）

```sh
pnpm paperclipai dashboard get --company-id <company-id>
```

## Heartbeat 命令（Heartbeat Command）

`heartbeat run` 现支持 context/api-key 等选项，并复用统一客户端栈：

```sh
pnpm paperclipai heartbeat run --agent-id <agent-id> [--api-base http://localhost:3100] [--api-key <token>]
```

## 本地存储默认路径（Local Storage Defaults）

默认本地实例根目录为 `~/.paperclip/instances/default`：

- config: `~/.paperclip/instances/default/config.json`
- 嵌入式 db: `~/.paperclip/instances/default/db`
- logs: `~/.paperclip/instances/default/logs`
- storage: `~/.paperclip/instances/default/data/storage`
- secrets key: `~/.paperclip/instances/default/secrets/master.key`

通过环境变量覆盖根目录或实例：

```sh
PAPERCLIP_HOME=/custom/home PAPERCLIP_INSTANCE_ID=dev pnpm paperclipai run
```

## 存储配置（Storage Configuration）

配置存储提供方与设置：

```sh
pnpm paperclipai configure --section storage
```

支持的提供方：

- `local_disk`（默认；本地单用户安装）
- `s3`（S3 兼容对象存储）
