# /clean-abap:refactor

Refactor existing ABAP code to conform to the Clean ABAP rules in `skills/clean-abap/AGENTS.md`.

## What this command does

You are refactoring ABAP source code to bring it into compliance with the Clean ABAP rule set. **Style and structure only — business logic stays identical.** If a change would alter what the code does, you do not make it; you flag it instead.

## Inputs

Accept any of the following. If nothing is provided, ask which target the user wants.

- A code block pasted into the conversation
- An ABAP object name (class, function group, program, include) — fetch the source via the SAP ABAP MCP Server
- A package name — refactor object by object, never as a single bulk change

## Procedure

1. **Load the rule set.** Read every `## RULE:` block in `skills/clean-abap/AGENTS.md`. Those are the only refactoring targets.
2. **Fetch the source via MCP** if not pasted inline. Never ask the user to paste code that the MCP can read.
3. **Run the review first.** Internally, perform the same analysis as `/clean-abap:review`. Use the prioritised list as your refactor plan.
4. **Refactor in passes**, in this order. Do not skip ahead:
   1. **Naming pass** — apply `use-problem-domain-names`, `no-magic-numbers-or-literals`
   2. **Declaration pass** — apply `prefer-inline-declarations`, `no-default-key-on-internal-tables`
   3. **Expression pass** — apply `use-string-templates-not-concatenate`, `prefer-is-not-initial-over-negation`, `prefer-case-over-long-if-elseif`, `use-table-expressions-not-read-table-plus-sy-subrc`
   4. **Method shape pass** — apply `methods-do-one-thing-and-stay-small`, `at-most-three-importing-parameters`, `prefer-returning-over-exporting`
   5. **Error pass** — apply `class-based-exceptions-not-sy-subrc`, `catch-specific-exceptions-not-cx-root`
   6. **Class shape pass** — apply `final-classes-and-private-members-by-default`, `prefer-new-over-create-object`
5. **Preserve behaviour.** After every pass, mentally diff the program logic. If you cannot prove a change is behaviour-preserving (e.g. a refactor would change exception types caught upstream, would change SELECT result order, would alter authority checks), **stop and ask** instead of guessing.
6. **Show the diff before writing back.** Produce a per-method or per-section before/after diff. Annotate every change with the rule name that motivated it.
7. **Ask for confirmation before writing back via MCP.** Explicit confirmation per object — not a blanket yes for a package. After write-back, confirm the object activated cleanly.

## Behaviour preservation — non-negotiable

These changes are out of scope for this command — they alter behaviour:

- Changing exception classes raised by a public method (callers catch them)
- Changing the order or set of database rows returned by a SELECT
- Removing authority checks, even if they look redundant
- Inlining or extracting code that crosses a `COMMIT WORK` / `ROLLBACK WORK`
- Changing the public signature of a method except `EXPORTING → RETURNING` for a single output where no caller relies on `IS SUPPLIED`

If a Clean ABAP rule appears to require one of the above, **flag it** in the report and skip the change. Behaviour change belongs in a separate task, not in a style refactor.

## Output format

```
# Clean ABAP Refactor — <OBJECT NAME>

## Plan
1. <rule-name> — <count> occurrences
2. <rule-name> — <count> occurrences
...

## Changes

### <method or section name>

**Rule:** <rule-name>
**Before:**
` ` `abap
<original>
` ` `
**After:**
` ` `abap
<refactored>
` ` `
**Rationale:** <one sentence, no longer>

(repeat per change)

## Behaviour-preserving check
- [ ] No public method signature changed (except EXPORTING→RETURNING for single output)
- [ ] No exception classes added or removed from public methods
- [ ] No SELECTs reordered or filtered differently
- [ ] No authority checks removed
- [ ] No statements moved across COMMIT/ROLLBACK boundaries

## Skipped
<rule-name> at <location> — <one-line reason it would have changed behaviour>

## Confirmation
Write these changes back to <OBJECT NAME> via MCP? (yes / no / per-method)
```

## Hard rules for this command

- **Style and structure only.** Never change what the code does.
- **One object at a time.** No silent batching across a package.
- **Confirm before write-back.** Always. Even if the user said yes for a previous object.
- **Activate after writing.** If activation fails, report the error and revert the write. Do not patch through activation errors.
- **No rule-by-rule chatter.** Group changes by method or section; one diff per change.
- **If a Clean ABAP rule conflicts with the ABAP Cloud / RAP overlay, the overlay wins.** Apply the overlay version of the rule. Note the conflict in the report.
