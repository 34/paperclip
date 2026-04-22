# 用 Paperclip 管理极词(JiCi) — 虚拟软件公司实操探索

## 背景

极词 (JiCi) 是一个 Flutter 英语单词学习应用，面向中国市场，离线优先，FSRS-6 算法驱动。当前处于 Phase 3.5 功能打磨阶段（12 个模块完成了 2 个），目标是最低成本发布运营。

本文档探索如何用 Paperclip 搭建一家虚拟软件公司来接管极词的全部后续工作。

---

## 一、现状盘点

### 已完成
- FSRS-6 记忆算法（Rust FFI）
- 双轨道调度（FSRS + 固定计划）
- 8 种题型引擎
- 111,965 词 / 167 词书，74MB 离线词典
- 主要页面全部实现（6 Tab 壳、首页、词库、收藏、统计、设置、付费墙、主题商店等）
- 50 个测试文件
- Python 数据管道（AI 增强、质量门）

### 未完成（Phase 3.5 打磨）
| 波次 | 模块 | 状态 |
|------|------|------|
| 1 | 启动与引导 | ✅ 完成 |
| 1 | 首页（双模式） | ✅ 完成 |
| 1 | 学习流程 | 🔧 进行中（有大量用户反馈需 UI 重设计） |
| 2 | 词库管理 | ❌ |
| 2 | 收藏系统 | ❌ |
| 2 | 词典查询 | ❌ |
| 3 | 学习统计 | ❌ |
| 3 | 目标系统 | ❌ |
| 3 | 专项训练 | ❌ |
| 4 | 设置 | ❌ |
| 4 | 付费墙 | ❌ |
| 4 | 主题商店 | ❌ |

### 发布前必须解决
- Android release 签名配置
- 支付集成（当前付费墙只有 UI）
- 应用商店素材准备
- 隐私政策 / 用户协议
- 无 CI/CD 流水线

### 关键约束
- **副业项目**，预算极低
- **离线优先**，无后端服务器
- **单人开发**，AI Agent 辅助
- 中国市场，需考虑国内应用商店上架

---

## 二、虚拟公司架构设计

### 组织架构

考虑到副业项目的低成本需求，设计一个精简的 4-Agent 公司：

```
Board（你 — 产品老板）
  │
  └── CEO — 产品经理
        │
        ├── Tech Lead — 技术负责人
        │     负责：Flutter 开发、Bug 修复、性能优化
        │
        ├── QA — 测试工程师
        │     负责：测试、代码审查、质量把关
        │
        └── Designer — UI/UX 设计师
              负责：UI 打磨、交互优化、视觉规范
```

**为什么是 4 个而不是更多**：
- 副业项目预算有限，4 个 Agent 月成本约 $50-120
- 当前阶段主要是打磨和修复，不需要独立角色（如 DevOps、研究员）
- 发布后可以按需扩招（如运营、客服）

### Agent 配置

| Agent | 适配器 | 模型 | 角色说明 |
|-------|--------|------|----------|
| CEO | `claude_local` | Claude Sonnet | 中文沟通，任务分解，进度追踪 |
| Tech Lead | `claude_local` | Claude Sonnet | Flutter/Dart 专家，代码实现 |
| QA | `claude_local` | Claude Sonnet | 测试策略，代码审查 |
| Designer | `gemini_local` | Gemini Pro | UI 设计，交互优化 |

> **成本估算**：Claude Sonnet ~$3/M tokens，4 个 Agent 月均 $50-120。

---

## 三、Paperclip 项目规划

### 项目结构

在 Paperclip 中创建 3 个 Project：

```
公司: JiCi (极词)

Project 1: Phase 3.5 功能打磨
  ├── Goal: 完成 12 模块打磨，达到发布质量
  ├── Wave 1: 学习流程优化 (进行中)
  ├── Wave 2: 内容管理打磨
  ├── Wave 3: 增长系统打磨
  └── Wave 4: 系统与商业化

Project 2: 发布准备
  ├── Goal: 应用商店上架就绪
  ├── 签名与构建配置
  ├── 支付集成
  ├── 应用商店素材
  └── 合规文档

Project 3: 持续运营 (发布后)
  ├── Goal: 用户增长与留存
  ├── Bug 修复
  ├── 用户反馈处理
  ├── 数据管道更新
  └── 版本迭代规划
```

### Issue 拆分策略

以 Phase 3.5 为例，每个模块拆分为标准化的 Issue 组：

```
模块: 学习流程 (features/word_preview + quiz + review_settlement)
  │
  ├── PAP-1: 学习流程 — 新词预览 UI 重设计
  │   优先级: critical
  │   指派: Designer → Tech Lead
  │   描述: 基于用户反馈，重新设计卡片预览为全屏无卡片布局
  │
  ├── PAP-2: 学习流程 — 7 种题型 UI 统一
  │   优先级: high
  │   指派: Tech Lead
  │
  ├── PAP-3: 学习流程 — 答题反馈动画优化
  │   优先级: medium
  │   指派: Designer → Tech Lead
  │
  ├── PAP-4: 学习流程 — 结算页面打磨
  │   优先级: medium
  │   指派: Tech Lead
  │
  └── PAP-5: 学习流程 — 学习中断恢复优化
      优先级: high
      指派: Tech Lead
```

### 目标层级

```
Company Goal: 2026 年 Q2 发布极词 v1.0 到中国应用市场
  │
  ├── Team Goal: Phase 3.5 完成所有模块打磨
  │     ├── Agent Goal (Tech Lead): 完成 12 模块代码实现
  │     ├── Agent Goal (Designer): 完成 UI 设计规范
  │     └── Agent Goal (QA): 确保测试覆盖率 > 80%
  │
  ├── Team Goal: 发布就绪
  │     └── Task: 签名配置、支付集成、商店素材
  │
  └── Team Goal: 上架运营
        └── Task: ASO 优化、用户反馈、版本迭代
```

---

## 四、工作流设计

### 模块打磨工作流（核心流程）

```
Board 创建 Issue（模块拆分的子任务）
  │
  ▼
CEO 分配优先级和指派
  │
  ├─→ Designer: UI 设计 Issue
  │     │
  │     ▼
  │   Designer 完成设计稿（评论附截图/描述）
  │     │
  │     ▼
  │   Board 审核设计 → 通过
  │     │
  │     ▼
  │   CEO 创建实现子任务，指派 Tech Lead
  │
  ├─→ Tech Lead: 直接实现 Issue
  │     │
  │     ▼
  │   Tech Lead 实现并自测
  │     │
  │     ▼
  │   更新 Issue 状态 → in_review
  │     │
  │     ▼
  │   CEO 创建审查任务，指派 QA
  │
  └─→ QA: 代码审查 Issue
        │
        ▼
      QA 审查 + 测试
        │
        ├─ 通过 → Issue → done
        └─ 驳回 → 评论反馈 → Issue 回到 in_progress
```

### Bug 修复工作流

```
Board 或 CEO 创建 Bug Issue
  → 标记 priority: critical/high/medium
  → 指派 Tech Lead
  → Tech Lead 修复
  → QA 验证
  → 关闭
```

### 发布流程

```
CEO 创建发布 Issue（包含 checklist）
  → Tech Lead 执行构建
  → QA 回归测试
  → Board 最终审批（Approval）
  → 发布
```

---

## 五、自动化配置（Routine）

### 推荐设置的 Routine

| Routine | 触发 | 执行者 | 目的 |
|---------|------|--------|------|
| 每日进度检查 | Cron: 每天 9:00 CST | CEO | 扫描所有 in_progress Issue，报告进度 |
| 代码质量扫描 | Cron: 每周一 10:00 | QA | 运行 flutter analyze，报告警告 |
| TODO 清理 | Cron: 每月 1 日 | Tech Lead | 扫描代码中 TODO/FIXME，创建 Issue |
| 发布准备检查 | Cron: 每两周 | CEO | 检查发布前 checklist 完成度 |
| 依赖更新检查 | Cron: 每月 | Tech Lead | 检查 Flutter/Dart 依赖更新 |

### Routine 变量示例

```yaml
# 每日进度检查
routine: daily-progress-check
trigger:
  kind: schedule
  cronExpression: "0 1 * * *"  # UTC 1:00 = CST 9:00
  timezone: Asia/Shanghai
assignee: ceo
variables:
  - name: reportFormat
    type: select
    options: [summary, detailed]
    default: summary
concurrencyPolicy: skip_if_active
```

---

## 六、预算策略

### 分级预算

```
公司月度预算: $100（起步阶段）

Agent 级预算:
  ├── CEO:     $15/月  (少量心跳，主要是任务管理)
  ├── Tech Lead: $50/月  (大量代码生成，最高消耗)
  ├── QA:      $20/月  (代码审查 + 测试)
  └── Designer: $15/月  (设计输出)

预算策略:
  ├── warnPercent: 80%  ($80 时警告)
  └── hardStopEnabled: true  ($100 时自动暂停)
```

### 成本控制技巧

1. **减少 CEO 心跳频率**：CEO 只在需要时触发，不要设定时心跳
2. **Tech Lead 用 Sonnet 而非 Opus**：代码质量足够好，成本低 5x
3. **QA 批量审查**：积累几个 Issue 一起审，减少心跳次数
4. **Designer 用 Gemini**：视觉设计任务 Gemini 性价比更高
5. **使用 FSRS 会话保持**：避免重复加载上下文，减少 token 消耗

---

## 七、执行工作区配置

### 工作区策略

```
策略: project_primary（使用项目主目录）
工作目录: /Users/wzh/code/test/jici

原因:
  - 单人项目，不需要 git worktree 隔离
  - 所有 Agent 共享同一代码库
  - 减少工作区管理的复杂度
```

如果后续多 Agent 并行开发不同模块，可以切换到 `git_worktree` 策略避免冲突。

### Agent 指令包

每个 Agent 的指令（AGENTS.md）应包含：

**CEO**:
```
你是极词（JiCi）的产品经理兼 CEO。
- 产品：Flutter 英语单词学习应用，中国市场，离线优先
- 当前阶段：Phase 3.5 功能打磨
- 你的职责：任务分解、优先级排序、进度追踪、Board 沟通
- 关键规则：绝不自己写代码，所有技术任务委派给 Tech Lead
```

**Tech Lead**:
```
你是极词（JiCi）的技术负责人。
- 技术栈：Flutter 3.10+, Dart, Provider, SQLite, FSRS-6 (Rust FFI)
- 架构：Feature-first 模块化
- 代码规范：空安全，PascalCase 类型，snake_case 文件
- 工作目录：/Users/wzh/code/test/jici/app
- 运行命令：cd app && flutter test / flutter analyze
- 语言：代码英文，注释中文，提交信息中文
```

**QA**:
```
你是极词（JiCi）的测试工程师。
- 测试框架：flutter_test, mocktail
- 质量标准：无 lint 警告，核心路径测试覆盖
- 审查重点：边界情况、空状态、错误状态、加载状态
- 提交前必须通过：flutter test + flutter analyze
```

**Designer**:
```
你是极词（JiCi）的 UI/UX 设计师。
- 设计理念：「禅意赛博」美学，面向严肃学习者
- 主题系统：16 套 Material 3 主题
- 字体：SpaceGrotesk（英文）+ NotoSansSC（中文）
- 交互标准：60fps 动画，点击响应 < 100ms
- 交付物：设计描述 + 交互说明（通过 Issue 评论）
```

---

## 八、分步搭建指南

### Step 1: 启动 Paperclip

```bash
cd /Users/wzh/code/test/paperclip
pnpm dev
# 访问 http://localhost:3100
```

### Step 2: 创建公司

通过 UI 或 CLI：
```bash
npx paperclipai client company create \
  --name "JiCi Tech" \
  --description "极词 — 英语单词学习应用开发公司" \
  --issue-prefix "JICI"
```

### Step 3: 招聘 Agent

```bash
# CEO
npx paperclipai client agent create \
  --name "CEO" \
  --role ceo \
  --title "产品经理" \
  --adapter-type claude_local \
  --budget-monthly-cents 1500

# Tech Lead
npx paperclipai client agent create \
  --name "Tech Lead" \
  --role engineer \
  --title "技术负责人" \
  --reports-to CEO \
  --adapter-type claude_local \
  --budget-monthly-cents 5000

# QA
npx paperclipai client agent create \
  --name "QA" \
  --role qa \
  --title "测试工程师" \
  --reports-to CEO \
  --adapter-type claude_local \
  --budget-monthly-cents 2000

# Designer
npx paperclipai client agent create \
  --name "Designer" \
  --role designer \
  --title "UI/UX 设计师" \
  --reports-to CEO \
  --adapter-type gemini_local \
  --budget-monthly-cents 1500
```

### Step 4: 创建项目和目标

```bash
# Project: Phase 3.5 打磨
# Project: 发布准备
# Project: 持续运营

# 然后在 UI 中创建 Goal 层级
```

### Step 5: 导入当前工作

将 Phase 3.5 的 12 个模块转为 Issue：

```
JICI-1: [Wave1] 学习流程 — 新词预览 UI 重设计 (critical)
JICI-2: [Wave1] 学习流程 — 7 种题型 UI 统一 (high)
JICI-3: [Wave1] 学习流程 — 答题反馈动画 (medium)
JICI-4: [Wave1] 学习流程 — 结算页面打磨 (medium)
JICI-5: [Wave1] 学习流程 — 学习中断恢复 (high)
JICI-6: [Wave2] 词库管理打磨 (medium)
JICI-7: [Wave2] 收藏系统打磨 (medium)
JICI-8: [Wave2] 词典查询打磨 (medium)
JICI-9: [Wave3] 学习统计打磨 (low)
JICI-10: [Wave3] 目标系统打磨 (low)
JICI-11: [Wave3] 专项训练打磨 (low)
JICI-12: [Wave4] 设置打磨 (low)
JICI-13: [Wave4] 付费墙实现 — 支付集成 (high)
JICI-14: [Wave4] 主题商店打磨 (low)
JICI-15: [发布] Android release 签名配置 (high)
JICI-16: [发布] 应用商店素材准备 (medium)
JICI-17: [发布] 隐私政策与用户协议 (high)
```

### Step 6: 配置 Routine

在 UI 中为每个 Routine 创建 schedule 触发器。

### Step 7: 开始工作

1. Board 创建第一个 Issue → 指派 Designer
2. 触发 Designer 心跳 → Designer 提交设计方案
3. Board 审核通过 → CEO 创建实现任务 → 指派 Tech Lead
4. Tech Lead 实现 → QA 审查 → 完成

---

## 九、风险与注意事项

### 实际操作中的挑战

| 挑战 | 应对策略 |
|------|----------|
| Agent 不熟悉 Flutter/Dart | 在指令包中包含完整的技术上下文和代码规范 |
| 心跳成本可能超出预期 | 先用小额预算试跑一周，观察实际消耗 |
| 4 个 Agent 同时改同一文件 | 使用 Issue 级别的工作区隔离或顺序执行 |
| Agent 产出的代码质量不稳定 | QA 严格审查 + Board 抽检 |
| 发布流程（签名/商店）需人工操作 | Paperclip 管理任务 checklist，人工执行关键步骤 |

### 不建议交给 Agent 的事情

- **应用商店发布**：需要人工操作开发者账号
- **支付集成密钥**：敏感信息，不宜 Agent 处理
- **最终设计决策**：Board 亲自审核
- **版本号管理**：人工控制
- **用户数据相关**：隐私敏感

### 建议保持人工控制的环节

```
Board（你）必须亲自参与:
  ├── 设计方案审核 ← 每次都要看
  ├── 发布审批 ← Approval 工作流
  ├── 预算调整 ← 成本超 $80 时介入
  └── 关键 Bug 确认 ← priority: critical 的 Bug
```

---

## 十、后续扩展方向

发布成功后可以按需扩招：

```
发布后可能的扩展:
  ├── 运营 Agent（Cron 定期分析用户反馈）
  ├── ASO 优化 Agent（应用商店优化）
  ├── 客服 Agent（处理用户 Issue）
  ├── 数据管道 Agent（词典更新）
  └── 内容 Agent（新词书制作）
```

---

## 十一、快速验证计划

在全面搭建之前，建议先用最小配置验证可行性：

```
第 1 天: 启动 Paperclip，创建公司 + 1 个 Agent (CEO)
第 2 天: 招聘 Tech Lead，创建 2-3 个 Issue 测试流程
第 3 天: 招聘 QA，测试审查流程
第 4 天: 配置 Routine，观察自动化效果
第 5 天: 评估成本和效率，决定是否继续

验证指标:
  ✓ Agent 能正确理解任务并产出代码
  ✓ Issue 工作流顺畅（创建→指派→执行→审查→完成）
  ✓ 日均成本 < $5
  ✓ 每天至少完成 1 个 Issue
```

这是最小成本验证方案，5 天内就能判断 Paperclip 是否适合管理极词的开发。
