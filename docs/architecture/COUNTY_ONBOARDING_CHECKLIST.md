# County Onboarding Checklist — DATA-015F

Formalizes `docs/architecture/CONNECTOR_AUTOMATION_CONTRACT.md`'s existing
"Minimum requirements for a new connector" section into the full
nine-item checklist DATA-015 asks for. Hillsborough is the reference
implementation every future jurisdiction (Pinellas, Maricopa, Rensselaer,
...) should be walked through the same way, rather than reverse-engineered
from Hillsborough's code each time.

## The nine items

| # | Item | How it's verified |
|---|---|---|
| 1 | `source_inventory_complete` | **Manual.** A DATA-015B-style document exists covering every relevant public-record category for the jurisdiction (see `docs/research/DATA-015B-hillsborough-source-inventory.md` for the Hillsborough reference). Not derivable from registry/DB state — nothing in the system can confirm a *human* looked broadly enough. |
| 2 | `licensing_review_complete` | **Manual**, partially informed by each source's `requires_terms_review` flag in the registry (a source flagged `true` there has *not* had this confirmed complete by that flag alone — it's a prompt to review, not a review record). |
| 3 | `connector_registered` | **Computed.** At least one `SourceDescriptor` exists in the jurisdiction's `*_public_sources.json`, reachable via `docs/sources/jurisdictions.json`. |
| 4 | `parser_complete` | **Computed.** At least one registered connector has a non-empty `parser` field. |
| 5 | `canonical_mapping_complete` | **Computed.** At least one registered connector declares `canonical_outputs`. |
| 6 | `deterministic_validation_complete` | **Computed.** `validate_connector()` returns zero errors for every registered connector (see `adapters/real_estate/sources/connectors.py`). |
| 7 | `health_monitoring_enabled` | **Computed, and automatic by construction.** `core/api/ops_queries.ops_sources()` / `ops_connector_health()` walk `load_all_source_registries()` with zero per-jurisdiction branching — any registered connector is visible the moment it's registered. There is no separate "wire up health monitoring" step to forget. |
| 8 | `dashboard_visibility_enabled` | **Computed**, same reasoning as #7. |
| 9 | `documentation_generated` | **Computed.** `scripts/generate_source_registry_docs.py` renders every onboarded jurisdiction's registry into `docs/sources/SOURCE_REGISTRY.md` automatically — nothing jurisdiction-specific to author by hand. |

Items 3-9 are computed live by `core.api.ops_queries.onboarding_checklist(jurisdiction_code)`,
exposed read-only at `GET /api/ops/onboarding_checklist?jurisdiction_code=<code>`.
Items 1-2 report `None` ("not computable from system state") rather than
a guessed `true`/`false` — per this project's "unresolved is better than
false resolution" principle, a checklist that can lie about a process step
it can't actually observe is worse than one that honestly says "check
manually."

## Reference state (real, checked 2026-07-28)

- **FL:HILLSBOROUGH**: `connector_registered` through `documentation_generated`
  all true (12 registered connectors, 0 validation errors). Items 1-2
  confirmed manually — `docs/research/DATA-015B-hillsborough-source-inventory.md`
  is the source-inventory record; per-source `requires_terms_review` flags
  are set where applicable.
- **FL:PINELLAS**: registered (2 connectors, parcels + recorder probe,
  `lifecycle_state: validation`), computed items true; not yet production-approved
  (`status: schema_verified_pending_production_approval` in
  `jurisdictions.json` — a status this checklist doesn't override).
- **AZ:MARICOPA**, **NY:RENSSELAER**: `status: planned`, no
  `source_registry` file yet — `connector_registered` and everything
  downstream of it correctly report `false`.

## Using this for a new jurisdiction

1. Do the source inventory (item 1) the way DATA-015B did it for
   Hillsborough — one document, every relevant category, real sources
   only, honest `not_started`/`unavailable` where nothing real was found.
2. Add an entry to `docs/sources/jurisdictions.json` and a
   `docs/sources/<jurisdiction>_public_sources.json` registry file
   (items 3-4-5).
3. Run `python3 -m adapters.real_estate.sources.connectors` validation (or
   just check `GET /api/ops/onboarding_checklist`) until item 6 is true.
4. Items 7-9 are then already true — there is nothing left to do for them.
5. Confirm licensing (item 2) per-source as connectors are built out.
