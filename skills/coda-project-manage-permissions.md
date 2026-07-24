---
name: Manage document sharing and permissions
description: Inspect and change who can access a Coda doc via the ACL permissions endpoints.
api: openapi/coda-project-openapi-original.json
operations: [getSharingMetadata, searchPrincipals, addPermission, getPermissions, deletePermission]
---

# Manage document sharing and permissions

Programmatically share a Coda doc and manage its access-control list. Requires a write token with access to the doc. Base URL: `https://coda.io/apis/v1`.

## Steps
1. **`getSharingMetadata`** — `GET /docs/{docId}/acl/metadata` to confirm you can share and see sharing capabilities.
2. **`searchPrincipals`** — `GET /docs/{docId}/acl/principals/search?query=...` to resolve a user/group/domain principal to grant.
3. **`addPermission`** — `POST /docs/{docId}/acl/permissions` with the `access` level (e.g. readonly, write, comment) and the resolved `principal`. Optionally suppress email notification.
4. **`getPermissions`** — `GET /docs/{docId}/acl/permissions` to list current grants and capture each `permissionId`. Paginate with `pageToken`.
5. **`deletePermission`** — `DELETE /docs/{docId}/acl/permissions/{permissionId}` to revoke a grant.

## Conventions & errors
- List responses are cursor-paginated (`pageToken` + `limit`).
- `403` means the token lacks share rights on the doc; `404` means the doc or permission id is not visible to the token.
- Respect the 10 writes / 6s limit when adding/removing many permissions; back off on `429`.
