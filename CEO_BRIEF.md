# CEO Brief — LUX

*Last updated: 2026-08-12 (SPRINT-020). Derived from AGENTS.md and verified against live Railway production.*

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
- Revenue packet: SPRINT-020 v4 decision surface is being refreshed after
  fidelity, BatchData resilience, and scoring policy work.
- Skip-trace shadow: validated read-only with mock providers; no paid provider
  calls and no outreach.
- Fidelity: restored to `FIDELITY_OK` after SPRINT-020 repair.
- BatchData: retry/backoff, deterministic request identity, sanitized errors,
  and shadow-safe observability are implemented; live shadow remains no-paid-call.
- Scoring readiness: data-integrity readiness and statistical/sample readiness
  are now separated; thresholds were not lowered.
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
2. Statistical scoring sample gate: production graph coverage is real but still
   below the existing threshold (34/159 linked Transactions, 21.38%).
4. `SKIP-PILOT-002`: founder authorization is required before any paid
   provider purchase.
5. `FACT-LIVE-001/002`: canonical facts are absent in current Railway
   production and need a separate architecture decision if persisted facts
   should become a live consumer path again.

## Recommended next technical roadmap

1. Complete `REVENUE-QUALIFICATION-004` v4 packet and production smoke.
2. Implement `PROVIDER-DNC-001`.
3. If authorized after blocker clearance, run `SKIP-PILOT-002` as a bounded paid pilot; no automated
   outreach.

## Non-negotiables

No paid provider spend, outreach/contact action, destructive production
operation, identity-policy change, or scoring-policy change proceeds without
founder approval.
