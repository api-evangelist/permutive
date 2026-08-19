---
name: Track events and identify Permutive users
description: Create a Permutive user ID, attach external identities to it, track behavioural events, and read back resolved identities.
api: openapi/permutive-events-api-openapi.yml
operations: [createUserId, identifyUser, getIdentities, createEvent]
generated: '2026-08-13'
method: generated
source: openapi/permutive-events-api-openapi.yml + openapi/permutive-identity-api-openapi.yml (both Permutive-published)
---

# Track events and identify Permutive users

Use this skill for the Permutive data plane: minting an identity, binding
external identifiers to it, and writing behavioural events against it. Every
`operationId` is verified against Permutive's published OpenAPI documents.

> Regenerated 2026-08-13. The previous version used `trackEvent` and
> `retrieveIdentities`, which are not operationIds on this API, and bearer
> auth, which this API does not accept.

## Base URL

```
https://api.permutive.app/v2.0
```

## Authentication

An API key (UUID v4) in the `X-API-Key` header, or on the `k` query parameter.
Both the Events and Identity specs declare exactly these two schemes.

Use a **public** key for anything running on a site, app or device — that is
what public keys are for. Use a private key only server-side.

```
POST https://api.permutive.app/v2.0/events
X-API-Key: <API_KEY>
Content-Type: application/json
```

## Consent comes first

Permutive is a data processor and **the API has no server-side consent gate**.
Permutive states plainly that it processes every event it receives. Before
calling `createEvent` or `identifyUser` for a user, confirm the controller has
consent under the applicable framework. If you do not have consent, use the
Contextual API instead — it evaluates page content, not people. See
<https://docs.permutive.com/governance/consent>.

## Steps

1. **Mint a user ID** — `createUserId` (`POST /users`). Returns a new Permutive
   user ID (UUID v4). This is a first-party, publisher-scoped identity; it does
   not follow the user across domains or devices on its own. Persist it — your
   application owns the identity lifecycle.
2. **Attach identities** — `identifyUser` (`POST /identify`). Body requires
   `user_id` and `aliases`, an array of `{tag, id, priority}`. `tag` names the
   identifier namespace (an email hash, ID5, RampID, UID2 …); `priority`
   orders resolution when several aliases resolve to the same person. Lower
   priority wins first — check
   <https://docs.permutive.com/guides/signals/identity/adding-identities-via-identify>
   for the priority configuration before choosing values.
3. **Track an event** — `createEvent` (`POST /events`), returns `201`. Body
   requires `name`; optionally `user_id`, `aliases`, `view_id`, `session_id`,
   `segments`, `cohorts` and `properties`. Permutive validates the event
   against the **event schema defined in your workspace** — a body that does
   not match that schema is a `400`, and the schema is workspace configuration,
   not part of the OpenAPI. Confirm the schema before writing a new event name:
   <https://docs.permutive.com/guides/connectivity/events/create-update-event-schema>.
   Permutive generates the event id and may enrich the event before persisting.
4. **Read identities back** — `getIdentities`
   (`GET /users/{userId}/aliases`). Returns every identity available for the
   user as `{id, tag, permutive_id}`. This one operation declares a `403` for
   insufficient key permissions; the others do not.

## Retries

There is no idempotency key anywhere on this API. A retried `createEvent` will
create a second event. If a call times out, prefer letting the event go than
double-counting it — Permutive's cohort frequency conditions
(`greater_than_or_equal_to`) are directly sensitive to duplicates.

## Errors

`{request_id, error{status, code, message, cause?, docs}}` as
`application/json`. Events declares `400`, `401`, `500`; Identity declares
`401`, `403`, `500`. Neither declares `429` even though per-workspace rate
limits exist — you will not get a documented throttling signal, so pace
yourself. See `errors/permutive-problem-types.yml` and
`rate-limits/permutive-rate-limits.yml`.
