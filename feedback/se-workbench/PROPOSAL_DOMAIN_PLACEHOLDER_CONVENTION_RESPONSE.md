# SE Workbench Response — Proposal: `@domain-placeholder` Convention v0.1.0

**From:** SE Workbench implementation session ("PDR Reconciliation & Baseline Alignment Workbench")
**Re:** `proposals/PROPOSAL_DOMAIN_PLACEHOLDER_CONVENTION.md` v0.1.0 (2026-07-26)
**Overall assessment:** Support merging this into Architecture Guidance. The generalization is accurate — nothing in the proposal misrepresents how this app actually uses the convention. This response flags one syntax correction, two real-world complications the proposal's flat-field framing doesn't cover yet, answers all three open questions from direct implementation experience, and offers one concrete example of why the mechanical-checkability argument in §1 is not hypothetical.

---

## 1. Syntax correction: this app does not use JSDoc-style tags

The proposal's example shows the marker as a JSDoc block comment:

```ts
/** @domain-placeholder */
name: string;
```

This app's actual implementation uses a plain line comment immediately above the field instead:

```ts
// @domain-placeholder
name: string;
```

Functionally these are interchangeable for a human reviewer or a simple grep-based check (`grep -rn "@domain-placeholder"`), which is all this app currently relies on. They are **not** interchangeable if open question #1 (tooling enforcement) ever becomes a real lint rule — most TS tooling that reads JSDoc tags natively (IDE hover, `tsdoc`-aware linters) expects the block-comment form; a plain line comment requires a custom parser regardless. Recommend the merged Architecture Guidance text pick one form explicitly rather than treating them as equivalent, and recommend it be the JSDoc form specifically, since that's the only one with a plausible path to native tooling support later. This app would need a mechanical, comment-only pass to conform if that's the direction chosen — low-risk, no structural change, same category as the retroactive-tagging note already in §4.4 of the proposal.

## 2. Two real-world complications the flat-field framing doesn't cover yet

The proposal's example (`ConfigurationItem.name`, `.description`) is a plain top-level field on a top-level entity. Two shapes this app actually has aren't covered by that framing, and the merged guidance should say something about both:

### 2.1 Whole-type tagging, not just whole-field tagging

`Specification.sections` is `Record<SpecSectionKey, string>` — a fixed set of twelve structural keys (`scope`, `safety`, `interfaces`, etc.), every one of whose *values* is program-specific requirement text. There's no single field to annotate; the tag has to live on the type alias itself:

```ts
// @domain-placeholder -- every section's text is program-specific requirement
// content, not generic structure. The 12 keys themselves are the reusable
// DID-derived structure and stay as-is.
export type SpecSections = Record<SpecSectionKey, string>;
```

Recommend the merged guidance explicitly allow (and show an example of) tagging a type alias, not only a field, for exactly this "fixed-key, variable-value" shape — it'll recur in any DID/CDRL-style structured-document entity, not just this app's.

### 2.2 Shared sub-types reused across multiple parent entities

`Attachment.label` (a link-only file reference) is nested inside `attachments: Attachment[]` on five different parent entities (`ConfigurationItem`, `CotsRecord`, `Specification`, `SafetyDeliverable`, `ProgramPlanningDeliverable`). The tag lives once, on `Attachment` itself, and applies transitively everywhere `Attachment[]` is used — it does not get re-tagged per parent. Same pattern for `QualifiedAlternate`'s two fields, nested inside `CotsRecord.qualifiedAlternates[]`. This is probably obvious once stated, but the proposal's single-entity example doesn't make it explicit, and a reviewer implementing this for the first time could plausibly wonder whether a shared sub-type needs re-tagging at every use site. Worth one sentence in the merged text ruling that out.

## 3. Answers to the open questions

**Open question #1 (tooling enforcement):** Worth gesturing at as a future direction, but the proposed heuristic — "flag any string field in `/data-schema` without either a structural-field justification or a `@domain-placeholder` tag" — will false-positive on a real category this app has: free-text string fields that are structural taxonomy, not program content. `ChecklistItem.domain: string` (values like "System Safety," "Verification & Validation") is deliberately **not** tagged, despite being a plain string with no enum constraint, because those category names are a reusable SE taxonomy, not this program's invented content — contrast with `ConfigurationItem.status: string`, a similarly-typed field that **is** tagged because its actual values ("Flagged for consolidation," "In reconciliation") are genuinely program-specific. A naive lint rule keyed on "untyped string field" alone can't distinguish these; whatever rule eventually gets built will need a way to affirmatively mark a field as "free text, but structural" (an allowlist comment, most likely) rather than only checking for the positive tag. Recommend noting this as a known limitation if/when tooling enforcement gets scoped, not something to solve now.

**Open question #2 (manifest format):** Markdown, at least for now. This app's manifest (`data-schema/DOMAIN_PLACEHOLDER_FIELDS.md`) has been genuinely useful as a human-reviewable document — exactly the audit/onboarding use case §2.1 of the proposal describes — and this repo's whole exchange model is markdown-document-driven, so format consistency with everything else here has real value. That said, agree the "Promises" UI use case (§2.2) is the actual argument *for* a machine-parseable format eventually: a generic cross-app Promises renderer can't hand-parse an arbitrary markdown table per app, but could trivially consume a shared JSON/YAML shape. Recommend: keep markdown as the required format now (don't block the convention on solving this), and treat a structured/machine-readable manifest as a natural v2 once a second app actually wants to reuse a shared Promises-UI implementation rather than hand-building its own, the way this app did.

**Open question #3 (retroactive tagging of mock-data/seed files):** No — recommend the guidance explicitly rule this out, not leave it open. This app deliberately marks only the schema/type-file default, never the mock-data values themselves, and states why directly in its own manifest: rewriting demo values into placeholder strings (e.g. `"<PLACEHOLDER>"`) would make the app unreadable as a demo, which defeats the actual purpose of having illustrative data in the first place. The tag's job is to tell a real deployment *which fields* need real content, not to make the current demo data illegible while it waits for that deployment. Recommend the merged Architecture Guidance text state this as settled, not as an open question — the answer isn't app-specific, it follows directly from what the marker is for.

## 4. One concrete example of why this matters (not hypothetical)

`CotsRecord.verificationMethod` was originally left untagged in this app's first pass, on the assumption that its values were drawn from a small fixed set of standard verification-method labels (Test/Analysis/Inspection/Demonstration-style). That assumption was wrong — the actual seed data holds descriptive free-text phrases (e.g. "inspection of vendor data sheet"), not a constrained enum — and the mistake sat unnoticed in the manifest for one full migration step before a later pass (converting the field into a real `VerificationEvent` reference) surfaced the discrepancy and it got corrected. This is exactly the failure mode §1 of the proposal describes — a human's field-by-field judgment call, made once under time pressure, quietly wrong for a while — and exactly what a `grep`-able manifest is good at catching once someone goes back and rechecks it against the actual data. Offered as supporting evidence for the proposal's core argument, not as a new concern.

## 5. Manifest filename

The proposal suggests `/data-schema/domain-placeholder-manifest.md` as the location. This app's actual file is `data-schema/DOMAIN_PLACEHOLDER_FIELDS.md` (same directory, different name/casing, matching this app's existing all-caps convention for standalone doc files). Noting the discrepancy in case the merged guidance wants to standardize on an exact filename — this app can rename in a trivial follow-up commit either way, so no preference here, just flagging that "the" filename isn't settled yet between the proposal text and this app's actual implementation.

---

## Scope note

This is feedback on the proposal document, not a request for Architecture Guidance to change on its own initiative — per the roles doc, canonical-document edits still route through Ron to the design chat. No code in this app has changed as a result of writing this response.
