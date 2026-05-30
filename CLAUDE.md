# claude-abap-skills

This repository is a **skill library for Claude Code** working with modern ABAP development. It does not contain any MCP server code — it contains the rules, prompt templates, and slash commands that turn raw model output into **production-quality modern ABAP**.

> The official SAP ABAP MCP Server (`SAPSE.adt-vscode`) gives Claude **hands and eyes** into the system.
> This library gives Claude the **brain**: how to write good code, not just what to access.

---

## Target systems

This library targets **modern ABAP only**:

- **SAP BTP ABAP Environment** (Steampunk)
- **SAP S/4HANA 2023 and later**, when developed in the **ABAP Cloud development model**

Everything in this library assumes those two scopes. The guidance is opinionated and intentionally narrow.

---

## Out of scope — never suggest

When working with this skill library, **never** generate code, refactorings, or recommendations that depend on any of the following:

- SAP ECC
- Classic non-Cloud ABAP development model (e.g. on-prem stacks before 2023)
- Classic dynpro (SE51), screen exits, GUI status programming
- Logical databases, SAP Query, classic batch input
- Unreleased SAP APIs, function modules, classes, or CDS entities
- `SELECT` directly from SAP-owned application tables (e.g. `VBAK`, `MARA`)
- BAPI-style RFM modules where a released cloud API exists
- `FORM`/`PERFORM` routines, includes-only programs, classic exception groups
- `CALL FUNCTION ... DESTINATION` patterns over released communication scenarios

If existing legacy code is provided, your job is to **recognise the legacy pattern, name it, and propose a modern Cloud-compatible alternative** — not to maintain or extend the legacy pattern.

---

## Tooling assumption — the SAP ABAP MCP Server

All system access (read object source, write source, run ATC, run unit tests, browse repository, inspect releases) is assumed to go through the **official SAP ABAP MCP Server** distributed as part of `SAPSE.adt-vscode`.

- Do **not** invent your own ABAP read/write tooling
- Do **not** ask the user to paste source code if the MCP can fetch it
- When you write code back to the system, do it via the MCP and confirm changes succeeded
- For setup instructions, see `docs/mcp-setup.md`

---

## How to load this skill library

Claude Code automatically loads any `AGENTS.md` file in active skill directories. This library exposes two always-on layers:

### Layer 1 — Clean ABAP base
`skills/clean-abap/AGENTS.md`

Universal rules from the SAP Clean ABAP styleguide, distilled into AI-enforceable form. **Always loaded** for any ABAP work.

Slash commands:
- `/clean-abap:review` — review existing code against the Clean ABAP rules
- `/clean-abap:refactor` — refactor existing code to Clean ABAP style

### Layer 2 — ABAP Cloud / RAP overlay
`skills/abap-cloud-rap/AGENTS.md`

System-specific, opinionated rules for the RAP programming model, clean core, and ABAP Cloud development. **Always loaded** in addition to Layer 1.

Slash commands:
- `/abap-cloud-rap:rap-bo-design` — design a new RAP business object end-to-end
- `/abap-cloud-rap:atc-remediation` — fix ATC violations methodically
- `/abap-cloud-rap:clean-core-check` — verify clean core compliance of an object or package

The two layers compose: **Clean ABAP rules apply first**, then the Cloud/RAP overlay refines or extends them.

---

## Default behaviour

- **Always prefer released APIs.** Before suggesting any SAP object (CDS entity, class, interface, function module), assume it must have a C1 release contract. If unsure, ask Claude to verify the release state via the MCP before using it.
- **Always prefer modern ABAP constructs**: inline declarations, `VALUE #( )`, string templates, `COND`/`SWITCH`, table expressions, class-based exceptions, `NEW` over `CREATE OBJECT`.
- **Always prefer CDS-based data access** over `SELECT` from physical tables.
- **Always prefer RAP** for any new transactional behaviour. Do not propose hand-written `BAPI`-style update logic.

---

## When context is ambiguous — ask

Several rules in this library differ between **BTP ABAP Environment** and **S/4HANA 2023+ in ABAP Cloud development model**. Examples:

- BTP forbids some constructs (e.g. some `@AbapCatalog.sqlViewName` usage on interface views) that S/4HANA on-prem still allows
- The set of released APIs differs between systems
- Some ATC variants behave differently

**If the target system is not clear from context, ask the user before generating code.** Do not silently pick one. A single sentence — "Are you targeting BTP ABAP Environment or S/4HANA 2023+ on-prem in Cloud development model?" — is always the right move when in doubt.

---

## License

Apache 2.0 — see `LICENSE`.
