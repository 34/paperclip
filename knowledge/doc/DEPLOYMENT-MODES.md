# 部署模式（Deployment Modes）

状态：规范部署与认证模式模型  
日期：2026-02-23

## 1. 目的（Purpose）

Paperclip 支持两种运行模式：

1. `local_trusted`
2. `authenticated`

`authenticated` 支持两种暴露策略：

1. `private`
2. `public`

这样在保留一套认证体系的同时，仍能区分低摩擦的私有网络默认行为与面向互联网的加固要求。

## 2. 规范模型（Canonical Model）

| 运行模式 | 暴露策略 | 人类认证 | 主要用途 |
|---|---|---|---|
| `local_trusted` | 不适用 | 无需登录 | 单人本机工作流 |
| `authenticated` | `private` | 需要登录 | 私有网络访问（如 Tailscale/VPN/局域网） |
| `authenticated` | `public` | 需要登录 | 面向互联网/云端部署 |

## 3. 安全策略（Security Policy）

### `local_trusted`

- 仅绑定回环地址
- 无人类登录流程
- 针对最快本地启动优化

### `authenticated + private`

- 需要登录
- 低摩擦 URL 处理（`auto` base URL 模式）
- 需要私有主机信任策略

### `authenticated + public`

- 需要登录
- 必须配置明确的公网 URL
- doctor 中会进行更严格的部署检查与失败判定

## 4. Onboarding UX 约定（Onboarding UX Contract）

默认 onboarding 保持交互式且无额外参数：

```sh
pnpm paperclipai onboard
```

服务端提示行为：

1. 询问模式，默认 `local_trusted`
2. 选项文案：`local_trusted`：「本地搭建最简单（无需登录、仅 localhost）」；`authenticated`：「需要登录；用于私有网络或公网托管」
3. 若选 `authenticated`，再问暴露策略：`private`：「私有网络访问（如 Tailscale），配置更简单」；`public`：「面向互联网部署，安全要求更严」
4. 仅在 `authenticated + public` 时询问明确的公网 URL

`configure --section server` 采用相同的交互逻辑。

## 5. Doctor UX 约定（Doctor UX Contract）

默认 doctor 无额外参数：

```sh
pnpm paperclipai doctor
```

Doctor 读取已配置的 mode/exposure 并执行与模式相关的检查。可选的覆盖参数为次要能力。

## 6. 董事会/用户集成约定（Board/User Integration Contract）

董事会身份必须由真实的 DB 用户主体表示，以便基于用户的功能一致工作。

需要的集成点：

- `authUsers` 中代表董事会身份的真实用户行
- `instance_user_roles` 中董事会管理员权限条目
- `company_memberships` 集成，用于用户级任务分配与访问

因为用户分配路径会校验 `assigneeUserId` 的活跃成员关系，上述集成是必需的。

## 7. Local Trusted -> Authenticated 认领流程（Claim Flow）

在 `authenticated` 模式下，若当前唯一的实例管理员是 `local-board`，Paperclip 会在启动时输出带一次性高熵认领 URL 的警告。

- URL 格式：`/board-claim/<token>?code=<code>`
- 用途：已登录人类认领董事会所有权
- 认领动作：将当前登录用户提升为 `instance_admin`、降级 `local-board` 管理员角色、确保认领用户在现有公司中拥有活跃的 owner 成员关系

这样在用户从长期使用的 local trusted 迁移到 authenticated 时不会出现锁死。

## 8. 当前代码现状（截至 2026-02-23）（Current Code Reality）

- 运行时可取值为 `local_trusted | authenticated`
- `authenticated` 使用 Better Auth 会话与引导邀请流程
- `local_trusted` 确保在 `authUsers` 中存在真实的本地 Board 用户主体，并具备 `instance_user_roles` 管理员权限
- 公司创建会确保创建者在 `company_memberships` 中，以便用户分配/访问流程一致

## 9. 命名与兼容策略（Naming and Compatibility Policy）

- 规范命名为 `local_trusted`、`authenticated` 及 `private`/`public` 暴露
- 对已废弃的命名变体不做长期兼容别名层

## 10. 与其他文档的关系（Relationship to Other Docs）

- 实现计划：`doc/plans/deployment-auth-mode-consolidation.md`
- V1 契约：`doc/SPEC-implementation.md`
- 运营者工作流：`doc/DEVELOPING.md` 与 `doc/CLI.md`
