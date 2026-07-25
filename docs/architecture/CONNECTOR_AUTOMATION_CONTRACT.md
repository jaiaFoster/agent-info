# Connector Automation Contract — SPRINT-004

The generic, jurisdiction-agnostic metadata and lifecycle model that lets
the scheduler (`ingestion/scheduler.py`) decide when a connector should run
without any per-county branching. Extends `SourceDescriptor`
(`adapters/real_estate/sources/connectors.py`) rather than introducing a
parallel registry, per this sprint's "extend, don't duplicate" rule — every
field below already lives on that dataclass and is loaded from each
jurisdiction's `docs/sources/*_public_sources.json` file via
`adapters/real_estate/sources/public_registry.load_all_source_registries()`.

## Identity

| Field | Meaning |
|---|---|
| `source_id` | Existing SPRINT-002 identity key. |
| `connector_id` (`effective_connector_id` property) | Defaults to `source_id`; may be overridden when a connector needs a distinct automation identity from its registry `source_id` (used today for `hillsborough_clerk_official_records_daily_indexes` → `hillsborough_clerk_dpm`). |
| `connector_version` | String, default `"1"`. Bump when the connector's automation behavior (not just its parser) changes materially. |
| `jurisdiction`, `source_owner`, `source_type` | Unchanged from SPRINT-002. |

## Lifecycle

```
planned -> experimental -> validation -> active <-> degraded -> disabled -> retired
                        \-> blocked <-/
```

States: `planned`, `experimental`, `validation`, `active`, `degraded`,
`blocked`, `disabled`, `retired` (`LIFECYCLE_STATES`). Only `validation`,
`active`, and `degraded` are executable (`can_execute()`) — `degraded`
still runs (it means "had trouble last time," not "must not run").
`LIFECYCLE_TRANSITIONS` defines the allowed edges; `can_transition()`
checks them. This is a distinct field from the pre-existing free-text
`status` (which still holds onboarding-readiness prose like
`"schema_verified_pending_production_approval"` — unchanged, for backward
compatibility).

## Scheduling

`expected_cadence_hours`, `preferred_run_window`,
`polling_frequency_minutes`, `schedule_timezone`, `retry_max_attempts`,
`backoff_seconds`, `max_runtime_seconds`, `freshness_threshold_hours`.
Consumed by `ingestion/scheduler.next_run_at()` / `is_due()` /
`backoff_until()` / `active_lock()`.

## Acquisition

`acquisition_mode` (`"incremental"` or `"full_export"`), `cursor_type`,
`checkpoint_strategy`, `expected_format`, `expected_size_range`,
`authentication_requirement`, `terms_or_license_reference`,
`paid_or_free`, `access_restrictions`. `acquisition_mode` distinguishes the
two checkpoint strategies documented in
`docs/architecture/EVIDENCE_EVENT_MODEL.md`'s sibling doc, the incremental
ingestion runbook.

## Processing

`parser`, `schema_version`, `identity_strategy`, `emitted_entity_types`,
`emitted_evidence_types`, `affected_score_types`, `supported_run_modes`
(`("dry_run",)` by default; must include `"write"` before a connector can
ever be driven into a real DB write through the generic scheduler).

## Validation

`validate_connector(descriptor) -> list[str]` (empty = valid) checks:
unknown lifecycle/acquisition-mode values, unknown `supported_run_modes`,
`ingest_supported` required before `"write"` can appear in
`supported_run_modes` (the Pre-Ingestion Source Rule, enforced a second
time at the scheduler call site — see below — since a registry
declaration alone is not itself an enforcement mechanism), non-negative
retry/backoff, positive cadence/runtime. Every currently-registered
connector across every onboarded jurisdiction validates cleanly
(`tests/test_connector_contract.py::RealRegistryValidationTests`).

**Note on `database_write_allowed`:** this field does not, on its own,
imply `ingest_supported`. `hillsborough_clerk_instrument_detail` is a real,
approved (2026-07-18) counter-example — a bounded enrichment write path
with its own script, not the generic per-source ingestion dispatcher.
`validate_connector()` does not flag this; only a `"write"` entry in
`supported_run_modes` without `ingest_supported` is flagged.

## Minimum requirements for a new connector

1. A registry JSON entry (`docs/sources/<jurisdiction>_public_sources.json`)
   with at least: `source_id`, `jurisdiction`, `lifecycle_state`,
   `acquisition_mode`, `expected_cadence_hours` (or document why none
   applies), and — if it will ever write — `supported_run_modes` including
   `"write"` only once `ingest_supported: true` is genuinely approved.
2. A `CONNECTOR_ADAPTER_SOURCE` entry in `ingestion/scheduler.py` mapping
   `connector_id -> (adapter, source)`, so the scheduler can call the
   existing, uniform `ingestion.service.run_ingestion(adapter, source,
   mode, ...)` entrypoint — no new dispatcher branch, no scheduler-level
   county-specific code.
3. If the connector is `acquisition_mode: "full_export"`, a
   `compute_fingerprint` callable for `scripts/run_scheduler.py`'s
   `FINGERPRINT_FUNCTIONS` table (see the incremental-ingestion runbook).
4. `validate_connector()` passes with zero errors.
