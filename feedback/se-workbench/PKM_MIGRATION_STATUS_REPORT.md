# SE Workbench — PKM Migration Status Report

**Version:** 1.5.0
**From:** SE Workbench implementation session ("PDR Reconciliation & Baseline Alignment Workbench")
**Reports against:** PKM Migration Plan v0.3.1 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/migration-plans/se-workbench/PKM_MIGRATION_PLAN.md)), PKM Entity & Relationship Model v0.4.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md))
**Changelog:**
- v1.5.0 (2026-07-29) — added §7, recording the standalone `AcquisitionMilestone` entity's actual removal (design chat ACTION item 1: the coexist-then-deprecate window from Step 9/§6 is now closed, not just verified-closeable). Renumbered former §7 (optional follow-ups) to §8. Updated `Reports against:` to PKM Migration Plan v0.3.1 / Entity Model v0.4.0 (both revved since v1.4.0 — the `ReconciliationEvent` addition; documentation-only from this app's side per the Migration Plan's own v0.3.1 note, no action taken here).
- v1.4.0 (2026-07-29) — added §6, documenting Migration Plan Step 8 (this app's own numbering: Step 9): consolidated the standalone `AcquisitionMilestone` entity into `Milestone` via `milestoneType`, per the canonical model's v0.3.0 broadening. Answers Migration Plan §9 item 5 (coexistence-window deadline). Renumbered former §6 (optional follow-ups) to §7. Also corrected this header's stale `Reports against:` versions (was citing Migration Plan v0.2.0 / Entity Model v0.2.1, two and three revisions behind), per the Migration Plan's own §9 item 5 context and this repo's feedback-staleness convention.
- v1.3.0 (2026-07-28) — added §5, documenting PKM Migration Step 8 (AAF Milestone A/B/C occurrence tracking) — a new `AcquisitionMilestone` entity — plus a direct answer to §5 optional-follow-up #6 from v1.2.0 (whether acquisition-phase/pathway concepts belong in the PKM model). Renumbered former §5 (optional follow-ups) to §6 and updated item #6's own text to reflect the answer.
- v1.2.0 (2026-07-27) — added §4, documenting the Acquisition Phase Workbench (new default guided navigation). No PKM entity/schema change beyond one additive evidence-type enum value. Renumbered former §3 (optional follow-ups) to §5 and added one new follow-up item.
- v1.1.0 (2026-07-27) — added §3, documenting a PDKM Promises UI redesign (grouped/collapsible/searchable). No entity/schema change.
- v1.0.0 (2026-07-26) — initial report: all 7 migration steps implemented, verified, deployed.

**Status:** All 7 original migration steps, this app's own Step 8 (AcquisitionMilestone) and Step 9 (its consolidation into Milestone), are implemented, verified, and deployed — and Step 8's entity has since been fully retired (§7). This is a status/handoff report, not a request for new feedback on the plan itself — flagged items at the end are optional follow-ups, not blockers.

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

## 8. Optional follow-ups (not blockers)

These surfaced during implementation and are offered as candidate topics for continued discussion, not requests:

1. **PKM §5 open question #2** (ChecklistItem domain tagging) is still open. This app's implementation used a plain string attribute as a pragmatic default — real usage may clarify whether "Domain" deserves first-class status.
2. **Role taxonomy for ActionItem/Recommendation.owner** (Step 7's own deferred decision) was resolved locally with a five-role starting set (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager). Worth confirming whether this is a reasonable default for other apps building against the same PKM, or program-specific enough that each app should define its own.
3. **Gap's evidence cardinality**: this implementation used a single `foundInEntityType`/`foundInEntityId` reference per Gap (one location per finding). Real data already shows a finding can be "found in" one place while being *referenced by* multiple other mechanisms (the CI flag + Delta Matrix row case in Step 6) — worth a future pass on whether Gap needs to support multiple `foundIn` references, not just multiple things pointing *at* it.
4. **Requirement/VerificationEvent/ChecklistItem decomposition of `Specification.sections`** remains deferred, per Step 4's and Step 5's own explicit scoping — flagged here only so it isn't lost as a known-remaining slice, not because it needs to happen next.
5. **Promises UI major-group taxonomy (§3)** is this app's own judgment call, not a PKM-derived structure — flagged in case a future cross-app Promises UI conversation wants to compare notes on grouping approaches.
6. **Acquisition-phase/pathway modeling (§4) — answered in §5 above, resolved.** `AcquisitionPhase` and `AcquisitionPathway` both stay app-side methodology/derived content, not PKM entities (already-conformant-as-a-union and derived-not-stored respectively). The actual gap this question surfaced — AAF Milestone A/B/C had no occurrence data at all — is closed by the new `AcquisitionMilestone` entity (§5). Not offered as a candidate PKM-entity addition: this app's own Milestone entity is already the precedent (a per-baseline gate-occurrence shape), so if a second app independently needs the same AAF-milestone-occurrence concept, extending PKM's own `Milestone` entity description ("SETR gate... etc.") to explicitly cover acquisition-decision gates generically — rather than adding a whole second parallel entity to the canonical model — may be the better fit; flagged here for design chat's judgment, not decided by this app.

---

*This report is implementation status, not a modification to the PKM Entity Model or Migration Plan documents themselves. If any of the optional follow-ups above turn into an actual PKM modeling question, that should route back through the normal conformance-feedback path.*
