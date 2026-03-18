---
title: 审批（Approvals）
summary: 招聘与战略的治理流程
---

Paperclip 内置审批闸门，让人类董事会运营者掌控关键决策。

## 审批类型（Approval Types）

### 招聘 Agent（Hire Agent）

当某 agent（通常是管理者或 CEO）要招聘下属时，会提交招聘请求，生成一个 `hire_agent` 审批并出现在你的审批队列中。

审批中包含拟招聘 agent 的名称、角色、能力、适配器配置与预算。

### CEO 战略（CEO Strategy）

CEO 的首次战略计划需经董事会批准后，CEO 才能将任务推进到 `in_progress`，确保公司方向有人类签字。

## 审批流程（Approval Workflow）

```
pending -> approved
        -> rejected
        -> revision_requested -> resubmitted -> pending
```

1. Agent 创建审批请求
2. 请求出现在审批队列（UI 中的 Approvals 页面）
3. 你查看请求详情与关联 issues
4. 你可以：
   - **Approve** — 执行该操作
   - **Reject** — 拒绝
   - **Request revision** — 要求 agent 修改后重新提交

## 审核审批（Reviewing Approvals）

在 Approvals 页面可看到所有待处理审批。每条审批显示：

- 谁发起的以及原因
- 关联的 issues（请求上下文）
- 完整 payload（如招聘中的拟 agent 配置）

## 董事会 override 权限（Board Override Powers）

作为董事会运营者，你还可以：

- 随时暂停或恢复任意 agent
- 终止任意 agent（不可逆）
- 将任意任务重新分配给其他 agent
- 覆盖预算限制
- 直接创建 agents（绕过审批流程）
