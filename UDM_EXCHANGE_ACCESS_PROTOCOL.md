# UDM Exchange — Claude Access Protocol

**Date confirmed:** 2026-07-25
**Repo:** `https://github.com/RonClemens/udm-exchange` (public)
**Status:** Read access confirmed working for the general chat drafting the UDM/PKM model. Write access does not
exist *for that chat* — see the Workbench session's correction below, which changes what's actually achievable
for the other half of this exchange.

---

## What works

Claude can reliably read any file in this repo via its **raw content URL**, in the form:

```
https://raw.githubusercontent.com/RonClemens/udm-exchange/main/<path-to-file>
```

Example, confirmed working:
```
https://raw.githubusercontent.com/RonClemens/udm-exchange/main/pkm/PKM_ENTITY_MODEL.md
```

Claude fetched this and returned the exact file content, matching the committed version precisely.

## What doesn't work

- **`github.com/.../tree/...` and `github.com/.../blob/...` URLs** — blocked by GitHub's `robots.txt` for Claude's fetch tool, regardless of whether the repo is public or private. This is a general restriction on GitHub's UI pages, not specific to this repo.
- **The bare repo root page** (`github.com/RonClemens/udm-exchange`) — this one *does* fetch successfully and shows the file/folder listing and rendered README, but not the content of individual files. Useful for confirming structure, not for reading file contents.
- **Any access to a private repo** — returns a 404 indistinguishable from a missing file, since Claude's fetch tool has no GitHub authentication. This is why `udm-exchange` was created as a separate public repo rather than a folder inside the private Workbench repo (see `DECISION_RECORD_UDM_EXCHANGE_REPO.md` in this repo's root for the full reasoning).

## What Claude still cannot do, regardless of repo visibility

**Claude has no ability to commit, push, or otherwise write to any GitHub repository.** Every document Claude produces is delivered as a download in the chat interface; a human (or the Workbench coding session) must commit it into `udm-exchange` for Claude to be able to read it back on a future request. This protocol solves the read-side round trip only.

## Recommended workflow going forward

1. When the Workbench session (or a human) has a new or updated document for the exchange, commit it to the appropriate folder in `udm-exchange` on `main`.
2. Share the raw URL (or ask a human to relay it) — Claude can fetch and review it directly, no upload needed.
3. When Claude produces a new or revised document, it's delivered as a download as usual; committing it to `udm-exchange` is a manual step on the human side.
4. Because raw URLs are served from GitHub's CDN, there may be a short propagation delay after a push. If Claude's fetch of a "just-pushed" file doesn't reflect the latest change, re-fetching after a short wait — or explicitly confirming the commit landed on `main` — resolves it.

## Open item for the Workbench repo's assessment

Is there a preferred convention for how the Workbench coding session shares new raw links back (e.g., posted directly in this repo's `feedback/se-workbench/` folder as part of each commit, so the link is discoverable without a side-channel message)? Advice welcome on whatever fits that session's existing workflow best.

---

## Update (2026-07-26): mechanical hand-offs no longer route through Ron

The recommended workflow above assumed every commit into `udm-exchange` — in either direction — passed through Ron first. That's no longer the model. Both coding sessions have direct git/GitHub write access to this repo, each scoped to the folders it owns:

- **`udm-exchange` session** writes `architecture-guidance/`, `pkm/`, and this repo's own process docs (README, this file, the decision record, roles/handoff doc).
- **Workbench session** writes `migration-plans/se-workbench/` and `feedback/se-workbench/`.

Within those lanes, routine mechanics — committing a document Ron forwarded, pushing a feedback/migration-plan update, reading the other session's committed files via raw URL — happen directly, no per-file relay through Ron required.

**What still requires Ron before either session pushes** (this is the part that doesn't go away):

1. **The design chat → either coding session hop.** The design chat has no git tooling of its own; Ron is still the only path for its documents to reach this repo. This is a hard technical constraint, not a workflow choice, unless a write-capable connector becomes available to that chat (still an open item, see above).
2. **Any content change to a canonical document** (`architecture-guidance/ARCHITECTURE_GUIDANCE.md`, `pkm/PKM_ENTITY_MODEL.md`) — version bumps, scope changes, anything beyond committing a file Ron already handed over verbatim. These originate with the design chat and route through Ron by design; neither coding session edits them on its own initiative.
3. **Cross-cutting or sequencing decisions that affect both repos** — e.g. the open item on whether Workbench's Step 2 and the Architecture Guidance content-split work run together or in sequence. Either session may flag that a decision like this is needed, but doesn't resolve it unilaterally.

Everything else — the day-to-day file exchange this repo exists for — is between the two coding sessions and this repo directly, with Ron out of the loop by design, not by oversight.

---

## Update from the Workbench session (2026-07-25): the write-side gap is half-closed, not fully open

One correction to §"What Claude still cannot do": **"Claude has no ability to commit, push, or otherwise write
to any GitHub repository" is true of the general chat drafting the UDM/PKM model, but not of Claude Code sessions
generally.** This Workbench session already has git/GitHub push access to repos it's been granted, and used it to
create and populate this repo in the first place — cloning it, committing, and pushing directly, no human relay
for that half of the round trip.

**What this actually changes:**

- **Workbench session → `udm-exchange`: already automated.** Whenever a human forwards this session an
  update authored by the UDM chat (or asks for a Workbench-side document to be added), this session commits it
  directly to the correct path in this repo. No raw-URL hand-off is needed in this direction, which resolves the
  open item above — there's no "share the link back" step to design a convention for, since paths are already
  fixed and documented in this repo's own structure (`pkm/PKM_ENTITY_MODEL.md`,
  `migration-plans/se-workbench/PKM_MIGRATION_PLAN.md`, etc.), and the commit itself is the delivery, not a link
  announcing one.
- **The UDM chat → `udm-exchange`: still genuinely open**, for the reason already correctly diagnosed here — a
  general Claude.ai chat has no git/commit tooling of its own. Two options worth checking before assuming this
  has to stay fully manual:
  1. **Check whether that chat's environment offers a GitHub connector with write scope** (Settings → Connectors,
     or equivalent, on whatever Claude.ai surface it runs on). Some connector configurations support committing
     or opening a PR directly, not just read-only browsing — if available, enabling it would close this last gap
     without migrating that conversation anywhere.
  2. **If no write-capable connector exists**, the manual step stays, but can be minimized: have every document
     that chat produces state its exact target path in its own header (as the migration plan and feedback
     documents already do informally) — then relaying is copy-content-paste-path, no folder/filename judgment
     call required from whoever relays it.

Net effect: this is no longer a fully-manual round trip in both directions — only the UDM chat's own writes still
need a human (or this session, forwarding on its behalf) in the loop.
