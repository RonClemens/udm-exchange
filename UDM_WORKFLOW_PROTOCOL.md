# UDM Effort — Workflow Protocol: States, Commands, and Handoff Triggers

**Version:** 1.3.0
**Last updated:** 2026-07-27
**Status:** Active
**Companion to:** `UDM_ROLES_AND_HANDOFF.md` v1.1.0 (defines *who* the four entities are; this document defines *how* they trigger each other)

**Changelog:**
- **v1.3.0** — Made §6's stall-check obligation symmetric across all three AI entities, not just design chat. Each entity now has the same concrete, written instruction rather than the principle applying by rule to one and by habit to the others.
- **v1.2.0** — Added `NEXT`, a seventh command: unlike `STATUS` (full picture), `NEXT` returns exactly one concrete action to unblock whichever actor is currently the holdup — added because Ron needed a trigger that resolves to a single instruction, not a board to interpret.
- **v1.1.0** — Added §6, the accountability convention, at Ron's explicit request: every entity, including Ron, can be the blocking node in this state machine, and every entity should say so plainly when it is.
- **v1.0.0** — Initial version.

---

## 1. Why a protocol, not just prose

Four entities — Ron, design chat, the `udm-exchange` session, and the Workbench session — coordinate through a human relay with no shared runtime. That's a good fit for a small, explicit **state machine** plus a **fixed command vocabulary**: every handoff becomes an unambiguous verb + object, every document has one of a small number of named states at all times, and "what's the status of X" always has one correct answer instead of a paragraph to reconstruct from memory.

This scales cleanly to future apps beyond Workbench — the states and verbs don't change per app, only the document being moved through them.

---

## 2. The document state machine

Every document in the exchange (Architecture Guidance, PKM model, a migration plan, a piece of feedback, this document) is, at all times, in exactly one of these states:

```
DRAFTING ──► READY ──► COMMITTED ──► IN_USE ──► FEEDBACK_READY ──► FEEDBACK_COMMITTED ──► IN_REVIEW ──┐
    ▲                                                                                                   │
    └───────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                                                                     (loops back to
                                                                                                 DRAFTING for next rev,
                                                                                                  or → RESOLVED / SUPERSEDED)
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
| **RESOLVED** | Loop closed, no further action expected | — terminal, until a new revision starts |
| **SUPERSEDED** | Replaced by a newer document, retained for history | — terminal |

Any document can also jump straight to **SUPERSEDED** from any state if it's overtaken by events (e.g., the two stale v0.1.0 feedback docs).

---

## 3. The command vocabulary

Seven verbs. Ron issues these (since Ron is the only entity in contact with all others); each entity recognizes them as unambiguous triggers rather than needing to infer intent from prose.

| Command | Form | What it does | State transition |
|---|---|---|---|
| **HANDOFF** | `HANDOFF <doc> → <entity>` | Ron delivers a document to a specific entity and expects action | READY → IN_USE (or triggers DRAFTING if it's a request for a new doc) |
| **CONFIRM** | `CONFIRM <path>` | Ron confirms a commit landed and content matches | → COMMITTED or → FEEDBACK_COMMITTED |
| **REVIEW** | `REVIEW <path>` | Ron asks an entity to read and assess a committed document | COMMITTED/FEEDBACK_COMMITTED → IN_REVIEW |
| **CLOSE** | `CLOSE <item>` | Marks a loop finished, no further action expected | → RESOLVED |
| **SUPERSEDE** | `SUPERSEDE <old> → <new>` | Marks a document replaced | → SUPERSEDED |
| **STATUS** | `STATUS` (optionally `STATUS <doc>`) | Asks an entity to report current state of everything active in its scope, or one specific item | No transition — a query, not a trigger |
| **NEXT** | `NEXT` | Asks: what is the single action, right now, that unblocks whichever actor is currently the holdup? Answered as one instruction — who does what, to what, sent where — never a status dump | No transition itself; the answer names the transition that's waiting to happen |

**Example, mapped to a real exchange from this session:**

```
Ron → design chat:  HANDOFF domain-placeholder-proposal → udm-exchange
  (design chat already delivered it as READY; this moves it toward COMMITTED)

Ron → udm-exchange session: [relays the file, session commits it]

Ron → design chat: CONFIRM feedback/architecture-guidance/PROPOSAL_DOMAIN_PLACEHOLDER_CONVENTION.md
  (design chat now treats it as COMMITTED, can fetch and reference by raw URL)
```

A single message from Ron containing one of these seven verbs is enough for any entity to know exactly what's being asked, without re-deriving context from a paragraph.

**`NEXT` vs. `STATUS`:** `STATUS` returns the whole board — useful for orientation. `NEXT` returns one line — useful mid-work, when the only thing wanted is "what do I do right this second to stop being the holdup."

---

## 4. Visual status indicator

A generated status board (HTML artifact, regenerated by design chat on request) shows every active document as a node positioned at its current state, color-coded by state family:

- **Blue** — in motion (DRAFTING, IN_USE, IN_REVIEW)
- **Green** — settled (COMMITTED, FEEDBACK_COMMITTED, RESOLVED)
- **Amber** — waiting on a human action (READY, FEEDBACK_READY)
- **Slate** — closed (SUPERSEDED)

This isn't live — there's no shared runtime across the three separate chat sessions to poll. It's a **snapshot**, regenerated whenever Ron issues `STATUS` or after a batch of handoffs, reflecting the state as of the last confirmed update. Treat it as a shared mental model artifact, not a dashboard with autonomous data.

---

## 5. What this doesn't change

This protocol formalizes triggers and status-tracking on top of the round-trip already defined in `UDM_ROLES_AND_HANDOFF.md` v1.1.0 — it doesn't change who does what, or the main-branch-only convention. Ron remains the only relay point between the other three entities.

---

## 6. Accountability convention

Every entity in this state machine can become the blocking node — a coding session that hasn't picked up a `HANDOFF`, design chat sitting on a revision, or Ron holding documents at `READY` waiting to be relayed. The state machine only works if being the bottleneck is named plainly and promptly by whichever entity notices it, rather than absorbed quietly. This applies equally to Ron and to all three AI entities — and is written the same concrete way for each of them, not just one:

**At the start of any UDM-related exchange, each AI entity checks for its own stalled items before doing anything else, and names them plainly if found — not as a footnote, not folded into other work.** "Stalled" means: anything in that entity's own scope sitting at `READY` or `FEEDBACK_READY` for more than one exchange with no `HANDOFF` or `CONFIRM` moving it forward.

Concretely, per entity:

- **Design chat** — before responding to a new UDM-related request, checks the status board for anything stalled at `READY`/`FEEDBACK_READY` and names the count and which items first.
- **`udm-exchange` session** — before acting on a new commit request, checks (a) whether anything in its own queue is unpushed, and (b) whether the requester's understanding of a document's current state matches what's actually on `main` — e.g. flags it if someone is treating a document as still `READY` when it's already `COMMITTED` or further along.
- **Workbench session** — before pushing new feedback or migration-plan updates, checks whether any of its own prior `FEEDBACK_READY` items — produced but not yet confirmed committed — are still outstanding, and names them.

No new command and no new state are needed for this — it's a discipline applied consistently, the same way the state machine already tracks documents, now applied to whichever entity is currently the holdup.
