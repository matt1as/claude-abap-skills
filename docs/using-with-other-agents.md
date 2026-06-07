# Using these rule sets with other AI coding agents

This repo's rules are packaged as Claude Code plugins (see [README](../README.md)), but the rule content is plain prose — it works in any AI coding agent. This doc shows the manual wiring for **GitHub Copilot** in VS Code and **Cursor**. For agents that only read a workspace `AGENTS.md` (Aider, Codex CLI, …), use [`templates/AGENTS.md`](../templates/AGENTS.md) directly.

The repo itself stays a Claude Code plugin source — there is no build pipeline that emits Copilot/Cursor files. You copy what you need into your ABAP project workspace.

## What's portable, what's not

| Layer | Claude Code | Copilot (VS Code) | Cursor | AGENTS.md-only |
|---|---|---|---|---|
| Rule content (30 rules in the two `CLAUDE.md` files) | `CLAUDE.md` per plugin | `.github/instructions/*.instructions.md` | `.cursor/rules/*.mdc` | `AGENTS.md` |
| Path-scoped (only ABAP files) | implicit | `applyTo:` frontmatter | `globs:` frontmatter | workspace-wide only |
| Slash commands | namespaced (`/plugin:skill`) | flat (`*.prompt.md`) | none (ambient rules only) | none |
| Procedural multi-step skills (e.g. `rap-bo-design`) | first-class | works via prompt files | weaker — relies on ambient rules | not available |
| MCP server for ABAP system access | ✓ | ✓ (VS Code MCP) | ✓ (Cursor MCP) | varies |
| One-command install | ✓ marketplace | copy files manually | copy files manually | copy one file |

The rule **content** carries over verbatim. The slash-command UX downgrades depending on the host.

---

## GitHub Copilot (VS Code)

Copilot reads:

- `.github/copilot-instructions.md` — workspace-wide, always-on
- `.github/instructions/*.instructions.md` — path-scoped via `applyTo:`
- `.github/prompts/*.prompt.md` — invokable as `/prompt-name` in Copilot Chat

### 1. Workspace bootstrap

Copy [`templates/AGENTS.md`](../templates/AGENTS.md) to `.github/copilot-instructions.md`. Trim the VS Code virtual-workspace section if you're not using ADT for VS Code.

### 2. Rule sets, path-scoped

One instructions file per plugin. Frontmatter limits it to ABAP-relevant files; the body is the plugin's `CLAUDE.md` content verbatim.

```markdown
<!-- .github/instructions/clean-abap.instructions.md -->
---
applyTo: "**/*.abap,**/*.acds,**/*.bdef,**/*.cds"
---

<paste contents of clean-abap/CLAUDE.md>
```

```markdown
<!-- .github/instructions/abap-cloud-rap.instructions.md -->
---
applyTo: "**/*.abap,**/*.acds,**/*.bdef,**/*.cds"
---

<paste contents of abap-cloud-rap/CLAUDE.md>
```

### 3. Slash commands

Each `SKILL.md` becomes a `.github/prompts/<name>.prompt.md`. Copilot's prompt namespace is flat, so pick names that don't collide:

| This repo's skill | Copilot prompt file | Invocation |
|---|---|---|
| `/clean-abap:review` | `.github/prompts/abap-review.prompt.md` | `/abap-review` |
| `/clean-abap:refactor` | `.github/prompts/abap-refactor.prompt.md` | `/abap-refactor` |
| `/abap-cloud-rap:rap-bo-design` | `.github/prompts/rap-bo-design.prompt.md` | `/rap-bo-design` |
| `/abap-cloud-rap:atc-remediation` | `.github/prompts/atc-remediation.prompt.md` | `/atc-remediation` |
| `/abap-cloud-rap:clean-core-check` | `.github/prompts/clean-core-check.prompt.md` | `/clean-core-check` |

For each: copy the body of the corresponding `SKILL.md` (drop the YAML frontmatter) into the prompt file. Re-target any references like `../CLAUDE.md` to point at the matching `.github/instructions/<plugin>.instructions.md` file.

---

## Cursor

Cursor reads `.cursor/rules/*.mdc` with frontmatter. `alwaysApply: true` keeps the rule in context for every chat; `globs:` scopes it to files that match.

```markdown
<!-- .cursor/rules/clean-abap.mdc -->
---
description: Clean ABAP — naming, declarations, expressions, method shape, error handling, class structure
globs: "**/*.abap,**/*.acds,**/*.bdef,**/*.cds"
alwaysApply: true
---

<paste contents of clean-abap/CLAUDE.md>
```

```markdown
<!-- .cursor/rules/abap-cloud-rap.mdc -->
---
description: ABAP Cloud / RAP — released APIs, no SAP-table reads, RAP BO design, ATC remediation
globs: "**/*.abap,**/*.acds,**/*.bdef,**/*.cds"
alwaysApply: true
---

<paste contents of abap-cloud-rap/CLAUDE.md>
```

### Slash-command gap

Cursor has no equivalent of Claude Code's `/<plugin>:<skill>`. With the rules above attached, invoke a skill by typing the request naturally:

> Review this class against the Clean ABAP rules.

> Design a RAP BO for material approval with header and items, applying the ABAP Cloud / RAP rules.

Functional but loses the explicit slash-command UX. The procedural skills — `rap-bo-design`, `atc-remediation`, `clean-core-check` — are weaker without a trigger that points the model at the full procedure. For those flows specifically, consider keeping Claude Code available alongside Cursor even if your daily editing happens in Cursor.

---

## AGENTS.md-only agents (Aider, Codex CLI, …)

Drop [`templates/AGENTS.md`](../templates/AGENTS.md) at the workspace root. For the full per-plugin rules, append the contents of `clean-abap/CLAUDE.md` and `abap-cloud-rap/CLAUDE.md` to the relevant section of `AGENTS.md`, or reference them inline in the agent's chat. No slash-command equivalent.

---

## What's next

If you adopt a non-Claude-Code agent as your primary and find the manual sync painful, the next step is a generator: single source of truth in this repo's `CLAUDE.md` / `SKILL.md` files, a build script emits Copilot prompts, Cursor rules, and a populated `AGENTS.md`. Not in this repo today — open an issue if you want to drive it.
