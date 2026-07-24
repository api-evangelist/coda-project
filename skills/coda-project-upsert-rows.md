---
name: Upsert and reconcile table rows
description: Idempotently upsert rows into a Coda table using key columns, then verify the result.
api: openapi/coda-project-openapi-original.json
operations: [listTables, listColumns, upsertRows, getMutationStatus, listRows]
---

# Upsert and reconcile table rows

Keep a Coda table in sync with an external source of truth using key-column upserts. Requires a write token. Base URL: `https://coda.io/apis/v1`.

## Steps
1. **`listTables`** — `GET /docs/{docId}/tables` to resolve the target `tableIdOrName`.
2. **`listColumns`** — `GET /docs/{docId}/tables/{tableIdOrName}/columns` to identify the column id(s) that uniquely identify a record (your `keyColumns`).
3. **`upsertRows`** — `POST /docs/{docId}/tables/{tableIdOrName}/rows` with `rows[].cells[]` and a `keyColumns` array. Coda updates existing rows whose key columns match and inserts the rest. Returns `202` + `requestId`.
4. **`getMutationStatus`** — `GET /mutationStatus/{requestId}`; poll until `completed`.
5. **`listRows`** — `GET /docs/{docId}/tables/{tableIdOrName}/rows?useColumnNames=true` to verify the reconciled state.

## Conventions & errors
- `keyColumns` gives upsert semantics — the closest thing to idempotency Coda offers; there is no `Idempotency-Key` header.
- Always poll `getMutationStatus` before verifying with `listRows` (writes are eventually consistent).
- Batch rows into one `upsertRows` call to stay under the 10 writes / 6s limit; back off on `429`.
