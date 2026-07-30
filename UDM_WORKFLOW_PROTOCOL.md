# UDM Effort — Workflow Protocol: States, Commands, and Handoff Triggers

**Version:** 1.10.0
**Last updated:** 2026-07-30
**Status:** Active
**Companion to:** `UDM_ROLES_AND_HANDOFF.md` v1.2.0 (defines *who* the four entities are; this document defines *how* they trigger each other)

**Changelog:**
- **v1.10.0** — Changed §3.2's `Timestamp:` convention from UTC to EDT, per Ron's direction (2026-07-30). All markdown command batch headers should carry an EDT timestamp going forward, not UTC — this document is what defines the format every entity follows, so the convention itself needed a real revision, not just a habit change on any one entity's side. Existing commands already timestamped in UTC during the transition period aren't errors; they predate this version.
- **v1.9.0** — Added §3.5, **Documentation Drift Reconciliation**: a named convention for handling the case where two-or-more canonical documents describe incompatible mechanics for the same thing, generalized from how this exact situation was actually handled on 2026-07-29 (a design-chat provenance check found `UDM_ROLES_AND_HANDOFF.md` v1.1.0 contradicting both `README.md` and `UDM_EXCHANGE_ACCESS_PROTOCOL.md` on whether Workbench has direct write access to `udm-exchange`). Corrected §5, which asserted "Ron remains the only relay point between the other three entities" — stale in the same way and by the same root cause as the Roles & Handoff line it was restating; see `UDM_ROLES_AND_HANDOFF.md` v1.2.0 for the corrected version. Updated the companion-document reference above to v1.2.0.
- **v1.8.0** — Made SHA-pinned verification symmetric: §3.1 now requires any entity re-verifying a `CONFIRM` — not just the entity that pushed it — to use a SHA-pinned URL before treating a mismatch as confirmed failure. Added directly after a branch-pinned URL returned stale content on design chat's side across many checks in this thread, while the SHA-pinned URL for the same commit was correct on first fetch. Also added §3.4, the cross-reference convention: structural header citations (`Companion to:`, `Supersedes:`, etc.) should carry a branch-pinned raw URL alongside the name/version, distinct from §3.1's SHA-pinned verification links — a citation should always resolve to current truth, not freeze a moment in time. Correction to a prior draft note: the `@domain-placeholder` convention is Architecture Guidance content (already merged there as §10, v1.4.0) — it was mistakenly listed as pending for this document; it isn't, and has no further action here.
- **v1.7.0** — §3.1 now requires SHA-pinned raw URLs, not branch-pinned ones, for self-verification and for any downstream `CONFIRM` check. Root cause: a branch-pinned URL (`.../main/<path>`) can lag the actual commit for a period after a push — observed directly when a Workbench self-verification and a design-chat re-check of the same branch-pinned URL disagreed for over an hour after a real, already-landed commit. A SHA-pinned URL (`.../<commit-sha>/<path>`) is immutable and doesn't have this problem by construction. This removes the failure mode rather than catching it after the fact.
- **v1.6.0** — Added a `Timestamp:` line to the `Source:`/`Target:` header (§3.2), and the `SENT` acknowledgment convention (§3.3): after relaying a batch, Ron posts `SENT` back into the source chat so each chat's own scrollback shows what's actually been relayed versus still pending.
- **v1.5.0** — Added §3.2, the `Source:`/`Target:` header convention for relayed command batches.
- **v1.4.0** — Added §3.1, mandatory self-verification before reporting `CONFIRM`.
- **v1.3.0** — Made §6's stall-check obligation symmetric across all three AI entities.
- **v1.2.0** — Added `NEXT`, a seventh command.
- **v1.1.0** — Added §6, the accountability convention.
- **v1.0.0** — Initial version.

---

## 1. Why a protocol, not just prose

Four entities — Ron, design chat, the `udm-exchange` session, and the Workbench session — coordinate through a human relay with no shared runtime. That's a good fit for a small, explicit **state machine** plus a **fixed command vocabulary**: every handoff becomes an unambiguous verb + object, every document has one of a small number of named states at all times, and "what's the status of X" always has one correct answer instead of a paragraph to reconstruct from memory.

This scales cleanly to future apps beyond Workbench — the states and verbs don't change per app, only the document being moved through them.

---

## 2. The document state machine

Every document in the exchange is, at all times, in exactly one of these states:

```
DRAFTING ──► READY ──► COMMITTED ──► IN_USE ──► FEEDBACK_READY ──► FEEDBACK_COMMITTED ──► IN_REVIEW ──►
  (loops back to DRAFTING for next rev, or → RESOLVED / SUPERSEDED)
```

| State | Meaning | Who holds it |
|---|---|---|
| **DRAFTING** | Being authored or revised | Design chat, or a coding session for its own feedback docs |
| **READY** | Finished, delivered as a download, awaiting relay | Whoever authored it, until Ron picks it up |
| **COMMITTED** | On `udm-exchange` main, Ron has confirmed the raw URL matches | Ron confirms; `udm-exchange` session executes |
| **IN_USE** | A coding session is actively implementing/reviewing against it | The consuming session (e.g. Workbench) |
| **FEEDBACK_READY** | A response document exists, delivered, awaiting relay | The session that produced it |
| **FEEDBACK_COMMITTED** | Feedback is on `main`, confirmed | Ron confirms |
| **IN_REVIEW** | Design chat (or whichever entity owns the source doc) is reading committed feedback | Design chat |
| **RESOLVED** | Loop closed, no further action expected | — terminal |
| **SUPERSEDED** | Replaced by a newer document, retained for history | — terminal |

Any document can also jump straight to **SUPERSEDED** from any state if it's overtaken by events.

---

## 3. The command vocabulary

Seven verbs.

| Command | Form | What it does | State transition |
|---|---|---|---|
| **HANDOFF** | `HANDOFF <doc> → <entity>` | Ron delivers a document to a specific entity and expects action | READY → IN_USE |
| **CONFIRM** | `CONFIRM <path>` | Ron confirms a commit landed and content matches | → COMMITTED or → FEEDBACK_COMMITTED |
| **REVIEW** | `REVIEW <path>` | Ron asks an entity to read and assess a committed document | COMMITTED/FEEDBACK_COMMITTED → IN_REVIEW |
| **CLOSE** | `CLOSE <item>` | Marks a loop finished | → RESOLVED |
| **SUPERSEDE** | `SUPERSEDE <old> → <new>` | Marks a document replaced | → SUPERSEDED |
| **STATUS** | `STATUS` (optionally `STATUS <doc>`) | Asks an entity to report current state | No transition |
| **NEXT** | `NEXT` | Asks: what is the single action, right now, that unblocks whichever actor is currently the holdup? | No transition itself |

A single message from Ron containing one of these seven verbs is enough for any entity to know exactly what's being asked, without re-deriving context from a paragraph.

**`NEXT` vs. `STATUS`:** `STATUS` returns the whole board. `NEXT` returns one line.

### 3.1 Self-verification before reporting a commit

Any entity that pushes a commit must fetch the raw URL itself and confirm the pushed content actually matches what was intended, **before** reporting it as done. Check at minimum: the version number in the header, and any specific section called out as having changed.

**Use the SHA-pinned raw URL, not the branch-pinned one:**

```
Branch-pinned (do not use for verification):
https://raw.githubusercontent.com/RonClemens/udm-exchange/main/<path>

SHA-pinned (use this):
https://raw.githubusercontent.com/RonClemens/udm-exchange/<commit-sha>/<path>
```

A branch-pinned URL can lag the actual commit for a period after a push. A SHA-pinned URL is immutable by construction and doesn't have this failure mode. Get the commit SHA from whatever push/commit response is available and use it for the verification fetch. Once verification passes, `CONFIRM` can still reference the branch-pinned URL as the canonical, human-readable link — the SHA-pinned form is for the verification step itself.

**This requirement is symmetric.** Any entity independently re-verifying someone else's reported commit — most often design chat, re-checking a `udm-exchange` self-verification — must also use a SHA-pinned URL for that re-check, not a branch-pinned one. A mismatch found only via a branch-pinned read is inconclusive, not a confirmed failure, until checked again against the SHA-pinned URL for that specific commit.

### 3.2 `Source:` / `Target:` / `Timestamp:` batch headers (optional, strengthens when present)

```
Source: <entity>
Target: <entity>
Timestamp: <YYYY-MM-DD HH:MM EDT>

<VERB> <object>
```

**Timestamp convention: EDT, not UTC** (changed 2026-07-30, per Ron's direction — see changelog). Every entity's batch headers should use EDT going forward. Commands timestamped in UTC from before this change are historical, not errors.

Multi-target batches are always split into separate headers, one per target. Format: structured markdown, not JSON — nothing parses these programmatically yet.

### 3.3 `SENT` acknowledgment

After Ron actually pastes a batch into its target chat, Ron posts a single-word `SENT` reply back into the **source** chat — the same chat the batch's content originated from, not the destination. This exists because no chat window can see any other chat window's scrollback.

### 3.4 Cross-reference links in structural headers

Structural header lines that name another document — `Companion to:`, `Supersedes:`, and equivalents — should carry the document's branch-pinned raw URL alongside its name and version, not name/version alone. Use the branch-pinned URL here, not the SHA-pinned form from §3.1 — a citation should point at current truth, not freeze a moment in time. Scope: structural header-level citations only, not every filename mention in prose. Backfill existing headers alongside each document's next substantive revision.

### 3.5 Documentation Drift Reconciliation

Canonical and near-canonical documents describe the same mechanics from different angles (roles, access, workflow) — they can drift out of agreement with each other, or with actual practice, without any single change being wrong on its own. This happened on 2026-07-29: `UDM_ROLES_AND_HANDOFF.md` v1.1.0 said Workbench "does not commit anything to `udm-exchange` directly," while `README.md` and `UDM_EXCHANGE_ACCESS_PROTOCOL.md` had already documented scoped direct-write access since 2026-07-25/26 — three canonical-ish documents, two agreeing with each other and with observed commits, one lagging.

**When any entity finds two or more documents describing incompatible mechanics for the same thing:**

1. **Flag it via `STATUS`, don't resolve it unilaterally** — even if it's obvious which document is stale. Cite both (or all) conflicting documents directly, by name and quoted line, not by characterization alone.
2. **The flag routes to Ron for a decision** — which document is authoritative, whether practice has actually diverged from all of them, and whether the fix is a documentation correction or a real process change.
3. **Once decided, the owning session applies the correction** — usually the `udm-exchange` session, since it holds most of this repo's own process docs — with a changelog entry that names what was found and by whom (which entity's check surfaced it), not just what changed. This keeps the drift itself part of the record, the same way a stale cross-reference or a superseded feedback doc is kept visible rather than silently fixed.

This is a discovery-and-repair pattern, not a new command verb — `STATUS` already covers step 1; nothing here changes the seven-verb vocabulary in §3.

---

## 4. Visual status indicator

A generated status board (HTML artifact, regenerated by design chat on request) shows every active document as a node positioned at its current state, color-coded: **Blue** (in motion), **Green** (settled), **Amber** (waiting on a human action), **Slate** (closed). Not live — a snapshot, regenerated on request.

---

## 5. What this doesn't change

This protocol formalizes triggers and status-tracking on top of the round-trip already defined in `UDM_ROLES_AND_HANDOFF.md` v1.2.0. Design chat's documents still route through Ron — it has no write access anywhere. The two coding sessions do not: each has scoped direct-write access to its own folders in `udm-exchange` and commits there without a Ron relay step for the commit itself (see Roles & Handoff v1.2.0 for the corrected model — this line originally said "Ron remains the only relay point between the other three entities," which was already inaccurate when written, per the same drift documented in §3.5).

---

## 6. Accountability convention

Every entity in this state machine can become the blocking node. Applies equally to Ron and to all three AI entities:

- **Design chat** — before responding to a new UDM-related request, checks for anything stalled at `READY`/`FEEDBACK_READY` and names it first. When independently re-verifying a `CONFIRM`, prefers the SHA-pinned raw URL (§3.1), and treats a mismatch on a branch-pinned URL alone as inconclusive rather than confirmed failure until a SHA-pinned check is available.
- **`udm-exchange` session** — before acting on a new commit request, checks whether anything in its own queue is unpushed, and whether the requester's understanding of a document's state matches what's actually on `main`.
- **Workbench session** — before pushing new feedback, checks whether any of its own prior `FEEDBACK_READY` items are still outstanding.
