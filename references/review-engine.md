# Review Engine Layer

This layer turns review into a structured, risk-scored, diff-anchored process.

## Contents

1. diff-based review
2. two-axis, dual-lens review
3. risk scoring
4. structured output

## General Principles

- Start from `git diff` and untracked files.
- Pin the relevant spec sources and standards sources before judging the diff.
- Keep Spec and Standards findings separate so one axis cannot hide failure in the other.
- Findings come first, ordered by severity.
- Actionable findings include file and line references.
- Say what was not verified.
- Do not approve a change with unresolved blocking findings.
- Persist review reports only when the user asks for durable output, the review is for release sealing, or the diff spans modules/repositories. Daily reviews default to inline output.

---

## 1. diff-based review

### Goal

Review only the changed parts, hunk by hunk, unless the change is a full-file rewrite.

### Inputs

- `git status --short`
- `git diff HEAD`
- Untracked new files
- Layer 2 diff analyzer report, if available
- Spec sources such as the active version README, PRD, acceptance criteria, or API contract
- Standards sources such as `AGENTS.md`, repo-local rules, `docs/process/boundaries.md`, or `docs/process/code-style-quickcheck.md`

### Steps

1. Run `git status --short` to list changes.
2. Run `git diff HEAD` to inspect staged and unstaged changes.
3. Read untracked new files directly.
4. Identify the spec and standards sources relevant to the changed slice. If either is missing, say so instead of inventing it from the diff.
5. For each hunk, answer:
   - What changed?
   - Why does it appear to have changed?
   - Does it satisfy the spec without missing behavior or scope creep?
   - Does it follow project standards and surrounding conventions?
   - Are there architecture or implementation side effects?
6. Record findings with severity, axis, lens, file, line, description, and suggested fix.

### Severity

| Severity | Meaning |
| --- | --- |
| P0 | Blocking. Must fix before merge or release. |
| P1 | Serious. Strongly recommended to fix. |
| P2 | Suggested. Worth improving, but may not block. |

### Common P0 Checks

- Public contract changed without updating consumers.
- Data migration or schema change lacks rollback or compatibility plan.
- Security, auth, billing, payment, or data-loss risk is introduced.
- SQL or command injection risk is introduced.
- Write operation loses transaction or consistency guarantees.

### Common P1 Checks

- Business logic is placed in the wrong layer.
- Module boundaries are bypassed.
- Sensitive data is logged or returned without masking.
- Missing validation or authorization.
- Missing meaningful verification for a risky change.

### Common P2 Checks

- Naming does not match local conventions.
- Comments or docs are stale.
- Duplicate logic could use existing helpers.
- Small performance concern such as avoidable repeated calls.

---

## 2. two-axis, dual-lens review

### Goal

Review delivery correctness and engineering conformance separately, then apply architecture and implementation lenses. Do not require parallel reviewers; use them only when the user requests strict independent review and parallel work is authorized.

### Axes

| Axis | Question | Primary sources |
| --- | --- | --- |
| Spec | Does the change faithfully deliver the requested behavior without omissions or scope creep? | Version README, PRD, acceptance criteria, API contract |
| Standards | Does the change follow documented rules and established project conventions? | `AGENTS.md`, repo-local rules, boundaries, style quickcheck |

If no usable Spec source exists, mark the Spec axis as not verified. If no documented Standards source exists, use surrounding code only as inferred convention and label it as inferred.

### Lenses

| Review surface | Focus |
| --- | --- |
| Spec × Architecture | Intended module ownership, data ownership, contracts, and version scope |
| Spec × Implementation | Required behavior, edge cases, acceptance criteria, and unrequested behavior |
| Standards × Architecture | Module boundaries, dependency direction, public contracts, and consistency guarantees |
| Standards × Implementation | Local conventions, validation, errors, security, performance, tests, and verification |

### Depth Control

- Routine review: scan both axes, then deepen only the lenses implicated by the diff.
- Strict, release-sealing, contract, or cross-module review: run the full four-surface matrix.
- If one source is missing, do not spend tokens simulating that axis; report the verification gap.

### Merge Rules

| Situation | Handling |
| --- | --- |
| More than one surface finds the same issue | Keep one finding, use the higher severity, and list all matching axes/lenses |
| Only one surface finds an issue | Keep it with that axis and lens; do not duplicate it elsewhere |
| Spec and Standards conflict | Keep both notes and mark "needs discussion"; do not let one silently override the other |

### Use When

- The diff crosses modules or repositories.
- Interface contracts change.
- The user asks for strict or dual review.
- Review happens before release sealing.

---

## 3. risk scoring

### Goal

Make merge or release judgment more objective.

Score from 0 to 10. Higher means riskier.

| Score | Level | Meaning | Recommendation |
| --- | --- | --- | --- |
| 0-3 | Low | Local change, limited impact, verified | Usually safe to merge |
| 4-6 | Medium | Some impact or incomplete verification | Fix P1s or confirm risks |
| 7-10 | High | Contract, data, security, or cross-system risk | Fix P0s and get human confirmation |

### Dimensions

| Dimension | 0 | 1 | 2 |
| --- | --- | --- | --- |
| Impact | Single file/module | Cross-module | Cross-repository/system |
| Complexity | Simple local logic | Business rules | State, transactions, async, concurrency |
| Boundary risk | None | Suspected | Confirmed |
| Data consistency | No data impact | Data impact with plan | Data impact without plan |
| Verification | Automated or strong manual verification | Limited manual verification | No meaningful verification |

Add up to 2 points for high-risk areas, capped at 10.

### Output Template

```markdown
## Risk Score

| Dimension | Score | Rationale |
| --- | --- | --- |
| Impact | 1/2 | Cross-module call path |
| Complexity | 1/2 | Business logic changed |
| Boundary risk | 0/2 | No boundary issue found |
| Data consistency | 0/2 | No data model change |
| Verification | 1/2 | Manual verification only |
| High-risk add-on | +0 | No high-risk area identified |
| **Total** | **3/10** | **Low risk** |
```

---

## 4. structured output

Use this shape for non-trivial reviews.

```markdown
# Review Report

> Generated: yyyy-MM-dd
> Reviewer: ai-project-workflow Layer 3
> Scope: <repository/module/version>
> Mode: <diff-based / full review matrix>

## 1. Findings

### [P0] <title>
- File: <file>:<line>
- Axis: <spec / standards / both / needs discussion>
- Lens: <architecture / implementation / both>
- Problem: <description>
- Suggestion: <fix>
- Status: <open / fixed / accepted with rationale>

## 2. Risk Score

| Dimension | Score | Rationale |
| --- | --- | --- |
| Impact | x/2 | ... |
| Complexity | x/2 | ... |
| Boundary risk | x/2 | ... |
| Data consistency | x/2 | ... |
| Verification | x/2 | ... |
| High-risk add-on | +x | ... |
| **Total** | **x/10** | **<low/medium/high> risk** |

## 3. Impact Analysis

- Modules: <list>
- Repositories: <list>
- Contract changes: <yes/no>
- Data changes: <yes/no>
- Client synchronization needed: <yes/no>

## 4. Verification

### Verified
- <item>

### Not Verified
- <item>

## 5. Recommendation

| Item | Content |
| --- | --- |
| Conclusion | <approve / request changes / needs discussion> |
| Must fix | <P0 list or none> |
| Should fix | <P1 list or none> |
| Confirm before merge | <items> |

## 6. Summary

<2-3 sentences. Summary comes last.>
```

### Output Rules

- Sort findings by severity: P0, P1, P2.
- If no issues are found, say "No blocking findings found" and list residual risks or unverified items.
- If a Spec or Standards source was unavailable, report that axis as not verified rather than treating it as passed.
- For release-sealing or cross-module review, persist the report under the relevant version docs when the user wants durable output.
- For routine reviews, keep the report inline unless the user explicitly asks to write it to project docs.
