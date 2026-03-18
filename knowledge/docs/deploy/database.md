 ---
 title: 数据库（Database）
 summary: 嵌入式 PGlite、本地 Docker Postgres 与托管方案
 ---

 Paperclip 通过 Drizzle ORM 使用 PostgreSQL，有三种运行方式。

 ## 1. 嵌入式 PostgreSQL（默认）

 零配置模式。如果未设置 `DATABASE_URL`，服务器会自动启动一个嵌入式 PostgreSQL 实例：

 ```sh
 pnpm dev
 ```

 首次启动时，服务器会：

 1. 创建 `~/.paperclip/instances/default/db/` 作为存储目录
 2. 确保存在名为 `paperclip` 的数据库
 3. 自动运行迁移
 4. 开始对外提供服务

 数据在重启之间会持久化。如需重置：`rm -rf ~/.paperclip/instances/default/db`。

 Docker 快速启动同样默认使用嵌入式 PostgreSQL。

 ## 2. 本地 PostgreSQL（Docker）

 如需在本地运行完整的 PostgreSQL 服务：

 ```sh
 docker compose up -d
 ```

 这会在 `localhost:5432` 启动 PostgreSQL 17。设置连接字符串：

 ```sh
 cp .env.example .env
 # DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip
 ```

 推送 schema：

 ```sh
 DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip \
   npx drizzle-kit push
 ```

 ## 3. 托管 PostgreSQL（Supabase）

 生产环境推荐使用托管提供方，例如 [Supabase](https://supabase.com/)。

 1. 在 [database.new](https://database.new) 创建项目
 2. 在 Project Settings > Database 中复制连接字符串
 3. 在 `.env` 中设置 `DATABASE_URL`

 迁移时使用 **直连**（端口 5432），应用运行时使用 **连接池**（端口 6543）。

 如使用连接池，需关闭 prepared statements：

 ```ts
 // packages/db/src/client.ts
 export function createDb(url: string) {
   const sql = postgres(url, { prepare: false });
   return drizzlePg(sql, { schema });
 }
 ```

 ## 在不同模式间切换（Switching Between Modes）

 | `DATABASE_URL` | 模式 |
 |----------------|------|
 | 未设置 | 嵌入式 PostgreSQL |
 | `postgres://...localhost...` | 本地 Docker PostgreSQL |
 | `postgres://...supabase.com...` | 托管 Supabase |

 无论哪种模式，Drizzle schema（`packages/db/src/schema/`）都是一致的。
*** End Patch```} -->
*** End Patch
