# claude-abap-skills

**The official SAP ABAP MCP Server gives Claude hands. This library gives it a brain.**

A Claude Code marketplace shipping two plugins for modern ABAP development — SAP BTP ABAP Environment and S/4HANA 2023+ in the ABAP Cloud development model.

---

## What this is

A collection of **always-on rules, prompt templates, and slash commands** that turn raw Claude output into production-quality modern ABAP, RAP, and ABAP CDS.

This is **not** an MCP server. There is no system connector code here. All system access — reading objects, writing source, running ATC, executing ABAP Unit — is assumed to flow through the **official SAP ABAP MCP Server**, distributed as part of the `SAPSE.adt-vscode` VS Code extension.

| Component                                  | Role                                                                         |
|--------------------------------------------|------------------------------------------------------------------------------|
| `SAPSE.adt-vscode` (official SAP MCP)      | **Hands & eyes** — create/activate objects, run ATC, run unit tests, generate RAP services |
| `claude-abap-skills` (this marketplace)    | **Brain** — what to write, what to avoid, how to structure RAP, how to review |

---

## The two plugins

### `clean-abap`
Universal Clean ABAP rules (auto-loaded into each skill) plus two skills:

- **`/clean-abap:review`** — review existing ABAP against the rule set, structured report grouped by severity
- **`/clean-abap:refactor`** — refactor to Clean ABAP style in deterministic passes, behavior-preserving, asks before writing back

### `abap-cloud-rap`
ABAP Cloud / RAP overlay (auto-loaded) plus three skills:

- **`/abap-cloud-rap:rap-bo-design`** — design a complete RAP business object end-to-end; drives the SAP ADT MCP `x-ui-service` / `ui-service` / `webapi-service` generators
- **`/abap-cloud-rap:atc-remediation`** — walk ATC violations methodically, grouped by category, refuses pseudo-comment suppressions by default
- **`/abap-cloud-rap:clean-core-check`** — audit objects for clean core compliance (released APIs, no SAP table reads, no classic constructs, no SAP-standard modifications)

The two layers compose: **Clean ABAP rules apply first**, then the Cloud/RAP overlay refines them.

---

## Supported systems

- **SAP BTP ABAP Environment** (Steampunk)
- **SAP S/4HANA 2023 and later**, when developed in the **ABAP Cloud development model**

Where the two diverge — released-API surface, allowed `@AbapCatalog` annotations, ATC variants — the skills call it out explicitly and ask which system you are targeting if unclear.

### Not supported

- SAP ECC
- Classic non-Cloud development model on S/4HANA (any release)
- S/4HANA on-prem releases earlier than 2023
- Classic dynpro, logical databases, SAP Query, classic batch input
- BAPI-style transactional code where RAP is the modern alternative

PRs adding content for unsupported scopes will be closed — see `CONTRIBUTING.md`.

---

## Prerequisites

1. **[Claude Code](https://docs.claude.com/en/docs/claude-code)** installed
2. The **official SAP ABAP MCP Server**, distributed as part of the [`SAPSE.adt-vscode`](https://marketplace.visualstudio.com/items?itemName=SAPSE.adt-vscode) extension, installed and connected to your ABAP system. Enable it under VS Code settings: `adt.mcpServer.enabled: true`
3. A target system: BTP ABAP Environment or S/4HANA 2023+ in Cloud development model

See `docs/mcp-setup.md` for connection details.

---

## Install

Two commands. Both plugins, available in every Claude Code session.

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/matt1as/claude-abap-skills

# 2. Install both plugins (or just one — they work independently, but pair best together)
claude plugin install clean-abap
claude plugin install abap-cloud-rap
```

Verify:

```bash
claude plugin list
```

You should see both plugins listed. The five skills auto-register and become invokable as `/clean-abap:review`, `/clean-abap:refactor`, `/abap-cloud-rap:rap-bo-design`, `/abap-cloud-rap:atc-remediation`, `/abap-cloud-rap:clean-core-check`.

### Important — `MCP_TIMEOUT`

The `rap-bo-design` skill drives the SAP ADT MCP's RAP generators, which routinely take 60–180 s for a multi-entity BO. Claude Code's default MCP tool timeout (~30 s) is shorter. Launch Claude Code with a longer timeout when generation is on the plan:

```bash
MCP_TIMEOUT=600000 claude   # 10 minutes — plenty of headroom
```

---

## Updating

```bash
claude plugin marketplace update claude-abap-skills
claude plugin update clean-abap
claude plugin update abap-cloud-rap
```

---

## Uninstall

```bash
claude plugin uninstall clean-abap
claude plugin uninstall abap-cloud-rap
claude plugin marketplace remove claude-abap-skills
```

---

## Repo layout

```
claude-abap-skills/
├── .claude-plugin/marketplace.json    # marketplace manifest
├── clean-abap/                        # Plugin 1
│   ├── .claude-plugin/plugin.json
│   ├── CLAUDE.md                      # universal Clean ABAP rule set
│   ├── review/SKILL.md
│   └── refactor/SKILL.md
├── abap-cloud-rap/                    # Plugin 2
│   ├── .claude-plugin/plugin.json
│   ├── CLAUDE.md                      # RAP / ABAP Cloud overlay rule set
│   ├── rap-bo-design/SKILL.md
│   ├── atc-remediation/SKILL.md
│   └── clean-core-check/SKILL.md
├── docs/mcp-setup.md
├── CLAUDE.md
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## Known limitations of the official ADT MCP

`SAPSE.adt-vscode` 1.0 ships an MCP server tagged as "experimental" in its manifest. It is **creation-focused** — strong for bootstrapping a fresh RAP service end-to-end, weak for everything else.

| Capability                                          | Status |
|-----------------------------------------------------|--------|
| Create new objects (class, CDS, BDEF, service def/binding, access control, …) | ✓ |
| Generate complete RAP UI service in one call        | ✓ (`x-ui-service` / `ui-service` / `webapi-service`) |
| Activate objects                                    | ✓ |
| Run ABAP Unit tests                                 | ✓ |
| Create / inspect transports                         | ✓ |
| Inspect OData service info of existing bindings     | ✓ |
| **Read source of an existing object**               | ✗ |
| **Search the repository**                           | ✗ |
| **Run ATC**                                         | ✗ |

Skills affected: `review`, `refactor`, `atc-remediation`, `clean-core-check` currently work on **pasted** source rather than MCP-fetched source. The `rap-bo-design` skill is a perfect fit and uses the generators end-to-end.

---

## Contributing

Contributions are very welcome. The bar is **high quality, opinionated, enforced rules** with working ABAP/CDS/BDEF code examples and a clear rationale. See `CONTRIBUTING.md` for the full process and scope policy.

---

## License

Apache 2.0 — see `LICENSE`.

## Author

[Mattias Johansson](https://github.com/matt1as)
