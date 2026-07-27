# Proposal: Source/Target/Batch Structure for Relayed Commands

**Version:** 0.2.0 (Proposal — not yet merged into the Workflow Protocol)
**Last updated:** 2026-07-27
**Status:** Draft, for review
**Origin:** v0.1.0 proposed a bare `Source:` line after Workflow Protocol §3.1 (self-verification) was used for the first time and the resulting `CONFIRM` reported the self-check only in surrounding prose. Ron then extended the idea further in the same session: every command has both an origin and a destination, and repeated commands between the same pair are naturally batched — this revision generalizes accordingly, before v0.1.0 had been reviewed by anyone else.
**Target:** `UDM_WORKFLOW_PROTOCOL.md` §3, proposed as a new subsection (§3.2) alongside §3.1.

**Changelog:**
- **v0.2.0** — Generalized from a single `Source:` line to a `Source:`/`Target:` pair plus explicit batching, and added the format-choice rationale (structured markdown now, not JSON, until something automated actually parses these).
- **v0.1.0** — Initial version: `Source:` line only, to attribute a `CONFIRM`/`CLOSE` to whichever entity actually performed the underlying check.

---

## 1. The gap

§3.1 requires an entity to self-verify a commit before reporting it as done. But the command vocabulary itself (`CONFIRM <path>`, `CLOSE <item>`, etc.) carries no field for *who issued it, to whom, or how it was verified* — all of that currently lives in prose around the command block (a chat-style "Ron → design chat:" header, or narrative like "self-verified per §3.1"), not in the command's own structure. Since this protocol's whole premise (§1) is that the vocabulary should carry the information rather than prose reconstructed alongside it, this is the same category of gap §1 already exists to close for everything else.

Concretely, without structure, none of the following is reliably recoverable from the command alone:
- Which entity originated the command (Ron manually, or relayed on behalf of a coding session's self-check).
- Which entity it's directed at (usually implicit from which chat window it's pasted into, but not from the text itself).
- Whether several commands in one message all share the same origin/destination, or are a mix that happens to be batched together for relay convenience.

## 2. What's being proposed

Every relayed command batch carries two header fields — **`Source:`** and **`Target:`** — followed by one or more commands. Multiple commands sharing the same source/target pair are listed together under one header, not repeated per line:

```
Source: <entity>
Target: <entity>

<VERB> <object>
<VERB> <object>
...
```

Worked example, matching what was actually relayed this session:

```
Source: udm-exchange session — self-verified per §3.1 (raw URL fetched, version + key section confirmed)
Target: design chat

CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_WORKFLOW_PROTOCOL.md
CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_ROLES_AND_HANDOFF.md
REVIEW https://raw.githubusercontent.com/RonClemens/udm-exchange/main/proposals/PROPOSAL_SOURCE_KEY_CONVENTION.md
```

A second batch to a different target, or from a different source, gets its own header rather than being folded into the first:

```
Source: Ron
Target: Workbench session

HANDOFF architecture-guidance/ARCHITECTURE_GUIDANCE.md v1.4.0 → Workbench
```

**Rule:** `Source:`/`Target:` are attribution and batching structure, not new verbs or new states — the state machine (§2) and command vocabulary (§3) are unchanged. A batch with no header is not a regression; it's simply unattributed, same as every command relayed before this convention existed.

## 3. Why structured markdown, not a JSON schema, for now

The natural next step once you have `{source, target, verb, object}` per command is to ask whether this should just be JSON. Deliberately not proposing that yet: **every current consumer of these commands is an AI chat entity reading prose that Ron copy-pastes between three separate chat UIs — nothing parses them programmatically.** JSON adds real value once something automated actually consumes a batch (a script, a generated status board reading a transaction log, a future dashboard) — until then, it's formatting overhead with no parser to benefit from it, and it makes every relay a hand-authored JSON object instead of a markdown block any of the three chat entities already read natively.

Structured markdown headers get most of the value now (attribution, batching, machine-*extractable* if something does want to parse it later — the format is regular enough to convert) without forcing JSON authoring into a workflow that is, at every step, a human pasting text between chat windows. If §4's status board (or some future tooling) ever needs to ingest these programmatically, converting this markdown convention to JSON at that point is a small, mechanical step — not a reason to pay the authoring cost today.

## 4. Open questions for review

1. Same as v0.1.0: should `Source:`/`Target:` become **mandatory** on every command once this is adopted, or stay optional structure that strengthens a command when present without invalidating one when absent?
2. Should `Target:` ever name more than one entity (e.g., a `STATUS` broadcast Ron wants multiple entities to see), or should multi-target batches always be split into separate headers, one per target, even if the command text is identical? No strong position here — whichever keeps the common case (one source, one target) simplest.

---

*Procedural only — this doesn't touch PKM/Architecture Guidance content, only how commands are relayed, attributed, and batched.*
