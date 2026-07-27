# Proposal: Source Key on Relayed Commands

**Version:** 0.1.0 (Proposal — not yet merged into the Workflow Protocol)
**Last updated:** 2026-07-27
**Status:** Draft, for review
**Origin:** Ron's request, immediately after Workflow Protocol §3.1 (self-verification) was used for the first time — the resulting `CONFIRM` block reported that this session had self-verified, but nothing in the command's actual format captured that; it was only stated in surrounding prose.
**Target:** `UDM_WORKFLOW_PROTOCOL.md` §3, proposed as a new subsection (§3.2) alongside §3.1.

---

## 1. The gap

§3.1 requires an entity to self-verify a commit before reporting it as done. But the command vocabulary itself (`CONFIRM <path>`, `CLOSE <item>`, etc.) carries no field for *how* that confirmation was arrived at. In practice this session included the self-verification claim as prose around the command block, not inside it — which means the provenance depends on Ron accurately relaying surrounding text, rather than being part of the structured trigger itself. Given this protocol's whole premise is that the vocabulary should carry the information, not prose reconstructed alongside it, this is the same category of gap §1 already warns about.

Concretely, right now design chat cannot tell, from the command alone, whether a `CONFIRM` reflects:
- Ron's own independent check of the raw URL, or
- an AI entity's self-verification (per §3.1), relayed by Ron, or
- neither — a `CONFIRM` sent before any check happened at all (the exact failure mode §3.1 was written to prevent).

## 2. What's being proposed

Add an optional **`Source:`** line immediately above any command batch that reports a completed action, naming which entity performed the underlying verification and how:

```
Source: <entity> — <method>

CONFIRM <path>
```

Examples:
```
Source: udm-exchange session — self-verified per SS3.1 (raw URL fetched, version + key section confirmed)

CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/UDM_WORKFLOW_PROTOCOL.md
```

```
Source: Ron — verified directly in GitHub UI

CONFIRM https://raw.githubusercontent.com/RonClemens/udm-exchange/main/architecture-guidance/ARCHITECTURE_GUIDANCE.md
```

**Rule:** a `CONFIRM` with no `Source:` line is not a regression — it's simply unattributed, same as today. But once §3.1 self-verification exists as a named obligation, any entity claiming to have satisfied it should say so in a form the receiving entity can parse and rely on, not just in accompanying prose that may or may not survive relay.

## 3. Why this is a small addition, not new machinery

- No new verb, no new state — same category as §6's accountability convention: a discipline layered onto the existing vocabulary, not a protocol rewrite.
- Optional, not mandatory on every command — `HANDOFF`, `REVIEW`, `STATUS`, `NEXT` don't report a completed verification, so they have no obvious use for a source key. It only makes sense on `CONFIRM` (and arguably `CLOSE`, when closing a loop based on a check rather than just a decision).
- Doesn't change what any entity is allowed to do — it only makes an existing claim ("I checked this") structured and attributable instead of implicit in prose.

## 4. Open question for review

Should `Source:` become **mandatory** on every `CONFIRM` once §3.1 is in force (i.e., a `CONFIRM` without one is treated as unverified and not trusted), or stay optional metadata that strengthens a `CONFIRM` when present but doesn't invalidate one when absent? This proposal doesn't take a position — whichever design chat judges keeps the protocol lightweight rather than adding a compliance burden nobody asked for.

---

*Procedural only — this doesn't touch PKM/Architecture Guidance content, only how commands are relayed and attributed.*
