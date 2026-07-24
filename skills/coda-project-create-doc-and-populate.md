---
name: Create a doc and populate a table
description: Create a new Coda doc, add a page, and insert rows into a table, handling async mutations.
api: openapi/coda-project-openapi-original.json
operations: [createDoc, createPage, listTables, upsertRows, getMutationStatus]
---

# Create a doc and populate a table

Provision a new Coda doc and write structured data into it. Requires a write-enabled token. Base URL: `https://coda.io/apis/v1`.

## Steps
1. **`createDoc`** — `POST /docs` with a `title` (optionally `sourceDoc` to copy a template, `folderId` to place it). Capture the returned `id` as `docId`.
2. **`createPage`** — `POST /docs/{docId}/pages` to add a page/section with a `name` and optional `subtitle`/content.
3. **`listTables`** — `GET /docs/{docId}/tables` to resolve the `tableIdOrName` you will write into (a template-based doc already has tables).
4. **`upsertRows`** — `POST /docs/{docId}/tables/{tableIdOrName}/rows` with `rows[].cells[]` (`column` + `value`). This returns `202` with a `requestId` — the write is asynchronous.
5. **`getMutationStatus`** — `GET /mutationStatus/{requestId}`; poll until `completed: true` before treating the rows as written.

## Conventions & errors
- **No idempotency key** — writes are async. Poll `getMutationStatus` and avoid blind retries that could double-insert; re-running `upsertRows` with a `keyColumns` set makes it an upsert (see the upsert skill).
- Write rate limit is 10 requests / 6s (doc-content writes 3 / 10s); back off on `429`.
- `400` with `validationErrors[]` means a cell value or column id is wrong; `403` means the token cannot write this doc.
