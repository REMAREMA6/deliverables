# Athletica — Non-UI Blueprint Compliance and Remediation

**Target repository:** `lordwhitefire/athletica`  
**Active application:** `rollout/`  
**Prepared against:** current `main` after the 2026-11-17 FM-02 correction commit  
**Purpose:** Give the implementation agent an explicit, auditable list of non-UI blueprint obligations, missed requirements, confirmed code defects, and verification work.

---

## 1. Scope and authority

This document is for **non-UI implementation and correction only**. The storefront and admin UI design/implementation are being handled separately. Do not redesign, restyle, or replace visual components, page layouts, or interaction design.

Use the current full-marketplace plan as the product scope:

- `blueprint/` — full-marketplace blueprint, 80-area checklist, gap questions, ISO 25010 review.
- `product-management/01-re-findings-consolidated.md` — reverse-engineering findings and constraints.
- `product-management/02-area-decisions-evidence.md` — area decisions and evidence.
- `product-management/03-feature-cards.md` — feature-level requirements.
- `product-management/04-creators-api-recommendation.md` and `PLAN-creators-api.md` — Amazon Creators API decisions.
- `openspec/changes/athletica-full-marketplace/` — current full-marketplace proposal, design, specs, and 26-task plan.
- `openspec/tasks/FM-01` through `FM-26` — task intent, behavioral checks, and implementation/test reports.

**Documentation conflict to resolve:** root `BLUEPRINT.md` still describes a smaller Study-mode assessment. It is not the current full-marketplace plan. Reconcile it with the Plan-mode material under `blueprint/`; do not silently mix the old Study assessment with the full-marketplace scope. Preserve historical documents as history where appropriate, but clearly label superseded decisions.

### Active-project boundaries

1. Build and maintain `rollout/` only.
2. `frontend/` is reference material for existing catalogue fields, product content, and domain knowledge. Do not treat it as the active application or copy its UI into `rollout/`.
3. The standalone Sanity Studio is no longer an active workspace. Do not restore or build a Studio application. Keep the server-side Sanity API integration where the active app needs it (for example, media upload); using Sanity APIs is not the same as needing Sanity Studio.
4. The product-detail route remains part of the application. Its current UI is being replaced separately; do not delete the route or build its visual interface.
5. Preserve the existing `/admin/signin` design. Do not delete `/admin/signin-preview` or alter auth-screen visuals without an explicit decision.
6. Do not modify project files outside `rollout/` unless a documentation or workspace correction is explicitly required.

### Business-model boundary

Athletica is an **affiliate marketplace**, not the merchant of record. Amazon owns the retail transaction. Therefore do not add a first-party cart, checkout, payment processing, shipping, inventory management, or order-management system. The product CTA leads through Athletica's redirect/click pipeline to the appropriate Amazon marketplace. Do not invent retailer capabilities.

---

## 2. Required agent conduct

Before changing code:

1. Read the full-marketplace blueprint and the relevant product-management evidence, not just the root `BLUEPRINT.md`.
2. Read the applicable FM task, its `testing.md`, and the latest implementation and testing reports.
3. Read the relevant Next.js 16.3.4 documentation from the installed package under `rollout/node_modules/next/dist/docs/` before changing Next.js conventions. `rollout/AGENTS.md` explicitly warns that this version differs from prior training.
4. Trace the actual active call path before changing a helper. A legacy-looking helper may be unused; prove usage with imports/call sites before deleting or replacing it.
5. Separate facts into **confirmed defect**, **documented deferral**, and **requires runtime verification**. Do not present static inspection as a successful live test.
6. Do not guess Amazon API behavior, marketplace mappings, payout rules, or schema semantics. Use the reverse-engineering evidence and official integration documentation already recorded in the project.
7. Make the smallest correction that satisfies the blueprint. Do not refactor unrelated code or silently expand scope.
8. Record each change, command, result, and unresolved item in the relevant `developer-workspace/` evidence/report location. Real deliverables belong in the project/workspace locations defined by the agent workflow, not as stray root files.

### UI exclusion

Do not edit CSS, design tokens, visual assets, layout markup, page composition, responsive styling, or UI copy as part of this assignment. Backend data access, server-side authorization, route handlers, schema, service functions, and API contracts are in scope. If a change requires touching a page file, isolate the server-side behavior and leave its visual output untouched; flag any UI-dependent change for the separate UI work.

---

## 3. Blueprint obligations the implementation must satisfy

### A. Canonical product and catalogue data

The data model must support the full catalogue rather than a single product-page demo:

- Products with stable IDs and slugs, ASIN, descriptive content, tags, media references, optional internal price, lifecycle status, availability dates, discontinuation date, and soft deletion.
- Separate product variants, including SKU uniqueness, variant attributes, status, and optional price in integer minor units.
- Categories and collections, with many-to-many product relationships.
- Explicitly defined product-to-category and product-to-collection access patterns.
- Product publication rules using the full three-condition gate: active status, not soft-deleted, and available at the current time (including a documented decision for NULL availability).
- The five product states specified by FM-02: `draft`, `active`, `archived`, `proposed`, and `rejected`. Do not leave old two-state types or validation in the active code.
- Optional price. Price is internal/admin data and must never appear on the public product page or public structured data unless the blueprint explicitly changes that rule.
- Integer cents for product prices; no floating-point product-price arithmetic.
- Product deletion is soft deletion. Affiliate balance records are append-only.
- No stored Amazon destination or affiliate URL. Derive it at redirect time from the ASIN, selected marketplace, and the admin-owned marketplace Associates tag.

Use `frontend/` only to inspect existing catalogue knowledge and legacy product records. Do not blindly copy its schema or assume every legacy field is required. Produce a field-by-field mapping and explain which fields are retained, normalized, split into variants, deprecated, or left as source-only metadata.

### B. Analytics and click integrity

Implement the specified server-side pipeline, not just event counters:

- Server-side view and click recording.
- Correct source attribution (including Instagram, WhatsApp, and affiliate tracker attribution where applicable).
- Visitor identity hashing with rotating salt and a grace window; store the hash, never raw IP.
- Domain separation/cross-site unlinkability as specified by the blueprint.
- Correct trusted IP-header precedence and country resolution.
- Bot filtering in the prescribed order, including the Headless Chrome override and false-positive handling.
- One-hour deduplication using the defined identity key.
- EU IP redaction before persistence.
- Explicit drop reasons and telemetry so suppressed events can be explained.
- Click writes must not delay the Amazon redirect.
- Redirect response must be 302 and include `X-Robots-Tag: noindex`.
- Rate-limit policies must be explicit per endpoint; fail-open behavior must be bounded and documented.

Do not count a click when the destination is unavailable/dead or the redirect cannot be validly derived. Do not suppress a visitor's redirect merely because analytics classified the visitor as a bot; the documented behavior is to redirect but suppress the click event.

### C. Amazon marketplace routing and Creators API

- Support the exact 22 marketplaces in the blueprint, including the UK `GB` country-code mapping.
- Resolve visitor country using the specified header chain, map it to a marketplace, and apply the documented fallback order (including product default marketplace and the configured default).
- Associates tags belong to the **admin/platform**, not individual affiliates. Affiliates are identified by tracker IDs; do not assign them Amazon tags or Creators API credentials.
- Tags must be configured per marketplace in the marketplace data/configuration model. Do not silently fall back to one global tag for every country.
- Derive Amazon URLs at request time. Never persist a prebuilt URL.
- Creators API is an optional enhancement and must soft-fail when credentials or eligibility are absent. It must not make the core affiliate redirect unavailable.
- Respect the recorded cache lifetimes, retry/backoff rules, API scope, request body, and marketplace/locale association.
- Keep product-owned data authoritative. API cache data must not overwrite manually curated product content.
- Do not invent availability or review data when the API returns no data.

### D. Affiliate program and financial integrity

Implement the affiliate system as a real ledger, not a balance field:

- Affiliate account signup, email verification, approval, and ownership boundaries.
- Tracker creation and attribution; campaigns and campaign-tracker relationships.
- Append-only balance ledger with idempotency on `(account_id, trace)`.
- Server-to-server conversion postback protected by the configured bearer token.
- Commission calculation, currency normalization/FX, reversals, and campaign terms as specified in the feature cards and FM tasks.
- Payout eligibility, minimum threshold, payout details, request, admin review, scheduling, payment, rejection, and notifications.
- Payout state machine: requested → scheduled → paid or rejected.
- Payout requests must be transactional: lock eligible unreleased ledger rows, derive the amount from the ledger, compare against caller input if supplied, release rows and create the payout atomically.
- Sequential payout numbers must remain correct under concurrent requests.
- SECURITY DEFINER RPCs that accept arbitrary account IDs must be executable only by `service_role`, unless the function itself proves ownership safely.
- Record state transitions in the audit log. Never update or delete historical balance entries to simulate a correction.

Do not trust client-submitted payout amounts, affiliate IDs, commission values, or ownership claims.

### E. Admin, API, security, and database boundaries

- Preserve the three-lock admin model: invite-only enrollment, door-key cookie gate, and JWT `app_metadata.role` authorization.
- Admin routes and APIs fail silently with 404 when authorization gates fail, as specified. Do not weaken the design to a public login or signup surface.
- Use `getClaims()` for the fast JWT role check where specified; use `getUser()` only where full user data is needed.
- Door-key validation must hash the presented token, reject expired/revoked/wrong-kind keys, and slide expiry only on valid checks.
- Sign-in must use neutral credential errors and the prescribed rate limit.
- Keep service-role credentials server-only. Use user-scoped clients where RLS is intended to enforce the user's permissions; use service-role clients only for narrowly justified trusted server operations.
- All custom-schema access must have the necessary schema/table grants, while RLS remains the actual row-level boundary.
- Enable and FORCE RLS on every required table.
- Use the `(select auth.jwt())` / `(select auth.uid())` wrapper consistently in policies.
- Affiliate tables must enforce account ownership. Admin-only and service-role-only tables/RPCs must not be reachable by ordinary authenticated users.
- Audit admin actions and financial state changes.
- Validate request bodies, IDs, enum values, pagination, file types/sizes, and origin/rate-limit rules at server boundaries.
- Do not trust a proxy cookie-presence check as the authorization boundary; it is only an early filter. The route/service authorization check remains mandatory.
- Add server-only boundaries to privileged modules where appropriate and avoid exposing internal errors or secrets.

### F. Operational and platform obligations

Implement the full-marketplace operational decisions, not a generic production checklist:

- Vercel deployment configuration and the required scheduled jobs.
- Cron authentication with `CRON_SECRET`.
- Salt rotation, dead-link probing, Creators API refresh, and payout-term checks at their specified schedules.
- Bounded job batches, retry behavior, idempotency, and observable failure states.
- Sentry configured with PII disabled and the required operational alerts.
- Uptime monitoring and alerts for failed/stuck payout and integration flows.
- Environment-variable documentation with placeholder-only `.env.example`; never commit real secrets.
- Correct migration order and explicit owner-run Supabase integration steps.
- API documentation, admin guide, affiliate guide, handoff, and phase smoke-test notes.
- Legal/compliance scope from FM-24, including privacy, terms, cookie notice, affiliate agreement, Amazon disclosure, deletion/export handling, and payout/tax documentation. Do not fabricate legal advice; flag owner/legal review where required.
- Internationalization for the 22 marketplace locales, with English fallback and the specified locale-switch behavior.

### G. Explicit skips — do not reintroduce

These are business-model decisions, not forgotten tasks:

- First-party cart, checkout, payment processing.
- First-party order management, shipping, and inventory.
- Mobile app, app-store distribution, push notifications, device APIs, mobile deep links.
- PWA/offline behavior.
- Outgoing webhooks (Creators API is polled).
- Multi-tenancy.

If a requirement appears to conflict with this list, stop and surface the conflict instead of implementing it.

---

## 4. Confirmed issues and corrections required from current repository inspection

The following are based on files currently present on `main`. Re-check call sites before changing anything.

### 4.1 Database TypeScript model is stale — confirmed

**File:** `rollout/src/lib/database.types.ts`

The file still says it was last updated 2026-10-27 and describes the old product status union (`published | unpublished`). It does not represent the FM-02 five-state product model, new product fields, variants, categories, collections, marketplaces, Amazon cache, affiliate schema, payout tables, audit log, or the updated analytics shape (including tracker attribution).

**Required:** regenerate or accurately update the database types from the applied schema. Do not hand-wave types with broad `any`. Make sure all services compile against the actual schema and correct schema-qualified table names.

### 4.2 FM-01 sign-in rate limiting is not wired — confirmed

**Files:** `rollout/src/lib/auth/signin-rate-limit.ts`, `rollout/src/app/api/admin/auth/signin/route.ts`

The rate-limit helper exists, but the current sign-in route shown in the repository does not import or call it. FM-01 implementation and test reports already identify this as an integration gap.

**Required:** wire the limiter into the route before password authentication, use the specified email+IP key and 5-per-minute threshold, and return the documented rate-limit response without leaking account existence. Verify the sixth request is rejected. Keep the neutral credential error for ordinary failures.

### 4.3 FM-01 implementation is not fully runtime-verified

The current middleware calls the `validate_door_key` RPC. The RPC is now present in `rollout/supabase-rollout-fm02-rpcs.sql`, but the FM-01 reports record no live Supabase run. The owner must apply the integration SQL and verify the door-key, JWT-role, invite race, sliding renewal, and rate-limit behavior against the live project.

Do not report FM-01 as behaviorally complete based only on static inspection.

### 4.4 FM-02 analytics partitioning remains deferred

The FM-02 task requires monthly partitioning. The implementation substituted a BRIN index and documented the deferral because the existing analytics table contains data.

**Required:** preserve the deferral as an explicit tracked follow-up with a migration strategy and trigger threshold. Do not mark the monthly-partition requirement fully complete. Do not perform a destructive table conversion without a migration/rollback plan.

### 4.5 FM-02 migration and RPC integration must be reconciled

The latest 2026-11-17 commit fixes:
- FM-02 migration order: drop the old status CHECK before migrating to the five-state constraint.
- Door-key hashing: use built-in PostgreSQL `sha256()` to match the Node SHA-256 bytes.
- Payout RPC: lock unreleased balance rows, derive payout amount from ledger rows, reject amount mismatch, return JSONB, and grant execution to `service_role` only.
- Dead/alive product RPCs: restrict execution to `service_role`.
- Custom-schema grants required for the Supabase Data API.
- Associates-tag environment-variable naming.

**Required:** retain these corrections. Review the final SQL against the service callers and actual Supabase schema. Apply the migration and RPC file in the documented order, then test the functions live. Do not reintroduce the old return shape or broader EXECUTE grants.

### 4.6 Legacy affiliate URL helper conflicts with the 22-marketplace model — confirmed mismatch; call-site status to verify

**File:** `rollout/src/lib/affiliate/affiliate-service.ts`

This helper still derives URLs from global environment configuration (`AMAZON_ASSOCIATES_TAG` and `AMAZON_MARKETPLACE`) and its host resolver only recognizes a few marketplaces. The new marketplace resolver reads the marketplace table and per-marketplace tag.

**Required:** trace every import/call site. If the helper is still used by a live path, consolidate it with the canonical 22-marketplace resolver and remove the global-marketplace shortcut. If unused, document that fact and remove it only after confirming no external/script dependency. There must be one authoritative URL-derivation path.

### 4.7 Creators API tag source needs alignment — confirmed design mismatch to resolve

**File:** `rollout/src/lib/amazon/creators-api-client.ts`

The client currently reads a single `AMAZON_ASSOCIATES_TAG` environment variable for its `partnerTag`, while the blueprint and marketplace resolver model Associates tags per marketplace and admin-owned.

**Required:** determine the correct tag from the selected marketplace and pass the corresponding admin-owned tag into the Creators API request. Do not use an affiliate's tracker ID as an Amazon tag. Confirm the API's exact partner-tag requirements against the project's Creators API research before changing the request contract.

### 4.8 Dead-link detection has incomplete behavior — confirmed from implementation

**File:** `rollout/src/lib/amazon/dead-link-detector.ts`

The current implementation has several gaps:
- The daily probe selects the first active products with an ASIN and a fixed batch limit; it does not filter/order by a persisted last-probed timestamp. A repeated cron may keep selecting the same first batch.
- `recordRedirect4xx()` is a no-op; its comment says the redirect layer must wire the observation.
- The Creators API check treats a null availability value as dead. Missing/unknown API data is not, by itself, proof that an Amazon product is dead.
- The probe currently marks products alive for any non-404/410 response, including transient or inconclusive responses.

**Required:** implement fair, resumable batches with last-probed/next-probe tracking; wire actual redirect 4xx observations and threshold logic; distinguish confirmed unavailable from unknown/API failure; and make state transitions auditable. Transient network errors and missing API data must not incorrectly mark a product dead or alive.

### 4.9 Product catalogue query behavior needs correction or explicit proof

**File:** `rollout/src/lib/products/product-service.ts`

Static inspection reveals:
- The category filter applies `categories.slug` while the query selects only `*`; verify that the relation is actually selected and filterable in the current Supabase query shape. If not, implement the M2M filter correctly.
- The `popular` sort currently orders by `created_at`, the same as newest. This is not a popularity sort.
- The shared public filter constant is declared but not used; query implementations duplicate the conditions.
- `getProductById()` does not apply the public publication/deletion/availability gate. Trace its callers and ensure public routes cannot use it to expose draft, archived, deleted, or not-yet-available products.

**Required:** implement correct category filtering and popularity ranking using the defined analytics signal (or label the sort honestly if popularity is not yet supported). Centralize the public eligibility predicate where practical and ensure every public read path enforces it.

### 4.10 Task registry and actual implementation status disagree

**Files:** `openspec/changes/athletica-full-marketplace/tasks.md`, `openspec/tasks/FM-*/`

The change task table still labels FM-01 through FM-26 as pending, while the repository contains the applied full-marketplace patch and implementation reports for the early tasks. Conversely, some reports explicitly record integration gaps and static-only testing.

**Required:** reconcile each task using code evidence + report + required runtime verification. Use separate states such as implemented, statically verified, runtime verified, blocked on owner integration, deferred, and not started. Do not bulk-mark all tasks complete because the patch applied. Do not leave completed work labeled pending without explanation.

---

## 5. Additional verification required before declaring the system ready

These are blueprint obligations that must be evidenced; do not assume they work because files exist.

### Database and migration
- Apply the base schema, admin-key schema, schema updates, RLS policies, FM-02 full marketplace migration, then the RPC/dead-link integration SQL in the exact documented order.
- Verify all required tables, columns, enums, indexes, foreign keys, unique constraints, and 22 marketplace rows.
- Verify UK maps to `GB`.
- Verify RLS is both enabled and forced on every required table.
- Test anon, ordinary authenticated, affiliate owner, admin, and service-role access separately.
- Test duplicate balance trace, SKU, token hash, and payout-number behavior.
- Verify no table stores a prebuilt Amazon URL.
- Record the exact migration version and live database result.

### Auth and authorization
- Test each admin lock independently and together.
- Test invite redemption race (one success only).
- Test door-key expiry, revocation, and 30-day sliding renewal.
- Test JWT role rejection for a non-admin.
- Test all protected admin APIs, not just page navigation.
- Test signin rate limit at the exact threshold.
- Confirm service-role keys and invite codes never enter client bundles or logs.

### Affiliate ledger and payouts
- Test duplicate postback/idempotency.
- Test invalid bearer token rejection.
- Test commission calculation, FX normalization, reversals, and ownership isolation.
- Test concurrent payout requests for the same account.
- Verify the RPC locks the correct unreleased rows and the amount comes from the ledger.
- Test every payout transition and audit entry.
- Confirm no affiliate can invoke admin payout/dead-link RPCs for another account.

### Redirect, analytics, and Amazon
- Verify the 302 destination for all 22 marketplace mappings and fallback cases.
- Verify missing ASIN/tag behavior and no stored destination URL.
- Verify bot redirects still occur while click recording is suppressed.
- Verify deduplication, salt rotation/grace, source attribution, EU redaction, and drop-reason telemetry.
- Verify redirect latency is not blocked by analytics persistence.
- Verify Creators API disabled/ineligible/error cases soft-fail.
- Verify cache TTLs and retry/backoff.
- Verify dead-link cron fairness, thresholds, and no false dead/alive state changes.

### Runtime, deployment, and documentation
- Verify each cron route is authenticated and configured in `vercel.json`.
- Verify `.env.example` contains every required variable with placeholders only.
- Verify the documented setup works from a fresh clone using `cd rollout`.
- Verify build, lint, and typecheck with the actual installed dependencies.
- Verify Sentry privacy settings, operational alerts, and uptime checks.
- Complete FM-24 legal owner review and FM-26 handoff documentation.
- Record what was not tested and what still requires owner credentials or production access.

---

## 6. Execution order

Do not start by polishing peripheral issues. Use dependency order:

1. **Reconcile the source of truth:** full-marketplace blueprint vs root Study-mode `BLUEPRINT.md`; confirm the 26 FM task statuses.
2. **Close the foundation:** FM-01 auth integration; FM-02 schema/type alignment; live migration and RPC verification. Data model correctness is the root dependency.
3. **Close the product/redirect/analytics contracts:** FM-03, FM-04, FM-05; verify publication gate, redirect, attribution, privacy, and deduplication.
4. **Close Amazon routing and product availability:** FM-19, FM-20, FM-21, FM-22; one authoritative marketplace/tag/URL path.
5. **Close admin data operations:** FM-10 through FM-14; CRUD, variants, categories, collections, bulk operations, CSV import, media API, analytics data.
6. **Close affiliate and financial flows:** FM-15 through FM-18; ownership, ledger, postback, payout transaction and audit.
7. **Close hardening and handoff:** FM-23 through FM-26; localization, legal, monitoring, cron, docs.
8. **Update the registry and evidence** only after each task's actual state is known.

Follow the one-task-at-a-time and phase-gate rules in the OpenSpec task plan. Do not claim an entire phase is complete when its live integration checks remain outstanding.

---

## 7. Required output from the agent

Produce:

1. A corrected, canonical non-UI implementation plan mapped to the current full-marketplace blueprint and FM-01…FM-26.
2. A file-by-file change list, with the reason and blueprint/task reference for every change.
3. A canonical schema/type map showing the current database model and TypeScript representation.
4. A discrepancy register separating confirmed defects, documented deferrals, and runtime checks still outstanding.
5. A migration and RPC application checklist for the owner.
6. Focused behavioral verification steps and results. No broad, unrelated test-suite expansion.
7. An updated task registry with honest statuses.
8. An evidence report recording files inspected, commands executed, outputs, changed files, and anything not verified.

**Do not implement UI. Do not change the visual design. Do not create a new Sanity Studio. Do not add first-party retail checkout. Do not invent data or silently close gaps.**
