# /clean-abap:review

Review existing ABAP code against the Clean ABAP rules in `skills/clean-abap/AGENTS.md`.

## What this command does

You are reviewing ABAP source code for compliance with the Clean ABAP rule set. Your job is to produce a **structured, prioritised review** — not to refactor the code (that is `/clean-abap:refactor`).

## Inputs

Accept any of the following. If none are provided, ask the user which they want.

- A code block pasted into the conversation
- An ABAP object name (class, function group, program, include) — fetch the source via the SAP ABAP MCP Server
- A package name — fetch the list of objects via MCP, then review each in turn

## Procedure

1. **Load the rule set.** Read every rule under `## RULE:` in `skills/clean-abap/AGENTS.md`. These are the only rules in scope for this review.
2. **Fetch the source via MCP** if it was not pasted inline. Do not ask the user to paste code that the MCP can read.
3. **Scan the source against every rule.** Record each violation with: the rule name, the exact line or block in the source, and a one-sentence diagnosis.
4. **Prioritise ATC-checkable violations first.** Rules with an `**ATC**:` line in the rule set are objectively flaggable by a tool — those go to the top of the report. Rules without an ATC reference are still valid findings but are lower priority.
5. **Assign a severity to each finding** using this scale:
   - **Critical** — produces a runtime exception, silently corrupts data, or fails activation (e.g. catching `cx_root` and continuing, `SELECT` on an unreleased table)
   - **Major** — ATC-checkable Clean ABAP violation that survives activation but is unambiguous (magic numbers in conditions, methods over ~30 lines, `EXPORTING` where `RETURNING` works, `READ TABLE ... sy-subrc`)
   - **Minor** — Clean ABAP styleguide preference with no ATC check (long names, missing inline declaration, sub-optimal naming)
6. **Suggest a concrete fix for every finding.** Show a short Do/Avoid snippet, not a vague instruction. The fix must obey every other rule in the set — do not fix one violation by introducing another.
7. **Do not modify the source.** This command is read-only. If the user wants the fixes applied, point them at `/clean-abap:refactor`.

## Output format

Use this exact structure. One section per object reviewed.

```
# Clean ABAP Review — <OBJECT NAME>

## Critical (N)

### 1. <rule-name> — <one-line diagnosis>
**Location:** <file/include> line <N>
**Found:**
` ` `abap
<offending code>
` ` `
**Fix:**
` ` `abap
<concrete replacement>
` ` `

## Major (N)

(same format)

## Minor (N)

(same format)

## Summary
- Critical: N
- Major:    N
- Minor:    N
- Total:    N

## Recommended next step
<one sentence — usually either "run /clean-abap:refactor" or "address the criticals manually first because …">
```

## Hard rules for this command

- **Cite the rule by name.** Every finding starts with `## RULE: <name>` from `AGENTS.md`. No findings without a rule.
- **Do not invent rules.** If something feels off but is not covered by a rule in `AGENTS.md`, mention it in a single "Out of scope observations" section at the very end — not in the main report.
- **Do not modify the source.** Read-only.
- **Do not skip rules to be polite.** A real review names everything; the severity scale handles the noise.
- **One report per object.** If reviewing a package, produce one report per object and a single summary table at the end with totals per object.
