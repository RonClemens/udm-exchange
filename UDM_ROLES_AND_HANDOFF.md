# UDM Effort — Roles & Handoff (v1.2.0)

**Version:** 1.2.0
**Last updated:** 2026-07-29
**Status:** Active
**Supersedes:** `UDM_ROLES_AND_HANDOFF.md` (2026-07-25, unversioned)

**Changelog:**
- **v1.2.0** — Corrected the SE Workbench and `udm-exchange` entity descriptions to document scoped direct-write access as current fact, not aspiration: both coding sessions have had direct git/GitHub write access to `udm-exchange` since 2026-07-25/26 per `UDM_EXCHANGE_ACCESS_PROTOCOL.md`'s own updates on those dates, each scoped to the folders it owns (Workbench: `migration-plans/se-workbench/`, `feedback/se-workbench/`; `udm-exchange` session: `architecture-guidance/`, `pkm/`, and this repo's own process docs). This document's v1.1.0 text still described feedback commits as routing through Ron for relay, which README.md and the Access Protocol had already correctly superseded — found via a design-chat provenance check on 2026-07-29 (SE Workbench feedback commit `225a771`, self-verified and landed without a `udm-exchange`-session or Ron relay step). Revised the "no two of the other three talk directly" line and the standard round-trip's steps 3-4 to match. Confirmed the main-only/no-PR branching convention still holds as observed practice (checked, not assumed) — unchanged.
- **v1.1.0** — Made Ron an explicit fourth entity in the workflow rather than an implicit coordinator; documented the main-branch-only convention for `udm-exchange`; added explicit confirmation points to the round-trip; added the supersession convention for stale documents; closed out the Step 2/content-split open item (resolved 2026-07-26, see SE Workbench's status report).
- **v1.0.0** (2026-07-25) — Initial version, following the move to a dedicated `udm-exchange` repo.

---

## The four entities

| Entity | Role | Repo / access |
|---|---|---|
| **Design chat** (this) | Design, research, document authoring, cross-app architecture decisions | None — produces documents only, no write access anywhere |
| **Ron** | Human coordinator — relays every document between the other three, confirms handoffs, makes or gathers program-level decisions none of the AI sessions should presume | Access to all three repos/sessions directly |
| **`udm-exchange` coding session** | Commits canonical documents (Architecture Guidance, PKM) and this repo's own process docs to `main`; reads feedback and migration-plan documents pushed there by participating apps | `RonClemens/udm-exchange` (public), **main branch only**, scoped to `architecture-guidance/`, `pkm/`, and this repo's own root-level docs |
| **SE Workbench coding session** | Implements against Architecture Guidance / PKM / migration plans; produces feedback and status reports | `RonClemens/MK--710-SSR-SysEng-Story` (private); **also** direct git/GitHub write access to `RonClemens/udm-exchange` (public), scoped to `migration-plans/se-workbench/` and `feedback/se-workbench/` — since 2026-07-25/26, per `UDM_EXCHANGE_ACCESS_PROTOCOL.md` |

Ron is listed as an entity, not just a mechanism, because the workflow depends on Ron doing real things at each handoff — not just passing files through unchanged: confirming a commit landed, deciding things none of the three sessions should decide unilaterally (e.g., the Step 2/content-split sequencing call), and knowing when a document is stale versus current.

**Design chat's documents still route through Ron** — design chat has no write access anywhere, and no way to reach either coding session directly, so this is a hard technical constraint, not a workflow choice. **The two coding sessions do not route routine, within-scope commits through Ron** — each writes directly to the folders it owns in `udm-exchange`, and reads the other's committed files there directly (this is the correction made in v1.2.0; v1.1.0's "everything routes through Ron" was already stale when written, contradicting `UDM_EXCHANGE_ACCESS_PROTOCOL.md`'s 2026-07-26 update). What still goes through Ron regardless of scope: any content change to a canonical document (Architecture Guidance, PKM Entity Model) — those originate with design chat by design — and any cross-cutting or sequencing decision that touches more than one entity's lane. Neither coding session talks to the other in real time; coordination happens by each reading what the other committed, or through Ron when a decision can't be made unilaterally.

---

## Branching convention

**`udm-exchange` uses `main` only — no feature branches, no PRs.** This was an explicit simplification decision (2026-07-26): the round-trip is already serial by construction (one document in flight between any two entities at a time, relayed by one human), so branch-based coordination would add process overhead without solving a problem that exists. Every commit lands directly on `main`. Raw URLs always resolve to the current state of a document — there's no "which branch is this on" question to ask.

If this ever needs to change (e.g., a second human coordinator joins, or two documents are genuinely in flight concurrently), that's worth revisiting explicitly rather than drifting into branches informally.

---

## The standard round-trip

1. **Design chat produces or revises a document** → delivered to Ron as a download.
2. **Ron relays it to the `udm-exchange` coding session** → committed to `main` at the correct path → **Ron confirms the commit landed** (raw URL resolves, content matches) before treating the handoff as complete.
3. **Ron relays the current raw URL to Workbench's session** (or Workbench pulls it directly if it already has the path) → Workbench implements or reviews against it → produces a feedback/status document.
4. **Workbench commits that document directly** under `feedback/se-workbench/` (or `migration-plans/se-workbench/`) on `main` — no `udm-exchange`-session or Ron relay step for the commit itself, per its scoped write access — self-verifying per `UDM_WORKFLOW_PROTOCOL.md` §3.1 before reporting it done. Ron (or any entity) can still independently re-verify the same way.
5. **Ron brings the new raw URL back to design chat** → design chat reads it directly via `web_fetch` → reviews, and either closes the item out or produces a revision → back to step 1.

**Confirmation points are not optional overhead — they're what keeps a serial, single-branch workflow safe.** Since there's no branch isolation and no PR review step, a bad or partial commit would otherwise only surface the next time someone reads the file. Confirming immediately after each commit catches that at the point of failure, not several handoffs later.

---

## Supersession convention

When a document is replaced by a newer version rather than revised in place (e.g., feedback against an old doc version, once the doc itself has moved on), mark the old one explicitly rather than deleting it:

```
**Status:** Superseded by: <path or filename of the current version>
```

Retain it in the repo for history. This already happened naturally with the two stale v0.1.0 feedback docs (`PKM_ENTITY_MODEL_CONFORMANCE.md`, `PKM_MIGRATION_PLAN_FEEDBACK.md`) once the underlying documents moved to v0.2.x — Ron marked both `Superseded by:` on 2026-07-26. That's now the standard pattern going forward, not a one-off fix.

---

## What each entity does and does not do (unchanged from v1.0.0)

### Design chat
- **Does:** research SE/PM/CM standards, draft and revise Architecture Guidance, the PKM model, and per-app migration plans; work through open design questions and tradeoffs raised by feedback from either coding session; read current documents directly via raw URL once Ron confirms a commit.
- **Does not:** have write access to any repository. Every document it produces is delivered as a download for Ron to forward.
- **Does not:** implement code or manage a live filesystem.

### Ron
- **Does:** relay every document in both directions; confirm each commit before the round-trip proceeds; make or gather decisions that would otherwise force one AI session to presume something outside its own repo (e.g., program-level sequencing calls); mark documents superseded when appropriate.
- **Does not:** author design content or code directly as part of this workflow — Ron's role here is coordination and decision-gathering, not authoring.

### `udm-exchange` coding session
- **Does:** commit documents Ron relays into `architecture-guidance/`, `pkm/`, and this repo's own process docs on `main`; read feedback or migration-plan documents Workbench has pushed directly to its own scoped folders; self-verify each commit per `UDM_WORKFLOW_PROTOCOL.md` §3.1; confirm sync when asked.
- **Does not:** originate design decisions or write new architecture/PKM content on its own. Opinions on design route back through Ron to design chat, same as any other feedback.
- **Does not:** touch the Workbench repo, or `udm-exchange`'s `migration-plans/se-workbench/` or `feedback/se-workbench/` folders (Workbench's own scoped lane). Does not use branches other than `main`.

### SE Workbench coding session
- **Does:** implement the app against Architecture Guidance and the PKM Migration Plan; generate conformance/migration/status feedback documents; commit those documents directly to `migration-plans/se-workbench/` and `feedback/se-workbench/` on `udm-exchange`'s `main`, self-verifying per `UDM_WORKFLOW_PROTOCOL.md` §3.1 before reporting done — no relay through Ron or the `udm-exchange` session for this, since 2026-07-25/26 (see `UDM_EXCHANGE_ACCESS_PROTOCOL.md`).
- **Does not:** write to any other folder in `udm-exchange` (`architecture-guidance/`, `pkm/`, this repo's own process docs) — those stay the `udm-exchange` session's lane. Does not use branches other than `main`.

---

## `udm-exchange` repo structure (reference, current as of v1.2.0)

```
udm-exchange/
├── README.md
├── DECISION_RECORD_UDM_EXCHANGE_REPO.md
├── UDM_EXCHANGE_ACCESS_PROTOCOL.md
├── UDM_ROLES_AND_HANDOFF.md               (this document)
├── UDM_WORKFLOW_PROTOCOL.md
├── UDM_DESIGN_CHAT_HANDOUT.md
├── architecture-guidance/
│   └── ARCHITECTURE_GUIDANCE.md
├── pkm/
│   ├── PKM_ENTITY_MODEL.md
│   └── examples/                          (non-canonical: synthetic pseudo-data validation pass)
│       ├── PDKM_EXAMPLE_PSEUDODATA.md
│       └── pdkm-pseudodata-v0.2.0.json
├── migration-plans/
│   └── se-workbench/
│       └── PKM_MIGRATION_PLAN.md
├── feedback/
│   └── se-workbench/
│       ├── PKM_MIGRATION_STATUS_REPORT.md
│       ├── PKM_ENTITY_MODEL_CONFORMANCE.md      (Superseded by: PKM_MIGRATION_STATUS_REPORT.md)
│       └── PKM_MIGRATION_PLAN_FEEDBACK.md       (Superseded by: PKM_MIGRATION_STATUS_REPORT.md)
└── proposals/                              (4 historical RESOLVED + 1 active Draft — see README)
```

Future participating apps get their own `migration-plans/<app-name>/` and `feedback/<app-name>/` folders on `main` — no other structural change needed.

---

## Resolved since v1.0.0

The Step 2 (Baseline enum→entity) / Architecture Guidance content-split sequencing question — the one open item carried over from the prior version of this document — is **closed**. Workbench coordinated both concerns in a single pass on `recoveryProgramGuidance.ts`; all 7 migration plan steps are implemented, verified, and deployed per the 2026-07-26 status report.

## Open item

None currently blocking. See the design chat's active thread for candidate PKM v0.3.0 topics (open questions #1 and #2, role taxonomy, Gap evidence cardinality) and the pending `@domain-placeholder` convention proposal — neither blocks any entity's current work.
