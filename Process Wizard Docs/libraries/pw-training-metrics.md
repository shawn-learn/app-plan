# `pw-training-metrics`

## Purpose

A pure-functions Python library for sport-agnostic training-load mathematics. Takes normalized activity records (from `pw-fit-parser`) or raw numeric series and computes: HR/power/pace zone classification, session training load (TSS for cycling/running, eTRIMP, volume-load for strength), rolling weekly training load (WTL), chronic load (28-day rolling average), Acute:Chronic Workload Ratio (ACWR), zone distribution, and recovery status.

Generalizes the running-specific logic in `services/training_load.py` to work across all four modalities (running, cycling, strength, other endurance). Removes the SQLAlchemy dependency — the library operates on plain Python data structures; the caller provides pre-fetched values.

**Not responsible for:** fetching data from the database (the caller does that), storing results, HTTP, UI, or zone definition authoring (zone thresholds are injected by the caller or read from a config dict).

---

## Status

**New** — generalizes `backend/src/process_wizard/services/training_load.py` and removes the DB dependency. The existing `training_load.py` becomes a thin adapter that fetches data and calls this library.

---

## Source location (of existing logic to generalize)

```
backend/src/process_wizard/services/training_load.py
```

New package at:

```
packages/training-metrics/
├── pyproject.toml
└── src/pw_training_metrics/
    ├── __init__.py
    ├── zones.py          ← zone classification (HR, power, pace)
    ├── session_load.py   ← per-session TSS / eTRIMP / volume-load
    ├── rolling.py        ← WTL, chronic_load, ACWR, zone_distribution
    └── recovery.py       ← recovery_status_from_inputs, apply_nudge_vs_drag
```

---

## Public API / Exports

### Zone classification

```python
from pw_training_metrics import classify_hr_zone, classify_power_zone, classify_pace_zone

classify_hr_zone(
    heart_rate_bpm: float,
    thresholds: HrZoneThresholds,      # caller-supplied from decision table
) -> ZoneLabel                          # "zone_1" | "zone_2" | ... | "zone_5b"

classify_power_zone(watts: float, ftp_watts: float) -> ZoneLabel
classify_pace_zone(seconds_per_km: float, lt1_pace: float, lt2_pace: float) -> ZoneLabel
```

### Session training load

```python
from pw_training_metrics import compute_tss, compute_etrimp, compute_volume_load

compute_tss(
    duration_seconds: float,
    avg_power_watts: float,
    ftp_watts: float,
    normalized_power_watts: float | None = None,
) -> float

compute_etrimp(
    duration_minutes: float,
    avg_hr_bpm: float,
    hr_max: float,
    hr_rest: float,
) -> float

compute_volume_load(
    sets: list[SetRecord],   # {reps: int, load_kg: float}
) -> float
```

### Rolling aggregates (pure, no DB)

```python
from pw_training_metrics import (
    weekly_wtl, chronic_load_28d, compute_acwr,
    zone_distribution, recovery_status_from_inputs, apply_nudge_vs_drag,
)

weekly_wtl(session_loads: list[SessionLoad]) -> float
# SessionLoad = {date: datetime, load: float}

chronic_load_28d(session_loads: list[SessionLoad], as_of: datetime) -> float | None
# Returns None when history < 14 days

compute_acwr(weekly: float, chronic: float | None) -> float | None

zone_distribution(
    samples: list[ZoneSample],   # {zone: ZoneLabel, duration_seconds: float}
) -> ZoneDistribution            # {zone_1_min: float, ..., pct_easy: float, pct_hard: float}
```

### Recovery status

```python
from pw_training_metrics import RecoveryStatus, AcwrZone
from pw_training_metrics import recovery_status_from_inputs, apply_nudge_vs_drag

recovery_status_from_inputs(
    avg_motivation: float | None,
    avg_soreness: float | None,
) -> RecoveryStatus   # "green" | "yellow" | "red"

apply_nudge_vs_drag(
    acwr_zone: AcwrZone,
    recovery_status: RecoveryStatus,
    base_volume_adjustment_pct: float,
    deload_flag: bool,
) -> tuple[AcwrZone, float, bool]
```

---

## Dependencies

**External (pip):** none (stdlib only)

**Internal pw-* dependencies:** none. Accepts `pw-fit-parser` output shapes as plain dataclasses, but does not import `pw-fit-parser` — callers bridge between the two.

---

## Out of scope

- Database queries — callers fetch data and pass lists of plain records
- Zone threshold authoring or storage — zone configs are injected by the caller (typically from a GoRules decision table result)
- Any UI or HTTP code

---

## Acceptance criteria

1. `compute_acwr(weekly_wtl, chronic_load)` returns the same value as the current `training_load.py` for the existing test fixture.
2. `recovery_status_from_inputs` and `apply_nudge_vs_drag` produce identical results to the current implementations for all documented test cases.
3. `zone_distribution` correctly sums zone minutes and computes `pct_easy`/`pct_hard` for a known sample series.
4. All functions are pure: same inputs → same outputs; no file I/O, no DB, no network calls.
5. The package installs with only stdlib as transitive dependencies.
6. The existing running-program evaluator in `plugins/fitness/` works unchanged after `training_load.py` is refactored to delegate to this library.

---

## Phase

**Phase A** — extract/create alongside the foundational backend libraries.
