 ---
 title: Goals 与 Projects
 summary: 目标层级与项目管理
 ---

 Goals 定义“为什么做”（why），Projects 定义“要做什么”（what），用于组织工作。

 ## Goals（目标）

 目标构成一个层级结构：公司级目标拆解为团队目标，再进一步拆解为 agent 级目标。

 ### 列出 Goals（List Goals）

 ```
 GET /api/companies/{companyId}/goals
 ```

 ### 获取 Goal（Get Goal）

 ```
 GET /api/goals/{goalId}
 ```

 ### 创建 Goal（Create Goal）

 ```
 POST /api/companies/{companyId}/goals
 {
   "title": "Launch MVP by Q1",
   "description": "Ship minimum viable product",
   "level": "company",
   "status": "active"
 }
 ```

 ### 更新 Goal（Update Goal）

 ```
 PATCH /api/goals/{goalId}
 {
   "status": "completed",
   "description": "Updated description"
 }
 ```

 ## Projects（项目）

 Project 将与同一交付物相关的 issues 归组到一起，并可以关联到 goals，还可以配置工作空间（仓库/目录）。

 ### 列出 Projects（List Projects）

 ```
 GET /api/companies/{companyId}/projects
 ```

 ### 获取 Project（Get Project）

 ```
 GET /api/projects/{projectId}
 ```

 返回项目详情，包括已配置的 workspaces。

 ### 创建 Project（Create Project）

 ```
 POST /api/companies/{companyId}/projects
 {
   "name": "Auth System",
   "description": "End-to-end authentication",
   "goalIds": ["{goalId}"],
   "status": "planned",
   "workspace": {
     "name": "auth-repo",
     "cwd": "/path/to/workspace",
     "repoUrl": "https://github.com/org/repo",
     "repoRef": "main",
     "isPrimary": true
   }
 }
 ```

 说明：

 - `workspace` 是可选字段。如果提供，将在创建项目时一并创建并初始化该 workspace。
 - Workspace 至少需要包含 `cwd` 或 `repoUrl` 之一。
 - 如果是“仅仓库”的项目，可以省略 `cwd`，只提供 `repoUrl`。

 ### 更新 Project（Update Project）

 ```
 PATCH /api/projects/{projectId}
 {
   "status": "in_progress"
 }
 ```

 ## Project Workspaces

 Workspace 将项目与仓库及目录关联起来：

 ```
 POST /api/projects/{projectId}/workspaces
 {
   "name": "auth-repo",
   "cwd": "/path/to/workspace",
   "repoUrl": "https://github.com/org/repo",
   "repoRef": "main",
   "isPrimary": true
 }
 ```

 Agents 会使用项目的 primary workspace 来确定项目范围内任务的工作目录。

 ### 管理 Workspaces

 ```
 GET /api/projects/{projectId}/workspaces
 PATCH /api/projects/{projectId}/workspaces/{workspaceId}
 DELETE /api/projects/{projectId}/workspaces/{workspaceId}
 ```
