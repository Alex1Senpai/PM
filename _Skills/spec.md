---
skill: spec
purpose: Interactive write a Spec.md for a feature
trigger: "/spec" or "write a spec for X"
---

# Spec — write a feature/product specification

When invoked, produce a fully filled-in Spec.md using `_Templates/Spec.md` as the structure.

## How to operate

### Step 1 — Ask clarifying questions BEFORE writing anything
You are spec'ing for a dev (Alex or Timur) who is currently lost. The spec must remove ambiguity. Before drafting, ask Bekhruz the questions that close the gaps. Use the `AskUserQuestion` tool for branching choices; use prose for open-ended.

Minimum questions to close before writing:
- Which Funnel stage does this belong to?
- Who is the primary user, and what job are they trying to do?
- What MUST it do (3–7 functional requirements)?
- What's the testable definition of done?
- What's the target ship date?
- Is this SIS-impacting? (For Parent App specs — does it affect the 2027-28 architecture?)
- What's explicitly out of scope?
- What dependencies exist (other specs, vendors, data)?

### Step 2 — Write the spec
- Use the exact template from `_Templates/Spec.md`.
- File goes in `Funnel/{stage}/Specs/{kebab-case-name}.md`.
- Frontmatter fully filled. No placeholder values left.
- Functional requirements numbered and unambiguous. If a dev would have to ask a question to start coding, rewrite the requirement.
- Update `Funnel/{stage}/_Hub.md` to link the new spec under "Active specs."

### Step 3 — Flag what needs follow-up
End with a short list of things Bekhruz still owes (open questions, decisions needed) before this spec can be broken into tasks.

## Rules
- Never invent product details. If Bekhruz didn't specify, ask.
- Never write a spec with `sis-impacting: true` for Parent App without naming the specific SIS-impacting decisions in the Future-state section.
- Never link to specs that don't exist. If you reference a dependency that hasn't been written, create it as a stub or flag it as an open question.
- Russian-language UI requirements should be noted explicitly — most parents are Russian-speaking. Latin program names (Futurum, Hereditum, Initium) should NOT appear in UI copy; use plain descriptors.
