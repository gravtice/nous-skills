# Agent Skills

`agent-skills` 是一个面向 agent 工作流的轻量 skill 集合，当前聚焦两类高频能力：

- 统一调用多模型、多模态 AI 能力
- 将本地文件上传到 SwapCloud 并返回临时分享链接

仓库内 skill 名称与实际发布名保持一致，便于直接复用到运行时或文档中。

## Skills Overview

| Skill | 主要功能 | 优势 | Runtime |
| --- | --- | --- | --- |
| `genai-calling` | 用统一接口调用多家模型，覆盖文本、图片、音频、视频、embedding，以及本地 MCP 工作流 | 一套命令同时覆盖 CLI、SDK、MCP；切换 provider 成本低；支持零参数 `.env` 加载，适合快速接入和自动化编排 | `genai-calling` |
| `agent-swapcloud` | 上传本地文件到 SwapCloud（Tencent COS），返回临时签名下载链接 | 非常适合分享大文件、给外部 API 传递文件 URL；支持过期时间和对象 TTL；命令简单，输出可直接消费 | `@gravtice/agent-swapcloud` |

## Skill Details

### `genai-calling`

统一的生成式 AI 调用入口，适合把不同 provider 的文本、图像、音频和视频能力放进同一条 agent workflow 里。

优势：

- 一套模型格式即可切换不同 provider，减少脚本和流程分叉
- 同时支持 CLI、Python SDK、MCP server，适合从本地实验扩展到自动化系统
- 自动加载 `.env.local > .env.production > .env.development > .env.test`，配置简单

入口文件：`skills/genai-calling/SKILL.md`

### `agent-swapcloud`

面向文件分发场景的上传 skill，把本地文件送到 Tencent COS，并返回可直接访问的临时签名链接。

优势：

- 适合截图、日志、JSON、音视频等文件的临时分享
- 支持链接过期时间和对象 TTL，方便控制暴露范围和清理周期
- 输出以可提取 URL 为核心，便于直接衔接下游 API 或消息通知

入口文件：`skills/agent-swapcloud/SKILL.md`

## Repository Structure

- `skills/genai-calling/SKILL.md`
- `skills/agent-swapcloud/SKILL.md`

## License

This repository is licensed under MIT. See [LICENSE](./LICENSE).
