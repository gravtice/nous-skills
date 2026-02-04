---
name: find-skills
description: 当用户问“有没有 skill 可以做 X / 帮我找一个 skill / 我想扩展能力”等时使用。注意：在 CoWork 中安装/更新 skills 必须通过 CoWork 提供的 skills 管理工具（mcp__cowork__skills_*），不要在容器内运行 npx skills / 手动 git clone 来绕过管理与安全约束。
---

# Find Skills（CoWork）

这个 skill 用来帮助你在常见 skill 仓库中发现可用的 skills，并在用户确认后安装到 Runner。

## 何时使用

当用户出现以下意图时使用：

- “有没有 skill 可以做 X / 找一个 skill”
- “你能不能自动做 X（可能需要专用工作流/工具）”
- “能不能扩展能力/装插件/装 skill”

## CoWork 的 skills 管理方式（必须遵守）

在 CoWork 里，skills 由 Runner 统一管理。Agent 只能通过 MCP 工具来发现/安装/删除：

- `mcp__cowork__skills_list`：列出已安装 skills
- `mcp__cowork__skills_discover`：从一个 source 发现 skills（不安装）
- `mcp__cowork__skills_install`：安装 skills（必须显式给出 `skills` 列表）
- `mcp__cowork__skills_delete`：删除已安装 skill

不要在容器里运行 `npx skills`、`git clone` 或自行写入 skills 目录来“手动安装”，否则会绕过 Runner 的来源记录/白名单/版本约束，造成不可控行为与维护成本。

## 热门 sources（优先从这些找）

优先尝试这些“直接包含 skills”的仓库：

- `vercel-labs/agent-skills`
- `supabase/agent-skills`
- `anthropics/skills`
- `RefoundAI/lenny-skills`

以下来源多为“目录/清单”，不一定能直接 discover 出 skills；可用于人工定位具体仓库再安装：

- `ComposioHQ/awesome-claude-skills`
- `affaan-m/everything-claude-code`
- `https://skills.sh/`

## 推荐工作流（最小闭环）

### 1) 先确认需求

把用户需求压缩成 2~5 个关键词（领域 + 任务），例如：

- “Playwright E2E” / “PR review” / “changelog” / “Next.js performance”

### 2) 从 sources 发现 skills（并做本地筛选）

对每个候选 source 调用一次 `mcp__cowork__skills_discover`，拿到列表后在本地按关键词过滤：

- 匹配字段：`install_name` / `name` / `description`
- 匹配方式：大小写不敏感；优先 `name` 和 `description`

把最相关的 3~5 个结果展示给用户，并说明它们来自哪个 source。

### 3) 用户确认后安装

用户选定后，调用 `mcp__cowork__skills_install`：

- `source`：与 discover 时一致
- `skills`：必须填，值为 discover 返回的 `install_name`（推荐一次只装 1~3 个）
- `replace`：默认 `false`；除非用户明确要求覆盖

安装完成后，再用 `mcp__cowork__skills_list` 确认已安装。

> 提示：安装后通常需要重启/新建一次 Agent 会话才能加载新 skill。若你看到工具返回 `restart_recommended=true`，请提示用户重启会话。

## 没找到合适 skill 时

- 直接说明“当前 sources 未发现匹配项”，并继续用通用能力完成任务。
- 若用户经常重复该任务，建议把流程沉淀为内部 skill（一个目录 + `SKILL.md`），再用 `mcp__cowork__skills_install` 从本地仓库安装。
