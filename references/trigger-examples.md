# Trigger Examples

`ai-project-workflow` is explicit-only. Use it through `$ai-project-workflow`, or through the `project-session-boundary` companion for the two session-boundary phrases.

## Bootstrap Mode

- `$ai-project-workflow 初始化项目上下文。`
- `$ai-project-workflow 恢复这个版本的开发上下文。`
- `$ai-project-workflow 刷新到 v0.3 的项目锚点。`
- `对话开始` -> companion routes to Bootstrap Mode.

## Layer 1 - Project Workflow

- `$ai-project-workflow 按版本推进这个目标。`
- `$ai-project-workflow 为这个功能建立新版本范围。`
- `$ai-project-workflow 逐项澄清权限和状态决策。`
- `$ai-project-workflow 把需求拆成可独立验证的垂直切片。`
- `$ai-project-workflow 组织这次版本文档。`

## Layer 2 - Code Intelligence

- `$ai-project-workflow 帮我理解这个系统或模块。`
- `$ai-project-workflow 追踪这个接口从 controller 到 storage。`
- `$ai-project-workflow 分析这次改动的影响面。`
- `$ai-project-workflow 为这个模块创建或刷新 API index。`

## Layer 3 - Explicit Review

- `$ai-project-workflow review 这个改动。`
- `$ai-project-workflow 严格 review 这个 PR。`
- `$ai-project-workflow 封版前检查风险。`
- `$ai-project-workflow 判断这个改动能否合并。`

## Handoff Mode

- `$ai-project-workflow 交接落库。`
- `$ai-project-workflow 更新版本 README 并写清下一步。`
- `$ai-project-workflow 为下一个 session 写窄 handoff。`
- `对话结束` -> companion routes to Handoff Mode and authorizes persistence.

## Do Not Use It For

- Ordinary implementation, debugging, SQL inspection, test execution, or follow-up discussion after Bootstrap Mode.
- Tiny one-off commands that do not need project bootstrap, planning, explicit review, or handoff.
- Re-reading unchanged workspace rules, version READMEs, code standards, or handoffs on every message.
- Implicitly expanding an interface or cross-module task into Code Intelligence or Review Engine without an explicit workflow request.
- Reading every `session-handoff-*.md`; read the version README index first and open only the relevant handoff.
- Treating quoted or discussed `对话开始` / `对话结束` text as a boundary command.
- Local machine setup, batch git staging, or release publishing unless the user explicitly asks this skill to coordinate the project documentation side.
