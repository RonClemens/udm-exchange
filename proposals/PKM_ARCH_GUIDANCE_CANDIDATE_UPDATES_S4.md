# PKM / Architecture Guidance — Candidate Updates from S4 SEMP Interview (§2.1)

**Status:** DRAFT — candidate content for review, not yet a version bump. Design chat has no repo write access; this is a document for Ron to relay to the udm-exchange coding session.
**Source:** S4 SEMP Interview, §2.1 Requirements Development (DI-SESS-81785B / DoD SEP Outline v4.1 §2.1), Q1–Q9.
**Version note:** Reconciled 2026-08-03 against SHA-pinned fetch of commit `543bf9d` — confirmed current: PKM Entity Model v0.7.3, Architecture Guidance v1.7.1. Candidates 1 and 2 below were revised after reading the actual v0.7.3 model (initial draft was written before this reconciliation and has been corrected in place, not left stale).
**Content-boundary check applied:** all items below pass the "same regardless of program" test. No program names, real requirement text, or CUI-adjacent specifics are included. Ron's verbatim answers (source material) live only in the companion transcript file, which is explicitly marked PDKM-adjacent/non-canonical.

---

## Candidate 1 — Generic `Lifecycle` construct (cross-entity)

**Problem:** Requirement review/approval was found to follow a 4-stage pattern (Draft → Released → Accepted → Approved), and this is not specific to `Requirement` — it's a pattern any documented management/technical data item should carry (per direct program input: "every documented management and technical detail should have some level of a lifecycle").

**Validated against current v0.7.3 state (2026-08-03):** stronger evidence for this than expected. Three entities already carry independently-invented status vocabularies — `Comment` (`Open`/`Resolved`), `ReconciliationEvent` (`Proposed`/`In Progress`/`Complete`), `RiskItem` (own `status` field) — exactly the per-entity reinvention this candidate would consolidate.

**Scope correction, found on the same review:** two of those three should stay excluded, not folded in.
- `Comment` (v0.7.0) is explicitly designed to have **no process gate** — "no severity, no mitigation strategy, no derived risk level... Comment exists specifically not to require any of it." Imposing a 4-stage review lifecycle on it would directly contradict its own stated design intent. Leave `Comment.status` as-is.
- `ReconciliationEvent`'s 3-state model (`Proposed`/`In Progress`/`Complete`) tracks a different kind of act — a reconciliation in progress, not a document under review — and doesn't map cleanly onto Draft→Released→Accepted→Approved. Recommend leaving it as its own status enum rather than forcing a fit.

**Proposal:** Add a shared `Lifecycle` (or `ReviewState`) construct — likely a mixin/shared value object — with states:
- `Draft` — item is in development/co-production
- `Released` — published for formal review (ISO/IEC/IEEE 15288:2023 §6.4.2 confirms this corresponds to the standard's own CM-baseline action for stakeholder needs, stakeholder requirements, and operational concept artefacts)
- `Accepted` — formal review passed
- `Approved` — final sign-off

**Applicable entities (revised):** `Requirement`, `RiskItem`, `Gap`, `ActionItem`, `ChecklistItem` — gated content entities only. **Explicitly not** `Comment` or `ReconciliationEvent`, for the reasons above. Each stage transition binds to the existing `Role` entity (v0.5.0) — no new role construct needed (see Candidate 2, revised).

**Open question for PKM team:** mixin/trait applied per-entity vs. a standalone `LifecycleState` entity referenced by FK from each lifecycle-bearing entity. Recommend the latter for auditability (supports many:1 history per Candidate 3's attribute table) — consistent with the derived-not-stored pattern already used for Baseline reconciliation status and Acquisition Phase.

---

## Candidate 2 — Requirement review as a role-chain, not a single author field

**Problem:** A single `authored_by`-style owner field on `Requirement` would not reflect actual practice.

**Revised finding:** `Role` already exists as a first-class PKM entity (v0.5.0), scoped to Project, with `isDefault` distinguishing standard taxonomy from program-added roles. No new construct is needed — this candidate is fully satisfied by binding each `LifecycleState` transition (Candidate 1) to the existing `Role` entity via `roleId`, the same pattern `ActionItem.assignedRoleId` and `RiskItem.ownerRoleId` already use.

**Proposal:** Model requirement authorship/review as a sequence of `Role`-bound Lifecycle transitions:
- Draft stage → co-producing role(s) (may be multiple, e.g., customer + program manager + chief engineer)
- Released stage → quality-validation role (e.g., Lead Systems Engineer performing a "well-formed requirement" check)
- Accepted stage → validating role(s)
- Approved stage → approving role (often external/customer-side)

No separate entity or field beyond the `LifecycleState.roleId` reference from Candidate 1.

---

## Candidate 3 — Expanded Requirement attribute set, with explicit cardinality

**Problem:** Attribute list needs to be broader than a minimal set, and several attributes are not 1:1 with a Requirement — this needs to be explicit in the schema, not implied.

**Proposal — attribute set with cardinality (relative to one Requirement instance):**

| Attribute | Cardinality | Note |
|---|---|---|
| Unique Identifier | 1:1, immutable | |
| Requirement text/statement | 1:1 | |
| Source | many:1 | |
| Rationale | 1:1 | human-readable summary; see Candidate 5 for explicit derivation |
| Status/Lifecycle state | 1:1 current / many:1 history | see Candidate 1 |
| Priority | 1:1 | |
| Stability/Volatility | 1:1 | change-frequency signal, distinct from Status |
| Owner | 1:1 (co-ownership is the exception) | |
| Safety related (flag) | 1:1 | |
| Safety criticality | 1:1 | |
| Verification method | many:1 | test+analysis+inspection+demo can co-apply |
| Verification level (element/subsystem/system) | many:1 | distinct from verification method |
| Traceability — parent (vertical) | many:many | child may derive from >1 parent |
| Traceability — horizontal (peer/sibling) | many:many | |
| Allocation (to architecture element/CI) | many:many | |
| Type (functional/performance/interface/etc.) | 1:1 | |
| Requirement domain (product vs. process/manufacturing) | 1:1 | see Candidate 6 |
| SWCI (Software Criticality Index) | 1:1 | |
| Version/Revision | many:1 (history) | |
| TBD/TBR flag | 1:1 | |
| Associated risk (→ `RiskItem`) | many:many | |
| Applicable standard/regulation reference | many:1 | |
| Change history/audit trail | many:1 | |

---

## Candidate 4 — Tool-agnostic traceability interchange schema

**Problem:** Traceability (vertical + horizontal) is currently tool-native across a heterogeneous toolchain (Excel/Word, DOORS NG, Aras Innovator, Codebeamer, Cameo — the specific set is PDKM/program-specific; the *category* of tool types is PKM-safe).

**Finding:** Programs already extract/assemble CSV or JSON traceability chains across these heterogeneous tools when needed — confirming tool-agnostic interchange is already an established practice, not a new burden.

**Proposal:** Architecture Guidance should define a canonical CSV/JSON export schema for vertical and horizontal traceability chains (Req ID ↔ parent Req ID(s); Req ID ↔ sibling Req ID(s) within a spec), independent of source RM tool. This directly supports the UDM's core mission — letting non-CUI structure interface with CUI program data without requiring a single tool standard across programs.

**Also recommend:** an open, extensible `tool_category` enum (spreadsheet/document-based; dedicated RM platform; MBSE/architecture tool) rather than a closed list of named tools, since specific tool choice is PDKM instance data and new tools will keep appearing.

---

## Candidate 5 — `DerivationTrace`, distinct from `Rationale`

**Problem:** A flat `Rationale` text attribute is a summary; it does not give a customer an explicit, reviewable derivation chain. Direct program finding: "the customer cannot adequately review the logic behind reqt derivation and validation — Rationale attempts to help here, but is only supportive, not explicit."

**Proposal:** Add a `DerivationTrace` construct — an ordered sequence of why-question/answer pairs or premise→conclusion links — as a richer, separate, explicit backing artifact. `Rationale` remains the human-readable summary; `DerivationTrace` is the reviewable derivation chain underneath it. Grounds directly in the iterative "why" questioning practice (Systems + Design Engineers jointly testing requirement necessity — a documented adaptation of 5-Whys/root-cause-analysis discipline, not a named SE-standard technique).

---

## Candidate 6 — `requirement_domain` classifier (product vs. process/manufacturing)

**Problem:** On complex programs, requirements scope extends beyond product requirements into Manufacturing/Build Process Specifications — these often have different owners, review chains, and verification approaches than product requirements.

**Proposal:** Add a `requirement_domain` classifier (e.g., `product` | `process/manufacturing`, extensible) to distinguish these at the schema level rather than relying on `Type` alone. Confirmed consistent with ISO/IEC/IEEE 15288:2023's recursive-application principle — its life cycle processes "can be applied iteratively and concurrently to a system and recursively to the system elements," which is exactly the system→subsystem→component scope pattern the program described.

---

## Grounding sources referenced across these candidates
- ISO/IEC/IEEE 15288:2023(E), §6.4.2 (Stakeholder Needs and Requirements Definition), §6.4.3 (System Requirements Definition) — project knowledge base copy
- SEBoK, "System Requirements Definition" / requirements-management attribute literature (rationale, status, priority, stability, owner)
- INCOSE SE Handbook 5th Edition, §2.3.5.2–2.3.5.3 — project knowledge base copy (member-personal-use only; not for repo commit)
- DI-SESS-81785B / DoD SEP Outline v4.1 (see prior S4 research report for full grounding)
