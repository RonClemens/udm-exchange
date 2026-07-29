# UDM v2.0 — Automated SEMP Generation Architecture (Planning Proposal)

**Version:** 0.2.1 (Exploratory / Proposal — not yet reviewed by Ron, not yet relayed to either coding chat)
**Last updated:** 2026-07-29
**Status:** Draft
**Depends on:** PKM Entity Model v0.4.0, Architecture Guidance v1.4.0, SE Workbench Migration Plan v0.3.1

**Changelog:**
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

This isn't a new modeling problem so much as a **document-assembly problem layered on top of what already exists.** Looking at the current PKM v0.4.0 entity table against the Handbook's 11-item outline, most sections already have a structural home:

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
| Responsibilities and authority of key engineering roles | `Role` is currently just a constrained string on `ActionItem`/`Recommendation.owner` (Migration Plan §9 item 3, already in flight with Workbench) — not a queryable, describable entity with authority/responsibility content of its own |
| Organization / SE interfaces with rest of org | No PKM home at all — likely inherently PDKM content (program org charts are about as program-specific as content gets), but PKM should at minimum define *where* that content type is referenced from |
| Technical objectives, assumptions, constraints | Same question — likely PDKM content, not PKM structure, per the content-boundary test; PKM's job may just be a stable reference point, not a decomposed entity |
| Infrastructure/resource management | No current PKM entity; unclear yet whether this needs one or is pure PDKM narrative |
| **Risk management** *(was entirely missing from this table in v0.1.0 — Ron flagged it)* | **Scoped in §3.1 below.** No PKM entity for risk/issue/opportunity tracking exists yet; the plan document itself (PRMP) turns out to already be PKM-conformant with no new structure needed |

This table is the actual scope of "what's new" — it's smaller than a full v2.0 might suggest at first. Worth stating plainly: **most of the hard modeling work already happened, across Steps 1–8.** This proposal is mostly about *assembly*, not new structure.

---

## 2. Proposed Architecture Guidance addition — §12, SEMP Generation Pattern

A new section for Architecture Guidance, following the same methodology/data separation discipline as everything else in that document:

- **New directory:** `/methodology/semp-template/` — parallel to `/methodology/prompts`, holding the section-by-section template mapping (the table in §1 above, formalized) and any generation prompts, versioned the same way.
- **Generation approach:** structured sections (schedule tables, deliverable lists, RACI-style role/action tables) are assembled directly from PKM+PDKM data — no AI call needed, this is pure query-and-format, same risk profile as any other read-only view. Narrative sections (SE process description, objectives/constraints prose) go through the provider abstraction (§3) via `completeStructured()` or `complete()`, same interface every other AI-assisted feature already uses — no new provider pattern needed.
- **Content-boundary test applies at the section level, same test as everywhere else:** is the *assembly logic* for a section (which fields to pull, how to lay out the table) the same regardless of program? If yes, methodology layer, public-safe. Is the *generated output* itself real program content? Always yes, by definition — so the generated SEMP document is data-layer output, generated at runtime, **never committed to any public repo**, same rule as any other real program data.
- **No new provider capability required.** This reuses the existing AI provider abstraction and existing PKM/PDKM data-injection pattern (§4) exactly as designed — the generation feature is a *consumer* of both, not a reason to change either.

---

## 3. Proposed PKM extension — `Role` as a first-class entity (tentative, tied to an open item already in motion)

This directly extends Migration Plan §9 item 3, which is already with Workbench for input (per Ron's direction: role taxonomy should be general-default-but-tailorable per program). If Workbench confirms their taxonomy needs to become program-configurable rather than a fixed enum, that's effectively the same structural change a `Role` entity would provide — worth resolving as one decision, not two:

**Sketch, not yet proposed as a PKM revision:**

| Entity | Represents | Key relationships |
|---|---|---|
| **Role** | A named responsibility/authority position within a program's SE organization | belongs to Project (default catalog) or Program; `name`, `authorityDescription`; referenced by `ActionItem.assignedRoleId` in place of a plain string |

This would let the SEMP's "responsibilities and authority" section generate directly from real `Role` records instead of a hardcoded list — and gives Ron's tailorability requirement (add/remove roles per program) a real structural home instead of an app-side enum each app manages independently.

**Not finalizing this here.** It's included so Workbench's answer to the role-taxonomy question (already asked) can be evaluated against this concrete shape rather than in the abstract.

---

## 3.1 Proposed PKM extension — Risk Management (`RiskItem`), grounded in the DoD RIO Guide

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
| `ownerRoleId` | References the same `Role` entity sketched in §3 above — RIO requires an owner ("RMB assigns an owner for each approved issue/risk"). **This is the second real use case for `Role`, beyond SEMP's own "responsibilities" section** — worth weighing together with the Migration Plan §9 item 3 decision, not separately. |
| `linkedMilestoneId`, `linkedCiId` | Optional references — RIO's register includes a "Linked WBS/IMS ID#" column; PKM already has `Milestone` and `CI` as the natural WBS/schedule anchors, no new structure needed for this linkage. |
| `identifiedDate`, `approvalDate`, `plannedClosureDate`, `actualClosureDate` | RIO register fields, same shape as `Milestone`'s actual/planned date pattern already established. |
| `status` | Program-defined open/closed/monitoring states — left as a plain string for now, same provisional treatment PKM already gives `ChecklistItem.domain` (§5 open question #2) rather than presuming a fixed enum ahead of real usage. |

**Mitigation ties to the existing `ActionItem` entity, not new structure.** RIO's mitigation implementation plan (what, when, who, cost/schedule/performance impact, resources) is the same shape as `ActionItem`'s remediation-task role. Proposed: `ActionItem.resolvesRiskItemId`, coexisting with the existing `resolvesGapId` — a risk's mitigation plan is an ActionItem the same way a Gap's remediation is.

### The real open question: `RiskItem` vs. `Gap` — not resolved here

These are related but not obviously the same thing, and this proposal deliberately doesn't collapse them:

- `Gap` is scoped to SE/CM conformance — "a finding — missing, inconsistent, or non-compliant item" — and is always already-certain (something *was* found).
- `RiskItem` (as an Issue) is also already-certain, but its scope is broader — any program-level cost/schedule/performance problem, including ones with zero SE/CM conformance dimension (RIO's own example: a supply-chain-driven schedule slip).

A `Gap` that has real program-level cost/schedule/performance consequence beyond its immediate SE/CM finding looks like it should be able to *escalate* into a formal `RiskItem` (typically `itemType: "Issue"`) — proposed as an optional `Gap.escalatedToRiskItemId` reference, not a merge of the two entities. This keeps Gap's narrow, precise SE/CM-conformance meaning intact while giving it a bridge into program-level risk tracking when that's actually warranted. **Flagging this as the real design decision in this section — not presuming it's the right answer, just the most defensible one found so far.**

### What stays PDKM (real content, never PKM structure)

Risk statement/description (RIO recommends an If-Then format), root cause narrative, mitigation strategy rationale, consequence justification, and any historical trend/burn-down data points are all real per-program content — same treatment as `Requirement.statement` or `Gap.description` already receive. Nothing about this proposal suggests decomposing narrative risk content into PKM structure.

**Not yet resolved, flagged rather than decided:** whether a Risk Management Board (RMB) — RIO's governance body that approves risks/issues and assigns owners — needs any PKM representation at all, or stays pure PDKM/methodology content (a meeting cadence + membership list, similar to how `Program`/`Project` don't model organizational governance structures today). Leaning toward "no PKM entity needed," but not committing to that without more evidence than one source document provides.

---

## 4. Phased migration plan (draft — risk-sequenced, same discipline as every prior migration plan)

| Phase | What | Risk | Depends on |
|---|---|---|---|
| **A** | Formalize the SEMP Template — section-by-section mapping table (§1), documentation only | Low | ISO/IEC/IEEE 24748-4:2026 access (§0 gap) |
| **B** | Resolve `Role` entity question (§3) — extend PKM only if Workbench's answer confirms the need | Low-moderate | Workbench's response (already requested) |
| **C** | Prototype: generate the **schedule/milestone table only** — pure structured assembly from existing `Milestone` records, zero new PDKM tailoring needed, zero AI calls | Low | Phase A |
| **D** | Extend structured generation to deliverables, RACI (once Role exists), technical scope (CI/Requirement tree) | Moderate | Phase B, C |
| **E** | Narrative sections via AI-assisted drafting from PDKM content (objectives, constraints, SE process description) | High — genuine content generation, not mechanical assembly | Phase D |
| **F** | Full-document assembly + validation pass — does it read coherently, does a real reviewer accept it as a usable SEMP draft | High (judgment, not technical) | Phase E |

This follows the same "mechanical/additive first, judgment-heavy last" sequencing Architecture Guidance §7 and every migration plan so far has used — Phase C in particular is deliberately chosen as the very first prototype because it requires **zero new PDKM content and zero AI calls**, just proves the assembly pattern works end-to-end on data that already exists.

---

## 5. Who does what (once this is ready to send — not yet)

- **udm-exchange:** would own committing Architecture Guidance §12, any resulting PKM `Role` extension, and a new `semp-generation/` migration-plan document — likely its own top-level folder in the repo, same precedent as `pkm/examples/` just established.
- **SE Workbench:** the natural first prototype target — richest existing PKM dataset (all 7 base steps + Step 8 in progress). Phase C is low-risk enough to hand them directly once Phase A's template mapping exists.

---

## 6. Open items — none of this should be relayed until these are addressed

1. **ISO/IEC/IEEE 24748-4:2026 access** (§0) — needed before the template mapping in Phase A can be finalized against the actual standard rather than secondary summaries. *(Ron is independently sourcing this.)*
2. **`Role` entity decision** (§3) — pending Workbench's answer to the already-relayed role-taxonomy question. **Now doubly motivated:** both SEMP's "responsibilities and authority" section and `RiskItem.ownerRoleId` (§3.1) need it — worth resolving once, not twice.
3. **`RiskItem` vs. `Gap` boundary** (§3.1, new) — is `Gap.escalatedToRiskItemId` the right bridge, or should these two entities relate some other way (or not at all)? Not decided here.
4. **Risk Management Board — PKM entity or not** (§3.1, new) — leaning "not," not committed.
5. **Whether generated SEMP output itself needs any PKM-level audit trail** (e.g., a record of when/from-what-data-snapshot a SEMP was generated) — genuinely new question, not addressed anywhere above, flagging so it isn't lost.
6. **Not yet drafted:** the actual migration-plan document and any coaching commands to either coding chat. This proposal is the planning pass Ron asked for — next step is Ron's review, then (if it holds up) turning §4 into a real versioned migration plan and §2 into an actual Architecture Guidance §12 draft, the same two-step pattern used for every prior change in this project.

---

*This document is a planning proposal only. Nothing in it has been committed anywhere, and no coaching commands should be drafted for udm-exchange or SE Workbench until Ron has reviewed it and the open items in §6 have at least a direction, if not final answers.*
