---
name: Run a Routebase test suite in CI and gate the build
description: Kick off a project's test suite, poll for completion, pull the report, and fail the build on failure — the routebase CLI flow, mapped to the public API operations.
api: openapi/routebase-public-api-openapi.json
operations: [cliListSuites, cliListEnvironments, cliRunTestSuite, cliGetRunStatus, cliDownloadReport]
---

# Run a Routebase test suite in CI

Use this to run contract/integration tests against an environment and gate a pipeline.

## Auth
Send `X-API-Key: rb_live_...` on every call. The key must carry the `tests:read` and
`tests:execute` scopes. US organizations must send `X-RB-Region: us` (omit for EU, the default).

## Steps
1. **Pick the suite** — `cliListSuites` (GET `/api/cli/suites`) to resolve the suite id, and
   `cliListEnvironments` (GET `/api/cli/environments`) to choose the environment whose variables
   the run should use. A read-only environment reports affected cases as *blocked*, not failed.
2. **Start the run** — `cliRunTestSuite` (POST `/api/cli/run`) with the `projectId`, `suiteId`,
   and `environmentId`. It returns a `testRunId`.
3. **Poll** — `cliGetRunStatus` (GET `/api/cli/runs/{id}/status`) until the run is complete.
   Honor rate limits: 100 requests / 60s per key-user, and back off on `429` using `Retry-After`.
4. **Fetch the report** — `cliDownloadReport` (GET `/api/cli/runs/{id}/report`). A `409` means the
   report is not ready yet — retry after a short delay.
5. **Gate** — fail the job on any failed assertion. With the `routebase` CLI directly, `routebase run`
   returns exit code `1` on test failure, `4` on auth failure — map those to your pipeline.

## Conventions
- Errors are `application/problem+json` (RFC 9457); read the machine-readable `code`, not `detail`.
- All identifiers are public UUIDs.
