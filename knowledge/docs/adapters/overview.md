 ---
 title: 适配器总览（Adapters Overview）
 summary: 适配器的作用以及它们如何连接 agents 与 Paperclip
 ---

 适配器是 Paperclip 编排层与 agent 运行时之间的桥梁。每个适配器都知道如何调用某一类 AI agent，并采集它的运行结果。

 ## 适配器如何工作（How Adapters Work）

 当一次心跳被触发时，Paperclip 会：

 1. 查找该 agent 的 `adapterType` 与 `adapterConfig`
 2. 调用适配器的 `execute()` 函数，并传入执行上下文
 3. 由适配器去启动或调用实际的 agent 运行时
 4. 适配器捕获 stdout，解析用量/成本数据，并返回结构化结果

 ## 内置适配器（Built-in Adapters）

 | 适配器 | 类型键 | 描述 |
 |--------|--------|------|
 | [Claude Local](/adapters/claude-local) | `claude_local` | 在本机运行 Claude Code CLI |
 | [Codex Local](/adapters/codex-local) | `codex_local` | 在本机运行 OpenAI Codex CLI |
 | [Process](/adapters/process) | `process` | 执行任意 shell 命令 |
 | [HTTP](/adapters/http) | `http` | 通过 webhook 调用外部 agent 服务 |

 ## 适配器架构（Adapter Architecture）

 每个适配器都是一个包含三个模块的包：

 ```
 packages/adapters/<name>/
   src/
     index.ts            # 共享元数据（类型、标签、可用模型）
     server/
       execute.ts        # 核心执行逻辑
       parse.ts          # 输出解析
       test.ts           # 环境诊断
     ui/
       parse-stdout.ts   # stdout -> 运行视图转录记录
       build-config.ts   # 表单值 -> adapterConfig JSON
     cli/
       format-event.ts   # 为 `paperclipai run --watch` 格式化终端输出
 ```

 三个注册表会消费这些模块：

 | 注册表 | 作用 |
 |--------|------|
 | **Server** | 执行 agents 并采集结果 |
 | **UI** | 渲染运行转录、提供配置表单 |
 | **CLI** | 为实时观察提供终端输出格式 |

 ## 如何选择适配器（Choosing an Adapter）

 - **需要代码型 agent？** 使用 `claude_local` 或 `codex_local`
 - **需要运行脚本或命令？** 使用 `process`
 - **需要调用外部服务？** 使用 `http`
 - **需要自定义行为？** 可以[创建自己的适配器](/adapters/creating-an-adapter)
