# claude-abap-skills

This repository contains **two independent Claude Code plugins** for modern ABAP development. It does not contain any MCP server code — it contains the rules, prompt templates, and slash commands that turn raw model output into **production-quality modern ABAP**.

> The official SAP ABAP MCP Server (`SAPSE.adt-vscode`) gives Claude **hands and eyes** into the system.
> This library gives Claude the **brain**: how to write good code, not just what to access.

This file describes the library for anyone working **on** it. End users install the plugins via `claude plugin marketplace add` — see `README.md`.

---

## Target systems

This library targets **modern ABAP only**:

- **SAP BTP ABAP Environment** (Steampunk)
- **SAP S/4HANA on-prem**, when developed in the **ABAP Cloud development model**

Everything in this library assumes those two scopes. The guidance is opinionated and intentionally narrow.

---

## Out of scope — never suggest

When working with this library, **never** generate code, refactorings, or recommendations that depend on any of the following:

- SAP ECC
- Classic non-Cloud ABAP development model (any release)
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

Known MCP limitations at the time of writing (`SAPSE.adt-vscode` 1.0):
- No object-source read tool — the source of an existing class/CDS/BDEF cannot be fetched
- No repository search
- No ATC tool

Skills that need read access (review, refactor, atc-remediation, clean-core-check) currently work on **pasted** source.

---

## Repository layout

```
claude-abap-skills/                          # repo root
├── .claude-plugin/
│   └── marketplace.json                     # manifest listing both plugins
├── CLAUDE.md                                # this file
├── README.md                                # install instructions
├── LICENSE                                  # Apache 2.0
├── CONTRIBUTING.md
├── docs/mcp-setup.md
├── clean-abap/                              # Plugin 1
│   ├── .claude-plugin/plugin.json
│   ├── CLAUDE.md                            # universal Clean ABAP rule set
│   ├── review/SKILL.md                      # /clean-abap:review
│   └── refactor/SKILL.md                    # /clean-abap:refactor
└── abap-cloud-rap/                          # Plugin 2
    ├── .claude-plugin/plugin.json
    ├── CLAUDE.md                            # RAP / ABAP Cloud overlay rule set
    ├── rap-bo-design/SKILL.md               # /abap-cloud-rap:rap-bo-design
    ├── atc-remediation/SKILL.md             # /abap-cloud-rap:atc-remediation
    └── clean-core-check/SKILL.md            # /abap-cloud-rap:clean-core-check
```

### How rules reach a skill at runtime

Each plugin's `CLAUDE.md` holds the **rule set**. It is **not** auto-loaded by Claude Code (the validator warns about this — by design). Each `SKILL.md` is responsible for explicitly reading the rule files it needs:

- `clean-abap` skills read `../CLAUDE.md` (their own plugin's rules)
- `abap-cloud-rap` skills read `../CLAUDE.md` (their own plugin's rules)

The two plugins are **independent** — neither reads the other's `CLAUDE.md` at runtime. This is intentional: when plugins are installed via Claude Code's plugin system, each lives in its own versioned subdirectory (e.g. `~/.claude/plugins/cache/claude-abap-skills/clean-abap/0.2.2/`), so cross-plugin relative paths cannot reliably resolve. If a contributor wants a Cloud/RAP skill to also surface Clean ABAP findings, embed the relevant rule inline in `abap-cloud-rap/CLAUDE.md` — do not reach for `../../clean-abap/CLAUDE.md`.

---

## Default behaviour

- **Always prefer released APIs.** Before suggesting any SAP object (CDS entity, class, interface, function module), assume it must have a C1 release contract. If unsure, ask Claude to verify the release state via the MCP before using it.
- **Always prefer modern ABAP constructs**: inline declarations, `VALUE #( )`, string templates, `COND`/`SWITCH`, table expressions, class-based exceptions, `NEW` over `CREATE OBJECT`.
- **Always prefer CDS-based data access** over `SELECT` from physical tables.
- **Always prefer RAP** for any new transactional behaviour. Do not propose hand-written `BAPI`-style update logic.

---

## When context is ambiguous — ask

Several rules in this library differ between **BTP ABAP Environment** and **S/4HANA on-prem in ABAP Cloud development model**. Examples:

- BTP forbids some constructs (e.g. some `@AbapCatalog.sqlViewName` usage on interface views) that S/4HANA on-prem still allows
- The set of released APIs differs between systems
- Some ATC variants behave differently

**If the target system is not clear from context, ask the user before generating code.** Do not silently pick one. A single sentence — "Are you targeting BTP ABAP Environment or S/4HANA on-prem in the ABAP Cloud development model?" — is always the right move when in doubt.

---

## Operational note — MCP_TIMEOUT

The RAP generators (`x-ui-service`, `ui-service`) routinely take 60–180 s for a multi-entity BO. Claude Code's default MCP tool timeout (~30 s) is shorter, producing false "operation timed out" while the server completes. **Launch Claude Code with `MCP_TIMEOUT=600000` (10 min)** when generation is on the plan.

---

## License

Apache 2.0 — see `LICENSE`.
