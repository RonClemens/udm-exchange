# UDM Exchange

Shared, public-safe methodology documents for the Unified Data Model (UDM) effort — the Process Knowledge Model (PKM), the Architecture Guidance apps are expected to follow, and the per-app migration plans and feedback that result from applying them.

**This repo holds methodology-layer content only.** No app source code, no real program data, no CUI content of any kind. If a document you're about to add contains real requirement text, real dates, real findings, or anything that identifies a specific program's actual technical content, it belongs in that program's CUI-side repo instead — not here. See `ARCHITECTURE_GUIDANCE.md` §1 for the "would this be the same for any program?" test.

---

## Current versions

| Document | Version | Location |
|---|---|---|
| Architecture Guidance | v1.7.1 | `architecture-guidance/ARCHITECTURE_GUIDANCE.md` |
| PKM Entity Model | v0.7.3 (exploratory) | `pkm/PKM_ENTITY_MODEL.md` |
| SE Workbench Migration Plan | v0.7.0 (draft) | `migration-plans/se-workbench/PKM_MIGRATION_PLAN.md` |
| UDM Workflow Protocol | v1.10.0 | `UDM_WORKFLOW_PROTOCOL.md` |
| UDM Roles & Handoff | v1.2.0 | `UDM_ROLES_AND_HANDOFF.md` |

## Pending proposals

Draft changes to a canonical document, awaiting review before they merge in. Not yet in effect.

| Proposal | Targets | Status |
|---|---|---|
| `@domain-placeholder` Convention | Architecture Guidance (proposed new §11) | **Merged** into Architecture Guidance v1.4.0 §10 — `RESOLVED` |
| Uniform Stall-Check Obligation | `UDM_WORKFLOW_PROTOCOL.md` §6 | **Merged** into Workflow Protocol v1.3.0 §6 — `RESOLVED` |
| Source/Target/Batch Structure for Commands | `UDM_WORKFLOW_PROTOCOL.md` §3.2/§3.3 | **Merged** into Workflow Protocol v1.6.0 — `RESOLVED` |
| Raw URL Required on Cross-Document References | `UDM_WORKFLOW_PROTOCOL.md` §3.4 | **Merged** into Workflow Protocol v1.8.0 — `RESOLVED` (no explicit `CLOSE` issued; marked resolved since §3.4's content fulfills it — flagged in case that's not yet intended) |
| UDM v2.0 — Automated SEMP Generation Architecture (`proposals/UDM_V2_SEMP_GENERATION_PROPOSAL.md` v0.6.2) | Architecture Guidance §11–§12 shipped; PKM `Role`/`RiskItem` (structure, data, SEMP-wiring) all shipped and verified; all 4 data-integrity findings now resolved/closed (Step 11 closed as not reproducible against Workbench's real data, source upstream of that repo) | **Draft** — only genuinely open item across all of 2026-07-30's work: whether DI-SESS-81785B/SEP Outline v4.1 supersedes 24748-4 (§6 item 1, with Ron). §6 item 8 (static-deploy caching limitation) still a candidate Architecture Guidance addition, not yet drafted. |

## Open item

None currently blocking. `UDM_ROLES_AND_HANDOFF.md` v1.1.0 landed 2026-07-27, closing the version-mismatch gap previously tracked here.

---

## Structure

```
UDM_TODO.md                # Lightweight, low-ceremony running list of loose ends that don't
                             yet belong in any document's own Open Questions section. No
                             changelog discipline required to add/update an entry — only to
                             commit changes to the file itself.

architecture-guidance/     # App-level architecture guidance — provider abstraction,
                             vendoring, directory conventions. Canonical source.

pkm/                        # Process Knowledge Model — the cross-program SE/PM/CM
                             ontology. Canonical source. Status: exploratory draft.
  examples/                   # Non-canonical: synthetic pseudo-data instance graphs
                               validating a PKM revision holds together end-to-end.
                               Fictional content only, per this repo's content-boundary
                               rule. Not a PDKM template or schema recommendation.

migration-plans/            # Per-app plans for adopting the PKM model.
  <app-name>/                 One folder per participating app.

feedback/                   # Per-app conformance/round-trip feedback on the above.
  <app-name>/                  One folder per participating app.

proposals/                  # Draft changes to a canonical document, pending review.
                             # Not in effect until folded into architecture-guidance/
                             # or pkm/ with a version bump and changelog entry.
```

## How this repo is used

1. Canonical documents (`architecture-guidance/`, `pkm/`) are edited here directly, versioned, and changelogged within each file. Content changes to these two folders originate with the design chat and route through Ron; the `udm-exchange` coding session commits them but doesn't author them.
2. A CUI-side app repo vendors a pinned copy of whatever it needs, per Architecture Guidance §8 — never edits the vendored copy directly.
3. When an app reviews a document against its real implementation, that review lands in `feedback/<app-name>/`, pushed directly by that app's own coding session.
4. When an app's team plans how to adopt the PKM model, that plan lands in `migration-plans/<app-name>/`, same direct-push arrangement.
5. Findings from `feedback/` and `migration-plans/` get folded back into the canonical documents' next version, same as any other changelog entry.

Each participating app's coding session and the `udm-exchange` coding session read and write this repo directly — no human relay for the routine mechanics. Ron (the human coordinator) is only in the loop for: relaying documents from the design chat (which has no git tooling of its own), approving content changes to canonical documents, and resolving decisions that cut across more than one app's repo. See `UDM_ROLES_AND_HANDOFF.md` and `UDM_EXCHANGE_ACCESS_PROTOCOL.md` for the full model.

## Feedback document conventions

`feedback/<app-name>/` is meant to reflect each app's *current* standing feedback — not an accumulating pile of every review ever written. Direct-push access to this folder comes with two obligations:

1. **State what version you're reviewing.** Every feedback document's header must name the exact version of the canonical document it reports against (e.g. "Re: PKM Entity Model v0.2.1"), checked against the **Current versions** table above at the time of writing. A document whose header cites an older version than what's in that table is stale by definition.
2. **Don't leave superseded feedback in place unmarked.** When a new feedback document replaces an older one on the same topic (because the canonical doc revved, or the review was redone), the old document must not simply sit alongside the new one indistinguishable from current input. Either:
   - move it to `feedback/<app-name>/archive/` before or as part of the same commit that adds its replacement, or
   - add a one-line marker at the top of the superseded file — `**Superseded by:** <new-file>, as of <date>` — in the same commit.

This applies whenever an app's session pushes feedback directly (see "What changed" in `UDM_EXCHANGE_ACCESS_PROTOCOL.md`) — there's no human relay step left to catch a stale re-send before it reaches whoever's reading raw URLs from this repo.

## Adding a new app to this exchange

Create `migration-plans/<app-name>/` and `feedback/<app-name>/` folders. No changes needed elsewhere — the canonical documents are already written to be program/app-agnostic.
