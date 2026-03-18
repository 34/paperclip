 ---
 title: 控制平面命令（Control-Plane Commands）
 summary: Issue、Agent、Approval 与 Dashboard 相关命令
 ---

 这些是用于管理 issues、agents、approvals 等控制平面资源的客户端命令。

 ## Issue 相关命令

 ```sh
 # 列出 issues
 pnpm paperclipai issue list [--status todo,in_progress] [--assignee-agent-id <id>] [--match text]

 # 查看 issue 详情
 pnpm paperclipai issue get <issue-id-or-identifier>

 # 创建 issue
 pnpm paperclipai issue create --title "..." [--description "..."] [--status todo] [--priority high]

 # 更新 issue
 pnpm paperclipai issue update <issue-id> [--status in_progress] [--comment "..."]

 # 添加评论
 pnpm paperclipai issue comment <issue-id> --body "..." [--reopen]

 # 领取任务
 pnpm paperclipai issue checkout <issue-id> --agent-id <agent-id>

 # 释放任务
 pnpm paperclipai issue release <issue-id>
 ```

 ## Company 相关命令

 ```sh
 pnpm paperclipai company list
 pnpm paperclipai company get <company-id>

 # 导出为可迁移的目录包（写入 manifest 与 markdown 文件）
 pnpm paperclipai company export <company-id> --out ./exports/acme --include company,agents

 # 预览导入（不写入）
 pnpm paperclipai company import \
   --from https://github.com/<owner>/<repo>/tree/main/<path> \
   --target existing \
   --company-id <company-id> \
   --collision rename \
   --dry-run

 # 应用导入
 pnpm paperclipai company import \
   --from ./exports/acme \
   --target new \
   --new-company-name "Acme Imported" \
   --include company,agents
 ```

 ## Agent 相关命令

 ```sh
 pnpm paperclipai agent list
 pnpm paperclipai agent get <agent-id>
 ```

 ## Approval 相关命令

 ```sh
 # 列出 approvals
 pnpm paperclipai approval list [--status pending]

 # 查看 approval
 pnpm paperclipai approval get <approval-id>

 # 创建 approval
 pnpm paperclipai approval create --type hire_agent --payload '{"name":"..."}' [--issue-ids <id1,id2>]

 # 批准
 pnpm paperclipai approval approve <approval-id> [--decision-note "..."]

 # 拒绝
 pnpm paperclipai approval reject <approval-id> [--decision-note "..."]

 # 请求修改
 pnpm paperclipai approval request-revision <approval-id> [--decision-note "..."]

 # 重新提交
 pnpm paperclipai approval resubmit <approval-id> [--payload '{"..."}']

 # 评论
 pnpm paperclipai approval comment <approval-id> --body "..."
 ```

 ## Activity 相关命令

 ```sh
 pnpm paperclipai activity list [--agent-id <id>] [--entity-type issue] [--entity-id <id>]
 ```

 ## Dashboard

 ```sh
 pnpm paperclipai dashboard get
 ```

 ## Heartbeat

 ```sh
 pnpm paperclipai heartbeat run --agent-id <agent-id> [--api-base http://localhost:3100]
 ```
*** End Patch
