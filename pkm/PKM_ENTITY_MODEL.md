# Process Knowledge Model (PKM) — Entity & Relationship Model

**Version:** 0.3.1 (Exploratory / Draft — not yet implemented in any app)
**Last updated:** 2026-07-28
**Status:** Draft
**Companion to:** Reusable SE Webapp Architecture Guidance v1.4.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md)) (§9, Forward Compatibility with a UDM)

**Changelog:**
- **v0.3.1** — Clarified Gap's `blocks` relationship (§2) as mutually exclusive per instance — a given Gap blocks a Milestone *or* a ChecklistItem, never both — surfaced as ambiguous during a pseudo-data validation pass exercising every entity. Documentation-only, no structural change.
- **v0.3.0** — Corrected Milestone's scope from Project to Baseline (§2, §3). The entity-table text had drifted from the relationship diagram in §3, which already showed Milestone nested under each Baseline — this brings §2 in line with §3, not the reverse. Per-baseline-lineage independence (SE Workbench Step 3: SETR events tracked independently per baseline lineage) confirms Baseline is the correct scope. Broadened Milestone to also cover acquisition-decision gates (e.g. Milestone A/B/C under AAF-style pathways) via a new `milestoneType` (`SETR` | `AcquisitionGate`) discriminator, rather than adding a second parallel entity — informed by SE Workbench's `AcquisitionMilestone` implementation (PKM Migration Step 8; [status report v1.3.0](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/feedback/se-workbench/PKM_MIGRATION_STATUS_REPORT.md), §5–§6 item 6). Added `pathway` and conditional `establishesBaselineId` semantics, type-dependent (§3, §4).
- **v0.2.2** — Corrected stale cross-reference: `Companion to:` cited Architecture Guidance v1.3.0, which had drifted two versions behind current (v1.4.0); §9's content and numbering are unaffected. Added the raw URL per Workflow Protocol §3.4. Documentation-only, no structural change.
- **v0.2.1** — Added SE Workbench's `AbCompatibilityRow` as real-world evidence toward open question #1 (§5). No structural change; documentation note only.
- **v0.2.0** — Corrected CI↔LogicalSubsystem relationship from one-to-many to many-to-many (§2, §3), per standard SE allocation-matrix practice (SEBoK Physical Architecture process). Resolves the conflict flagged in SE Workbench's v0.1.0 conformance review.
- **v0.1.0** — Initial draft.

**Scope:** This document defines the **Process Knowledge Model (PKM)** only — the public-safe, cross-program ontology of SE/PM/CM structure. It does **not** define the Product/Domain Knowledge Model (PDKM), which holds real CUI content and is out of scope here. No entity in this document may ever contain program-identifying content (real program names, real CI names, real requirement text) — those live only in a program's PDKM, linked by reference ID.

**Status note:** This is a draft for review, not a system to implement yet. It's being shared across apps (starting with the SE Workbench) to get implementation-level feedback before it hardens into something apps build against.

---

## 1. Grounding

This model draws on three disciplines, kept intentionally distinct because they define different things:

- **Program/Project Management (PMI):** a Program coordinates two or more Projects to realize benefits not achievable by managing them individually; Projects are temporary efforts focused on delivering defined outputs, while Programs focus on outcomes.
- **Configuration Management (EIA-649 / MIL-HDBK-61A):** a Baseline is a formally controlled configuration state at a point in time. Baselines progress through three technical types — Functional (performance requirements), Allocated (requirements distributed to elements), and Product (as-built, verified attributes) — each established at a specific lifecycle milestone (SFR, PDR/CDR-family reviews, PCA respectively). A separate Acquisition Program Baseline (APB) tracks cost/schedule/performance at the program level and is not a technical baseline.
- **Real-world CM practice:** more than one Baseline can legitimately coexist under the same Project — most visibly in modernization/obsolescence programs, where a mature legacy baseline and a new baseline under development run in parallel until reconciled. The model must support this as a normal case, not an edge case.

---

## 2. Entity Table

| Entity | Represents | Key relationships |
|---|---|---|
| **Program** | Coordinated benefit realization across Projects | has many Projects |
| **Project** | A temporary effort delivering a defined output | belongs to Program; has one or more Baselines over time |
| **Baseline** | A formally controlled configuration state at a point in time | belongs to Project; has `baselineType` (Functional / Allocated / Product / Acquisition-Program); established at a Milestone; may coexist with sibling Baselines under the same Project |
| **Milestone** | A gate event within a baseline lineage — either a SETR technical review (SRR, SFR, PDR, CDR, TRR, etc.) or an acquisition decision gate (e.g. Milestone A/B/C under an AAF-style pathway) | belongs to **Baseline** (corrected from Project in v0.2.x — see §3); `milestoneType` (`SETR` \| `AcquisitionGate`) distinguishes the two; a `SETR` milestone may establish the Baseline it belongs to and has many ChecklistItems; an `AcquisitionGate` milestone gates progress within an already-established baseline lineage, does not establish a Baseline, and carries a `pathway` reference (e.g. `"MCA"`) |
| **ChecklistItem** | A discrete readiness criterion within an SE domain | belongs to Milestone; evaluated against evidence from Requirement / CI / Deliverable / VerificationEvent |
| **LogicalSubsystem** | Functional decomposition layer | belongs to Baseline; allocated to/from CI(s) (many-to-many, see §3); may map to a physical enclosure/structure |
| **CI (Configuration Item)** | A structural node in the CI decomposition | belongs to Baseline; allocated to one or more LogicalSubsystems (many-to-many, see §3); satisfies Requirement(s); has Deliverables |
| **Requirement** | A structural requirement node (not the requirement text itself) | belongs to Baseline; satisfiedBy CI; traces to ParentRequirement; verifiedBy VerificationEvent |
| **Deliverable** | A document/artifact type (SSDD, RTM, TDP, SEMP, etc.) | required by Milestone; produced for CI or Project |
| **Gap** | A finding — missing, inconsistent, or non-compliant item | belongs to Baseline; blocks a Milestone *or* a ChecklistItem (mutually exclusive per instance, not both); references the entity it was found in |
| **ActionItem** | A remediation task | resolves Gap; assigned to a role (not a named person, by default) |
| **VerificationEvent** | Test / inspection / analysis / demonstration record | verifies Requirement; produces evidence for ChecklistItem |

---

## 3. Why Baseline sits between Project and everything technical, why CI↔LogicalSubsystem is many-to-many, and why Milestone belongs to Baseline

This is one structural change from the earlier sketch, and it matters: **CI, Requirement, and LogicalSubsystem are scoped to a Baseline, not directly to a Project.** Without this, "Baseline A's CI-042" and "Baseline B's CI-042" would collide on the same identity. With Baseline as an explicit parent, they're representable as related-but-distinct entities — which is exactly the reconciliation problem that shows up in real modernization/dueling-baseline programs.

A second correction from the initial draft: **CI and LogicalSubsystem relate many-to-many, not one-to-many.** Standard SE architecture practice (SEBoK's Physical Architecture process, consistent with INCOSE SE Handbook and ISO/IEC/IEEE 15288) treats function/logical-to-physical allocation as a matrix, not a tree — a physical architecture's expected output is an allocation matrix of functional/logical elements to physical elements, precisely because one physical element commonly serves multiple functions and vice versa. A CI mapping to more than one LogicalSubsystem is standard, expected architecture, not an anomaly to constrain away. This also means an "over-decomposition" or "integration bloat" signal — a CI carrying unusually many subsystem allocations — is a legitimate, representable finding, not something the model should structurally prevent. If a "primary" or "home" subsystem concept is still wanted for a given CI, express it as an attribute (e.g., a `primarySubsystemId` pointer alongside the full `subsystemIds` set), not by constraining the relationship itself.

A third correction, this one bringing §2's text into line with the diagram below rather than changing the diagram: **Milestone belongs to Baseline, not Project.** The diagram has shown Milestone nested under each Baseline since the initial draft; §2's entity-table text simply hadn't caught up. Real evidence confirms the diagram was right: SE Workbench's Step 3 implementation tracks one Milestone record per SETR event *per baseline lineage*, because Baseline A and Baseline B run independent SETR timelines — the same review (e.g. PDR) can be `Complete` on one baseline and not yet reached on the other, which is only representable if Milestone is Baseline-scoped.

The same evidence also motivated broadening Milestone rather than adding a parallel entity for acquisition-decision gates. Workbench's PKM Migration Step 8 needed to track AAF Milestone A/B/C occurrence (status, actual/planned dates) per baseline lineage — structurally the identical shape as SETR tracking, just a different event vocabulary and no baseline-establishment semantics. Rather than a second entity, this version adds a `milestoneType` discriminator (`SETR` | `AcquisitionGate`) to the one Milestone entity:

- **`SETR` milestones** (SRR, SFR, PDR, CDR, TRR, etc.) — may establish the Baseline they belong to; no `pathway` field; has many ChecklistItems.
- **`AcquisitionGate` milestones** (e.g. MS-A, MS-B, MS-C) — carry a `pathway` field naming the acquisition framework (e.g. `"MCA"`) per the stable-external-id convention in §4: doctrine-defined and identical for every program using that pathway, not per-program content, so no separate entity is needed for the pathway itself. Gate progress within an already-established baseline lineage; does not establish a Baseline; `establishesBaselineId` is left empty for this type.

Keeping this as one entity with a type discriminator — rather than two entities related structurally — mirrors how SE Workbench itself related the two: AAF gates reference SETR milestones only through each acquisition phase's `entryMilestone`/`exitMilestone` fields (structural adjacency), not shared identity. A second app that needs SETR events alone, or acquisition gates alone, or both, gets one entity to model against either way, rather than needing to know both exist to model just one.

```
Program
 └── Project
      ├── Baseline (Legacy / mature, e.g. near Product Baseline)
      │    ├── LogicalSubsystem ⇄ CI (many-to-many) → Requirement
      │    └── Milestone (SETR and/or AcquisitionGate) → ChecklistItem
      └── Baseline (New / in development, e.g. Functional→Allocated)
           ├── LogicalSubsystem ⇄ CI (many-to-many) → Requirement
           └── Milestone (SETR and/or AcquisitionGate) → ChecklistItem
```

A `reconciledFromBaselineId` / `reconciledIntoBaselineId` pair of reference fields is worth reserving on Baseline itself, to represent the eventual merge point without needing a special-case entity — open question, see §5.

---

## 4. ID & Reference Conventions (per Architecture Guidance §9)

- Every entity carries a stable, external ID meaningful outside any single app's database (e.g., `BASELINE-A`, `CI-042`, `REQ-118`), not just an internal auto-increment key.
- All relationships are explicit typed reference fields (`belongsToBaselineId`, `satisfiesRequirementId`, `blocksMilestoneId`), never denormalized text.
- No entity in this model holds content — only structure and references. Real content (what CI-042 actually is, what REQ-118 actually says) lives in a program's PDKM, keyed by these same IDs.
- A stable external id can itself be structural, not program content, when its value is doctrine-defined and identical across every program using it — e.g. `Milestone.pathway = "MCA"`. Apply the §1.1 test from Architecture Guidance (would this value be the same regardless of which program it's for?) to the id's *value*, the same way it applies to any other field.

---

## 5. Open Questions for Next Discussion

1. **Baseline reconciliation** — does "reconciled into" deserve to be a relationship on Baseline (as sketched above), or a distinct event/entity (e.g., a `ReconciliationEvent`) given it's a significant, auditable act with its own evidence trail? **Real-world evidence:** the SE Workbench app's `AbCompatibilityRow` is a working example of this exact shape — each record compares the same interface across two baselines simultaneously (`baselineAState`, `baselineBIntent`), with a `compatibilityStatus`, `riskNote`, and `lastReviewedDate`, and cannot be assigned to a single Baseline without losing meaning. This is a strong signal toward a distinct `ReconciliationEvent`-style entity rather than a simple field on Baseline, though not yet a final decision.
2. **ChecklistItem domain tagging** — the earlier PDR readiness app groups checklist items into 7 SE domains. Should "Domain" be a first-class PKM entity, or just an attribute on ChecklistItem?
3. **Role model for ActionItem assignment** — "assigned to a role, not a person" was stated above as a default; worth confirming this holds across Program Management too (where accountability is often named), not just SE/CM.
4. **Program-level parallel Projects** — the model currently allows multiple Baselines per Project, but should it also explicitly allow a Program to run parallel Projects targeting the *same* eventual system (rather than only nesting parallelism inside one Project's Baselines)? This may matter for very large modernization Programs.

---

*This document is a companion artifact to the Architecture Guidance doc, not a replacement. Apps should continue following the Architecture Guidance for provider abstraction, vendoring, and directory conventions; this PKM model is what §9 of that document anticipates apps eventually consuming.*
