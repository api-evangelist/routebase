---
name: Promote an OpenAPI spec version into an environment
description: List a project's specs, inspect their versions, and promote a version into a target environment after deployment — with re-promotion of a prior version as the rollback path.
api: openapi/routebase-public-api-openapi.json
operations: [getApiSpecs, getSpecVersions, exportSpecVersion, promoteSpecVersion, exportApiSpec]
---

# Promote a spec version

Use this after a deploy to record which spec version is live in an environment.

## Auth
`X-API-Key: rb_live_...` with `specs:read` and `specs:publish` scopes. Send `X-RB-Region` for US orgs.

## Steps
1. **Find the spec** — `getApiSpecs` (GET `/api/projects/{projectId}/specs`).
2. **List versions** — `getSpecVersions` (GET `/api/projects/{projectId}/specs/{specId}/versions`)
   to pick the `versionId` to promote; each version carries a `VersionStatus`.
3. **(Optional) inspect** — `exportSpecVersion` (GET `.../versions/{versionId}/export`) or
   `exportApiSpec` (GET `.../specs/{specId}/export`) to diff before promoting.
4. **Promote** — `promoteSpecVersion` (POST `.../versions/{versionId}/promote`) with an
   `environmentId` in the body. Returns a `PromotionResult` (`promotionId`, source `PinSource`).
5. **Rollback** — there is no dedicated undo. Re-run `promoteSpecVersion` with an earlier
   `versionId` from step 2 to effectively roll back. No time window is enforced.

## Conventions
- Not idempotent: no Idempotency-Key is supported, so do not blind-retry a promote on a timeout —
  re-check state with `getSpecVersions` first (see conventions/routebase-conventions.yml).
- `409 Conflict` signals a conflicting state, returned as `application/problem+json`.
