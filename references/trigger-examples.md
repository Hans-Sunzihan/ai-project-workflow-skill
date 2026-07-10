# Trigger Examples

Use `ai-project-workflow` for requests like the examples below.

## Layer 1 - Project Workflow

- "Work on this goal by version."
- "Create a new version scope for this feature."
- "Archive this session's work into docs."
- "Where should this note live: project, process, or visions?"
- "Write a handoff for the next session."
- "Split this feature across multiple sessions."
- "Clarify the permission and state decisions before implementation."
- "Ask me one product decision at a time and give your recommendation."
- "Break this feature into independently verifiable vertical slices."
- "Seal this version."
- "Record this pitfall."

## Layer 2 - Code Intelligence

- "Help me understand how this system is organized."
- "What does this module own?"
- "Map the module dependencies."
- "Trace this API from controller to storage."
- "How large is the impact of this change?"
- "Build an API index for this module."
- "Who consumes this interface?"
- "Create or refresh the system map."

## Layer 3 - Review Engine

- "Review this change."
- "Review this PR."
- "Is this change risky?"
- "Do a strict review."
- "Use dual-perspective review."
- "Review this against both the spec and project standards."
- "Review before release."
- "Can this change be merged?"
- "Does this interface contract change look safe?"

## Do Not Use It For

- Tiny one-off commands that do not touch project state.
- Pure implementation tasks with a clear local scope and no need for durable docs.
- Repeatedly questioning the user about facts available in code or docs, or opening a clarification gate for clear low-risk work.
- Reading every `session-handoff-*.md` just because handoff files exist. Read the active or target version README index first, then open only the relevant handoff files.
- Local machine setup or environment-specific automation unless the project docs define that flow.
- Batch git staging or release publishing unless the user explicitly asks this skill to coordinate the documentation side.
