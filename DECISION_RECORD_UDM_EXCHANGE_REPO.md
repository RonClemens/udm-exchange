# Decision Record: Standalone `udm-exchange` Repo vs. In-Repo Folder

**Date:** 2026-07-25
**Status:** Decided
**Context documents:** Architecture Guidance v1.3.0, PKM Entity Model v0.2.1

---

## Background

To let Claude read and review UDM exchange documents (Architecture Guidance, PKM model, migration plans, feedback) without a human manually transferring files each round, a shared folder was set up at `docs/udm-exchange` inside the SE Workbench app's own (private) repo, first on a working branch, then pushed to `main`.

## Problem

Claude's web-fetch tool could not read from that location for two independent reasons:

1. **GitHub's `robots.txt` disallows automated access to `github.com` blob/tree UI pages entirely**, regardless of branch or repo visibility. This affected every URL form tried, including direct file links.
2. **The Workbench repo is (correctly) private**, given it contains real application source, structural schema details, and program-specific content (e.g., `Baseline A`/`Baseline B` naming, `recoveryProgramGuidance.ts`). Private-repo file requests return the same 404 to an unauthenticated fetch as a genuinely missing file, and Claude's fetch tool has no GitHub credentials to authenticate as the repo owner.

Making the *whole* Workbench repo public to solve the read-access problem was considered and rejected: GitHub visibility is repo-wide, not per-folder, so it would have exposed the app's actual source code and program-specific content — a much larger disclosure than intended, and not cleanly reversible once public (clones/forks/caches could persist through even a brief public window).

## Decision

Create a **new, standalone, public repository** (`udm-exchange`) containing only the methodology-layer exchange documents, separate from any app's own repo. This follows the same "toolkit repo vs. data repo" separation principle already established earlier for the public/CUI sync architecture — applied here to documentation instead of code.

## Rationale

- **Preserves the Workbench repo's privacy** — no visibility change needed there at all.
- **Solves the actual access problem** — a public repo's files are reachable via `raw.githubusercontent.com` links, which are not blocked by the `robots.txt` restriction that blocks `github.com` UI pages.
- **Matches existing content-boundary discipline** — exchange documents were already required to be program-agnostic (Architecture Guidance §1's "would this be the same for any program?" test); a dedicated repo makes that boundary structural rather than just a writing convention.
- **Scales to future apps** — a single shared exchange repo, organized by app under `migration-plans/<app-name>/` and `feedback/<app-name>/`, avoids needing a new exchange arrangement invented per app as more apps join the UDM effort.

## Consequences

- Claude can now read canonical documents and per-app feedback directly via raw file links, once committed to `udm-exchange`.
- Claude still has **no write/commit capability** to any repository, public or private — the app owner or the relevant app's coding session must still push documents into `udm-exchange` after Claude produces them. This decision resolves the read-side friction only, not the full round-trip.
- Any future document proposed for `udm-exchange` must pass the same content test as everything else in this effort: structural/methodology content only, never real program data.

## Alternatives considered

| Option | Rejected because |
|---|---|
| Keep `docs/udm-exchange` folder in Workbench repo (private) | Not fetchable by Claude — no GitHub auth available |
| Make the Workbench repo public | Repo-wide visibility exposes real app code and program-specific content well beyond the exchange docs; not reversible |
| Continue manual upload/download only | Works reliably but keeps a human in the loop for every round-trip, which this effort was specifically trying to reduce |
