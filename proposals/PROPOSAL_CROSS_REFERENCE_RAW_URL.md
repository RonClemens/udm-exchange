# Proposal: Raw URL Required on Every Cross-Document Reference

**Version:** 0.1.0 (Proposal — not yet merged into the Workflow Protocol)
**Last updated:** 2026-07-27
**Status:** Draft, for review
**Origin:** Ron's observation, after `UDM_ROLES_AND_HANDOFF.md` was handed off twice in a row with the framing "has never existed on main" when it had already been committed and unchanged both times. Ron's diagnosis: the recurring staleness isn't really a self-verification problem (§3.1 already covers that for the entity doing the pushing) — it's that a document's own header cites another document by name and version only (e.g. `UDM_WORKFLOW_PROTOCOL.md`'s "Companion to: `UDM_ROLES_AND_HANDOFF.md` v1.1.0"), with no link the citing entity could actually check itself.
**Target:** `UDM_WORKFLOW_PROTOCOL.md` §3, proposed as a new subsection (§3.4).

---

## 1. The gap

§3.1 (self-verification) and the SHA-pinned URL requirement (v1.7.0) both solve *"is the commit I just pushed actually correct."* They don't solve a different, related problem: *"is the document I'm citing, as I remember or assume it, actually what's currently committed."*

`UDM_WORKFLOW_PROTOCOL.md`'s header has said "Companion to: `UDM_ROLES_AND_HANDOFF.md` v1.1.0" since v1.4.0. That claim has been correct every time — but nothing in the header lets the citing document's author (or Ron, relaying it) confirm that without a side-channel check. The result, observed twice in one session: a HANDOFF instruction asserted `UDM_ROLES_AND_HANDOFF.md` "has never existed on main," which was already false both times, because the claim was carried in prose rather than checked against a link.

## 2. What's being proposed

Any time one document's header or body cross-references another document by name (a "Companion to:" line, a "Supersedes:" line, a "Target:" line in a proposal, etc.), the reference includes that document's current raw URL, not just its name and assumed version:

```
**Companion to:** [`UDM_ROLES_AND_HANDOFF.md`](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_ROLES_AND_HANDOFF.md) v1.1.0
```

This doesn't obligate the citing entity to fetch the link every time — it obligates the reference to *carry* the means to check, the same way §3.1 obligates a pusher to carry proof of its own work rather than asserting it. Whoever relays a HANDOFF (or reviews one) now has a link sitting right in the text instead of needing to reconstruct "where does this even live" from memory or a separate lookup.

**Branch-pinned is sufficient here**, unlike §3.1's verification step — a cross-reference is meant to always resolve to whatever is currently true, not to freeze a moment in time the way a self-check does. SHA-pinning a cross-reference would make it go stale the next time the referenced document revs, which is the opposite of what a "Companion to:" line is for.

## 3. Why this is the right fix for the actual recurring failure

Both times `UDM_ROLES_AND_HANDOFF.md` was mis-described as never-committed, the mistake was made by an entity that hadn't just pushed it (so §3.1 didn't apply) and had no link in front of it to check against (so there was nothing to catch the assumption). A raw URL sitting in the reference itself is a low-cost habit that makes the check trivial whenever anyone — Ron, this session, design chat — actually looks, rather than relying on every entity remembering to go verify a bare name-and-version claim on its own initiative.

## 4. Scope

This applies to cross-references *between* documents in this repo (Architecture Guidance ↔ PKM Entity Model ↔ Roles & Handoff ↔ Workflow Protocol, etc.), not to every mention of a filename in running prose — a changelog entry narrating what changed doesn't need a link on every noun, only the structural header-level "this document depends on / supersedes / targets that document" lines.

## 5. Open question for review

Should this apply retroactively — i.e., should the existing "Companion to:" lines in `UDM_WORKFLOW_PROTOCOL.md` and `ARCHITECTURE_GUIDANCE.md` (its "Companion to: PKM Entity Model" style references, if any) be updated to add the raw URL the next time each document revs anyway — or is a backfill commit worth doing on its own, even without another content change riding along? No strong position; whichever is less disruptive to the versioning discipline already in place.

---

*Procedural only — this doesn't touch PKM/Architecture Guidance content, only how documents cite each other.*
