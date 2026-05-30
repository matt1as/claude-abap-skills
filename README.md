# claude-abap-skills

**The official SAP ABAP MCP Server gives Claude hands. This library gives it a brain.**

A skill library for [Claude Code](https://docs.claude.com/en/docs/claude-code) targeting **modern ABAP** — SAP BTP ABAP Environment and S/4HANA 2023+ in the ABAP Cloud development model.

---

## What this is

A collection of **always-on rules, prompt templates, and slash commands** that turn raw Claude output into production-quality modern ABAP, RAP, and ABAP CDS.

This is **not** an MCP server. There is no system connector code here. All system access — reading objects, writing source, running ATC, executing ABAP Unit — is assumed to flow through the **official SAP ABAP MCP Server**, distributed as part of the `SAPSE.adt-vscode` VS Code extension.

| Component                                  | Role                                                                         |
|--------------------------------------------|------------------------------------------------------------------------------|
| `SAPSE.adt-vscode` (official SAP MCP)      | **Hands & eyes** — read source, write source, run ATC, run unit tests        |
| `claude-abap-skills` (this repo)           | **Brain** — what to write, what to avoid, how to structure RAP, how to review |

---

## The problem this solves

A connected MCP gives Claude the ability to read any object in your ABAP system and write any object back. That is necessary, but it is not sufficient. Without opinionated guidance, Claude will happily produce:

- `SELECT` from `VBAK` (forbidden in ABAP Cloud — not a released table)
- Unreleased function module calls (will not activate in ABAP Cloud)
- BAPI-style transactional code where RAP is mandatory
- Classic dynpro suggestions that have no place in modern ABAP
- ATC violations that a human would catch in five seconds

**MCP exposes _what_ to access. Skills encode _how_ to write good code.** This library is the second half.

---

## Two-layer architecture

```
┌─────────────────────────────────────────────────────────┐
│  Layer 2 — ABAP Cloud / RAP overlay                     │
│  System-specific, opinionated patterns                  │
│  Clean core • RAP behavior • CDS layering • ATC fixes   │
├─────────────────────────────────────────────────────────┤
│  Layer 1 — Clean ABAP base                              │
│  Universal rules for any modern ABAP                    │
│  Naming • methods • conditionals • errors • classes     │
└─────────────────────────────────────────────────────────┘
```

Both layers load automatically when this library is wired into a project. Layer 1 applies to **all** modern ABAP work. Layer 2 refines and extends Layer 1 for RAP, ABAP Cloud, and clean core compliance.

---

## Supported systems

- **SAP BTP ABAP Environment** (Steampunk)
- **SAP S/4HANA 2023 and later**, when developed in the **ABAP Cloud development model**

Where the two diverge — for example released-API surface, allowed `@AbapCatalog` annotations, ATC variants — the skills call it out explicitly and ask which system you are targeting if unclear.

## Not supported

- SAP ECC
- Classic non-Cloud development model on S/4HANA (any release)
- S/4HANA on-prem releases earlier than 2023
- Classic dynpro, logical databases, SAP Query, classic batch input
- BAPI-style transactional code where RAP is the modern alternative

Pull requests adding content for unsupported scopes will be closed — see `CONTRIBUTING.md`.

---

## Prerequisites

1. **[Claude Code](https://docs.claude.com/en/docs/claude-code)** installed
2. The **official SAP ABAP MCP Server**, distributed as part of the [`SAPSE.adt-vscode`](https://marketplace.visualstudio.com/items?itemName=SAPSE.adt-vscode) extension, installed and connected to your ABAP system
3. A target system that is one of: BTP ABAP Environment, or S/4HANA 2023+

See `docs/mcp-setup.md` for connection details.

---

## Installation

You can wire this skill library into a project in either of two ways.

### Option A — clone next to your project

```bash
git clone https://github.com/matt1as/claude-abap-skills.git
```

Then in your ABAP project's `CLAUDE.md`, point Claude at this library:

```markdown
# Project CLAUDE.md

Load the modern ABAP skill library at `../claude-abap-skills/CLAUDE.md`.
```

### Option B — use as a git submodule

```bash
git submodule add https://github.com/matt1as/claude-abap-skills.git .claude/skills/abap
```

Then reference `.claude/skills/abap/CLAUDE.md` from your project's `CLAUDE.md`.

Once wired in, the two `AGENTS.md` files load automatically and the slash commands below become available.

---

## Available slash commands

### `/clean-abap:review`
Review existing ABAP code against Clean ABAP rules. Outputs a structured report grouped by severity, with ATC-checkable violations first.

### `/clean-abap:refactor`
Refactor existing ABAP code to Clean ABAP style. Explains every change, leaves business logic untouched, and asks before writing back via MCP.

### `/abap-cloud-rap:rap-bo-design`
Design a complete RAP business object — BDEF, CDS layer plan, managed vs unmanaged decision, draft strategy, projections — from a short spec.

### `/abap-cloud-rap:atc-remediation`
Pull ATC results via MCP, group violations by category, and walk through fixes with explanations. Refuses pseudo-comment suppressions unless explicitly justified.

### `/abap-cloud-rap:clean-core-check`
Check an object or package for clean core compliance: unreleased API usage, direct table `SELECT`s on SAP-owned tables, classic constructs disallowed in ABAP Cloud, modifications to SAP standard.

---

## Repo structure

```
claude-abap-skills/
├── CLAUDE.md                          # Top-level entry point Claude reads first
├── README.md                          # This file
├── LICENSE                            # Apache 2.0
├── CONTRIBUTING.md                    # Contribution guide and scope policy
├── skills/
│   ├── clean-abap/
│   │   ├── AGENTS.md                  # Always-on Clean ABAP rules
│   │   ├── review.md                  # /clean-abap:review
│   │   └── refactor.md                # /clean-abap:refactor
│   └── abap-cloud-rap/
│       ├── AGENTS.md                  # Always-on RAP / ABAP Cloud rules
│       ├── rap-bo-design.md           # /abap-cloud-rap:rap-bo-design
│       ├── atc-remediation.md         # /abap-cloud-rap:atc-remediation
│       └── clean-core-check.md        # /abap-cloud-rap:clean-core-check
└── docs/
    └── mcp-setup.md                   # SAP ABAP MCP connection notes (WIP)
```

---

## Contributing

Contributions are very welcome. The bar is **high quality, opinionated, enforced rules** with working ABAP/CDS/BDEF code examples and a clear rationale. See `CONTRIBUTING.md` for the full process and scope policy.

---

## License

Apache 2.0 — see `LICENSE`.

## Author

[Mattias Johansson](https://github.com/matt1as)
