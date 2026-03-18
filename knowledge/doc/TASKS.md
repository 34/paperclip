# 任务管理数据模型（Task Management Data Model）

Paperclip 中任务追踪的参考：实体、关系及任务生命周期规则。采用目标模型描述——部分已实现，部分为规划。

---

## 实体层级（Entity Hierarchy）

```
Workspace
  Initiatives（计划/路线图级目标，跨季度）
    Projects（有期限的交付物，可跨团队）
      Milestones（项目内阶段）
        Issues（工作单元，核心实体）
          Sub-issues（父 issue 下的拆解工作）
```

自上而下。Initiative 包含 Projects；Project 包含 Milestones 与 Issues；Issue 可有子 issues。

---

## Issues（核心实体）

Issue 是工作的基本单元。

### 字段（Fields）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | uuid | 是 | 主键 |
| `identifier` | string | 计算 | 人类可读，如 `ENG-123`（团队 key + 自增号） |
| `title` | string | 是 | 简短摘要 |
| `description` | text/markdown | 否 | 支持 Markdown |
| `status` | WorkflowState FK | 是 | 默认团队默认状态 |
| `priority` | enum (0-4) | 否 | 默认 0（无） |
| `estimate` | number | 否 | 复杂度/点数 |
| `dueDate` | date | 否 | |
| `teamId` | uuid FK | 是 | 每个 issue 属于唯一团队 |
| `projectId` | uuid FK | 否 | 至多一个 project |
| `milestoneId` | uuid FK | 否 | 至多一个 milestone |
| `assigneeId` | uuid FK | 否 | **单一负责人** |
| `creatorId` | uuid FK | 否 | 创建者 |
| `parentId` | uuid FK (self) | 否 | 父 issue |
| `goalId` | uuid FK | 否 | 关联目标 |
| `sortOrder` | float | 否 | 视图内排序 |
| `startedAt` / `completedAt` / `cancelledAt` | timestamp | 计算 | 进入对应状态时设置 |

---

## 工作流状态（Workflow States）

Issue 状态**不是**扁平枚举，而是团队自定义的命名状态，每个状态属于固定**类别**之一：Triage、Backlog、Unstarted、Started、Completed、Cancelled。规则：每团队在这些类别下定义自己的状态；至少每类一个状态；新 issue 默认团队第一个 Backlog 状态；进入 Started/Completed/Cancelled 时自动设置对应时间戳。

---

## 优先级（Priority）

固定数值：0=无、1=Urgent、2=High、3=Medium、4=Low。

---

## 团队（Teams）

团队是主要组织单元，几乎一切以团队为作用域。字段：id、name、key（如 ENG，用于 identifier）、description。每个 issue 属于恰好一个团队；工作流状态按团队；标签可团队或 workspace 级；项目可跨团队。

---

## 项目与里程碑（Projects / Milestones）

Project 将 issues 归组为有期限的交付物，可跨团队。Milestone 将项目拆分为阶段。Issue 至多属于一个 project、一个 milestone。

---

## 标签（Labels / Tags）

Workspace 级或 Team 级。可分组（组内标签在同一 issue 上互斥）。多对多通过 `issue_labels`。

---

## Issue 关系/依赖（Issue Relations）

类型：`related`、`blocks`、`blocked_by`、`duplicate`。重复会将被标记的 issue 自动移到 Cancelled。阻塞不传递。

---

## 负责人（Assignees）

**单一负责人模型**。每个 issue 同一时间至多一个 assignee。在我们场景中 assignee 为 agent。

---

## 子 Issue（Sub-issues）

通过 `parentId` 建立父子；支持多级。子 issue 创建时继承父的 project（不继承 team/标签/负责人）。父完成时未完成的子 issue 可自动完成。

---

## 评论（Comments）

字段：body、issueId、authorId（用户或 agent）、parentId（回复）、resolvedAt、createdAt、updatedAt。

---

## Initiatives

最高层规划，将 projects 归组到战略目标。有 owner、status、targetDate，通常用结果/OKR 衡量。

---

## 标识符（Identifiers）

人类可读格式 `{TEAM_KEY}-{NUMBER}`，如 ENG-123。换团队时生成新 identifier，旧值保留在 `previousIdentifiers`。

---

## 实现优先级（Implementation Priority）

建议顺序：1. Teams；2. Workflow states；3. Labels；4. Issue Relations；5. Sub-issues；6. Comments；7. 状态时间戳；8. Milestones；9. Initiatives；10. Estimates。
