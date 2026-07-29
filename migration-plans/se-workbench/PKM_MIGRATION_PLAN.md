# SE Workbench — PKM Migration Plan

**Version:** 0.5.0 (Draft — for Workbench repo feedback)
**Target repo:** the SE Workbench app (formerly "PDR Reconciliation & Baseline Alignment Workbench")
**Based on:** PKM Entity & Relationship Model v0.6.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md)), Architecture Guidance v1.5.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md))

**Changelog:**
- **v0.5.0** — Added Step 10: implement `RiskItem` as a first-class entity (PKM v0.6.0 §2–§3), approved to proceed now per Ron's Q2/Q3 answers on the SEMP-generation proposal (2026-07-29). Renumbered "Open questions" from §10 to §11 to avoid collision with the new step, same renumbering discipline used when Step 9 was added.
- **v0.4.0** — Added Step 9: implement `Role` as a first-class entity (PKM v0.5.0 §2–§3), per Ron's go-ahead to proceed now, decoupled from the broader SEMP-generation proposal decision. Scoped directly from Workbench's own estimate in their Migration Plan §10 item 3 response (new entity + CRUD route + client wiring across both type-mirror files + admin UI + `owner`→`assignedRoleId` migration).
- **v0.3.1** — Resolved §9 item 5 (no hard deadline needed for the `AcquisitionMilestone` coexistence window, per Workbench's Step 9 report). Added a note under Step 2 (§2 below): PKM v0.4.0 replaced Baseline's reserved `reconciledFromBaselineId`/`reconciledIntoBaselineId` fields with a new `ReconciliationEvent` entity — since Workbench never populated those reserved fields, this is a documentation-only note for Workbench's own future Step 2 record, not a migration action; no code change implied unless/until Workbench chooses to model reconciliation at all.
- **v0.3.0** — Steps 1–7 recorded as **complete**, per Workbench's own status report v1.3.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/feedback/se-workbench/PKM_MIGRATION_STATUS_REPORT.md)) — §0 updated, §8 sequencing table annotated. Open questions §9 items 1, 2, and 4 marked **resolved**, with the implementation evidence that answered each, rather than left presumed here. Item 3 marked **resolved locally, open for cross-app default** — matches Workbench's own framing of it as a candidate default, not a final answer. Added **Step 8 — Consolidate `AcquisitionMilestone` into `Milestone`**: canonical guidance for folding Workbench's already-implemented standalone `AcquisitionMilestone` entity (their own Step 8, status report §5) into the PKM's now-broadened `Milestone` entity via the new `milestoneType` discriminator (PKM v0.3.0/v0.3.1, §2–§3). This is new guidance *from* this plan to Workbench, not a report *of* something Workbench already did — the direction is reversed from every other step so far.
- **v0.2.1** — Corrected stale cross-reference: `Based on:` cited PKM Entity Model v0.2.0 and Architecture Guidance v1.3.0, both since revved (now v0.2.1 and v1.4.0) without this plan's header keeping pace. No step content changed. Added raw URLs per Workflow Protocol §3.4.
- **v0.2.0** — Renamed this plan's numbered units from "Phase" to "Step" to resolve terminology collision with Architecture Guidance §7's own migration-sequencing phases. Expanded Step 2's blast radius to include `sempExport.ts` and `recoveryProgramGuidance.ts`, and flagged its dependency on Architecture Guidance's pending content-split step. Carved `AbCompatibilityRow` out of Step 2's single-`baselineId` treatment. Added explicit methodology/data split note to Step 3. All per SE Workbench's round-2 feedback.
- **v0.1.0** — Initial draft.

**Purpose:** A concrete, phased plan for migrating this app's schema toward PKM conformance, sequenced by regression risk per Architecture Guidance §7 (mechanical/low-risk first, judgment-heavy content work last). Steps 1–7 are historical record at this point — Workbench has already implemented, verified, and deployed all of them. **Step 8 is the only step in this document Workbench has not yet implemented** — it exists because PKM v0.3.0/v0.3.1 changed shape *after* Workbench's own independent Step 8 (`AcquisitionMilestone`) shipped, and that entity needs to be reconciled with the now-canonical shape.

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

- `satisfiedByCiIds` modeled many-to-many. Real seed data (`delta-001`) confirmed this is a genuine case, not a hypothetical — **resolves open question §11 item 1 below.**
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

- Real unification demonstrated: a finding previously tracked by two separate mechanisms (a CI's over-decomposition flag and a Delta Matrix row) now references one `Gap` record — **resolves open question §11 item 2 below.**
- `Recommendation` deliberately left untouched here, picked up in Step 7.
- Workbench's own follow-up (status report §6 item 3): current implementation uses a single `foundInEntityType`/`foundInEntityId` per Gap, but real data shows a finding can be found in one place while referenced by multiple other mechanisms — flagged as a candidate future PKM cardinality question, not resolved by this plan.

---

## 7. Step 7 — ActionItem role-assignment enforcement — ✅ Complete

*(Historical record — implemented and verified per status report v1.0.0 §1.)*

- `Recommendation.owner` converted from free text to a five-role taxonomy: Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager.
- `resolvesGapId` added, coexisting with the existing `relatedCiId`.
- **Partially resolves open question §11 item 3 below** — resolved locally; Workbench's own status report frames this as "a pragmatic starting cut, not a definitive one" and asks whether it's a reasonable cross-app default. That question is not answered by this plan.

---

## 8. Step 8 — Consolidate `AcquisitionMilestone` into `Milestone` (new guidance)

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

## 9. Step 9 — Implement `Role` as a first-class entity (new, approved to proceed)

**Approved to proceed now**, decoupled from the broader SEMP-generation proposal decision (Ron, 2026-07-29) — this doesn't wait on that proposal's other open items.

This is guidance *to* Workbench again, same direction as Step 8, but this time scoped directly from Workbench's own estimate (Migration Plan §11 item 3 response) rather than design chat guessing at implementation cost:

- **New `Role` entity:** `id`, `projectId`, `name`, `authorityDescription` (optional), `isDefault` (boolean). Seed with the existing five-role set (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager) as `isDefault: true` records — this preserves the taxonomy Workbench already validated, just makes it data instead of a hardcoded union.
- **New CRUD route** for `Role` (create/list/update/delete), mirroring the pattern already used for every other entity.
- **`ActionItem`/`Recommendation.owner` migrates to `assignedRoleId`**, coexisting with `owner` during the transition — same coexist-then-deprecate discipline as every step so far. `owner` is not removed in this step.
- **Admin surface:** the existing fixed dropdown (`RecommendationsPage.tsx`) needs to read live `Role` records instead of the hardcoded array, plus a minimal add/remove-role UI — Workbench's own estimate frames this as comparable in scope to a full step, not a quick patch, and this plan agrees.
- **Both type-mirror files** need the `Role` type and the `assignedRoleId` field added consistently, per Workbench's own standard practice across every prior step.

**Risk:** low-moderate, mechanical in shape (new entity + CRUD + coexisting reference field, the same recipe as Steps 1–3) but touches more surface area than those did (a new admin UI, not just data modeling) — closer in scope to Step 7's original role-taxonomy work than to Step 1's pure additive record creation.

**Not part of this step:** any `RiskItem`-side use of `Role` (proposed in the SEMP-generation proposal, `proposals/UDM_V2_SEMP_GENERATION_PROPOSAL.md` §3.1) — that's a separate, still-gated decision. This step is scoped to closing Migration Plan §11 item 3 alone.

---

## 10. Step 10 — Implement `RiskItem` as a first-class entity (new, approved to proceed)

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

## 11. Open questions for Workbench's team before implementation starts

1. ~~Does a `Requirement` in this app's real data ever get satisfied by more than one CI, or is that always one-to-one?~~ **Resolved.** Yes — confirmed many-to-many by real seed data (`delta-001`), per status report §1 Step 4. No further input needed.
2. ~~For Gap unification (Step 6): are there UI features, filters, or reports built specifically against `DeltaMatrixRow`, `overDecompositionFlag`, or `Recommendation` as distinct types that would need parallel updates, or can they be safely generalized?~~ **Resolved.** Successfully unified in practice — one existing finding tracked by two prior mechanisms now references a single `Gap` record, per status report §1 Step 6. A related but distinct question (`foundIn` cardinality — one location per finding vs. multiple) remains open per status report §6 item 3, not this question.
3. ~~What role taxonomy should `ActionItem`/`Recommendation.owner` constrain to?~~ **Resolved.** Per PKM v0.5.0 §2–§3: `Role` is now a first-class entity, tailorable per program. Step 9 (above) implements this — no longer a cross-app-default question, since the taxonomy is now data, not a fixed enum.
4. ~~Should Step 2 (Baseline enum→entity) and Architecture Guidance's pending content-split step run as one coordinated effort, or in an explicit order?~~ **Resolved.** Closed per Roles & Handoff v1.1.0 — Workbench coordinated both concerns in a single pass on `recoveryProgramGuidance.ts`.
5. ~~Does the coexistence window for `AcquisitionMilestone` need a hard deadline, or is "until the gate-status UI is confirmed reading from consolidated `Milestone`" sufficient as a completion criterion on its own?~~ **Resolved.** Per Workbench's Step 9 report: no hard deadline needed.

---

*This plan is Workbench-specific and does not modify the PKM Entity Model or Architecture Guidance documents themselves. If implementation surfaces a PKM modeling gap, that should route back as conformance feedback the same way it has each round so far.*
