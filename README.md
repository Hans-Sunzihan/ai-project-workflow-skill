# AI Project Workflow Skill
### 把 AI coding 从“单次对话记忆”升级为“基于 workspace 公共事实源的可交接工程协作”。

一个可复用的 AI coding 工作流 skill，用 workspace 文件而不是单次对话记忆来对齐 Codex、Claude Code、Trae 等 AI agent 的项目执行方式。

它适合已经进入真实工程复杂度的项目：需求会跨 session 延续，代码改动需要版本边界，review 需要证据，部署前要关心接口契约、鉴权、账单、DDL、配置、回滚和上线顺序。这个 skill 的目标不是让 agent “多读一点文档”，而是让 agent 每次开工都能从仓库里的公共事实源恢复上下文，并把新的事实继续沉淀回仓库。

## 真实价值

- 把 AI agent 从“靠单次聊天记忆干活”拉回到 workspace 公共事实源：`AGENTS.md`、`docs/project`、`docs/process`、`docs/visions`。
- 把项目工作拆成 Layer 1/2/3，避免每次都做全量系统理解或重 review：不确定时默认只走 Layer 1，只有任务需要时再加 Layer 2/3。
- 让长任务有版本锚点：连续多轮开发、修复和上线准备都落在版本 README 与窄 handoff 中，后续 session 不必凭旧印象判断进度。
- 让改动更小、更可审查：先确定版本范围、模块边界和影响面，再实现、验证、记录，而不是一边猜上下文一边大范围改动。
- 让 AI coding 更像工程协作：允许使用 AI 提效，但最终以代码、diff、验证记录和项目文档为准。

## 适用场景

- 按版本推进需求、修复和交付。
- 维护 `docs/project`、`docs/process`、`docs/visions` 等长期项目文档。
- 做 session handoff，让下一个 AI 或人类接手时少重新推导。
- 做系统代码理解、影响面分析和 diff-based review。
- 协调多 session 并行工作，并保持目标、边界和交接清晰。
- 在大型、多仓库或生产风险较高的项目中局部推进。

## 目录结构

```text
ai-project-workflow-skill/
├── SKILL.md
├── SKILL.zh.md
├── README.md
├── LICENSE
├── agents/
│   └── openai.yaml
└── references/
    ├── trigger-examples.md
    ├── code-intelligence.md
    └── review-engine.md
```

## 核心思想

| 原则 | 说明 |
| --- | --- |
| Workspace is the source of truth | 长期上下文沉淀在仓库文档中，而不是单次聊天记忆中 |
| Work by version | 先明确版本范围，再实现、验证、记录和封版 |
| Local and incremental | 只刷新当前任务需要的文档和代码切片 |
| Diff anchored review | Review 从实际 diff 出发，按风险和证据输出 |

## 三层工作流

| Layer | 名称 | 解决的问题 | 主要产物 |
| --- | --- | --- | --- |
| Layer 1 | Project Workflow | 版本推进、必要澄清、垂直切片、文档落点、交接 | 版本 README、实施顺序、handoff、过程记录 |
| Layer 2 | Code Intelligence | 系统/模块/API 理解、影响面分析 | system-map、module-map、api-index、diff analyzer |
| Layer 3 | Review Engine | Spec / Standards 双轴、Architecture / Implementation 双视角、风险评分 | 结构化 review、阻塞项、验证缺口 |

默认先用 Layer 1。只有任务需要系统、模块、API、依赖或影响面理解时，加 Layer 2；只有任务需要 review、合并判断、风险评估、封版，或触及高风险区时，加 Layer 3。

## 推荐项目文档结构

这个 skill 假设目标项目可以逐步沉淀如下文档。不存在时不强制创建全部内容，只在任务需要时增量补齐。

```text
docs/
├── README.md
├── project/
│   ├── index.md
│   ├── system-map.md
│   ├── modules.md
│   └── api-index/
├── process/
│   ├── index.md
│   ├── boundaries.md
│   ├── code-style-quickcheck.md
│   └── handoff.md
└── visions/
    ├── README.md
    └── v0.2-team-plugin-api/
        └── README.md
```

新建版本目录建议使用 `v0.x-业务英文短名`，例如 `v0.2-team-plugin-api`。这只是命名约定；核心要求仍然是一个版本一个目录，并包含 `README.md`。

## 使用方式

把本仓库作为 skill 引用到你的 AI 开发工具中。当请求涉及版本推进、文档归档、代码理解、review 或 handoff 时，agent 会读取 `SKILL.md` 并根据任务形态选择合适层级。

建议在目标项目的全局 agent 规则（例如 `AGENTS.md`、`CLAUDE.md` 或团队约定的 rules 文件）中补充：

- 公共事实源：agent 不依赖单次对话记忆，先读取 workspace 文档恢复上下文。
- skill 触发条件：版本工作、文档归档、review、handoff、系统代码理解、影响面分析等任务触发 `ai-project-workflow`。
- 启动读取顺序：`AGENTS.md -> docs/README.md -> docs/project/index.md -> docs/process/index.md -> docs/visions/README.md -> 目标版本 README -> repo-local rules -> code-style quickcheck`。
- 功能保真优先：不能为了省 token 跳过 workspace 规则、版本 README、repo-local rules 或代码风格自查；少读和准确理解冲突时，优先准确理解。
- 三层路由：Layer 1 负责版本、文档、handoff；Layer 2 负责 system-map、module-map、api-index、diff analyzer；Layer 3 负责 diff-based review、风险评分和结构化 review。
- 按需分层：默认先用 Layer 1；只有系统/模块/API 理解、跨模块/跨仓库、接口契约或影响分析时加 Layer 2；只有 review、封版、高风险区或较大 diff 时加 Layer 3。
- 实施前澄清：只有真实决策歧义会改变方案时才逐项确认；代码和文档能查到的事实由 agent 自己查，问题一次一个并给出推荐答案。
- 垂直切片：多步骤需求优先按可独立验证的端到端行为拆分，并在版本 README 中记录阻塞关系和涉及的前端、API、数据、权限面。
- Review 四检查面：分开判断 Spec 与 Standards，再按 Architecture 与 Implementation 视角检查；日常 review 按需深入，严格或封版 review 使用完整矩阵。
- handoff 读取纪律：先读目标版本 README 的 `Session Handoff` 索引，只打开与当前任务相关的 `session-handoff-*.md`，不默认全量读取。
- handoff 写入规则：用户明确要求“交接落库”“更新交接文档”时，写入目标版本目录并更新 README 索引；小改动优先更新版本 README，不必每次新建 handoff。
- 文档落点：长期事实写 `docs/project`，过程规则和踩坑写 `docs/process`，版本范围、实施、验证、封版和交接写 `docs/visions`。
- 代码边界：进入具体仓库前读取该仓库规则；保持改动局部化，不做无关重构、批量格式化或目录搬迁。

## 许可证

MIT-0 — 免费使用、修改、分发，无需署名。

## 致谢

本项目在工作流理念上参考了 [DevTaskFlow](https://github.com/cwyhkyochen-a11y/devtaskflow) 的一些观点，并结合通用 AI coding skill 场景重新整理为独立实现。

需求澄清、垂直切片和 Spec / Standards 双轴 review 的部分思路也参考了 [Matt Pocock Skills](https://github.com/mattpocock/skills)，并按本项目的 workspace 事实源和三层工作流重新实现。
