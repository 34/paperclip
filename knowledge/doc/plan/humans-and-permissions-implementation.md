# 人类与权限实现（V1）（Humans and Permissions Implementation）

状态：草稿  
日期：2026-02-21  
负责人：Server + UI + CLI + DB + Shared  
配套计划：`doc/plan/humans-and-permissions.md`

## 1. 文档角色（Document role）

本文档是人类与权限计划的工程实现契约，将产品决策转化为具体的 schema、API、中间件、UI、CLI 与测试工作。若与先前探索性笔记冲突，以本文档为 V1 执行依据。

## 2. 已锁定的 V1 决策（Locked V1 decisions）

1. 保留两种部署模式：`local_trusted`、`cloud_hosted`。
2. `local_trusted`：无登录 UX、隐式本地实例管理员主体、仅回环绑定、本地仍可使用全部管理/设置/邀请/审批能力。
3. `cloud_hosted`：人类使用 Better Auth、仅邮箱+密码、V1 不要求邮箱验证。
4. 权限：人类与 agent 共用一套授权体系；规范化授权表（`principal_permission_grants`）；无单独「agent 权限引擎」。
5. 邀请：V1 仅复制链接（不发邮件）；统一 `company_join` 链接支持人类或 agent 路径；接受后创建 `pending_approval` 加入请求；未经管理员批准前无访问权。
6. 加入审核元数据：需要来源 IP；V1 不做 GeoIP/国家查询。
7. Agent API key：默认长期有效、静态存储哈希、认领时仅展示一次、支持撤销/重新生成。
8. 本地入站：V1 不包含公共/不可信入站；V1 无 `--dangerous-agent-ingress`。

## 3. 当前基线及差异（Current baseline and delta）

当前基线：服务端 actor 默认 `board`、授权主要为 `assertBoard` + 公司检查、本地 schema 无人类认证/会话表、无 principal 成员或授权表、无邀请或加入请求生命周期。所需差异：从 board-vs-agent 授权迁移到基于 principal 的授权、在云端模式接入 Better Auth 等。（完整 schema/API/UI 变更见英文原文。）
