 ---
 title: 部署总览（Deployment Overview）
 summary: 各部署模式一览
 ---

 Paperclip 提供三种部署配置，从零摩擦的本地模式到面向互联网的生产部署。

 ## 部署模式（Deployment Modes）

 | 模式 | 认证 | 适用场景 |
 |------|------|----------|
 | `local_trusted` | 无需登录 | 单人本机使用 |
 | `authenticated` + `private` | 需要登录 | 私有网络（Tailscale、VPN、局域网） |
 | `authenticated` + `public` | 需要登录 | 面向公网的云端部署 |

 ## 快速对比（Quick Comparison）

 ### Local Trusted（默认）

 - 仅绑定回环地址（localhost）
 - 无人类登录流程
 - 本地启动速度最快
 - 适用于：个人开发与实验

 ### Authenticated + Private

 - 通过 Better Auth 要求登录
 - 绑定所有网卡，支持网络访问
 - 自动 Base URL 模式（降低配置摩擦）
 - 适用于：在 Tailscale 或局域网中与团队共享

 ### Authenticated + Public

 - 需要登录
 - 必须显式配置公网 URL
 - 更严格的安全检查
 - 适用于：云端托管、面向互联网的部署

 ## 如何选择模式（Choosing a Mode）

 - **只是想试用 Paperclip？** 使用默认的 `local_trusted`
 - **需要在私有网络内共享给团队？** 使用 `authenticated` + `private`
 - **要部署到云端面对公网？** 使用 `authenticated` + `public`

 在 Onboard 过程中设置模式：

 ```sh
 pnpm paperclipai onboard
 ```

 或者之后再修改：

 ```sh
 pnpm paperclipai configure --section server
 ```
