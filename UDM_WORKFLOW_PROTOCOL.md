# UDM Effort — Workflow Protocol: States, Commands, and Handoff Triggers

**Version:** 1.7.0
**Last updated:** 2026-07-27
**Status:** Active
**Companion to:** `UDM_ROLES_AND_HANDOFF.md` v1.1.0 (defines *who* the four entities are; this document defines *how* they trigger each other)

**Changelog:**
- **v1.7.0** — §3.1 now requires SHA-pinned raw URLs, not branch-pinned ones, for self-verification and for any downstream `CONFIRM` check. Root cause: a branch-pinned URL (`.../main/<path>`) can lag the actual commit for a period after a push — observed directly when a Workbench self-verification and a design-chat re-check of the same branch-pinned URL disagreed for over an hour after a real, already-landed commit. A SHA-pinned URL (`.../<commit-sha>/<path>`) is immutable and doesn't have this problem by construction. This removes the failure mode rather than catching it after the fact.
- **v1.6.0** — Added a `Timestamp:` line to the `Source:`/`Target:` header (§3.2), and the `SENT` acknowledgment convention (§3.3): after relaying a batch, Ron posts `SENT` back into the *source* chat so each chat's own scrollback shows what's actually been relayed versus still pending. Added because Ron needs to pick this workflow back up after time away, across three separate chat sessions with no shared view between them — the timestamp resolves "which version of this batch is current" and `SENT` resolves "did I already send this."
- **v1.5.0** — Added §3.2, the `Source:`/`Target:` header convention for relayed command batches: optional structure that attributes a batch's origin and destination and groups multiple commands sharing the same pair, rather than reconstructing that context from surrounding prose. Multi-target batches are always split into separate headers, one per target.
- **v1.4.0** — Added §3.1, mandatory self-verification before reporting `CONFIRM`: any entity that pushes a commit must fetch the raw URL itself and check the content matches before telling Ron it's done. Added after two consecutive commits were reported complete when the pushed content didn't actually match (once on `UDM_WORKFLOW_PROTOCOL.md` itself, once on `UDM_ROLES_AND_HANDOFF.md` never landing at all). This isn't about which entity made the mistake — it closes the gap so a reported commit is reliable without Ron needing to independently re-check every time.
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

### 3.1 Self-verification before reporting a commit

Any entity that pushes a commit — currently only the `udm-exchange` session, but this applies to any future entity with write access — must fetch the raw URL itself and confirm the pushed content actually matches what was intended, **before** reporting it as done. Check at minimum: the version number in the header, and any specific section called out in the handoff instructions as having changed.

**Use the SHA-pinned raw URL, not the branch-pinned one:**

```
Branch-pinned (do not use for verification):
https://raw.githubusercontent.com/RonClemens/udm-exchange/main/<path>

SHA-pinned (use this):
https://raw.githubusercontent.com/RonClemens/udm-exchange/<commit-sha>/<path>
```

A branch-pinned URL can lag the actual commit for a period after a push — this was observed directly (2026-07-27): a real, already-landed commit read as stale on the branch-pinned URL for over an hour, across both a self-verification check and an independent design-chat re-check, while the SHA-pinned URL for the same commit was correct immediately. A SHA-pinned URL is immutable by construction and doesn't have this failure mode. Get the commit SHA from whatever push/commit response is available (e.g., a `git` command's output, an API response) and use it for the verification fetch. Once verification passes, `CONFIRM` can still reference the branch-pinned URL as the canonical, human-readable link — the SHA-pinned form is for the verification step itself, not necessarily the one shared onward.

A reported completion that turns out not to match on re-check costs more than the extra minute of self-checking would have: it costs a full round-trip (Ron relays a `CONFIRM`, design chat catches the mismatch, Ron relays back, the commit gets redone), and it means every future `CONFIRM` from that entity needs independent re-verification rather than being trusted at face value. Self-verification isn't an optional courtesy — it's what keeps `CONFIRM` meaning "this is actually done" rather than "I attempted this."

### 3.2 `Source:` / `Target:` / `Timestamp:` batch headers (optional, strengthens when present)

Any relayed command or batch of commands may be preceded by a three-line header naming who issued it, who it's for, and when it was sent:

```
Source: <entity>
Target: <entity>
Timestamp: <YYYY-MM-DD HH:MM, timezone>

<VERB> <object>
<VERB> <object>
```

This is attribution, batching, and recency structure only — it doesn't add a new verb or state, and a batch with no header isn't a regression, just unattributed and untimed. It earns its keep mainly when Ron is relaying on another entity's behalf (e.g., reporting a `udm-exchange` self-check per §3.1), batching several commands that share one origin and destination, or picking the workflow back up after time away and needing to tell at a glance which of several similar-looking batches across three separate chat windows is actually the current one.

**Worked example:**

```
Source: udm-exchange session — self-verified per §3.1 (raw URL fetched, version + key section confirmed)
Target: design chat
Timestamp: 2026-07-27 07:47 ET

CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_WORKFLOW_PROTOCOL.md
CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_ROLES_AND_HANDOFF.md
```

**Multi-target batches are always split into separate headers, one per target**, even if the command text is identical — this keeps the common case (one source, one target) simple and avoids ambiguity about whether a batch was actually seen by every named recipient or only pasted once.

**Format note:** structured markdown, not JSON. Every current consumer of these commands is an AI chat entity reading prose that Ron copy-pastes between three separate chat UIs — nothing parses them programmatically yet, so JSON would be authoring overhead with no parser to benefit from it. Revisit only if a future automated consumer (e.g., a status board reading a transaction log) actually needs to ingest batches directly; the markdown form converts to JSON mechanically at that point.

### 3.3 `SENT` acknowledgment

After Ron actually pastes a batch into its target chat, Ron posts a single-word `SENT` reply back into the **source** chat — the same chat the batch's content originated from or was drafted in, not the destination.

This exists specifically because no chat window can see any other chat window's scrollback. Without it, stepping away and coming back gives no way to tell, just from looking at one chat, whether a drafted batch was ever actually delivered anywhere. With it, each chat's own history answers that on its own: a batch followed by `SENT` was delivered; a batch with no `SENT` under it wasn't, regardless of how much time has passed or how many other chats were touched in between.

`SENT` doesn't require an entity to do anything — it's Ron's own bookkeeping, posted into whichever window drafted the batch, and every entity should treat its presence or absence as informative rather than needing to ask what it means.

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

- **Design chat** — before responding to a new UDM-related request, checks the status board for anything stalled at `READY`/`FEEDBACK_READY` and names the count and which items first. When independently re-verifying a `CONFIRM`, prefers the SHA-pinned raw URL (§3.1) if one has been provided, and treats a mismatch on a branch-pinned URL alone as inconclusive rather than a confirmed failure until a SHA-pinned check is available.
- **`udm-exchange` session** — before acting on a new commit request, checks (a) whether anything in its own queue is unpushed, and (b) whether the requester's understanding of a document's current state matches what's actually on `main` — e.g. flags it if someone is treating a document as still `READY` when it's already `COMMITTED` or further along.
- **Workbench session** — before pushing new feedback or migration-plan updates, checks whether any of its own prior `FEEDBACK_READY` items — produced but not yet confirmed committed — are still outstanding, and names them.

No new command and no new state are needed for this — it's a discipline applied consistently, the same way the state machine already tracks documents, now applied to whichever entity is currently the holdup.
