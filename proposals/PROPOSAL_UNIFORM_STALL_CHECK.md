# Proposal: Uniform Stall-Check Obligation Across All Three AI Entities

**Version:** 0.1.0 (Proposal — not yet merged into the Workflow Protocol)
**Last updated:** 2026-07-27
**Status:** Draft, for review
**Origin:** Ron's request, prompted by `UDM_WORKFLOW_PROTOCOL.md` v1.2.0 §6 — wanting a standing guarantee that work never silently stalls, not just a rule that exists in principle.
**Target:** `UDM_WORKFLOW_PROTOCOL.md` §6, proposed as v1.3.0.

---

## 1. The gap

§6's stated rule is symmetric: "This applies equally to Ron and to the three AI sessions." But the only concrete, operational instruction in §6 is scoped to one entity:

> "In practice, for design chat specifically: at the start of any UDM-related exchange, if the status board shows items stalled at `READY`/`FEEDBACK_READY`, design chat names the count and which items before doing anything else..."

The `udm-exchange` session and the Workbench session have no equivalent written instruction. In practice this session has been doing something close to it anyway (e.g. flagging the stale `UDM_ROLES_AND_HANDOFF.md` version mismatch and the outdated worked example before proceeding, in the exchange right after `UDM_WORKFLOW_PROTOCOL.md` was committed) — but that was judgment, not a rule it's obligated to follow every time. A principle that only binds by convention for one entity and by habit for the others isn't actually uniform, and habit is exactly the kind of thing that erodes under time pressure — the same failure mode §1 of the workflow protocol and §1 of the `@domain-placeholder` proposal both already name for a different kind of judgment call.

## 2. What's being proposed

Extend §6's operational clause to all three AI entities symmetrically:

> **At the start of any UDM-related exchange, each AI entity checks for its own stalled items before doing anything else, and names them plainly if found — not as a footnote, not folded into other work.** "Stalled" means: anything in that entity's own scope sitting at `READY` or `FEEDBACK_READY` for more than one exchange with no `HANDOFF` or `CONFIRM` moving it forward; for the `udm-exchange` session specifically, this also includes any document it holds knowledge of that a state-tracking entity (design chat) is operating on stale information about — e.g. treating something as still `READY` when it's actually already `COMMITTED` or further along.

Concretely, for each entity:
- **Design chat** — already has this in writing (§6, unchanged).
- **`udm-exchange` session** — before acting on a new request, checks: (a) anything in its own commit queue sitting unpushed, (b) whether the requester's (Ron's, or a relayed entity's) understanding of a document's state matches what's actually committed on `main`.
- **Workbench session** — before pushing new feedback/migration-plan updates, checks whether any of its own prior `FEEDBACK_READY` items (that it produced but that haven't been confirmed committed) are still outstanding.

## 3. Why this doesn't need new machinery

This is a **discipline addition, not a tooling addition** — same category as the feedback-freshness convention already in `README.md`. No new command, no new state. It just makes the existing §6 principle enforceable the same way for all three entities instead of one, closing the exact asymmetry that would otherwise let "it applies equally" be true in principle and false in practice.

## 4. Open question for review

Should this be folded directly into `UDM_WORKFLOW_PROTOCOL.md` §6 (replacing the design-chat-only clause with the symmetric one above), or kept as a separate short addendum section (§6.1) so the original §6 text and its authorship history stay intact? No strong preference from this session — whichever design chat finds easier to maintain going forward.

---

*This proposal is procedural, not a claim about PKM/Architecture Guidance content — it only touches how the four entities coordinate with each other, which `UDM_WORKFLOW_PROTOCOL.md` already governs.*
