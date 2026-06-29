# ai-project-workflow-skill

一个可公开复用的 AI project workflow skill，用 workspace 文件而不是对话记忆来对齐 Codex、Claude、Trae 等 AI agent 的工作方式。

它适用于：

- 按版本推进需求、修复和交付
- 维护 `docs/project`、`docs/process`、`docs/visions` 等长期项目文档
- 做 session handoff，让下一个 AI 或人类接手时少重新推导
- 做代码理解、影响面分析和 diff-based review
- 协调多 session 并行工作

例如：
- 在已有大型、多仓库项目中局部推进；
- 先确定版本、模块边界和影响面；
- 修改必须小而可审查；
- 强调接口契约、账单、鉴权、DDL、部署顺序等生产风险；
- 通过版本文档和 handoff 保存事实；
- 对 Token 成本敏感，倾向“最小必要切片”。
- 允许使用AI coding，但要为自己的代码负责。

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
    └── v0.1/
        └── README.md
```

## 使用方式

把本仓库作为 skill 引用到你的 AI 开发工具中。当请求涉及版本推进、文档归档、代码理解、review 或 handoff 时，agent 会读取 `SKILL.md` 并根据任务形态选择合适层级。

建议在目标项目的全局 agent 规则（例如 `AGENTS.md`、`CLAUDE.md` 或团队约定的 rules 文件）中补充：

- 公共事实源：agent 不依赖单次对话记忆，先读取 workspace 文档恢复上下文。
- skill 触发条件：版本工作、文档归档、review、handoff、系统代码理解、影响面分析等任务触发 `ai-project-workflow`。
- 启动读取顺序：`AGENTS.md -> docs/README.md -> docs/project/index.md -> docs/process/index.md -> docs/visions/README.md -> 目标版本 README -> repo-local rules -> code-style quickcheck`。
- 功能保真优先：不能为了省 token 跳过 workspace 规则、版本 README、repo-local rules 或代码风格自查；少读和准确理解冲突时，优先准确理解。
- 三层路由：Layer 1 负责版本、文档、handoff；Layer 2 负责 system-map、module-map、api-index、diff analyzer；Layer 3 负责 diff-based review、风险评分和结构化 review。
- 按需分层：默认先用 Layer 1；只有系统/模块/API 理解、跨模块/跨仓库、接口契约或影响分析时加 Layer 2；只有 review、封版、高风险区或较大 diff 时加 Layer 3。
- handoff 读取纪律：先读目标版本 README 的 `Session Handoff` 索引，只打开与当前任务相关的 `session-handoff-*.md`，不默认全量读取。
- handoff 写入规则：用户明确要求“交接落库”“更新交接文档”时，写入目标版本目录并更新 README 索引；小改动优先更新版本 README，不必每次新建 handoff。
- 文档落点：长期事实写 `docs/project`，过程规则和踩坑写 `docs/process`，版本范围、实施、验证、封版和交接写 `docs/visions`。
- 代码边界：进入具体仓库前读取该仓库规则；保持改动局部化，不做无关重构、批量格式化或目录搬迁。

## 许可证

MIT-0 — 免费使用、修改、分发，无需署名。

## 致谢

本项目在工作流理念上参考了 [DevTaskFlow](https://github.com/cwyhkyochen-a11y/devtaskflow) 的一些观点，并结合通用 AI coding skill 场景重新整理为独立实现。
