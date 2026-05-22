---
skill: sprint
purpose: Plan a 2-week sprint with a single goal, committed tasks, and risks
trigger: "/sprint" or "plan the next sprint"
---

# Sprint — plan the next two weeks

When invoked, plan a 2-week sprint. Output a file in `Sprints/S-NNN-{slug}.md` using `_Templates/Sprint.md`.

## How to operate

### Step 1 — Decide the sprint goal
Ask Bekhruz: "What is the one thing this sprint must prove or ship?" If he gives you two, push back. A sprint with two goals has none.

For Parent App context:
- Days remaining to Aug 15 should drive goal urgency.
- Goals should be outcome-shaped, not output-shaped. "Payments end-to-end working in staging" beats "build payments features."

### Step 2 — Pull candidate tasks
- Tasks with status `pending`, priority `urgent` or `high`, that serve the sprint goal.
- Tasks blocked by external dependencies should NOT be committed unless the dependency clears this sprint.

### Step 3 — Commit realistically
Capacity rule of thumb:
- Alex: 1 large + 1 medium task per week. So ~2 large or 4 medium across a 2-week sprint.
- Timur: 2 medium tasks per week. So ~4 medium across a 2-week sprint.
- Do not commit "stretch" until committed work is realistic.

Honest budget: cut commitments by 20% from what feels possible. Sprints overrun, not underrun.

### Step 4 — Write the sprint file
- Use `_Templates/Sprint.md`.
- Fill the committed-work table with task links.
- Risks section: name 2–3 specific things that could blow up this sprint.
- Out of scope section: 2–3 attractive things being explicitly deferred.

### Step 5 — Update task target-done dates
For each committed task, set `target-done` to a date within the sprint window. Without this, "committed" is meaningless.

## Rules
- Sprints are 14 days. Start Monday. End Friday-of-week-2.
- One goal per sprint. Reject scope creep.
- Surface risks honestly. If a sprint depends on a vendor reply, that's a risk.
- After Bekhruz confirms, mark the sprint `status: active` and update relevant Hubs.
