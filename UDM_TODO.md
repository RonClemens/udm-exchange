# UDM — TODO / Comments

**Version:** 0.3.0
**Last updated:** 2026-07-31
**Status:** Active

**Changelog:**
- **v0.3.0** — T-011 moved: formalized as `Comment`, PKM Entity Model v0.7.0 §2–§3. Per Ron's direction (2026-07-31) that this applies to the entire UDM data structure, not one entity or one app — implemented as a polymorphic entity reusing `Gap`'s existing `foundInEntityType`/`foundInEntityId` pattern, attachable to any current or future PKM entity, or left unattached. One real gap surfaced in formalizing it, carried into PKM §5 item 7: it needs a genuinely user-editable Architecture Guidance UI pattern, which §10.5's read-only "Promises" view doesn't cover.
- **v0.2.0** — Added T-011: Ron's direction that this concept should become a standard in-app feature (user-visible, user-editable TODO/comments per program), not just a design-chat-maintained coordination file. Includes an unvetted sketch of a possible `Comment`/`TodoItem` PKM entity — explicitly not ready to relay as guidance, same plan-before-propose discipline used for `Role` and `RiskItem`.
- **v0.1.0** — Initial version, ten items pre-populated from things flagged across 2026-07-29/30 threads that lacked one clear home.

**Purpose:** A running list of loose ends — ideas, candidate follow-ups, and cross-cutting notes — that don't yet belong in any specific document's own versioned "Open Questions" section, either because they span multiple documents, aren't yet well-formed enough to be a real proposal, or are just easy to lose track of otherwise. Entries here are lightweight by design: no changelog discipline required to add or update one, unlike the canonical documents. When an item here matures into a real decision or proposal, it moves to the relevant document (PKM, Migration Plan, Architecture Guidance, a new proposal) and gets marked `Moved` here, not deleted — so there's a record of where it went.

**Not a substitute for:**
- PKM Entity Model §5 (Open Questions) — structural/entity-modeling questions belong there, not here.
- SE Workbench Migration Plan §15 (Open Questions) — Workbench-specific implementation questions belong there.
- Any proposal's own §6-style open-items section, once a proposal exists.

This document is for everything that doesn't cleanly fit one of those.

---

## How to use this

Add an entry with a short table row. Status values: `Open`, `Parked` (deliberately not being worked, with a reason), `Moved` (promoted to a real document — link it), `Closed` (resolved here directly, doesn't warrant a full document entry).

| ID | Item | Source | Related document(s) | Status | Date added |
|---|---|---|---|---|---|

---

## Current items

| ID | Item | Source | Related document(s) | Status | Date added |
|---|---|---|---|---|---|
| T-001 | Static-deploy caching limitation (GitHub Pages/localStorage backfill only covers entirely-missing collections, never new fields on existing rows) — candidate Architecture Guidance addition, affects any app using §3.1's static-only exception, not just Workbench | Workbench status report v1.9.0 §12 item 8 | Architecture Guidance §3.1 | Open | 2026-07-30 |
| T-002 | Gap's `foundIn` cardinality — current model is one `foundInEntityType`/`foundInEntityId` per Gap, but real data shows a finding can be found in one place while being *referenced by* multiple other mechanisms | Workbench status report §6 item 3 (originally), repeated in later reports | PKM Entity Model (Gap entity) | Open | 2026-07-29 |
| T-003 | Two `RiskItem` interpretive calls not literally specified in PKM v0.6.0 §2–§3, offered for upstream confirmation: (a) `RiskItemStatus`'s four values derived from lifecycle dates, not quoted from spec; (b) treating an `Issue`'s null `likelihood` as 1 for `riskLevel` derivation is Workbench's own reading, not a literal quote. Worth checking a second implementation lands the same way, if/when one exists | Workbench status report v1.9.0 §12 item 7 | PKM Entity Model (RiskItem entity) | Open | 2026-07-30 |
| T-004 | Promises UI major-group taxonomy (11 SE-domain groups) is Workbench's own judgment call, not derived from any PKM/Architecture Guidance document — worth comparing notes if a second app builds a similar view, or if this becomes a shared cross-app pattern | Workbench status report §3 / §12 item 5 | Architecture Guidance §10 (`@domain-placeholder`) | Open | 2026-07-29 |
| T-005 | Whether ISO/IEC/IEEE 24748-4:2026 access is still needed, given the generated SEMP package's direct grounding in DI-SESS-81785B and OSD SEP Outline v4.1 (real, specific, more directly applicable DoD documents) | Design chat, SEMP migration package review | SEMP Generation Proposal §0, §6 item 1 | Open | 2026-07-30 |
| T-006 | Step 11 export discrepancy's root cause never identified — `baselineId`/`projectId` showed null in one export design chat evaluated, but populated in every commit Workbench could check, including a fresh named-commit export. Closed as not-reproducible on Workbench's side; the actual cause (likely in how that export was generated) is still unknown | Design chat / Workbench, joint investigation | Migration Plan §11 (closed) | Open — low priority, not blocking | 2026-07-30 |
| T-007 | Risk Management Board (RMB) — deliberately no PKM entity for now, explicitly provisional pending a dedicated risk-planning session Ron intends to hold | Ron's Q3 answer, SEMP proposal review | PKM Entity Model §5 item 5, SEMP Generation Proposal §3.1 | Parked — pending Ron's session | 2026-07-29 |
| T-008 | Whether a generated SEMP needs any PKM-level audit trail (record of when/from-what-data-snapshot it was generated) | Design chat, SEMP proposal drafting | SEMP Generation Proposal §6 item 5 | Open | 2026-07-29 |
| T-009 | Program-level parallel Projects — should a Program be able to run parallel Projects targeting the same eventual system, not just parallel Baselines inside one Project? | PKM Entity Model, original draft | PKM Entity Model §5 item 4 | Open | (predates this session) |
| T-010 | ChecklistItem `domain` tagging — plain string attribute today; worth first-class `Domain` entity status if real usage across apps clarifies a need | PKM Entity Model, original draft | PKM Entity Model §5 item 2 | Open | (predates this session) |
| T-011 | ~~This document's concept should eventually become a standard **in-app feature**...~~ | Ron, 2026-07-30/31 | **Moved to PKM Entity Model v0.7.0** (`Comment` entity, §2–§3) | Moved | 2026-07-30 |
| T-012 | `Comment` needs a genuinely user-editable Architecture Guidance UI pattern — §10.5's "Promises" view is the closest precedent but is explicitly read-only; real user input/persistence in a live deployment isn't covered by any existing pattern | Design chat, formalizing T-011 | PKM Entity Model §5 item 7; Architecture Guidance §10.5 (new subsection needed) | Open | 2026-07-31 |

---

## T-011 — moved, see PKM Entity Model v0.7.0

Formalized as `Comment` — polymorphic attachment (reusing `Gap`'s existing pattern), lightweight by design (no severity/category/derived score, unlike `Gap`/`RiskItem`), the first entity in this model created by direct end-user input rather than a defined SE process step. Full rationale in PKM Entity Model §3. Not yet relayed to either coding chat for implementation.

---

*Items T-002, T-003, T-004, T-009, and T-010 already have a home in an existing document's own Open Questions section — they're cross-listed here for visibility, not duplicated tracking. If one gets resolved in its home document, update the Status column here to `Moved` with a pointer, rather than leaving it looking still-open in two places.*
