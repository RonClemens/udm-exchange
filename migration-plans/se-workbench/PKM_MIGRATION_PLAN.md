# SE Workbench — PKM Migration Plan

**Version:** 0.6.2 (Draft — for Workbench repo feedback)
**Target repo:** the SE Workbench app (formerly "PDR Reconciliation & Baseline Alignment Workbench")
**Based on:** PKM Entity & Relationship Model v0.6.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md)), Architecture Guidance v1.5.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md))

**Changelog:**
- **v0.6.2** — Step 11 closed as **not reproducible against this app's real data**, verified against status report v1.9.1: a fresh export taken directly against `mock-data/seed.json` at a named commit (`0909c24...`) still shows 0/25 null on `baselineId`/`projectId` across all four entity types — same clean result as every other check run. Unlike Steps 13/14 items 1&3, the stale-static-demo theory doesn't explain this one (these fields predate the entire exchange and are populated in every commit checked). The discrepancy is real but its source is upstream of this repo — likely an artifact of how the original export was generated or sourced, not a Workbench defect. No further app-side action expected; flagged for Ron's awareness if the export process itself needs review. Also fixed a citation typo Workbench caught in their own report ("§79–82" → "§2–§3," their own copy-paste error, not design chat's).
- **v0.6.1** — Steps 12 and 14 item 2 confirmed complete, verified against status report v1.9.0 §11. Steps 13 and 14 items 1/3 confirmed **already resolved before the ACTION was sent** — the "real export" design chat cross-checked was taken against Workbench's GitHub Pages static demo, last redeployed 2026-07-28, two days stale relative to the work actually completed. Root cause identified precisely (§12 item 8 of the same report): the static/localStorage deploy mode's backfill logic only backfills entirely-missing collections for a cached browser session, never new fields on an already-existing collection's rows — so a browser that cached the demo before a schema change misses it silently, indefinitely, with no auto-recovery. **Step 11 remains genuinely unresolved** — Workbench checked every commit available and found `baselineId`/`projectId` populated in all of them, so the stale-cache explanation doesn't account for design chat's original finding there. Recommending a fresh export directly against `mock-data/seed.json` (not the live demo) to settle this definitively, per Workbench's own §12 item 8 recommendation.
- **v0.6.0** — Added Steps 11–14, all sourced from a real backend export cross-checked against the generated SEMP migration package (design chat, 2026-07-30), not from prose review alone. Step 11: backfill `baselineId`/`projectId` on CI/Specification/SafetyDeliverable/ProgramPlanningDeliverable records — the legacy `baseline` string field is still doing the real work everywhere. Step 12: migrate real, populated `Baseline.reconciledFromBaselineId`/`reconciledIntoBaselineId` data into an actual `ReconciliationEvent` record — this predates `ReconciliationEvent`'s existence and was never previously assigned as a migration step; that's a planning gap on design chat's side, not an implementation defect, called out as such in Step 12 below. Step 13: verify/backfill `Milestone.milestoneType` — every milestone record in the export is missing it, and the generated schedule table's Type column renders blank as a direct, visible symptom. Step 14: backfill `Gap.escalatedToRiskItemId`, `ActionItem.resolvesRiskItemId`, and `ActionItem.assignedRoleId` on the three legacy records whose narrative text already claims these relationships but whose structural fields don't reflect them. Fixed stale `Based on:` Architecture Guidance version (was v1.4.0, now v1.5.0).
- **v0.5.0** — Added Step 10: implement `RiskItem` as a first-class entity (PKM v0.6.0 §2–§3), approved to proceed now per Ron's Q2/Q3 answers on the SEMP-generation proposal (2026-07-29). Renumbered "Open questions" from §10 to §11 to avoid collision with the new step, same renumbering discipline used when Step 9 was added.
- **v0.4.0** — Added Step 9: implement `Role` as a first-class entity (PKM v0.5.0 §2–§3), per Ron's go-ahead to proceed now, decoupled from the broader SEMP-generation proposal decision. Scoped directly from Workbench's own estimate in their Migration Plan §10 item 3 response (new entity + CRUD route + client wiring across both type-mirror files + admin UI + `owner`→`assignedRoleId` migration).
- **v0.3.1** — Resolved §9 item 5 (no hard deadline needed for the `AcquisitionMilestone` coexistence window, per Workbench's Step 9 report). Added a note under Step 2 (§2 below): PKM v0.4.0 replaced Baseline's reserved `reconciledFromBaselineId`/`reconciledIntoBaselineId` fields with a new `ReconciliationEvent` entity — since Workbench never populated those reserved fields, this is a documentation-only note for Workbench's own future Step 2 record, not a migration action; no code change implied unless/until Workbench chooses to model reconciliation at all.
- **v0.3.0** — Steps 1–7 recorded as **complete**, per Workbench's own status report v1.3.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/feedback/se-workbench/PKM_MIGRATION_STATUS_REPORT.md)) — §0 updated, §8 sequencing table annotated. Open questions §9 items 1, 2, and 4 marked **resolved**, with the implementation evidence that answered each, rather than left presumed here. Item 3 marked **resolved locally, open for cross-app default** — matches Workbench's own framing of it as a candidate default, not a final answer. Added **Step 8 — Consolidate `AcquisitionMilestone` into `Milestone`**: canonical guidance for folding Workbench's already-implemented standalone `AcquisitionMilestone` entity (their own Step 8, status report §5) into the PKM's now-broadened `Milestone` entity via the new `milestoneType` discriminator (PKM v0.3.0/v0.3.1, §2–§3). This is new guidance *from* this plan to Workbench, not a report *of* something Workbench already did — the direction is reversed from every other step so far.
- **v0.2.1** — Corrected stale cross-reference: `Based on:` cited PKM Entity Model v0.2.0 and Architecture Guidance v1.3.0, both since revved (now v0.2.1 and v1.4.0) without this plan's header keeping pace. No step content changed. Added raw URLs per Workflow Protocol §3.4.
- **v0.2.0** — Renamed this plan's numbered units from "Phase" to "Step" to resolve terminology collision with Architecture Guidance §7's own migration-sequencing phases. Expanded Step 2's blast radius to include `sempExport.ts` and `recoveryProgramGuidance.ts`, and flagged its dependency on Architecture Guidance's pending content-split step. Carved `AbCompatibilityRow` out of Step 2's single-`baselineId` treatment. Added explicit methodology/data split note to Step 3. All per SE Workbench's round-2 feedback.
- **v0.1.0** — Initial draft.

**Purpose:** A concrete, phased plan for migrating this app's schema toward PKM conformance, sequenced by regression risk per Architecture Guidance §7 (mechanical/low-risk first, judgment-heavy content work last). **All 14 steps in this document are now resolved** — 13 complete/verified, and Step 11 closed as not reproducible against this app's real data by any method tried (its original discrepancy is real but not attributable to this app; see Step 11 below).

---

## 0. Already conformant — no action needed

- **ID/reference conventions** — already matches PKM §4, confirmed in both prior reviews.
- **CI↔LogicalSubsystem many-to-many** — `ConfigurationItem.subsystemIds: string[]` already matches PKM v0.2.0's corrected model exactly. This was a genuine app-ahead-of-doc case; no migration needed here, just note it in the app's own changelog as "already PKM v0.2.0 conformant."
- **Milestone scoped to Baseline, not Project** — Workbench's Step 3 implementation (one `Milestone` record per SETR event per baseline lineage) was, in retrospect, also an app-ahead-of-doc case: PKM's entity-table text said "belongs to Project" through v0.2.2, while the relationship diagram had shown Baseline-scoping since the initial draft. PKM v0.3.0 corrected the text to match. No Workbench-side change needed here — this was a documentation bug, not an implementation gap.

---

## 1. Step 1 — Program & Project entities (additive, mechanical) — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- Added `Program` and `Project` records: one of each, matching this app's single-program scope.
- Added `projectId` as an optional field across Baseline-tagged entity types.
- **Verified:** additive-only, zero regression surface, per status report §1.

---

## 2. Step 2 — Promote Baseline from enum to entity — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- `Baseline` entity created with `baselineType`, `projectId`, `reconciledFromBaselineId`/`reconciledIntoBaselineId` reserved per PKM §5 open question #1 (at the time; **see note below — superseded in PKM v0.4.0**).
- Coordinated with `recoveryProgramGuidance.ts`'s own methodology/data content-split, resolving the blocking dependency flagged in v0.2.0 of this plan — confirmed closed per Roles & Handoff v1.1.0's "Resolved since v1.0.0" section.
- `AbCompatibilityRow` left unmigrated as planned — its evidence directly informed PKM v0.4.0's `ReconciliationEvent` entity (see PKM Entity Model changelog and §3).
- **Note added in this plan's v0.3.1:** the reserved `reconciledFromBaselineId`/`reconciledIntoBaselineId` fields above were never populated in Workbench's implementation, per Workbench's own confirmation. PKM v0.4.0 replaced them with a `ReconciliationEvent` entity — historical record only, no code action implied for Workbench unless/until reconciliation modeling is actually taken up as a future step.

---

## 3. Step 3 — Milestone entity — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- `Milestone` records created from the `SETR_EVENTS` catalog, one per SETR event per baseline lineage.
- Methodology/data split held: generic event definitions stayed in the methodology layer; only per-baseline status/dates live on the entity.
- **Note for Step 8 below:** this existing per-baseline-lineage shape is exactly what PKM v0.3.0's `milestoneType` discriminator generalizes — Step 8 extends this same entity rather than introducing a new one.

---

## 4. Step 4 — Requirement entity — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- `satisfiedByCiIds` modeled many-to-many. Real seed data (`delta-001`) confirmed this is a genuine case, not a hypothetical — **resolves open question §15 item 1 below.**
- `parentRequirementId` demonstrated via a second requirement implicitly part of the first.

---

## 5. Step 5 — ChecklistItem and VerificationEvent entities — ✅ Complete (first slice)

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- First slice only, per this plan's own risk note — content authoring, not mechanical conversion. Covers two in-progress milestones and one COTS verification-method conversion.
- `domain` left as a plain string attribute, per PKM §5 open question #2 (still open).
- Remaining `Specification.sections` decomposition explicitly deferred (status report §6 item 4) — not a blocker, just not yet done.

---

## 6. Step 6 — Unify Gap representation — ✅ Complete (first slice)

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- Real unification demonstrated: a finding previously tracked by two separate mechanisms (a CI's over-decomposition flag and a Delta Matrix row) now references one `Gap` record — **resolves open question §15 item 2 below.**
- `Recommendation` deliberately left untouched here, picked up in Step 7.
- Workbench's own follow-up (status report §6 item 3): current implementation uses a single `foundInEntityType`/`foundInEntityId` per Gap, but real data shows a finding can be found in one place while referenced by multiple other mechanisms — flagged as a candidate future PKM cardinality question, not resolved by this plan.

---

## 7. Step 7 — ActionItem role-assignment enforcement — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- `Recommendation.owner` converted from free text to a five-role taxonomy: Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager.
- `resolvesGapId` added, coexisting with the existing `relatedCiId`.
- **Partially resolves open question §15 item 3 below** — resolved locally; Workbench's own status report frames this as "a pragmatic starting cut, not a definitive one" and asks whether it's a reasonable cross-app default. That question is not answered by this plan.

---

## 8. Step 8 — Consolidate `AcquisitionMilestone` into `Milestone` (new guidance) — ✅ Complete

**Why this step exists, and why it's different from every step above:** Workbench independently identified and closed a real gap — AAF Milestone A/B/C decision-gate occurrence had no data representation at all — by adding a new `AcquisitionMilestone` entity (their own Step 8, status report §5). That work was good and directly informed PKM v0.3.0's own Milestone broadening. But the canonical model resolved the underlying question differently than a fully separate entity: **one `Milestone` entity with a `milestoneType` discriminator (`SETR` | `AcquisitionGate`)**, not two parallel entities. This step asks Workbench to fold its already-shipped `AcquisitionMilestone` entity into the existing `Milestone` entity, rather than leaving both to drift as separate structures.

**This is coexist-then-deprecate, same pattern as every prior step**, just applied to two of the app's *own* entities rather than a legacy enum/free-text field:

- Add `milestoneType: "SETR" | "AcquisitionGate"` to the existing `Milestone` entity. Backfill all current `Milestone` records with `milestoneType: "SETR"` — this is the only value that has ever existed for that entity, so backfill is unambiguous and mechanical.
- Add `pathway: string | null` to `Milestone`, populated only for `AcquisitionGate`-type records (e.g. `"MCA"`); `null`/absent for `SETR` records.
- Migrate each existing `AcquisitionMilestone` record (`event`, `pathway`, `baselineId`, `status`, `actualDate`, `plannedDate`) into a new `Milestone` record with `milestoneType: "AcquisitionGate"`. Field mapping is 1:1 — no data reshaping needed, since `AcquisitionMilestone`'s shape was already what motivated the canonical model's shape.
- **`establishesBaselineId` semantics differ by type, per PKM v0.3.1 §3:** `SETR` records may populate it (as today); `AcquisitionGate` records leave it null — MS-A/B/C gate progress within an already-established baseline lineage, they don't establish one.
- **Recommended coexistence window:** keep the standalone `AcquisitionMilestone` type/table in place, deprecated but not removed, until the Acquisition Phase Workbench's gate-status UI (status report §4) is confirmed reading from the consolidated `Milestone` records instead. This UI is the one consumer of `AcquisitionMilestone` today — it's the accurate measure of "migration done," not just data presence.
- **`deriveCurrentMilestone()` unaffected:** per status report §5's own reasoning, SRR–PRR ordering is load-bearing for that function and stays scoped to `milestoneType: "SETR"` records only — this consolidation does not ask that function to reason about AAF gates.
- **PDKM Promises tab:** confirm the consolidated `Milestone` entity's `@domain-placeholder` tagging (if any) still applies correctly post-merge — the manifest at `/data-schema/DOMAIN_PLACEHOLDER_FIELDS.md` should reflect one entity, not two, once this is done.

**Risk:** low-moderate. Regression surface is narrower than most prior steps — one existing UI consumer (§4's gate-status display), one existing entity to retire, no new judgment calls about how to interpret ambiguous prose (unlike Steps 4–6). The main risk is timing: don't remove the standalone `AcquisitionMilestone` type until the UI cutover above is verified, same coexist-then-deprecate discipline as every step before it.

---

## 9. Step 9 — Implement `Role` as a first-class entity — ✅ Complete

**Approved to proceed now**, decoupled from the broader SEMP-generation proposal decision (Ron, 2026-07-29) — this doesn't wait on that proposal's other open items.

This is guidance *to* Workbench again, same direction as Step 8, but this time scoped directly from Workbench's own estimate (Migration Plan §15 item 3 response) rather than design chat guessing at implementation cost:

- **New `Role` entity:** `id`, `projectId`, `name`, `authorityDescription` (optional), `isDefault` (boolean). Seed with the existing five-role set (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager) as `isDefault: true` records — this preserves the taxonomy Workbench already validated, just makes it data instead of a hardcoded union.
- **New CRUD route** for `Role` (create/list/update/delete), mirroring the pattern already used for every other entity.
- **`ActionItem`/`Recommendation.owner` migrates to `assignedRoleId`**, coexisting with `owner` during the transition — same coexist-then-deprecate discipline as every step so far. `owner` is not removed in this step.
- **Admin surface:** the existing fixed dropdown (`RecommendationsPage.tsx`) needs to read live `Role` records instead of the hardcoded array, plus a minimal add/remove-role UI — Workbench's own estimate frames this as comparable in scope to a full step, not a quick patch, and this plan agrees.
- **Both type-mirror files** need the `Role` type and the `assignedRoleId` field added consistently, per Workbench's own standard practice across every prior step.

**Risk:** low-moderate, mechanical in shape (new entity + CRUD + coexisting reference field, the same recipe as Steps 1–3) but touches more surface area than those did (a new admin UI, not just data modeling) — closer in scope to Step 7's original role-taxonomy work than to Step 1's pure additive record creation.

**Not part of this step:** any `RiskItem`-side use of `Role` (proposed in the SEMP-generation proposal, `proposals/UDM_V2_SEMP_GENERATION_PROPOSAL.md` §3.1) — that's a separate, still-gated decision. This step is scoped to closing Migration Plan §15 item 3 alone.

---

## 10. Step 10 — Implement `RiskItem` as a first-class entity — ✅ Complete

**Approved to proceed now** (Ron, 2026-07-29) — same decoupling as Step 9, independent of the SEMP proposal's remaining open items (24748-4 access, Risk Management Board, SEMP audit trail).

- **New `RiskItem` entity:** `id`, `projectId`, `itemType` (`Risk` \| `Issue` \| `Opportunity`), `category`, `likelihood` (1–5, omit/null for `itemType: "Issue"`), `consequenceCost`/`consequenceSchedule`/`consequencePerformance` (1–5 each), `mitigationStrategy` (`Accept` \| `Avoid` \| `Transfer` \| `Control`), `ownerRoleId` (references the `Role` entity from Step 9 — implement Step 9 first if not already done), optional `linkedMilestoneId`/`linkedCiId`, `identifiedDate`/`approvalDate`/`plannedClosureDate`/`actualClosureDate`, `status`. `riskLevel` is derived client-side (likelihood × max consequence via a standard risk-matrix mapping), not stored.
- **New CRUD route** for `RiskItem`, same pattern as every other entity.
- **`Gap.escalatedToRiskItemId`** (new, optional, nullable) — additive field, no migration of existing `Gap` records required; populate only when a real escalation occurs.
- **`ActionItem.resolvesRiskItemId`** (new, optional, coexisting with the existing `resolvesGapId`) — an ActionItem resolves a Gap *or* a RiskItem, not both.
- **Both type-mirror files** need `RiskItem`'s type and the two new reference fields added consistently, per standard practice across every prior step.
- **PRMP note, not an action item:** if Workbench ever adds a Program Risk Management Plan artifact, it needs no new entity — `Deliverable.type = "PRMP"` is already sufficient. Flagging so this isn't mistaken for a gap when it's actually already conformant.

**Risk:** moderate — a genuinely new entity with more fields than `Role` had, though the shape (additive fields, coexisting references, standard CRUD) follows the same mechanical recipe as every step so far. No admin-UI requirement the way Step 9 had (RiskItem records are expected to be created/edited directly, not chosen from a managed catalog), which somewhat offsets the larger field count.

**Not part of this step:** any Risk Management Board representation (deliberately excluded from PKM v0.6.0, parked pending Ron's separate risk-planning session) — don't add board/governance structure speculatively ahead of that.

---

## 11. Step 11 — Backfill `baselineId`/`projectId` on CI/Specification/SafetyDeliverable/ProgramPlanningDeliverable — 🔒 Closed, not reproducible

**Status update (2026-07-30, final):** confirmed via two independent checks (real current data, and a fresh export run directly against `mock-data/seed.json` at commit `0909c24d8ca2beb4e9467402f328748fc188ee90`, per design chat's follow-up request) — **0 null/missing across all 25 records** (CI ×5, Specification ×5, SafetyDeliverable ×9, ProgramPlanningDeliverable ×6), both times. This was Migration Step 1/2 work, complete long before this exchange began. **Closing this step as not reproducible against this app's real data by any method tried.** Unlike Steps 13/14 items 1&3 (below), this doesn't fit the stale-static-demo explanation — these fields predate the whole exchange and are populated in every commit checked. The original export's discrepancy is real but its cause is upstream of this repo, not a Workbench defect — most likely an artifact of how that specific export was generated. No further app-side action expected.

**Found via a real backend export cross-check (design chat, 2026-07-30), not a code review.** Every `CI`, `Specification`, `SafetyDeliverable`, and `ProgramPlanningDeliverable` record in the export has `baselineId: null` and `projectId: null` — the generated SEMP document's baseline grouping is entirely dependent on a legacy `baseline: "Baseline A"` string field, not the real foreign-key relationship these entities are supposed to have per PKM (and per this plan's own Step 2, which promoted `Baseline` to an entity specifically so other entities could reference it properly).

- **Backfill, don't rename:** populate `baselineId` (and `projectId` where applicable) on every existing record by resolving the legacy `baseline` string against the real `Baseline` entity (`"Baseline A"` → `BASELINE-A`'s id, etc.). This is a one-time data migration, not a schema change — the fields already exist and are already null, just unpopulated.
- **Coexist, then deprecate the string field** — same discipline as every step so far. Don't remove `baseline` until every consumer (UI, exports, this SEMP generator) reads `baselineId` instead and that's been verified.
- **This is likely the root cause of the interface-misgrouping issue** design chat found in the generated SEMP package (a Baseline A ↔ Baseline A interface record filed under the "Baseline B" section) — the generator has nothing reliable to group by, so worth re-checking that specific case once this backfill lands.

**Risk:** low-moderate. Pure backfill, no new entity, but touches four record types across the whole dataset — verify row counts before/after to confirm nothing silently failed to resolve.

---

## 12. Step 12 — Migrate real `Baseline` reconciliation data into a `ReconciliationEvent` record — ✅ Complete

**This one needs a correction on design chat's part, stated plainly:** PKM v0.4.0's changelog claimed `Baseline.reconciledFromBaselineId`/`reconciledIntoBaselineId` were "reserved but never populated in any known implementation," and removed them from the model outright on that basis — treating it as a zero-migration-cost change. **That assumption was wrong.** The real export shows both fields populated (`BASELINE-A.reconciledIntoBaselineId = BASELINE-B`, `BASELINE-B.reconciledFromBaselineId = BASELINE-A`), and no migration step was ever assigned to move that real data onto the new entity before this was discovered. This is a planning gap on design chat's side — Workbench's implementation did nothing wrong; it was simply never asked to do this step, because design chat incorrectly believed there was nothing to migrate.

- **Create one `ReconciliationEvent` record** from the existing data: `fromBaselineId: "BASELINE-B"`, `intoBaselineId: "BASELINE-A"`, `status`: your best read of current reality (likely `"In Progress"`, given Baseline B is still early — SFR in progress — and this is an active reconciliation effort, not a completed one). `initiatedDate`/`completedDate`: use whatever real dates you have, or leave `completedDate` null if it's ongoing.
- **`evidenceEntityType`/`evidenceEntityIds`**: populate from `abCompatibility` — both `ab-001` and `ab-002` are exactly the kind of per-interface assessment PKM v0.4.0's `ReconciliationEvent` design anticipated as evidence. `evidenceEntityType: "AbCompatibilityRow"`, `evidenceEntityIds: ["ab-001", "ab-002"]`.
- **Remove `Baseline.reconciledFromBaselineId`/`reconciledIntoBaselineId`** once the `ReconciliationEvent` record is created and verified — this is the actual deprecation this time, not a documentation-only note like v0.3.1 mistakenly recorded.

**Risk:** low. One record to create from data that already exists in a clear, unambiguous shape; one field removal after verification.

---

## 13. Step 13 — Verify/backfill `Milestone.milestoneType` — ✅ Already resolved before this ACTION was sent

**Status update (2026-07-30):** already backfilled in current data (0/22 missing), completed earlier in this same exchange via Steps 8/9. The original export finding (16/16 missing) matched exactly a commit that was live on Workbench's GitHub Pages static demo before a 2026-07-28→2026-07-30 redeploy gap — the export was almost certainly taken against that stale cached build, not this repo's current state. **Root cause identified:** the static/localStorage deploy mode's backfill logic only backfills entirely-missing top-level collections for a cached browser session, never new fields added to an already-existing collection's rows — a browser that cached the demo before a schema change misses it silently, indefinitely, with no auto-recovery. This is a real, previously-undocumented limitation worth Architecture Guidance's attention for any app using the §3.1 static-deploy exception, not just this one.

Every milestone record in the export is missing `milestoneType` entirely — not defaulted to `"SETR"`, just absent. This is directly visible in the generated SEMP's schedule table: the "Type" column is blank on all 16 rows, both baselines.

- If Step 8's original backfill (v0.3.0, "backfill all current `Milestone` records with `milestoneType: 'SETR'`") was never actually run against this environment's data, run it now — this should be a fast, low-risk operation given the field already exists in the schema.
- If the field *was* backfilled and this export simply predates that, no action needed beyond confirming a fresh export shows it populated — flag which case this turns out to be, since it affects how much to trust other "already complete" steps' backfills against real data going forward.

**Risk:** low. Straightforward backfill or a confirmation that one already happened.

---

## 14. Step 14 — Backfill `Gap.escalatedToRiskItemId` and `ActionItem` `resolvesRiskItemId`/`assignedRoleId` on legacy records — ✅ Complete (item 2 fresh; items 1/3 already resolved)

**Status update (2026-07-30):** item 2 (`rec-003.resolvesRiskItemId`) implemented fresh, backfilled to `"risk-002"`. Items 1 and 3 (`gap-003.escalatedToRiskItemId`, `assignedRoleId` on the three original recommendations) were already populated in current data as part of Workbench's own Steps 11/12 work — the original export's stale-static-demo build predates `Role`/`RiskItem` entirely, so it couldn't have shown them populated regardless. Same root cause as Step 13, above.

Three specific records where the narrative text already claims a relationship the structural fields don't reflect:

- **`gap-003.escalatedToRiskItemId`** should be `"risk-001"` — `risk-001`'s own description explicitly states "escalated from gap-003."
- **`rec-003` (Recommendation)** — its text describes the mitigation `risk-002` (the interface-compatibility Issue) needs; if this Recommendation *is* that mitigation's ActionItem, set `resolvesRiskItemId: "risk-002"`.
- **All three `recommendations` records** have `owner: ""` and no `assignedRoleId` — these predate the Step 9 Role migration and were never backfilled. Assign real roles where the text implies one (e.g. `rec-001`/`rec-002`, CI-structure and ECP recommendations, plausibly `role-cm`; `rec-003`, the software-side adapter-layer work, plausibly `role-sw`) — your call on the actual mapping, this is a judgment call design chat shouldn't make for you.

**Risk:** low. Small, targeted backfill on three known records — no schema change, no new entity.

---

## 15. Open questions for Workbench's team before implementation starts

1. ~~Does a `Requirement` in this app's real data ever get satisfied by more than one CI, or is that always one-to-one?~~ **Resolved.** Yes — confirmed many-to-many by real seed data (`delta-001`), per status report §1 Step 4. No further input needed.
2. ~~For Gap unification (Step 6): are there UI features, filters, or reports built specifically against `DeltaMatrixRow`, `overDecompositionFlag`, or `Recommendation` as distinct types that would need parallel updates, or can they be safely generalized?~~ **Resolved.** Successfully unified in practice — one existing finding tracked by two prior mechanisms now references a single `Gap` record, per status report §1 Step 6. A related but distinct question (`foundIn` cardinality — one location per finding vs. multiple) remains open per status report §6 item 3, not this question.
3. ~~What role taxonomy should `ActionItem`/`Recommendation.owner` constrain to?~~ **Resolved.** Per PKM v0.5.0 §2–§3: `Role` is now a first-class entity, tailorable per program. Step 9 (above) implements this — no longer a cross-app-default question, since the taxonomy is now data, not a fixed enum.
4. ~~Should Step 2 (Baseline enum→entity) and Architecture Guidance's pending content-split step run as one coordinated effort, or in an explicit order?~~ **Resolved.** Closed per Roles & Handoff v1.1.0 — Workbench coordinated both concerns in a single pass on `recoveryProgramGuidance.ts`.
5. ~~Does the coexistence window for `AcquisitionMilestone` need a hard deadline, or is "until the gate-status UI is confirmed reading from consolidated `Milestone`" sufficient as a completion criterion on its own?~~ **Resolved.** Per Workbench's Step 9 report: no hard deadline needed.

---

*This plan is Workbench-specific and does not modify the PKM Entity Model or Architecture Guidance documents themselves. If implementation surfaces a PKM modeling gap, that should route back as conformance feedback the same way it has each round so far.*
