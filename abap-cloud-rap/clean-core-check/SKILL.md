---
name: clean-core-check
description: Check an ABAP object or package for clean core compliance — unreleased APIs, direct SELECT on SAP-owned tables, classic constructs disallowed in ABAP Cloud, modifications to SAP standard. Use when the user asks to audit, verify, or check clean core compliance, ABAP Cloud compatibility, released API usage, or extension point usage. Produces a categorised report with hard violations and soft warnings. Read-only — never writes back. Targets BTP ABAP Environment and S/4HANA 2023+ in the ABAP Cloud development model.
license: Apache-2.0
---

# abap-cloud-rap:clean-core-check

Check an ABAP object or package for clean core compliance. Pulls the source via the SAP ABAP MCP Server and produces a categorised compliance report with concrete remediation steps.

## What this command does

You are auditing one or more ABAP objects for compliance with the **clean core** principles enforced by ABAP Cloud development model. The rules in `../CLAUDE.md` define what is allowed; this command finds what is not. Output is a structured report — **read-only**, no writes back via MCP.

## Inputs

If not provided, ask. Defaults to the narrowest scope.

- An ABAP object name (class, function group, program, include, CDS entity, BDEF)
- A package name
- A transport request — check every object in the transport

## What to check

For each object in scope, look for the following. Each maps to a rule in `CLAUDE.md`.

### Hard violations — never allowed in either BTP or S/4HANA 2023+ Cloud development

| Check                                                | Source rule                              |
|------------------------------------------------------|------------------------------------------|
| Use of an SAP class/interface/FM with no C1 release  | `released-apis-only`                     |
| Direct `SELECT` from a SAP-owned database table      | `no-direct-select-on-sap-owned-tables`   |
| Classic dynpro (`CALL SCREEN`, screen exits)         | `abap-cloud-language-scope-only`         |
| `SUBMIT`, `CALL TRANSACTION`, `LEAVE TO`             | `abap-cloud-language-scope-only`         |
| `FORM` / `PERFORM`                                   | `abap-cloud-language-scope-only`         |
| `CALL FUNCTION ... DESTINATION 'NONE'` for internal logic | `abap-cloud-language-scope-only`    |
| Modification of SAP standard objects                  | `no-modification-of-sap-standard`        |
| Use of an SAP enhancement point that is not released | `no-modification-of-sap-standard`        |
| `define view` (legacy DDL view) instead of `define view entity` | `interface-entity-required-annotations` |

### Soft warnings — allowed in S/4HANA 2023+ Cloud development for transitional reasons, never allowed in BTP

| Check                                                | Note                                                        |
|------------------------------------------------------|-------------------------------------------------------------|
| Use of a SAP API released only with `Use System-Internally` contract | Allowed on-prem during migration, hard-blocked on BTP |
| Released-API use that is deprecated and slated for removal | Will become a hard violation in a future release |
| Custom CDS view selecting from another *unreleased* custom view | Allowed today, but introduces a chain that will break |

## Procedure

1. **Get the source.** The official `SAPSE.adt-vscode` 1.0 MCP does **not** expose object-source reads, so ask the user to paste the source inline (from ADT in VS Code) or attach a community MCP that exposes reads. Do not paraphrase or imagine source code.
2. **For each object, walk the hard checks.** Record every hit with: object, line/element, check name, root-cause rule, and a concrete remediation.
3. **Then walk the soft checks** for objects targeting S/4HANA 2023+. Skip soft checks if the target is BTP — every soft finding is hard there.
4. **For every finding, propose a concrete fix.** Examples:
   - Direct `SELECT vbak` → "use released `I_SalesOrder`; field `VBELN` becomes `SalesOrder`, `NETWR` becomes `TotalNetAmount`"
   - Unreleased FM call → "use released class `<X>` instead; signature: `<methods>`"
   - `define view ZX_...` → "migrate to `define view entity ZX_...`; remove `@AbapCatalog.sqlViewName`"
   - `PERFORM process_order` → "move logic to a method on a `FINAL` class; declare per Clean ABAP `methods-do-one-thing-and-stay-small`"
5. **Do not write changes.** This command is read-only. The user runs `/abap-cloud-rap:atc-remediation` or fixes by hand.

## Output format

```
# Clean Core Compliance Report — <SCOPE>

Target system: BTP | S/4HANA 2023+
Objects audited: <N>
Hard violations: <N>
Soft warnings: <N>

## Hard violations
(Listed by object, then by check.)

### Object: <name>
#### HV-001 — <one-line violation>
- **Where:** <file> line <N>
- **Check:** <check name from the hard table above>
- **Rule:** <rule-name from CLAUDE.md>
- **Found:**
` ` `abap
<offending code>
` ` `
- **Fix:**
` ` `abap
<concrete replacement, ready to paste>
` ` `
- **Notes:** <e.g. field-name mapping when migrating from a table to a released CDS view>

(repeat per finding)

## Soft warnings (S/4HANA 2023+ only)
(Same structure as hard violations; only emitted when target is S/4HANA on-prem.)

## Summary
| Object                | Hard | Soft | Clean core compliant? |
|-----------------------|------|------|-----------------------|
| <name>                | N    | N    | yes / no              |
...

## Recommended next actions
1. Fix the hard violations first — they prevent activation on BTP.
2. For ATC-checkable findings, run `/abap-cloud-rap:atc-remediation` to apply mechanical fixes in batches.
3. For semantic migrations (e.g. `vbak → I_SalesOrder` with field renames), do them by hand and re-run this command to confirm.
4. Re-run this command after fixes to confirm.
```

## Hard rules for this command

- **Read-only.** No writes back via MCP. Ever.
- **Always state the target system.** Hard vs soft violation depends on it.
- **Always cite the check name and the source rule.** Never present a finding without both.
- **Always show a concrete remediation.** A finding without a fix is not useful.
- **For field-name migrations (e.g. `vbak.vbeln → I_SalesOrder.SalesOrder`), list every renamed field used in the surrounding code** — not just the table swap. Half a migration is worse than none.
- **Pseudo-comment suppression is not a remediation.** If the user wants to ignore a finding, they should justify it through `/abap-cloud-rap:atc-remediation`, not here.
- **Do not include findings that have no rule in `CLAUDE.md`.** This is a compliance check against the library, not a free-form code review.
