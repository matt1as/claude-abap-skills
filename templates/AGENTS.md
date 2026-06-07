# AGENTS.md — ABAP project workspace

Drop this file at the root of your ABAP project workspace. It tells AI coding agents how to behave when working on ABAP in this project — applies to any agent that reads `AGENTS.md` (e.g. GitHub Copilot, Cursor, Claude Code).

Replace the `<PLACEHOLDER>` values, delete the option you don't want under **Target system**, and trim sections that don't apply.

---

## Target system

This project targets: **<SAP BTP ABAP Environment | SAP S/4HANA on-prem in ABAP Cloud development model>**.

Pick one — the two scopes differ in released APIs, allowed CDS annotations, and ATC variants. Never silently mix them. If a request is ambiguous, ask the user before generating code.

---

## General rules

- Use cloud-compliant ABAP syntax and APIs. Released APIs (C1 release contract) only.
- The package `$tmp` is used for local experiments — no transports needed.
- When creating classes in `$tmp`, use the prefix `<YOUR_PREFIX>`.
- Create new ABAP development objects via the ABAP MCP server (`SAPSE.adt-vscode`), not by hand-writing source files.
- Prefer modern ABAP constructs: inline declarations, `VALUE #( )`, string templates, `COND` / `SWITCH`, table expressions, class-based exceptions, `NEW` over `CREATE OBJECT`.
- Prefer CDS-based data access over `SELECT` from physical tables.
- Use RAP for new transactional behaviour. Do not write `BAPI`-style update logic by hand.

---

## When running inside ADT for VS Code

These rules apply **only** when the workspace is hosted by ADT for VS Code (the SAP `SAPSE.adt-vscode` extension). They don't apply in ADT for Eclipse or when an MCP client is talking to the language server out-of-process.

- The ABAP repository is exposed as a **virtual workspace** — files don't live on local disk.
- Adjust file search to browse by directory first. Recursive content-grep over the virtual FS is unreliable.
- Add or edit source code through the VS Code editor. If a file isn't open yet, open it via the path the MCP server provides — don't try to write to local paths.

---

## Testing

- Run unit tests after adding new tests or changing source code.
- Add unit tests to the `testclass` include — it's a dedicated file alongside the class.
- Use the MCP server's test-run tool rather than asking the user to run tests manually.

---

## Code-quality rule sets

This project relies on the [`claude-abap-skills`](https://github.com/<OWNER>/claude-abap-skills) plugins for detailed rules and slash commands:

- **Clean ABAP** — naming, declarations, expressions, method shape, error handling, class structure.
  - Rules: `clean-abap/CLAUDE.md` in the installed plugin.
  - Commands: `/clean-abap:review`, `/clean-abap:refactor`.
- **ABAP Cloud / RAP** — RAP BO design, ATC remediation, clean-core compliance.
  - Rules: `abap-cloud-rap/CLAUDE.md` in the installed plugin.
  - Commands: `/abap-cloud-rap:rap-bo-design`, `/abap-cloud-rap:atc-remediation`, `/abap-cloud-rap:clean-core-check`.

Apply the plugin rule sets as defaults for any ABAP code generated or edited, not just when a slash command is invoked.

---

## Out of scope — never suggest

- SAP ECC
- Classic non-Cloud ABAP development model (any release)
- Classic dynpro (SE51), screen exits, GUI status programming
- Logical databases, SAP Query, classic batch input
- Unreleased SAP APIs, function modules, classes, or CDS entities
- `SELECT` directly from SAP-owned application tables (e.g. `VBAK`, `MARA`)
- BAPI-style RFM modules where a released cloud API exists
- `FORM` / `PERFORM` routines, includes-only programs, classic exception groups
- `CALL FUNCTION ... DESTINATION` over released communication scenarios

If legacy code surfaces, recognise the pattern, name it, and propose a modern Cloud-compatible alternative — don't extend the legacy pattern.

---

## Operational note — MCP timeout

The RAP generators (`x-ui-service`, `ui-service`) routinely take 60–180 s for a multi-entity BO. If your AI coding agent's MCP client has a short default tool timeout (~30 s is common), raise it to ~10 min (e.g. `MCP_TIMEOUT=600000`) when RAP generation is on the plan.
