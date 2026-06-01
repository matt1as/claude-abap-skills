# `abap-cloud-rap` plugin

ABAP Cloud / RAP rules and three skills for Claude Code. One of two plugins in the [claude-abap-skills](../README.md) repo (the other is [`clean-abap`](../clean-abap/README.md) — independent).

```bash
claude plugin install abap-cloud-rap@claude-abap-skills
```

Independent from the [`clean-abap`](../clean-abap/README.md) plugin — install both for full coverage, but neither loads the other at runtime.

---

## What this plugin gives you

- **15 ABAP Cloud / RAP rules** in [`CLAUDE.md`](CLAUDE.md), covering clean core (released APIs, no SAP-owned table reads, ABAP Cloud language scope, no SAP-standard modification), RAP behavior design (managed vs unmanaged, semantic key, draft handling, determinations/validations/side effects, action design, feature control), CDS layering (interface → projection → consumption, required annotations, no business logic in projections, composition vs association), and testing (ABAP Unit with CDS test doubles). Each rule has concrete CDS/BDEF/ABAP examples and the related ATC checks. Where BTP and S/4HANA on-prem differ, the rule calls it out.
- **Three skills** invokable in any Claude Code session once installed:

| Skill | Invoke when |
|-------|-------------|
| **`/abap-cloud-rap:rap-bo-design`** | The user asks to design, scaffold, generate, or create a new RAP BO, OData service, or Fiori UI service. Prefers the official SAP ADT MCP's `x-ui-service` / `ui-service` / `webapi-service` generators (creates the persistent table, draft table, CDS layers, BDEFs, behavior pool, access control, service def/binding, metadata extension in one call); falls back to hand-crafted CDS/BDEF skeletons when no generator fits. |
| **`/abap-cloud-rap:atc-remediation`** | The user asks to fix, remediate, address, or triage ATC findings on an object, package, or transport. Groups findings by category, proposes a concrete fix per finding, refuses pseudo-comment suppressions by default. |
| **`/abap-cloud-rap:clean-core-check`** | The user asks to audit clean core compliance, ABAP Cloud compatibility, released-API usage, or extension-point usage. Read-only. Distinguishes hard violations (never allowed in either system) from soft warnings (transitional in S/4HANA on-prem, hard in BTP). |

## Scope — targets and out-of-scope

- **Supported:** SAP BTP ABAP Environment, SAP S/4HANA on-prem in the ABAP Cloud development model
- **Not supported:** SAP ECC, classic (non-Cloud) development model, classic dynpro / SUBMIT / FORM / unreleased FMs / direct SAP-table SELECTs

This plugin **enforces** clean core. It will not generate code that violates it. PRs adding content for unsupported scopes will be closed — see [`CONTRIBUTING.md`](../CONTRIBUTING.md).

## Token cost

Run `claude plugin details abap-cloud-rap` to see live numbers. As of v0.2.2:

| Layer | Cost |
|-------|------|
| Always-on (skill descriptions) | ~491 tok |
| `/abap-cloud-rap:rap-bo-design` on-invoke | ~3.5k tok |
| `/abap-cloud-rap:atc-remediation` on-invoke | ~1.5k tok |
| `/abap-cloud-rap:clean-core-check` on-invoke | ~1.6k tok |

The 15-rule CLAUDE.md (~26 KB) is **not** always-on — each skill reads it explicitly when invoked.

## MCP capability note

This plugin assumes the **official SAP ABAP MCP Server** (in `SAPSE.adt-vscode`) for system access — that MCP is creation-focused, which is exactly what `rap-bo-design` needs to drive the generators end-to-end. But the official MCP **does not** expose:

- Object source reads (affects `clean-core-check` — paste-only today)
- Repository search
- ATC results (affects `atc-remediation` — paste-only today)

See the top-level [README — Known limitations](../README.md#known-limitations-of-the-official-adt-mcp) and the per-skill MCP caveats inside each SKILL.md.

The official MCP is tagged `"experimental"` and its capability set is expected to grow. Once source reads and an ATC tool land, `clean-core-check` and `atc-remediation` move from paste-only to live MCP calls without any change to the skill contracts.

### `MCP_TIMEOUT`

The RAP generators routinely take 60–180 s for a multi-entity BO; Claude Code's default MCP tool timeout (~30 s) is shorter. Launch Claude Code with `MCP_TIMEOUT=600000` (10 min) when running `/abap-cloud-rap:rap-bo-design`, or the tool call will report a false timeout while the server keeps generating in the background.

## Files in this plugin

```
abap-cloud-rap/
├── .claude-plugin/plugin.json     # manifest (name, version, license)
├── CLAUDE.md                      # 15-rule ABAP Cloud / RAP rule set
├── README.md                      # this file
├── rap-bo-design/SKILL.md         # /abap-cloud-rap:rap-bo-design
├── atc-remediation/SKILL.md       # /abap-cloud-rap:atc-remediation
└── clean-core-check/SKILL.md      # /abap-cloud-rap:clean-core-check
```

## License

Apache 2.0 — see [`../LICENSE`](../LICENSE).
