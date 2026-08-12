# CEO Brief — LUX

*Last updated: 2026-08-12 (STATE-RECONCILE-001). Derived from AGENTS.md and verified against live Railway production.*

## What LUX is

LUX turns fragmented public evidence into explainable market decisions:
canonical evidence, seller/buyer intelligence, ranked matches, and
human-reviewed outreach preparation. Residential real estate in Hillsborough
County remains the reference vertical for a reusable market-intelligence core.

## Current verified state

SPRINT-016, SPRINT-017, SPRINT-018, and SPRINT-019 are complete as technical
workstreams. Production is no longer in the stale “buyer intelligence empty /
zero matches” state that older roadmap files described.

Live production verified on 2026-08-12:

- Health: `/api/health` returned `ok`.
- Graph: 34/159 Transactions linked to Assets (21.38%), classification
  `GRAPH_PARTIAL_LINK_COVERAGE`.
- Evidence: 560,102 SourceRecords, 11,255,764 Observations, 53 RawEvidence
  rows, 82 PipelineRuns.
- Seller opportunity evidence: 69 financial-opportunity subjects, 821
  financial-opportunity observations, 28 Asset-linked financial observations.
- Buyer intelligence: 57 scored buyers.
- Matching: 957 persisted/quality-analyzed matches.
- Candidate ranking: 957 candidate matches surfaced for bounded review.
- Revenue packet: recommendation is
  `proceed_with_smaller_pilot_after_blockers_cleared`.
- Skip-trace shadow: validated read-only with mock providers; no paid provider
  calls and no outreach.
- Canonical facts: current Railway production reports `canonical_facts=0`;
  this remains an audit/follow-up item, not an active consumer blocker unless a
  live consumer is moved back to persisted facts.

## The milestone that matters

MVP-001 — First Dollar: close one wholesale transaction sourced and matched
through LUX. A qualified lead or outreach packet is an intermediate step; the
milestone is a closed transaction.

## Current blockers

1. `PROVIDER-DNC-001`: DNC.com/DNCScrub adapter is still the hard compliance
   blocker before any paid skip-trace or callable-contact flow.
2. BatchData resilience: shadow validation found no retry/backoff path for
   transient provider failures or HTTP 429.
3. Scoring-readiness policy: production graph coverage is real but still below
   the existing threshold (34/159 linked Transactions, 21.38%).
4. `SKIP-PILOT-002`: founder authorization is required before any paid
   provider purchase.
5. `FACT-LIVE-001/002`: canonical facts are absent in current Railway
   production and need a separate architecture decision if persisted facts
   should become a live consumer path again.

## Recommended next technical roadmap

1. Implement `PROVIDER-DNC-001`.
2. Add BatchData retry/backoff resilience.
3. Resolve the scoring-readiness threshold decision with founder input.
4. Re-run the bounded revenue-pilot decision packet with fresh metrics.
5. If authorized, run `SKIP-PILOT-002` as a bounded paid pilot; no automated
   outreach.

## Non-negotiables

No paid provider spend, outreach/contact action, destructive production
operation, identity-policy change, or scoring-policy change proceeds without
founder approval.
