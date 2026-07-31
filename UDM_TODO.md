# UDM — TODO / Comments

**Version:** 0.2.0
**Last updated:** 2026-07-30
**Status:** Active

**Changelog:**
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
| T-011 | This document's concept should eventually become a standard **in-app feature**, not just a design-chat-maintained file — each webapp should display TODO/comment items and let the end **user** view and add their own, per their program's real data | Ron, 2026-07-30 | PKM Entity Model (new entity, not yet proposed); Architecture Guidance §10.5 (Promises view is the closest existing precedent — read-only; this would need to be user-editable, which is new) | Open — not yet scoped as a real proposal | 2026-07-30 |

---

## T-011 — sketch, not a proposal yet

A rough shape, offered so this doesn't sit as a bare one-line idea — **not vetted, not grounded, not ready to relay as guidance**, same "sketch before proposal" discipline used for `Role` and `RiskItem` before either was formalized:

- Likely a new PKM entity (working name `Comment` or `TodoItem`): `id`, `projectId`, polymorphic `entityType`/`entityId` (optional — a comment can attach to any existing record, *or* stand alone unattached, the way a repo-level TODO does), `text` (real PDKM content — user-authored, not structural), `status` (`Open`/`Resolved`), `createdByRoleId` (reusing `Role`, same as `RiskItem.ownerRoleId`), `createdDate`, `resolvedDate`.
- Architecture Guidance precedent: closest existing pattern is §10.5's "Promises" view — but that's explicitly read-only, framed as a promissory note about synthetic data. This would need to be genuinely user-editable in a live deployment, which §10.5 doesn't cover and would need new guidance for (real user input, real persistence, not a demo/placeholder concept).
- Real open questions before this is proposal-ready: does a comment attach to a specific entity by default, or start unattached? Does `Comment`/`TodoItem` relate to `Gap`/`RiskItem` at all, or is it deliberately lighter-weight and unstructured by design (the whole point being *not* to require the rigor those two entities have)? Should this replace or complement `ActionItem` for informal, non-remediation notes?

**Not being relayed to either coding chat yet.** This needs the same plan-then-propose treatment as the SEMP-generation work before any entity design or Architecture Guidance addition goes out.

---

*Items T-002, T-003, T-004, T-009, and T-010 already have a home in an existing document's own Open Questions section — they're cross-listed here for visibility, not duplicated tracking. If one gets resolved in its home document, update the Status column here to `Moved` with a pointer, rather than leaving it looking still-open in two places.*
