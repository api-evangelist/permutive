---
name: Segment users without a Permutive SDK
description: Run a Direct API deployment — call the CCS API for consented users and the Contextual API for everyone, then merge both signal sets for the ad server.
api: openapi/permutive-segmentation-api-openapi.yml
operations: [postCcsV1Segmentation, postCcsV1SegmentationStateless, getContextualSegments]
generated: '2026-08-13'
method: generated
source: https://docs.permutive.com/sdks/api/overview + openapi/permutive-segmentation-api-openapi.yml
---

# Segment users without a Permutive SDK

Use this skill for a **Direct API deployment** — integrating Permutive by
calling the API directly, for server-side architectures, CTV/hybrid-TV setups
and non-owned-and-operated properties where an SDK cannot be deployed.
Permutive's own guidance: if an SDK fits your environment, use the SDK.

Two APIs, chosen by consent state:

| User consent | Call | Signals you get |
|---|---|---|
| Consented | CCS API **and** Contextual API | Behavioural + contextual |
| Not consented | Contextual API only | Contextual only |

**The CCS API has no server-side consent gate.** It processes every event it
receives. Enforcing consent is your job, not Permutive's.

## Step 1 — Behavioural signals (consented users only)

Base URL `https://api.permutive.app`. Key on `X-Api-Key` header or `k` query
parameter. Client-side integrations must use a **public** key; server-side
integrations must use a **private** key.

Two modes:

- `postCcsV1Segmentation` — `POST /ccs/v1/segmentation`. Permutive persists the
  user's segmentation state and applies it to later requests. Body:
  `{user_id, alias?, aliases?, events[]}`. Response:
  `{user_id, cohorts[], activations{}}`.
- `postCcsV1SegmentationStateless` — `POST /ccs/v1/segmentation/stateless`.
  You round-trip the state blob yourself; nothing is persisted. Body adds
  `state`; the response returns the updated `state` alongside the cohorts. Use
  this for testing and for high-performance paths where you already hold state.

Query parameters worth knowing:

- `activations=true` — return the activated cohort list (the ad-server-ready
  targeting values), not just cohort ids. You almost always want this.
- `synchronous-validation=true` — validate the events against the workspace
  event schema **before** segmenting, so a schema problem surfaces as an error
  instead of being swallowed asynchronously. Permutive recommends this only
  during development; it costs latency.

**Batch limit: 10 events per request.** Split larger batches across calls.

## Step 2 — Contextual signals (all users)

`getContextualSegments` — `POST /segment` under `https://api.permutive.com/ctx/v1`
(the contextual reference still documents the `.com` host). Auth is the same
API key on `?k=`. Send the page URL and properties; you get back
`{cohorts[], activations{}, contextual_data{classifications{...}}}` with
categories, keywords, entities, sentiment, emotion and concepts from whichever
classification providers are enabled on the account (IBM Watson, TextRazor,
Silverbullet 4D, OS Data Solutions, or your own webhook provider).

Safe to call for unconsented users: it evaluates content, not people.

## Step 3 — Merge and activate

Merge the `activations` maps from both responses and pass the combined
targeting values to your ad server or SSP. The map is keyed by destination
(`target_dfp`, `appnexus_adserver`, …), so a merge is a per-key union.

## Operational notes

- **No idempotency key.** A retried segmentation call re-processes the events.
- **No documented rate limits and no `429` in any spec.** Both APIs tell you to
  ask your Permutive representative for your workspace's limits. Build in your
  own pacing; you will get no runtime backoff signal.
- Errors: `{request_id, error{status, code, message, cause?, docs}}`. CCS
  declares `400`, `401`, `403`, `500`; the Contextual reference documents only
  `200`, `400`, `401`, `500` — no `403`, no `404`.

See `conventions/permutive-conventions.yml`, `rate-limits/permutive-rate-limits.yml`.
