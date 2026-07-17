---
name: ai-project-workflow
description: "Explicit-only workflow for project session bootstrap, version planning, explicitly requested code intelligence or review, and end-of-session handoff. Use only when the user invokes $ai-project-workflow or the project-session-boundary companion routes a session start/end command. Do not use for routine implementation, debugging, SQL, or follow-up discussion."
license: MIT-0
---

# AI Project Workflow

## Overview

This skill keeps AI coding sessions aligned through workspace files instead of chat memory. It is organized as three on-demand layers:

| Layer | Name | Solves | Reference |
| --- | --- | --- | --- |
| 1 | Project Workflow | How to move a project forward by version, clarification, vertical slices, docs, and handoff | this file |
| 2 | Code Intelligence | How to understand a system, module, API chain, or diff impact | `references/code-intelligence.md` |
| 3 | Review Engine | How to review changes with diff anchors, risk scoring, and structured output | `references/review-engine.md` |

Layers are selected by task shape. Do not run all layers by default.

This skill is explicit-only. Invoke it directly with `$ai-project-workflow`, or let the lightweight `project-session-boundary` companion route a clear session boundary command to it. Ordinary project messages must not reactivate it.

## Bootstrap Mode

Run Bootstrap Mode only when one of these conditions is true:

- The user explicitly invokes `$ai-project-workflow` to initialize, start, resume, or refresh project context.
- `project-session-boundary` routes a clear `对话开始` session-start command.
- The active version or task scope changed materially, or context compaction removed information required to work safely.

Before reading files, check whether this conversation already has a usable project anchor. If Bootstrap Mode already completed and the active version, task scope, and relevant files have not changed, reuse that anchor and do not reread the startup bundle.

When Bootstrap Mode is actually needed, read the available files in this order. Skip missing files without drama, but note important gaps when they affect the task.

1. `AGENTS.md`
2. `docs/README.md`
3. `docs/project/index.md`
4. `docs/process/index.md`
5. `docs/visions/README.md`
6. The target version `README.md` under `docs/visions/<version>/`, if one is named or active.
7. Repo-local rules for touched code, such as `AGENTS.md`, `CLAUDE.md`, or language-specific standards.
8. For code changes, read `docs/process/code-style-quickcheck.md` if it exists.

End Bootstrap Mode with a compact project anchor containing the active version, current goal, authoritative files, relevant module boundaries, known risks, and immediate next step. Do not create or update project docs during bootstrap unless the user also requests a planning or documentation change.

## Context Reuse Discipline

- Treat successfully read workspace rules, version documents, and repo-local standards as reusable conversation context.
- Do not reread an unchanged file merely because the user sent another message or the skill was invoked for a different explicit mode.
- Reread only the affected slice when a file changed, the active version or module changed, current code contradicts the cached understanding, or compaction removed a required fact.
- Reuse existing `system-map`, `modules.md`, API indexes, version READMEs, and relevant handoffs before rebuilding context.
- Do not invoke Layer 2 or Layer 3 merely to refresh context.

## Ordinary Development Rules

- Keep durable, always-on repository conventions in a concise `AGENTS.md` or nested repo-local rule file.
- After Bootstrap Mode, routine implementation, debugging, SQL inspection, testing, and follow-up discussion proceed without re-invoking this skill.
- Read a nested repo-local rule or code standard when first touching its scope, when that rule changed, or when the task moves into a different subtree. Do not reload it on every message.
- Use the current task files, code, diff, and verification evidence directly during implementation.

## Functional Fidelity First

This skill is a full project-agent workflow at explicit project boundaries, not a per-message initializer. Token savings are useful only when they do not weaken project execution.

- Do not skip required workspace rules, the active or target version README, repo-local rules, or code-style quickchecks during a real bootstrap, planning, review, or handoff operation just to save tokens.
- If "read less" conflicts with accurately understanding the project, prefer accuracy and read the needed source files.
- Keep the three-layer model, version anchoring, durable docs, review flow, handoff flow, and local code rules intact.
- Reusing verified, unchanged context is part of accurate execution; it is not a reason to weaken validation.

## Handoff Reading Discipline

`session-handoff-*.md` files are important for long work and session handoff, but they are not a startup bundle.

- During Bootstrap Mode, read the active or target version README and its `Session Handoff` index when present. Do not open every `session-handoff-*.md` by default.
- Open a handoff file only when the current task matches its module, date, keyword, risk, or when the README explicitly points to it.
- Treat old handoff files as historical evidence. Before answering current progress or making changes, cross-check with the active README, current code or diff, and the newest relevant handoff.
- If a handoff conflicts with current code or newer version notes, treat the newer checked source as authoritative and call out the stale handoff.

## Core Principle

Layer 2 and Layer 3 are specialized workflow engines documented under `references/`.
This entry file decides when to invoke them. Execution details are authoritative in the reference files.

This entry file should not duplicate:

- system understanding logic
- diff impact analysis logic
- review scoring logic
- architecture reasoning

When a selected layer runs, follow its reference manual directly:

- Layer 2: `references/code-intelligence.md`
- Layer 3: `references/review-engine.md`

## Explicit Mode and Layer Selection

Pick layer(s) only after this skill has been invoked explicitly or routed by the session-boundary companion.

| Explicit mode | Layer 1 | Layer 2 | Layer 3 |
| --- | --- | --- | --- |
| Bootstrap / resume context | yes | on demand for an unfamiliar module | no |
| Version planning or durable docs organization | yes | on demand | no |
| Explicit system/module/API understanding or impact analysis | optional | yes | no |
| Explicit change/PR review | yes | on demand | yes |
| Handoff / closeout | yes | no | only when sealing or explicitly requested |

If uncertain:

- Default to Layer 1 only.
- Add Layer 2 only when the explicit request needs system, module, API, dependency, or impact understanding.
- Add Layer 3 only when the user explicitly asks for review, merge judgment, risk assessment, or release sealing.

---

## Layer 1 - Project Workflow

### Decision Tree

- If the user names a version, use that version document as the scope anchor.
- If no version is named, inspect `docs/visions/README.md` for the active version.
- If no suitable version exists and the task is more than a tiny one-off, create or propose a new version directory with `README.md` before implementation.
- For new version directories, prefer the naming pattern `v0.x-business-english-short-name`, for example `v0.2-team-plugin-api`. This is a convention, not a hard requirement; the durable requirement is still one version per directory plus a `README.md`.
- Open a clarification gate only when unresolved product, permission, state, contract, or acceptance decisions could change the solution. Look up facts; ask one decision at a time with a recommendation; wait for confirmation. Skip clear, low-risk work.
- For multi-step work, plan independently verifiable vertical slices with blockers. Each slice should deliver end-to-end behavior across relevant frontend, API, data, or permission boundaries. Keep the plan in the version README unless the user asks otherwise.
- If the task is a review, follow Layer 3 in `references/review-engine.md`.
- If the task is a handoff, follow `docs/process/handoff.md` if it exists, otherwise use the handoff format below.
- If the task changes long-lived project facts, update `docs/project`.
- If the task records methods, boundaries, pitfalls, or coordination rules, update `docs/process`.
- If the task changes requirements, scope, acceptance, verification, or release status, update `docs/visions`.

### Execution Rules

- Treat the workspace as the source of truth.
- Work by version: define scope, implement, verify, update version notes, then seal or leave explicit next steps.
- Keep edits local. Avoid moving legacy docs or refactoring unrelated code during feature work.
- For long tasks, split phases or sessions with clear handoff.
- For exploratory tasks, save process notes and evidence before context gets thin.
- For parallel work, the main session owns goals, interfaces, boundaries, and final integration; worker sessions own isolated tasks.

---

## Layer 2 - Code Intelligence

Goal: give the AI and the human a trustworthy model of the system before changing it.

Use `references/code-intelligence.md` for:

- `system-map`
- `module-map`
- `api-index`
- `diff analyzer`

### When to Invoke Layer 2

Invoke Layer 2 only when this skill has already been explicitly invoked for code intelligence or planning and at least one condition below applies:

- The change touches more than one module or repository.
- The user asks how the system, module, dependency graph, or API chain is organized.
- The task changes an interface contract, request/response shape, authorization behavior, error code, or data model.
- Layer 3 review needs impact context and no fresh map exists.
- A new session is onboarding to an unfamiliar module.

### Layer 2 Principles

- Read source first, infer second. Mark inferred conclusions clearly.
- Output durable artifacts under `docs/project/`.
- Check existing artifacts such as `docs/project/system-map.md`, `docs/project/modules.md`, `docs/project/api-index/`, and the target version README before rebuilding context from scratch.
- Refresh only the slice touched by the current task.
- Mark uncertain items as "To confirm".
- Respect existing module boundaries and high-risk area definitions.

---

## Layer 3 - Review Engine

Goal: turn review into a structured, risk-scored, diff-anchored process.

Use `references/review-engine.md` for:

- diff-based review
- Spec / Standards review with Architecture / Implementation lenses
- risk scoring
- structured output

### When to Invoke Layer 3

Invoke Layer 3 only when the user explicitly requests a change/PR review, merge judgment, risk assessment, or version seal through this skill. After Layer 3 is selected, deepen the review when:

- The change touches interface contracts, data migration, authentication, authorization, billing, security, or other high-risk areas.
- The diff spans more than about five files or more than one module.

### Layer 3 Principles

- Start from `git diff`.
- Pin spec and standards sources. Review Spec and Standards separately through Architecture and Implementation lenses; deepen only relevant checks except for strict, release, or cross-module reviews.
- Lead with findings, ordered by severity.
- Include file and line references for actionable findings.
- Say what was not verified.
- Do not approve changes with unresolved blocking findings.

---

## Handoff Mode

Run Handoff Mode only when one of these conditions is true:

- The user explicitly invokes `$ai-project-workflow` for handoff, closeout, or persistence.
- `project-session-boundary` routes a clear `对话结束` session-end command.

Treat `对话结束` routed by the companion as an explicit request to persist the settled project context. Reuse the current conversation anchor; do not rerun Bootstrap Mode or reread the full startup bundle.

Before writing, inspect only the evidence needed for closeout:

1. The active or target version README and its `Session Handoff` index.
2. `docs/process/handoff.md` when it exists.
3. Current `git status`, relevant diff, completed work, and verification results.
4. Any project/process document directly changed by the work.

Write a concise handoff that includes:

- Goal
- Completed work
- Next step
- Background
- Files to read
- Current risks
- Things not to do
- Verification status

When Layer 2 or Layer 3 produced artifacts, reference them so the next session can continue without re-deriving context.

### Persistence Rules

- Persist settled context when the user explicitly asks to "persist handoff", "write the handoff to docs", "update handoff docs", says `对话结束` through the companion, or uses equivalent wording.
- For small changes, prefer updating the target version README instead of creating a new handoff file.
- Keep persisted handoff files narrow: one session, one phase, one clear next-step set.
- When writing a persisted handoff, place it in the relevant version directory and update the version README's `Session Handoff` index if that structure exists.
- Do not run a full review during closeout unless the user explicitly requested review or the version is being sealed.

## References

- `references/trigger-examples.md`
- `references/code-intelligence.md`
- `references/review-engine.md`
