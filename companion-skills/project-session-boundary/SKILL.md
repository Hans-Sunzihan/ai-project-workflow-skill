---
name: project-session-boundary
description: "Use only when the user gives 对话开始 or 对话结束 as a clear command to start or close the current software-project conversation. Route 对话开始 to ai-project-workflow Bootstrap Mode and 对话结束 to ai-project-workflow Handoff Mode. Do not trigger when these phrases are merely quoted, discussed, documented, translated, or used in an unrelated sentence."
---

# Project Session Boundary

Act as a lightweight dispatcher for `$ai-project-workflow`. Keep all project workflow logic in that skill.

## Match the Boundary Intent

Proceed only when the user's intent is clearly to start or close the current project conversation:

- `对话开始`
- `对话开始，继续 v0.x`
- `对话结束`
- `对话结束，交接落库`

Do not dispatch when the user is discussing, editing, testing, quoting, or asking about these trigger phrases.

## Dispatch

1. Locate the available `ai-project-workflow` skill through the skill catalog. If needed, resolve its installed `SKILL.md` from the normal skill roots.
2. Read `ai-project-workflow/SKILL.md` once for this boundary operation.
3. Follow only the routed mode. Do not load Code Intelligence or Review Engine references unless that mode and the user's explicit request require them.

### `对话开始`

Treat it as this explicit request:

```text
$ai-project-workflow Bootstrap Mode: initialize the project context, confirm the active version, authoritative sources, task boundary, risks, and immediate next step.
```

If Bootstrap Mode already completed in this conversation and the user repeats `对话开始` without a material version or scope change, reuse the existing anchor instead of rereading unchanged files.

### `对话结束`

Treat it as this explicit request:

```text
$ai-project-workflow Handoff Mode: close the current project session, persist settled context to the active version README or a necessary narrow handoff, record verification and risks, and leave a clear next step.
```

Treat this phrase as explicit authorization to persist the handoff within the project documentation rules. Reuse the current conversation context and do not rerun Bootstrap Mode.

## Failure Boundary

If `ai-project-workflow` is not installed or cannot be located, report that briefly instead of inventing a different bootstrap or handoff workflow.
