---
name: Manage Permutive audience cohorts
description: Create, list, inspect, update and delete audience cohorts in a Permutive workspace using the Cohorts API.
api: openapi/permutive-cohorts-api-openapi.yml
operations: [getV2Cohorts, postV2Cohorts, getV2CohortsCohortid, patchV2CohortsCohortid, deleteV2CohortsCohortid]
generated: '2026-08-13'
method: generated
source: openapi/permutive-cohorts-api-openapi.yml (Permutive-published) + https://docs.permutive.com/api/cohorts/introduction
---

# Manage Permutive audience cohorts

Use this skill to manage **cohorts** — the reusable audience definitions
Permutive evaluates from behavioural and contextual signals. Every
`operationId` below is verified against Permutive's own published OpenAPI
document; do not invent others.

> Regenerated 2026-08-13 against Permutive's real spec. The previous version of
> this skill used operationIds (`createCohort`, `getCohorts`…), a bearer-token
> auth scheme and a base URL that do not exist on this API.

## Base URL

```
https://api.permutive.app/cohorts-api
```

`https://api.permutive.com` still works for older deployments but Permutive
asks new integrations to use the `.app` host.

## Authentication

A **private** API key — a version-4 UUID — passed on the `k` query parameter
(the scheme the Cohorts spec declares) or in the `X-API-Key` header.

```
GET https://api.permutive.app/cohorts-api/v2/cohorts?k=<PRIVATE_API_KEY>
```

A private key defaults to **read-only** access to cohorts in the workspace that
owns it plus cohorts inherited from ancestor workspaces. Write access and
child-workspace access are granted per key by Permutive Technical Services. If
you get a `403`, that is the reason — do not retry, escalate.

See `authentication/permutive-authentication.yml`.

## Steps

1. **List existing cohorts** — `getV2Cohorts` (`GET /v2/cohorts`). Returns every
   cohort in summary form, **without** the `query` field. Pass
   `include-child-workspaces=true` to reach workspaces below yours in the
   organization hierarchy (requires the access level). There is **no
   pagination** on this operation — the whole set comes back in one response.
   Check here before creating a duplicate.
2. **Create a cohort** — `postV2Cohorts` (`POST /v2/cohorts`), returns `201`.
   Body requires `name` and `query`; `description`, `tags` and `segmentType`
   are optional. `query` uses Permutive's cohort query format
   (<https://docs.permutive.com/api/cohorts/cohort-query-format>), for example:
   ```json
   {
     "name": "Football readers",
     "query": {
       "event": "Pageview",
       "frequency": { "greater_than_or_equal_to": 2 },
       "where": { "property": "properties.client.url", "condition": { "contains": "football" } },
       "during": { "the_last": { "value": 30, "unit": "days" } }
     },
     "tags": ["sport"]
   }
   ```
   **There is no idempotency key on this API.** If the request times out, do
   not blind-retry — call `getV2Cohorts` and check whether the cohort landed.
3. **Inspect a cohort** — `getV2CohortsCohortid` (`GET /v2/cohorts/{cohortId}`).
   This is the only operation that returns the cohort's `query`, along with
   `state`, `tags`, `segmentType`, `liveAudienceSize`, `createdAt` and
   `lastUpdatedAt`.
4. **Update a cohort** — `patchV2CohortsCohortid` (`PATCH /v2/cohorts/{cohortId}`).
   Partial update over `name`, `description`, `query`, `tags`, `state`.
5. **Delete a cohort** — `deleteV2CohortsCohortid` (`DELETE /v2/cohorts/{cohortId}`).
   Destructive and not reversible through the API. Confirm with the human
   before calling it, and only after reading the cohort in step 3.

## Identifiers

A cohort has two: `id` (UUID, globally unique) and `code` (integer, short and
workspace-scoped). Ad-server targeting values use the `code`. Always pass the
`id` in a path parameter.

## Errors

Errors return `{request_id, error{status, code, message, cause?, docs}}` as
`application/json`. This API declares `400`, `401`, `403`, `404`, `409`, `500`.
Quote `request_id` when escalating. Note that Permutive guarantees a numeric
`error.code` subcode but publishes no subcode registry — do not branch on it.
See `errors/permutive-problem-types.yml`.

## What this API cannot do

No search, no filtering, no paging, no bulk operations. Natural-language cohort
search and audience measurement exist only on Permutive's invitation-only MCP
server — see `mcp/permutive-tool-crosswalk.yml`.
