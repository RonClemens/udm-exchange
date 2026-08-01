# UDM Effort — Handout for a New Design Chat Session

**Prepared:** 2026-07-28, by the `udm-exchange` coding session, at Ron's request, to onboard a fresh design-chat conversation with accurate current state — not a canonical document itself, a briefing snapshot. Where it summarizes a canonical document, treat the raw URL as ground truth if the two ever disagree.

---

## 1. What this effort is

A Unified Data Model (UDM) effort: a shared, public-safe **Process Knowledge Model (PKM)** — a cross-program SE/PM/CM ontology — paired with **Architecture Guidance** that apps are expected to follow, so multiple SE tools can eventually become views over shared data rather than each owning an isolated schema. SE Workbench is the first participating app, acting as the real-world trial for adopting the PKM.

## 2. The four entities and how they coordinate

| Entity | Role | Access |
|---|---|---|
| **Design chat** (you, starting fresh) | Design, research, document authoring, cross-app architecture decisions | No write access anywhere — every document you produce is delivered to Ron as a download |
| **Ron** | Human coordinator — relays documents, confirms handoffs, makes/gathers decisions no AI session should presume | Access to all repos/sessions directly |
| **`udm-exchange` coding session** | Commits canonical documents (Architecture Guidance, PKM) to `main`; reads feedback pushed by participating apps | `RonClemens/udm-exchange` (public), main-branch-only |
| **SE Workbench coding session** | Implements the app against Architecture Guidance/PKM; produces feedback and status reports | `RonClemens/MK--710-SSR-SysEng-Story` (private) |

No two of the other three talk to each other directly — everything routes through Ron. Full detail: `UDM_ROLES_AND_HANDOFF.md` (raw URL below).

## 3. How coordination actually happens — the command protocol

Full detail lives in `UDM_WORKFLOW_PROTOCOL.md` (raw URL below); this is the cheat sheet.

Every document is at all times in one state: `DRAFTING → READY → COMMITTED → IN_USE → FEEDBACK_READY → FEEDBACK_COMMITTED → IN_REVIEW → (loop, or → RESOLVED / SUPERSEDED)`.

Seven commands, always issued by Ron:

| Command | Form | Meaning |
|---|---|---|
| `HANDOFF` | `HANDOFF <doc> → <entity>` | Deliver a document, expect action |
| `CONFIRM` | `CONFIRM <path>` | A commit landed and content matches |
| `REVIEW` | `REVIEW <path>` | Read and assess a committed document |
| `CLOSE` | `CLOSE <item>` | Loop finished |
| `SUPERSEDE` | `SUPERSEDE <old> → <new>` | Document replaced |
| `STATUS` | `STATUS` / `STATUS <doc>` | Report current state (query, no transition) |
| `NEXT` | `NEXT` | The single action that unblocks the current holdup |

Conventions layered on top (all in `UDM_WORKFLOW_PROTOCOL.md`, current version v1.8.0):

- **§3.1 — SHA-pinned self-verification.** Any entity that pushes a commit must fetch the *SHA-pinned* raw URL (`.../<commit-sha>/<path>`, not `.../main/<path>`) to confirm content before reporting done — branch-pinned URLs can lag a push. This requirement is **symmetric**: if you're independently re-checking a `CONFIRM`, use the SHA-pinned URL too, and treat a branch-pinned mismatch alone as inconclusive, not a confirmed failure. (This exact caching lag has bitten this effort twice — once on `raw.githubusercontent.com`, once again on a local git-fetch cache during this handout's own verification. Don't assume "still stale" without checking a second way.)
- **§3.2 — `Source:`/`Target:`/`Timestamp:` headers.** Precede a relayed command batch with who sent it, who it's for, and when — optional, strengthens when present.
- **§3.3 — `SENT` acknowledgment.** After Ron pastes a batch into its target chat, Ron posts `SENT` back into the *source* chat, so each chat's own scrollback shows what's actually been delivered.
- **§3.4 — Cross-reference links.** A structural citation to another document (`Companion to:`, `Supersedes:`) should carry that document's branch-pinned raw URL alongside its name/version, not name/version alone.
- **§6 — Accountability convention, symmetric across all three AI entities.** At the start of any exchange, check for your own stalled items and name them plainly before doing anything else — don't bury a "this is stale" finding after other content.

## 4. Current canonical documents (verified 2026-07-28, all current, no proposals pending)

| Document | Version | Raw URL |
|---|---|---|
| Architecture Guidance | v1.6.0 | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md |
| PKM Entity Model | v0.7.1 (exploratory) | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md |
| SE Workbench Migration Plan | v0.7.0 (draft) | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/migration-plans/se-workbench/PKM_MIGRATION_PLAN.md |
| UDM Workflow Protocol | v1.10.0 | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_WORKFLOW_PROTOCOL.md |
| UDM Roles & Handoff | v1.2.0 | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_ROLES_AND_HANDOFF.md |
| This repo's README (structure, conventions) | — | https://raw.githubusercontent.com/RonClemens/udm-exchange/main/README.md |

**Current feedback on file** (`feedback/se-workbench/`): `PKM_MIGRATION_STATUS_REPORT.md` (v1.2.0 — all 7 migration steps complete, plus the Acquisition Phase Workbench and PDKM Promises UI redesign) and `PROPOSAL_DOMAIN_PLACEHOLDER_CONVENTION_RESPONSE.md` (Workbench's response, already folded into Architecture Guidance v1.4.0 §10). Two older feedback docs are marked `Superseded by:` and retained for history only.

**Proposals folder** (`proposals/`) holds 4 historical proposals, all already merged and marked `RESOLVED` in the README's table — kept for record, not awaiting action.

**No open items are currently blocking anything.**

## 5. SE Workbench app — current real-world state (private repo, for context only — design chat has no access)

The app was recently rebranded from "PDR Reconciliation & Baseline Alignment Workbench" to **SE Workbench**, reflecting its expanded scope: a phase-driven guided navigation spanning the full Major Capability Acquisition lifecycle (Materiel Solution Analysis through Production & Deployment), not just PDR-era reconciliation. All 7 PKM migration steps are implemented and deployed. Recent work grounded its acquisition-phase guidance directly in the actual INCOSE SE Handbook, 5th Edition (citations to section/figure numbers only — no verbatim reproduction, per that document's own copyright terms). This app's own private repo just merged its development branch to `main` cleanly (verified via `git merge-base --is-ancestor` after an earlier merge attempt landed on the wrong branch and was corrected).

## 6. INCOSE SE Handbook, 5th Edition — access note

The Handbook PDF is licensed for the member's **personal use only** — its own front matter prohibits public *or* systematic distribution, so it is not and should not be committed to any repo, public or private. Two things exist instead:
- An **original, paraphrased section-summary** (not a reproduction) lives in the private Workbench repo at `docs/reference/INCOSE_SE_HANDBOOK_5TH_ED_SUMMARY.md` — internal reference only, not accessible to design chat.
- If design chat needs its own reference to the Handbook, the recommended path is uploading a personal-use copy directly to this Project's **Knowledge** section (Claude.ai), not requesting it be added to any repo.

## 7. What to do with this handout

Fetch the raw URLs in §4 directly rather than relying solely on this summary — this document is a snapshot as of 2026-07-28 and will drift as work continues. Per §6 of the Workflow Protocol, start by checking whether anything is stalled (this handout reports none as of preparation time) before taking on new work.
