# SE Workbench — PKM Migration Status Report

**Version:** 1.0.0
**From:** SE Workbench implementation session ("PDR Reconciliation & Baseline Alignment Workbench")
**Reports against:** PKM Migration Plan v0.2.0, PKM Entity & Relationship Model v0.2.1

**Status:** All 7 steps of the migration plan are implemented, verified, and deployed. This is a status/handoff report, not a request for new feedback on the plan itself — flagged items at the end are optional follow-ups, not blockers.

---

## 1. What shipped

Each step below is implemented, has passing server/client builds (both server-backed and static deploy modes), a live-server smoke test, and a Playwright UI smoke test, and is deployed to the app's static demo.

| Step | Entity/Change | Notes |
|---|---|---|
| 1 | `Program`, `Project` | Additive. One Program, one Project, matching this app's existing single-program scope. |
| 2 | `Baseline` (enum → entity) | Coordinated with the app's own methodology/data content-split for the recovery-program guidance file. `reconciledFromBaselineId`/`reconciledIntoBaselineId` used per PKM §5 open question #1; `AbCompatibilityRow` left unmigrated, per the plan's own carve-out. |
| 3 | `Milestone` | One record per SETR event per baseline lineage (Baseline A and B run independent timelines). Generic event definitions stay in the methodology layer; only per-baseline status/dates live on the entity, per the plan's explicit methodology/data split for this step. |
| 4 | `Requirement` | `satisfiedByCiIds` modeled many-to-many, mirroring `ConfigurationItem.subsystemIds` — real seed data (`delta-001`) is an actual case of one requirement whose as-built satisfaction spans three CIs, not the single CI its paper allocation named. `parentRequirementId` demonstrated via a second requirement that is implicitly part of the first. |
| 5 | `ChecklistItem`, `VerificationEvent` | First slice only, per the plan's own risk note (this is content authoring, not mechanical conversion). Covers the two currently in-progress milestones and one COTS verification-method conversion. `ChecklistItem` uses a polymorphic `evidenceType`/`evidenceId` reference. `domain` is a plain string attribute, left provisional pending PKM §5 open question #2. |
| 6 | `Gap` | First slice only. Demonstrates real unification, not just new structure: one existing finding was already tracked by two separate mechanisms (a CI's own over-decomposition flag and a Delta Matrix row) and both now reference the same `Gap` record. `Recommendation` deliberately not touched here — see Step 7. |
| 7 | ActionItem role enforcement | `Recommendation.owner` converted from free text to a fixed role taxonomy (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager) — a pragmatic starting cut, not a definitive one. Added `resolvesGapId`, coexisting with the existing `relatedCiId`. |

**Consistent pattern across all 7 steps:** additive-only, coexist-then-deprecate. No existing field, table, filter, or export was removed or restructured — every new entity/reference sits alongside what already worked. This kept the migration's actual regression surface at zero across all 7 steps' UI/build/smoke verification.

## 2. Also shipped this pass (not in the migration plan, but related)

- **`@domain-placeholder` marker convention** — every field across every entity whose *value* is program-specific illustrative content (as opposed to structural fields: ids, references, fixed enums, timestamps) is now tagged in both type files, with a manifest doc explaining the convention and listing every tagged field per entity.
- **PDKM Promises tab** — a new, read-only, filterable browser over every current record's `@domain-placeholder` values (including nested attachments, qualified alternates, and specification sections), framed as a promissory note: each value is synthetic and will be replaced once a real Product/Domain Knowledge Model exists for the program a given deployment serves, arriving via landing-zone upload or direct user entry.
- **Forward-compatibility note for a future guided/wizard interface** — this app's maintainer indicated an intent to eventually build a "TurboTax-style" prompted-question interface, fed by the same underlying model as the tabbed webapp, that would toggle which CDRLs apply in real time based on DID-derived trigger rules. `ChecklistItem`'s shape (a discrete, user-answerable criterion + toggleable status + structured evidence reference) was designed with this specifically in mind — nothing in the current schema should need to change shape when that UI is eventually built. No wizard UI or rules engine has been built yet; this is a documented design intent only.

## 3. Optional follow-ups (not blockers)

These surfaced during implementation and are offered as candidate topics for continued discussion, not requests:

1. **PKM §5 open question #2** (ChecklistItem domain tagging) is still open. This app's implementation used a plain string attribute as a pragmatic default — real usage may clarify whether "Domain" deserves first-class status.
2. **Role taxonomy for ActionItem/Recommendation.owner** (Step 7's own deferred decision) was resolved locally with a five-role starting set (Lead Systems Engineer, CM Lead, Software Lead, Safety Lead, Program Manager). Worth confirming whether this is a reasonable default for other apps building against the same PKM, or program-specific enough that each app should define its own.
3. **Gap's evidence cardinality**: this implementation used a single `foundInEntityType`/`foundInEntityId` reference per Gap (one location per finding). Real data already shows a finding can be "found in" one place while being *referenced by* multiple other mechanisms (the CI flag + Delta Matrix row case in Step 6) — worth a future pass on whether Gap needs to support multiple `foundIn` references, not just multiple things pointing *at* it.
4. **Requirement/VerificationEvent/ChecklistItem decomposition of `Specification.sections`** remains deferred, per Step 4's and Step 5's own explicit scoping — flagged here only so it isn't lost as a known-remaining slice, not because it needs to happen next.

---

*This report is implementation status, not a modification to the PKM Entity Model or Migration Plan documents themselves. If any of the optional follow-ups above turn into an actual PKM modeling question, that should route back through the normal conformance-feedback path.*
