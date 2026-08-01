---
name: Track events and identify Permutive users
description: Create user IDs, track behavioural events, and associate identities for a Permutive user.
api: openapi/permutive-openapi.yml
operations: [createUserId, trackEvent, identifyUser, retrieveIdentities]
---

# Track events and identify Permutive users

Use this skill to feed first-party behavioural signal into Permutive and to
manage a user's cross-platform identity.

## Authentication
Workspace **API key** as a bearer token. See
`authentication/permutive-authentication.yml`.

## Base URL
`https://api.permutive.com`

## Steps
1. **Create a user ID** — `createUserId` (`POST /users`) to mint a Permutive
   user identifier when you do not already have one.
2. **Track an event** — `trackEvent` (`POST /events`) with an event `name` and
   `properties`. The event is persisted for downstream segmentation.
3. **Identify a user** — `identifyUser` (`POST /identify`) to associate one or
   more external identities (aliases) with the Permutive user.
4. **Retrieve identities** — `retrieveIdentities`
   (`GET /users/{userId}/aliases`) to read all identities linked to a user.

## Rules
- Send events with accurate `user_id` so segmentation attributes them correctly.
- Respect consent — see https://developer.permutive.com/governance/consent.
- Handle `400` (validation) and `429` (rate limit) per
  `errors/permutive-problem-types.yml`; back off and retry on `429`.
