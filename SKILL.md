---
name: ai-project-workflow
description: "Use for project workflow, code intelligence, diff impact analysis, review, version documentation, handoff, and multi-session coordination in a software workspace."
---

# AI Project Workflow

## Overview

This skill keeps AI coding sessions aligned through workspace files instead of chat memory. It is organized as three on-demand layers:

| Layer | Name | Solves | Reference |
| --- | --- | --- | --- |
| 1 | Project Workflow | How to move a project forward by version, docs, handoff, and task boundaries | this file |
| 2 | Code Intelligence | How to understand a system, module, API chain, or diff impact | `references/code-intelligence.md` |
| 3 | Review Engine | How to review changes with diff anchors, risk scoring, and structured output | `references/review-engine.md` |

Layers are selected by task shape. Do not run all layers by default.

## Startup

When this skill triggers, read the available files in this order. Skip missing files without drama, but note important gaps when they affect the task.

1. `AGENTS.md`
2. `docs/README.md`
3. `docs/project/index.md`
4. `docs/process/index.md`
5. `docs/visions/README.md`
6. The target version `README.md` under `docs/visions/<version>/`, if one is named or active.
7. Repo-local rules for touched code, such as `AGENTS.md`, `CLAUDE.md`, or language-specific standards.
8. For code changes, read `docs/process/code-style-quickcheck.md` if it exists.

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

## Layer Selection

Pick layer(s) by task shape.

| Task shape | Layer 1 | Layer 2 | Layer 3 |
| --- | --- | --- | --- |
| Version planning, docs organization, handoff | yes | no | no |
| Tiny single-file bug fix | yes | no | optional |
| Cross-module feature | yes | yes | yes |
| "Help me understand this system/module" | optional | yes | no |
| "Review this change/PR" | yes | on demand | yes |
| Interface contract change | yes | yes | yes |
| Exploratory task with unclear implementation path | yes | yes | no |

If uncertain:

- Default to Layer 1 only.
- Add Layer 2 when the task needs system, module, API, dependency, or impact understanding.
- Add Layer 3 when the task asks for review, merge judgment, risk assessment, release sealing, or touches a high-risk area.

---

## Layer 1 - Project Workflow

### Decision Tree

- If the user names a version, use that version document as the scope anchor.
- If no version is named, inspect `docs/visions/README.md` for the active version.
- If no suitable version exists and the task is more than a tiny one-off, create or propose a new version directory with `README.md` before implementation.
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

- The change touches more than one module or repository.
- The user asks how the system, module, dependency graph, or API chain is organized.
- The task changes an interface contract, request/response shape, authorization behavior, error code, or data model.
- Layer 3 review needs impact context and no fresh map exists.
- A new session is onboarding to an unfamiliar module.

### Layer 2 Principles

- Read source first, infer second. Mark inferred conclusions clearly.
- Output durable artifacts under `docs/project/`.
- Refresh only the slice touched by the current task.
- Mark uncertain items as "To confirm".
- Respect existing module boundaries and high-risk area definitions.

---

## Layer 3 - Review Engine

Goal: turn review into a structured, risk-scored, diff-anchored process.

Use `references/review-engine.md` for:

- diff-based review
- dual-perspective review
- risk scoring
- structured output

### When to Invoke Layer 3

- The user asks to review a change or PR.
- Before sealing a version.
- A change touches interface contracts, data migration, authentication, authorization, billing, security, or other high-risk areas.
- The diff spans more than about five files or more than one module.

### Layer 3 Principles

- Start from `git diff`.
- Lead with findings, ordered by severity.
- Include file and line references for actionable findings.
- Say what was not verified.
- Do not approve changes with unresolved blocking findings.

---

## Handoff

If asked to hand off to another session, write a concise handoff unless the user asks to persist it. Include:

- Goal
- Completed work
- Next step
- Background
- Files to read
- Current risks
- Things not to do
- Verification status

When Layer 2 or Layer 3 produced artifacts, reference them so the next session can continue without re-deriving context.

## References

- `references/trigger-examples.md`
- `references/code-intelligence.md`
- `references/review-engine.md`
