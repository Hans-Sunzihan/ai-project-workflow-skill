---
name: ai-project-workflow-zh
description: "用于软件项目 workspace 中的项目工作流、系统代码理解、diff 影响面分析、review、版本文档、交接和多 session 协同。"
license: MIT-0
---

# AI 项目工作流

## 概览

这个 skill 用 workspace 文件而不是聊天记忆来对齐 AI 编码 session。它分成三个按需启用的层级：

| 层级 | 名称 | 解决的问题 | 参考文档 |
| --- | --- | --- | --- |
| 1 | 项目工作流 | 如何按版本、文档、交接和任务边界推进项目 | 本文件 |
| 2 | 系统代码理解 | 如何理解系统、模块、接口链路和 diff 影响面 | `references/code-intelligence.md` |
| 3 | Review 引擎 | 如何基于 diff、风险评分和结构化输出做 review | `references/review-engine.md` |

不要默认跑完所有层级。根据任务形态选择需要的层。

## 启动流程

触发 skill 后，按顺序读取以下存在的文件。文件缺失时跳过；如果缺失会影响任务判断，需要在回复或文档中说明。

1. `AGENTS.md`
2. `docs/README.md`
3. `docs/project/index.md`
4. `docs/process/index.md`
5. `docs/visions/README.md`
6. 如果指定了版本或存在活跃版本，读取 `docs/visions/<version>/README.md`
7. 涉及代码修改时，读取相关仓库本地规则，例如 `AGENTS.md`、`CLAUDE.md` 或语言规范
8. 涉及代码修改时，读取 `docs/process/code-style-quickcheck.md`（若存在）

## 功能保真优先

这个 skill 是完整的项目 agent 工作流，不是轻量捷径。只有在不削弱项目执行效果时，少量节省 token 才有意义。

- 不要为了节省 token 跳过 workspace 级规则、active/target 版本 README、repo-local rules 或 code-style quickcheck。
- 如果“少读”和“准确理解项目”冲突，优先准确理解项目，并读取必要的源文件。
- 保持三层模型、版本锚定、文档沉淀、review 流程、handoff 流程和本地代码规则不变。
- 本 skill 的目标是稳定按项目工作流执行，而不是极限压缩 token。

## Handoff 读取纪律

`session-handoff-*.md` 对长任务和 session 接力很重要，但它们不是启动时必须全量读取的资料包。

- 启动时读取 active/target 版本 README 及其中的 `Session Handoff` 索引（如果存在），不要默认打开所有 `session-handoff-*.md`。
- 只有当前任务命中模块、日期、关键词、风险点，或 README 明确指向某个 handoff 时，才打开对应 handoff 原文。
- 旧 handoff 是历史证据。回答当前进度或开始改动前，必须用 active README、当前代码/diff 和最新相关 handoff 交叉校验。
- 如果 handoff 与当前代码或更新的版本记录冲突，以较新的、已核对的来源为准，并说明该 handoff 已过时。

## 核心原则

Layer 2 和 Layer 3 是专门的工作流引擎，细节在 `references/` 中。本入口文件只负责判断什么时候调用它们。

- Layer 2：`references/code-intelligence.md`
- Layer 3：`references/review-engine.md`

## 层级选择

| 任务形态 | Layer 1 | Layer 2 | Layer 3 |
| --- | --- | --- | --- |
| 版本规划、文档归档、交接 | 是 | 否 | 否 |
| 单文件小 bug 修复 | 是 | 否 | 可选 |
| 跨模块新功能 | 是 | 是 | 是 |
| “帮我理解这个系统/模块” | 可选 | 是 | 否 |
| “review 这个改动/PR” | 是 | 按需 | 是 |
| 接口契约变更 | 是 | 是 | 是 |
| 不确定怎么做的探索性任务 | 是 | 是 | 否 |

不确定时：

- 默认只用 Layer 1。
- 任务需要理解系统、模块、API、依赖或影响面时，加 Layer 2。
- 任务需要 review、合并判断、风险评估、封版，或触及高风险区时，加 Layer 3。

---

## Layer 1 - 项目工作流

### 决策树

- 如果用户指定版本，以该版本文档作为范围锚点。
- 如果未指定版本，检查 `docs/visions/README.md` 中的活跃版本。
- 如果没有合适版本且任务不只是一次性小修，在实现前创建或建议新的版本目录和 `README.md`。
- 如果任务是 review，遵循 `references/review-engine.md`。
- 如果任务是 handoff，优先遵循 `docs/process/handoff.md`；不存在时使用本文末尾交接格式。
- 如果任务改变长期项目事实，更新 `docs/project`。
- 如果任务沉淀方法、边界、踩坑或协作规则，更新 `docs/process`。
- 如果任务改变需求、范围、验收、验证或封版状态，更新 `docs/visions`。

### 执行规则

- 以 workspace 为事实源。
- 按版本工作：明确范围、实现、验证、更新版本记录、封版或留下明确下一步。
- 保持改动局部化，避免在功能开发中搬迁旧文档或做无关重构。
- 长任务拆阶段或拆 session，并写清交接。
- 探索性任务在上下文变薄前保存过程笔记和证据。
- 并行工作时，主 session 负责目标、接口、边界和最终集成；子 session 负责隔离任务。

---

## Layer 2 - 系统代码理解

目标：在修改系统之前，让 AI 和人类拥有可信的系统模型。

使用 `references/code-intelligence.md` 执行：

- `system-map`
- `module-map`
- `api-index`
- `diff analyzer`

### 何时启用

- 变更涉及多个模块或多个仓库。
- 用户询问系统、模块、依赖图或接口链路如何组织。
- 任务变更接口契约、请求/响应结构、鉴权行为、错误码或数据模型。
- Layer 3 review 需要影响面上下文，但没有最新地图。
- 新 session 第一次接触陌生模块。

### 原则

- 先读源码，再推断；推断要明确标注。
- 产物落到 `docs/project/`，让后续 session 可复用。
- 先检查现有产物，例如 `docs/project/system-map.md`、`docs/project/modules.md`、`docs/project/api-index/` 和目标版本 README，再决定是否从头推导。
- 只刷新当前任务触及的切片。
- 不确定内容标记为“待确认”。
- 尊重现有模块边界和高风险区定义。

---

## Layer 3 - Review 引擎

目标：把 review 变成基于 diff、可评分、可复用的结构化流程。

使用 `references/review-engine.md` 执行：

- diff-based review
- 双视角 review
- 风险评分
- 结构化输出

### 何时启用

- 用户要求 review 某个改动或 PR。
- 封版前。
- 变更触及接口契约、数据迁移、登录鉴权、计费、安全或其他高风险区。
- diff 超过约 5 个文件，或跨多个模块。

### 原则

- 从 `git diff` 开始。
- Findings 优先，并按严重度排序。
- 可执行问题必须带文件和行号。
- 说明未验证项。
- 有未解决阻塞项时不要给 approve 结论。

---

## Handoff 交接

如果用户要求交接，除非要求落盘，否则在回复中写一封简洁交接信。包含：

- 目标
- 已完成工作
- 下一步
- 背景
- 需要阅读的文件
- 当前风险
- 不要做的事
- 验证状态

如果 Layer 2 或 Layer 3 产生了产物，要在交接中引用，避免下个 session 重新推导。

### 何时落盘 handoff

- 只有用户明确要求“交接落库”“写入交接文档”“更新 handoff 文档”等等价表达时，才写入 handoff 文件。
- 小改动优先更新目标版本 README，不必每次新建 handoff。
- 落盘 handoff 保持窄：一次 session、一个阶段、一组明确下一步。
- 写入 handoff 时，将文件放到相关版本目录，并更新版本 README 的 `Session Handoff` 索引（如果项目采用该结构）。

## 参考文档

- `references/trigger-examples.md`
- `references/code-intelligence.md`
- `references/review-engine.md`
