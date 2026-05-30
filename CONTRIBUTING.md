# Contributing to claude-abap-skills

Thank you for considering a contribution. This library exists to make Claude Code produce **production-grade modern ABAP** — that is the only goal. Every change should serve it.

---

## Mission

This is a **high quality, opinionated, enforced rule set** for modern ABAP. It is not a wiki, not a tutorial, and not an exhaustive reference. Rules earn their place by being:

- **Specific** — enforceable by an AI agent without further interpretation
- **Actionable** — there is a concrete thing to do or avoid
- **Justified** — there is a real consequence to ignoring the rule
- **Modern** — applies to BTP ABAP Environment or S/4HANA 2023+ in ABAP Cloud development model

Vague advice ("write clean code", "use OOP") does not belong here.

---

## Rule quality bar

Every rule added to an `AGENTS.md` must include:

1. **A short descriptive name** — `## RULE: use-released-apis-only` style
2. **A one-sentence rationale** — the *consequence* of not following the rule, not a restatement of it
3. **A concrete Do example** — real, syntactically plausible modern ABAP / CDS / BDEF code
4. **A concrete Avoid example** — the specific anti-pattern you are ruling out
5. **An ATC check reference where one exists** — name the check (`code pal for ABAP`, SAP standard ATC, ABAP Cloud development variant). If no ATC check exists, omit the line.

Rules that cannot meet this bar should be rejected. If a rule is real but cannot be expressed concretely, that is a signal it is too soft to enforce — it belongs in a styleguide, not in this library.

### Code example requirements

- ABAP must use modern syntax: inline `DATA(...)`, string templates, `VALUE #( )`, `NEW`, class-based exceptions
- CDS must be **ABAP CDS** syntax (DDL source code, not HANA Calculation View XML)
- BDEF must use **RAP BDEF** syntax (`managed implementation in class ... unique`, etc.)
- Where BTP and S/4HANA 2023+ differ, **call it out explicitly** — never silently pick one
- No pseudo-code, no `...` placeholders in the middle of an example. If the example is too long, simplify the scenario instead

---

## Scope policy — enforced

This library targets **modern ABAP only**. The following are **out of scope** and PRs adding them will be closed without merge:

- SAP ECC content
- S/4HANA on-prem releases earlier than 2023
- Classic (non-Cloud) development model content for any release
- Classic dynpro, logical databases, SAP Query, classic batch input
- BAPI-style transactional patterns where RAP is the modern alternative
- Patterns that rely on unreleased SAP APIs

If you believe a borderline topic belongs here, open an issue first to discuss before opening a PR.

---

## Process

### Reporting issues, requesting rules

Open a GitHub issue describing:
- The specific scenario where current guidance is missing or wrong
- A real code example of the problem
- Which target system it applies to (BTP, S/4HANA 2023+, or both)

### Proposing a new rule or skill

1. **Open an issue first.** Describe the rule, the consequence of violating it, and the system scope. Wait for a thumbs-up before writing a PR — this saves you from writing a PR that gets closed on scope grounds.
2. **Implement the rule** following the quality bar above.
3. **Open a PR** referencing the issue. Keep PRs focused — one rule or one slash command per PR.
4. **Self-review checklist before requesting review:**
   - [ ] Rule name is descriptive and lowercase-kebab
   - [ ] Rationale states the *consequence*, not the rule restated
   - [ ] Do example is real, modern ABAP / CDS / BDEF
   - [ ] Avoid example shows the specific anti-pattern
   - [ ] ATC check named if one exists
   - [ ] System scope called out if BTP and S/4HANA 2023+ differ
   - [ ] No additions outside scope (no ECC, no classic ABAP, no pre-2023)

### Editing existing rules

Existing rules can be edited to:
- Tighten the rationale
- Improve the code examples
- Add an ATC check reference that has since become available
- Clarify a BTP / S/4HANA 2023+ difference

Edits that **weaken** a rule (make it vaguer, remove the example, broaden it into a guideline) need an issue and discussion first.

---

## Slash command contributions

A new slash command must:

- Have a single clear purpose stated in one sentence at the top of the file
- Specify the inputs Claude should ask for if not provided
- Specify the output format
- Explicitly defer system access to the MCP, never invent its own tooling
- Apply the rules from the relevant `AGENTS.md` consistently

---

## License

By contributing you agree that your contribution is licensed under the **Apache License 2.0**, the same license as this project.
