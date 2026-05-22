---
skill: pm-status
purpose: Produce a fast, honest health snapshot across all Funnel stages
trigger: "/pm-status" or "give me a PM status"
---

# PM Status — health snapshot across the Funnel

When invoked, generate a status report covering all three Funnel stages. Be honest, not performative. If something is yellow or red, say so. If something is green but only because nothing is happening, say so.

## What to read
1. `Funnel/_Hub.md` — top-level state.
2. `Funnel/1-School-site/_Hub.md`, `Funnel/2-CRM/_Hub.md`, `Funnel/3-Parent-App/_Hub.md`.
3. All open Tasks across all three stages — look for status, assignee, target-done.
4. The most recent 3 files in `Status/` (team's weekly status logs).
5. The most recent decisions in `Funnel/*/Decisions/`.
6. Git log: `git -C /Users/thebekhruz/Desktop/Career-Vault/PM log --oneline -n 30` for recent activity. Use `mcp__workspace__bash` with the workspace path translation if needed.

## What to output

```
## PM Status — {{date}}

### Funnel/3-Parent-App (Aug 15 deadline)
- **Health:** {{green | yellow | red}}
- **Days to deadline:** {{count days from today to 2026-08-15}}
- **Open tasks:** N (M assigned to Alex, K to Timur, L unassigned)
- **Blocked:** {{list any blocked tasks with reasons}}
- **Recent merges:** {{from git log}}
- **What I'd worry about:** {{your honest read}}

### Funnel/2-CRM
... same structure

### Funnel/1-School-site
... same structure

### Cross-funnel
- **Unassigned work above priority "low":** N (this is the gap)
- **Specs without tasks:** N (gap between thinking and doing)
- **Tasks without target-done:** N (commitment gaps)

### Bekhruz's call to action
The 2–3 things only Bekhruz can unblock right now.
```

## Rules
- Do not sugarcoat. If Parent App is behind, say "yellow" or "red" with the reason.
- Color logic for health:
  - **Green:** on track, blockers cleared, recent commits
  - **Yellow:** on track but with concerning signals (slipping target dates, growing unassigned pile, no recent commits)
  - **Red:** off track. Deadline at risk. Major blocker unresolved.
- If team status logs are stale (>10 days), call that out — it means the team isn't using the system.
- End with concrete, named asks for Bekhruz. Not "we should think about" — actual decisions.
