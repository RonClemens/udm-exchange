# UDM v2.0 — Automated SEMP Generation Architecture (Planning Proposal)

**Version:** 0.6.2 (Exploratory / Proposal — Role, RiskItem, Phase C, and Phase D's RiskItem slice all real, implemented, and data-verified; all four export-cross-check findings now closed; grounding-source question still open)
**Last updated:** 2026-07-30
**Status:** Draft
**Depends on:** PKM Entity Model v0.6.0, Architecture Guidance v1.5.0, SE Workbench Migration Plan v0.6.2

**Changelog:**
- **v0.6.2** — Migration Plan Step 11 closed as not reproducible against Workbench's real data (two independent checks, including a fresh export against a named commit of `mock-data/seed.json`) — the original discrepancy is real but its source is upstream of this repo, not a Workbench defect. All four export-cross-check findings from earlier today are now resolved (three fixed, one closed-unexplained). Fixed a citation typo Workbench's own report caught and corrected.
- **v0.6.1** — Workbench closed out three of the four v0.6.0 findings same-day: `ReconciliationEvent` migrated (Migration Plan Step 12), `Gap`/`ActionItem` backfills complete (Step 14), and Phase D's `RiskItem` slice implemented and verified (`buildRiskRegister()`, wired into §3.2.1). The fourth (Step 11, `baselineId`/`projectId`) turned out to already be populated in Workbench's real data all along — the original export was taken against a stale GitHub Pages static-demo build, not current state; root cause identified (static-deploy caching limitation, worth an Architecture Guidance note). Step 11 stays open for a fresh-export re-check, not because it's a confirmed real gap.
- **v0.6.0** — Design chat cross-checked a real Workbench backend export against a generated SEMP migration package (2026-07-30) and found four data-integrity gaps, none previously visible from prose review alone: unpopulated `baselineId`/`projectId` on four entity types (Migration Plan Step 11), real `Baseline` reconciliation data never migrated onto `ReconciliationEvent` (Step 12 — a planning gap on design chat's part, corrected there), missing `Milestone.milestoneType` causing a visible blank column in the generated schedule (Step 13), and narrative-only Gap/ActionItem relationships not reflected structurally (Step 14). Also found: the generated package cites DI-SESS-81785B and OSD SEP Outline v4.1 directly — real, specific DoD documents — which may supersede the need for ISO/IEC/IEEE 24748-4 as this proposal's blocking grounding source; not yet decided. §4 Phase D partially approved, scoped to wiring `RiskItem` into §3.2.1 only.
- **v0.5.2** — Caught by udm-exchange before committing v0.5.1 (2026-07-30 0020 UTC): Workbench's status report v1.8.0 / SEMP-generation feedback v1.1.0 confirm `RiskItem` is now implemented (their own Step 12 — CRUD route, `RiskItemsPage.tsx`, seed data covering all three `itemType` values, both new `Gap`/`ActionItem` reference fields) and that the Step 10 HANDOFF *did* land on 2026-07-29, just after v0.5.1 was drafted — meaning v0.5.1's "not yet relayed" language was stale before it could even be committed. §5, §6 items 6-7 corrected to reflect RiskItem as implemented and verified, same status as Role and Phase C.
- **v0.5.1** — Caught on self-review (not flagged externally this time): §6 items 6 and 7 still had the same class of stale claim just fixed in §5 — item 6 said RiskItem/Step 10 was "relayed to Workbench as real guidance" when Workbench had already confirmed that hadn't happened; item 7 described Phase C as pending when it's complete. Both corrected to match §5's accurate state.
- **v0.5.0** — Phase C complete (Workbench: `buildMilestoneSchedule()`, verified). Corrected a factual error in §5, flagged directly by Workbench (2026-07-29 2353 UTC): this document had claimed Workbench was "the proven implementation partner for Role and RiskItem alike, both verified end-to-end" — RiskItem has not been implemented anywhere; Migration Plan Step 10 exists and is approved but had not yet reached Workbench's session as an explicit relay. §4's Phase C row updated to Complete.
- **v0.4.0** — Everything but 24748-4 access moved forward in one pass (Ron, 2026-07-29): `RiskItem` implemented as PKM v0.6.0 (§3.1's design shipped, not just proposed — see PKM Migration Plan v0.5.0 Step 10 for Workbench guidance). Architecture Guidance v1.5.0 §11–§12 drafted (SEMP Generation Pattern + brief RiskItem tagging note) — §2 below is now historical record of what was proposed, matching what shipped. §4's Phase B (Role) already complete; Phase C (schedule-table prototype) approved to relay to Workbench as real guidance, no longer held behind "wait for full proposal review." Phase A remains genuinely blocked on 24748-4 access — the one item explicitly deferred, not resolved.
- **v0.3.0** — Ron's answers to §6 items 2–5: (2) `Role` **approved to proceed now**, decoupled from this proposal's broader decision — implemented as PKM v0.5.0's `Role` entity and Migration Plan v0.4.0's Step 9, not just sketched here anymore. (3) `RiskItem`/`Gap` boundary — **agreed as proposed** (the `escalatedToRiskItemId` bridge). (4) Risk Management Board — **no PKM entity, for now** — explicitly parked pending a separate risk-planning session, not fully closed. (5) Generated SEMP audit trail — **not yet**, left open. §3's Role sketch updated to reflect the entity is now real, not tentative; §3.1 updated to reflect items 3–4's resolution; §6 renumbered accordingly.
- **v0.2.1** — Fixed a formatting defect from v0.2.0's edit: the "## 4. Phased migration plan" section header was accidentally deleted when §3.1 was inserted, leaving the Phase A–F table floating under §3.1 with no heading. Content of the table itself was never affected — this is a heading-only fix. Also corrected §1's stale in-prose reference to "PKM v0.3.1" (the table header already said v0.4.0 correctly; the sentence above it hadn't been updated to match).
- **v0.2.0** — Added §3.1, Risk Management, closing the gap Ron flagged in review: this proposal's original §1 mapping table had no home for risk at all. Grounded in the DoD RIO Management Guide (Dec 2023, now in Project Knowledge) and INCOSE SE Handbook §2.3.4.4/2.3.4.5. Proposes a unified `RiskItem` entity (Risk/Issue/Opportunity, one entity with a type discriminator — mirroring the Milestone `SETR`/`AcquisitionGate` pattern already established, and directly following the RIO Guide's own suggestion that programs may combine all three registers into one). Identifies PRMP as already PKM-conformant via the existing `Deliverable` entity — no new entity needed for the plan document itself. Flags the `Gap`/`RiskItem` boundary as a real open design question, not presumed here.
- **v0.1.0** — Initial planning pass, scoping what a v2.0 architecture update for automated SEMP generation would actually require, before any migration plan or implementation coaching goes out.

---

## 0. Grounding — what a SEMP is required to contain

Per this project's own convention (SE/PM/CM domain claims need authoritative grounding): the INCOSE SE Handbook, 5th Edition states the complete authoritative SEMP outline lives in **ISO/IEC/IEEE 24748-4**, not the Handbook itself — the Handbook gives an 11-item summary (SE-organization interfaces, role responsibilities/authority, system boundaries/scope, technical objectives/assumptions/constraints, infrastructure/resource management, technical schedule and decision gates, SE process definitions, Technical Process planning, Technical Management Process planning, quality-characteristic approaches, major technical deliverables) and explicitly defers to 24748-4 for the complete version.

**Gap to flag before this hardens into a template:** design chat has the Handbook's summary and public abstract-level search results for 24748-4 (confirming a five-area structure: technical project summary, organization, technical definition planning, execution and control, supporting process plans — plus note that a second edition, 24748-4:2026, superseded the 2016 first edition earlier this year), but not the standard's own full clause-by-clause text. **Recommend obtaining a personal-use copy of ISO/IEC/IEEE 24748-4:2026 the same way the INCOSE Handbook was handled** — uploaded to this Project's Knowledge section, not committed to any repo — before the SEMP Template (§3 below) is finalized section-by-section. Everything in this proposal is scoped at the category level, which is stable enough to plan against; the exact clause numbering isn't.

A second, DoD-flavored structure is also on file (JSSSEH, referencing the older MIL-STD/SEMP tradition): technical program planning & control, the SE process itself, and engineering-specialty integration (safety, reliability, maintainability, human engineering, etc.). Given Workbench's AAF/MCA acquisition-pathway framing, this structure may map more naturally to a DoD-context program than the pure ISO framing does — worth keeping both in view rather than picking one prematurely.

---

## 1. The core idea: three inputs, one generated document

```
SEMP Template (methodology layer, public-safe)
        +
PKM instance data (structure — already exists per-baseline/per-project)
        +
PDKM instance data (real program content — per-program, CUI)
        ↓
   Generated SEMP (program-specific output — never committed publicly)
```

This isn't a new modeling problem so much as a **document-assembly problem layered on top of what already exists.** Looking at the current PKM v0.6.0 entity table against the Handbook's 11-item outline, most sections already have a structural home:

| SEMP section (Handbook summary) | Existing PKM source |
|---|---|
| Technical schedule, milestones, decision gates | `Milestone` (both `SETR` and `AcquisitionGate` types) |
| Major technical deliverables | `Deliverable` |
| System boundaries and scope | `CI`, `LogicalSubsystem`, `Requirement` |
| SE process definition / Technical Process planning | `ChecklistItem` (criteria already structured per milestone) |
| Findings / open risk items feeding planning | `Gap`, `ActionItem` |

**Sections with no clean existing home** — these are the real gaps, not the ones above:

| SEMP section | Gap |
|---|---|
| Responsibilities and authority of key engineering roles | **Resolved — `Role` is now a first-class PKM entity** (v0.5.0 §2–§3), approved and proceeding as Migration Plan v0.4.0 Step 9. No longer a gap. |
| Organization / SE interfaces with rest of org | No PKM home at all — likely inherently PDKM content (program org charts are about as program-specific as content gets), but PKM should at minimum define *where* that content type is referenced from |
| Technical objectives, assumptions, constraints | Same question — likely PDKM content, not PKM structure, per the content-boundary test; PKM's job may just be a stable reference point, not a decomposed entity |
| Infrastructure/resource management | No current PKM entity; unclear yet whether this needs one or is pure PDKM narrative |
| **Risk management** *(was entirely missing from this table in v0.1.0 — Ron flagged it)* | **Resolved — `RiskItem` is now a first-class PKM entity** (v0.6.0 §2–§3), implemented via Migration Plan v0.5.0 Step 10. No longer a gap. |

This table is the actual scope of "what's new" — it's smaller than a full v2.0 might suggest at first. Worth stating plainly: **most of the hard modeling work already happened, across Steps 1–8.** This proposal is mostly about *assembly*, not new structure.

---

## 2. Architecture Guidance §12, SEMP Generation Pattern — DRAFTED, matches this proposal

**This section is now historical record.** Architecture Guidance v1.5.0 §12 was drafted matching what's proposed below almost exactly — where this reads as "a new section for Architecture Guidance," treat it as "what's now live in §12."

A new section for Architecture Guidance, following the same methodology/data separation discipline as everything else in that document:

- **New directory:** `/methodology/semp-template/` — parallel to `/methodology/prompts`, holding the section-by-section template mapping (the table in §1 above, formalized) and any generation prompts, versioned the same way.
- **Generation approach:** structured sections (schedule tables, deliverable lists, RACI-style role/action tables) are assembled directly from PKM+PDKM data — no AI call needed, this is pure query-and-format, same risk profile as any other read-only view. Narrative sections (SE process description, objectives/constraints prose) go through the provider abstraction (§3) via `completeStructured()` or `complete()`, same interface every other AI-assisted feature already uses — no new provider pattern needed.
- **Content-boundary test applies at the section level, same test as everywhere else:** is the *assembly logic* for a section (which fields to pull, how to lay out the table) the same regardless of program? If yes, methodology layer, public-safe. Is the *generated output* itself real program content? Always yes, by definition — so the generated SEMP document is data-layer output, generated at runtime, **never committed to any public repo**, same rule as any other real program data.
- **No new provider capability required.** This reuses the existing AI provider abstraction and existing PKM/PDKM data-injection pattern (§4) exactly as designed — the generation feature is a *consumer* of both, not a reason to change either.

---

## 3. `Role` as a first-class entity — RESOLVED, no longer a proposal

**This section is now historical.** What was sketched here as tentative is real: `Role` is implemented in PKM v0.5.0 (§2–§3) and approved to proceed as Migration Plan v0.4.0's Step 9, decoupled from the rest of this document's still-open items.

The sketch below matches what shipped almost exactly — the one refinement made during formalization was scoping `Role` to Project only (not "Project or Program," which the original sketch left ambiguous) and adding an `isDefault` flag so an app can seed a standard taxonomy while still allowing per-program additions, per Ron's explicit tailorability requirement:

| Entity | Represents | Key relationships |
|---|---|---|
| **Role** | A named responsibility/authority position within a program's SE organization | belongs to Project; `name`, `authorityDescription`, `isDefault`; referenced by `ActionItem.assignedRoleId` |

This lets the SEMP's "responsibilities and authority" section generate directly from real `Role` records — see §1's gap table, now marked resolved. `RiskItem.ownerRoleId` (§3.1 below) is a second, independent consumer of the same entity, which reinforced rather than duplicated this decision.

---

## 3.1 Risk Management (`RiskItem`) — IMPLEMENTED, no longer a proposal

**This section is now historical.** `RiskItem` shipped as PKM v0.6.0 (§2–§3) — everything sketched below matches what was implemented; where the text below reads as "proposed," treat it as "implemented as described" instead. See PKM Migration Plan v0.5.0 Step 10 for the guidance now relayed to Workbench.

Ron flagged this directly: risk management was entirely absent from v0.1.0's scope, despite most SEMP outlines treating it as core content (JSSSEH's DoD-flavored three-part SEMP structure names "program risk analysis" explicitly; the RIO Guide itself notes a Program Risk Management Plan — PRMP — typically coordinates with the SEMP/technical strategy). This section closes that gap, grounded in the DoD Risk, Issue, and Opportunity (RIO) Management Guide for Defense Acquisition Programs (Dec 2023) and INCOSE SE Handbook §2.3.4.4–2.3.4.5.

**One finding worth stating up front: the plan document itself needs no new entity.** A PRMP is a document/artifact type, structurally identical to every other Deliverable already in PKM (SSDD, RTM, TDP, SEMP itself). No new entity — `Deliverable.type = "PRMP"` is already sufficient, the same "already conformant" finding PKM Migration Step 0 recorded for CI↔LogicalSubsystem. The actual gap is the *tracked content* — individual risks, issues, and opportunities — which has no PKM home at all today.

### Proposed entity: `RiskItem`

The RIO Guide treats Risk, Issue, and Opportunity as running the same five-step process (planning, identification, analysis, mitigation/handling, monitoring) through the same register shape, and explicitly states programs may combine all three into a single register. INCOSE's Handbook treatment agrees: opportunity management is "the above [risk] process activity description, with a few adjustments in terminology, and the term 'opportunity' substituted for the term 'risk.'" This is the same shape of evidence that led to Milestone's `SETR`/`AcquisitionGate` discriminator (PKM v0.3.0) — one entity, one type field, rather than three parallel entities:

| Field | Source / rationale |
|---|---|
| `itemType` | `"Risk"` \| `"Issue"` \| `"Opportunity"` — RIO's own continuum: an Issue is a Risk whose likelihood has reached 1 (already happened); an Opportunity is a Risk with a positive rather than negative consequence framing. Same register, same process, per RIO Guide §2/§3 and INCOSE's substitution note. |
| `category` | e.g. `"Technical"` \| `"Cost"` \| `"Schedule"` \| `"Programmatic"` \| `"Business"` — RIO's own risk-register categories (Table 2-4 example). |
| `likelihood` | Ordinal 1–5, RIO Table 2-2 pattern. **Null for `itemType: "Issue"`** — RIO is explicit that issue probability is treated as 1, so likelihood isn't scored for issues at all, not just defaulted to a max value. |
| `consequenceCost`, `consequenceSchedule`, `consequencePerformance` | Ordinal 1–5 each, RIO's three-dimension consequence scoring (Table 2-1) — the risk level uses the *maximum* of these three, not an average. |
| `riskLevel` | Derived (`Low`/`Moderate`/`High`), not stored independently — computed from `likelihood` × max(consequence scores) via the standard risk-matrix mapping (RIO Figure 2-5, INCOSE Figure 2.28's generic I–V scale). Same derived-not-stored pattern already used for Acquisition Phase and Baseline reconciliation status. |
| `mitigationStrategy` | `"Accept"` \| `"Avoid"` \| `"Transfer"` \| `"Control"` — RIO's four standard options (§2.4), same four terms apply to issues per RIO's explicit cross-reference. |
| `ownerRoleId` | References the `Role` entity (now implemented, PKM v0.5.0 §2 — see §3 above) — RIO requires an owner ("RMB assigns an owner for each approved issue/risk"). This was the second real use case for `Role`, alongside SEMP's own "responsibilities" section — both are now served by the same resolved entity. |
| `linkedMilestoneId`, `linkedCiId` | Optional references — RIO's register includes a "Linked WBS/IMS ID#" column; PKM already has `Milestone` and `CI` as the natural WBS/schedule anchors, no new structure needed for this linkage. |
| `identifiedDate`, `approvalDate`, `plannedClosureDate`, `actualClosureDate` | RIO register fields, same shape as `Milestone`'s actual/planned date pattern already established. |
| `status` | Program-defined open/closed/monitoring states — left as a plain string for now, same provisional treatment PKM already gives `ChecklistItem.domain` (§5 open question #2) rather than presuming a fixed enum ahead of real usage. |

**Mitigation ties to the existing `ActionItem` entity, not new structure.** RIO's mitigation implementation plan (what, when, who, cost/schedule/performance impact, resources) is the same shape as `ActionItem`'s remediation-task role. Proposed: `ActionItem.resolvesRiskItemId`, coexisting with the existing `resolvesGapId` — a risk's mitigation plan is an ActionItem the same way a Gap's remediation is.

### `RiskItem` vs. `Gap` — RESOLVED, agreed as proposed

Ron confirmed: distinct entities, bridged rather than merged, as originally proposed:

- `Gap` is scoped to SE/CM conformance — "a finding — missing, inconsistent, or non-compliant item" — and is always already-certain (something *was* found).
- `RiskItem` (as an Issue) is also already-certain, but its scope is broader — any program-level cost/schedule/performance problem, including ones with zero SE/CM conformance dimension (RIO's own example: a supply-chain-driven schedule slip).
- **`Gap.escalatedToRiskItemId`** (optional) is the confirmed bridge — a Gap with real program-level cost/schedule/performance consequence beyond its immediate SE/CM finding can escalate into a formal `RiskItem` (typically `itemType: "Issue"`), without merging the two entities or diluting Gap's narrow, precise meaning.

This is now settled design direction and was implemented exactly as agreed — `RiskItem`'s `Gap.escalatedToRiskItemId` bridge shipped in PKM v0.6.0 and is confirmed live in Workbench's own implementation (status report v1.8.0).

### What stays PDKM (real content, never PKM structure)

Risk statement/description (RIO recommends an If-Then format), root cause narrative, mitigation strategy rationale, consequence justification, and any historical trend/burn-down data points are all real per-program content — same treatment as `Requirement.statement` or `Gap.description` already receive. Nothing about this proposal suggests decomposing narrative risk content into PKM structure.

**Risk Management Board (RMB) — parked, not fully closed.** Ron's direction: no PKM entity for now, but this is explicitly provisional pending a separate, dedicated risk-planning session — not a considered "no" the way the Role and Gap/RiskItem questions above are. Treat this as unresolved-but-not-blocking rather than settled; revisit once that session happens rather than treating "no entity" as final architecture.

---

## 4. Phased migration plan (draft — risk-sequenced, same discipline as every prior migration plan)

| Phase | What | Risk | Status |
|---|---|---|---|
| **A** | Formalize the SEMP Template — section-by-section mapping table (§1), documentation only | Low | **Blocked** — genuinely needs ISO/IEC/IEEE 24748-4:2026 (§0), deferred by Ron to tomorrow. The only phase still gated. |
| **B** | `Role` entity | Low-moderate | **Complete** — PKM v0.5.0, Migration Plan Step 9, implemented and verified by Workbench (v1.7.0). |
| **C** | Prototype: generate the **schedule/milestone table only** — pure structured assembly from existing `Milestone` records, zero new PDKM tailoring needed, zero AI calls | Low | **Complete** — `buildMilestoneSchedule()`, verified (Playwright confirmed chronological ordering via DOM inspection). Zero schema change, zero AI calls, as planned. |
| **D** | Extend structured generation to deliverables, RACI (`Role`/`RiskItem` both now exist), technical scope (CI/Requirement tree) | Moderate | **`RiskItem` slice complete** — `buildRiskRegister()` implemented and verified, wired into §3.2.1, sorted by derived risk score descending. RACI and deliverables-list slices remain not yet approved — narrower scope than the full Phase D as originally sketched. |
| **E** | Narrative sections via AI-assisted drafting from PDKM content (objectives, constraints, SE process description) | High — genuine content generation, not mechanical assembly | Blocked on Phase A (needs the real section list from 24748-4 before drafting narrative-section prompts). |
| **F** | Full-document assembly + validation pass — does it read coherently, does a real reviewer accept it as a usable SEMP draft | High (judgment, not technical) | Blocked on Phase E. |

This follows the same "mechanical/additive first, judgment-heavy last" sequencing Architecture Guidance §7 and every migration plan so far has used — Phase C in particular is deliberately chosen as the very first prototype because it requires **zero new PDKM content and zero AI calls**, just proves the assembly pattern works end-to-end on data that already exists.

---

## 5. Who does what — status update

- **udm-exchange:** has committed Architecture Guidance §11–§12, PKM `Role` (v0.5.0) and `RiskItem` (v0.6.0), and Migration Plan Steps 9–10 — all confirmed relayed and implemented. What remains uncommitted: Phase A's actual template-mapping document, blocked on 24748-4 access.
- **SE Workbench:** proven implementation partner for `Role` (verified, v1.7.0), Phase C (schedule-table prototype, verified, `buildMilestoneSchedule()`), and now `RiskItem` (their own Step 12, status report v1.8.0 / feedback v1.1.0 — CRUD route, `RiskItemsPage.tsx`, seed data covering all three `itemType` values, both new reference fields). All three of this document's implementation threads are now shipped and verified as of 2026-07-30.

---

## 6. Open items — status as of 2026-07-30 (updated same day, twice)

1. **Grounding source — still open.** The generated SEMP package cites DI-SESS-81785B and OSD SEP Outline v4.1 directly. Not yet decided whether this supersedes the need for ISO/IEC/IEEE 24748-4 access.
2. ~~`Role` entity decision~~ **Resolved and shipped.** PKM v0.5.0, Migration Plan v0.4.0 Step 9, implemented and verified by Workbench (status report v1.7.0).
3. ~~`RiskItem` vs. `Gap` boundary~~ **Fully resolved, design and data both confirmed.** `Gap.escalatedToRiskItemId` populated (`gap-003` → `risk-001`), verified against real data as of Migration Plan Step 14.
4. **Risk Management Board — still parked, not resolved.** (§3.1) Explicitly provisional pending a separate risk-planning session.
5. **Whether generated SEMP output itself needs any PKM-level audit trail — still not yet.** Left open.
6. ~~`RiskItem` itself~~ **Fully resolved — structure, data, and SEMP-generation wiring all complete and verified.** Migration Plan Steps 10/14, `buildRiskRegister()` implemented and live in §3.2.1.
7. ~~Four data-integrity gaps from the 2026-07-30 export cross-check~~ **All four closed.** Three resolved via real implementation (Steps 12, 14). The fourth (Step 11, `baselineId`/`projectId`) closed as not reproducible against Workbench's real data by any method tried, including a fresh export against a named commit — the original discrepancy is real but not attributable to this app.
8. **New: static-deploy caching limitation, worth Architecture Guidance's attention.** Workbench identified that their GitHub Pages/localStorage deploy mode (§3.1's static-only exception) only backfills entirely-missing collections for a cached browser session, never new fields on existing rows — meaning any app using this pattern can serve silently stale schema indefinitely to a returning visitor. This affects any app built against §3.1, not just Workbench. Candidate for an Architecture Guidance addition, not yet drafted.
9. **Migration plan and coaching commands — status.** Steps 9–10, 12, 14 all relayed, implemented, and verified. Step 11 open (see item 7). Step 13 resolved. Architecture Guidance §11–§12 drafted and live. Phase C complete; Phase D's `RiskItem` slice complete. What remains undrafted: Phase A's template-mapping document and Phase E/F's narrative-generation guidance — grounding source for both still an open question (item 1).

---

*Grounding source (item 1) is still open — DI-SESS-81785B/SEP Outline v4.1 may be sufficient, not yet decided. Everything else in this document — Role, RiskItem (structure, data, and generation wiring), Phase C, and all four export-cross-check findings — is now resolved or closed.*
