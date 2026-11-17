# Athletica UI patch set

These patches target the production app under `rollout/`. They are deliverables only; they have **not** been applied to `lordwhitefire/athletica`.

## Apply order

Run from the Athletica repository root:

1. `ATHLETICA-PRODUCT-DETAIL-UI.patch` — product detail presentation.
2. `ATHLETICA-STOREFRONT-UI-BATCH-01.patch` — storefront header, homepage composition and `/shop` page.
3. `ATHLETICA-ADMIN-UI-BATCH-02.patch` — shared admin header and analytics presentation.
4. `ATHLETICA-ADMIN-MEDIA-UI-BATCH-03.patch` — `/admin/media` upload and browse screen. Depends on batch 02's `AdminHeader`.
5. `ATHLETICA-AFFILIATE-UI-BATCH-04.patch` — affiliate signup, verification, dashboard, trackers, balance, payouts, plus admin affiliate review, campaigns and payout review.
6. `ATHLETICA-ADMIN-CATALOG-UI-BATCH-05.patch` — categories, collections, settings and product CSV import flow.
7. `ATHLETICA-PRODUCT-BULK-UI-BATCH-06.patch` — product row selection, select-all, bulk action bar, action-specific inputs and visible queued/loading feedback.

Example:

```bash
git apply ATHLETICA-PRODUCT-DETAIL-UI.patch
git apply ATHLETICA-STOREFRONT-UI-BATCH-01.patch
git apply ATHLETICA-ADMIN-UI-BATCH-02.patch
git apply ATHLETICA-ADMIN-MEDIA-UI-BATCH-03.patch
git apply ATHLETICA-AFFILIATE-UI-BATCH-04.patch
git apply ATHLETICA-ADMIN-CATALOG-UI-BATCH-05.patch
git apply ATHLETICA-PRODUCT-BULK-UI-BATCH-06.patch
```

## Current coverage

- Product detail: visual redesign.
- Storefront: homepage and catalog browse.
- Admin: shared navigation and analytics visual treatment.
- Media: browse/upload UI using the existing authenticated media APIs.
- Affiliate: nine route screens with multi-step signup, ledger/history tables, tracker link copy, campaign configuration and payout state actions.
- Admin catalog: category and collection management screens, settings sections, and CSV import upload → mapping → processing → completion states. These screens use local UI preview state; backend persistence is not claimed.
- Product bulk UI: row checkboxes, select-visible, clear-selection, status/archive/tag/category/collection actions, action inputs, and queued/loading feedback. The action handler is a visible preview only; no backend execution is claimed.

## Not yet covered

The core screen list in the UI blueprint is now represented by the patch set. Final integration/build validation remains to be done after applying the patches.

## Constraints

- `frontend/` is reference-only; no changes target it.
- No cart, checkout, payment, shipping, inventory or order-management UI is added.
- No fake product, analytics or affiliate data is introduced.
- The media screen does not show a delete action because the current rollout API exposes listing and upload, but no delete endpoint.
- Preserve the existing `/admin/signin` visual design.
- Batch 04 is a UI-only scaffold: its expected affiliate/admin API routes are not present in the inspected source tree yet. The coding agent must implement the API contracts before the data/actions work end-to-end. It does not fabricate sample records.
- These patches have not been built, typechecked, visually browser-tested, or applied to the production repo.
