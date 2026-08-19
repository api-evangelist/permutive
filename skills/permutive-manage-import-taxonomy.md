---
name: Manage Permutive import segment taxonomy
description: List second- and third-party data imports and create, read, update and delete the segments in their taxonomy, individually or in batches.
api: openapi/permutive-taxonomy-api-openapi.yml
operations: [getImports, getImportsImportid, getImportsImportidSegments, postImportsImportidSegments,
  patchImportsImportidSegments, getImportsImportidSegmentsSegmentid, patchImportsImportidSegmentsSegmentid,
  deleteImportsImportidSegmentsSegmentid, getImportsImportidSegmentsCodeSegmentcode,
  patchImportsImportidSegmentsCodeSegmentcode, deleteImportsImportidSegmentsCodeSegmentcode]
generated: '2026-08-13'
method: generated
source: openapi/permutive-taxonomy-api-openapi.yml (Permutive-published)
---

# Manage Permutive import segment taxonomy

Use this skill to manage the **taxonomy** of a data import — the mapping of
segment codes to human-readable names and metadata that Permutive cohorts are
then built against. All eleven `operationId`s are verified against Permutive's
published OpenAPI document.

> A **segment** here is NOT a Permutive cohort. The Taxonomy API's `Segment` is
> an entry in an imported partner taxonomy. The Cohorts API also uses `Segment*`
> in its schema names but means a cohort. Do not cross the two.

## Base URL

```
https://api.permutive.app/audience-api/v1
```

## Authentication

A **private** API key. Note a real gap: this spec declares **no**
`securitySchemes` at all, but the documentation states every request must carry
a private key. Send it in the `X-API-Key` header (or on `k`) exactly as for the
other APIs.

## Steps

1. **List imports** — `getImports` (`GET /imports`). Returns
   `{items: [Import]}`. Each import carries `id`, `name`, `code`, `relation`
   (`second` or `third` party), `identifiers`, optional `inheritance`
   (the ancestor workspace it came from) and `source` (the data source and its
   state — `live_ramp_3p`, `RealtimeAPI`, `Active`, `Creating`, `SetupFailure`…).
   Check `source.state` before writing: an import that is `Creating` or in
   `SetupFailure` is not ready.
2. **Read one import** — `getImportsImportid` (`GET /imports/{importId}`).
3. **List its segments** — `getImportsImportidSegments`
   (`GET /imports/{importId}/segments`). **This is the only paged operation on
   the whole Permutive platform.** Omit `pagination_token` for the first page;
   then follow `pagination.nextToken` until it is absent. `pagination.totalCount`
   gives the size. Do not assume one page.
4. **Create a segment** — `postImportsImportidSegments`
   (`POST /imports/{importId}/segments`). Requires `name` and `code`; optional
   `description`, `cpm` (the price you attach to the segment for data
   collaboration) and `categories`.
5. **Read a single segment** — two addressing modes, pick one and stay
   consistent:
   - by public UUID: `getImportsImportidSegmentsSegmentid`
     (`GET /imports/{importId}/segments/{segmentId}`)
   - by partner code: `getImportsImportidSegmentsCodeSegmentcode`
     (`GET /imports/{importId}/segments/code/{segmentCode}`)
6. **Update a segment** — `patchImportsImportidSegmentsSegmentid` (by UUID) or
   `patchImportsImportidSegmentsCodeSegmentcode` (by code). Body accepts
   `name`, `description`, `cpm`, `categories`. `code` is not updatable.
7. **Delete a segment** — `deleteImportsImportidSegmentsSegmentid` or
   `deleteImportsImportidSegmentsCodeSegmentcode`. Destructive; confirm first.
   Note the spec's own summaries for these two are swapped relative to their
   paths — trust the path, not the summary.
8. **Batch changes** — `patchImportsImportidSegments`
   (`PATCH /imports/{importId}/segments`). This is how you sync a taxonomy of
   any size. Each element of `operations` is one of `Create` (`{data}`),
   `Update` (`{code, data}`) or `Delete` (`{code}`).
   - **Maximum 5,000 operations per batch.**
   - **Operation order is not guaranteed.** Never put a Delete and a Create for
     the same `code` in one batch, and never rely on one operation observing
     another's result.
   - The request shape is a non-empty list (`{operations: {head, tail[]}}`),
     not a plain array — check the schema before building the body.

## Errors — this API differs from the rest of Permutive

Validation `400`s come back as **`text/plain`** strings such as
`Invalid value for: path parameter importId`, not as the JSON error envelope.
Everything else falls through a single `default` response typed as
`HttpErrorResponse`, whose `error` is a 23-variant union
(`AuthorizationMissing`, `InvalidAPIKey`, `InsufficientAPIKey`, `DoesNotExist`,
`AlreadyExists`, `ValidationError`, `RouteDeprecated`, `UnknownError` …) with
**no discriminator**, so you cannot reliably tell the variants apart from the
wire format. Handle by HTTP status first. This spec also spells the envelope
`requestId` where the rest of Permutive spells it `request_id`.

See `errors/permutive-problem-types.yml`.
