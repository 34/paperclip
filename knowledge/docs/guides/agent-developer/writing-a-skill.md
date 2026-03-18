---
title: 编写 Skill（Writing a Skill）
summary: SKILL.md 格式与最佳实践
---

Skill 是 agent 在心跳中可调用的可复用指令，以 Markdown 文件形式教 agent 如何完成特定任务。

## Skill 结构（Skill Structure）

一个 skill 是一个包含带 YAML frontmatter 的 `SKILL.md` 的目录：

```
skills/
└── my-skill/
    ├── SKILL.md          # 主 skill 文档
    └── references/       # 可选参考文件
        └── examples.md
```

## SKILL.md 格式（SKILL.md Format）

```markdown
---
name: my-skill
description: >
  简短描述该 skill 做什么、何时使用。
  用作路由逻辑——agent 通过它决定是否加载完整内容。
---

# My Skill

给 agent 的详细指令...
```

### Frontmatter 字段（Frontmatter Fields）

- **name** — skill 的唯一标识（kebab-case）
- **description** — 路由描述，告诉 agent 何时使用该 skill；写成决策逻辑，而非宣传文案

## 运行时的 Skill 行为（How Skills Work at Runtime）

1. Agent 在上下文中看到 skill 元数据（name + description）
2. Agent 判断该 skill 是否与当前任务相关
3. 若相关，加载完整 SKILL.md 内容
4. Agent 按 skill 中的指令执行

这样基础 prompt 保持较小，完整 skill 内容仅在需要时加载。

## 最佳实践（Best Practices）

- **description 写成路由逻辑** — 包含“何时使用”“何时不用”的指引
- **具体可执行** — agent 应能无歧义地按 skill 执行
- **包含代码示例** — 具体的 API 调用与命令示例比文字描述更可靠
- **一个 skill 一个关注点** — 不要混入无关流程
- **谨慎引用文件** — 辅助内容放在 `references/`，避免主 SKILL.md 过长

## Skill 注入（Skill Injection）

由适配器负责让 skill 对其 agent 运行时可见。`claude_local` 使用临时目录 + 符号链接 + `--add-dir`；`codex_local` 使用全局 skill 目录。详见[创建适配器](/adapters/creating-an-adapter)指南。
