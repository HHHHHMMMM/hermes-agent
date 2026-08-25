---
name: drybulk-lbl-semantic-process-flow
description: "Compose drybulk process graphs from independently reviewed current definitions and confirmed actual-definition drafts."
metadata:
  version: 0.23.0
  author: zbyl-lbl
  platforms: [macos, linux]
  hermes:
    tags: [drybulk, zbyl-lbl, process-graph, define-language]
---

# Drybulk Semantic Process Flow

Compose a process draft from the original request, complete conversation,
confirmed actual-definition draft, and definitions that the backend explicitly
makes available after independent review.

## Directed Host Repair Override

- Inspect `hostValidationFailure` before any contract or capability read. If it
  identifies a direct format/field/JSON-Pointer repair, apply only that repair
  to `hostValidationFailure.rejectedCandidate` when present or to the current
  process artifact. Do not rediscover unchanged evidence or rerun unaffected
  validation merely because the host opened a same-phase correction.
- For a one-JSON format error, return exactly one response-contract JSON object
  and nothing else. Rediscover only when the error lacks a usable target,
  conflicts with evidence, or the repair changes a binding or business meaning.
- If the host identifies a duplicate in `metadata.capabilityRefs`, keep its
  first exact occurrence and remove every later occurrence only. This is a
  metadata cleanup, not a reason to reread contracts or change the graph.

## Entry Gate

- Proceed only when every confirmed actual definition has passed phase-2 review
  and the backend has mounted it as a current session definition.
- Do not invent a definition identity or substitute a similar definition for an
  unresolved slot.
- If a required slot is unresolved, follow the current host continuation or
  blocking response contract. This skill does not edit a mounted definition
  inline; definition repair is performed by returning to the preceding
  standard-definition phase.

## Mandatory Contract Discovery

1. Call `drybulk_query_process_context` once before composing. The backend
   binds the reviewed session definitions from the current Hermes run; never
   copy or resubmit definition JSON from the conversation.
2. Call `drybulk_query_authoring_contract_catalog` with `scope=modeling`.
   Read `authoringContractCatalog` from that authoritative response and
   identify the response, graph, value, and referenced contracts required by
   the current phase.
3. For every required entry, call `drybulk_query_authoring_contract` with its
   `schemaId`. Use the returned self-contained `jsonSchema` and resolve its
   local references inside that document. Fetch a separate catalog entry only
   for an independently validated nested artifact.
   For `drybulk-define-graph/v2`, the returned `semanticAdmissionContract` is
   mandatory: check every applicable `backendValidationChecks` row before
   validation instead of learning semantic rules from backend errors.
4. Use only current host-mounted session definitions and identities returned by
   `drybulk_query_process_context`. Never query or reuse the shared standard library.
   Each process node must copy one exact current `definitionId`; legacy
   assignments, when present, are informational and do not define topology.
5. Do not begin composition from a remembered envelope, field list, schema id,
   previous turn, or example.

## Composition Flow

1. Read the original request, complete conversation, confirmed actual-definition
   draft, run requirements, requested observations, and current definitions.
2. Create exactly the topology expressed by those facts. Do not add, remove,
   reorder, connect, or duplicate business steps to satisfy a familiar process
   pattern or visual convention.
   Mount one independently named business responsibility as one standard-step
   node. Do not put multiple sequential business responsibilities inside one
   standard step merely to make the chain runnable; connect their separate
   nodes in the process graph. Technical support objects internal to one
   responsibility are not separate business steps.
3. Copy definition identities, parameters, ports, connection contracts,
   outputs, and other authoring identities only from current definitions and
   backend tool responses.
4. Bind only values with an explicit current fact source. If a required value
   or relationship is missing or ambiguous, use the question response contract
   fetched for the current phase.
   For every node `parameterBindings[]` entry, first read the matched mounted
   definition parameter. `DimensionlessUnit` uses a bare `scalar` with no unit.
   Every other non-null `unitType` uses a `quantity` with an explicit,
   JaamSim-compatible unit, including zero values; a bare scalar is forbidden.
   Use the canonical symbol for that type: `TimeUnit`=`s` (or confirmed
   `min`/`h`/`day`), `DistanceUnit`=`m`, `VolumeUnit`=`m3`,
   `VolumeFlowUnit`=`m3/s`, `MassUnit`=`kg`, `MassFlowUnit`=`kg/s`,
   `SpeedUnit`=`m/s`, `AccelerationUnit`=`m/s2`, `AngleUnit`=`rad`,
   `AngularSpeedUnit`=`rad/s`, `AreaUnit`=`m2`, `DensityUnit`=`kg/m3`,
   `PressureUnit`=`Pa`, `ViscosityUnit`=`Pa*s`, `EnergyUnit`=`J`,
   `EnergyDensityUnit`=`J/m3`, `SpecificEnergyUnit`=`J/kg`,
   `PowerUnit`=`W`, `CostUnit`=`$`, `CostRateUnit`=`$/s`,
   `LinearDensityUnit`=`kg/m`, `LinearDensityVolumeUnit`=`m3/m`, and
   `RateUnit`=`/s`. `UserSpecifiedUnit` must use the exact unit returned by
   the current input contract; never guess it. Do not use Chinese labels,
   Java class names, or display units as native input units. Treat a missing
   or incompatible unit as a validation defect and repair it before returning
   the graph.
5. Before authoring any node, edge, output, or run operation, read the current
   Hermes-local capability contract opened during phase 1. Use the business
   directories for semantics, `contractObjectDirectory` for every exact
   main/support object, and `contractOutputDirectory` for every exact native
   output and linked business metric. Call
   `listModelingCatalog` exactly once with those exact `objects` and `outputs`.
   Match every context `requiredCapabilityRefs` value against the lightweight
   catalog's `objectRequiredSignatureId`, `inputCatalog.requiredSignatureId`,
   port `requiredSignatureIds`, or output binding `requiredSignatureId`.
   `businessCapabilityId` values are navigation labels only. Then call
   `getModelingBundle` exactly once with every matched object, input, output,
   and `portCatalog.selectionIds` needed by the complete graph. A selection ID
   is authoritative only when copied whole from `portCatalog.selectionIds` in
   `authoringInputId|edgeType` form. Never combine an input ID with a generic
   edge type listed elsewhere in the contract, such as `queueLink`. The bundle's
   returned `requiredSignatureIds` must cover every
   `metadata.capabilityRefs` entry of every selected definition and
   the exact port contract for every authored edge; a run-settings-only bundle
   is incomplete. `metadata.capabilityRefs` is a set, not an event log: merge
   the exact required signature IDs and write each string once, even when the
   same capability is used by multiple nodes or edges. Copy every returned `edgeParameterBindings` entry
   byte-for-byte. For `catalogReference`, graph `catalogInputId` is always the
   canonical FQCN/Keyword value returned in `edgeParameterBindings`, for example
   `com.jaamsim.ProcessFlow.Server/WaitQueue`; the short
   `authoringInputId` such as `Server/WaitQueue` is only a catalog/bundle
   selection identity and must never be written into the graph binding.
   For definition outputs, keep the selected bundle binding's
   `authoringSourceTemplate`: copy its short `nativeOutput` value into
   `source.outputRef`. A full `outputId` such as
   `com.jaamsim.ProcessFlow.EntitySink/NumberAdded` is lookup/validation
   identity only and must never be used as `source.outputRef`.
   Keep the returned source `OUT` and target `IN` roles unchanged.
   Inspect the snapshot only by passing its opaque `snapshotRef` to
   `drybulk_read_jaamsim_capability_snapshot`. Reuse
   `bundleRef`/`coveredRef` on `ALREADY_COVERED`; do not use legacy single-item
   operations during normal authoring. Never use `execute_code`, a terminal,
   `read_file`, `search_files`, a model-controlled path, or arbitrary code. If
   the snapshot reference must be recovered, call
   `drybulk_query_jaamsim_capability_catalog` with an empty object; it only
   reopens the installed local contract and never downloads it from Java.
6. Map every confirmed run-policy value through the authoritative
   `runAuthoringContract` returned by process context. Always emit exactly one
   `processGraph.runSettings[]` entry using its `graphRunSettingKey`. For
   `HOST_ONLY`, do not query or author a native JaamSim run input. In particular,
   `replications` must never become `StartingRunNumber`, `EndingRunNumber`, or a
   run-index input, and `randomSeed` must never become `GlobalSubstreamSeed`.
   Only `HOST_AND_NATIVE` also creates a runPlan operation. For that operation,
   copy only the graph-scoped `authoringOperationTemplate` returned by the
   batch result; bind its value from the same confirmed fact, copy its
   `catalogInputId` byte-for-byte, and never invent a run input identity or
   typed-value shape. A directory candidate, first match, or similar label
   alone is not authoring evidence.
   Confirmed semantic `duration` maps to host key `workHours` and also to the
   exact native `Simulation/RunDuration` operation with the same duration.
   Both entries are mandatory, use the same finite positive quantity, the unit
   is exactly `h`, and their `factRefs` sets are identical.
   `replications` is also mandatory and must be a scalar integer in
   `1..10000`. `randomSeed`, `sampleIntervalSeconds`,
   `frameMaxReplications`, and `frameMaxEntities` use scalar integer values,
   never quantity values with units; apply the exact bounds in
   `runAuthoringContract.settings[].valueContract`.
   Graph-level `randomSeed` is only the optional host base seed. Never map it
   to a distribution object's native `RandomSeed`, never add native
   `RandomSeed` operations to definitions, and never ask the user for
   per-distribution seeds. Inputs marked `HOST_ASSIGNED` are injected by the
   backend as distinct values for every distribution and replication.
7. Build the process artifact exactly against the fetched `jsonSchema`. Do not
   invent fields, aliases, edge semantics, provider behavior, defaults, hidden
   nodes, or compiler behavior.
8. Treat disconnected, cyclic, or otherwise non-linear topology according to
   the confirmed business model and backend contracts, not generic assumptions
   about what a process should look like.
9. Preserve each definition output's stable business metric identity. Start
   with `selectedOutputs` empty; every output and collection mode declared by a
   mounted definition is automatically materialized for every node using that
   definition. `selectedOutputs` is only an optional fact-reference assertion
   list, not an enable/disable list. Add an assertion only when a confirmed fact
   must be attached to that exact node/output/mode identity.
   If any definition declares `sampledMetric`, include the explicit positive
   scalar-integer `sampleIntervalSeconds` run setting in `1..2147483647` seconds required by the fetched host contract; quantity kind and units are forbidden.
   If any definition declares `frameMetric`, include
   `frameMaxReplications=1` and an explicit positive
   scalar-integer `frameMaxEntities` in `1..2147483647`; quantity kind and units are forbidden. Do not author `eventTraceMode`; the host automatically
   enables the complete event lane for `eventDerived`. These are complete-model
   run settings and must never be written into a standard definition.
9. Create `objectives` only from confirmed business targets. Each objective
   references the exact host-materialized series id made from node id, output
   key, and collection mode. Write `targetValue` as the plain numeric value in
   that business output's published display unit. Never submit `unit`, a
   JaamSim unit class, or a typed quantity for an objective; the backend derives
   the display unit, native unit, and conversion from `seriesId`. If no target
   was confirmed, emit no objective.
10. Never call `drybulk_cleanup_jaamsim_capability_snapshot`. The backend owns
    retention and cleanup after durable terminal processing.

## Capability and Evidence Boundary

- Definition availability is established only by the current context and
  definition query.
- Native authoring capability is established only by the current pinned-JAR
  snapshot and the definitions returned by the backend for this session.
- Snapshot objects never determine topology. They describe independent safe
  authoring mechanics only.
- A stored definition, prior successful run, remembered component, or similar
  scene is not current evidence.

## Validation and Repair

1. Call `drybulk_validate_process_graph` exactly as its current tool schema
   requires before returning a draft. Copy each validation `schemaId` and
   `schemaHash` from the fetched contract's `validationIdentity`, not its
   `jsonSchemaIdentity`. Validate against the current session definitions;
   never construct or send a `BusinessModelSpec`. Return the exact validated
   `processGraph`; if any field changes after the successful tool call, validate
   the complete changed graph again before returning it.
2. Do not send definitions or assignments to the validation tool. The backend
   uses the same run-bound definitions that produced `allowedStepSelections`.
3. For every graph node, copy `definitionId`, `definitionVersion`, and
   `definitionHash` from one matching current `allowedStepSelections` row.
4. On failure, read the diagnostics and fetched contracts, repair only the
   diagnosed structure or binding, and rerun validation.
   If validation sets `repairTarget=STANDARD_DEFINITION`, stop process-graph
   repair and follow the host transition back to standard-definition repair.
   After the corrected definitions are remounted, rebuild this process graph
   with their new identities and rerun complete-model validation.
   Validation is the release gate: do not return a draft while
   `technicalAdmissionStatus=FAIL`; native compile, headless run, output
   materialization and application-projection failures remain blocking.
5. If repair requires a missing business fact, return the fetched question
   response. If current evidence proves an unsupported capability, return the
   fetched capability-gap response.
6. Never bypass validation, silently weaken the confirmed intent, or expose
   internal tool failures to the business user.

## Final Response

- Return exactly one JSON object and no markdown, prose, or code fence.
- The response and every nested artifact must validate completely against the
  `jsonSchema` fetched for the current phase.
- Emit no field, identity, value, or wrapper absent from the fetched contracts
  and current backend evidence.
