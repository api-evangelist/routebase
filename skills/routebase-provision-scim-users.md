---
name: Provision users and groups via SCIM 2.0
description: Create, search, patch, and deprovision users and groups in a Routebase organization over the SCIM 2.0 endpoints, as an identity provider (Okta, Microsoft Entra) would.
api: openapi/routebase-public-api-openapi.json
operations: [scimServiceProviderConfig, scimListUsers, scimCreateUser, scimGetUser, scimPatchUser, scimDeleteUser, scimListGroups, scimCreateGroup, scimPatchGroup]
---

# Provision users and groups via SCIM 2.0

Use this from an IdP connector to sync identities into a Routebase organization.

## Auth
SCIM does **not** use an API key. Authenticate with the organization's **SCIM bearer token**,
issued per organization when SCIM provisioning is enabled: `Authorization: Bearer <scim-token>`.
Every path is scoped to the org: `/scim/v2/{orgSlug}/...`. SCIM is rate limited **per source IP**
(all calls from one IdP share one budget), so serialize bulk syncs and honor `Retry-After` on `429`.

## Steps
1. **Discover** — `scimServiceProviderConfig` (GET `/scim/v2/{orgSlug}/ServiceProviderConfig`) and
   `scimSchemas` to confirm supported operations and filters.
2. **Find or create a user** — `scimListUsers` (GET `/Users`, filter by `externalId`/`userName`,
   `startIndex`/`count` pagination) then `scimCreateUser` (POST `/Users`) if absent.
3. **Update** — `scimPatchUser` (PATCH `/Users/{id}`) with a SCIM `Operations[]` patch, or
   `scimReplaceUser` (PUT) for a full replace.
4. **Groups** — `scimCreateGroup` / `scimPatchGroup` to manage membership (`members[]` reference users).
5. **Deprovision** — `scimDeleteUser` (DELETE `/Users/{id}`). There is no restore path, so soft-disable
   via PATCH (`active: false`) when reversibility matters rather than delete.

## Conventions
- Errors use the **SCIM error envelope**, not RFC 9457: `schemas` contains
  `urn:ietf:params:scim:api:messages:2.0:Error`.
- Pagination is 1-based (`startIndex` defaults to 1, `count` to 100).
