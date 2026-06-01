# `clean-abap` plugin

Universal Clean ABAP rules and two skills for Claude Code. One of two plugins in the [claude-abap-skills](../README.md) repo (the other is [`abap-cloud-rap`](../abap-cloud-rap/README.md) — independent).

```bash
claude plugin install clean-abap@claude-abap-skills
```

---

## What this plugin gives you

- **15 always-on Clean ABAP rules** in [`CLAUDE.md`](CLAUDE.md), distilled from the [SAP Clean ABAP styleguide](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md) into AI-enforceable form with concrete Do/Avoid ABAP examples and the related ATC checks. Each skill reads the rule set on invocation — not loaded into your session by default.
- **Two skills** invokable in any Claude Code session once installed:

| Skill | Invoke when |
|-------|-------------|
| **`/clean-abap:review`** | The user asks to review, audit, or check ABAP for clean-code violations. Output: a structured report grouped by severity, ATC-checkable findings first. Read-only — never writes back. |
| **`/clean-abap:refactor`** | The user asks to refactor, clean up, modernize, or tidy ABAP. Applies the rules in deterministic passes (naming → declarations → expressions → method shape → errors → classes), asks before writing back via MCP, never changes behavior. |

## Scope — targets and out-of-scope

- **Supported:** SAP BTP ABAP Environment, SAP S/4HANA on-prem in the ABAP Cloud development model
- **Not supported:** SAP ECC, classic (non-Cloud) development model on any release, classic dynpro

The plugin is opinionated and intentionally narrow. PRs adding content for unsupported scopes will be closed — see [`CONTRIBUTING.md`](../CONTRIBUTING.md).

## Token cost

Run `claude plugin details clean-abap` to see live numbers. As of v0.2.2:

| Layer | Cost |
|-------|------|
| Always-on (skill descriptions) | ~255 tok |
| `/clean-abap:review` on-invoke | ~970 tok |
| `/clean-abap:refactor` on-invoke | ~1.3k tok |

The 15-rule CLAUDE.md (~13 KB) is **not** always-on — each skill reads it explicitly when invoked.

## MCP capability note

These skills work best with an MCP that exposes ABAP object-source reads. The official `SAPSE.adt-vscode` 1.0 MCP does not — only creation, generation, activation, and unit-test tools. To use these skills today, paste the source inline (from ADT in VS Code) or attach a community MCP that exposes reads (e.g. [`mario-andreschak/mcp-abap-adt`](https://github.com/mario-andreschak/mcp-abap-adt)). See the top-level [README — Known limitations](../README.md#known-limitations-of-the-official-adt-mcp).

The official MCP is tagged `"experimental"` and its capability set is expected to grow. Once source reads land, these skills pick them up without any change — treat paste-only as a transitional workflow, not a permanent one.

## Files in this plugin

```
clean-abap/
├── .claude-plugin/plugin.json     # manifest (name, version, license)
├── CLAUDE.md                      # 15-rule Clean ABAP rule set
├── README.md                      # this file
├── review/SKILL.md                # /clean-abap:review
└── refactor/SKILL.md              # /clean-abap:refactor
```

## License

Apache 2.0 — see [`../LICENSE`](../LICENSE).
