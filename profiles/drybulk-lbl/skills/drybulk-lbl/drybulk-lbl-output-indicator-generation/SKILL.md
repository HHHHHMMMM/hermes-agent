---
name: drybulk-lbl-output-indicator-generation
description: "Generate reusable drybulk simulation result output indicator definitions from generic result tables, business questions, and explicit current-result data profiles."
metadata:
  version: 0.4.0
  author: zbyl-lbl
  platforms: [macos, linux]
  hermes:
    tags: [drybulk, zbyl-lbl, result-analysis, output-indicator]
---

# Drybulk Output Indicator Generation

Generate one reusable output-indicator definition for the user's current
business question. Use only current result-analysis context and backend policy.

## Source of Truth

- Treat the current user request and current backend context as the only data
  sources.
- Use only the current graph's declared output-series catalog. Do not invent,
  reuse from another scene, or silently substitute a series id.
- A schema declaration shows what may exist; it does not prove that a particular
  value exists in the current result. Require current profile evidence or an
  explicit user choice whenever a concrete value is needed.
- Generate a reusable query and visualization template, never materialized chart
  rows or a snapshot of result data.

## Mandatory Contract Discovery

1. Call `drybulk_query_output_indicator_generation_context` before asking
   questions or drafting an indicator.
2. Call `drybulk_query_authoring_contract_catalog` with
   `scope=output_indicator`. Read `authoringContractCatalog` from that
   response. Identify the current response, indicator, query-policy,
   visualization, and referenced contracts needed for the turn.
3. Call `drybulk_query_authoring_contract` with every required `schemaId` and
   follow each returned self-contained `jsonSchema` exactly, resolving local
   references inside that document. Fetch a separate catalog entry only for an
   independently validated nested artifact.
4. Use only the current context's declared data schema, result profile, existing
   indicators, and policy evidence. Do not infer an allowed field, source,
   expression, placeholder, chart form, metadata value, or identifier from this
   skill or a prior turn.
5. Refresh context and contracts after any policy, schema, profile, or repair
   change.

## Generation Flow

1. Translate the user's request into the business measure, comparison or trend
   they want to observe.
2. Check whether the current result context contains the data evidence required
   to answer that question under the fetched policy.
3. If a necessary business choice or current-result value is missing, return the
   question response defined by the fetched phase contract. Ask only for the
   missing business fact.
4. Otherwise, build the indicator exactly against the fetched schemas. Use only
   declared sources and fields, and obey every current query-scope, safety,
   template, encoding, metadata, and identity rule returned by the backend.
5. Put every graph series consumed by the query in `sourceSeriesIds`. A query
   may combine several declared series, but it may not reference a metric or
   series literal outside that list. Indicator enablement controls presentation,
   not collection.
6. For every selected series, copy its `collectionMode` from the current
   `availableOutputSeries`, map that mode through the current backend policy's
   `collectionModeStorageTables`, and read only the mapped physical table.
   Never choose a table from similar column names.
7. Set `presentationScopes` explicitly from the user's intended page:
   `MONITORING` for monitoring, `OUTPUT` for result presentation, or both. This
   choice never changes collection.
8. Keep business-facing titles and descriptions understandable to the user. Do
   not expose physical storage, tool, provider, or internal identity details as
   the business explanation.

## Validation and Repair

1. Call `drybulk_validate_output_indicator` exactly as its current tool schema
   requires before returning a definition. Copy `schemaId` and `schemaHash`
   from the fetched contract's `validationIdentity`, not its
   `jsonSchemaIdentity`.
2. On failure, use the returned diagnostics and fetched contracts to repair only
   the diagnosed query, template, identity, metadata, or scope defect, then
   rerun validation.
3. If the validator shows that a real business fact or current-result value is
   missing, return the fetched question response. If the backend cannot express
   the request, return only a failure branch that is explicitly present in the
   fetched response schema; never invent a blocking shape.
4. Do not relax policy, widen result scope, guess data values, or return an
   unvalidated definition.

## Final Response

- Return exactly one JSON object and no markdown, prose, or code fence.
- The object must validate completely against the response `jsonSchema` fetched
  for the current mode.
- Emit no field, value, source, policy identity, or wrapper absent from current
  contracts and backend evidence.
