---
name: rap-bo-design
description: Design a complete RAP business object end-to-end and create it via the official SAP ABAP MCP. Use when the user asks to design, scaffold, generate, or create a new RAP BO, OData service, Fiori UI service, or RAP managed/unmanaged behavior. Prefers the MCP x-ui-service / ui-service / webapi-service generators; falls back to hand-crafted CDS/BDEF skeletons when no generator fits. Applies the ABAP Cloud / RAP overlay rules in ../CLAUDE.md (loaded on invocation). Targets BTP ABAP Environment and S/4HANA on-prem in the ABAP Cloud development model.
license: Apache-2.0
---

# abap-cloud-rap:rap-bo-design

Design a new RAP business object end-to-end and create it via the SAP ABAP MCP Server.

## What this command does

You are designing a complete RAP business object from a short spec, then creating it in the connected ABAP system. The official ADT MCP Server (in `SAPSE.adt-vscode`) exposes three RAP **generators** that bootstrap a conformant stack in one call — that is the primary path. You then validate the generated stack against the rules in `../CLAUDE.md` and refine it.

The "hand-crafted skeleton" path is a **fallback** for cases the generators do not fit.

## Inputs to collect

Before doing anything, confirm or ask for each of the following. If anything is ambiguous, ask — do not silently pick a default. Bundle the questions in one short turn rather than multi-turn interrogation.

1. **Project name** — short label that becomes the SAP Object Type / Service Definition / Service Binding name (max 24 chars). E.g. `Flight`, `Travel`, `WorkOrder`.
2. **Artifact prefix and suffix** — used for the generated CDS / BDEF / table names. E.g. prefix `Z` → `ZR_Flight`, `ZI_Flight`, `ZFlight` table.
3. **Application type** — `readOnly`, `withDraft`, or `withoutDraft` (transactional). `withDraft` is the default for editable BOs.
4. **Entities** — for each entity:
   - Entity alias (e.g. `Flight`)
   - Composition parent (if a child entity) and cardinality (`toOne` / `toMany`)
   - Field list with:
     - field name
     - data type (`uuid`, `char`, `numc`, `integer`, `decimal`, `date`, `timestamp`, `amount`, `currencyCode`, `quantity`, `unitOfMeasure`, `string`, `boolean`)
     - whether it is a **semantic key** (the human-readable business identifier — required per `semantic-key-alongside-technical-uuid`)
     - length / decimals where relevant
     - linked currency / unit field for amounts and quantities
5. **Package** — `$TMP` for trial / throwaway work, or a real `Z*` package. Real packages need a transport request.
6. **Target system** — BTP ABAP Environment, or S/4HANA on-prem in ABAP Cloud development model. Released-API sets differ.

## Procedure

### Step 1 — pick the generation path

Decision tree, in order:

| Situation                                       | Path                                                                          |
|-------------------------------------------------|-------------------------------------------------------------------------------|
| New BO, no existing persistent table            | **`x-ui-service` generator** — creates persistent table, draft table, CDS interface, projection, BDEF, projection BDEF, behavior pool, access control, service definition, service binding, metadata extension |
| New BO, persistent table already exists         | **`ui-service` generator** — reuses the table, generates the rest of the UI stack |
| New BO, Web API only (no Fiori UI)              | **`webapi-service` generator** — reuses table, generates Web API stack only   |
| Pattern outside the generators (e.g. heterogeneous compositions, non-CDS sources, abstract entities) | **Hand-crafted fallback** — see Step 5 |

Call `mcp__abap-adt__abap_generators-list_generators` once if you want to confirm the available IDs in this system.

### Step 2 — fetch the schema and build the spec

Call `mcp__abap-adt__abap_generators-get_schema` for the chosen generator. The schema returns a `referenceContent` block with a `sessionId` — **use that sessionId verbatim** in the spec you submit. Then build the JSON spec:

- `metadata.package` — the chosen package
- `serviceConfiguration.serviceNaming.projectName` — the project name
- `serviceConfiguration.objectsNaming.prefix` / `.suffix` — for generated artifact names
- `serviceConfiguration.applicationType` — `readOnly` / `withDraft` / `withoutDraft`
- `businessEntities` — one entry per entity, with `entityName`, `compositionParent` (if a child), `compositionCardinality`
- `businessEntitiesFields` — one entry per entity, with the field list

Apply the rules in `CLAUDE.md` while building the spec:
- **`semantic-key-alongside-technical-uuid`** — at least one field per entity must have `isSemanticKey: true`. The generator adds the technical UUID key automatically.
- **`composition-for-children-association-for-references`** — declare child entities with `compositionCardinality` `toOne` or `toMany`; reference data goes via associations in the post-generation refinement, not as composition.
- For amounts and quantities — link `currencyCode` / `unitOfMeasure` to the appropriate currency or UoM field on the same entity.

### Step 3 — request a transport (if needed)

If the package is not `$TMP`, call `mcp__abap-adt__abap_transport-get` for the package (objectType `DEVC/K`, `isCreation: true`). **Ask the user** to pick an existing transport or create a new one — never auto-select.

### Step 4 — generate, activate, verify

**Precondition — MCP client timeout.** Claude Code's default MCP tool timeout (~30 s) is shorter than a full `x-ui-service` run, which routinely takes 60–180 s for a multi-entity BO. The tool call will return "operation timed out" while the MCP server keeps generating in the background. **Before running this step**, ask the user to launch Claude Code with `MCP_TIMEOUT=600000` (10 min). If they cannot restart, proceed anyway — the server will finish — and use the recovery path below to discover what was created.

1. Call `mcp__abap-adt__abap_generators-generate_objects` with the spec from Step 2 and the transport from Step 3.
2. Collect the URIs and names of the generated objects from the response. **The response is the only authoritative source for actual names** — see naming caveat below.
3. Call `mcp__abap-adt__abap_activate_objects` with those URIs.
4. Run the **post-generation review** below.

**Naming caveat — names are not deterministic from the spec.** The generator inserts auto-namespace fragments that are not present in the input spec. Observed pattern: with `prefix: "Z"` and `projectName: "Flight"`, the consumption entity surfaced as `ZC_01FLIGHT` (note the `_01`), the service binding as `ZUI_FLIGHT_O4`. Do not predict names from the spec — use the response. If the response was lost (e.g. client-side timeout), recover by calling `mcp__abap-adt__abap_business_services-fetch_services` with the most likely service binding name (`Z<prefix>UI_<PROJECTNAME>_O4` for V4 UI services); a successful response confirms generation succeeded and surfaces the entity-set names.

### Step 5 — fallback (hand-crafted skeletons)

Only when no generator fits. Emit:
- Interface entity (`ZI_<Entity>`) — `define view entity`, every annotation per `interface-entity-required-annotations`
- Projection entity (`ZR_<Entity>`) — `provider contract transactional_query`, redirected compositions only, no business logic
- Consumption entity (`ZC_<Entity>`) — UI annotations live here
- BDEF — `managed implementation in class zbp_<root> unique; strict ( 2 ); with draft;` if drafts
- Behavior pool class — one per entity, `FINAL`, `PRIVATE`-constructor, local handler classes for validations / determinations / actions

Each is then created via `mcp__abap-adt__abap_creation-create_object` after `abap_creation-run_validation`.

## Post-generation review

Apply the rules from `CLAUDE.md` to whatever the generator produced. The generator emits a conformant skeleton, but the rules are stricter than the defaults.

Check, in order:

1. **BDEF `strict ( 2 );`** — required for new behaviors. Add if missing.
2. **`@AccessControl.authorizationCheck: #CHECK`** on every interface entity. Add if missing.
3. **Semantic key fields** — confirm the right fields are marked as semantic keys; technical UUID is not the semantic key.
4. **Composition vs association** — confirm compositions are used for parent-owned children only; references to other BOs (e.g. Customer, Product) should be associations to released CDS view entities, not compositions or direct table joins.
5. **No business logic in the projection layer** — projections should be pure structural mapping per `projection-views-contain-no-business-logic`.
6. **Determinations / validations / side effects separation** — the generator does not add these; you do, per `separate-determinations-validations-and-side-effects`.
7. **Test class missing** — the generator does **not** create ABAP Unit tests. Add a local test class per behavior implementation, using `cl_cds_test_environment`, per `every-behavior-class-has-abap-unit-with-cds-test-doubles`.

Flag every gap in the output report. Do not silently fix issues that change semantics.

## Output format

```
# RAP Business Object Design — <Project>

## Decision summary
- Generator: x-ui-service | ui-service | webapi-service | hand-crafted
- Application type: readOnly | withDraft | withoutDraft — <one sentence why>
- Target system: BTP | S/4HANA on-prem
- Package: <name> — <transport: TR / $TMP>

## Generated spec
` ` `json
<full JSON spec submitted to abap_generators-generate_objects, including sessionId>
` ` `

## Generated objects
| Type                          | Name                        | URI                              |
|-------------------------------|-----------------------------|----------------------------------|
| Persistent table              | <name>                      | <uri>                            |
| Draft table                   | <name>                      | <uri>                            |
| CDS interface view            | ZI_<Entity>                 | <uri>                            |
| CDS projection view           | ZR_<Entity>                 | <uri>                            |
| Metadata extension            | ZR_<Entity>                 | <uri>                            |
| BDEF                          | ZI_<Entity>                 | <uri>                            |
| Projection BDEF               | ZR_<Entity>                 | <uri>                            |
| Behavior pool class           | zbp_<entity>                | <uri>                            |
| Service definition            | <name>                      | <uri>                            |
| Service binding               | <name>                      | <uri>                            |
| Access control                | <name>                      | <uri>                            |
| Node type / object type       | <name>                      | <uri>                            |

## Activation
<activation result, error count, any failures>

## Post-generation review
| Check                                                                   | Status | Note                        |
|-------------------------------------------------------------------------|--------|-----------------------------|
| BDEF has `strict ( 2 );`                                                | ✓ / ✗  | <if ✗: fix proposed>        |
| Every interface entity has `@AccessControl.authorizationCheck: #CHECK`  | ✓ / ✗  |                             |
| Semantic key fields correctly marked                                    | ✓ / ✗  |                             |
| Composition only for parent-owned children                              | ✓ / ✗  |                             |
| Projection layer has no business logic                                  | ✓ / ✗  |                             |
| Determinations / validations / side effects in place (or noted as TODO) | ✓ / ✗  |                             |
| ABAP Unit test class with CDS test doubles present                      | ✗      | always TODO — generator does not create |

## Next steps
1. Address any ✗ from post-generation review.
2. Add ABAP Unit tests in the behavior pool include (always required, never generated).
3. Preview the service binding to confirm the UI renders.
```

## Hard rules for this command

- **Use the generator when one fits.** A hand-crafted skeleton is the fallback, not the default.
- **Always include `sessionId`** from the schema response in the generation spec — verbatim.
- **Set `MCP_TIMEOUT=600000` (10 min) before launching Claude Code** when generation is on the plan. A 30 s client timeout against a 2 min server operation looks like a failure but is not.
- **Names come from the generation response, not the spec.** The generator adds suffixes (`_01`) and infixes (`UI`) that are not in the input. If the response is missing, recover via `fetch_services` with the conventional binding name pattern, never invent names downstream.
- **Apply every rule from this plugin's `CLAUDE.md`** in the post-generation review.
- **Always emit a semantic key field** for every entity, regardless of whether the spec explicitly mentioned it.
- **Always include `strict ( 2 );` in the BDEF** — add it post-generation if missing.
- **Never propose `unmanaged` for a new BO on new tables.** The generators do not even offer it.
- **Ask before requesting a transport.** Never auto-select an existing TR; never auto-create one.
- **Call out BTP vs S/4HANA on-prem differences** wherever they affect the design (released APIs, annotations).
- **Tests are always TODO.** The generator never emits them; the post-generation review always flags this.
