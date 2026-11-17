# Athletica — OpenSpec Investigation and Alignment Draft

**Status:** Draft for OpenSpec investigation  
**Project repository:** `lordwhitefire/athletica`  
**Active application:** `rollout/`  
**Current reference revision:** `19230dd621d48b98e88637e049e29eea43dccac2` (2026-11-17)  
**Purpose:** Establish what needs to be investigated and agreed in the architecture before any implementation begins.

---

## 1. Why this draft exists

Athletica has had several rounds of product, UI, database, affiliate, and rollout work. The latest changes have exposed conflicts between parts of the application, its data model, its services, and the intended system.

We do not want those issues fixed one by one through assumptions. We want the system to be understood and aligned first, then have the architecture and specifications describe the system we actually intend to build.

This document is an initial set of findings and questions. It is not a final technical design and it does not approve implementation.

## 2. Roles and boundaries

### Product management

Product management is the source of product intent: what Athletica is for, who uses it, what users should be able to achieve, and the business rules and boundaries that have already been agreed.

Product-management material may help explain the intended product. It is not the place to decide technical architecture, code structure, database implementation, API design, or other engineering choices.

### OpenSpec / architecture

The OpenSpec agent is responsible for understanding the existing system and turning the agreed product intent into a clear technical architecture and set of specifications.

The agent should use product-management material as evidence of product intent, inspect the actual code and existing technical documents, and identify where they agree or conflict. Technical decisions belong in the architecture and specifications.

### Developer

The developer will work from the approved architecture and specifications. The developer is not being asked to resolve unclear product or architecture decisions by guessing.

## 3. Initial findings to investigate and confirm

These findings come from the latest repository review and the latest commit report. They are starting points for investigation, not instructions to make particular code changes. Confirm each against the current code, relevant specifications, and actual call paths.

### A. The current application does not build cleanly

The latest Athletica commit report says the application still has 54 TypeScript errors, compared with 40 before that change. The Vercel check for the commit also failed.

The report identifies examples including:
- A missing `createServiceClient` export from the Supabase server module.
- Product administration code using functions that are not exported by the current product service.
- Admin product screens still relying on older product-status values while the data model has moved to a five-state model.

**What needs to be established:** the complete current set of build and type errors, their causes, which are symptoms of a deeper architecture mismatch, and what a successful, agreed build state should mean.

### B. The data model and application behaviour may not describe the same product system

The product lifecycle has moved beyond the earlier two-state published/unpublished model. The current intended states recorded in the project include draft, active, archived, proposed, and rejected.

Some application areas appear to retain older assumptions. The OpenSpec investigation should establish whether the database types, validation, admin behaviour, public catalogue behaviour, and service contracts all represent one consistent product lifecycle.

The investigation should also check whether optional product pricing, variants, categories, collections, and publication rules are represented consistently across the system.

### C. Supabase schema access and affiliate data boundaries need clarification

The project uses both `rollout` and `affiliate` schemas. Existing server-side Supabase helpers appear to be tied to the `rollout` schema, while affiliate services access affiliate tables.

The project also contains affiliate ownership policies and account records with a separate `auth_user_id`. The relationship between authenticated users, affiliate accounts, and related affiliate records needs to be checked carefully.

**What needs to be established:** how each part of the application is expected to access each schema, which identity owns each record, where user-level access control applies, and where trusted server-side access is appropriate. The intended security boundary must remain clear; service-level access must not hide an incorrect ownership model.

The Supabase project's exposed-schema settings may also need owner-side confirmation. Repository SQL alone cannot prove the dashboard configuration.

### D. Affiliate payout-term enforcement may be incomplete

The current repository review found an affiliate payout-term RPC described as a no-op placeholder. The cron route's payout-term-check branch also appeared to return success without carrying out the check.

The intended meaning of campaign payout terms, when commissions become eligible, how balances are released, and how payout requests relate to those balances need to be traced through the existing product decisions, data model, and financial flows.

Do not assume a payout period or invent financial rules where the agreed behaviour is not clear. Any missing business decision should be raised for confirmation.

### E. Existing UI and backend contracts may not line up

Earlier repository audits identified gaps between some planned screens and the available backend routes or service contracts. Examples include bulk product actions, CSV import, and affiliate/payout workspaces.

The OpenSpec investigation should check the current state again rather than rely on an older audit. The goal is to establish which user-facing capabilities are supported by real system behaviour, which are incomplete, and which are only planned.

The interface should not imply that an action or data source works when the system behind it is not ready.

### F. Sign-in options need to retain their planned place without pretending to work

Google and GitHub sign-in options are to remain visible in the sign-in interface. They are not being implemented in this stage.

When selected, they should clearly communicate that the option is coming soon. The current experience should not start an OAuth flow or suggest that sign-in has succeeded.

This is a deliberate deferral, not a decision to remove the options from the product.

### G. The latest change set and its report may not agree

The latest commit report describes the work as non-UI remediation and says the UI was not touched. However, the commit contains changes to a number of UI-related files.

The investigation should establish what actually changed, whether those changes were intended, and whether the current UI and architecture remain aligned with the approved scope. Do not assume that a report accurately describes a change set without checking it.

### H. The current baseline and earlier deliverables need to be reconciled

Some existing patch deliverables and reports were prepared against earlier repository revisions. The latest remediation report also notes that its patch baseline differed from the current baseline.

The investigation should identify which documents and patches still apply, which are historical, and what the actual current baseline is. No earlier patch should be treated as safe to apply merely because it exists in the deliverables folder.

## 4. Product and system boundaries to preserve

The investigation should keep the already established product boundaries in view and confirm that the current architecture still reflects them:

- Athletica is an affiliate marketplace. Amazon owns the retail transaction.
- Athletica is not intended to become the merchant of record.
- A first-party cart, checkout, payment processing, shipping, inventory, or order-management system is outside the agreed product scope.
- The active application is `rollout/`. The old frontend may be useful as reference material, but should not automatically be treated as the active application.
- The standalone Sanity Studio is no longer required as an active workspace. The use of Sanity APIs, where needed by the active application, is a separate matter.
- Google and GitHub sign-in remain planned interface options but are deferred, with a clear coming-soon response.
- Existing UI work and planned UI features should not be discarded simply because their supporting backend work is incomplete. Their actual readiness and dependencies need to be made clear.

If the current code or documents conflict with these boundaries, record the conflict for a decision instead of silently choosing a direction.

## 5. What the investigation should establish

By the end of its investigation, the OpenSpec agent should be able to explain, with evidence:

1. What the current application and architecture actually contain.
2. Which of the initial findings above are confirmed, partly correct, outdated, or incorrect.
3. What important problems or missing decisions were not listed here.
4. Where product intent, technical specifications, database structure, service contracts, and application behaviour disagree.
5. Which existing specifications remain valid and which are incomplete, conflicting, or out of date.
6. Which decisions are already settled and which still need the owner's confirmation.
7. What the system should look like when the identified gaps have been addressed, expressed as outcomes and observable behaviour rather than premature implementation choices.

The investigation should distinguish confirmed facts from assumptions, unresolved questions, and matters that require runtime or owner-side verification.

## 6. Required next deliverable: a proposed architecture/specification plan

**The plan is to be written after the investigation is complete.** It should not be written from this draft alone.

After investigating, the OpenSpec agent should prepare a separate plan for review. The plan should explain:

- The issues and specification gaps it proposes to address.
- The intended outcome of each proposed piece of work.
- Which architecture documents and specifications would need to change.
- How the work should be grouped into moments or phases, based on the actual dependencies discovered.
- What each moment or phase is expected to settle or produce.
- Dependencies, risks, unresolved questions, and decisions that require owner input.
- How the agent will know that the architecture and specifications are aligned when the work is complete.

The plan should describe the proposed work and its expected results. It should not prescribe code-level implementation steps or choose technical solutions that have not been justified by the investigation.

## 7. Review gate — no specification changes or implementation yet

The proposed plan must be returned for owner review before the OpenSpec agent begins updating the architecture or specifications.

The owner and reviewer will check the plan together, request changes where needed, and continue reviewing it until its scope, outcomes, order, and boundaries are agreed.

Until that approval is given:

- No application code changes.
- No database or migration changes.
- No implementation work.
- No automatic execution of the proposed plan.
- No rewriting specifications based on unconfirmed assumptions.

After the plan is approved, the OpenSpec agent may carry out the agreed architecture and specification work. That work must still remain separate from application implementation. Development begins only after the updated architecture and specifications have been reviewed and accepted.

---

## 8. Expected result

The expected result of this process is one coherent, agreed technical blueprint for Athletica.

It should accurately reflect the intended product, the real state of the current application, the required system behaviour, and the technical decisions needed to support that behaviour.

The developer should later be able to work from the approved specifications without having to guess what the system is supposed to do or make important architectural decisions during implementation.
