 ---
 title: 创建适配器（Creating an Adapter）
 summary: 构建自定义适配器的指南
 ---

 通过构建自定义适配器，可以将 Paperclip 连接到任意 agent 运行时。

 <Tip>
 如果你在用 Claude Code，可以使用 `create-agent-adapter` 技能，它会以交互方式引导你完成整个适配器构建流程。只需让 Claude 创建一个新适配器，它会逐步带你完成每一步。
 </Tip>

 ## 包结构（Package Structure）

 ```text
 packages/adapters/<name>/
   package.json
   tsconfig.json
   src/
     index.ts            # 共享元数据
     server/
       index.ts          # 服务端导出
       execute.ts        # 核心执行逻辑
       parse.ts          # 输出解析
       test.ts           # 环境诊断
     ui/
       index.ts          # UI 导出
       parse-stdout.ts   # 运行转录解析器
       build-config.ts   # 配置构建器
     cli/
       index.ts          # CLI 导出
       format-event.ts   # 终端输出格式化
 ```

 ## 第 1 步：根级元数据（Root Metadata）

 `src/index.ts` 会被三个消费者同时引入，应保持无额外依赖。

 ```ts
 export const type = "my_agent";        // snake_case，需全局唯一
 export const label = "My Agent (local)";
 export const models = [
   { id: "model-a", label: "Model A" },
 ];
 export const agentConfigurationDoc = `# my_agent configuration
 Use when: ...
 Don't use when: ...
 Core fields: ...
 `;
 ```

 ## 第 2 步：服务端执行逻辑（Server Execute）

 `src/server/execute.ts` 是核心实现。它接收 `AdapterExecutionContext`，返回 `AdapterExecutionResult`。

 关键职责：

 1. 使用安全的 helper（如 `asString`、`asNumber` 等）读取配置
 2. 使用 `buildPaperclipEnv(agent)` 加上上下文变量构建环境
 3. 从 `runtime.sessionParams` 中解析会话状态
 4. 使用 `renderTemplate(template, data)` 渲染提示词
 5. 通过 `runChildProcess()` 启动子进程，或用 `fetch()` 调用远端服务
 6. 解析输出中的用量、成本、会话状态与错误
 7. 处理未知会话错误（重新以全新会话重试，并设置 `clearSession: true`）

 ## 第 3 步：环境测试（Environment Test）

 `src/server/test.ts` 用于在真正运行前验证适配器配置。

 返回结构化诊断信息：

 - 使用 `error` 表示无效/不可用的配置
 - 使用 `warn` 表示非阻塞问题
 - 使用 `info` 表示通过的检查

 ## 第 4 步：UI 模块（UI Module）

 - `parse-stdout.ts` —— 将 stdout 行转换为运行视图所需的 `TranscriptEntry[]`
 - `build-config.ts` —— 将表单值转换为 `adapterConfig` JSON
 - 在 `ui/src/adapters/<name>/config-fields.tsx` 中实现配置字段的 React 组件

 ## 第 5 步：CLI 模块（CLI Module）

 `format-event.ts` —— 使用 `picocolors` 美化 `paperclipai run --watch` 的终端输出。

 ## 第 6 步：在各注册表中注册（Register）

 将适配器注册到三个注册表中：

 1. `server/src/adapters/registry.ts`
 2. `ui/src/adapters/registry.ts`
 3. `cli/src/adapters/registry.ts`

 ## 技能注入（Skills Injection）

 在不写入 agent 工作目录的前提下，让 Paperclip 技能对运行时可见：

 1. **最佳方式：临时目录 + 标志位** —— 创建临时目录、将技能通过符号链接挂载进去，通过 CLI 标志传入路径，并在结束后清理
 2. **可接受方式：全局配置目录** —— 将技能挂载到运行时的全局插件目录
 3. **可接受方式：环境变量** —— 使用环境变量指向仓库中的 `skills/` 目录
 4. **最后手段：提示词注入** —— 在提示模板中直接嵌入技能内容

 ## 安全性（Security）

 - 将 agent 输出视为不可信数据（解析时要防御性处理，绝不直接执行）
 - 通过环境变量而不是提示词注入 secrets
 - 如运行时支持网络控制，务必配置访问策略
 - 始终启用超时和宽限期设置
