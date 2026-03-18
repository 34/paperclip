# 内置 PostgreSQL 详解（Embedded PostgreSQL）

说明：当未设置 `DATABASE_URL` 时，Paperclip 使用「内置 PostgreSQL」模式。本文说明它是什么、如何工作、以及和「只内置一个数据库」的区别。

---

## 1. 是「内置了一个数据库」还是「内置了整台 PG 服务」？

**是后者：内置的是一整套 PostgreSQL 服务端**，不是单个数据库文件或内存库。

- **一个 PostgreSQL 实例（cluster）** = 一个 `postgres` 进程 + 一块数据目录，可以里面有多个数据库（如 `postgres`、`paperclip` 等）。
- Paperclip 的内置模式会：
  - 在本地 **启动一个真正的 PostgreSQL 进程**（和你在本机装 PG 或 Docker 里跑 PG 一样）；
  - 在这个实例里 **创建一个名为 `paperclip` 的数据库**，应用只连这个库。

所以：**内置 = 内置了「一整台」PostgreSQL 服务；应用用的是这台服务里的「一个」数据库 `paperclip`**。

---

## 2. 如何「内置」的？—— 启动时自动拉起 PG 进程

实现方式可以概括为：

| 项目 | 说明 |
|------|------|
| **依赖** | npm 包 [`embedded-postgres`](https://www.npmjs.com/package/embedded-postgres)（server 的 optionalDependencies） |
| **二进制来源** | 按操作系统/架构安装可选依赖（如 `@embedded-postgres/windows-x64`），里面是 **上游 PostgreSQL 官方二进制**（来自 [zonkyio/embedded-postgres](https://github.com/zonkyio/embedded-postgres)） |
| **运行形态** | 这些二进制在 **Node 进程外** 被 **spawn 成独立进程**，监听一个 TCP 端口（默认 54329），和「本机安装的 postgres」或「Docker 里的 postgres」在协议层面一致 |
| **何时启动** | 在 **Paperclip 服务器启动时**：若未配置 `DATABASE_URL`，则在启动逻辑里 `import('embedded-postgres')`，创建实例并调用 `initialise()`（如需）和 `start()`，从而拉起 PG 进程 |

也就是说：**不是「把数据库嵌在进程里」，而是「由应用进程在启动时自动启动一个真实的 PG 进程」**。

---

## 3. 启动流程（代码级简析）

当 **没有设置 `DATABASE_URL`** 时，`server/src/index.ts` 会走「嵌入式 PostgreSQL」分支，大致顺序如下。

### 3.1 加载 embedded-postgres 并检查是否已在跑

- 动态 `import('embedded-postgres')`；若无此可选依赖则报错，提示装可选依赖或设置 `DATABASE_URL`。
- 解析配置：数据目录 `embeddedPostgresDataDir`（如 `~/.paperclip/instances/default/db`）、端口 `embeddedPostgresPort`（默认 54329）。
- 若数据目录下已有 `postmaster.pid` 且其中记录的 PID 仍在运行，则认为 **已有 PG 进程在跑**，直接复用该进程和端口，不再 `initialise`/`start`。

### 3.2 首次运行：初始化集群并启动进程

- 若数据目录下 **没有** `PG_VERSION`（表示集群尚未初始化）：
  - 创建 `EmbeddedPostgres` 实例（传入 `databaseDir`、`user`、`password`、`port`、`persistent: true` 等）；
  - 调用 **`initialise()`**：在 `databaseDir` 里生成标准 PostgreSQL 集群文件（等价于 `initdb`）；
  - 再调用 **`start()`**：**spawn 出 postgres 可执行文件**，在该端口上启动服务。
- 若目录已存在（`PG_VERSION` 存在），则只调用 **`start()`** 启动进程，不重复 initialise。

### 3.3 创建应用用的数据库并跑迁移

- 用管理员连接串（`postgres://paperclip:paperclip@127.0.0.1:${port}/postgres`）调用 **`ensurePostgresDatabase(..., "paperclip")`**：
  - 若不存在则 `CREATE DATABASE paperclip`；
  - 返回 `"created"` 或 `"exists"`。
- 用应用连接串（`postgres://paperclip:paperclip@127.0.0.1:${port}/paperclip`）跑 **Drizzle 迁移**（`ensureMigrations`），必要时自动应用 pending migrations。
- 之后所有业务代码通过 **`createDb(embeddedConnectionString)`** 连的就是这个 `paperclip` 库。

### 3.4 小结

- **内置 = 启动时自动启动一个真实的 PostgreSQL 进程**；
- 数据目录在磁盘上（`embeddedPostgresDataDir`），**持久化**；
- 应用只使用该实例下的 **一个数据库**：`paperclip`。

---

## 4. 和你可能想到的几种方式对比

| 方式 | 说明 |
|------|------|
| **当前实现（embedded-postgres）** | 真实 PG 二进制，**独立进程**，TCP 端口，标准 PG 协议；数据目录在本地磁盘，可复用、可备份。 |
| **本机安装的 PostgreSQL** | 同样是独立进程 + 端口；区别只是「谁启动」：你手动启动 vs 由 Paperclip 通过 embedded-postgres 启动。 |
| **Docker 里的 Postgres** | 也是独立进程，只不过跑在容器里；设置 `DATABASE_URL` 指向容器端口即可，不再走 embedded-postgres。 |
| **PGlite** | 文档/AGENTS 里曾提到「embedded PGlite」；**当前实现并未使用 PGlite**，而是使用 **embedded-postgres**（真实 PG 进程）。若未来改为 PGlite，则是 in-process、无独立 PG 进程的形态。 |

---

## 5. 数据与进程位置（便于排查）

| 内容 | 典型位置（默认） |
|------|------------------|
| PG 数据目录 | `~/.paperclip/instances/default/db`（或配置中的 `embeddedPostgresDataDir`） |
| PG 进程 | 由 Node 进程 spawn，PID 写在数据目录下的 `postmaster.pid` |
| 监听端口 | 默认 54329（`embeddedPostgresPort`），若被占用会自动选下一个可用端口 |
| 应用使用的数据库名 | `paperclip`（由 `ensurePostgresDatabase` 创建） |

重置内置库：删除上述数据目录后重新启动，会重新 initialise + 建库 + 跑迁移。

---

## 6. 总结

- **内置 PG = 内置了一整套 PostgreSQL 服务**（一个 postgres 进程 + 一块数据目录），不是「只内置一个数据库」。
- **实现方式**：通过 npm 包 `embedded-postgres`，在 **启动时自动 spawn 真实 PG 二进制**，监听本地端口；应用连到该端口上的 **一个** 数据库 `paperclip`。
- 因此：**无需在本机单独安装 PostgreSQL**，只要装好 Node 和项目依赖（含 optional 的 embedded-postgres），就能在「无 DATABASE_URL」的情况下本地跑起来；若希望用 Docker 或其他 PG，只需设置 `DATABASE_URL` 即可切走。
