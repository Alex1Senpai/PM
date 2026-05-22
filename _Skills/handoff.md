---
skill: handoff
purpose: Turn a Spec into one or more buildable Tasks ready for Alex or Timur
trigger: "/handoff" or "break this spec into tasks"
---

# Handoff — convert a spec into assignable tasks

When invoked, take a spec and produce the tasks needed to build it. Each task must be small enough that a dev can finish it in 1–5 days and unambiguous enough that they don't need to ask you a question to start.

## How to operate

### Step 1 — Read the spec
- Read the full spec file Bekhruz points you at.
- If the spec has unresolved Open Questions, STOP and ask Bekhruz to close them first. Do not write tasks against an incomplete spec.

### Step 2 — Decompose
Break the spec into the smallest set of tasks that together satisfy all acceptance criteria. Heuristics:
- One task ≠ one acceptance criterion. Acceptance criteria are tests; tasks are work.
- One task should produce a reviewable PR (or PR equivalent in a vendor handoff).
- If a task spans more than 5 working days, split it.
- Setup/scaffolding tasks (e.g. "set up the data model") are valid and should come first.

### Step 3 — Decide assignee
For each task, ask Bekhruz whether Alex or Timur owns it. Defaults to "Unassigned" if Bekhruz doesn't say. Heuristics:
- Architecture / first-of-its-kind work → Alex
- Repetitive implementation, well-scoped surfaces → Timur
- Vendor integration glue code → Alex
- UI/UX implementation following a clear design → either

### Step 4 — Write task files
- Each task as a new file in `Funnel/{stage}/Tasks/T-NNN-{slug}.md`.
- Use the template at `_Templates/Task.md`.
- IDs are sequential per stage. Check existing tasks to find the next number.
- Frontmatter fully filled: id, title, spec, funnel-stage, status (pending), priority, assignee, estimate, created, target-done.
- "What to build" steps are imperative and concrete. No "consider" or "explore" — those belong in a spike task, not an implementation task.
- "Definition of done" items are checkable.
- Update the spec file: link the tasks under "Tasks" section.
- Update the stage `_Hub.md`: link tasks under "Active tasks."

### Step 5 — Sequence
At the end, propose a sequence — which tasks block which. Surface this to Bekhruz so he can confirm.

## Rules
- Never assign a task without Bekhruz's explicit go-ahead.
- Never write a task with empty "Definition of done."
- If you can't write a clear Definition of Done, the task isn't ready — split it or ask Bekhruz more questions.
- Estimates are rough. "Small (<1d), Medium (1-3d), Large (3-5d), XLarge (>5d, split it)."
