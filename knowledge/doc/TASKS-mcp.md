# 任务管理 MCP 接口（Task Management MCP Interface）

Paperclip 任务管理系统的函数契约，定义 agents（及外部工具）通过 MCP 可用的操作。底层数据模型见 [TASKS.md](./TASKS.md)。

所有操作返回 JSON。ID 为 UUID。时间戳为 ISO 8601。Issue 标识符（如 `ENG-123`）在需要 issue `id` 的地方均可使用。

---

## Issues

### `list_issues`

列出并筛选工作区中的 issues。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `query` | string | 否 | 标题与描述全文搜索 |
| `teamId` | string | 否 | 按团队筛选 |
| `status` | string | 否 | 按工作流状态筛选 |
| `stateType` | string | 否 | 按状态类别：triage、backlog、unstarted、started、completed、cancelled |
| `assigneeId` | string | 否 | 按负责人（agent id）筛选 |
| `projectId` / `parentId` / `labelIds` / `priority` |  | 否 | 见英文原文 |
| `includeArchived` | boolean | 否 | 默认 false |
| `orderBy` | string | 否 | created、updated、priority、due_date，默认 created |
| `limit` / `after` / `before` |  | 否 | 分页 |

**返回：** `{ issues: Issue[], pageInfo: { hasNextPage, endCursor, hasPreviousPage, startCursor } }`

### `get_issue`

按 ID 或 identifier 获取单个 issue，并展开所有关联（state、assignee、labels、relations、children、parent、comments）。

### `create_issue` / `update_issue` / 其他

（完整参数与返回值见英文原文。）
