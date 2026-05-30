# /abap-cloud-rap:atc-remediation

Walk through ATC violations from the connected ABAP system and fix them methodically, grouped by category, with an explanation per finding.

## What this command does

You are pulling ATC (ABAP Test Cockpit) results from the system via the SAP ABAP MCP Server, grouping them by check category, and proposing a concrete fix for each — applying the rules from `skills/clean-abap/AGENTS.md` and `skills/abap-cloud-rap/AGENTS.md`. Pseudo-comment suppressions are off-limits unless the user explicitly justifies and approves them.

## Inputs to collect

If not provided, ask which of these to target. Default to the narrowest scope.

- A single ABAP object name
- A package name
- A transport request number
- An existing ATC run / result ID

Also ask which check variant to run against, defaulting to **`ABAP_CLOUD_DEVELOPMENT_DEFAULT`** for ABAP Cloud development scope.

## Procedure

1. **Run ATC via MCP** against the chosen scope and variant. Do not ask the user to paste ATC results — fetch them.
2. **Group findings by check category.** Typical categories:
   - **Clean Core / Cloud compatibility** — unreleased API usage, direct SELECT on SAP-owned tables, language-version violations
   - **Security & Authorization** — missing authorization checks, unsafe SQL, hard-coded user IDs
   - **Performance** — `SELECT *`, missing indexes, table reads in loops
   - **Code quality / Clean ABAP** — magic numbers, method length, parameter count, `CREATE OBJECT`, chained declarations
   - **CDS / RAP modelling** — missing mandatory annotations, incorrect composition/association, missing draft setup
   - **Testing** — missing ABAP Unit, test classes that hit live data
3. **For each finding, in priority order (Cloud compatibility → Security → Performance → Code quality → CDS/RAP → Testing):**
   - State the violation in one sentence
   - Cite the ATC check name and severity
   - Identify the root cause — name the rule from `AGENTS.md` that applies
   - Show the fix as a Before/After code snippet
   - Propose the action: **auto-apply**, **ask before applying**, or **manual only** (see severity ladder below)
4. **Severity ladder for auto-apply decisions:**
   - **Auto-apply candidates** — mechanical fixes that cannot change behavior: missing annotations (`@AccessControl.authorizationCheck`, `@AbapCatalog.preserveKey`), `CREATE OBJECT → NEW`, `READ TABLE + sy-subrc → line_exists`, chained `DATA:` declarations split. Even auto-apply candidates require batch confirmation, never silent application.
   - **Ask-before-applying** — fixes that touch logic shape but not semantics: `EXPORTING → RETURNING` (only if no caller depends on `IS SUPPLIED`), method extraction, magic-number → named constant.
   - **Manual only** — fixes that change interfaces, exceptions, or data access patterns: replacing `SELECT FROM vbak` with `SELECT FROM I_SalesOrder` (column names differ), replacing unreleased function modules, removing a missing `authorization` block.
5. **Never suppress with pseudo-comments unsolicited.** `"#EC NOTEXT`, `"#EC CI_USAGE_OK`, and friends are not fixes. Refuse to insert them by default. If the user requests suppression, demand: (a) the specific check name, (b) a one-sentence written justification, (c) a JIRA/issue link if one exists. Emit the pseudo-comment with the justification in a comment above it.
6. **After each batch of fixes, re-run ATC via MCP** and report the delta: violations resolved, violations remaining, new violations introduced (if any — back out the offending change).

## Output format

```
# ATC Remediation — <SCOPE>

Variant: <check variant>
Total findings: <N>

## Category breakdown
| Category                       | Total | Auto-apply | Ask | Manual |
|--------------------------------|-------|------------|-----|--------|
| Clean Core / Cloud             | N     | N          | N   | N      |
| Security & Authorization       | N     | N          | N   | N      |
| Performance                    | N     | N          | N   | N      |
| Code quality / Clean ABAP      | N     | N          | N   | N      |
| CDS / RAP modelling            | N     | N          | N   | N      |
| Testing                        | N     | N          | N   | N      |

## Findings

### Category: Clean Core / Cloud
#### F-001 — <one-line violation>
- **Object:** <name> line <N>
- **ATC check:** <check name> — severity <error|warning|info>
- **Root cause rule:** <rule-name from AGENTS.md>
- **Before:**
` ` `abap
<offending code>
` ` `
- **After:**
` ` `abap
<fixed code>
` ` `
- **Disposition:** auto-apply | ask | manual
- **Note:** <if any — e.g. "released CDS view I_SalesOrder used in place of VBAK; field name differs from VBELN to SalesOrder">

(repeat per finding, grouped by category)

## Apply plan
- Auto-apply batch: N changes across M objects — confirm before writing.
- Ask-before-applying: N changes — I will confirm each one.
- Manual: N changes — listed above with full context for you to do by hand.

## Suppressions requested by user
<empty by default; if user explicitly requested any, listed here with justification>
```

## Hard rules for this command

- **Always run ATC via MCP.** Do not work from pasted output.
- **Always group by category.** A flat list of 200 findings is unactionable.
- **Always cite the ATC check name.** "Fix this" is not a remediation.
- **Never apply a fix that changes semantics without asking.** EXPORTING→RETURNING is borderline — confirm.
- **Refuse pseudo-comment suppression by default.** Only emit with an explicit user-provided justification in a comment above the suppressed line.
- **Re-run ATC after each batch.** Report the delta. If new findings appear, roll back the change that introduced them and report.
- **One object = one transaction.** Do not write across many objects in a single irreversible batch.
