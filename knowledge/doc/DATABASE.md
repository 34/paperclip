# 数据库（Database）

Paperclip 通过 [Drizzle ORM](https://orm.drizzle.team/) 使用 PostgreSQL。有三种运行方式，从最简单到更适合生产。

## 1. 嵌入式 PostgreSQL — 零配置（Embedded PostgreSQL）

若不设置 `DATABASE_URL`，服务会自动启动嵌入式 PostgreSQL 实例并管理本地数据目录。

```sh
pnpm dev
```

即可。首次启动时服务会：

1. 创建 `~/.paperclip/instances/default/db/` 作为存储目录
2. 确保存在 `paperclip` 数据库
3. 对空库自动执行迁移
4. 开始对外服务

数据在 `~/.paperclip/instances/default/db/` 持久化。重置本地开发数据只需删除该目录。

此模式适合本地开发与一键安装。

Docker 说明：Docker 快速启动镜像默认也使用嵌入式 PostgreSQL。持久化 `/paperclip` 可在容器重启后保留 DB 状态（见 `doc/DOCKER.md`）。

## 2. 本地 PostgreSQL（Docker）（Local PostgreSQL）

在本地运行完整 PostgreSQL 服务时，使用仓库内的 Docker Compose：

```sh
docker compose up -d
```

会在 `localhost:5432` 启动 PostgreSQL 17。然后设置连接字符串：

```sh
cp .env.example .env
# .env 中已包含：
# DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip
```

执行迁移（或在迁移生成问题修复后）或使用 `drizzle-kit push`：

```sh
DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip \
  npx drizzle-kit push
```

启动服务：

```sh
pnpm dev
```

## 3. 托管 PostgreSQL（Supabase）（Hosted PostgreSQL）

生产环境使用托管 PostgreSQL 提供方。[Supabase](https://supabase.com/) 是可选方案之一，有免费档。

### 设置（Setup）

1. 在 [database.new](https://database.new) 创建项目
2. 进入 **Project Settings > Database > Connection string**
3. 复制 URI，将密码占位符替换为实际数据库密码

### 连接字符串（Connection string）

Supabase 提供两种连接方式：

**直连**（端口 5432）— 用于迁移与一次性脚本：

```
postgres://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:5432/postgres
```

**通过 Supavisor 的连接池**（端口 6543）— 用于应用：

```
postgres://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres
```

### 配置（Configure）

在 `.env` 中设置 `DATABASE_URL`：

```sh
DATABASE_URL=postgres://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres
```

若使用连接池（端口 6543），`postgres` 客户端需关闭 prepared statements。修改 `packages/db/src/client.ts`：

```ts
export function createDb(url: string) {
  const sql = postgres(url, { prepare: false });
  return drizzlePg(sql, { schema });
}
```

### 推送 Schema（Push the schema）

```sh
# Schema 变更使用直连（端口 5432）
DATABASE_URL=postgres://postgres.[PROJECT-REF]:[PASSWORD]@...5432/postgres \
  npx drizzle-kit push
```

### 免费档限制（Free tier limits）

- 500 MB 数据库存储
- 200 并发连接
- 一周无活动后项目会暂停

详见 [Supabase 定价](https://supabase.com/pricing)。

## 模式切换（Switching between modes）

数据库模式由 `DATABASE_URL` 决定：

| `DATABASE_URL` | 模式 |
|---|---|
| 未设置 | 嵌入式 PostgreSQL（`~/.paperclip/instances/default/db/`） |
| `postgres://...localhost...` | 本地 Docker PostgreSQL |
| `postgres://...supabase.com...` | 托管 Supabase |

无论哪种模式，Drizzle schema（`packages/db/src/schema/`）保持一致。

## 密钥存储（Secret storage）

Paperclip 将 secret 元数据与版本存储在：

- `company_secrets`
- `company_secret_versions`

本地/默认安装下，当前提供方为 `local_encrypted`：

- 密钥内容使用本地主密钥加密存储
- 默认密钥文件：`~/.paperclip/instances/default/secrets/master.key`（缺失时自动创建）
- CLI 配置位置：`~/.paperclip/instances/default/config.json` 中的 `secrets.localEncrypted.keyFilePath`

可选覆盖：

- `PAPERCLIP_SECRETS_MASTER_KEY`（32 字节密钥，base64、hex 或 32 字符原始字符串）
- `PAPERCLIP_SECRETS_MASTER_KEY_FILE`（自定义密钥文件路径）

严格模式可禁止新的内联敏感 env 值：

```sh
PAPERCLIP_SECRETS_STRICT_MODE=true
```

可通过以下命令设置严格模式与提供方默认值：

```sh
pnpm paperclipai configure --section secrets
```

内联 secret 迁移命令：

```sh
pnpm secrets:migrate-inline-env --apply
```
