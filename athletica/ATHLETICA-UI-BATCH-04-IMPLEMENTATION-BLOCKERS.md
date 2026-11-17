# Athletica UI continuation — Batch 04 repository audit

**Source repository:** `lordwhitefire/athletica`  
**Inspected revision:** `582df3e154298431eaeecc675c37f5b5ee10232d`  
**Purpose:** Record verified integration blockers before adding controls that would appear functional but cannot work against the current app.

## Product management

Existing UI:
- `rollout/src/app/admin/products/page.tsx` — real product listing, search, status filter, sort, pagination, create modal, publish/unpublish, delete confirmation, loading/error/empty states.
- `rollout/src/app/admin/products/product-editor.tsx` — substantial create/edit form.
- `rollout/src/app/admin/products/[id]/edit/page.tsx` — edit route.

Existing backend:
- `rollout/src/lib/products/bulk-operations.ts` — bulk status update, archive, collection add/remove.
- `rollout/src/app/api/admin/products/route.ts` — GET/POST/PATCH/DELETE only.

Blockers:
1. No bulk-products API route exists. Adding selection checkboxes and bulk action buttons without an endpoint would create dead controls.
2. The shared bulk module currently supports status, archive, and collection operations; it does not implement the full feature-card scope for tags and categories.
3. The product API POST currently rejects missing/zero price, while the product-management decision records say price is optional. The UI must not silently work around that mismatch.

## CSV import

Existing engine:
- `rollout/src/lib/products/csv-import.ts` implements mapping and processing.
- The import registry is a module-level in-memory `Map`.
- There is no import API route or admin import page in the inspected route tree.

Blocker:
- A browser preview followed by a separate processing request cannot safely depend on a process-local Map in a serverless deployment. The import state needs durable storage or a single request/session strategy before the two-phase UI can be wired honestly. The engine also inserts draft products directly and is not currently connected to the normal product API.

## Affiliate and payout workspace

The README and feature cards describe:
- `/affiliate/signup`
- `/affiliate/trackers`
- `/affiliate/balance`
- `/affiliate/payouts`
- `/admin/affiliates`
- `/admin/payouts`

The repository search at the inspected revision surfaced affiliate services and task reports, but did not surface the corresponding page routes or admin/affiliate API route implementations. These screens must be wired only after confirming the actual route and service contracts; placeholder dashboards or fabricated balances are not acceptable.

## UI work already delivered

See `ATHLETICA-UI-PATCH-INDEX.md` for the four existing patch batches:
1. Product detail
2. Storefront
3. Admin workspace / analytics
4. Admin media library

These patches are deliverables only; they have not been applied to the Athletica repository or build-tested.

## Next implementation order

1. Resolve the bulk API adapter and align the optional-price contract.
2. Make CSV import state durable and expose admin-guarded mapping/processing endpoints.
3. Implement the bulk-selection and two-phase import screens against those real contracts.
4. Verify affiliate/payout route existence and implement the remaining UI against actual service contracts.
