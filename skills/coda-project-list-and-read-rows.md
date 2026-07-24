---
name: List docs and read table rows
description: Discover a Coda doc, find a table, and read its rows using the Coda Docs API.
api: openapi/coda-project-openapi-original.json
operations: [whoami, listDocs, listTables, listColumns, listRows, getRow]
---

# List docs and read table rows

Use the Coda (Superhuman Docs) Docs API to locate a doc and read structured data from a table.

## Auth
Send `Authorization: Bearer <api_token>` on every request. A read-restricted token is sufficient for this flow. Base URL: `https://coda.io/apis/v1`.

## Steps
1. **`whoami`** — `GET /whoami` to confirm the token is valid and see the associated user.
2. **`listDocs`** — `GET /docs` (optionally with `query`, `isOwner`, `isStarred`) to find the target `docId`. Follow `nextPageToken` for more pages.
3. **`listTables`** — `GET /docs/{docId}/tables` to get each table's `id`/`name`; pick the `tableIdOrName`.
4. **`listColumns`** — `GET /docs/{docId}/tables/{tableIdOrName}/columns` to learn column ids/names for interpreting cells.
5. **`listRows`** — `GET /docs/{docId}/tables/{tableIdOrName}/rows`. Use `useColumnNames=true` for readable keys, `valueFormat=rich` for typed values, and `query` to filter by a column. Paginate with `limit` + `pageToken`.
6. **`getRow`** — `GET /docs/{docId}/tables/{tableIdOrName}/rows/{rowIdOrName}` for a single row's detail.

## Conventions & errors
- Pagination is cursor-based (`pageToken` + `limit`); read the `nextPageToken`/`nextPageLink` from each response.
- On `429` back off and retry (read limit is 100 requests / 6s). On `401`/`403` the token is invalid or lacks access; `404` means the doc/table/row is not visible to this token.
