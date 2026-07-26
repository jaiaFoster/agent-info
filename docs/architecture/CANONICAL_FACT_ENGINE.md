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

## Consumer shadow path

`core/facts/consumers.py` exposes buyer-intelligence shadow helpers. This lets
legacy transaction parsing and fact-backed intelligence run side-by-side before
cutover.

No legacy consumer is removed in this foundational PR.

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
