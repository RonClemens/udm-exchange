# UDM Exchange

Shared, public-safe methodology documents for the Unified Data Model (UDM) effort — the Process Knowledge Model (PKM), the Architecture Guidance apps are expected to follow, and the per-app migration plans and feedback that result from applying them.

**This repo holds methodology-layer content only.** No app source code, no real program data, no CUI content of any kind. If a document you're about to add contains real requirement text, real dates, real findings, or anything that identifies a specific program's actual technical content, it belongs in that program's CUI-side repo instead — not here. See `ARCHITECTURE_GUIDANCE.md` §1 for the "would this be the same for any program?" test.

---

## Current versions

| Document | Version | Location |
|---|---|---|
| Architecture Guidance | v1.3.0 | `architecture-guidance/ARCHITECTURE_GUIDANCE.md` |
| PKM Entity Model | v0.2.1 (exploratory) | `pkm/PKM_ENTITY_MODEL.md` |
| SE Workbench Migration Plan | v0.2.0 (draft) | `migration-plans/se-workbench/PKM_MIGRATION_PLAN.md` |

---

## Structure

```
architecture-guidance/     # App-level architecture guidance — provider abstraction,
                             vendoring, directory conventions. Canonical source.

pkm/                        # Process Knowledge Model — the cross-program SE/PM/CM
                             ontology. Canonical source. Status: exploratory draft.

migration-plans/            # Per-app plans for adopting the PKM model.
  <app-name>/                 One folder per participating app.

feedback/                   # Per-app conformance/round-trip feedback on the above.
  <app-name>/                  One folder per participating app.
```

## How this repo is used

1. Canonical documents (`architecture-guidance/`, `pkm/`) are edited here directly, versioned, and changelogged within each file. Content changes to these two folders originate with the design chat and route through Ron; the `udm-exchange` coding session commits them but doesn't author them.
2. A CUI-side app repo vendors a pinned copy of whatever it needs, per Architecture Guidance §8 — never edits the vendored copy directly.
3. When an app reviews a document against its real implementation, that review lands in `feedback/<app-name>/`, pushed directly by that app's own coding session.
4. When an app's team plans how to adopt the PKM model, that plan lands in `migration-plans/<app-name>/`, same direct-push arrangement.
5. Findings from `feedback/` and `migration-plans/` get folded back into the canonical documents' next version, same as any other changelog entry.

Each participating app's coding session and the `udm-exchange` coding session read and write this repo directly — no human relay for the routine mechanics. Ron (the human coordinator) is only in the loop for: relaying documents from the design chat (which has no git tooling of its own), approving content changes to canonical documents, and resolving decisions that cut across more than one app's repo. See `UDM_ROLES_AND_HANDOFF.md` and `UDM_EXCHANGE_ACCESS_PROTOCOL.md` for the full model.

## Adding a new app to this exchange

Create `migration-plans/<app-name>/` and `feedback/<app-name>/` folders. No changes needed elsewhere — the canonical documents are already written to be program/app-agnostic.
