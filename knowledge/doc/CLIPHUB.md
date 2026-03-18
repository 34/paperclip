# ClipHub — 公司注册表（The Company Registry）

**下载一家公司。**

ClipHub 是公开注册表，用于分享、发现和下载 Paperclip 公司配置。公司模板是一种可移植制品，包含完整组织——agents、汇报结构、适配器配置、角色定义、种子任务——一条命令即可启动。

---

## 是什么（What It Is）

ClipHub 之于 Paperclip，如同包注册表之于编程语言。Paperclip 已支持可导出的组织配置（见 [SPEC.md](./SPEC.md) §2）。ClipHub 是存放这些导出的公共目录。

用户在 Paperclip 中搭建好一家公司——开发工作室、营销代理、研究实验室、内容工作室——导出为模板并发布到 ClipHub。任何人都可以浏览、搜索、下载并在自己的 Paperclip 实例上启动该公司。

口号：**你可以直接下载一家公司。**

---

## 发布内容（What Gets Published）

ClipHub 包即**公司模板导出**——Paperclip 规范中定义的可移植制品格式，包含：

| 组件 | 描述 |
|---|---|
| **公司元数据** | 名称、描述、适用场景、分类 |
| **组织图** | 完整汇报层级 |
| **Agent 定义** | 每个 agent：名称、角色、职位、能力描述 |
| **适配器配置** | 每个 agent 的适配器类型与配置 |
| **种子任务** | 可选的启动任务与计划，用于公司首次运行 |
| **预算默认值** | 建议的 agent/公司 token 与成本预算 |

模板是**结构而非状态**。不含进行中任务、历史成本数据或运行时产物，只是蓝图。

### 子包（Sub-packages）

并非所有场景都需要整家公司。ClipHub 也支持发布单独组件：**Agent 模板**、**团队模板**、**适配器配置**。可混入现有公司使用。

---

## 核心功能（Core Features）

### 浏览与发现（Browse & Discover）

首页从多个维度展示公司：Featured、Popular、Recent、Categories。每条展示名称、简介、组织规模、分类、适配器类型、星标与下载数、迷你组织图预览。

### 搜索（Search）

支持**语义搜索**（基于向量嵌入），可按意图搜索；并支持按分类、agent 数量、适配器类型、星标、时间筛选。

### 公司详情页（Company Detail Page）

展示完整描述、可交互组织图、Agent 列表、种子任务、预算概览、安装命令、版本历史、社区数据（星标、评论、fork）。

### 安装与 Fork（Install & Fork）

**安装**：`paperclip install cliphub:<publisher>/<company-slug>`，下载模板并在本地实例创建新公司。**Fork**：复制到你的 ClipHub 账号，可修改后作为自己的变体发布，fork  lineage 可追踪。

### 星标与评论（Stars & Comments）

星标用于收藏与质量信号；评论为每条列表下的讨论。

---

## 发布（Publishing）

拥有 GitHub 账号即可通过 GitHub OAuth 发布。从 Paperclip 导出公司为模板后发布：`paperclip export --template my-company`，`paperclip publish cliphub my-company`。或通过 Web UI 直接上传模板导出。

发布时需提供：slug、name、description、category、tags、version（semver）、changelog、readme、license。模板使用语义化版本，每次发布为不可变版本。

---

## 分类（Categories）

按使用场景组织：软件开发、营销与增长、内容与媒体、研究与分析、运营、销售、财务与法务、创意、通用等。分类非互斥，可设主分类加标签。

---

## 审核与信任（Moderation & Trust）

**认证发布者**：达到一定门槛可获得认证徽章，认证模板在搜索中排名更高。**安全审查**：自动化扫描适配器配置、社区举报、人工审核。**账号门槛**：新账号需等待期才能发布。

---

## 架构（Architecture）

ClipHub 是**独立于 Paperclip 的服务**。Paperclip 自托管；ClipHub 为托管注册表，Paperclip 实例与之通信。集成层：ClipHub Web、ClipHub API、Paperclip CLI（`paperclipai install`、`publish`、`cliphub sync`）、Paperclip UI 中的「Browse ClipHub」面板。技术栈：React + Vite、TypeScript + Hono、PostgreSQL、向量搜索、GitHub OAuth、对象存储。

---

## 用户流程（User Flows）

「我想开一家公司」：在 ClipHub 浏览或搜索 → 选模板 → 运行 `paperclipai install cliphub:acme/lean-saas-shop` → 配置 API key 与预算 → 启动。「我做了好东西想分享」：在 Paperclip 中搭建并导出 → 发布到 ClipHub。「我想改进别人的公司」：Fork → 本地修改 → 再发布。「我只要一个 agent」：搜索 agent 模板 → `paperclipai install cliphub:acme/senior-python-eng --agent` → 分配到现有公司的上级。

---

## 与 Paperclip 的关系（Relationship to Paperclip）

使用 Paperclip 不依赖 ClipHub；可从零搭建公司。但 ClipHub 显著降低门槛：新用户几分钟即可获得可运行公司；老用户可分享成熟配置；生态形成正循环。

---

## V1 范围（V1 Scope）

**必须有**：模板发布与浏览、详情页、语义搜索、`paperclipai install cliphub:<publisher>/<slug>`、GitHub OAuth、星标、下载数、版本与版本历史、基础审核（举报、自动隐藏）。**V2**：评论、Fork 与 lineage、Agent/团队子包、认证发布者徽章、适配器配置安全扫描、Paperclip UI 内 ClipHub 面板、`cliphub sync`、发布者主页。**不在范围**：付费模板、私有注册表、在 ClipHub 上运行公司。
