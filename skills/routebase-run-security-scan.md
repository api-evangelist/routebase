---
name: Run an OWASP security scan and export findings as SARIF
description: Enqueue a security scan against a project's scan profile, poll it to completion, then read findings and export them as SARIF for a code-scanning dashboard.
api: openapi/routebase-public-api-openapi.json
operations: [enqueueScanRun, getScanRunById, getSecurityFindings, exportSecurityFindingsSarif]
---

# Run a security scan and export SARIF

Routebase scans against the OWASP API Top 10 (13 scanners). Use this to gate releases on findings.

## Auth
`X-API-Key: rb_live_...` with `security:read` and `security:execute` scopes. `X-RB-Region` for US orgs.

## Steps
1. **Enqueue** — `enqueueScanRun` (POST `/api/projects/{projectId}/security/scan-profiles/{profileId}/run`).
   Returns a `ScanRun` with an `id` and a `ScanStatus`.
2. **Poll** — `getScanRunById` (GET `/api/projects/{projectId}/security/scan-runs/{runId}`) until
   the `ScanStatus` is terminal. Back off on `429` using `Retry-After`.
3. **Read findings** — `getSecurityFindings` (GET `/api/projects/{projectId}/security/findings`);
   each `SecurityFinding` has `severity`, `confidence`, and a `FindingStatus`.
4. **Export SARIF** — `exportSecurityFindingsSarif`
   (GET `/api/projects/{projectId}/security/findings/export/sarif`) to feed a code-scanning UI.
5. **Gate** — fail the job on findings above your severity threshold. With the CLI, `routebase scan`
   returns exit code `1` on a policy violation.

## Conventions
- Errors are RFC 9457 `application/problem+json`; branch on `code`.
- All identifiers are public UUIDs.
