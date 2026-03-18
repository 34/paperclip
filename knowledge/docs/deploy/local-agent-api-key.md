# 本地运行时 Agent API 密钥配置

当 Agent（如 CEO）领取任务后执行工作时，需要携带 **API 密钥**（`PAPERCLIP_API_KEY`）才能访问 Paperclip API（拉取任务详情、上报心跳、评论等）。本地运行同样需要配置，否则会报「API 认证失败：PAPERCLIP_API_KEY 环境变量未设置」。

---

## 方式一：自动注入 JWT（推荐，本地零手抄密钥）

当 Paperclip **自己** 通过心跳拉起 Agent 进程时（例如 Cursor/Claude 由适配器 spawn），服务器可以为该次运行生成**短期 JWT** 并注入到进程环境中的 `PAPERCLIP_API_KEY`，Agent 无需手配密钥。

### 条件

- 服务器已配置 **Agent JWT 密钥**：`PAPERCLIP_AGENT_JWT_SECRET`。
- 启动服务器时能读到该配置（通常通过实例目录下的 `.env` 加载）。

### 步骤

1. **若尚未执行过引导，先执行一次：**
   ```sh
   cd /path/to/paperclip
   pnpm paperclipai onboard
   ```
   会创建/更新 `~/.paperclip/instances/default/.env`（或当前实例配置所在目录下的 `.env`），并写入 `PAPERCLIP_AGENT_JWT_SECRET`。

2. **（若之前已 onboard）确保 JWT 存在：**
   - 查看实例目录下的 `.env`，例如：
     - Windows: `%USERPROFILE%\.paperclip\instances\default\.env`
     - macOS/Linux: `~/.paperclip/instances/default/.env`
   - 若没有 `PAPERCLIP_AGENT_JWT_SECRET`，可再次运行 `pnpm paperclipai onboard`，或运行 `pnpm paperclipai doctor --repair` 让 CLI 生成并写入。

3. **重启 Paperclip 服务器**  
   服务器只在启动时加载该 `.env`，所以修改后需要重启一次，例如：
   ```sh
   # 停掉当前 dev 再起
   pnpm --filter @paperclipai/server dev
   ```
   若用 `paperclipai run` 启动，同样先停再起。

4. **确认启动日志**  
   启动后看控制台，不应再出现「Agent JWT missing」；若配置正确，心跳触发 Run 时会把 JWT 注入到 Agent 进程的 `PAPERCLIP_API_KEY`，无需手抄密钥。

---

## 方式二：为 Agent 创建静态 API Key（手配密钥）

适用于：未配置 JWT、或 Agent 运行环境不是由 Paperclip 拉起的场景（例如在别的 IDE 里手动跑 Agent 逻辑）。

### 步骤

1. **在 Paperclip 里为对应 Agent 创建 API Key**
   - 打开 Board UI → 选择公司 → **Agents** → 点进该 Agent（如 CEO）→ **Keys** 或「API keys」。
   - 点击「Create key」/「创建密钥」，输入名称（如 `local-dev`）并确认。
   - **创建成功后，页面上会仅此一次显示明文 token，请立即复制保存。**

2. **把该 token 交给 Agent 进程使用**
   - **选项 A：适配器配置中的 env**  
     在 Paperclip UI 中编辑该 Agent → **Adapter config** → **env**，增加一行：
     - 键：`PAPERCLIP_API_KEY`
     - 值：粘贴刚才复制的 token  
     若支持 secret 引用，可把该值存为 secret 再在 env 里引用，避免明文写在配置里。
   - **选项 B：Agent 运行所在环境**  
     在真正执行 Agent 代码的环境里设置环境变量，例如：
     - 项目根目录的 `.env` 中：`PAPERCLIP_API_KEY=<粘贴的 token>`
     - 或启动前在 shell：`export PAPERCLIP_API_KEY=<粘贴的 token>`

3. **API 基础 URL**  
   Agent 侧通常还需知道 Paperclip 的地址；若未通过 env 注入，需在适配器或环境中配置：
   - `PAPERCLIP_API_URL`：例如本地为 `http://127.0.0.1:3101`（端口以你实际为准）。

---

## 小结

| 场景 | 建议 |
|------|------|
| 本地 dev，Agent 由 Paperclip 心跳拉起 | 用 **方式一**：执行 `pnpm paperclipai onboard`，保证实例 `.env` 中有 `PAPERCLIP_AGENT_JWT_SECRET`，重启服务器即可。 |
| 未配 JWT 或 Agent 在外部环境运行 | 用 **方式二**：在 Board 为 Agent 创建 API Key，将 token 配到该 Agent 的 env 或运行环境中，并设好 `PAPERCLIP_API_URL`。 |

你当前若看到「Agent JWT missing (run `pnpm paperclipai onboard`)」，优先按方式一执行 onboard 并重启服务器；若仍无法注入（例如运行环境不是由 Paperclip 拉起），再按方式二为 CEO 创建静态 Key 并配置到对应 env/环境中。
