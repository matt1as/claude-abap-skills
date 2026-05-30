# /abap-cloud-rap:rap-bo-design

Design a new RAP business object end-to-end and emit skeleton CDS and BDEF code ready to create via the SAP ABAP MCP Server.

## What this command does

You are designing a complete RAP business object from a short spec — not implementing it. The output is a **design document plus skeleton code**: BDEF, interface entity, projection entity, consumption entity, and a behavior pool stub. Every rule from `skills/clean-abap/AGENTS.md` and `skills/abap-cloud-rap/AGENTS.md` is applied throughout.

## Inputs to collect

Before writing anything, confirm or ask for each of the following. **Do not guess silently** — if any answer is unclear, ask. Use a single short bulleted question block rather than multi-turn interrogation.

1. **Entity name** — singular, problem-domain, no `Z` prefix in the design (you will add it when emitting code)
2. **Persistent table name** — `z*` table name, or "to be created" if new
3. **Semantic key field(s)** — the human-readable business key (e.g. `SalesOrder`, `EmployeeNumber`). Must exist alongside the technical UUID.
4. **Composition children** — list of child entity names, or "none"
5. **Associations to other BOs** — list of target entity names and cardinalities, or "none"
6. **Draft required?** — yes / no, with one-sentence reason
7. **Actions needed** — list of non-CRUD actions with cardinality (instance / static / factory) and intent
8. **Target system** — BTP ABAP Environment, or S/4HANA 2023+ in ABAP Cloud development model. Some annotations and released APIs differ.

## Procedure

1. **Confirm inputs.** If anything in the spec is incomplete or ambiguous, ask before continuing.
2. **Decide managed vs unmanaged.** Apply `managed-vs-unmanaged-decision-criteria` from the overlay. State the decision and a one-sentence rationale. New BO on customer tables → managed.
3. **Plan the CDS layer.** Three layers always:
   - **Interface entity** (`ZI_<Entity>`) — pure data shape, includes compositions and associations, all mandatory annotations.
   - **Projection root** (`ZR_<Entity>`) — `provider contract transactional_query`, redirected compositions, no business logic.
   - **Consumption** (`ZC_<Entity>`) — projection of the projection, all UI annotations live here.
   Repeat for every composition child.
4. **Plan the BDEF.**
   - `managed implementation in class zbp_<root> unique;`
   - `strict ( 2 );`
   - `with draft;` if drafts enabled
   - For each entity: `persistent table`, `draft table` if applicable, `lock master total etag <field>` for root, `lock dependent` for children, `authorization master ( instance )`, `etag master <field>`, semantic-key field declared `mandatory`, technical-key field declared `numbering : managed, readonly`.
   - Declare every action with the right kind (`action`, `static action`, `factory action`).
   - Declare determinations, validations, and side effects separately — apply `separate-determinations-validations-and-side-effects`.
   - Declare composition relationships and feature control.
5. **Plan the behavior pool.** One implementation class per entity in the BDEF (`zbp_<entity>`), one local class per concern (validations, determinations, actions). Local handler classes are `FINAL` with `PRIVATE` constructors.
6. **Plan testing.** One ABAP Unit test class per behavior implementation class, using `cl_cds_test_environment`. Apply `every-behavior-class-has-abap-unit-with-cds-test-doubles`.

## Output format

```
# RAP Business Object Design — <Entity>

## Decision summary
- Implementation type: managed | unmanaged — <one sentence why>
- Draft: enabled | disabled — <one sentence why>
- Target system: BTP | S/4HANA 2023+ — <which annotations or APIs differ>

## CDS layer plan
| Layer       | Entity              | Role                                        |
|-------------|---------------------|---------------------------------------------|
| Interface   | ZI_<Entity>         | data shape, compositions, associations      |
| Projection  | ZR_<Entity>         | transactional_query, redirected comps       |
| Consumption | ZC_<Entity>         | UI annotations                              |
(one row per entity including children)

## BDEF skeleton
` ` `abap
<full BDEF source, ready to create as a Behaviour Definition via MCP>
` ` `

## Interface entity skeleton
` ` `abap
<ZI_<Entity> DDL, ready to create>
` ` `

## Projection entity skeleton
` ` `abap
<ZR_<Entity> DDL>
` ` `

## Consumption entity skeleton
` ` `abap
<ZC_<Entity> DDL>
` ` `

(repeat the three CDS blocks for every composition child)

## Behavior pool stub
` ` `abap
<zbp_<root> CLASS DEFINITION + IMPLEMENTATION skeleton, with method stubs
 for every declared determination, validation, and action>
` ` `

## ABAP Unit skeleton
` ` `abap
<local test class skeleton, using cl_cds_test_environment, with one
 placeholder test method per validation>
` ` `

## Next steps
1. Confirm the design.
2. Approve creation via MCP — I will create the database table (if new),
   then the interface, projection, consumption, BDEF, behavior pool, and test class
   in that order.
3. Run ATC after each layer; do not proceed if a layer has unresolved errors.
```

## Hard rules for this command

- **Apply every rule from both AGENTS.md files.** This is not optional. If a rule appears to conflict with the spec, surface the conflict instead of silently violating it.
- **Always emit a semantic key field**, regardless of whether the spec explicitly mentioned it. Apply `semantic-key-alongside-technical-uuid`.
- **Always include `strict ( 2 );` in the BDEF** — required for new RAP behaviors in modern ABAP.
- **Never propose `unmanaged` for a new BO on new tables.** Push back and ask whether the user really has a legacy persistence layer to wrap.
- **Never write back via MCP at design time.** This command produces a design — creation is a separate confirmed step.
- **Call out BTP vs S/4HANA 2023+ differences** wherever they affect the design (released APIs, annotations).
- **No placeholders inside code.** Every emitted snippet is syntactically plausible and ready to paste into ADT.
