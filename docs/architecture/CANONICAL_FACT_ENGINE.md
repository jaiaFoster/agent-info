# SPRINT-006 — Canonical Fact Engine

Status: foundational implementation PR.

Goal: parse deterministic evidence once, persist reusable facts once, let all
intelligence consumers query the same explainable knowledge layer.

## Boundary

Evidence states what a source said. Facts state what LUX can prove
deterministically from evidence. Derived facts are deterministic transforms of
facts. Scores and decisions are not facts.

```text
SourceRecord / RawEvidence
  -> asserted canonical_facts
  -> derived canonical_facts
  -> entity profiles
  -> confidence / buyer intelligence / matching / decisions
```

## Storage

Table: `canonical_facts`

Important fields:

- `fact_kind`: `asserted` or `derived`
- `subject_type`, `subject_id`
- `predicate`
- `value_type`, `value`
- `source_record_id`, `evidence_id`
- `input_fact_ids`
- `producer`, `producer_version`, `method`
- `jurisdiction_code`
- `observed_at`, `valid_from`, `valid_to`
- `confidence_class`
- `explanation`
- `idempotency_key`
- `status`, `supersedes_fact_id`, `superseded_by_fact_id`
- `lineage_json`, `metadata_json`

## Invariants

- Asserted facts require source evidence (`source_record_id` or `evidence_id`).
- Derived facts require input fact lineage.
- Exact replays are idempotent by `idempotency_key`.
- Same source/rule cannot emit two active facts for same subject/predicate
  without supersession.
- Corrections supersede; they do not silently overwrite old facts.
- Producers return facts; repositories write facts.
- Business scores and policy decisions never persist as facts.

## Producer model

Core owns:

- `core/facts/types.py`
- `core/facts/repository.py`
- `core/facts/producers.py`
- `core/facts/derived.py`
- `core/facts/profiles.py`
- `core/facts/consumers.py`

Real-estate producer rules live adapter-side:

- `adapters/real_estate/facts/producers.py`

Initial producers:

1. `real_estate.ownership_assertion`
2. `real_estate.transaction_asset_link`
3. `real_estate.transaction_party_role`
4. `real_estate.acquisition_activity`
5. `real_estate.legal_description_components`
6. `real_estate.financing_assertion`
7. `real_estate.entity_affiliation`
8. `real_estate.geography_membership`

## Initial derived facts

- `purchase_count` from active `entity_acquired_asset` facts.
- `transaction_count` from active `transaction_relates_to_asset` facts.
- `active_months` from dated acquisition facts.
- `purchases_per_year` from dated acquisition facts.
- `median_days_between_purchases` from dated acquisition facts.
- `average_purchase_amount` from acquisition amounts >= 1000.
- `preferred_jurisdictions` from acquisition fact jurisdiction codes.
- `evidence_diversity_count` from acquisition raw evidence IDs.
- `last_transfer_date` from linked transaction facts.
- `current_ownership_duration_days` from latest transfer date.
- `ownership_chain_length` from linked transaction facts.
- `investor_purchase_count` from acquisition facts grouped by jurisdiction.
- `repeat_buyer_share` from acquisition buyer IDs grouped by jurisdiction.

Excluded deliberately:

- `cash_purchase_ratio` until financing evidence can prove cash/financed state.
- fuzzy geography or affiliation derivations.

## Consumer shadow path

`core/facts/consumers.py` exposes buyer-intelligence shadow helpers. This lets
legacy transaction parsing and fact-backed intelligence run side-by-side before
cutover.

The same module now exposes a confidence bridge:

- `fact_as_confidence_evidence(...)`
- `compute_confidence_from_facts(...)`
- `confidence_shadow_parity(...)`

This converts active canonical facts into the signal-like evidence shape used
by the shared confidence foundation. Facts remain reusable knowledge; the
confidence engine remains the consumer. The bridge is read-only and does not
change scoring/matching behavior.

No legacy consumer is removed in this foundational PR.

## Read-only audit

```bash
python3 scripts/audit_fact_engine.py --limit 25
```

Reports:

- fact inventory by predicate, subject type, producer
- orphan asserted facts
- derived facts missing lineage
- sample fact-backed profiles
- buyer-intelligence shadow parity rate
- confidence shadow parity rate

Buyer-intelligence runner sidecar report:

```bash
python3 scripts/run_buyer_intelligence.py --fact-shadow-report data/output/buyer_fact_shadow.json
```

The normal buyer-intelligence artifact and score behavior stay unchanged unless
this optional read-only report flag is passed.

## Operational command

Dry-run:

```bash
python3 scripts/run_fact_engine.py --max-records 100 --include-derived
```

Write:

```bash
python3 scripts/run_fact_engine.py --write --max-records 100 --include-derived
```

Write mode creates canonical facts only. It does not send outreach, activate
paid providers, or modify scores/matches.

## First-write review packet

Before the first production fact write, generate a read-only packet:

```bash
python3 scripts/run_fact_engine.py --review-packet --include-derived --max-records 1000
```

This packet projects asserted facts, previews derived facts from current active
facts plus eligible projected asserted facts, and records safety flags showing
that no DB mutation, score write, match write, outreach, paid provider call, or
consumer cutover occurred. This closes the review gap where a dry-run could
show asserted fact counts while derived facts remained invisible until after a
write.
