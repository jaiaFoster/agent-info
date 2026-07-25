# Evidence Event Model — SPRINT-004

Extends the existing `EventModel` (`core/db/models.py`) rather than
introducing a parallel "evidence event" concept. Before this sprint,
`EventModel` was already a generic, source-agnostic append record tied to
one `SourceRecord` and optionally one `Asset` — the only production usage
(`ingestion/service.py`'s `"county_document_recorded"` type) already matched
exactly what this sprint calls an "evidence event." All new columns are
nullable/defaulted so every pre-SPRINT-004 row remains valid unmigrated.

## New fields (all additive)

`connector_id`, `jurisdiction_code`, `raw_evidence_id` (FK to
`raw_evidence`), `effective_at`, `ingested_at`, `evidence_version`,
`material_change_reason`, `affected_asset_ids` (JSON list),
`affected_participant_ids` (JSON list), `dedup_key` (unique, nullable).

`affected_asset_ids`/`affected_participant_ids` are plain JSON arrays, not
a join table — deliberately: a bounded, evidence-driven event affects a
small number of subjects (usually one), and a join table would be
overengineering relative to what WP6's targeted-recompute queries
actually need (`core/scoring/targeted_recalculation.affected_subjects_
for_events()` just reads the arrays).

## Deduplication

`ingestion/evidence_events.compute_dedup_key()` hashes
`connector_id:source_record_id:event_type:material_change_reason:
evidence_version`. `emit_evidence_event()` looks up this key before
inserting — identical evidence re-ingested (a rerun, a replay) produces
zero new events, returning the existing row with `created=False`. This is
what makes event emission safe to call from a rerun without inflating
downstream targeted-rescore work.

## Emission

`ingestion/evidence_events.emit_events_for_pipeline_run()` is the generic,
connector-agnostic post-ingestion step: for every `SourceRecord` a given
`pipeline_run_id` created, it emits one evidence event, deriving
`affected_asset_ids` from that source record's own `Observation` rows
(`subject_type="asset"`) — both the Hillsborough and Pinellas parcel
adapters already tag `Observation.subject_id` with the Asset id, so this
works identically for either without a per-county branch. This is a
scheduler-integration step, layered on top of the existing ingestion
functions rather than modifying their internals (those functions are
tested, production-critical, and out of scope for this sprint's changes).

## Example event types

`deed_recorded`, `mortgage_recorded`, `lien_recorded`,
`foreclosure_signal_observed`, `code_case_opened`, `code_case_updated`,
`permit_issued`, `tax_delinquency_observed`, `parcel_attribute_changed`,
`owner_or_participant_evidence_changed`, `buyer_activity_observed`
(`KNOWN_EVENT_TYPES` — documented, not a DB-enforced enum, since
`event_type` has always been a free string and a new connector may
legitimately introduce a new one).

## Reading events

`ingestion/evidence_events.query_evidence_events()` — always bounded
(`limit`, clamped to 1000), filterable by `connector_id`,
`jurisdiction_code`, `event_type`, `asset_id`, `since`. No unbounded "all
events" call exists by design.

## Downstream consumption

`core/scoring/targeted_recalculation.EVENT_TYPE_TO_SCORE_FAMILIES` maps
each event type to the score families (`seller_pressure`, `buyer_demand`,
`match`) it can plausibly affect — see the targeted-rescore section of
`docs/research/SPRINT-004-SUMMARY.md` and
`core/scoring/targeted_recalculation.py`'s module docstring for the full
recompute pipeline this feeds.
