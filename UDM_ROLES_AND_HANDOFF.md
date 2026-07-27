# UDM Effort — Roles & Handoff (v1.1.0)

**Version:** 1.1.0
**Last updated:** 2026-07-26
**Status:** Active
**Supersedes:** `UDM_ROLES_AND_HANDOFF.md` (2026-07-25, unversioned)

**Changelog:**
- **v1.1.0** — Made Ron an explicit fourth entity in the workflow rather than an implicit coordinator; documented the main-branch-only convention for `udm-exchange`; added explicit confirmation points to the round-trip; added the supersession convention for stale documents; closed out the Step 2/content-split open item (resolved 2026-07-26, see SE Workbench's status report).
- **v1.0.0** (2026-07-25) — Initial version, following the move to a dedicated `udm-exchange` repo.

---

## The four entities

| Entity | Role | Repo / access |
|---|---|---|
| **Design chat** (this) | Design, research, document authoring, cross-app architecture decisions | None — produces documents only, no write access anywhere |
| **Ron** | Human coordinator — relays every document between the other three, confirms handoffs, makes or gathers program-level decisions none of the AI sessions should presume | Access to all three repos/sessions directly |
| **`udm-exchange` coding session** | Commits exchange documents to `main`; reads feedback pushed there by participating apps | `RonClemens/udm-exchange` (public), **main branch only** |
| **SE Workbench coding session** | Implements against Architecture Guidance / PKM / migration plans; produces feedback and status reports | `RonClemens/MK--710-SSR-SysEng-Story` (private) |

Ron is listed as an entity, not just a mechanism, because the workflow depends on Ron doing real things at each handoff — not just passing files through unchanged: confirming a commit landed, deciding things none of the three sessions should decide unilaterally (e.g., the Step 2/content-split sequencing call), and knowing when a document is stale versus current.

No two of the other three talk to each other directly. Everything routes through Ron.

---

## Branching convention

**`udm-exchange` uses `main` only — no feature branches, no PRs.** This was an explicit simplification decision (2026-07-26): the round-trip is already serial by construction (one document in flight between any two entities at a time, relayed by one human), so branch-based coordination would add process overhead without solving a problem that exists. Every commit lands directly on `main`. Raw URLs always resolve to the current state of a document — there's no "which branch is this on" question to ask.

If this ever needs to change (e.g., a second human coordinator joins, or two documents are genuinely in flight concurrently), that's worth revisiting explicitly rather than drifting into branches informally.

---

## The standard round-trip

1. **Design chat produces or revises a document** → delivered to Ron as a download.
2. **Ron relays it to the `udm-exchange` coding session** → committed to `main` at the correct path → **Ron confirms the commit landed** (raw URL resolves, content matches) before treating the handoff as complete.
3. **Ron relays the current raw URL to Workbench's session** (or Workbench pulls it directly if it already has the path) → Workbench implements or reviews against it → produces a feedback/status document.
4. **Ron relays Workbench's document to the `udm-exchange` session** → committed under `feedback/<app-name>/` on `main` → **Ron confirms the commit landed**, same as step 2.
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
- **Does:** commit documents Ron relays into the correct path on `main`; read feedback or migration-plan documents pushed there by participating apps' sessions; confirm sync when asked.
- **Does not:** originate design decisions or write new architecture/PKM content on its own. Opinions on design route back through Ron to design chat, same as any other feedback.
- **Does not:** touch the Workbench repo or any other app's repo. Does not use branches other than `main`.

### SE Workbench coding session
- **Does:** implement the app against Architecture Guidance and the PKM Migration Plan; generate conformance/migration/status feedback documents.
- **Does not:** commit anything to `udm-exchange` directly — feedback documents get delivered to Ron for relay into `udm-exchange`'s `feedback/se-workbench/` folder via the `udm-exchange` session.

---

## `udm-exchange` repo structure (unchanged, reference)

```
udm-exchange/
├── README.md
├── DECISION_RECORD_UDM_EXCHANGE_REPO.md
├── UDM_EXCHANGE_ACCESS_PROTOCOL.md
├── UDM_ROLES_AND_HANDOFF.md               (this document)
├── architecture-guidance/
│   └── ARCHITECTURE_GUIDANCE.md
├── pkm/
│   └── PKM_ENTITY_MODEL.md
├── migration-plans/
│   └── se-workbench/
│       └── PKM_MIGRATION_PLAN.md
└── feedback/
    └── se-workbench/
        ├── PKM_MIGRATION_STATUS_REPORT.md
        ├── PKM_ENTITY_MODEL_CONFORMANCE.md      (Superseded by: PKM_MIGRATION_STATUS_REPORT.md)
        └── PKM_MIGRATION_PLAN_FEEDBACK.md       (Superseded by: PKM_MIGRATION_STATUS_REPORT.md)
```

Future participating apps get their own `migration-plans/<app-name>/` and `feedback/<app-name>/` folders on `main` — no other structural change needed.

---

## Resolved since v1.0.0

The Step 2 (Baseline enum→entity) / Architecture Guidance content-split sequencing question — the one open item carried over from the prior version of this document — is **closed**. Workbench coordinated both concerns in a single pass on `recoveryProgramGuidance.ts`; all 7 migration plan steps are implemented, verified, and deployed per the 2026-07-26 status report.

## Open item

None currently blocking. See the design chat's active thread for candidate PKM v0.3.0 topics (open questions #1 and #2, role taxonomy, Gap evidence cardinality) and the pending `@domain-placeholder` convention proposal — neither blocks any entity's current work.
