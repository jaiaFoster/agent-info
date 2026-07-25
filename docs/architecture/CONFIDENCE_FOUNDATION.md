# Confidence Foundation — SPRINT-004

`core/confidence/foundation.py`, `ALGORITHM_VERSION = "SPRINT-004-CONF-v1"`.

## Why this is a separate module from opportunity scoring

`core/intel/seller_pressure.py` answers "how strong is this opportunity."
This module answers "how sure are we." A CRITICAL-readiness seller
pressure score built from one stale, unreliable signal must read as
high-strength/low-confidence — never silently blended into one number.
Confidence output never carries an opportunity `score` field, and
opportunity-score output is never gated on confidence inside the scoring
function itself (gating happens one layer up, in
`core/simulation/policy.py`, as an explicit policy choice).

## Dimensions computed (v1)

| Dimension | How it's computed | Honesty note |
|---|---|---|
| `evidence_count` | `len(signals)` | — |
| `independent_source_count` | distinct `source_adapter` values | **Caveat, stated in the output itself**: this is a coarse proxy for true source independence until per-connector tagging propagates onto `Signal` rows (today every Hillsborough signal shares `source_adapter="real_estate"`). |
| `evidence_freshness` | mean of per-signal exponential decay, 180-day half-life | Unknown `detected_at` decays as one half-life old — conservative, never treated as fresher than known evidence. |
| `source_reliability` | weighted average from `SOURCE_RELIABILITY` (0.85 direct public record, 0.6 derived projection, 0.5 default for unknown adapters) | Unknown adapters get the **neutral default, not zero** — missing data is never treated as negative evidence. |
| `corroboration` | count of distinct evidence types present | — |
| `contradictions` | same `signal_type`, different sources, value spread > 0.3 within the same evidence set | Heuristic v1, explicitly labeled as such in the output — not a validated conflict detector. |
| `missing_expected_evidence` | `COMPANION_EXPECTATIONS` lookup (e.g. `foreclosure` commonly co-occurs with `lien`) | Informational only — absence is surfaced, never silently penalized. |
| `identity_resolution_strength` | caller-supplied `identity_resolution_status` (`resolved`/`ambiguous`/`unresolved`), mapped to `{1.0, 0.5, 0.0}` | Defaults to `"resolved"` (the common case in this codebase's deterministic-identity model). |
| `score_model_version` | `ALGORITHM_VERSION` constant | Always known, trivially. |
| `jurisdiction_coverage` | caller-supplied `jurisdiction_codes`, else the literal string `"not_computed_v1"` | `Signal` rows don't carry jurisdiction yet — rather than fabricate a number, this dimension honestly reports it isn't computed until that data exists. |
| `temporal_decay` | `FRESHNESS_HALF_LIFE_DAYS` constant (180) reported alongside `evidence_freshness` | — |

## Formula (v1, documented, not tuned/validated)

```
composite = 0.25*evidence_volume_score + 0.20*corroboration_score
          + 0.20*freshness + 0.20*reliability + 0.15*identity_strength
          - contradiction_penalty  (0.15 per contradiction, capped at 0.45)
```

`evidence_volume_score = min(count/4, 1.0)`,
`corroboration_score = min(distinct_types/3, 1.0)`.

**Explicit single-signal cap**: one uncorroborated data point cannot reach
HIGH confidence regardless of how fresh or reliable it is — `composite` is
capped at `0.4` whenever `evidence_count == 1`. Stated as an explicit rule
(matching the project's "formulas must be explicit" principle) rather than
left as an emergent side effect of the weights above.

Thresholds: HIGH ≥ 0.75, MEDIUM ≥ 0.45, else LOW.

## `is_calibrated_probability: False`

Always present, always `False`, in every `compute_confidence()` output.
This index is an uncalibrated, versioned heuristic — never a validated
probability of anything — until (if ever) a calibration study is done
against real outcomes. Downstream consumers (`core/simulation/policy.py`)
must not present it as one.

## Where it's wired in today

`core/scoring/targeted_recalculation.recompute_seller_pressure_for_
subjects()` and `scripts/run_seller_pressure.py`'s `compute_all()` both
attach `confidence_components` into the persisted `ScoreModel.explanation`
JSON, so every seller-pressure score — whether produced by the full-batch
runner or the new targeted path — carries the same confidence breakdown.
`core/simulation/policy.py` reads it for threshold filtering.

## Simulation guide (core/simulation/policy.py)

Read-only; never imports `core.leadgen` (the only code path that can reach
a real skip-trace/DNC provider) — a structural, not runtime-checked,
guarantee. Never mutates outreach/lead state.

```python
from core.simulation.policy import PolicyThresholds, simulate_policy, compare_policies

result = simulate_policy(session, PolicyThresholds(
    name="conservative",
    min_pressure_score=0.5,
    min_confidence_index=0.6,
    min_independent_source_count=2,
    max_contradictions=0,
    require_no_missing_expected_evidence=False,
    hypothetical_unit_cost=25.0,   # a caller-supplied guess, never a real quote
))
# result.qualifying_count, result.estimated_spend, result.by_jurisdiction,
# result.top_missing_evidence_reasons, result.cohort_opportunity_ids

compare_policies(session, [policy_a, policy_b])  # side-by-side, same cohort data
```

A row missing `confidence_components` entirely (scored before this sprint,
or by a path that hasn't been updated) is **excluded** from any
confidence-based threshold, never silently passed or failed — see
`core/simulation/policy._passes()`. `estimated_spend` is `None` unless a
`hypothetical_unit_cost` is explicitly supplied, and its
`estimated_spend_basis` string always states plainly that it's an estimate
from a supplied cost, not a measured or quoted price.
