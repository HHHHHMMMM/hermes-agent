---
name: drybulk-lbl-standard-process-step-generation
description: "Draft and independently review actual drybulk definitions against current schemas, capability evidence, validation, and compile-test gates."
metadata:
  version: 0.24.0
  author: zbyl-lbl
  platforms: [macos, linux]
  hermes:
    tags: [drybulk, zbyl-lbl, standard-step, define-language, compile-test]
---

# Drybulk Standard Definition Draft and Review

Use this skill both while producing the actual pre-confirmation definitions and
while independently reviewing them after user confirmation. There is no intent,
preview, business-spec, or `standard_step_drafts` layer.

## Evidence Boundary

- Use only current user facts, the complete conversation, current host context,
  schemas fetched through `drybulk_query_authoring_contract`, and the current
  immutable JaamSim capability snapshot.
- Never query, compare, transform, or reuse a shared standard-definition
  library, historical model, example, remembered simulator contract, or stale
  snapshot.
- Capability candidates prove available mechanics; they do not choose business
  topology, values, run settings, or requested outputs.
- Copy object, operation, input, port, output, unit, adapter, selector,
  `catalogInputId`, and capability identities byte-for-byte from current
  evidence. Never derive them from labels or validation messages.

## Directed Host Repair Override

- Before any discovery, inspect `hostValidationFailure`. If it names a direct
  repair target (one-JSON format, JSON Pointer/field, expected value, or repair
  hint), use `hostValidationFailure.rejectedCandidate` when supplied, otherwise
  the current confirmed definition/context, and repair only that target.
- A one-JSON format error means return one response-contract JSON object only:
  no prose, Markdown, code fence, second object, or trailing text. Do not
  rediscover contracts, capability snapshots, or rerun validation/compile tests
  when the definition content is unchanged.
- Read fresh evidence and rerun only affected gates only when the failure is
  not specific enough to repair safely, conflicts with existing evidence, or
  changes a definition, binding, business behavior, or validation basis.

## Required Discovery

1. Call `drybulk_query_standard_step_generation_context`. Then call
   `drybulk_query_authoring_contract_catalog` with `scope=modeling` and read
   its `authoringContractCatalog`.
2. In phase 1 (`CLARIFICATION`), copy the exact IDs from
   `authoringContractCatalog` and separately fetch
   `drybulk-clarification-response/v1`,
   `drybulk-definition-draft/v1`, and `drybulk-define/v2` with
   `drybulk_query_authoring_contract`. Read every complete returned
   `jsonSchema`.
   All three calls are required even when the response will be `questions` or
   `capability_gap`; a `$ref` does not replace the explicit contract query.
   In phase 2, fetch the current definition-review response schema and
   `drybulk-define/v2` in the same way.
   `drybulk-define/v2` returns a `contractSectionCatalog` instead of the former
   78K aggregate. Before drafting, fetch its `jsonSchema.root`,
   `jsonSchema.$defs`, `semanticAdmission.core`,
   `semanticAdmission.semanticRules`, and
   `semanticAdmission.backendValidationChecks.STANDARD_DEFINITION_DRAFT`
   sections one at a time with
   `drybulk_query_authoring_contract_section`. During phase-2 review, fetch
   those sections again plus
   `semanticAdmission.backendValidationChecks.HOST_COMPILE_GATE`. Inspect each
   section before requesting the next; never request or reconstruct the former
   aggregate. Check every row in the applicable backend-validation section;
   the broader `semanticRules` summaries do not replace those rows.
   Apply `validationStage` exactly: phase 1 must pass
   `STANDARD_DEFINITION_DRAFT` before confirmation, phase 2 must pass
   `HOST_COMPILE_GATE` during the independent review after confirmation, and
   concrete values marked `PROCESS_GRAPH_MATERIALIZATION` stay for the later
   graph node.
3. Read `backendCapabilityCatalog` from the context. Fetch exact sections with
   `drybulk_query_backend_capability`; every definition draft or review must
   fetch `defineOperationRegistry`, `defineAuthoringSemantics`, and
   `standardCompileWitnessContract`. Fetch the port, output, collection, run,
   or other sections only when the current definition consumes them.
4. During phase 1, open the installed Hermes-local contract through
   `drybulk_query_jaamsim_capability_catalog`. Use its business capability and
   output directories for semantic navigation, its complete
   `contractObjectDirectory` for every main/support object, and its complete
   `contractOutputDirectory` for every native output and linked
   `businessMetricIds`. During phase 2 and repair, recover the same
   `snapshotRef`; never download the large contract from Java.
5. Inspect the snapshot only through
   `drybulk_read_jaamsim_capability_snapshot`: once per phase with
   `listModelingCatalog` using exact `objects` for every main/support mechanic
   and exact registered `outputs` selected from those directories, then once
   with `getModelingBundle` using
   exact input and required `portCatalog.selectionIds` identities copied from
   the lightweight catalog. `portCatalog.selectionIds` is the only authority
   for a port selection: copy one complete `authoringInputId|edgeType` value
   byte-for-byte. A generic edge type elsewhere in the contract (for example
   `queueLink`) is not a permitted selection and must never be combined with
   an input ID. Reuse `bundleRef`/`coveredRef` when
   the reader returns `ALREADY_COVERED`. Never use the legacy single-item
   operations during normal authoring. Never pass a file path or use
   code, terminal, or generic file tools.
   Copy `catalogVersion` and `catalogDefinitionHash` only from the returned
   snapshot identity; every reader response repeats it. Never invent or reuse
   an identity from an earlier session.

## Exact Read and Dependency Closure

- Select exact candidates and `businessMetricId` values from the business
  navigation and complete contract directories. Include support mechanics
  required by confirmed behavior, including exact distribution objects.
  Never report a capability gap until the relevant exact object/native-output
  directory entry has been checked and included in the batch.
  Select `authoringInputId` values and required
  `portCatalog.selectionIds` from `listModelingCatalog`. Never invent them
  from model memory or combine an input ID with a generic edge type. Fetch the
   complete selected evidence in one
   `getModelingBundle`; each object, dependency, input, port, and output appears
   once under `coveredRef`.
- `businessCapabilityId` is a navigation label, never a contract identity.
  Never write it into `metadata.capabilityRefs`. Copy
  `metadata.capabilityRefs` only from the batch result's exact
  `requiredSignatureIds`, keeping each exact ID once. This field is a set, so
  repeated use of one capability must not create repeated array items; the host compile gate will normalize this field from
  the actual compiled plan before mounting the definition.
- An input with `platformAuthoringPolicy.status=HOST_ASSIGNED` is native
  runtime closure owned by the backend, not a business fact or definition
  operation. Omit it from `compilePlan`, do not ask the user for it, and do not
  treat it as unresolved. This applies to every distribution object's
  `RandomSeed`; the backend assigns distinct seeds during isolated compile and
  complete graph compile.
- Use the exact stable `businessMetricId` returned in the bundle as `outputKey`,
  copy one verified native `bindings`, exact `derivedBindings`, or exact
  `artifactBindings` entry. For event/frame outputs copy its artifact kind,
  selector/unit rule, and authoring rule without guessing,
  and declare only outputs explicitly
  requested by the user or technically required at runtime. Do not copy all
  reportable outputs or all supported collection modes. Never invent a derived
  formula not published by `getOutput`. Provider expression authoring is
  `DISABLED`; always omit `expression` and `expressionBindings` and use a
  published native, derived, or artifact binding.
  For a native or derived binding, copy `source` from its
  `authoringSourceTemplate` and replace only the `objectRef` placeholder.
  `source.outputRef` must remain the template's short `nativeOutput` or
  `numeratorNativeOutput`; `outputId`/`numeratorEffectiveOutputId` is only the
  full identity used to locate and validate the contract and must never be
  used as `outputRef`.
  For `process.cycle.duration`, use `minute` for both `durationUnit` and
  `output.unitType`. Local objectRef scopes must match this definition's
  declared events; `portId` scope is unsupported and must be omitted.
  node/nodeDefinition scopes may reference another drafted
  step and will be resolved against that exact mounted definition and event in
  the process graph.
- For every `createObject`, read the selected object's complete entry and make
  a dependency-closure checklist for every input marked required. Classify
  each required input as exactly one of:
  - graph-owned, when an exact current port contract supplies it through the
    confirmed topology;
  - business-bound, when it consumes a confirmed user fact through a declared
    definition parameter;
  - internal-support, when it is a business-neutral technical dependency that
    must be created and bound inside the definition; or
  - unresolved, when current evidence cannot prove one of the above.
- Never ask the user for an internal-support choice. Create it only from an
  exact compatible alias and operation template returned by the snapshot. In
  phase 1, return `capability_gap` when there is no deterministic supported
  choice. In phase 2, never hide the gap behind `PASS`; use the current review
  contract, and request business revision only when supported alternatives
  would change user-visible behavior. Do not invent a technical choice.
- `EntityGenerator/PrototypeEntity` is an internal-support dependency. When
  its exact input contract lists `SimEntity` as compatible, create one local
  `SimEntity` prototype and bind it with the returned `entityRef` operation.
  Its generated entity attributes become business questions only when those
  attributes affect requested behavior.
- Do not duplicate graph-owned bindings inside the definition. For example,
  bind `NextComponent` or `WaitQueue` locally only when the exact current
  authoring contract says the definition, rather than a process edge, owns it.
- `drybulk_validate_standard_step` now enforces this checklist against the
  current snapshot. Do not skip it or treat compile-test PASS/model self-review as
  an equivalent substitute. Phase 1
  must close it before showing a draft, and phase 2 must independently rebuild
  it before returning `PASS` or `TECHNICAL_REPAIRED`.

## Phase 1 Authoring

- Work directly in the actual `drybulk-define/v2` shape. Return the definitions
  inside the fetched `definition_draft` response when every consumed business
  fact is known.
- Give each phase-1 definition a unique schema-valid temporary `id` and set
  `definitionVersion` to the schema-required initial value `0`. The backend
  replaces that temporary `id` with a durable session-owned ID before showing
  the draft; phase 2 must preserve the backend-owned ID exactly.
- If an actual parameter, value, unit, connection, run requirement, output,
  stochastic law, or other behavior is missing or ambiguous, return the
  current `questions` response instead of inventing it.
- The draft's `modelingLogicDescription` is presentation text only. It must
  match the actual definitions exactly, use only user-visible business
  language, and must not hide assumptions or technical substitutions. Never
  put an internal metric/type/schema/input/port ID, Java class name, JSON
  field, reference, or dotted identifier (for example
  `process.cycle.duration`) in it; replace it with its business label.
- The backend assigns durable IDs before displaying the draft. Do not predict a
  durable ID or create a second AI confirmation artifact.

## Phase 2 Independent Review

- Review the original request, all clarification answers, the entire confirmed
  draft, the same snapshot, and newly fetched schemas. Do not merely endorse the
  first response.
- Preserve the exact set of backend-assigned definition IDs.
- Return `PASS` with an empty definitions list when no defect exists. The
  backend reuses the exact confirmed definitions; do not reproduce them.
- Return `TECHNICAL_REPAIRED` only when corrections preserve user-visible
  behavior. Examples include schema shape, type, capability identity,
  operation binding, output adapter, or compiler defects.
- Return `BUSINESS_REVISION_REQUIRED` with direct user questions and an empty
  definitions list if any necessary correction changes business quantity,
  topology meaning, connection, run requirement, output, stochastic behavior,
  or another confirmed decision. The backend then returns to phase 1 and
  requires a new user confirmation.
- AI reports the review outcome; only the backend changes phases.

## Definition Authoring Rules

- A standard definition declares reusable parameters, ports, compile
  operations, output sources, and supported collection modes. Concrete node
  parameter values, process run settings, collection intervals, replay limits,
  objectives, scenarios, and replication identities belong to the process
  graph.
- One independently named business responsibility is one standard definition.
  Do not combine two or more sequential business responsibilities into one
  definition merely to make a complete runnable chain. The process graph owns
  the handoff and flow between those definitions. Technical support objects
  needed only to implement that one responsibility (for example a distribution,
  prototype entity, or internal resource binding) remain inside its definition
  and do not create a separate business definition.
- Expose each consumed business scalar or quantity as a required parameter with
  nonempty `factRefs`, `defaultClassification=NO_DEFAULT`, `defaultValue=null`,
  and `defaultValueBinding=null`. `defaultValueBinding` is not a fact-source
  annotation and must not carry a business fact. In `compilePlan`, consume the
  parameter through `scalarRef` to `$param.<key>`; the later graph supplies the
  typed value through the node's `parameterBindings`; source IDs remain in `factRefs`.
  Only capability-backed business-neutral constants may be inline.
- Treat every `factRefs` array as an unordered set of unique IDs. The backend
  canonicalizes its order; never change or retry a definition only to sort it.
- Add only ports and connections supported by current authoring port contracts.
  Do not invent unsupported companion objects, aliases, cardinality, or
  endpoint roles. Required business-neutral support objects are allowed only
  under the exact-read dependency-closure rules above.
- Add reportable outputs only from current output contracts and copy their
  collection modes and source templates exactly. Output declaration is
  collectible capability, not a presentation selection.
- For a RATIO output, expose the confirmed denominator as a required
  fact-bound parameter and set `denominatorParameterKey` to that parameter.
  Do not put the concrete denominator in the definition. Set `bindingHash` to
  null; the backend materializes it before confirmation.
- `sampledMetric`, `eventDerived`, and `frameMetric` must use the fetched host
  templates and registered selectors. Their run-wide sampling or replay limits
  remain in the process graph.
- Preserve confirmed stochastic behavior. For a confirmed Poisson arrival
  process, create the capability-backed `ExponentialDistribution`, bind its
  mean from the confirmed arrival rate or interval, and bind
  `EntityGenerator/InterArrivalTime` with `distributionRef` exactly as
  `$distribution.<created-ref>`; `$object.<created-ref>` is not valid. Do not
  reduce it to a deterministic scalar interval. If the required distribution
  reference is unsupported, report a capability gap.

## Validation and Compile Test

1. For every phase-2 definition, call `drybulk_validate_standard_step` using
   the definition contract's `validationIdentity`, then call
   `drybulk_compile_test_standard_step`.
2. On a technical failure, use the returned diagnostics and freshly fetched
   schema to repair only the defect, then rerun both gates in order.
3. Never repair by deleting, weakening, substituting, or deferring confirmed
   behavior. A meaning-changing fix requires `BUSINESS_REVISION_REQUIRED`.
4. Host-created compile witnesses and `extensions.compileFixture` are internal
   evidence and must not appear in returned definitions.
5. Keep the shared capability snapshot through phase-2 repairs and later
   process generation; final process success owns cleanup.

## Final Response

- Return exactly one JSON object that validates against the current phase
  response schema, with no markdown, prose, code fence, or undeclared field.
- Copy only identities required by the fetched current-phase schema; in phase 2,
  copy `confirmedDraftHash` exactly from host context. Do not emit candidate
  identity fields.
