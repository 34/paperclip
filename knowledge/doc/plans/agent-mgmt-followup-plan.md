# Agent 管理后续计划（CEO PATCH + 配置回滚 + Issue↔Approval 关联）

状态：提议  
日期：2026-02-19  

## 1. 调查结论（Investigation Findings）

- **CEO PATCH 失败原因**：路由逻辑显式限制「agent 只能修改自身」，CEO 虽有招聘权限仍被拦截。
- **评论质量**：当前 skill 未要求状态评论的 Markdown 格式（链接、结构），agents 易产出纯文本与裸 ID。
- **Issue↔Approval 关联缺失**：无直接 DB 关系，UI 无法可靠交叉链接。
- **配置回滚缺失**：无独立修订历史表或回滚端点。

## 2. 产品/行为变更（Product/Behavior Changes）

- 允许 CEO 对同公司其他 agents 执行 PATCH。
- 提升评论格式要求与 API 参考。
- 建立 issue–approval 规范关联与 UI 展示。
- 引入 agent 配置修订历史与回滚接口。

（完整清单与实现见英文原文。）
