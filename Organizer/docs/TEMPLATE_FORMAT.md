# TEMPLATE_FORMAT.md

## 🎯 Purpose

This document defines a structured JSON format for workout templates in the Organizer system. It supports:

- Single workouts
- Recurring workout periods
- Multi-phase periodized plans

### Key Features

- **RRULE-style recurrence** for clarity and structure (RFC 5545-inspired)
- **Nestable set groups** (e.g., supersets within circuits)
- **Standardized intensity targets** for automated load suggestions
- **Support for diverse modalities**: strength, cardio, assisted, banded exercises
- **Input field system** for dynamic template instantiation
Templates reference exercises by their `name` or any alias.
The `/exercises/lookup` endpoint resolves these names against the `exercise_aliases` table, keeping templates independent of database IDs.

---

## ⏰ Recurrence Syntax

We use a simplified recurrence format based on RFC 5545 RRULE:

```json
"repeat": {
  "freq": "WEEKLY",
  "interval": 2,
  "byweekday": ["MO", "WE"],
  "count": 0
}
```

This supports:

- `"every Monday"` → `{"freq": "WEEKLY", "interval": 1, "byweekday": ["MO"]}`
- `"every 2 weeks on Tuesday and Thursday"` → `{"freq": "WEEKLY", "interval": 2, "byweekday": ["TU", "TH"]}`

Phrases may be used in the template and converted to RRULEs before planning.

---

## 📏 Units & Exercise Attributes

Exercises support context-specific fields:

| Type     | Fields                                         |
|----------|------------------------------------------------|
| Strength | `sets`, `reps`, `intensity_target`, `note`     |
| Cardio   | `duration_min`, `distance_km`, `pace_min_per_km`, `pace_min_per_mi` |
| Bands    | `resistance_color`                             |
| Assisted | `assist_percent`                               |
| Other    | `tempo`, `rest_sec`, `note`                    |

---

## 🧠 Intensity Targets

Use the `intensity_target` field to provide structured load guidance:

```json
"intensity_target": { "type": "percent_1rm", "value": 75 }
```

or

```json
"intensity_target": { "type": "rpe", "value": 7 }
```

If 1RM is known, system computes: `weight = 1RM × (value / 100)`

Use `note` for additional cues like `"explosive reps"` or `"controlled tempo"`.

---

## 🧩 Nestable Set Groups

You can nest `superset` and `circuit` groups with `rounds` that may be static or dynamic (`{{variable_name}}`).
Each set group can be given a **label** (default is `"Sets"`) so you can rename it to things like `"Circuit"` or `"Superset"`.

```json
{
  "superset": {
    "rounds": 3,
    "exercises": [
      { "name": "Pull‑Up", "sets": 1, "reps": 8 },
      {
        "circuit": {
          "rounds": "{{rounds_per_circuit}}",
          "exercises": [
            { "name": "Push‑Up", "sets": 1, "reps": 15 },
            { "name": "Band Row", "sets": 1, "reps": 12, "resistance_color": "red" }
          ]
        }
      }
    ]
  }
}
```

---

## 🧾 Hand‑Writing Guide

1. Pick `template_type`: `"single"`, `"recurring"`, `"periodized"`
2. Add `title`, `description`, `input_fields`
3. Use RRULE-style `repeat` to define workout recurrence
4. Build workouts with either:
   - Plain exercises
   - Nested `superset` or `circuit` blocks
5. Use `note` and `intensity_target` to guide load without fixed weights
6. Validate structure & ensure any `{{variable}}` is in `input_fields`

---

## ✅ Example: Full Periodized Plan with Nesting, Cardio, and Intensity

```json
{
  "template_type": "periodized",
  "title": "12‑Week Mixed Strength & Cardio",
  "input_fields": ["start_date", "rounds_per_group", "rounds_per_circuit"],
  "periods": [
    {
      "name": "Base – Weeks 1–6",
      "duration": { "value": 6, "unit": "weeks" },
      "workouts": [
        {
          "name": "Strength Circuit",
          "repeat": { "freq": "WEEKLY", "interval": 1, "byweekday": ["MO", "TH"] },
          "superset": {
            "rounds": "{{rounds_per_group}}",
            "exercises": [
              {
                "name": "Squat",
                "sets": 1,
                "reps": 8,
                "intensity_target": { "type": "percent_1rm", "value": 75 },
                "note": "Controlled descent"
              },
              {
                "circuit": {
                  "rounds": "{{rounds_per_circuit}}",
                  "exercises": [
                    { "name": "Push‑Up", "sets": 1, "reps": 15 },
                    {
                      "name": "Band Row",
                      "sets": 1,
                      "reps": 12,
                      "resistance_color": "red"
                    }
                  ]
                }
              }
            ]
          }
        },
        {
          "name": "Cardio Day",
          "repeat": { "freq": "WEEKLY", "interval": 1, "byweekday": ["WE"] },
          "exercises": [
            {
              "name": "20 Min Run",
              "duration_min": 20,
              "note": "Zone 2 steady pace"
            },
            {
              "name": "Bike Ride",
              "distance_km": 40,
              "duration_min": 120,
              "note": "Endurance focus"
            }
          ]
        }
      ]
    },
    {
      "name": "Build – Weeks 7–12",
      "duration": { "value": 6, "unit": "weeks" },
      "workouts": [
        {
          "name": "Heavy Squat",
          "repeat": { "freq": "WEEKLY", "interval": 2, "byweekday": ["TU"] },
          "exercises": [
            {
              "name": "Squat",
              "sets": 5,
              "reps": 5,
              "intensity_target": { "type": "percent_1rm", "value": 85 },
              "note": "2 min rest"
            },
            {
              "name": "Assisted Dip",
              "sets": 3,
              "reps": 10,
              "assist_percent": 50,
              "note": "Use green band"
            }
          ]
        }
      ]
    }
  ]
}
```

---

## 🔗 Integration Notes

- Validate with JSON schema or Pydantic models
- Recurrence expansion: use `dateutil.rrule` or similar
- At log time: resolve `intensity_target` to auto-fill weights
- Show notes and auto-suggested values on workout UI
