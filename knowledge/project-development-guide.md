# 用 Paperclip 持续完成整项目开发 — 实操指南

本文说明：如何让 Agent 在 Paperclip 里**持续运行**、**按项目完成所有任务**、并尽量**保证正确执行**；并以「开发一个英文学习应用」为例，串联从目标到上线的完整流程。

---

## 1. 核心机制回顾：系统如何“持续运行并完成任务”

### 1.1 任务从哪来、到哪去

- **目标（Goal）**：公司级/团队级/任务级，形成树（parent_id）。例如「3 个月内上线英文学习 App」。
- **项目（Project）**：一组相关工作的容器，可关联一个 Goal。例如「英文学习 App V1」。
- **任务（Issue）**：具体工单，挂在 Project 下，可有 parent issue（子任务）。例如「实现用户登录」「实现单词列表 API」「前端首页」。
- **原则**：所有工作都要能追溯到公司目标（通过 goal_id / parent_id / project→goal 链）。

Agent 不会“自动知道”要做什么，而是：

1. 被**唤醒**（定时 / 分配任务时 / 手动 / 自动化）。
2. 通过 Paperclip API 拿到**当前分配给自己的任务**（及可选上下文：目标摘要、最近评论等）。
3. **Checkout** 一个任务（原子占用，状态变为 in_progress）。
4. 在本地或远程执行**实际工作**（写代码、调 API、更新 Issue 评论等）。
5. 把任务推进到 **in_review** 或 **done**，或创建子任务再继续。
6. **Release** 当前 checkout（若需要），再取下一个任务，循环。

所以要“完成一个项目的所有任务”，需要两件事配合：**你把项目拆成 Issues** + **Agent 的 prompt/行为是「每次醒来就取任务→做一个→再取下一个」**。

### 1.2 持续运行的三个支柱

| 支柱 | 作用 | 配置要点 |
|------|------|----------|
| **心跳策略** | 决定 Agent 何时被拉起、多频繁 | `enabled`、`intervalSec`、`wakeOnAssignment`、`wakeOnOnDemand` |
| **会话延续** | 同一 Agent 多次心跳之间保留上下文 | 适配器存 sessionId（如 Claude/Codex），下次复用 |
| **执行约束** | 避免跑飞、超支、死循环 | `timeoutSec`、`graceSec`、预算、Board 随时暂停/取消 |

- **定时唤醒（timer）**：例如 `intervalSec: 300` 每 5 分钟唤醒一次，适合「持续扫任务队列」的节奏。
- **分配即唤醒（wakeOnAssignment）**：一有任务 assign 给该 Agent 就唤醒，适合事件驱动，减少空转。
- **手动唤醒（wakeOnOnDemand）**：你在 UI 点「唤醒」或调 API，适合调试或补一刀。

建议：**开发整项目**时同时开「定时 + 分配唤醒」：有活就立刻干，没活时定时起来看看有没有新任务或未完成的 in_progress。

### 1.3 “正确运行”靠什么保证

- **原子 checkout**：一个 Issue 同一时刻只能被一个 Run 占用（in_progress + assignee + executionRunId）。避免多个 Run 抢同一任务。
- **单一受让人**：一个 Issue 要么 assign 给一个 Agent，要么一个 User，不会两人同时干。
- **状态机**：只有合法状态转换（如 todo→in_progress→in_review→done），防止乱改状态。
- **审批与治理**：敏感操作（如再雇一个 Agent、CEO 策略）可走 Approval，Board 批准后才生效。
- **预算硬停**：Agent/公司超预算会自动 pause，防止成本失控。
- **人工兜底**：Board 可随时暂停 Agent、改 assignee、取消任务、看 Activity 和 Run 日志排查问题。

因此「保证正确」= **系统不让你乱来**（原子性、状态机、预算） + **你通过 prompt 和任务拆分引导 Agent 做对事** + **人做关键节点审核（如 in_review→done）**。

---

## 2. Agent 如何“完成一个项目的所有任务”

### 2.1 项目 ↔ 任务的关系

- 一个 **Project** 下有很多 **Issues**（`issue.projectId`）。
- Agent 看到的「我的工作」来自：**分配给自己的 Issues**（`assignee_agent_id = 我`），通常再按 status 过滤（todo / in_progress / backlog）。
- Agent **一次只应 checkout 一个 Issue**，做完（或做到一个可交付节点）后：
  - 更新状态（in_progress → in_review 或 done），
  - 可选 release 或直接让下一个心跳再取新任务。

所以「完成整个项目」在实现上 = **把项目拆成若干 Issues，部分或全部 assign 给一个或多个 Agent，Agent 按「取任务 → 做 → 更新状态/评论 → 再取」循环，直到没有未完成任务**。

### 2.2 单 Agent 串行做完一个项目

- 所有（或大部分）Issue 都 assign 给**同一个 Agent**。
- 该 Agent 的 **prompt 模板**要明确写清行为，例如（意译）：
  - 你是 Agent {{agent.name}}，为 Paperclip 公司工作。
  - 每次运行时，先通过 API 获取当前分配给自己的、状态为 todo/backlog/blocked 的任务；若没有则查 in_progress（是否自己之前 checkout 的）。
  - 若有待办，选一个 checkout，然后根据描述和评论完成开发/调试/文档，并在过程中用 API 更新评论、子任务。
  - 完成到可交付时，将状态改为 in_review 或 done，并 release；若只做了一部分，可留 in_progress 并写清进度，下次心跳继续。
  - 不要一次 checkout 多个任务；不要跳过 API 直接改本地文件而不更新 Paperclip。
- **心跳策略**：开启 `wakeOnAssignment`（新任务一来就做），并设 `intervalSec`（例如 300），这样没新任务时也会定期起来「收尾」未完成的 in_progress。
- **工作目录（cwd）**：指向**同一个代码仓库**（例如 monorepo 或前端+后端各一个 repo 的父目录），保证多次心跳在同一代码库上累积修改。

这样，从「英文学习 App」的 Issue 1 到 Issue N，都会由同一 Agent 按顺序（或按优先级）一个个做完。

### 2.3 多 Agent 分工做一个项目

- 把不同 Issue 分配给不同 Agent（例如前端 Agent、后端 Agent、运维 Agent）。
- 每个 Agent 的 prompt 和 cwd 按角色设定（例如前端 Agent 的 cwd 指向前端 repo）。
- 依赖关系用 **Issue 的 parent_id** 或 **描述/评论** 写清（例如「先完成 API 再做前端对接」）；人或在后续迭代中通过「先做父任务再做子任务」的顺序 assign。
- 同一时刻每个 Issue 仍只有一个 assignee，因此不会冲突；需要协作时通过 **评论 @ 对方** 或创建「对接」类 Issue 再 assign 给对方。

### 2.4 持续运行不中断的注意点

- **Session 延续**：Claude/Codex 等适配器会保存 session，下次心跳在同一会话里继续，适合多步开发。
- **超时与重试**：设好 `timeoutSec`、`graceSec`；若 Run 超时或失败，Agent 会回到 idle，下次心跳可重新 checkout 同一任务（若仍 assign 给自己）继续。
- **预算**：给 Agent（或公司）设月度预算，避免跑爆；超限会自动 pause，你可在 UI 提高预算或暂停不重要的 Agent。
- **Stale 与中断**：若一个 Run 卡死或你手动取消，Paperclip 会释放 execution lock；下次心跳可再次 checkout 该任务（或 Board 先 reassign）。

---

## 3. 示例：从零到“开发一个英文学习应用”的完整流程

下面用「英文学习 App」作为一条龙示例：从创建公司到 Agent 持续开发、直到项目完成。

### 3.1 第一步：创建公司与顶层目标

1. 登录 Paperclip，创建公司（例如「EnglishApp Inc」）。
2. 创建**公司级目标**：
   - 标题：`Ship an English learning app in 3 months`
   - 描述：可写 MVP 范围（单词学习、简单测验、用户账号）。
   - 级别：company，parent 为空。

### 3.2 第二步：创建项目与拆解任务

1. 创建项目：
   - 名称：`English Learning App — MVP`
   - 关联上面的 Goal（goal_id）。
   - 状态：planned → 随后可改为 in_progress。

2. 在项目下创建一批 **Issues**（建议先 10～20 个，可后续再加）：
   - **产品/设计**：如「定义 MVP 功能清单」「设计单词卡片 UI」。
   - **后端**：如「用户注册/登录 API」「单词列表 API」「学习进度 API」「数据模型与迁移」。
   - **前端**：如「登录/注册页」「首页/仪表盘」「单词列表与详情」「测验页」。
   - **集成与发布**：如「前后端联调」「部署到 Vercel + 后端托管」「基础监控与错误上报」。

每个 Issue：标题清晰、描述里写清验收标准（可拆子任务用 parent_id 或列在描述里）。优先级按需（critical/high/medium/low），状态先 backlog。

### 3.3 第三步：创建并配置开发 Agent

1. 在同一个公司下 **Hire Agent**（或 Board 直接创建）：
   - 名称：如 `Dev Lead` 或 `Full-Stack Dev`。
   - 角色：如 `engineer`。
   - 适配器：选 `claude_local` 或 `codex_local`（本机已装好对应 CLI）。

2. **Adapter 配置**（关键）：
   - **cwd**：指向你要开发的项目目录（例如 `~/projects/english-app` 或 monorepo 根目录）。确保该目录存在且 Agent 进程有权限。
   - **promptTemplate**：写清「每次运行都先查分配给我的任务，逐个 checkout→做→更新状态/评论→release 再取下一个」的逻辑（参见 2.2）。
   - **env**：至少配置 `PAPERCLIP_API_KEY`（或用 secret ref），让 CLI 能调 Paperclip API；若用 Claude/OpenAI API 计费，配置 `ANTHROPIC_API_KEY` / `OPENAI_API_KEY`。
   - **timeoutSec** / **graceSec**：例如 1800 / 60，避免单次心跳跑太久或杀不掉。

3. **心跳策略（runtimeConfig.heartbeat）**：
   - `enabled: true`
   - `intervalSec: 300`（5 分钟一轮，可按需要调大或调小）
   - `wakeOnAssignment: true`（新任务分配下来就立刻做）
   - `wakeOnOnDemand: true`（方便你手动点「唤醒」测试）

4. **工作空间**：若用 project workspace，把该 Agent 绑到「English Learning App」项目对应的工作目录，这样上下文里会带 `paperclipWorkspace`，Agent 知道在哪个 repo 干活。

### 3.4 第四步：把任务分配给 Agent 并启动

1. 把「English Learning App — MVP」下的 Issues 按执行顺序**逐个或批量 assign** 给 `Dev Lead`（或你设的 Agent）。  
   - 不必一次全 assign，可以先 assign 前 3～5 个，做完再 assign 下一批，避免列表过长干扰 prompt。

2. 第一次可**手动点「唤醒」**该 Agent，确认：
   - Run 能成功启动（看 Run 状态、stdout/stderr）；
   - Agent 能调通 Paperclip API（看日志里是否有 401/403）；
   - 能 checkout 一个 Issue 并开始写代码/更新评论。

3. 之后保持 **Paperclip server 和（若本地适配器）开发机常开**；Agent 会按 `intervalSec` 和 `wakeOnAssignment` 持续被唤醒，直到：
   - 没有未完成的任务，或
   - 你暂停/终止 Agent，或
   - 预算用尽自动 pause。

### 3.5 第五步：监控与纠偏

- **Dashboard / Issues 列表**：看哪些 in_progress、哪些 done、哪些 blocked。
- **Activity**：看谁创建/更新了任务、评论、Run 结果。
- **Agents / Run 详情**：看某次心跳的日志、错误、token 用量；失败时根据 stderr 修环境或 prompt。
- **Approvals**：若有 hire_agent 或 strategy 审批，在 UI 处理。
- **Budget**：成本接近上限时提前加预算或暂停次要 Agent。

若 Agent 行为不对（例如不 checkout 就改代码、或一次占很多任务）：
- 改 **promptTemplate**，强调「一次一个、必须 checkout、必须用 API 更新状态」；
- 必要时 **Reset session** 清空会话再跑；
- 或暂时 **Pause**，修好配置再 **Resume**。

### 3.6 第六步：收尾与发布

- 当大部分 Issues 被标为 **done**，剩余的可由 Board 自己关或再 assign 给 Agent 收尾。
- 发布、上线、监控可单独再建 Issue 或新项目，同一套流程：Goal → Project → Issues → Assign → 心跳持续跑。

---

## 4. 保证“正确运行”的检查清单

- [ ] **目标与项目**：公司下至少有一个 Goal，项目挂在 Goal 上；所有 Issue 能追溯到该 Goal（通过 project 或 goal_id/parent_id）。
- [ ] **任务拆分**：每个 Issue 描述清晰、可验收；大功能拆成多个 Issue，避免单 Issue 过大一次做不完。
- [ ] **Agent 身份与权限**：API Key 有效，且该 Agent 有权限读/写自己公司下的 issues 和 comments。
- [ ] **Prompt**：明确写「先取分配给我的任务 → checkout 一个 → 做 → 更新状态/评论 → 再取下一个」；强调只用 Paperclip API 更新任务，不要脱离系统改完就不管。
- [ ] **cwd / 工作空间**：指向同一代码库，多次心跳在同一目录上累积。
- [ ] **心跳策略**：`wakeOnAssignment` 开，`intervalSec` 设成合理间隔；Run 失败时能自动回到 idle 以便下次重试。
- [ ] **超时与预算**：`timeoutSec` 足够单次任务完成，但不要无限大；设预算防止跑飞。
- [ ] **人工复核**：关键 Issue 可先到 in_review，Board 检查后再标 done；必要时用 Approval 控制大变更。

---

## 5. 小结

- **持续运行**：靠心跳策略（定时 + 分配唤醒）+ 会话延续 + 超时/预算约束。
- **完成项目所有任务**：把项目拆成 Issues，assign 给 Agent，用 prompt 约束 Agent「每次取一个任务→做→更新→再取」，单 Agent 串行或多 Agent 分工均可。
- **正确运行**：靠原子 checkout、状态机、预算、审批 + 清晰的 prompt 与任务拆分 + Board 监控与人工干预。

按上述流程，从「派发：开发一个英文学习应用」到真正跑通整项目开发，是可行且可重复的；你可以先用小项目（5～10 个 Issue）跑通一版，再放大到更大项目或更多 Agent。
