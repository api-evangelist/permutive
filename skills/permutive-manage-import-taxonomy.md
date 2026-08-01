---
name: Manage Permutive import segment taxonomy
description: Browse second-party data imports and create, read, update and delete their segments.
api: openapi/permutive-openapi.yml
operations: [getImports, retrieveImport, getSegmentsForImport, getSegmentById, getSegmentByCode, createSegment, updateSegmentById, updateSegmentByCode, batchUpdateSegments, deleteSegmentById, deleteSegmentByCode]
---

# Manage Permutive import segment taxonomy

Use this skill to manage the **segment taxonomy** attached to second-party data
**imports** in Permutive.

## Authentication
Workspace **API key** as a bearer token. See
`authentication/permutive-authentication.yml`.

## Base URL
`https://api.permutive.com`

## Steps
1. **List imports** — `getImports` (`GET /imports`), then `retrieveImport`
   (`GET /imports/{importId}`) for detail.
2. **List segments** — `getSegmentsForImport`
   (`GET /imports/{importId}/segments`). This is **cursor-paginated**; pass the
   `cursor` query parameter to page through results.
3. **Read a segment** — by public ID (`getSegmentById`) or by code
   (`getSegmentByCode`).
4. **Create a segment** — `createSegment`
   (`POST /imports/{importId}/segments`).
5. **Update a segment** — `updateSegmentById` / `updateSegmentByCode`, or apply
   many changes at once with `batchUpdateSegments`
   (`PATCH /imports/{importId}/segments`) — **max 5,000 operations per request;
   operation order is not guaranteed**.
6. **Delete a segment** — `deleteSegmentById` / `deleteSegmentByCode`.

## Rules
- Because batch operation order is not guaranteed, do not depend on one
  operation's result within the same batch.
- Follow the pagination cursor to completion; see
  `conventions/permutive-conventions.yml`.
- Handle `404` (missing import/segment) and `401`/`403` per
  `errors/permutive-problem-types.yml`.
