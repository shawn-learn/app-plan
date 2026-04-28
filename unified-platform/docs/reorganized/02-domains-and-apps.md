# 02 — Domains and App Composition

## Domain Responsibilities

### `domain-industrial`
- SOP/procedure step types
- Compliance workflows
- Signature-heavy execution UX
- Deviation/branching behavior

### `domain-organizer`
- Personal tasks/lists
- Habits
- Personal goals and lightweight planning overlays
- Calendar/scheduling glue for personal usage

### `domain-fitness`
- Exercise catalog
- Workout authoring/runtime
- Intake + assessment flows
- FIT parsing and training metrics
- Periodization and coach-athlete team workflows

## App-Shell Composition

### `app-industrial`
- Composes platform + `domain-industrial`
- Emphasizes tenancy, signatures, auditability, offline execution

### `app-organizer`
- Composes platform + `domain-organizer`
- Supports personal or light-tenant mode

### `app-coach`
- Composes platform + `domain-fitness`
- May optionally add `domain-organizer` for personal scheduling surfaces

## Shared Experience Guarantees

All apps share:
- Typed API boundary
- Authentication/session lifecycle
- Wizard execution model
- Template versioning and run records
- Optional sync and messaging seams
