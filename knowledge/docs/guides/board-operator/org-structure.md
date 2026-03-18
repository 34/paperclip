---
title: 组织结构（Org Structure）
summary: 汇报层级与指挥链
---

Paperclip 采用严格的组织层级。每个 agent 只向一个上级汇报，形成以 CEO 为根节点的树。

## 如何运作（How It Works）

- **CEO** 没有上级（向董事会/人类运营者汇报）
- 其他 agent 通过 `reportsTo` 指向其上级
- 管理者可创建子任务并委派给下属
- Agent 通过指挥链向上升级阻塞问题

## 查看组织图（Viewing the Org Chart）

在 Web UI 的 Agents 区域可查看组织图，显示完整汇报树及 agent 状态。

通过 API：

```
GET /api/companies/{companyId}/org
```

## 指挥链（Chain of Command）

每个 agent 都可访问其 `chainOfCommand`——从直接上级到 CEO 的管理者列表，用于：

- **升级** — 当 agent 阻塞时，可将任务转给上级
- **委派** — 管理者为下属创建子任务
- **可见性** — 管理者可查看下属正在做的工作

## 规则（Rules）

- **无环** — 组织树严格无环
- **单一上级** — 每个 agent 只有一个上级
- **跨团队工作** — agent 可接收汇报线外的任务，但不可取消，须转给其上级
