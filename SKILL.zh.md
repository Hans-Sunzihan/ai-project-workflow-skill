---
name: ai-project-workflow-zh
description: "仅用于显式调用的软件项目 session 初始化、版本规划、显式代码理解或 review，以及 session 结束交接。只有用户调用 $ai-project-workflow，或 project-session-boundary 将对话开始/对话结束路由到本 skill 时使用；普通开发、排错、SQL 和后续讨论不要使用。"
license: MIT-0
---

# AI 项目工作流

## 概览

这个 skill 用 workspace 文件而不是聊天记忆来对齐 AI 编码 session。它分成三个按需启用的层级：

| 层级 | 名称 | 解决的问题 | 参考文档 |
| --- | --- | --- | --- |
| 1 | 项目工作流 | 如何按版本、需求澄清、垂直切片、文档和交接推进项目 | 本文件 |
| 2 | 系统代码理解 | 如何理解系统、模块、接口链路和 diff 影响面 | `references/code-intelligence.md` |
| 3 | Review 引擎 | 如何基于 diff、风险评分和结构化输出做 review | `references/review-engine.md` |

不要默认跑完所有层级。根据任务形态选择需要的层。

本 skill 只接受显式调用：用户直接使用 `$ai-project-workflow`，或由轻量的 `project-session-boundary` 将明确的 session 边界指令路由进来。普通项目消息不得重新激活本 skill。

## Bootstrap Mode（初始化模式）

仅在以下情况执行 Bootstrap Mode：

- 用户显式调用 `$ai-project-workflow`，要求初始化、开始、续作或刷新项目上下文。
- `project-session-boundary` 将明确的 `对话开始` 指令路由到这里。
- 活跃版本或任务范围发生实质变化，或者上下文压缩后缺少安全执行所需的信息。

读取文件前，先判断当前对话是否已有可用的项目锚点。如果 Bootstrap Mode 已完成，活跃版本、任务范围和相关文件均未变化，直接复用，不要重复读取初始化资料包。

确实需要 Bootstrap Mode 时，按顺序读取以下存在的文件。文件缺失时跳过；如果缺失会影响任务判断，需要在回复或文档中说明。

1. `AGENTS.md`
2. `docs/README.md`
3. `docs/project/index.md`
4. `docs/process/index.md`
5. `docs/visions/README.md`
6. 如果指定了版本或存在活跃版本，读取 `docs/visions/<version>/README.md`
7. 涉及代码修改时，读取相关仓库本地规则，例如 `AGENTS.md`、`CLAUDE.md` 或语言规范
8. 涉及代码修改时，读取 `docs/process/code-style-quickcheck.md`（若存在）

Bootstrap Mode 结束时输出简洁的项目锚点：活跃版本、当前目标、权威文件、相关模块边界、已知风险和紧接着要做的事。除非用户同时要求规划或文档修改，否则初始化阶段不写项目文档。

## 上下文复用纪律

- 已成功读取的 workspace 规则、版本文档和 repo-local 标准是当前对话可复用的上下文。
- 不要因为用户又发了一条消息，或本 skill 被显式用于另一个模式，就重新读取未变化的文件。
- 只有文件发生变化、活跃版本或模块改变、当前代码与已有理解冲突，或上下文压缩丢失必要事实时，才刷新受影响的切片。
- 优先复用已有 `system-map`、`modules.md`、API index、版本 README 和相关 handoff。
- 不要为了刷新上下文而启动 Layer 2 或 Layer 3。

## 普通开发规则

- 持久且始终生效的仓库规范放在精简的 `AGENTS.md` 或嵌套 repo-local rules 中。
- Bootstrap Mode 完成后，普通实现、排错、SQL 检查、测试和后续讨论都不再调用本 skill。
- 第一次进入某个子树、规则已变化或任务切换到新子树时，读取对应的 repo-local rule 或代码规范；不要每条消息都重读。
- 实施阶段直接使用当前任务文件、代码、diff 和验证证据。

## 功能保真优先

这个 skill 是显式项目边界上的完整工作流，不是每条消息都运行的初始化器。只有在不削弱项目执行效果时，节省 token 才有意义。

- 真正执行初始化、规划、review 或交接时，不要为了节省 token 跳过必需的 workspace 级规则、active/target 版本 README、repo-local rules 或 code-style quickcheck。
- 如果“少读”和“准确理解项目”冲突，优先准确理解项目，并读取必要的源文件。
- 保持三层模型、版本锚定、文档沉淀、review 流程、handoff 流程和本地代码规则不变。
- 复用已经核验且未变化的上下文属于准确执行，不等于降低验证标准。

## Handoff 读取纪律

`session-handoff-*.md` 对长任务和 session 接力很重要，但它们不是启动时必须全量读取的资料包。

- Bootstrap Mode 中读取 active/target 版本 README 及其中的 `Session Handoff` 索引（如果存在），不要默认打开所有 `session-handoff-*.md`。
- 只有当前任务命中模块、日期、关键词、风险点，或 README 明确指向某个 handoff 时，才打开对应 handoff 原文。
- 旧 handoff 是历史证据。回答当前进度或开始改动前，必须用 active README、当前代码/diff 和最新相关 handoff 交叉校验。
- 如果 handoff 与当前代码或更新的版本记录冲突，以较新的、已核对的来源为准，并说明该 handoff 已过时。

## 核心原则

Layer 2 和 Layer 3 是专门的工作流引擎，细节在 `references/` 中。本入口文件只负责判断什么时候调用它们。

- Layer 2：`references/code-intelligence.md`
- Layer 3：`references/review-engine.md`

## 显式模式与层级选择

仅在本 skill 已被显式调用，或由 session-boundary companion 路由后，才选择层级。

| 显式模式 | Layer 1 | Layer 2 | Layer 3 |
| --- | --- | --- | --- |
| 初始化或恢复上下文 | 是 | 陌生模块时按需 | 否 |
| 版本规划或长期文档组织 | 是 | 按需 | 否 |
| 显式系统/模块/API 理解或影响分析 | 可选 | 是 | 否 |
| 显式改动/PR review | 是 | 按需 | 是 |
| 交接或结束 session | 是 | 否 | 仅封版或显式要求时 |

不确定时：

- 默认只用 Layer 1。
- 只有显式请求需要理解系统、模块、API、依赖或影响面时，加 Layer 2。
- 只有用户显式要求 review、合并判断、风险评估或封版时，加 Layer 3。

---

## Layer 1 - 项目工作流

### 决策树

- 如果用户指定版本，以该版本文档作为范围锚点。
- 如果未指定版本，检查 `docs/visions/README.md` 中的活跃版本。
- 如果没有合适版本且任务不只是一次性小修，在实现前创建或建议新的版本目录和 `README.md`。
- 新建版本目录时，建议使用 `v0.x-业务英文短名`，例如 `v0.2-team-plugin-api`。这是约定，不是强制要求；硬性骨架仍然是一个版本一个目录，并包含 `README.md`。
- 只有产品、权限、状态、契约或验收决策会改变方案时，才开启澄清门。事实由 Agent 自己查；决策一次问一个并给出推荐答案，确认后继续。清晰、低风险的任务跳过。
- 多步骤工作按可独立验证的垂直切片规划并标明阻塞关系；每个切片贯穿相关前端、API、数据或权限边界，交付端到端行为。除非用户另有要求，计划写在版本 README 中。
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

只有本 skill 已经因为代码理解或规划被显式调用，并且至少符合以下一项时，才启用 Layer 2：

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
- Spec / Standards 双轴与 Architecture / Implementation 双视角 review
- 风险评分
- 结构化输出

### 何时启用

只有用户通过本 skill 显式要求改动/PR review、合并判断、风险评估或封版时，才启用 Layer 3。Layer 3 已选中后，符合以下情况时加深 review：

- 变更触及接口契约、数据迁移、登录鉴权、计费、安全或其他高风险区。
- diff 超过约 5 个文件，或跨多个模块。

### 原则

- 从 `git diff` 开始。
- 固定需求源和工程规范源；分开检查 Spec 与 Standards，并用 Architecture 与 Implementation 作为视角。日常 review 只深入相关面，严格、封版或跨模块 review 使用完整矩阵。
- Findings 优先，并按严重度排序。
- 可执行问题必须带文件和行号。
- 说明未验证项。
- 有未解决阻塞项时不要给 approve 结论。

---

## Handoff Mode（交接模式）

仅在以下情况执行 Handoff Mode：

- 用户显式调用 `$ai-project-workflow`，要求交接、结束或落库。
- `project-session-boundary` 将明确的 `对话结束` 指令路由到这里。

companion 路由的 `对话结束` 视为用户明确要求把已确定的项目上下文交接落库。复用当前对话锚点，不要重新执行 Bootstrap Mode，也不要重读完整初始化资料包。

写入前只核对结束 session 所需的证据：

1. active/target 版本 README 及其 `Session Handoff` 索引。
2. `docs/process/handoff.md`（若存在）。
3. 当前 `git status`、相关 diff、已完成工作和验证结果。
4. 本轮直接改变的 project/process 文档。

写一封简洁交接信，包含：

- 目标
- 已完成工作
- 下一步
- 背景
- 需要阅读的文件
- 当前风险
- 不要做的事
- 验证状态

如果 Layer 2 或 Layer 3 产生了产物，要在交接中引用，避免下个 session 重新推导。

### 落库规则

- 用户明确要求“交接落库”“写入交接文档”“更新 handoff 文档”，通过 companion 说 `对话结束`，或使用等价表达时，沉淀已确定的上下文。
- 小改动优先更新目标版本 README，不必每次新建 handoff。
- 落盘 handoff 保持窄：一次 session、一个阶段、一组明确下一步。
- 写入 handoff 时，将文件放到相关版本目录，并更新版本 README 的 `Session Handoff` 索引（如果项目采用该结构）。
- 除非用户显式要求 review 或正在封版，结束 session 时不要自动运行完整 review。

## 参考文档

- `references/trigger-examples.md`
- `references/code-intelligence.md`
- `references/review-engine.md`
