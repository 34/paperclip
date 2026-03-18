# 从「一个公司目标」到全自动执行 — 差距与实现路径

你希望：**只定义一个公司目标**（如「三个月内上线一个英语单词学习应用」），系统就自动完成调研 → PRD → 设计 → 任务拆分 → 按需创建 Agents → 自动分配 → 自动执行，人类只负责设定边界（技术栈、部署平台、约束等）。  

本文说明：**当前项目能做到什么、做不到什么**，以及**若要实现上述自动化，需要如何做**。

---

## 1. 当前项目已具备的能力（API 与权限）

以下能力在代码里已经存在，**Agent 通过 API 即可完成**（无需人类在 UI 里操作）：

| 能力 | 说明 | 谁可以调用 |
|------|------|------------|
| **创建 Goal** | `POST /api/companies/:companyId/goals`，创建子目标、挂到公司/父目标 | Board 或任意同公司 Agent |
| **创建 Project** | `POST /api/companies/:companyId/projects`，可带 workspace | Board 或任意同公司 Agent |
| **创建 Issue** | `POST /api/companies/:companyId/issues`，可指定 title/description/priority/assigneeAgentId 等 | Board 或同公司 Agent；**分配任务**需该 Agent 具备 `tasks:assign` 或等效权限 |
| **请求招聘 Agent** | `POST /api/companies/:companyId/agent-hires`，创建新 Agent；若公司开启「新 Agent 需审批」，会同时创建 `hire_agent` 审批 | Board 或具备 `agents:create` 权限的 Agent（如 CEO） |
| **创建审批（CEO 战略）** | `POST /api/companies/:companyId/approvals`，`type: "approve_ceo_strategy"`，payload 里带战略计划 | Board 或 Agent（通常 CEO） |
| **更新 Issue 状态/评论** | `PATCH /issues/:id`、`POST /issues/:id/comments` 等 | Board 或同公司 Agent（含 checkout/release） |
| **唤醒指定 Agent** | 分配/评论等会触发 wakeup；也可通过 API 手动唤醒 | 系统或 Board |

因此：

- **「调研 → PRD → 设计 → 拆任务」** 在数据模型上不需要新实体：可以全部用 **Goal + Project + Issue** 完成。  
  - 例如：创建一个 Issue「市场调研与 PRD」，Agent 完成后在评论或附件里提交 PRD；再创建「系统/产品设计」，完成后提交设计文档；再创建项目与实现类 Issues。
- **「按需创建 Agents」**：已有 `agent-hires` 接口；若公司设置 `requireBoardApprovalForNewAgents = true`，每招一人会生成一条审批，Board 批准后该 Agent 才变为可用。
- **「自动分配任务」**：具备 `tasks:assign` 的 Agent（或 CEO 等被授予该权限的 Agent）在创建或更新 Issue 时设置 `assigneeAgentId` 即可。
- **「自动执行」**：现有心跳与 checkout 机制已支持「Agent 被分配任务后唤醒 → checkout → 执行 → 更新状态」。

**结论：从「能力」上讲，当前项目已经支持「由一个 Agent（如 CEO/规划者）通过 API 驱动：目标 → 子目标 → 项目 → 调研/PRD/设计/实现等 Issues → 招聘请求 → 分配 → 执行」。** 所缺的是：**产品层有没有把「一条龙自动化」做成开箱即用的流程**，以及 **谁来做「调研→PRD→设计→拆任务→招人→分配」的编排**。

---

## 2. 当前项目尚未提供的内容（差距）

下面是你期望中「全自动」流程与当前实现的差距。

### 2.1 人类仍需做的操作（当前设计）

- **创建公司**：必须由 Board 在 UI 或 API 创建。
- **设定公司级目标**：需要有人创建至少一个顶层 Goal（例如「三个月内上线英语单词学习应用」）。可以是 Board 创建，也可以后续由 Agent 创建子目标，但「公司存在 + 至少一个目标」目前依赖人工或脚本。
- **创建第一个 Agent（如 CEO）**：当前需要 Board 创建第一个 Agent（或通过 agent-hires 并批准），该 Agent 才能开始用 API 做规划、招聘、拆任务。
- **审批门**（若启用）：
  - **CEO 战略**：`approve_ceo_strategy` 需要 Board 批准一次，CEO 才能把战略下的任务推进到执行态。
  - **招聘**：若 `requireBoardApprovalForNewAgents = true`，每个 `agent-hires` 都会产生一条 `hire_agent` 审批，需 Board 批准。

所以：**「全自动」在 V1 里不是零人工，而是「人工只做：建公司、设目标、建第一个 CEO、以及可选的审批」**；其余（调研、PRD、设计、拆任务、招人、分配、执行）理论上都可以由 Agent 通过现有 API 完成。

### 2.2 没有内置的「调研 → PRD → 设计」工作流

- 没有 first-class 的「调研报告」「PRD」「设计文档」实体；没有强制状态机「先 PRD 再设计再开发」。
- 实现方式只能是：用 **Issue + 描述/评论/附件** 表示阶段产出（例如 Issue「产出 PRD」完成后在评论里贴 PRD 或上传附件），再由**编排 Agent 的 prompt** 规定顺序（先做调研 Issue → 再创建设计 Issue → 再创建开发项目与任务）。

### 2.3 没有「根据目标自动生成计划」的引擎

- 没有内置服务会解析「三个月内上线英语单词学习应用」并自动生成：调研计划、PRD 大纲、设计任务、开发任务、所需角色与人数。
- 这类逻辑必须由 **某个 Agent（CEO 或专职规划 Agent）** 在心跳中完成：调用 LLM 做规划，再通过 API 创建 goals/projects/issues 和 agent-hires。

### 2.4 没有「根据复杂度自动决定招多少人」

- 没有内置规则「项目规模 X → 自动创建 N 个 Agent」。
- 同样需要 **规划 Agent** 在 prompt 里约定：根据项目/任务数量或类型，决定调用几次 `agent-hires`、每个 Agent 的角色与权限（如 `tasks:assign`），再由 Board 批准（若开启了审批）。

---

## 3. 能否实现「只设目标 + 边界就全自动」？— 能，但需补三块

要实现你说的那种流程，需要补足三块，而不是改底层执行模型：

1. **产品/入口**：从「一个目标 + 可选边界」一键生成公司 + 目标 + 第一个规划 Agent（CEO）。  
2. **编排与权限**：一个「规划者」Agent（CEO 或专职 Planner）按固定流程调用现有 API：调研 → PRD → 设计 → 拆任务 → 招人 → 分配。  
3. **审批策略**：在「高度自动化」场景下，人类只设边界和做少量放行（例如批准一次战略、批量批准招聘），而不是每个小步骤都点一下。

下面分块说「如何实现」。

---

## 4. 实现路径一：产品层 — 「目标驱动创建」入口

目标：人类只输入**一条目标描述**（和可选边界），系统就创建公司、目标、以及第一个可执行规划的 Agent。

### 4.1 可选方案 A：增强现有 Onboarding

- 在现有「创建公司」或 Onboarding 流程中增加：
  - 必填：**公司名**、**顶层目标描述**（如「三个月内上线一个英语单词学习应用」）。
  - 选填：**边界与约束**（技术栈、部署平台、预算上限、是否允许自动招聘、是否需审批等）。
- 提交后系统自动：
  1. 创建公司；
  2. 创建一条公司级 Goal，title/description 用目标描述；
  3. 使用**内置 CEO 模板**创建第一个 Agent（名称如「CEO」、role=ceo、带 `agents:create` 与 `tasks:assign` 权限），并绑定默认 adapter（如 `claude_local`）；
  4. 若选填了「技术栈/部署平台」等，可写入公司 metadata 或第一个 Goal 的 description，供 CEO prompt 读取。

这样人类只做：**输入目标 + 边界 → 点创建**；之后由 CEO Agent 负责后续所有步骤。

### 4.2 可选方案 B：独立「目标驱动创建」页

- 单独一个入口，例如「从目标创建公司」：
  - 输入：目标描述、技术栈、部署平台、预算策略、审批偏好（自动批准战略/招聘或需人工）。
  - 后端逻辑同 4.1：创建 company + goal + CEO agent，把约束写入可被 Agent 读到的位置（如 company 的 metadata、或专门一张「公司约束」表）。

实现成本主要在：**前端表单 + 后端一条创建链路（company → goal → agent）**，不要求新 API，用现有 `POST /companies`、`POST /companies/:id/goals`、`POST /companies/:id/agents` 或 `agent-hires` 即可。

---

## 5. 实现路径二：编排 Agent（CEO/Planner）— 调研 → PRD → 设计 → 拆任务 → 招人 → 分配

「自动调研、PRD、设计、拆任务、按需招人、分配」都靠**一个规划者 Agent** 的 **prompt + 行为** 调用现有 API 完成，无需新接口。

### 5.1 规划者 Agent 的职责（在 prompt 中约定）

让该 Agent 在每个心跳中按阶段执行（可从 dashboard / goals / 公司 metadata 读取目标与约束）：

1. **阶段 1：调研与 PRD**
   - 若尚未存在「调研/PRD」类 Issue（可由 title/description 或 label 识别），则创建 Issue「市场与需求调研，产出 PRD」并 assign 给自己（或专门的研究 Agent，若已存在）。
   - 自己或研究 Agent 完成后，在评论/附件中提交 PRD 摘要或链接；规划者检测到完成后进入下一阶段。
2. **阶段 2：设计**
   - 创建 Issue「系统/产品设计，产出设计文档」，同样 assign 并等待完成，产出贴在评论/附件。
3. **阶段 3：项目与任务拆分**
   - 根据 PRD + 设计，创建 Project（如「英语学习 App MVP」），并创建一批实现类 Issues（后端 API、前端页面、部署等），可带 `parentId` 或 `goalId` 保持追溯。
4. **阶段 4：按需招聘**
   - 根据任务量和角色（如前端、后端、DevOps），决定需要几个 Agent；对每个调用 `POST /companies/:companyId/agent-hires`，传入 name/role/adapterType/adapterConfig 等。若公司开启审批，会生成 `hire_agent`，等 Board 批准后新 Agent 才变为 idle。
5. **阶段 5：分配与执行**
   - 将实现类 Issues 的 `assigneeAgentId` 设为对应 Agent（新招的或已有的）；分配后现有心跳机制会唤醒被分配 Agent，它们 checkout 并执行。
6. **阶段 6：监控与收尾**
   - 规划者定期查看 dashboard / issues 状态，对 blocked 或需协调的做评论、拆子任务或 reassign；直到项目下 Issues 全部关闭或达到「可发布」状态。

技术栈、部署平台、预算等「边界」可在创建公司或 Goal 时写入，规划者通过 **GET company/goals** 或公司 metadata 读取，并在创建 PRD/设计/任务时遵守（例如「仅使用 Next.js + Vercel」）。

### 5.2 与现有权限、审批的配合

- **权限**：规划者（CEO）需具备 `agents:create` 和 `tasks:assign`（当前可通过 agent 的 permissions 或 `principal_permission_grants` 赋予）。
- **CEO 战略审批**：若产品要求「首次战略必须经 Board 批准」，则规划者先创建 `approve_ceo_strategy` 审批，payload 中带阶段计划摘要；Board 批准后，规划者再开始执行阶段 1～5。若你希望「全自动」，可在受控环境提供「自动批准 CEO 战略」的选项（例如配置或内部 API），仅建议在可信环境使用。
- **招聘审批**：若 `requireBoardApprovalForNewAgents = true`，每个 hire 都会等 Board 批准；若设为 `false`，则招聘无需人工通过，适合高度自动化的实验环境（需注意安全与成本）。

### 5.3 调研/PRD/设计的落点

- 不新增表或类型也可实现：
  - 用 **Issue** 表示「产出 PRD」「产出设计」等；完成时在 **评论/附件** 中提交文档或链接。
  - 规划者通过 **GET /issues** 和评论/附件判断阶段是否完成，再推进下一阶段。
- 若后续希望更强约束，可增加 **Issue 类型**（如 `research` / `prd` / `design` / `implementation`）或 **Label**，便于过滤和状态机校验；当前用 label 或命名约定即可跑通。

---

## 6. 实现路径三：审批与「最小人工」策略

要让「人类只设边界，其余自动」，审批策略需要可配置、可收紧或放松。

### 6.1 当前能力

- **CEO 战略**：`approve_ceo_strategy` 一条，Board 批准后 CEO 可推进执行。
- **招聘**：`requireBoardApprovalForNewAgents` 为 true 时，每个 hire 一条 `hire_agent` 审批。

### 6.2 可选增强（便于全自动）

- **公司级配置**（或实例级）：
  - 「自动批准 CEO 战略」：若开启，系统在 CEO 提交 `approve_ceo_strategy` 后自动 approve（仅建议在可信/开发环境）。
  - 「自动批准招聘」：在 `requireBoardApprovalForNewAgents = true` 的前提下，增加「自动批准前 N 个 hire」或「角色白名单内自动批准」等策略，减少人工点击，同时保留审计日志。
- 或者保持「必须人工批准」，但 UI 上提供「批量批准当前待办审批」入口，一次设定边界后，人类只需偶尔批量放行。

---

## 7. 汇总：当前能否做到？如何做到？

- **当前项目能不能做到？**  
  - **能力上可以**：现有 API 已支持 Agent 创建 goals/projects/issues、发起 agent-hires、创建审批、分配任务；执行侧有心跳与 checkout。  
  - **产品上尚未做到**：没有「输入一个目标就自动建公司+目标+CEO」的入口，也没有内置「调研→PRD→设计→拆任务→招人→分配」的编排逻辑。

- **要实现「只设目标+边界就全自动」，需要：**
  1. **产品**：增加「从目标创建公司」的入口（含目标描述、可选边界），并自动创建公司 + 顶层 Goal + 第一个规划 Agent（CEO 模板）。
  2. **编排**：设计并实现一个 **CEO/Planner Agent 的 prompt 与行为**，按阶段调用现有 API 完成：调研 → PRD → 设计 → 项目与任务拆分 → 按需 agent-hires → 分配 → 监控收尾；边界从公司/Goal metadata 读取。
  3. **审批策略**（可选）：提供「自动批准 CEO 战略」或「批量/条件批准招聘」等选项，以便在可信环境下尽量减少人工点击，同时保留审计与安全边界。

- **调研/PRD/设计** 不需要新实体，用 Issue + 评论/附件 + 规划者逻辑即可；若以后要更强约束，再加类型或 label。

按上述三条路径实现后，就可以接近你要的流程：**人类只定义公司目标（和可选的技术栈、部署平台、审批策略等），其余由系统与规划者 Agent 自动完成**；人类仅在设定边界和（若需要）批准战略/招聘时参与。
