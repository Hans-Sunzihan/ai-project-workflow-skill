# ai-project-workflow-skill

一个可公开复用的 AI project workflow skill，用 workspace 文件而不是对话记忆来对齐 Codex、Claude、Trae 等 AI agent 的工作方式。

它适用于：

- 按版本推进需求、修复和交付
- 维护 `docs/project`、`docs/process`、`docs/visions` 等长期项目文档
- 做 session handoff，让下一个 AI 或人类接手时少重新推导
- 做代码理解、影响面分析和 diff-based review
- 协调多 session 并行工作

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

## 许可证

MIT-0 — 免费使用、修改、分发，无需署名。

## 致谢

本项目在工作流理念上参考了 [DevTaskFlow](https://github.com/cwyhkyochen-a11y/devtaskflow) 的一些观点，并结合通用 AI coding skill 场景重新整理为独立实现。
