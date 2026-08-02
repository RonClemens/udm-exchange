# SE Workbench — PKM Migration Status Report

**Version:** 2.1.1
**From:** SE Workbench implementation session ("PDR Reconciliation & Baseline Alignment Workbench")
**Reports against:** PKM Migration Plan v0.7.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/migration-plans/se-workbench/PKM_MIGRATION_PLAN.md)), PKM Entity & Relationship Model v0.7.3 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md)), Architecture Guidance v1.7.1 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md))
**Changelog:**
- v2.1.1 (2026-08-02) — marked §14 items 9 and 10 resolved upstream, per udm-exchange's confirmation: PKM Entity Model v0.7.3 §4 now documents the closed-union-vs-plain-string convention item 9 raised, citing this app's `Comment` implementation by name; Architecture Guidance v1.7.1 §8.1 now explicitly blesses build-time JSON import as equally conformant to runtime `fetch()`, citing this app's dual deploy modes as the motivating case. Updated `Reports against:` to Entity Model v0.7.3 / Architecture Guidance v1.7.1.
- v2.1.0 (2026-08-01) — added §13, documenting an Architecture Guidance §8.1 compliance fix flagged directly by design chat: this app's footer was displaying stale, literally-copy-pasted example version values (v1.4.0, two version bumps of undetected drift). Re-vendored to v1.7.0, replaced hardcoded TS constants with the new `/data-schema/PKM_VERSIONS.json` single source of truth, and added the same object as a `meta` block to both data-export features. Renumbered former §13 (optional follow-ups) to §14, added one new item. Updated `Reports against:` Architecture Guidance to v1.7.0 (was citing v1.6.0, one revision behind).
- v2.0.0 (2026-08-01) — added §12, documenting Migration Plan v0.7.0 Step 15: `Comment` implemented as a first-class entity with both surfaces of its Architecture Guidance §13 UI pattern (inline detail-view affordance and a global list view), backend and UI together per this step's own deliberate sequencing. Renumbered former §12 (optional follow-ups) to §13. Updated `Reports against:` to Migration Plan v0.7.0 / Entity Model v0.7.0, and added Architecture Guidance v1.6.0 to this header (first step whose guidance lives partly outside the Migration Plan/Entity Model pair).
- v1.9.1 (2026-07-30) — two small fixes, both flagged by design chat's v1.9.0 review: (1) a fresh check run directly against this repo's `mock-data/seed.json` (not the live demo) at the current HEAD (`0909c24`) re-confirms Step 11's finding — `baselineId`/`projectId` are 0/25 null across all four entity types (CI, Specification, SafetyDeliverable, ProgramPlanningDeliverable) — same result as the v1.9.0 check, now against a named, reproducible commit rather than "current data" generically. (2) corrected a citation typo in §11 and §12 item 7: "PKM Entity Model v0.6.0 §79–82" should have read "§2–§3" (RiskItem's actual entity-table row and design-rationale sections) — a copy-paste artifact from this app's own drafting, not anything design chat introduced.
- v1.9.0 (2026-07-30) — added §11, documenting Migration Plan v0.6.0 Steps 12 and 14 item 2 (implemented) and a factual correction on Steps 11, 13, and 14 items 1/3 (already resolved in this app's real data before the ACTION was sent — the export design chat cross-checked appears to predate this app's own recent redeploy). Also documents SEMP-generation proposal Phase D's approved slice (`RiskItem` wired into SEP Outline 3.2.1). Renumbered former §11 (optional follow-ups) to §12, added one new item. Updated `Reports against:` to Migration Plan v0.6.0.
- v1.7.0 (2026-07-29) — added §9, documenting this app's own Step 11: implemented `Role` as a first-class entity per Migration Plan Step 9 / PKM v0.5.0 §2–§3, resolving Migration Plan §10 item 3. Renumbered former §9 (optional follow-ups) to §10 and retired item 2 there (superseded by this step). Updated `Reports against:` to Migration Plan v0.4.0 / Entity Model v0.5.0.
- v1.6.0 (2026-07-29) — added §8, documenting this app's own Step 10: a first-slice decomposition of `Specification.sections` into `Requirement`/`ChecklistItem`/`VerificationEvent` records, per design chat's ACTION item 2 (closing the gap Steps 4 and 5 both explicitly deferred). Renumbered former §8 (optional follow-ups) to §9.
- v1.5.0 (2026-07-29) — added §7, recording the standalone `AcquisitionMilestone` entity's actual removal (design chat ACTION item 1: the coexist-then-deprecate window from Step 9/§6 is now closed, not just verified-closeable). Renumbered former §7 (optional follow-ups) to §8. Updated `Reports against:` to PKM Migration Plan v0.3.1 / Entity Model v0.4.0 (both revved since v1.4.0 — the `ReconciliationEvent` addition; documentation-only from this app's side per the Migration Plan's own v0.3.1 note, no action taken here).
- v1.4.0 (2026-07-29) — added §6, documenting Migration Plan Step 8 (this app's own numbering: Step 9): consolidated the standalone `AcquisitionMilestone` entity into `Milestone` via `milestoneType`, per the canonical model's v0.3.0 broadening. Answers Migration Plan §9 item 5 (coexistence-window deadline). Renumbered former §6 (optional follow-ups) to §7. Also corrected this header's stale `Reports against:` versions (was citing Migration Plan v0.2.0 / Entity Model v0.2.1, two and three revisions behind), per the Migration Plan's own §9 item 5 context and this repo's feedback-staleness convention.
- v1.3.0 (2026-07-28) — added §5, documenting PKM Migration Step 8 (AAF Milestone A/B/C occurrence tracking) — a new `AcquisitionMilestone` entity — plus a direct answer to §5 optional-follow-up #6 from v1.2.0 (whether acquisition-phase/pathway concepts belong in the PKM model). Renumbered former §5 (optional follow-ups) to §6 and updated item #6's own text to reflect the answer.
- v1.2.0 (2026-07-27) — added §4, documenting the Acquisition Phase Workbench (new default guided navigation). No PKM entity/schema change beyond one additive evidence-type enum value. Renumbered former §3 (optional follow-ups) to §5 and added one new follow-up item.
- v1.1.0 (2026-07-27) — added §3, documenting a PDKM Promises UI redesign (grouped/collapsible/searchable). No entity/schema change.
- v1.0.0 (2026-07-26) — initial report: all 7 migration steps implemented, verified, deployed.

**Status:** All 7 original migration steps, this app's own Step 8 (AcquisitionMilestone, since retired — §7), Step 9 (its consolidation into Milestone), Step 10 (a first-slice `Specification.sections` decomposition — §8), Step 11 (`Role` as a first-class entity — §9), Step 12 (`RiskItem` as a first-class entity — §10), Steps 13-14 (§11: `ReconciliationEvent` entity, targeted backfills, and SEMP Phase D's `RiskItem`→3.2.1 wiring), and Step 15 (`Comment` entity plus its Architecture Guidance §13 UI pattern — §12), are implemented, verified, and deployed. This is a status/handoff report, not a request for new feedback on the plan itself — flagged items at the end are optional follow-ups, not blockers.

---

## 1. What shipped

Each step below is implemented, has passing server/client builds (both server-backed and static deploy modes), a live-server smoke test, and a Playwright UI smoke test, and is deployed to the app's static demo.

| Step | Entity/Change | Notes |
|---|---|---|
| 1 | `Program`, `Project` | Additive. One Program, one Project, matching this app's existing single-program scope. |
| 2 | `Baseline` (enum → entity) | Coordinated with the app's own methodology/data content-split for the recovery-program guidance file. `reconciledFromBaselineId`/`reconciledIntoBaselineId` used per PKM §5 open question #1; `AbCompatibilityRow` left unmigrated, per the plan's own carve-out. |
| 3 | `Milestone` | One record per SETR event per baseline lineage (Baseline A and B run independent timelines). Generic event definitions stay in the methodology layer; only per-baseline status/dates live on the entity, per the plan's explicit methodology/data split for this step. |
| 4 | `Requirement` | `satisfiedByCiIds` modeled many-to-many, mirroring `ConfigurationItem.subsystemIds` — real seed data (`delta-001`) is an actual case of one requirement whose as-built satisfaction spans three CIs, not the single CI its paper allocation named. `parentRequirementId` demonstrated via a second requirement that is implicitly part of the first. |
| 5 | `ChecklistItem`, `VerificationEvent` | First slice only, per the plan's own risk note (this is content authoring, not mechanical conversion). Covers the two currently in-progress milestones and one COTS verification-method conversion. `ChecklistItem` uses a polymorphic `evidenceType`/`evidenceId` reference. `domain` is a plain string attribute, left provisional pending PKM §5 open question #2. |
| 6 | `Gap` | First slice only. Demonstrates real unification, not just new structure: one existing finding was already tracked by two separate mechanisms (a CI's own over-decomposition flag and a Delta Matrix row) and both now reference the same `Gap` record. `Recommendation` deliberately not touched here — see Step 7. |
| 7 | ActionItem role enforcement | `Recommendation.owner` converted from free text to a fixed role taxonomy (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager) — a pragmatic starting cut, not a definitive one. Added `resolvesGapId`, coexisting with the existing `relatedCiId`. |

**Consistent pattern across all 7 steps:** additive-only, coexist-then-deprecate. No existing field, table, filter, or export was removed or restructured — every new entity/reference sits alongside what already worked. This kept the migration's actual regression surface at zero across all 7 steps' UI/build/smoke verification.

## 2. Also shipped this pass (not in the migration plan, but related)

- **`@domain-placeholder` marker convention** — every field across every entity whose *value* is program-specific illustrative content (as opposed to structural fields: ids, references, fixed enums, timestamps) is now tagged in both type files, with a manifest doc explaining the convention and listing every tagged field per entity.
- **PDKM Promises tab** — a read-only, filterable browser over every current record's `@domain-placeholder` values (including nested attachments, qualified alternates, and specification sections), framed as a promissory note: each value is synthetic and will be replaced once a real Product/Domain Knowledge Model exists for the program a given deployment serves, arriving via landing-zone upload or direct user entry. See §3 for a UI redesign of this tab since v1.0.0.
- **Forward-compatibility note for a future guided/wizard interface** — this app's maintainer indicated an intent to eventually build a "TurboTax-style" prompted-question interface, fed by the same underlying model as the tabbed webapp, that would toggle which CDRLs apply in real time based on DID-derived trigger rules. `ChecklistItem`'s shape (a discrete, user-answerable criterion + toggleable status + structured evidence reference) was designed with this specifically in mind. See §4 — this intent now has its first real, working slice.

## 3. Update since v1.0.0 (2026-07-27): PDKM Promises UI redesign

The Promises tab described in §2 originally rendered as one flat table — one row per (entity, record, field) triple. That flat shape had a real usability problem: on a narrow viewport, the Field and Value columns scrolled out of view, so a record with two tagged fields (e.g. a Milestone's `actualDate` and `plannedDate`) rendered as two visually identical adjacent rows with no visible way to tell they differed. A user flagged this directly as apparent data duplication before the actual cause (a cropped table, not a real bug) was identified.

The tab has been rebuilt around a two-level hierarchy instead of a flat table:

1. **Major group** (collapsible) — a curated consolidation of the ~18 underlying entity/attachment type strings into 11 SE-domain-area groups: Program & Project, Schedule & Milestones, Requirements & Verification, Gaps & Recommendations, Technical Baseline, Traceability & Compatibility, COTS & Parts, Specifications, Safety, Program Planning, Attachments. This grouping is this app's own judgment call, not derived from any PKM or Architecture Guidance document.
2. **Data element** (entity type + record identity, one heading) — nested inside its group, with its own tagged fields rendered as name/value pairs directly underneath it, so a multi-field record is unambiguous at a glance.

On top of that hierarchy: a pill-based multi-select filter row (one pill per major group, plus "All"), and a free-text search across entity/record/field/value. Search narrows to matching leaf rows only — it does not flatten the view. A match always stays visible inside its full group/element header context (and force-expands a collapsed group), rather than surfacing as a bare row with no structural context.

No entity, field, or `@domain-placeholder` tagging changed — this is a presentation-layer change only, over the same field set from the Step 5-era manifest work in §2.

This was offered as a concrete reference point for a future shared cross-app Promises UI (per this app's own earlier proposal-response feedback on the `@domain-placeholder` convention), not a request for canonical guidance to change.

## 4. Update since v1.1.0 (2026-07-27): Acquisition Phase Workbench

This app's default navigation is no longer a flat tab list. It's now a "left-to-right," time/system-maturity-phase-driven guided workbench, built around the DoD Adaptive Acquisition Framework's Major Capability Acquisition (MCA) pathway: Materiel Solution Analysis → Technology Maturation & Risk Reduction → Engineering & Manufacturing Development → Production & Deployment → Operations & Support.

**Phase taxonomy.** A new methodology module (`aafPhaseGuidance.ts`) bands the same 8 SETR events this app already tracks (SRR through PRR) under those 5 MCA phases — the same banding technique this app's own TDP/IEEE-12207 guidance already uses to group those identical 8 events by TDP maturity level and software life-cycle stage. This is a coarser lens over existing data, not a new competing taxonomy. Materiel Solution Analysis and Operations & Support are explicit, visibly-marked **stub phases**: this app's SETR modeling only spans SRR–PRR, so those two phases surface an out-of-scope note rather than fabricated content.

**Pathway extensibility, deliberately unbuilt further than this.** The pathway type is a one-member union (`"MCA"` only) — `Record<Pathway, Phase[]>`-shaped so a second AAF pathway (e.g. Software Acquisition Pathway) could be added later without reshaping this data. No pathway-selection UI, no PDKM-driven intake flow, and no `pathway` field on any entity exist yet — that would be schema/UX churn for a value that only ever has one option today. Flagged in case this is relevant context for any future PKM discussion of whether acquisition-pathway/phase concepts belong in the PKM model itself, or stay purely app-side methodology content — no position taken either way here; see §5.

**Baseline-scoped, derived, not stored.** A baseline's "current phase" is computed client-side from its existing `Milestone` records (first SETR event that isn't yet `Complete`) — no new stored field, no schema change. Baseline A and Baseline B, which already run fully independent SETR timelines in this app's data, correctly show different current phases side by side.

**Guided prompting — the first real consumer of `ChecklistItem`'s forward-compat design.** Viewing a baseline's current phase surfaces a domain-grouped, click-to-answer panel over that milestone's `ChecklistItem` records: status (Not Evaluated/Met/Not Met/Waived) saves immediately on click, and a modal form covers full-record create/edit. This is exactly the "TurboTax-style" interaction `ChecklistItem`'s own shape was designed for back at PKM Migration Step 5 (§2 above) — nothing about its schema needed to change to support it.

**Coexist, not replace.** The original 12-tab bar is fully preserved, byte-for-byte unchanged, reachable via an "All Tabs" toggle — same coexist-then-deprecate pattern as all 7 migration steps in §1. Nav-mode choice (guided workbench vs. all-tabs) persists per-browser.

**Schema footprint:** one additive enum value only — `ChecklistItemEvidenceType` gained `"Specification"` (spec-related criteria had no valid evidence type to point at). Nothing else in the entity model changed.

**Verification:** clean typecheck/build in both workspaces; both server-backed and static (`localStorage`) deploy modes smoke-tested end-to-end via Playwright with zero console errors, including default-landing, baseline/phase switching, guided-panel status persistence across reload, Edit Mode overrides on the new guidance prose, and confirmation that two unrelated existing tabs are unaffected.

## 5. Update since v1.2.0 (2026-07-28): AAF Milestone A/B/C occurrence tracking (PKM Migration Step 8)

This directly answers §6 optional-follow-up #6 from v1.2.0: whether an acquisition-phase/pathway concept belongs
in the PKM model itself. Short answer, with the actual gap this surfaced:

- **`AcquisitionPathway` needs no data-layer entity.** It's already PKM-conformant as a plain, stable, external,
  human-meaningful id (`"MCA"`) per PKM §4 / Architecture Guidance §9 — its name, definition, and phase banding
  are DoD Adaptive Acquisition Framework doctrine, identical for every program that uses MCA, not per-program data
  the way a `Program` or `Project` record's name/description is. This is an "already conformant, no entity needed"
  finding, the same category PKM Migration Step 0 recorded for CI↔LogicalSubsystem cardinality — flagged here
  rather than left to be rediscovered.
- **`AcquisitionPhase` (MSA/TMRR/EMD/PD/OS) also needs no stored entity.** A baseline's "current phase" is already
  fully and cheaply derived from its own `Milestone` records (first SETR event not yet `Complete`) — this app's
  existing "derived, not stored" design choice, unchanged by this pass. Storing it would create a second source of
  truth that could silently drift from the `Milestone` records that already determine it, for no new information.
- **The actual gap: AAF's own decision gates (Milestone A/B/C) had no occurrence data at all.** Unlike SETR events
  (SRR–PRR), which PKM Migration Step 3 already promoted from static definitions into real per-baseline `Milestone`
  records (status, dates), MS-A/B/C existed only as static catalog metadata
  (`methodology/guidance/aafPhaseGuidance.ts`'s `MCA_MILESTONE_GATES`) with no way to record whether a given
  baseline had actually passed a given gate, or when — the same gap SETR events had before Step 3. This pass adds
  a new entity, `AcquisitionMilestone` (`id`, `event: "MS-A"|"MS-B"|"MS-C"`, `pathway`, `baselineId`, `status`,
  `actualDate`, `plannedDate`), following the identical Step-3 promotion pattern, seeded for both baselines
  (Baseline A: all three gates `Complete`, consistent with its current phase already being Production &
  Deployment; Baseline B: only Milestone A `Complete`, consistent with its current phase still being Technology
  Maturation & Risk Reduction).
- **Deliberately a separate entity from `Milestone`, not a broadened `MilestoneEvent` union.** MS-A/B/C are
  acquisition-decision events (DAU/AAF doctrine — resourcing and program-level authorization to proceed), not SE
  technical reviews, and `MILESTONE_EVENTS`' fixed SRR–PRR ordering is load-bearing for this app's
  `deriveCurrentMilestone()` ("first non-Complete gate in canonical order") — folding AAF events into that
  array/type would require re-deriving that ordering assumption for no benefit, since AAF milestones already
  relate to SETR events structurally via each phase's `entryMilestone`/`exitMilestone` fields, not by shared
  identity.
- **UI**: the Acquisition Phase Workbench's entry/exit gate display (previously static name + decision-summary
  text only) now shows each gate's real per-baseline status via the same click-to-set status-pill interaction the
  CDRL panel below it already uses — no new interaction pattern introduced.

**Schema footprint:** one additive entity (`AcquisitionMilestone`), zero changes to any existing entity or field.
Coexist-then-deprecate is not applicable here since nothing existing is superseded — this is pure net-new
structure filling a gap that had no prior representation at all, not a promotion of an existing free-text field
(contrast with Steps 2, 3, 5, and 7, which each had a prior free-text/enum field to coexist alongside).

**Verification:** clean typecheck/build in both workspaces; Playwright smoke test covering default-landing,
gate-status click-to-set on both baselines, persistence across reload, the existing All Tabs toggle, and the
PDKM Promises tab rendering the new entity's group without errors — zero console errors, zero regressions in
unrelated tabs.

## 6. Update since v1.3.0 (2026-07-29): Consolidated AcquisitionMilestone into Milestone (Migration Plan v0.3.0 §8; this app's own Step 9)

Implements the new canonical guidance from PKM Migration Plan v0.3.0 §8, which folded the standalone
`AcquisitionMilestone` entity (this app's own Step 8, §5 above) into the broadened `Milestone` entity — one
entity with a `milestoneType: "SETR" | "AcquisitionGate"` discriminator, per PKM Entity Model v0.3.0/v0.3.1's own
correction (informed directly by this app's Step 8 work). This is the first step in this plan's whole history to
run in the opposite direction from every one before it — canonical guidance *to* this app, not a report *of* work
this app already did independently.

- **`Milestone` gained `milestoneType` and `pathway`.** `milestoneType: "SETR"` backfilled on all 16 pre-existing
  records (the only value that ever existed); `pathway: null` likewise, since SETR records have no pathway
  concept. Six new `Milestone` records with `milestoneType: "AcquisitionGate"` were added — a 1:1 migration of the
  six former `AcquisitionMilestone` rows (same `event`, `pathway`, `baselineId`, `status`, `actualDate`,
  `plannedDate`; new ids consistent with this array's own naming convention).
- **`establishesBaselineId` semantics (PKM v0.3.1 §3):** not added as a new field — this app already models that
  relationship in the *reverse* direction (`Baseline.establishedAtMilestoneId`, Step 2), populated only with
  `milestoneType: "SETR"` record ids in practice. No structural change needed to preserve the type-dependent
  semantics the canonical model describes.
- **`deriveCurrentMilestone()` and `milestoneStatusesForPhase()` scoped explicitly to `milestoneType: "SETR"`**,
  per the plan's own instruction — `MILESTONE_EVENTS`' SRR–PRR ordering still means nothing for
  `AcquisitionGate` records. Behaviorally unchanged from before this consolidation.
- **Coexist-then-deprecate, same discipline as every step so far.** The standalone `AcquisitionMilestone` type,
  its seed data, its CRUD API route (`/api/acquisition-milestones`), and its client entity export
  (`acquisitionMilestonesApi`) are all still present, untouched, and independently fetchable — not removed. What
  changed is which source the UI reads: the Acquisition Phase Workbench's gate-status panel and the PDKM
  Promises tab now read/write the consolidated `Milestone` records (`gateMilestoneFor()`, replacing the retired
  `acquisitionMilestoneFor()`) instead of the standalone entity. This app's `App.tsx` also stopped actively
  fetching the standalone entity, since nothing renders it anymore — the type/table/route themselves are
  untouched, only the now-genuinely-dead client-side fetch was removed.
- **PDKM Promises tab:** the "Schedule & Milestones" group's `Milestone` rows now include the six
  `AcquisitionGate` records alongside the sixteen `SETR` ones (44 tagged values total, up from 32) — one
  `rowsFor("Milestone", ...)` call surfaces both, so the separate "Acquisition Milestone" group is gone rather
  than duplicating now-redundant data. `data-schema/DOMAIN_PLACEHOLDER_FIELDS.md` merged accordingly: one
  `Milestone` section (noting the broadened scope), `AcquisitionMilestone`'s former section marked
  `Superseded by: Milestone`, not deleted.
- **Answers Migration Plan §9 item 5** (does the coexistence window need a hard deadline?): **no fixed deadline
  needed** — "until the gate-status UI is confirmed reading from consolidated `Milestone`" is sufficient as a
  completion criterion on its own, and that criterion is now met (verified below). No further action is expected
  on the deprecated `AcquisitionMilestone` table from this app's side; it's inert until a future pass removes it
  outright, whenever that's judged worthwhile.

**Schema footprint:** two additive fields on `Milestone` (`milestoneType`, `pathway`), six additive records. Zero
fields removed from `Milestone` or `AcquisitionMilestone`; zero records deleted.

**Verification:** clean `tsc -b` build in both workspaces (caught two real type errors from the broadened
`MilestoneEvent` union during implementation — `SetrEvent`-scoped `.includes()` calls needed explicit narrowing —
both fixed with justified casts, not `any`/`as unknown`). Playwright smoke test: gate-status click-to-set on both
baselines against the consolidated entity, persistence confirmed via a direct API read after each click, the
PDKM Promises tab rendering 44 unified values with no separate Acquisition Milestone group, zero console errors,
zero regressions in unrelated tabs.

## 7. Update since v1.4.0 (2026-07-29): AcquisitionMilestone retired

Per design chat's ACTION item 1 (2026-07-29 1615 UTC batch): "coexist-then-deprecate is complete,
this is the actual deprecation step." Confirmed nothing referenced the standalone entity before
removing it — App.tsx, the Phase Workbench gate display, and the PDKM Promises tab all already
read the consolidated `Milestone` entity as of Step 9 (§6) — then removed outright:

- The `AcquisitionMilestone` type, `AcquisitionMilestoneEvent` union, and `ACQUISITION_MILESTONE_EVENTS`
  const (both type-mirror files).
- The `acquisitionMilestones` field on `Database` (both type-mirror files).
- Seed data: the `acquisitionMilestones` array (`mock-data/seed.ts` and `mock-data/seed.json`).
- The CRUD API route (`/api/acquisition-milestones`) and its `REQUIRED_KEYS` entry.
- The client entity export (`acquisitionMilestonesApi`) and its `localStorage`-mode wiring
  (static/GitHub-Pages deploy path).
- `data-schema/DOMAIN_PLACEHOLDER_FIELDS.md`'s now-empty `AcquisitionMilestone` section, folded
  into `Milestone`'s.

Nothing about `Milestone` itself changed in this pass — this is pure removal of what Step 9 had
already superseded, not a further schema change.

**Verification:** clean `tsc -b` build in both workspaces; a live-server check confirming
`GET /api/acquisition-milestones` now returns `404`; Playwright smoke test covering
default-landing, gate-status display (reading `Milestone` correctly), the All Tabs toggle, and
the PDKM Promises tab still rendering AAF gate records under the unified `Milestone` group — zero
console errors, zero regressions.

## 8. Update since v1.5.0 (2026-07-29): Specification.sections decomposition (this app's Step 10, first slice)

Per design chat's ACTION item 2 (2026-07-29 1615 UTC batch): "doesn't depend on any currently
open PKM question... proceed whenever convenient, same content-authoring risk profile as Step
5's first slice." Closes the gap Steps 4 and 5 both explicitly flagged and deferred at the time
("the higher-effort part of this phase... a separate, later sub-phase").

- **Scope:** one representative Specification (`spec-002`, the Test Set CI Development spec),
  not a sweep across all five — same first-slice discipline Step 5 used (two milestones, one
  COTS conversion), not a full pass.
- **`Requirement` gained two additive, nullable fields:** `sourceSpecificationId`,
  `sourceSpecSection` (the latter a fixed `SpecSectionKey` enum) — traces a decomposed
  requirement back to where it came from. Both null for requirements that predate this step
  (`req-001` through `req-004`, which came from `DeltaMatrixRow`/`CotsRecord` content instead).
- **`functionalPerformance` → two new `Requirement` records.** `spec-002`'s two "shall"
  statements ("shall generate and capture UUT stimulus/response signals..."; "shall format
  diagnostic messages...") are now `req-005`/`req-006` — children of `req-001` (the system-level
  flow-down this spec's CI-level content elaborates), satisfied by `ci-001` (the CI `spec-002`
  is linked to).
- **`verificationProvisions` → one new `VerificationEvent`.** This section's text names two
  verification methods: "verified by test" (not represented anywhere) and "verified by
  inspection of vendor data sheet" (already covered by `ve-001`, Step 5, via `CotsRecord`). Only
  the genuinely missing half became `ve-003`, against `req-005`.
- **`ChecklistItem` demonstrates the actual payoff.** `check-008` evidences `req-005`
  individually (`evidenceType: "Requirement"`) — a criterion that could not have existed before
  this step, since `req-005` didn't exist as its own record; only as free text inside a
  Specification section with nothing to point a checklist criterion at.

**Schema footprint:** two additive fields on `Requirement`, two new `Requirement` records, one
new `VerificationEvent`, one new `ChecklistItem`. Zero fields removed, zero existing records
changed.

**Verification:** clean `tsc -b` build in both workspaces; live-server API checks confirming the
new record counts; Playwright pass covering the guided checklist panel (Baseline A's current
TRR milestone surfaces `check-008`), the Specifications tab, and the PDKM Promises tab
(surfacing the two new requirement statements) — zero console errors, zero regressions.

## 9. Update since v1.6.0 (2026-07-29): Role implemented as a first-class entity (this app's Step 11)

Per Migration Plan v0.4.0 Step 9 (canonical guidance *to* this app, same direction as Step 8):
implements `Role` per PKM Entity Model v0.5.0 §2–§3, resolving Migration Plan §10 item 3 and this
report's own §10 item 2 below.

- **New `Role` entity:** `id`, `projectId`, `name`, `authorityDescription` (optional), `isDefault`
  (boolean). Seeded with the existing five-role set (Lead Systems Engineer, CM Lead, Software
  Lead, Safety Lead, Program Manager) as `isDefault: true` — the exact taxonomy from Step 7,
  preserved as data instead of a hardcoded union.
- **New CRUD route** (`/api/roles`), same pattern as every other entity.
- **`Recommendation.assignedRoleId`** references a `Role` record; `owner` coexists, unmodified,
  not removed. Backfilled once for all three existing `Recommendation` records via their `owner`
  value — an unambiguous one-time mapping, not an ongoing sync.
- **Admin surface:** `RecommendationsPage.tsx`'s assignment dropdown now reads live `Role`
  records instead of the retired hardcoded array — the actual point of this step, not just a
  structural reference. A minimal inline "Manage Roles" panel (add/edit/delete) reuses the same
  DataTable/EntityForm/Modal components every other entity page already uses — no new UI
  patterns introduced.
- **`@domain-placeholder` reclassification:** `Role.name`/`authorityDescription` are newly
  tagged. Reversal from how these values were treated as `RecommendationOwnerRole` (a structural
  taxonomy, untagged, like `ChecklistItem.domain`) — promoting them to a real, program-tailorable
  entity flips the classification, since they're no longer fixed structure.

**Schema footprint:** one additive entity (`Role`, 5 seeded records), one additive field on
`Recommendation` (`assignedRoleId`). Zero fields removed, zero existing records restructured.

**Verification:** clean `tsc -b` build in both workspaces; live-server API checks confirming
seeded roles and backfilled `assignedRoleId` values; Playwright pass confirming a newly-added
custom role persists and is immediately selectable in the assignment dropdown (the actual
tailorability this step exists to provide), the PDKM Promises tab surfaces `Role` values, zero
regressions in unrelated tabs.

## 10. Update since v1.7.0 (2026-07-29): `RiskItem` implemented as a first-class entity (this app's Step 12)

Per Migration Plan v0.5.0 Step 10 (canonical guidance *to* this app): implements `RiskItem` per PKM
Entity Model v0.6.0 §2–§3, closing the gap this app's own `SEMP_GENERATION_STATUS_REPORT.md` §2
flagged (2026-07-29 2353 UTC) — that proposal document's v0.4.0 claim that Workbench was "the
proven implementation partner for Role and RiskItem alike, both verified end-to-end" was not yet
accurate for `RiskItem` at the time it was flagged. It is now.

- **New `RiskItem` entity:** `id`, `projectId`, `itemType` (`Risk` | `Issue` | `Opportunity`,
  same discriminator pattern as `Milestone.milestoneType`'s `SETR`/`AcquisitionGate` split),
  `category` (free string, deliberately left untagged/unconstrained — same treatment as
  `ChecklistItem.domain`), `likelihood` (1-5, `null` for `itemType: "Issue"` — this app's own
  reading of an issue as "already occurred," so probability isn't independently scored),
  `consequenceCost`/`consequenceSchedule`/`consequencePerformance` (1-5 each), `mitigationStrategy`
  (`Accept` | `Avoid` | `Transfer` | `Control`), `ownerRoleId` (references `Role`, per Step 11),
  optional `linkedMilestoneId`/`linkedCiId`, four lifecycle date fields, `status` (`Identified` |
  `Approved` | `Mitigating` | `Closed` — this app's own reading of the entity's own lifecycle
  dates, not a literal quote from the canonical spec). `description` is this app's own addition
  (not literally named in the PKM spec), following `Requirement.statement`'s precedent for
  entities that need real illustrative content, not just structural fields.
- **`riskLevel` is derived, not stored** — `client/src/utils/riskItem.ts`'s `deriveRiskLevel()`
  computes `likelihood x max(consequence dimensions)` per a standard 5x5 risk-matrix banding
  (1-4 Low, 5-9 Moderate, 10-14 High, 15-25 Critical), treating a null `Issue` likelihood as 1 for
  this derivation only — same "derived-not-stored" pattern already used for Acquisition Phase and
  Baseline reconciliation status.
- **New CRUD route** (`/api/risk-items`), same pattern as every other entity. New `RiskItemsPage.tsx`
  (direct CRUD, no admin-picklist wrapper needed) — the four 1-5 score fields use `type: "select"`
  with string options, converted to numbers at the form boundary, mirroring `CotsRecordsPage.tsx`'s
  own textarea-to-array conversion pattern (`EntityForm` has no native numeric input type).
- **`Gap.escalatedToRiskItemId`** and **`Recommendation.resolvesRiskItemId`** (both new, optional,
  nullable): coexist with the pre-existing `blocksMilestoneId`/`resolvesGapId` references, not a
  replacement for either. Seed data demonstrates both in use: `gap-003` escalates to `risk-001`,
  and a new `rec-004` resolves `risk-001` directly (distinct from `rec-002`, which already
  resolves the same underlying finding on the Gap side — a Recommendation resolves a Gap *or* a
  RiskItem, never both, per this field's own comment).
- **Seed data** covers all three `itemType` values: `risk-001` (`Risk`, escalated from `gap-003`),
  `risk-002` (`Issue`, `likelihood: null`, the Baseline A/B serial-protocol divergence already
  tracked in `ab-001`/`rec-003`), `risk-003` (`Opportunity`, a parallel-test-session idea arising
  from Baseline B's Ethernet move).

**Schema footprint:** one additive entity (`RiskItem`, 3 seeded records covering all itemType
values), one additive field each on `Gap` and `Recommendation`. Zero fields removed, zero existing
records restructured.

**Verification:** clean `tsc -b` build in both workspaces; live-server API check confirming seeded
`RiskItem` records round-trip correctly (including `likelihood: null` for the `Issue` record); a
Playwright pass confirming the derived risk-level column renders the correct band for all three
seeded records (High/Low/Low, matching the 5x5 matrix math by hand), the `Issue` record's edit
form correctly shows its likelihood field as unset (not defaulted to a stray value), a full
create→verify→delete round-trip through the real form (not just direct API calls) leaves the
seed data unchanged, and zero regressions in unrelated tabs (including the PDKM Promises tab,
which now surfaces `RiskItem.description` under "Gaps & Recommendations").

## 11. Update since v1.8.0 (2026-07-30): Migration Plan v0.6.0 Steps 12/14 item 2, SEMP Phase D — plus a factual correction on Steps 11/13/14 items 1&3

**Real work done, per the ACTION relayed 2026-07-30:**

- **Step 12 (`ReconciliationEvent`):** implemented exactly as specified. New entity (`fromBaselineId`, `intoBaselineId`, `status`, `initiatedDate`, `completedDate`, `evidenceEntityType`/`evidenceEntityIds`), full CRUD route. One record created from the real data: `fromBaselineId: "BASELINE-B"`, `intoBaselineId: "BASELINE-A"`, `status: "In Progress"`, `evidenceEntityIds: ["ab-001", "ab-002"]`. `Baseline.reconciledFromBaselineId`/`reconciledIntoBaselineId` removed from both type-mirror files — this app's one real code consumer (`findReconciliationTargetBaseline()` in `recoveryProgramGuidance.ts`, used by the CI Inventory tab and the SEMP export) now derives the same fact by querying `ReconciliationEvent` instead, same derived-not-stored pattern as Acquisition Phase.
- **Step 14 item 2:** `rec-003.resolvesRiskItemId` backfilled to `"risk-002"` — `risk-002`'s own description already names `rec-003` as its mitigation plan.
- **SEMP-generation proposal Phase D (partial, RiskItem → SEP Outline 3.2.1 only):** new `buildRiskRegister()` (`client/src/utils/sempExport.ts`), same structured-assembly pattern as `buildMilestoneSchedule()` (Phase C) — a Risk/Issue/Opportunity register table, sorted by derived risk score descending, assembled directly from `RiskItem` records. Wired into both the downloaded SEMP Migration package and a new on-screen "Generated SEMP: Risk Register" section. Section 3.2.1's own source-description text updated to reflect a dedicated register now exists, replacing the old "no dedicated risk-register entity" disclaimer.

**Factual correction, stated plainly (same discipline as Step 12's own correction of design chat's planning gap):** before implementing, this app checked its own real data (both current and the commit that was actually live on the deployed static demo before 2026-07-30) against the ACTION's four other claims, per this exchange's own "verify before acting" convention:

- **Step 11 (`baselineId`/`projectId` on CI/Specification/SafetyDeliverable/ProgramPlanningDeliverable):** checked all four types' real data — **0 null/missing across every record, in this app's current seed data and in the commit that was live on the deployed demo before this ACTION was sent.** This was Migration Plan Step 1/2 work, complete long before this session. No backfill was needed or performed. **Re-confirmed (v1.9.1) via a fresh check run directly against `mock-data/seed.json` at commit `0909c24d8ca2beb4e9467402f328748fc188ee90`, per design chat's own follow-up request:** CI (5 records), Specification (5), SafetyDeliverable (9), ProgramPlanningDeliverable (6) — 25 total, 0 null/missing on either field, on any record. Same result as the original check; not a one-off artifact of how that first check was run.
- **Step 13 (`Milestone.milestoneType`):** already backfilled in this app's current data (0/22 missing) as part of this app's own Step 8/9 work, completed earlier in this same exchange. **However, the commit that was live on the deployed static demo before 2026-07-30 does match the ACTION's finding exactly** (16/16 missing, same record count) — strong evidence the "real backend export" was taken against that older deployed version, not this repo's current state.
- **Step 14 items 1 & 3 (`gap-003.escalatedToRiskItemId`, `assignedRoleId` on all three original recommendations):** both already populated in current data (`gap-003.escalatedToRiskItemId: "risk-001"`; `rec-001`/`002`/`003.assignedRoleId` all set) as part of this app's own Step 11/12 work. The previously-deployed commit lacks `Role`/`RiskItem` entirely, consistent with the same stale-export theory.

**Likely root cause, not confirmed:** this app's GitHub Pages static demo (browser-cached localStorage) was last redeployed 2026-07-28, and wasn't redeployed again until 2026-07-30 (a separate housekeeping fix, unrelated to this ACTION). A browser session that visited the demo in that window would have loaded the pre-Step-8-through-12 app build — explaining Steps 13 and 14 items 1/3's findings exactly, record-count and all. Item 1 (Step 11) doesn't fit this theory as cleanly (`baselineId`/`projectId` predate this whole exchange and are populated in every commit checked), so it may have a different explanation not yet identified. Flagging this now rather than silently redoing already-complete work, and recommending a fresh export against the now-current deployed demo (or directly against this repo's `mock-data/seed.json`) as the more reliable source for any future cross-check — see §13 item 8 below.

---

## 12. Update since v1.9.1 (2026-08-01): `Comment` implemented with its Architecture Guidance §13 UI pattern (this app's Step 15)

Per Migration Plan v0.7.0 Step 15 (canonical guidance *to* this app): implements `Comment` per PKM
Entity Model v0.7.0 §2–§3 and its Architecture Guidance v1.6.0 §13 UI pattern, backend and UI
together in one pass, per this step's own deliberate sequencing (`Comment` applies across every
UDM app, so the UI guidance was drafted before this implementation, not retrofitted after).

- **New `Comment` entity:** `id`, `projectId`, optional `entityType`/`entityId`, `text`, `status`
  (`Open` | `Resolved`), `createdByRoleId` (references `Role`), `createdDate`, `resolvedDate`
  (nullable). One deliberate departure from a literal reading of "reuse `Gap`'s polymorphic
  pattern, don't reinvent it": `Gap.foundInEntityType` is a closed union of the 7 entity types
  `Gap` can be found in, but `Comment`'s own entity-model text says it "may attach to any current
  or future PKM entity" — a closed union would need editing every time a new entity type is
  added, defeating that forward-compatibility. `entityType`/`entityId` are both plain
  `string | null` instead — the two-field polymorphic *mechanism* is reused exactly, the type
  constraint on one of its two fields is not. Flagging this interpretive call explicitly in case
  it should instead be a documented PKM convention (any future polymorphic-attachment field
  meant to be forward-compatible uses `string`, not a closed union) rather than an app-local
  choice — see §13 item 9 below.
- **New CRUD route** (`/api/comments`), same pattern as every other entity.
- **Architecture Guidance §13.1's two surfaces, both built:**
  - **Inline affordance** (`EntityComments.tsx`): a comment-count badge on an entity detail
    view, expanding to a thread with create/edit-own-text/resolve/reopen, `entityType`/`entityId`
    populated from the surrounding page's own record, not user entry. Wired into all three of
    this app's true per-record detail views — CI, LogicalSubsystem, Specification (this app's
    list/table pages don't have individual per-row detail screens the way these three do, so
    there was no fourth or fifth surface to add it to).
  - **Global list view** (`CommentsPage.tsx`): full CRUD via this app's standard
    DataTable/EntityForm/Modal pattern, filterable by `status` and `entityType`, sortable by
    `createdDate` — the only place an unattached `Comment` is visible, per §13.1's own framing.
- **Resolution over deletion (§13.3):** the inline surface's primary action is "Mark Resolved"
  (setting `resolvedDate`), not delete; the global view's delete button carries an explicit
  in-app nudge toward resolving instead. Hard delete still available, not the default reach.
- **No `@domain-placeholder` tagging** (§13.4): `Comment.text` is real content by construction,
  documented in the domain-placeholder manifest with its own rationale (distinct from both
  `ContentEntry`'s "not program data at all" and `ReconciliationEvent`'s "no free-text field"
  reasons — this is the first entity with a genuinely free-text field that still isn't tagged).
- **Deliberately not surfaced on the PDKM Promises tab:** that tab's whole purpose is browsing
  `@domain-placeholder`-tagged synthetic defaults awaiting real content — `Comment` records are
  never synthetic placeholders by definition, so including them there would misrepresent what
  the tab means, not just add noise.
- **Seed data:** three illustrative records covering the shape's real range — one attached to a
  CI (the inline-surface case), one unattached (the global-list-only case), one demonstrating the
  resolved lifecycle.

**Schema footprint:** one additive entity (`Comment`, 3 seeded records), zero existing fields
touched. Zero regression surface on any pre-existing entity or UI.

**Verification:** clean `tsc -b` build in both workspaces; live-server API check confirming seeded
records round-trip correctly (including the unattached, both-null-fields case); a Playwright pass
confirming a full create→resolve round-trip through the real inline UI (not just direct API
calls) on the CI detail view, the comment-count badge rendering correctly on all three detail
pages, the global list correctly showing both attached and unattached rows with working
status/entityType filters, and zero regressions in unrelated tabs.

---

## 13. Update since v2.0.0 (2026-08-01): Architecture Guidance §8.1 compliance fix — re-vendored to v1.7.0

Flagged directly by design chat, not a Migration Plan step (Architecture Guidance / vendoring
discipline, not a PKM entity change): this app's in-app footer was displaying `v1.4.0
(2026-07-27)` — literally the example values from §8.1's *prior* code snippet, never actually
updated since the very first vendoring bump, two real version bumps (v1.5.0, v1.6.0) of
undetected drift. Per design chat's own framing, this was the old §8.1 snippet's own failure mode
(hardcoded example values shipped directly in copy-paste code), not a defect in how this app used
it — stated here for the record, not as a self-criticism.

- **Re-vendored** `/vendor/architecture-guidance-v1.7.0.md`, removing the stale v1.4.0 copy.
- **New `/data-schema/PKM_VERSIONS.json`** — the single source of truth for
  `architectureGuidanceVersion`/`Date` and `pkmEntityModelVersion`/`Date`, replacing the old
  hand-maintained `ARCHITECTURE_VERSION`/`ARCHITECTURE_DATE` TS constants (`client/src/config/architectureVersion.ts`,
  now deleted).
- **One adaptation from §8.1's literal text, flagged rather than silently substituted:** §8.1 says
  "read by the footer at runtime (fetch, not embedded constants)" — a genuine network `fetch()`
  call. This app's footer instead reads `PKM_VERSIONS.json` via a build-time JSON import
  (`import pkmVersions from "../../../data-schema/PKM_VERSIONS.json"`), the same pattern this
  app already uses for other shared repo-root content (e.g. `sempExport.ts`'s methodology
  imports). Reasoning: this app has two deploy modes (server-backed and a static GitHub Pages
  build with no server), and `/data-schema` sits outside `client/public` — making a true runtime
  fetch require either duplicating the file into `client/public` (a second copy to keep in sync,
  the exact problem §8.1 exists to eliminate) or a build step to copy it there. A build-time
  import reads the same single canonical file with zero duplication and updates automatically on
  next build, which satisfies §8.1's actual stated goal ("single source of truth... never two
  places to keep in sync") without the added deploy-mode complexity a literal fetch would need
  here. Flagging as a candidate note for §8.1 itself (a bundled/static SPA may reasonably read
  this file via build-time import rather than runtime fetch) rather than silently deviating — see
  §14 item 10 below.
- **`meta` block added to both of this app's data-export features** (§8.1's own requirement):
  "Export JSON" (`ExportImport.tsx`) and the SEMP Migration package (`sempExport.ts`) both now
  carry the same `PKM_VERSIONS.json` object. Stripped back out on import (`api.importData`) so
  re-importing a previously-exported file doesn't persist a stray `meta` key into either deploy
  mode's store.
- **Every in-repo doc/comment citing the vendored version or path** (seven files) updated to
  v1.7.0.

**Schema footprint:** none — this is a vendoring/tooling fix, not a PKM entity change.

**Verification:** clean `tsc -b` build in both workspaces (including a new `resolveJsonModule`
compiler option and `/data-schema` include-path addition, needed for the JSON import); a
Playwright pass confirming the footer now shows `v1.7.0 (2026-08-01)` and links to the re-vendored
file, the "Export JSON" download includes the correct `meta` block, re-importing that same file
succeeds without error and doesn't persist `meta` into the server's `db.json`, and the SEMP
Migration package's generated Markdown carries the same version line near its top.

---

## 14. Optional follow-ups (not blockers)

These surfaced during implementation and are offered as candidate topics for continued discussion, not requests:

1. **PKM §5 open question #2** (ChecklistItem domain tagging) is still open. This app's implementation used a plain string attribute as a pragmatic default — real usage may clarify whether "Domain" deserves first-class status.
2. ~~**Role taxonomy for ActionItem/Recommendation.owner** (Step 7's own deferred decision) was resolved locally with a five-role starting set...~~ **Resolved by §9 above.** `Role` is now a first-class, program-tailorable entity per PKM v0.5.0 — no longer a cross-app-default question, since the taxonomy is data, not a fixed enum.
3. **Gap's evidence cardinality**: this implementation used a single `foundInEntityType`/`foundInEntityId` reference per Gap (one location per finding). Real data already shows a finding can be "found in" one place while being *referenced by* multiple other mechanisms (the CI flag + Delta Matrix row case in Step 6) — worth a future pass on whether Gap needs to support multiple `foundIn` references, not just multiple things pointing *at* it.
4. ~~**Requirement/VerificationEvent/ChecklistItem decomposition of `Specification.sections`** remains deferred...~~ **Resolved by §8 above** (first slice, spec-002 only — not a full sweep across all five Specifications; the remaining four are still open, but the mechanism is now proven).
5. **Promises UI major-group taxonomy (§3)** is this app's own judgment call, not a PKM-derived structure — flagged in case a future cross-app Promises UI conversation wants to compare notes on grouping approaches.
6. **Acquisition-phase/pathway modeling (§4) — answered in §5 above, resolved.** `AcquisitionPhase` and `AcquisitionPathway` both stay app-side methodology/derived content, not PKM entities (already-conformant-as-a-union and derived-not-stored respectively). The actual gap this question surfaced — AAF Milestone A/B/C had no occurrence data at all — is closed by the new `AcquisitionMilestone` entity (§5). Not offered as a candidate PKM-entity addition: this app's own Milestone entity is already the precedent (a per-baseline gate-occurrence shape), so if a second app independently needs the same AAF-milestone-occurrence concept, extending PKM's own `Milestone` entity description ("SETR gate... etc.") to explicitly cover acquisition-decision gates generically — rather than adding a whole second parallel entity to the canonical model — may be the better fit; flagged here for design chat's judgment, not decided by this app.
7. **Two `RiskItem` interpretive calls (§10 above) not literally specified in PKM v0.6.0 §2–§3** — offered for upstream confirmation, not blocking: (a) `RiskItemStatus`'s four values (`Identified`/`Approved`/`Mitigating`/`Closed`) were derived from the entity's own four lifecycle date fields, not quoted from the canonical spec; (b) treating an `Issue`'s null `likelihood` as 1 for `riskLevel` derivation purposes is this app's own reading of "an issue has already occurred," not a literal quote. If a second app implements `RiskItem` independently, worth checking both land the same way.
8. **New, from §11 above:** cross-checks against "a real backend export" should specify whether the export came from this app's live GitHub Pages static demo (browser-cached, can go stale between deploys and between a visitor's own sessions) or a fresh instance seeded directly from this repo's current `mock-data/seed.json` (always current). This app's static-deploy mode has a real, previously-undocumented limitation worth naming: its `normalize()`/`backfillNewCollectionsFromSeed()` logic (`client/src/api/localStore.ts`) only backfills entirely-missing top-level collections for a cached browser session, never new fields added to an already-existing collection's existing rows — so a browser that cached the demo before a schema change will silently miss that change indefinitely, with no automatic recovery. Not asking for a redesign; flagging so a future cross-check knows which source it's really reading.
9. ~~**`Comment.entityType`/`entityId` were implemented as plain `string | null` rather than reusing `GapEntityType`'s closed-union pattern**...~~ **Resolved upstream, PKM Entity Model v0.7.3 §4.** Now the documented general convention: a closed union when a field must validate against a known, enumerable set (`Gap.foundInEntityType`), a plain `string` when forward-compatibility with future entity types matters (`Comment.entityType`) — this app's implementation cited by name as the surfacing case.
10. ~~**§8.1 as written specifies a runtime `fetch()` for the footer; this app instead uses a build-time JSON import**...~~ **Resolved upstream, Architecture Guidance v1.7.1 §8.1.** Build-time import of `PKM_VERSIONS.json` is now explicitly blessed as equally conformant to runtime `fetch()` for bundled SPAs, with this app's dual server-backed/static deploy modes cited as the motivating case and a worked example added.

---

*This report is implementation status, not a modification to the PKM Entity Model or Migration Plan documents themselves. If any of the optional follow-ups above turn into an actual PKM modeling question, that should route back through the normal conformance-feedback path.*
