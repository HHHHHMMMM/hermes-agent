---
name: drybulk-lbl-assist-modeling
description: "Unified drybulk assisted modeling skill for actual-definition clarification, confirmation, independent review, process drafting, validation repair, and capability gaps."
metadata:
  version: 0.25.0
  author: zbyl-lbl
  platforms: [macos, linux]
  hermes:
    tags: [drybulk, zbyl-lbl, assist-modeling, define-language]
---

# Drybulk Assisted Modeling

Guide one user-facing modeling conversation. The backend owns phase identity,
confirmation revisions, routing, contract versions, validation, compilation,
persistence, and UI rendering. The AI returns only the response allowed by the
current phase.

## Source of Truth

- Use only the original request, the complete current conversation, structured
  host context, current authoring schemas, and the current JaamSim capability
  snapshot.
- Do not generate or reconstruct `BusinessModelSpecIntent`,
  `BusinessModelSpecPreview`, `BusinessModelSpec`, or `standard_step_drafts`.
  They are not part of this workflow.
- Do not query, compare, merge, or reuse shared standard definitions or old
  modeling runs. Every definition belongs to this conversation.
- Ask only for missing business facts actually consumed by the requested model.
  Never invent defaults, topology, parameters, outputs, or run settings to make
  validation pass.
- A positive replication count and a finite duration with a unit are required
  execution facts. Ask for either when missing. Ask for sampling or replay
  limits only when the selected output mode requires them.
- Preserve confirmed stochastic behavior exactly. For example, a Poisson
  arrival process cannot be silently reduced to a deterministic mean interval.
- Omit every native input whose `platformAuthoringPolicy.status` is
  `HOST_ASSIGNED`. In particular, never ask the user for a distribution
  object's `RandomSeed` and never author it in a definition. The backend
  assigns distinct distribution seeds from the optional graph-level
  `randomSeed`.

## Contract and Capability Discovery

## Directed Host Repair Override

- Inspect `hostValidationFailure` before any discovery. If it gives a specific
  repair target (one-JSON response formatting, a JSON Pointer/field, expected
  value, or repair hint), repair only that target using
  `hostValidationFailure.rejectedCandidate` when present, otherwise the
  current phase artifact in context. Do not read a contract, capability
  catalog, snapshot, or rerun an unaffected validation/compile gate merely
  because this is a repair invocation.
- For a one-JSON response error, return exactly one response-contract JSON
  object with no prose, Markdown, code fence, second object, or trailing text.
- Re-enter discovery only when the host error has no actionable target, its
  evidence conflicts, or the targeted repair would change business behavior,
  a technical binding, or its validation basis. Then read only the evidence
  and rerun only the gates affected by that repair.

1. Read `v2Orchestration.phase` and its response contract before drafting.
   A response never changes its own phase.
2. In every initial or evidence-ambiguous `CLARIFICATION` invocation, call
   `drybulk_query_standard_step_generation_context`, then call
   `drybulk_query_authoring_contract_catalog` with `scope=modeling`. Copy the
   exact IDs from its `authoringContractCatalog`, and fetch all three of these
   schemas separately with `drybulk_query_authoring_contract`:
   `drybulk-clarification-response/v1`,
   `drybulk-definition-draft/v1`, and `drybulk-define/v2`.
   All three calls are mandatory before returning `questions`,
   `definition_draft`, or `capability_gap`; following a `$ref` from one fetched
   schema does not count as fetching another schema.
   `drybulk-define/v2` is deliberately sectioned: its contract query returns
   `contractSectionCatalog`, not the former 78K aggregate. Before returning a
   `definition_draft`, use `drybulk_query_authoring_contract_section` to read
   `jsonSchema.root`, `jsonSchema.$defs`, `semanticAdmission.core`,
   `semanticAdmission.semanticRules`, and
   `semanticAdmission.backendValidationChecks.STANDARD_DEFINITION_DRAFT`.
   During phase-2 review, read those sections again plus
   `semanticAdmission.backendValidationChecks.HOST_COMPILE_GATE`. Copy section
   identities exactly, inspect one returned section before requesting the
   next, and never reconstruct or request the former aggregate.
   Check every row in the applicable backend-validation section; the broader
   `semanticRules` summaries do not replace those rows.
   Use `validationStage` literally: satisfy `STANDARD_DEFINITION_DRAFT` before
   showing the draft for confirmation, satisfy `HOST_COMPILE_GATE` during the
   independent phase-2 review after confirmation, and never move a
   `PROCESS_GRAPH_MATERIALIZATION` value into the reusable definition.
3. Read `backendCapabilityCatalog` from the standard-step context and fetch
   only the exact current sections needed for the artifact with
   `drybulk_query_backend_capability`. A definition draft must read
   `defineOperationRegistry`, `defineAuthoringSemantics`, and
   `standardCompileWitnessContract`; read port, output, collection, run, or
   other sections only when the requested artifact consumes them. Never infer
   section content from its name or from a previous turn.
4. Also at the start of the first phase, call
   `drybulk_query_jaamsim_capability_catalog` once with an empty object. It
   opens the complete capability contract already installed in Hermes local
   storage. `businessCapabilityDirectory` and `businessOutputDirectory` are
   semantic navigation. `contractObjectDirectory` contains every main and
   support object in the contract, and `contractOutputDirectory` contains every
   native output plus any linked `businessMetricIds`. It never downloads the large contract from Java.
5. Inspect that contract only with
   `drybulk_read_jaamsim_capability_snapshot`, passing only its opaque
   `snapshotRef`. Use the business directories to understand the requested
   semantics, then select every main and support object by exact
   alias/nativeTypeId from `contractObjectDirectory`. Select registered
   `businessMetricId` values through `contractOutputDirectory` and
   `businessOutputDirectory`. In each phase, call
   `listModelingCatalog` once with those exact `objects` and `outputs`, then
   copy exact input and required `portSelections` identities from that result
   into one
   `getModelingBundle` call. A port selection must be copied exactly from
   `portCatalog.selectionIds` in `authoringInputId|edgeType` form. That list
   is authoritative: copy one whole value byte-for-byte. Never combine an
   input ID with a generic edge type elsewhere in the contract (for example
   `queueLink`), because it is not a permitted selection for that input.
   Never use a
   path, terminal, code execution, file reader, or arbitrary code. A repeated
   catalog call only reopens the same installed local contract.
   Copy `catalogVersion` and `catalogDefinitionHash` from the returned
   snapshot identity. Every reader result repeats this identity; never reuse
   one from another session.
   `listModelingCatalog` is the lightweight technical catalog for the exact
   directory selections. `getModelingBundle` returns each selected object, dependency,
   input, port, and output contract once with a stable `coveredRef`. Repeated
   reads in the same phase return `ALREADY_COVERED`; reuse
   `bundleRef`/`coveredRef` and do not request the contracts again. The legacy
   single-item operations are diagnostic only and must not be used in normal
   authoring.
6. Re-query the relevant context, catalog, schemas, and backend capability
   sections after every backend phase change
   or repair transition. Never rely on remembered fields, hashes, examples, or
   candidate shapes.
7. Start every new draft by selecting object and `businessMetricId` values from
   the business navigation and complete contract directories. Do not stop at
   the main process objects: include every technical support object needed by
   the confirmed behavior, such as the exact distribution object for a
   stochastic interval. Pass those exact values to one
   `listModelingCatalog` call, then select `authoringInputId` and required
  `portCatalog.selectionIds` only from that result, never by combining an
  input ID with a generic edge type. Fetch the selected
   contracts in one `getModelingBundle` call.
   Treat `businessCapabilityId` as navigation only; never copy it into
   `metadata.capabilityRefs`. That field must contain only exact
   `requiredSignatureIds` returned by the batch contract. Treat
   `metadata.capabilityRefs` as a set: merge all required signatures for the
   authored graph, keep each exact string once, and remove duplicates before
   returning JSON. Repeating the same signature for multiple nodes or edges is
   never valid and does not add evidence.
   For each created object, close every required
   input as graph-owned, business-bound, internal-support, or unresolved.
   Never infer completeness from a preview or from a successful compile gate.
   Never report a capability gap for an object or native output until its exact
   ID has been checked in `contractObjectDirectory` or
   `contractOutputDirectory` and the relevant contract has been included in
   the batch.
   Never probe or guess an input such as `Server/Capacity`.
   Map each user-requested or runtime-required output from the bundle's stable
   `businessMetricId`. Set
   `outputKey` to that ID and copy one exact native `bindings` entry or exact
   `derivedBindings` or `artifactBindings` entry. For `eventDerived` and
   `frameMetric`, copy the published artifact kind, selector/unit rule, and
   authoring rule exactly. Provider expression authoring is disabled: always
   omit `expression` and `expressionBindings`.
   For every native or derived binding, build `source` by copying that
   binding's `authoringSourceTemplate`. Replace only its `objectRef` placeholder
   with the selected createObject ref. `outputRef` must be copied from the
   template and therefore equals the short `nativeOutput` (or
   `numeratorNativeOutput`) such as `NumberAdded`. `outputId` and
   `numeratorEffectiveOutputId` are full identities used only for lookup and
   validation; never write either one into `source.outputRef`.
   `process.cycle.duration` uses `minute` for both selector duration and output
   unit. An endpoint scoped to a local object or port must match this
   definition's `metadata.eventTrace.events`; node/nodeDefinition scopes may
   point to another drafted step and are resolved as an exact combination when
   the process graph is built. Never invent a derived formula or artifact selector. Never declare every
   reportable object output merely because it is available.
8. Internal-support dependencies are not user decisions. Use only an exact
   compatible alias and operation returned by the snapshot. In particular,
   when the exact `EntityGenerator/PrototypeEntity` contract lists `SimEntity`
   as compatible, create and bind a local `SimEntity`; ask the user only about
   generated-entity attributes that change requested business behavior.

## Phase 1: Clarify and Draft Actual Definitions

- `CLARIFICATION` may return only `questions`, `definition_draft`, or the
  fetched `capability_gap` branch.
- When the host returns from a completed process to `CLARIFICATION` after a new
  user message, treat earlier confirmed definitions and process drafts as
  history only. Re-clarify the changed request and require a new draft
  confirmation before phase 2.
- Design against the real `drybulk-define/v2` schema and current capability
  evidence while clarifying. This is how you determine which parameters,
  values, units, connections, run settings, and outputs are still missing.
- If a consumed fact is missing or ambiguous, return `questions`. Each prompt
  must be one direct business-language question; do not expose schema names,
  JSON pointers, slot keys, or other internal identifiers.
- When the model is fully specified, return `definition_draft`. Its
  `definitions` are the actual definitions intended for execution, not an
  intent or preview. Its `modelingLogicDescription` must accurately state the
  business behavior, values and units, connections, run requirements, outputs,
  stochastic behavior, and any explicit assumptions represented by those
  definitions. It is presentation text only: use only user-visible business
  language. Never write an internal metric/type/schema/input/port ID, Java
  class name, JSON field, reference, or dotted identifier such as
  `process.cycle.duration`; write the corresponding business label instead.
- Give every drafted definition a unique schema-valid temporary `id` and set
  `definitionVersion` to the schema-required initial value `0`. The backend
  replaces the temporary `id` with a durable session-owned ID before displaying
  the confirmation card; never predict or reuse that durable ID.
- Before returning `definition_draft`, complete the exact dependency-closure
  check for every created object. An unresolved technical dependency is a
  capability gap, not a hidden default and not a business clarification.
- For every consumed SESSION_BOUND business value, declare a required parameter
  with nonempty `factRefs`, `defaultClassification=NO_DEFAULT`,
  `defaultValue=null`, and `defaultValueBinding=null`. The definition keeps fact
  identity only in `factRefs`; the later process node supplies both the typed
  concrete typed value in `parameterBindings`; keep source IDs in `factRefs`.
  `DimensionlessUnit` later uses a bare `scalar`; every other non-null
  `unitType` later uses a `quantity` with an explicit compatible JaamSim unit,
  including a zero-valued time or length. Use the canonical unit mapping in
  the process-flow skill (for example time `s`/`min`/`h`/`day`), never a
  Chinese label, Java class name, display unit, or guessed symbol.
- Split the confirmed workflow by business responsibility: every independently
  named business link must become its own standard definition. Do not hide a
  sequence of business links inside one definition merely to make a runnable
  chain; connect the resulting standard-step nodes in the process graph.
  Technical support objects needed solely inside one business link stay inside
  that link's definition.
- `factRefs` is an unordered unique-ID set. The backend canonicalizes ordering;
  never spend a repair round sorting it.
- A RATIO output keeps its denominator graph-owned: declare a required
  fact-bound parameter, reference it with `denominatorParameterKey`, and set
  `bindingHash=null` for backend materialization. Never place the concrete
  denominator value in the definition.
- Do not call the response confirmed. The backend assigns durable definition
  IDs and a draft hash, displays the actual draft, and waits for the structured
  `CONFIRM_MODEL` action.

## Phase 2: Independent Definition Review

- `STANDARD_DEFINITION` is a fresh review invocation. Re-read the original
  request, complete conversation, host-confirmed draft, the same capability
  snapshot, and the freshly fetched response and definition schemas.
- Review the whole confirmed draft. Keep exactly the same definition ID set.
- Independently rebuild the exact dependency-closure check for every created
  object. A prior validation or compile-test PASS is not proof that required
  inputs, port-owned bindings, and internal support objects are complete.
- Return `PASS` with an empty `definitions` list when no change is needed; the
  backend reuses the exact confirmed definitions instead of asking you to copy
  them.
- Return `TECHNICAL_REPAIRED` only for meaning-preserving corrections such as a
  schema shape, type, capability identity, compiler binding, or other technical
  defect. Validate and compile-test every returned definition.
- If a correction would change a business quantity, process meaning,
  connection, run requirement, requested output, or other user-visible
  behavior, return `BUSINESS_REVISION_REQUIRED` with direct questions and no
  definitions. Do not make the change silently.
- The backend decides the transition. `BUSINESS_REVISION_REQUIRED` returns to
  phase 1 and requires a new confirmation. AI output never performs a jump.

## Phase 3: Process Generation

- Start only after the backend mounts the phase-2 definitions in
  `sessionDefinitions` and moves to `PROCESS_DRAFT`.
- Query `drybulk_query_process_context`, fetch the current process response and
  graph schemas, and use the semantic-process-flow skill.
- Compose topology from the original request, complete conversation, and
  confirmed actual-definition draft. Each node copies a current
  `definitionId`; do not construct a parallel business specification or use a
  shared definition.
- Validate with `drybulk_validate_process_graph`. If the backend reports
  `repairTarget=STANDARD_DEFINITION`, stop graph repair; the backend returns to
  phase 2. After corrected definitions are remounted, regenerate and validate
  the process. The final `processGraph` must be the exact graph that received a
  successful validation result. If any field changes after validation, validate
  the complete graph again before returning it.
- Never call `drybulk_cleanup_jaamsim_capability_snapshot`. Snapshot retention
  and deletion are owned by the host after durable terminal processing.

## Validation Rules

- Copy validation `schemaId` and `schemaHash` from the fetched contract's
  `validationIdentity`, never its `jsonSchemaIdentity`.
- Phase 2 must call `drybulk_validate_standard_step` and then
  `drybulk_compile_test_standard_step` for every definition. A compile fixture
  is host-only evidence and must not be copied into the response.
- Repair only diagnosed technical defects. Never delete, weaken, replace, or
  defer confirmed behavior to make a gate pass.
- If a real business fact is missing, use the current question branch. If the
  capability contract proves the requested behavior unsupported, use the
  current capability-gap branch. Never expose internal failures to the user.

## Final Response

- Return exactly one JSON object and no markdown, prose, or code fence.
- The whole object must validate against the schema fetched for the current
  phase. Emit no undeclared alias, field, value, wrapper, or stale identity.
- Do not use a terminal, code runner, or file tool to construct or syntax-check
  the final JSON.
