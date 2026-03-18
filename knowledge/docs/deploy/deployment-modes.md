 ---
 title: 部署模式（Deployment Modes）
 summary: local_trusted 与 authenticated（private/public）
 ---

 Paperclip 提供两种运行模式，对应不同的安全特性。

 ## `local_trusted`

 默认模式，针对单人本地使用进行了优化。

 - **主机绑定**：仅绑定到回环地址（localhost）
 - **认证**：无需登录
 - **适用场景**：本地开发、个人实验
 - **董事会身份**：自动创建的本地 board 用户

 ```sh
 # 在 Onboard 中设置
 pnpm paperclipai onboard
 # 选择 "local_trusted"
 ```

 ## `authenticated`

 需要登录认证，支持两种暴露策略。

 ### `authenticated` + `private`

 供私有网络访问（Tailscale、VPN、局域网）。

 - **认证**：通过 Better Auth 登录
 - **URL 处理**：自动 Base URL 模式（配置更简单）
 - **主机信任**：需要配置受信任主机策略

 ```sh
 pnpm paperclipai onboard
 # 选择 "authenticated" -> "private"
 ```

 允许自定义 Tailscale 主机名：

 ```sh
 pnpm paperclipai allowed-hostname my-machine
 ```

 ### `authenticated` + `public`

 面向互联网公开部署。

 - **认证**：需要登录
 - **URL**：必须配置明确的公网 URL
 - **安全性**：`doctor` 中会启用更严格的部署检查

 ```sh
 pnpm paperclipai onboard
 # 选择 "authenticated" -> "public"
 ```

 ## 董事会接管流程（Board Claim Flow）

 当从 `local_trusted` 迁移到 `authenticated` 时，Paperclip 在启动时会输出一次性接管 URL：

 ```
 /board-claim/<token>?code=<code>
 ```

 已登录用户访问该 URL 即可接管董事会所有权，这会：

 - 将当前用户提升为实例管理员
 - 降级自动创建的本地 board 管理员
 - 确保该用户是所有目标公司的成员

 ## 切换模式（Changing Modes）

 更新部署模式：

 ```sh
 pnpm paperclipai configure --section server
 ```

 也可以通过环境变量在运行时覆盖：

 ```sh
 PAPERCLIP_DEPLOYMENT_MODE=authenticated pnpm paperclipai run
 ```
