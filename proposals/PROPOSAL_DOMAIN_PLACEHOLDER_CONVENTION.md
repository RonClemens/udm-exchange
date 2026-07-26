# Proposal: The `@domain-placeholder` Convention — Making Data-Layer Boundaries Machine-Checkable

**Version:** 0.1.0 (Proposal — not yet merged into Architecture Guidance)
**Last updated:** 2026-07-26
**Status:** Draft, for review
**Origin:** Observed in SE Workbench's PKM Migration Status Report (2026-07-26) as a self-initiated addition, not part of the migration plan. Proposed here for generalization into Architecture Guidance so it becomes a required pattern for all apps, not a Workbench-specific one.
**Target:** Architecture Guidance — proposed as new §11, or folded into §2/§4 if reviewers prefer it live alongside the directory convention and data injection pattern rather than standalone.

---

## 1. The problem this solves

Architecture Guidance §1's core test — *"would this line of code be the same regardless of program?"* — and §1.1's anti-pattern note (editable override ≠ real separation) both rely on a human correctly judging, field by field, whether a given piece of content is genuinely reusable methodology or disguised program-specific data. That judgment is exactly the kind of thing that erodes under time pressure, changes hands between developers, or gets missed during a fast migration.

`@domain-placeholder` turns that judgment into a **visible, greppable marker in the source itself**, so the boundary Architecture Guidance already requires becomes something a linter, a reviewer, or a future migration pass can check mechanically instead of re-litigating by eye every time.

## 2. The convention

**Tag every field whose current value is program-specific illustrative content** — as distinct from structural fields (IDs, foreign-key references, fixed enums, timestamps, booleans) — with an `@domain-placeholder` marker in the type definition.

```ts
interface ConfigurationItem {
  id: string;                          // structural — not tagged
  subsystemIds: string[];              // structural — not tagged
  baselineId: string;                  // structural — not tagged

  /** @domain-placeholder */
  name: string;                        // illustrative program content — tagged

  /** @domain-placeholder */
  description: string;                 // illustrative program content — tagged
}
```

The rule for what gets tagged is the same test Architecture Guidance already uses (§1), applied at field granularity rather than file granularity: **if this field's current value is a synthetic stand-in for something a real program will eventually supply, tag it.** Structural fields — the shape itself — are never tagged, because the shape is exactly what's meant to be reused; only the illustrative *values* sitting in that shape are.

### 2.1 The manifest

A companion manifest document lists every tagged field, per entity, in one place — not just scattered inline comments. This gives a single, reviewable inventory of the app's entire program-specific-content surface, useful for:
- A fast audit of "how much of this app's current data is actually placeholder" without reading every type file.
- Confirming, at CUI-vendoring time (Architecture Guidance §8), that nothing tagged has silently become load-bearing default content.
- Onboarding — a new contributor can read the manifest instead of inferring the boundary from scratch.

### 2.2 The "Promises" view (UI pattern, optional but recommended)

Workbench paired the marker convention with a read-only, filterable in-app view surfacing every current record's `@domain-placeholder` values in one place, framed explicitly as a **promissory note**: each value is synthetic and will be replaced once a real Product/Domain Knowledge Model (PDKM) exists for the program a given deployment serves — arriving via landing-zone upload or direct user entry, per the data injection pattern in §4.

This isn't required for the marker convention itself to be useful, but it's a strong, low-cost pattern worth recommending: it makes the promise visible to anyone using the app, not just to developers reading source, and it gives a natural home for a future "load real PDKM data" workflow to attach to.

## 3. Why this belongs in Architecture Guidance, not just Workbench

This is a direct generalization of work Architecture Guidance already requires:

- It's a **stronger, checkable version of §1's own test** — the test already exists in prose; this makes it visible in code.
- It **closes the exact loophole §1.1 was written to name** — a field can be runtime-editable (via a `ContentEntry`-style override) and still be tagged `@domain-placeholder` if its baked-in default is program-specific. The marker applies to the source-file default, same as §1.1 already insists on, just made mechanically checkable instead of relying on a reviewer remembering the rule.
- It **extends naturally from §9's UDM forward-compatibility conventions** — §9 already asks for stable external IDs and explicit reference fields specifically so a future PDKM can attach cleanly. `@domain-placeholder` is the missing third piece: it marks exactly *where* a PDKM's real values are meant to attach once one exists, field by field.
- It **fits the existing directory/vendoring model (§2, §6, §8)** with no new machinery — the manifest is just another versioned file living alongside `/data-schema`, and vendoring/patch discipline already applies to it the same way it applies to everything else in the methodology tree.

## 4. What's being proposed, concretely

1. Add `@domain-placeholder` as a required marker convention in Architecture Guidance, most likely as new §11 (or folded into §2/§4 — open to reviewer preference on placement).
2. Require a manifest file per app (suggested location: `/data-schema/domain-placeholder-manifest.md` or `.json`, consistent with §2's existing `/data-schema` directory).
3. Recommend, but not mandate, a "Promises"-style read-only UI view as a strong pattern — every app doesn't need identical UI, but every app should have *some* visible way to answer "what in this deployment is still synthetic."
4. Add a migration-checklist line to §7 for apps not yet using this convention: retroactively tagging existing fields is additive and low-risk (pure annotation, no structural change), so it can slot in early in an app's sequencing rather than waiting for the content-split step.

## 5. Open questions for review before merging into Architecture Guidance

1. **Tooling enforcement** — should this stay a documentation/code-review convention, or eventually get a lint rule (e.g., flag any string field in `/data-schema` without either a structural-field justification or a `@domain-placeholder` tag)? Not proposing to build that now, but worth deciding whether the guidance should gesture at it as a future direction.
2. **Manifest format** — markdown table (human-readable, easy to review) vs. a structured JSON/YAML manifest (machine-parseable, could feed a future "Promises" UI generically across apps without hand-building one per app). Workbench's current manifest is markdown; worth deciding whether Architecture Guidance should recommend a specific format or leave it open per app.
3. **Does this apply retroactively to `mock-data`/`seed` files too**, or only to the `/data-schema` type definitions? The type-level tag is what's been demonstrated; whether the same marker convention should also touch seed/mock data files (which are already understood to be synthetic by virtue of being mock data) is a separate question this proposal doesn't resolve.

## 6. Non-goals

This proposal does not ask any app to build a wizard/guided interface, a rules engine, or any specific PDKM ingestion mechanism. Those remain separate, later decisions (Workbench's own forward-compatibility note on a future TurboTax-style interface is unrelated to this proposal and shouldn't be conflated with it). This proposal is scoped narrowly to the marker + manifest convention, plus the optional-but-recommended UI pattern.

---

*If this lands well on review, the next step is folding it into Architecture Guidance as v1.4.0 with a changelog entry, and updating the migration checklist (§7) accordingly.*
