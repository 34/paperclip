 ---
 title: 快速开始（Quickstart）
 summary: 几分钟内跑起来 Paperclip
 ---

 在本地几分钟内就能跑起来 Paperclip。

 ## 快速启动（推荐）

 ```sh
 npx paperclipai onboard --yes
 ```

 上述命令会引导你完成初始化配置，设置环境，并启动 Paperclip。

 ## 本地开发（Local Development）

 前置条件：Node.js 20+ 与 pnpm 9+。

 ```sh
 pnpm install
 pnpm dev
 ```

 这会在 [http://localhost:3100](http://localhost:3100) 启动 API 服务与 UI。

 不需要外部数据库——Paperclip 默认使用嵌入式 PostgreSQL 实例。

 ## 一键启动（One-Command Bootstrap）

 ```sh
 pnpm paperclipai run
 ```

 该命令会在缺少配置时自动执行引导，运行健康检查并尝试自动修复，然后启动服务器。

 ## 接下来做什么（What's Next）

 当 Paperclip 运行起来之后：

 1. 在 Web UI 中创建你的第一家公司
 2. 定义公司的目标
 3. 创建一个 CEO agent 并配置其适配器
 4. 继续扩展组织结构，创建更多 agents
 5. 设置预算并分配初始任务
 6. 点击开始——agents 会开始按心跳协议运行，公司开始运转

 <Card title="核心概念" href="/start/core-concepts">
   了解 Paperclip 背后的关键概念
 </Card>
