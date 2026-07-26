# UDM Effort — Roles & Handoff

**Originally established:** 2026-07-25
**Updated:** 2026-07-26 — mechanical file exchange no longer routes through Ron; see "What changed" below.

This document establishes the working relationship between the three participants in the UDM effort.

---

## The three participants

| Participant | Role | Repo |
|---|---|---|
| **Design chat** (Claude, claude.ai) | Design, research, document authoring, cross-app architecture decisions | none — produces documents, has no write access anywhere |
| **`udm-exchange` coding session** | Commits exchange documents; reads feedback from participating apps | `RonClemens/udm-exchange` (public) |
| **SE Workbench coding session** | Implements the PKM migration plan in the actual app; sends feedback | `RonClemens/MK--710-SSR-SysEng-Story` (private) |

SE Workbench is the trial app building out and adopting the UDM — the first of what's expected to be several participating apps.

Ron is the human coordinator across all three, but is no longer a relay for every file. See below for exactly what still needs him.

---

## What changed (2026-07-26)

The original version of this document routed every file — in both directions — through Ron. That was more caution than the setup actually needed: both coding sessions have direct git/GitHub write access to `udm-exchange`, each scoped to folders it owns, so the routine mechanics of the exchange don't need a human forwarding files back and forth.

**What's now direct, no Ron required:**
- Workbench committing its own migration-plan updates and feedback documents to `migration-plans/se-workbench/` and `feedback/se-workbench/`.
- The `udm-exchange` session committing documents Ron has already handed over, and reading anything either session has pushed via raw file URLs.
- Either session reading the other's committed documents to inform its own work (still never writing into the other app's folder, or into the other's actual repo).

**What still requires Ron, by design, not by oversight:**
1. **The design chat's documents.** That chat has no git tooling of its own — every document it produces still reaches this repo only via Ron relaying it. This is a technical constraint, not a process choice, and remains open pending whether a write-capable connector ever becomes available to that chat.
2. **Any content change to a canonical document** — `architecture-guidance/ARCHITECTURE_GUIDANCE.md` or `pkm/PKM_ENTITY_MODEL.md`. These originate with the design chat; neither coding session edits or version-bumps them on its own initiative.
3. **Decisions that cut across both repos** — e.g., whether Workbench's Step 2 (Baseline enum → entity) and the Architecture Guidance content-split work run together or in sequence (see open item below). Either session can flag that such a decision is needed; neither resolves it unilaterally.

No two of the three participants communicate directly with each other outside of this — a coding session doesn't message the design chat, and the two coding sessions don't message each other. Exchange happens by reading and writing this shared repo; decisions in the three categories above still go through Ron.

---

## What each participant does and does not do

### Design chat
- **Does:** research SE/PM/CM standards, draft and revise Architecture Guidance, the PKM model, and per-app migration plans; work through open design questions and tradeoffs raised by feedback from either coding session.
- **Does not:** have write access to any repository. Every document it produces is delivered as a download for Ron to forward.
- **Does not:** implement code or manage a live filesystem — it stays in the design/research space intentionally, not as a limitation to work around.

### `udm-exchange` coding session
- **Does:** commit documents the design chat produces into the correct path in `udm-exchange` (see structure below); read documents and feedback pushed there directly by participating apps' sessions (e.g., Workbench); commit its own process-document updates (README, this file, the access protocol, the decision record).
- **Does not:** originate design decisions or write new architecture/PKM content on its own — it's the "hands" for canonical-document commits, not a second design authority. If it has opinions on the design, those route back through Ron to the design chat, same as any other feedback.
- **Does not:** touch the Workbench repo or any other app's repo, or write into another app's `migration-plans/`/`feedback/` folder.

### SE Workbench coding session
- **Does:** implement the app against Architecture Guidance and the PKM Migration Plan; generate conformance/migration feedback documents; commit its own migration-plan and feedback updates directly to `migration-plans/se-workbench/` and `feedback/se-workbench/` in `udm-exchange`, following the feedback-freshness convention in `README.md` (cite the exact version reviewed; archive or mark superseded documents in the same commit that replaces them).
- **Does not:** write to `architecture-guidance/` or `pkm/`, or to any other app's folders.
- **Note:** the `docs/udm-exchange` folder inside the Workbench repo itself (created earlier, before the standalone repo existed) is superseded. Workbench's role going forward references the standalone `udm-exchange` repo, not its own folder.

---

## `udm-exchange` repo structure (reference)

```
udm-exchange/
├── README.md
├── DECISION_RECORD_UDM_EXCHANGE_REPO.md
├── UDM_EXCHANGE_ACCESS_PROTOCOL.md
├── UDM_ROLES_AND_HANDOFF.md
├── architecture-guidance/
│   └── ARCHITECTURE_GUIDANCE.md
├── pkm/
│   └── PKM_ENTITY_MODEL.md
├── migration-plans/
│   └── se-workbench/
│       └── PKM_MIGRATION_PLAN.md
└── feedback/
    └── se-workbench/
        ├── PKM_ENTITY_MODEL_CONFORMANCE.md
        └── PKM_MIGRATION_PLAN_FEEDBACK.md
```

Future participating apps get their own `migration-plans/<app-name>/` and `feedback/<app-name>/` folders — no other structural change needed.

---

## The round-trip, going forward

1. Design chat produces or revises a document → delivered to Ron as a download (unavoidable — no git tooling on that side).
2. Ron forwards it to the `udm-exchange` coding session → committed to the correct path directly.
3. Workbench (or any other app's session) reads the current document via its raw URL, implements against it, and pushes feedback and migration-plan updates to its own folders directly — no relay needed.
4. The `udm-exchange` session reads that feedback directly to stay current; if it or Ron judges the feedback should prompt a design revision, that goes to the design chat via Ron, back to step 1.
5. Anything touching canonical-document content, or a decision that affects more than one app's repo, goes through Ron before either session pushes — see "What changed" above for the exact boundary.

---

## Open item carried over from the last review

**Still unresolved, blocking Workbench's Step 2 (Baseline enum → entity) from starting:** whether Step 2 and Architecture Guidance's pending content-split work (untangling `recoveryProgramGuidance.ts`) run as one coordinated effort or in an explicit sequence. This needs a decision from Ron or Workbench's team — it isn't something either coding session should resolve unilaterally, since it affects both repos' work.
