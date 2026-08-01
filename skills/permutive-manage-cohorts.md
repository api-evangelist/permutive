---
name: Manage Permutive audience cohorts
description: Create, list, inspect, update and delete audience cohorts in a Permutive workspace.
api: openapi/permutive-openapi.yml
operations: [createCohort, getCohorts, getCohort, updateCohort, deleteCohort]
---

# Manage Permutive audience cohorts

Use this skill to manage audience **cohorts** — the reusable audience
definitions Permutive builds from behavioural and contextual signals.

## Authentication
All requests use a workspace **API key** sent as a bearer token
(`Authorization: Bearer <API_KEY>`). Every cohort operation is scoped to the
workspace that owns the key. See `authentication/permutive-authentication.yml`.

## Base URL
`https://api.permutive.com`

## Steps
1. **List existing cohorts** — `getCohorts` (`GET /v2/cohorts`) returns all
   cohorts in the workspace. Check here before creating a duplicate.
2. **Create a cohort** — `createCohort` (`POST /v2/cohorts`) with a `name` and a
   `query` in Permutive's cohort query format
   (https://developer.permutive.com/api/cohorts/cohort-query-format).
3. **Inspect a cohort** — `getCohort` (`GET /v2/cohorts/{cohortId}`) to read a
   single cohort's definition and metadata.
4. **Update a cohort** — `updateCohort` (`PATCH /v2/cohorts/{cohortId}`). Some
   fields may be immutable depending on the key's access level.
5. **Delete a cohort** — `deleteCohort` (`DELETE /v2/cohorts/{cohortId}`).

## Rules
- Handle `401` (bad/missing key) and `403` (key lacks workspace access) per
  `errors/permutive-problem-types.yml`.
- There is no documented idempotency key; guard against duplicate creates by
  listing first (step 1).
- For agent-driven discovery, the Permutive **MCP server** exposes
  `search_cohorts`, `list_cohorts` and `get_cohort_detail` as read tools.
