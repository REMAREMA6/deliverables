# Athletica UI patch set

These patches target the production app under `rollout/`. They are deliverables only; they have **not** been applied to `lordwhitefire/athletica`.

## Apply order

Run from the Athletica repository root:

1. `ATHLETICA-PRODUCT-DETAIL-UI.patch` — product detail presentation.
2. `ATHLETICA-STOREFRONT-UI-BATCH-01.patch` — storefront header, homepage composition and `/shop` page.
3. `ATHLETICA-ADMIN-UI-BATCH-02.patch` — shared admin header and analytics presentation.
4. `ATHLETICA-ADMIN-MEDIA-UI-BATCH-03.patch` — `/admin/media` upload and browse screen. Depends on batch 02's `AdminHeader`.

Example:

```bash
git apply ATHLETICA-PRODUCT-DETAIL-UI.patch
git apply ATHLETICA-STOREFRONT-UI-BATCH-01.patch
git apply ATHLETICA-ADMIN-UI-BATCH-02.patch
git apply ATHLETICA-ADMIN-MEDIA-UI-BATCH-03.patch
```

## Current coverage

- Product detail: visual redesign.
- Storefront: homepage and catalog browse.
- Admin: shared navigation and analytics visual treatment.
- Media: browse/upload UI using the existing authenticated media APIs.

## Not yet covered

The remaining admin product-management workflows, bulk operations, two-phase CSV import, affiliate signup/onboarding, affiliate dashboard, trackers, campaigns, balance and payouts still need their UI work.

## Constraints

- `frontend/` is reference-only; no changes target it.
- No cart, checkout, payment, shipping, inventory or order-management UI is added.
- No fake product, analytics or affiliate data is introduced.
- The media screen does not show a delete action because the current rollout API exposes listing and upload, but no delete endpoint.
- Preserve the existing `/admin/signin` visual design.
- These patches have not been built, typechecked, visually browser-tested, or applied to the production repo.
