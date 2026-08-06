# PKM / Architecture Guidance — Candidate Updates from S4 SEMP Interview (§2.2)

**Status:** DRAFT — candidate content for review, not yet a version bump. Design chat has no repo write access; this is a document for Ron to relay to the udm-exchange coding session.
**Source:** S4 SEMP Interview, §2.2 Architectures and Interface Control (DI-SESS-81785B / DoD SEP Outline v4.1 §2.2), Q1–Q9.
**Version baseline:** Reconciled against SHA-pinned commit `543bf9d` (PKM v0.7.3, Architecture Guidance v1.7.1) — same baseline as the §2.1 candidates already committed (`proposals/`, commit `eaf0f0a`).
**Content-boundary check applied:** all items below pass the "same regardless of program" test. No program names, real requirement text, or CUI-adjacent specifics included.
**Carried-forward open item (still unresolved as of this batch):** udm-exchange has not yet confirmed whether `Gap`, `ActionItem`, or `ChecklistItem` carry existing status fields that would conflict with the `LifecycleState` entity from the §2.1 batch. That confirmation is still needed before *either* batch (§2.1 or §2.2) is HANDOFF-ready for a version bump.

---

## Candidate 7 — `Interface` entity

**Problem:** Interfaces (internal and external) need to be tracked as first-class PKM data, not just implied by architecture models.

**Proposal:** New `Interface` entity:
- `scope`: `internal` | `external` (Q4) — single model with a scope attribute, not two separate entity types.
- `trackingMethod`: `model-embedded (MBx)` | `document/register (DBx)` (Q6/Q7) — records which upstream method produced the interface's definition; ties to the program's architecture-method choice (Q3) rather than being chosen independently per interface.
- `irsReference`: reference to the Interface Requirement Specification — a **design-input** artifact (Q9 elaboration; see Candidate 8).
- `icdReference`: reference to the Interface Control Document — a **design-output** artifact, the formalized/realized definition (Q9 elaboration).
- `LifecycleState`-bearing (extends the §2.1 Candidate 1 construct — add `Interface` to its closed-union applicable-entity list): Draft (interface identified in model/register) → Released (ICD published, at the preliminary design/architecture baseline per Q5) → Accepted → Approved.

**Note on revision history:** an earlier version of this candidate (from Q7, before the Q9 exchange) proposed a single `icdReference` field. Q9's elaboration on Design Inputs vs. Design Outputs showed this was incomplete — an interface produces two distinct artifacts across its lifecycle, not one. Revised accordingly rather than left as originally drafted.

---

## Candidate 8 — `artifactRole` classifier (Design Input / Design Output)

**Problem:** Q9 surfaced a fundamental, cross-cutting distinction not yet captured anywhere in PKM: **Design Inputs** (e.g., Requirements, IRS) vs. **Design Outputs** (e.g., TDP, ICD, physical products). This is a different axis from §2.1 Candidate 6's `requirement_domain` (product vs. process/manufacturing) — orthogonal, not overlapping.

**Proposal:** Add an `artifactRole` enum (`design_input` | `design_output`) applicable across the `Requirement` and `Interface` entity families (and potentially others as evidence accumulates — same evidence-before-generalization discipline used elsewhere in this document). `Requirement` and `Interface.irsReference` are `design_input`; `Interface.icdReference` and the new `TDP` entity (Candidate 9) are `design_output`.

**Open question for PKM team:** should this be a field on each applicable entity, or a property of the *entity type itself* (i.e., `Requirement` is always `design_input`, no per-instance field needed) — likely the latter, since the role appears to be a property of the artifact type, not something that varies per instance. Flagging rather than presuming.

---

## Candidate 9 — `TDP` (Technical Data Package) entity

**Problem:** Q9 raised "the overall challenge is establishing the right level of engineering documentation for PDR, then for CDR SETR events" — an unresolved maturity-gating question. Subsystem-level architecture was described as deriving IRS/ICD, which further decompose to CI (HW/SW) level with their own requirement specifications and TDPs.

**Grounding:** MIL-STD-31000C already defines exactly this maturity concept — a TDP consists of a **Level** (Conceptual, Developmental, Product) and a **Type** (2D/3D), with TDP elements (engineering design data, specifications, software documentation, QAPs, etc.). This is a real, existing standard — not something to invent. TDP Levels map naturally onto SETR review gates: Conceptual ≈ early/SRR-adjacent, Developmental ≈ PDR/prototype, Product ≈ CDR/production, though this mapping is design chat's inference, not something MIL-STD-31000C states explicitly, and should be validated against actual program SETR practice before being treated as fixed.

**Proposal:** New `TDP` (or `TechnicalDataPackage`) entity:
- `ciId`: FK to `CI` — TDPs are scoped to Configuration Items, consistent with `Requirement.satisfiedBy → CI` already establishing `CI` as the right anchor point for downstream design artifacts.
- `level`: `conceptual` | `developmental` | `product` (per MIL-STD-31000C, closed union — this is a fixed, standards-defined set, not open-ended).
- `type`: `2D` | `3D` (per MIL-STD-31000C).
- `artifactRole`: `design_output` (Candidate 8) — TDPs are always design outputs, no per-instance variation expected.
- `LifecycleState`-bearing (extends §2.1 Candidate 1 — add `TDP` to its applicable-entity list), with stage transitions tied to SETR-event `Milestone`s specifically (not a generic date), directly resolving the Q5/Q9 maturity-gating question.

**Open question for PKM team:** does `TDP.level` transition over the same record's lifetime (one `TDP` progressing Conceptual→Developmental→Product), or are these three distinct `TDP` records per `CI` (one per level, superseding the prior)? MIL-STD-31000C's own language ("TDP levels provide for a natural progression of a design from its inception to production") suggests progression of a single conceptual TDP, but the practical CM question (is a Conceptual-level TDP retained after a Product-level one exists, or superseded) needs a program-practice answer before this is settled — a good candidate for a future interview question in a later SEMP section (§3.2.10 Configuration and Change Management) rather than resolved here on inference alone.

---

## Related refinement — `tool_category` (§2.1 Candidate 4)

Q6 (single Cameo/SysML tool spanning both Requirements and Architecture) suggests `tool_category` needs an `entity_scope` field — which PKM entities a given tool instance actually covers — rather than assuming a 1:1 tool-to-artifact-type mapping. Not a new candidate number, an amendment to the existing one.

## Grounding sources referenced
- ISO/IEC/IEEE 15288:2023(E), §6.4.4 (System Architecture Definition), interface-management thread across §6.4.4/§6.4.5 — project knowledge base copy
- INCOSE SE Handbook 5th Edition, §3.2.4 (Interface Management), §1.3.2 (Emergence) — project knowledge base copy (member-personal-use only; not for repo commit)
- MIL-STD-31000C (Technical Data Packages) — project knowledge base copy
- DoD RIO Management Guide (Dec 2023), §4 (Management of Cross-Program Risks) — project knowledge base copy
