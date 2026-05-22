---
skill: standup
purpose: Generate a 1-minute standup summary from git activity + task state
trigger: "/standup"
---

# Standup — what's moving, what's stuck, what's next

When invoked, produce a tight 1-minute standup summary. Pretend you're the PM giving a verbal update at the start of a workday.

## What to read
1. Git log of the PM folder for last 7 days:
   `git -C /Users/thebekhruz/Desktop/Career-Vault/PM log --since="7 days ago" --oneline --all`
2. Tasks with status changes in the last 7 days (frontmatter `status: done` files touched recently).
3. Status logs from `Status/` in the last 7 days.
4. Open tasks with target-done in next 7 days.

## What to output

```
## Standup — {{date}}

**Shipped (last 7 days):**
- {{1-line items}}

**In flight:**
- {{Alex: ...}}
- {{Timur: ...}}
- {{Bekhruz: ...}}

**Blocked:**
- {{item, who can unblock}}

**This week, must close:**
- {{deadlines hitting in the next 7 days}}

**Bekhruz's plate:**
- {{decisions or specs only Bekhruz can produce}}
```

## Rules
- Tight. No paragraphs. Bullets only.
- If nothing shipped in 7 days for Parent App and we're inside 90 days of Aug 15, call that out as the headline. Don't bury it.
- If Status/ files are empty or stale, say so — it's a signal.
- If a task is past its target-done date with no completion note, list it as "slipping."
