# CEO Brief — LUX

*Last updated: 2026-07-25 (SPRINT-005-GOV governance reconciliation). Derived from AGENTS.md; regenerate whenever the Ticket Registry's Current Project Status changes.*

*For a visual, diagrams-first snapshot (geography, roadmap, engine/intelligence maturity, current milestone), see [`PROJECT_PROGRESS.md`](PROJECT_PROGRESS.md). This document remains the narrative brief and decision log.*

## What LUX is

LUX (Linked Unified Exchange) is a market-intelligence platform: it turns
scattered public records into canonical evidence, scores that evidence into
seller and buyer intelligence, ranks buyer↔seller matches, and routes the
best matches into a human-reviewed outreach queue. Residential real estate
(Tampa Bay, FL) is the first validation vertical — **this is not a real
estate app**; real estate proves out a market-agnostic intelligence engine.

## Where we are

The infrastructure phase is complete and has been operationally proven in
production: canonical data model, public-record ingestion, immutable
provenance, deterministic Transaction→Asset linking, and read-only graph
queries are all live. Seller-pressure and buyer-demand scoring (INTEL-001,
INTEL-002) are live and generating ranked, persisted matches (MATCH-001).
Human-reviewed outreach drafting (OUTREACH-001) shipped 2026-07-22.

**The current bottleneck is not intelligence — it's real, compliant
contacts.** Outreach is currently running against mock contact providers.
The path to real contacts is the Lead Engine go-live sequence
(LEADGEN-002 through LEADGEN-005), which is the direct blocker on this
company's one named milestone.

## The one milestone that matters: MVP-001 — First Dollar

Close one real wholesale transaction entirely sourced and matched through
LUX. A qualified lead and an approved outreach packet are steps on this
path, not the milestone itself. Every ticket should be evaluated against:
does this move LUX closer to that first real transaction?

## What's next, in order

1. **LEADGEN-002** — run the production database migration (unblocks
   everything else in this list).
2. **LEADGEN-003** — record the lawful-use attestation (skip-trace is
   hard-blocked without it).
3. **LEADGEN-004** — provision paid skip-trace/DNC-scrub provider accounts
   and keys.
4. **LEADGEN-005** — activate live providers, run the first bounded live
   skip-trace batch.
5. **OUTREACH-003** — feed real, compliant contacts into the existing
   OUTREACH-001 approval queue.
6. **MVP-001** — the first actual closed transaction.

## Geographic expansion (SPRINT-001 findings, 2026-07-24)

Research validated the existing plan rather than changing it, and added
execution detail:
- **Florida**: keep expanding. Evidence-ranked next counties: Pinellas
  (already in progress), Orange, Pasco, Duval.
- **Phoenix (Maricopa County, AZ)**: keep as the first out-of-state
  architecture-validation target — evidence fully supports the original
  rationale.
- **Troy (Rensselaer County, NY)**: keep, but with an honest risk note —
  population has been declining since 2020 and it has the weakest data
  access (permits, probate) of any market researched. Frame it explicitly
  as a non-revenue architecture-validation bet, which is how the project
  already treats it.
- Deprioritized: Austin, TX and Salt Lake County, UT — both are
  non-disclosure states, which breaks LUX's evidence-based model
  regardless of their macro appeal.

**Update 2026-07-26:** Phoenix and Troy now have formal tickets —
`AZ-ADAPTER-001` and `NY-ADAPTER-001` (the latter's scope extended to
Albany County at Tim's request). Neither authorizes adapter code yet; both
are blocked on the same shared asset-linker identity-vocabulary decision
(Maricopa uses APN, NY uses Section-Block-Lot, and the shared linker has
no slot for either). Also added `MARKET-001`, a new, independent, unblocked
ticket for a national investor-purchase heatmap (free Redfin Data Center
quarterly data) to help prioritize which market to evaluate next.

## Pinellas expansion progress (SPRINT-002, 2026-07-25)

Built the first repeatable jurisdiction-onboarding framework and a full
Pinellas County parcel/assessor adapter (source-side only — production
ingestion not yet approved, zero `core/` changes, 450 tests passing with
zero Hillsborough regressions). **One thing needs your attention before
this goes further**: a Pinellas adapter stub already existed in the repo,
plausibly Kyle's own placeholder for his in-progress `INGEST-002` ticket
(same goal). This patch built a working implementation without knowing
whether Kyle has independent progress — that's a call for you and Kyle,
not something to resolve unilaterally. See `OPEN_DECISIONS.yaml`.

## Pinellas production validation (SPRINT-003, 2026-07-25)

Proved the framework against real, live Pinellas data instead of
assumptions. Corrected a real schema mistake from SPRINT-002 (the actual
join only needs 3 tables, not the 6 assumed before real data was
downloaded), fixed several real-world data defects (ZIP-wrapped responses,
malformed JSON, a required HTTP header the site silently 403s without), and
ran a real 200-parcel ingestion end-to-end in an isolated database — zero
integrity failures, verified idempotent on rerun. 464 tests passing, zero
`core/` changes, zero Hillsborough regressions. **No production database
write occurred** — `database_write_allowed`/`ingest_supported` remain
`false`, exactly as the ticket required, pending your explicit approval.
Also added a **Merge Authority** section to `AGENTS.md`: you are the sole
merge authority, and no ticket, agent, or automation can grant itself
merge rights, regardless of what its own fields claim. Produced a Maricopa
County readiness assessment (research only, no code) — see
`docs/research/MARICOPA_READINESS_ASSESSMENT.md` and the new decisions
below. Full packet: `docs/research/SPRINT-003-SUMMARY.md`.

## Continuous intelligence automation (SPRINT-004, 2026-07-25)

Built the automation layer that lets LUX keep itself current instead of
being manually re-run: a generic scheduler that decides when each source
should run (no per-county special-casing), safe incremental/full-export
ingestion with checkpoints, deterministic evidence events, targeted
rescoring (only affected sellers/buyers/matches recompute, not everything),
a versioned confidence layer kept explicitly separate from opportunity
strength, continuous re-ranking with preserved history, and a read-only
simulator that estimates how many opportunities would qualify for skip
tracing — and what it would cost — at different confidence thresholds,
**without ever calling a paid provider**. Proven against real, live
Hillsborough and Pinellas data end-to-end into an isolated database (no
production write). **This sprint also changed governance**: the prior
"only Jaia merges, ever" rule became a quality-gated, per-sprint auto-merge
policy, with paid/destructive/external actions still hard-gated on your
explicit approval regardless. You reviewed PR #55 manually and merged it
(2026-07-25) — the worker paused rather than unilaterally flipping GitHub's
branch-protection/auto-merge settings for a first-time governance change,
consistent with "default is manual merge unless explicitly granted."

## Governance & state reconciliation (SPRINT-005-GOV, 2026-07-25)

A documentation/governance-only sprint (no product or database changes):
removed several leftover contradictory "Jaia merges, never self-merge"
statements scattered across `AGENTS.md` that predated the SPRINT-004
policy change and were never cleaned up; restated the merge policy in one
clean, current form; updated the North Star and operating philosophy to
**Evidence → Intelligence → Confidence → Decision → Paid Identification →
Outreach → Revenue**, reflecting that infrastructure is done and the
current gap is intelligence/confidence calibration, not outreach
mechanics; reconciled stale state (SPRINT-004 shown as "PR open" when it
had already merged, a resolved `jurisdiction_code` decision still marked
"awaiting"); and added `PROJECT_PROGRESS.md`, a diagrams-first executive
dashboard. This sprint's own PR carries the same explicit,
self-scoped auto-merge grant SPRINT-004 introduced — see its ticket for
the exact conditions.

## What needs a decision from you right now

See `OPEN_DECISIONS.yaml` for the full, structured list. Highlights:
- Three new proposals from SPRINT-001's research (a cheap new vacancy
  signal, a heir/inherited-property signal) are still sitting unscoped.
- **Pinellas production approval**: schema verification is done and a
  real bounded ingestion succeeded cleanly (SPRINT-003) — the only thing
  blocking a real production ingestion run is your explicit go-ahead, plus
  the still-open (no longer blocking, but still undecided) Kyle/INGEST-002
  question below.
- **Kyle/INGEST-002**: the ownership-overlap question is no longer treated
  as a blocker on Pinellas work (SPRINT-004), but which implementation Kyle
  actually intended is still genuinely undecided whenever you two want to
  look at it.
- **Two Maricopa decisions** (SPRINT-003 readiness assessment, no code
  written yet): (1) Maricopa identifies parcels by APN, not FOLIO/PIN/STRAP
  — the shared linker code needs a decision on how to extend for that before
  any Maricopa adapter work starts; (2) Maricopa's recorder/deed data is a
  paid purchase, unlike every free recorder source used so far — decide
  whether to fund it or scope Maricopa's first pass as parcel-only.
- **This mirror repository itself (OPS-SYNC-001)**: live and auto-syncing;
  no action needed.

## Standing rules that changed, and what didn't

Default merge authority is yours; a sprint may explicitly grant a worker
quality-gated autonomous merge for itself (SPRINT-004, then restated
cleanly by SPRINT-005-GOV) — see AGENTS.md's Merge Authority section for
the exact conditions. What did **not** change: paid actions (skip tracing,
contact purchases, any provider spend), outreach/contact, production-
ingestion approval, destructive production database operations, and
identity-resolution-policy changes still always require your explicit
approval in chat, no matter what a sprint ticket's merge policy claims.
This mirror repository (`jaiaFoster/agent-info`) is a read-only,
automatically generated snapshot for AI/executive inspection; it is not a
place for direct development, and nothing here is a substitute for reading
`AGENTS.md` in the canonical repository when precision matters.
