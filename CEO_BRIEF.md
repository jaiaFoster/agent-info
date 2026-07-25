# CEO Brief — LUX

*Last updated: 2026-07-25. Derived from AGENTS.md; regenerate whenever the Ticket Registry's Current Project Status changes.*

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

## What needs a decision from you right now

See `OPEN_DECISIONS.yaml` for the full, structured list. Highlights:
- Three new proposals from SPRINT-001's research (a cheap new vacancy
  signal, a heir/inherited-property signal, and a proposed `core/` schema
  change for multi-jurisdiction support) are all sitting unscoped, awaiting
  your call.
- **Pinellas production approval**: schema verification is now done and a
  real bounded ingestion succeeded cleanly (SPRINT-003) — the only thing
  blocking a real production ingestion run is your explicit go-ahead, plus
  resolving the Kyle/INGEST-002 ownership overlap above.
- **Two new Maricopa decisions** (SPRINT-003 readiness assessment, no code
  written yet): (1) Maricopa identifies parcels by APN, not FOLIO/PIN/STRAP
  — the shared linker code needs a decision on how to extend for that before
  any Maricopa adapter work starts; (2) Maricopa's recorder/deed data is a
  paid purchase, unlike every free recorder source used so far — decide
  whether to fund it or scope Maricopa's first pass as parcel-only.
- **This mirror repository itself (OPS-SYNC-001)**: source-side work
  (workflow, artifacts, docs) is ready in a PR awaiting your merge;
  `jaiaFoster/agent-info` has been bootstrapped with the initial artifact
  set so the first automated sync has a `main` branch to update.

## Standing rules that don't change

Jaia merges every PR to the canonical repository. No agent — human-directed
AI or otherwise — self-merges, ever. This mirror repository
(`jaiaFoster/agent-info`) is a read-only, automatically generated snapshot for
AI/executive inspection; it is not a place for direct development, and
nothing here is a substitute for reading `AGENTS.md` in the canonical
repository when precision matters.
