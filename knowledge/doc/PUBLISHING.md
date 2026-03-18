# 发布到 npm（Publishing to npm）

本文说明如何构建并将 `paperclipai` CLI 包发布到 npm。

## 前置条件（Prerequisites）

- Node.js 20+
- pnpm 9.15+
- 拥有 `paperclipai` 包发布权限的 npm 账号
- 已登录 npm：`npm login`

## 一键发布（One-Command Publish）

最快方式：一次完成版本号升级、构建、发布、恢复、提交与打标签：

```bash
./scripts/bump-and-publish.sh patch          # 0.1.1 → 0.1.2
./scripts/bump-and-publish.sh minor          # 0.1.1 → 0.2.0
./scripts/bump-and-publish.sh major          # 0.1.1 → 1.0.0
./scripts/bump-and-publish.sh 2.0.0          # 指定版本号
./scripts/bump-and-publish.sh patch --dry-run # 除 npm publish 外全部执行
```

脚本会按顺序执行下面 6 步。需要干净工作区且已 `npm login`（除非使用 `--dry-run`）。完成后执行：

```bash
git push && git push origin v<version>
```

## 手动分步（Manual Step-by-Step）

若希望逐步执行：

### 速查（Quick Reference）

```bash
# 升级版本
./scripts/version-bump.sh patch      # 0.1.0 → 0.1.1

# 构建
./scripts/build-npm.sh

# 预览将要发布的内容
cd cli && npm pack --dry-run

# 发布
cd cli && npm publish --access public

# 恢复开发用 package.json
mv cli/package.dev.json cli/package.json
```

## 分步说明（Step-by-Step）

### 1. 升级版本（Bump the version）

```bash
./scripts/version-bump.sh <patch|minor|major|X.Y.Z>
```

会更新两处版本号：`cli/package.json`（主来源）与 `cli/src/index.ts` 中的 Commander `.version()`。

### 2. 构建（Build）

```bash
./scripts/build-npm.sh
```

构建脚本执行：禁止 token 检查、TypeScript 类型检查、esbuild 打包、生成可发布的 package.json、输出摘要。跳过禁止 token 检查：`./scripts/build-npm.sh --skip-checks`。

### 3. 预览（可选）

查看 npm 将发布的内容：`cd cli && npm pack --dry-run`。

### 4. 发布（Publish）

```bash
cd cli && npm publish --access public
```

### 5. 恢复开发用 package.json（Restore dev package.json）

发布后恢复支持 workspace 的 `package.json`：`mv cli/package.dev.json cli/package.json`。

### 6. 提交与打标签（Commit and tag）

```bash
git add cli/package.json cli/src/index.ts
git commit -m "chore: bump version to X.Y.Z"
git tag vX.Y.Z
```

## package.dev.json

开发时 `cli/package.json` 使用 `workspace:*` 引用；构建时会备份为 `package.dev.json` 并生成带真实依赖版本的 `package.json` 供发布。发布后需恢复 `package.dev.json` 为 `package.json`。`package.dev.json` 在 `.gitignore` 中。

## 打包方式（How the bundle works）

CLI 从 monorepo 的 `@paperclipai/server`、`@paperclipai/db` 等包导入代码；esbuild 将 workspace 代码打成单一 `dist/index.js`，外部 npm 包保持为普通 import，由用户 `npx paperclipai onboard` 时安装。配置见 `cli/esbuild.config.mjs`。

## 禁止 token 检查（Forbidden token enforcement）

构建过程包含与 git pre-commit 相同的禁止 token 检查。token 列表：`.git/hooks/forbidden-tokens.txt`。单独运行：`pnpm check:tokens`。

## npm 脚本参考（npm scripts reference）

| Script | 命令 | 描述 |
|---|---|---|
| `bump-and-publish` | `pnpm bump-and-publish <type>` | 一键升级 + 构建 + 发布 + 提交 + 打标签 |
| `build:npm` | `pnpm build:npm` | 完整构建 |
| `version:bump` | `pnpm version:bump <type>` | 升级 CLI 版本 |
| `check:tokens` | `pnpm check:tokens` | 仅运行禁止 token 检查 |
