# SE Workbench — SEMP Generation Proposal Status Report

**Version:** 1.1.0
**From:** SE Workbench implementation session ("PDR Reconciliation & Baseline Alignment Workbench")
**Reports against:** UDM v2.0 SEMP Generation Proposal v0.5.0 ([raw](https://raw.githubusercontent.com/RonClemens/udm-exchange/main/proposals/UDM_V2_SEMP_GENERATION_PROPOSAL.md))

**Changelog:**
- v1.1.0 (2026-07-30) — added §3: the §2 correction is now moot — `RiskItem` has since been implemented for real via an explicit HANDOFF (Migration Plan v0.5.0 Step 10; see `PKM_MIGRATION_STATUS_REPORT.md` v1.8.0 §10). Also notes the proposal document's own v0.5.0 already corrected the overstated claim independently. Updated `Reports against:` to proposal v0.5.0.
- v1.0.0 (2026-07-29) — initial report: Phase C (schedule-table prototype) implemented, verified, deployed. Also flags a factual correction needed in the proposal document itself (see §2).

**Status:** Phase C only. This is a new, separate feedback document from `PKM_MIGRATION_STATUS_REPORT.md` since it tracks against a different document lineage (the SEMP-generation proposal, not the PKM Migration Plan) — created now because this is this app's first delivery against that proposal specifically.

---

## 1. Phase C — schedule/milestone table prototype: implemented

Per design chat's ACTION (2026-07-29 2342 UTC): a Baseline-grouped, chronologically-ordered
schedule table assembled directly from existing `Milestone` records (both `SETR` and
`AcquisitionGate` types). Pure query-and-format, per Architecture Guidance v1.5.0 §12.3's
"structured sections" path — no new PDKM content, no AI/provider calls, no schema change.

- **`buildMilestoneSchedule()`** (new, `client/src/utils/sempExport.ts`): groups `Milestone`
  records by `baselineId`, sorts chronologically within each group (`actualDate` if set, else
  `plannedDate`; a milestone with neither sorts last within its baseline rather than being
  dropped), and returns `{ event, milestoneType, status, actualDate, plannedDate }` rows per
  baseline.
- **On-screen:** a new "Generated SEMP: Schedule (Phase C prototype)" section in the existing
  SEMP Migration tab (`SempMigrationPage.tsx`) — explicitly framed in-app as a first slice of a
  future Generated SEMP view, not a standalone feature. No new nav entry, per the suggested shape.
- **Export:** the SEMP Migration package's own pre-existing milestone table (§3.2.13 of the
  generated Markdown) was reworked to use the same `buildMilestoneSchedule()` function — it
  previously showed all milestones flat and unsorted with no type column, which had become stale
  now that `Milestone` covers both `SETR` and `AcquisitionGate` records (Migration Plan Step 9).
  Both the on-screen view and the export now read from one source, not two divergent
  implementations.

**Schema footprint:** none. Pure read/format over the existing `Milestone`/`Baseline` entities.

**Verification:** clean `tsc -b` build in both workspaces; a Playwright pass confirming correct
chronological interleaving of `SETR` and `AcquisitionGate` events within each baseline (verified
via direct DOM inspection of the rendered table's row order, not just text search), the export's
new Type column and values, zero regressions in unrelated tabs.

## 2. Correction needed: the proposal document overstates this app's `RiskItem` status

Proposal v0.4.0 §4 states: *"SE Workbench: already the proven implementation partner for Role and
RiskItem alike, both verified end-to-end."* This is accurate for `Role` (Migration Plan Step 9 —
see `PKM_MIGRATION_STATUS_REPORT.md` v1.7.0 §9) but **not accurate for `RiskItem`**: this app has
not implemented, and has not been asked to implement, `RiskItem` in any form. No `RiskItem` type,
route, or seed data exists in this repo.

This app has also not yet received an explicit HANDOFF/ACTION for Migration Plan v0.5.0's Step 10
(`RiskItem` as a first-class entity) — the step exists in the document, marked "approved to
proceed now," but per this exchange's own workflow protocol (§6, accountability convention),
implementation work proceeds from an actual relayed command, not from this app reading ahead in a
canonical document on its own initiative. Flagging this now, before the "already verified"
claim propagates further, rather than silently implementing RiskItem unprompted to make the
claim retroactively true.

**Not a blocker** — this app is ready to take up Step 10 whenever it's relayed as an explicit
HANDOFF/ACTION, same as every step so far.

## 3. §2 resolved: `RiskItem` now genuinely implemented

The gap flagged in §2 is closed. An explicit HANDOFF for Migration Plan v0.5.0 Step 10 was
relayed on 2026-07-29, and `RiskItem` is now a real, implemented, verified entity in this app —
see `PKM_MIGRATION_STATUS_REPORT.md` v1.8.0 §10 for the full implementation report (entity shape,
derived `riskLevel`, CRUD route, `RiskItemsPage.tsx`, seed data covering all three `itemType`
values, and the two new `Gap`/`Recommendation` reference fields).

Separately, the proposal document's own v0.5.0 had already corrected the overstated §5 claim
this report's §2 flagged, independent of (and consistent with) this resolution. Recording both
here so this document's own changelog reflects the correction landing on both sides, not just
the proposal's.

---

*This report is implementation status, not a modification to the SEMP Generation Proposal, PKM Entity Model, or Migration Plan documents themselves.*
