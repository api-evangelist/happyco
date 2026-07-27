---
name: Manage HappyCo inspections
description: List, create, update, and photo-attach property inspections via the HappyCo gRPC API.
api: grpc.happyco.com (happyco.inspect.inspection.v1.InspectionService)
operations: [ListInspections, CreateInspections, UpdateInspections, AddInspectionPhotos, ArchiveInspections]
---

# Manage HappyCo inspections

Use the HappyCo gRPC API to run the inspection lifecycle for a folder of assets.

## Auth
Authenticate with HTTP Basic over gRPC: Base64-encode `client_id:client_secret`
and send it in the `authorization` metadata header (`Basic <base64>`) on a
TLS gRPC channel to `grpc.happyco.com`. See `authentication/happyco-authentication.yml`.

## Steps
1. **Find the inspections** — call `ListInspections` with a `happyco.type.v1.Paging`
   token and a folder filter. Page with the returned `PagingResponse` cursor.
2. **Create work** — call `CreateInspections` (inspections are normally created
   with `scheduled` status). You may reference records by your own `external_id`
   via `happyco.type.v1.IntegrationID` instead of HappyCo IDs.
3. **Update** — call `UpdateInspections` to change inspection contents. Note
   `asset_id` and `template_id` cannot be changed after creation.
4. **Attach photos** — call `AddInspectionPhotos`; insert the returned photo IDs
   into the inspection so they display. This RPC is not transactional and returns
   a response per photo.
5. **Retire** — call `ArchiveInspections` to soft-delete; `UnarchiveInspections`
   to restore. There is no purge.

## Rules
- Bulk mutation RPCs run in a single transaction and roll back entirely on error
  (except `AddInspectionPhotos`).
- No idempotency key is available; de-duplicate on your side using `external_id`.
- Errors return as gRPC status codes. See `conventions/happyco-conventions.yml`.
