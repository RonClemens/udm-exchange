# Product/Domain Knowledge Model (PDKM) — Pseudo-Data Example

**Version:** 0.2.0 (Exploratory / Draft — first pass, not a template for real program data)
**Last updated:** 2026-07-28
**Status:** Draft
**Companion to:** PKM Entity Model v0.3.1
**Machine-readable companion:** `pdkm-pseudodata-v0.2.0.json` (§10) — field-for-field match to this document; bump together going forward.

**Changelog:**
- **v0.2.0** — Added a companion machine-readable JSON export (§10, `pdkm-pseudodata-v0.2.0.json`) matching this document's entity instances field-for-field, structured for a webapp to fetch/parse as synthetic mock data per Architecture Guidance §4's data-injection pattern. Established the convention that the JSON's version tracks this document's version going forward — bump both together on any content change, not just the JSON. No changes to entity content itself.
- **v0.1.0** — Initial pseudo-data pass, exercising every PKM v0.3.0 entity (including the new Milestone `milestoneType` discriminator) against a single fully synthetic scenario, to check the model holds together with actual instance data before it's proposed as guidance for a real app's PDKM.

---

## 0. What this is, and isn't

Every value below is **fictional** — a synthetic scenario invented to exercise the PKM structure, not derived from, or resembling, any real program. Program name, CI names, requirement text, findings, and dates are all placeholder content, in the same spirit as Architecture Guidance's `/mock-data` convention: public-safe by construction because nothing in it is real.

This is **not** a PDKM template or schema recommendation — the PKM Entity Model already defines structure; PDKM content shape is inherently program-specific and out of this project's scope per the PKM document's own stated boundary. This exists only to answer one question: *does a full instance graph across all eleven-plus PKM entities actually cohere*, or does building real data reveal gaps the entity table alone doesn't show?

Two things below are flagged as speculative rather than settled: the `ReconciliationEvent` shape (PKM §5 open question #1 is still open) and the `AcquisitionGate` milestone's exact field set (new in v0.3.0, not yet implemented anywhere). Both are included so the scenario stays complete, not because either is decided.

**Scenario, one sentence:** a fictional modernization program running two parallel baselines — a mature legacy baseline nearing completion, and a new baseline still early in its SETR sequence — with one open gap and one AAF acquisition-decision gate in progress.

---

## 1. Program → Project

| Field | PROGRAM-001 |
|---|---|
| id | `PROGRAM-001` |
| name | "Synthetic Sensor Recapitalization Program" *(fictional)* |
| description | Coordinates modernization of a fictional legacy sensor line across two concurrent baselines. |

| Field | PROJECT-001 |
|---|---|
| id | `PROJECT-001` |
| programId | `PROGRAM-001` |
| name | "Synthetic Sensor Avionics Refresh" *(fictional)* |
| baselineIds | `[BASELINE-A, BASELINE-B]` |

---

## 2. Baseline (two, coexisting)

| Field | BASELINE-A (legacy) | BASELINE-B (new) |
|---|---|---|
| id | `BASELINE-A` | `BASELINE-B` |
| projectId | `PROJECT-001` | `PROJECT-001` |
| baselineType | `Product` (as-built, near PCA) | `Functional` (early, post-SFR) |
| reconciledIntoBaselineId | — (see §9, speculative) | — (see §9, speculative) |

---

## 3. Milestone — both `milestoneType` values exercised

| Field | MS-A-PDR | MS-B-SRR | MS-B-MSA |
|---|---|---|---|
| id | `MS-A-PDR` | `MS-B-SRR` | `MS-B-MSA` |
| baselineId | `BASELINE-A` | `BASELINE-B` | `BASELINE-B` |
| milestoneType | `SETR` | `SETR` | `AcquisitionGate` |
| event | `PDR` | `SRR` | `MS-A` |
| pathway | — (SETR type; field n/a) | — (SETR type; field n/a) | `MCA` |
| status | `Complete` | `Complete` | `Complete` |
| actualDate | `2024-03-14` *(fictional)* | `2025-01-09` *(fictional)* | `2025-02-01` *(fictional)* |
| plannedDate | `2024-03-01` *(fictional)* | `2025-01-15` *(fictional)* | `2025-02-15` *(fictional)* |
| establishesBaselineId | `BASELINE-A` (PDR establishes the Allocated→Product transition for this baseline) | — (SRR doesn't establish a baseline in this scenario) | — (AcquisitionGate type; field n/a) |

**What this exercises:** the same `PDR`-type event name could recur for Baseline B later without id collision (each Milestone id is baseline-scoped), and `MS-B-MSA` shows the `AcquisitionGate` type coexisting with `SETR` milestones under the same baseline, exactly the shape PKM v0.3.0 §3 describes. Baseline A has no `AcquisitionGate` milestone in this scenario — nothing requires every baseline to have one.

---

## 4. ChecklistItem (belongs to a Milestone)

| Field | CHK-B-SRR-001 |
|---|---|
| id | `CHK-B-SRR-001` |
| milestoneId | `MS-B-SRR` |
| criterion | "Mission need statement approved" *(fictional, illustrative)* |
| status | `Met` |
| evidenceType | `Deliverable` |
| evidenceId | `DEL-001` |
| domain | `"Requirements"` *(plain string per PKM §5 open question #2 — still unresolved, used here as-is)* |

---

## 5. LogicalSubsystem ⇄ CI (many-to-many)

| Field | LS-001 | LS-002 |
|---|---|---|
| id | `LS-001` | `LS-002` |
| baselineId | `BASELINE-A` | `BASELINE-A` |
| name | "Signal Processing" *(fictional)* | "Power Distribution" *(fictional)* |

| Field | CI-001 | CI-002 |
|---|---|---|
| id | `CI-001` | `CI-002` |
| baselineId | `BASELINE-A` | `BASELINE-A` |
| name | "Synthetic Processor Module" *(fictional)* | "Synthetic Power Board" *(fictional)* |
| subsystemIds | `[LS-001, LS-002]` *(one CI serving two subsystems — the many-to-many case PKM v0.2.0 corrected for)* | `[LS-002]` |
| primarySubsystemId | `LS-001` | `LS-002` |

---

## 6. Requirement (satisfiedBy CI, many-to-many)

| Field | REQ-001 |
|---|---|
| id | `REQ-001` |
| baselineId | `BASELINE-A` |
| statement | *(real requirement text is PDKM content that would live here — omitted; this row demonstrates structure only)* |
| satisfiedByCiIds | `[CI-001, CI-002]` *(one requirement, satisfaction spans two CIs — mirrors the real `delta-001` case Workbench reported for Step 4)* |
| parentRequirementId | — (top-level in this scenario) |
| verifiedByVerificationEventId | `VER-001` |

---

## 7. Deliverable, Gap, ActionItem, VerificationEvent

| Field | DEL-001 |
|---|---|
| id | `DEL-001` |
| requiredByMilestoneId | `MS-B-SRR` |
| type | `"SSDD"` |
| producedForId | `PROJECT-001` |

| Field | GAP-001 |
|---|---|
| id | `GAP-001` |
| baselineId | `BASELINE-A` |
| blocksMilestoneId | — (blocks a ChecklistItem instead, not a Milestone directly, in this instance) |
| blocksChecklistItemId | `CHK-B-SRR-001`* |
| foundInEntityType | `CI` |
| foundInEntityId | `CI-001` |
| description | *(real finding text is PDKM content — omitted)* |

\* *illustrates that "blocks Milestone or ChecklistItem" (§2) is an either/or per instance, not both — worth confirming that's the intended reading when this hardens.*

| Field | ACT-001 |
|---|---|
| id | `ACT-001` |
| resolvesGapId | `GAP-001` |
| assignedRole | `"Lead Systems Engineer"` *(mirrors Workbench's real Step 7 role taxonomy)* |
| status | `Open` |

| Field | VER-001 |
|---|---|
| id | `VER-001` |
| verifiesRequirementId | `REQ-001` |
| producesEvidenceForChecklistItemId | — (not linked to a ChecklistItem in this instance) |
| method | `"Test"` |
| result | `Pass` |

---

## 8. What this pass surfaced (candidate follow-ups, not decisions)

1. **Gap's "blocks" relationship reads as either/or in practice** (§7 above) — **resolved in PKM v0.3.1**, which now states this explicitly as mutually exclusive per instance. Independent of Workbench's own already-flagged `foundIn`-cardinality question (their optional follow-up #3), which remains open.
2. **`AcquisitionGate` milestones have no natural ChecklistItem use in this scenario** — `MS-B-MSA` has zero associated ChecklistItems, which is fine (Workbench's real implementation drives gate status directly, not via checklist evidence), but it does mean "has many ChecklistItems" in §2's Milestone row is really a `SETR`-type-only capability, not universal to Milestone. Minor wording gap, not a structural one.
3. Nothing else broke — every other entity/relationship in v0.3.0 populated cleanly with one instance each, including the many-to-many CI↔LogicalSubsystem case and the cross-baseline Requirement satisfaction case.

---

## 9. Speculative section — Baseline reconciliation (PKM §5 open question #1, still unresolved)

Included for scenario completeness only; this section should **not** be read as PDKM guidance until open question #1 is actually resolved.

**If `reconciledIntoBaselineId` stays a field on Baseline:**

| Field | BASELINE-B |
|---|---|
| reconciledIntoBaselineId | `BASELINE-A` *(speculative — not a real decision)* |

**If it becomes a distinct `ReconciliationEvent` entity instead** (the direction PKM §5 leans toward, given `AbCompatibilityRow` evidence):

| Field | REC-001 (speculative) |
|---|---|
| id | `REC-001` |
| reconciledFromBaselineId | `BASELINE-B` |
| reconciledIntoBaselineId | `BASELINE-A` |
| compatibilityStatus | `"Under Review"` *(mirrors `AbCompatibilityRow.compatibilityStatus`)* |
| lastReviewedDate | `2026-06-01` *(fictional)* |

Both are shown only to confirm neither shape breaks anything else in the graph above — not to nudge the open question toward either answer.

---

## 10. Machine-readable companion

`pdkm-pseudodata-v0.2.0.json` mirrors every entity instance above field-for-field, as a single fetchable file — the same shape as Architecture Guidance §4's `/mock-data/synthetic-program.json` pattern: a webapp loads it at build/dev time as a stand-in `dataSource`, same schema a real PDKM would eventually populate.

**Structure:** one top-level array per entity type (`programs`, `projects`, `baselines`, `milestones`, `checklistItems`, `logicalSubsystems`, `configurationItems`, `requirements`, `deliverables`, `gaps`, `actionItems`, `verificationEvents`), plus a `meta` block and a clearly separated `speculative` block holding both candidate shapes for the still-open Baseline-reconciliation question (§9) — a consuming app should treat anything under `speculative` as non-stable and not build against it.

**Versioning:** this JSON is versioned identically to this document (currently v0.2.0) and the two are bumped together — the JSON is not an independently-versioned artifact.

---

*This document is a scratch validation pass, not a canonical artifact. If the PKM v0.3.0 revision is accepted as-is, this file doesn't need to accompany it to `udm-exchange` — it's useful here as a design-chat sanity check, not necessarily as something Workbench or a future app needs to consume.*
