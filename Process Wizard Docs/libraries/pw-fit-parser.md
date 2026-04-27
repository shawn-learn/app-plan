# `pw-fit-parser`

## Purpose

A pure Python conversion library that parses Garmin `.fit` binary files into a normalized, sport-agnostic activity record. Produces two outputs: an **activity summary** (sport, device, start time, duration, distance, elevation gain, average/max HR, average/max power, average pace) and a **time series** (per-second or per-record stream of timestamp, HR, power, cadence, lat/lon, altitude). The normalized record is the input contract for `pw-training-metrics` and for the FIT upload endpoint in the app.

**Not responsible for:** computing training load or zone classifications (that is `pw-training-metrics`), storing data (the app does that), HTTP handling, or any UI.

---

## Status

**New** — no existing source in the repo. Wraps `fitparse` or `garmin-fit-sdk`.

---

## Source location

New package at:

```
packages/fit-parser/         ← Python package (not TypeScript)
├── pyproject.toml
└── src/pw_fit_parser/
    ├── __init__.py
    ├── parser.py            ← parse_fit_file() entry point
    ├── models.py            ← ActivitySummary, ActivitySample, FitActivity
    └── sports.py            ← sport enum + normalization mappings
```

---

## Public API / Exports

```python
from pw_fit_parser import parse_fit_file, FitActivity, ActivitySummary, ActivitySample, Sport

activity: FitActivity = parse_fit_file(path_or_bytes)

@dataclass
class ActivitySummary:
    sport: Sport              # running | cycling | strength | swimming | other
    device: str | None
    start_time: datetime      # UTC
    duration_seconds: float
    distance_meters: float | None
    elevation_gain_meters: float | None
    avg_heart_rate_bpm: float | None
    max_heart_rate_bpm: float | None
    avg_power_watts: float | None
    max_power_watts: float | None
    avg_cadence_rpm: float | None
    avg_pace_seconds_per_km: float | None   # None for non-running sports

@dataclass
class ActivitySample:
    timestamp: datetime
    heart_rate_bpm: float | None
    power_watts: float | None
    cadence_rpm: float | None
    latitude: float | None
    longitude: float | None
    altitude_meters: float | None
    distance_meters: float | None
    speed_ms: float | None

@dataclass
class FitActivity:
    summary: ActivitySummary
    samples: list[ActivitySample]   # ordered by timestamp

class Sport(StrEnum):
    RUNNING = "running"
    CYCLING = "cycling"
    STRENGTH = "strength"
    SWIMMING = "swimming"
    OTHER = "other"
```

`parse_fit_file` accepts `str | Path | bytes | BinaryIO`.

---

## Dependencies

**External (pip):**
- `fitparse>=1.2` (or `garmin-fit-sdk`; decision TBD based on multi-sport coverage)

**Internal pw-* dependencies:** none.

---

## Out of scope

- Zone classification, ACWR, TSS, or any training-load math — `pw-training-metrics`
- Any database writes or HTTP calls
- Garmin Connect API integration (this library only parses local files)
- Real-time streaming or Bluetooth device pairing

---

## Acceptance criteria

1. `parse_fit_file(sample_running.fit)` returns a `FitActivity` with a non-null `summary.distance_meters`, `summary.avg_heart_rate_bpm`, and at least one `ActivitySample` with a non-null `heart_rate_bpm`.
2. `parse_fit_file(sample_cycling.fit)` returns `Sport.CYCLING` and non-null `avg_power_watts` when power data is present in the file.
3. `parse_fit_file(b"bad bytes")` raises a descriptive `FitParseError` (not a raw `fitparse` exception).
4. `parse_fit_file()` is pure: no file system writes, no DB, no HTTP.
5. Calling `parse_fit_file` twice on the same file produces identical results (deterministic, no side effects).
6. The package installs in a fresh virtualenv with only `fitparse` as a transitive dependency.

---

## Phase

**Phase A** — created as part of foundations; wired to `/api/fit/upload` in the app once the library is stable.
