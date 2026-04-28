# Activity JSON Schemas

This folder defines versioned JSON schemas for activity templates and training plans used by the Organizer system.  Version `v1` covers five activity types plus a plan schema:

- `workout`
- `agenda`
- `list`
- `form`
- `log`
- `plan`

The schemas live under `schemas/activity/v1/` and follow [JSON Schema draft‑07](https://json-schema.org/).  Each schema contains a `version` field so that future revisions can coexist.

Workouts and training plans can include optional `milestones` arrays. Each milestone uses the shared `milestone` definition from `common.schema.json` and maps to the `planned_milestones` and `milestone_log` tables.

Use these schemas to validate template JSON before saving it to the database or sending it to the frontend.  Example:

```python
import json
from jsonschema import validate

with open('schemas/activity/v1/workout.schema.json') as f:
    workout_schema = json.load(f)

validate(instance=my_workout_json, schema=workout_schema)

with open('schemas/activity/v1/plan.schema.json') as f:
    plan_schema = json.load(f)

validate(instance=my_plan_json, schema=plan_schema)
```

An additional helper schema, `measurement.schema.json`, defines a rich
set of measurement types (weight, distance, duration, intensity levels, etc.) that can be
referenced from workout templates. It lives in the repository root under
`schemas/` and may be loaded alongside the activity schemas when performing
validation. The `duration` measurement accepts a single numeric value or a
numeric range (`min`/`max`) with units of `sec`, `min`, or `hrs`.

Measurements and their available units are also stored in the database
(`measures` and `units` tables). Values are persisted using the base unit
defined for each measure and converted at input/output time.

New versions should be added as subdirectories under `schemas/activity/` (e.g. `v2`).

