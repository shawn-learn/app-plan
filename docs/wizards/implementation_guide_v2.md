# Process Wizard MVP — Implementation Guide (Revised)
## Form.io + GoRules + SQLite Stack

### How to Use This Guide

You're an experienced programmer directing Claude Code from your phone during the day and reviewing results on your desktop at night. Each step is designed as a **self-contained task** you can hand to a Claude Code cloud session.

For each step:
- **Prompt** — what to tell Claude (copy-paste or paraphrase)
- **What Claude delivers** — the expected output
- **What you verify at night** — your desktop review checklist
- **Branch name** — for the PR

Every step ends with "push to branch." You review and merge from your desktop.

---

## Prerequisites

Before starting, from your desktop:

1. Create a GitHub repo called `process-wizard`
2. Clone it locally
3. Commit these files to `main`:
   - `CLAUDE.md` (provided separately)
   - `docs/mvp_spec.md` (from our earlier session)
   - `docs/build_list.md` (from our earlier session)
   - `docs/implementation_guide.md` (this file)
   - `.claude/settings.json` (below)
   - `.github/workflows/ci.yml` (provided separately)
4. Push to GitHub
5. Open claude.ai/code, connect the repo, configure your environment

**`.claude/settings.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "which python3 > /dev/null && cd backend 2>/dev/null && pip install -e '.[dev]' --quiet 2>/dev/null; cd ../frontend 2>/dev/null && npm install --silent 2>/dev/null; echo 'Environment ready'",
        "timeout": 180
      }
    ]
  }
}
```

---

## Phase 1 — Foundation (Week 1)

### Step 1: Project Scaffolding

**Prompt:**
> Set up the project scaffolding per CLAUDE.md. Create:
>
> Backend (`backend/`):
> - `pyproject.toml` with Python 3.12+, dependencies: fastapi, uvicorn[standard], sqlalchemy[asyncio], aiosqlite, asyncpg, alembic, pydantic>=2.0, python-dotenv, zen-engine, httpx
> - Dev dependencies: pytest, pytest-asyncio, ruff, httpx
> - Source layout: `backend/src/process_wizard/` with subpackages: `api/`, `models/`, `engine/`, `db/`, `services/`
> - `main.py` with FastAPI app and health check at `/api/health`
> - `alembic.ini` configured for async SQLAlchemy
> - `.env.example` with DATABASE_URL defaulting to SQLite
>
> Frontend (`frontend/`):
> - Vite + React + TypeScript project
> - Dependencies: @formio/react, formiojs, react-router-dom
> - Dev dependencies: vitest, @testing-library/react
> - Tailwind CSS configured
> - Placeholder App.tsx rendering "Process Wizard"
>
> Root:
> - `docker-compose.yml` with Postgres 16 service only (for desktop testing)
> - `.gitignore` covering Python, Node, SQLite, .env
>
> Verify: start the backend with uvicorn, confirm health check returns 200. Start the frontend with vite dev, confirm it renders. Push to branch `step-01-scaffolding`.

**What Claude delivers:** Running backend on port 8000, running frontend on port 5173, all files committed.

**Verify at night:**
- [ ] Clone the branch, run `pip install -e '.[dev]'` — no errors
- [ ] `uvicorn process_wizard.main:app` — health check works
- [ ] `npm install && npm run dev` — frontend renders
- [ ] `pyproject.toml` has correct dependency versions (not outdated)
- [ ] Alembic configured for async: check `env.py` uses `run_async`

---

### Step 2: Pydantic Models — Core Types

**Prompt:**
> Create the Pydantic v2 models in `backend/src/process_wizard/models/`. Read `CLAUDE.md` for conventions (extra="forbid", snake_case, type hints).
>
> `enums.py`:
> - DocumentType: procedure, form, questionnaire, workout, assessment, checklist, plan, log
> - LifecycleStatus: draft, review, published, superseded, archived
> - RunStatus: in_progress, completed, abandoned
> - InputMethod: wizard, retroactive, voice, ai
> - HitPolicy: unique, first, collect
> Use Python StrEnum.
>
> `document.py`:
> - DocumentMetadata: author (str), status (LifecycleStatus), tags (list[str]), estimated_duration (str|None)
> - OutputRule: field (str), expression (str), condition (str|None) — expressions are Python expressions evaluated by the output evaluator, not JSON Logic
> - ReferenceableSpec: key (str), fields (list[str])
> - OutputSection: rules (list[OutputRule]), referenceable (ReferenceableSpec|None)
> - DocumentRecord: id (UUID), schema_version (str), version (str), type (DocumentType), title (str), status (LifecycleStatus), metadata (DocumentMetadata), formio_schema (dict) — the Form.io JSON schema, output (OutputSection|None), data_requirements (dict|None)
>
> `run.py`:
> - Run: id (UUID), document_id (UUID), document_version (str), user_id (UUID|None), status (RunStatus), input_method (InputMethod), collected_data (dict), overrides (dict|None), output_data (dict|None), parent_run_id (UUID|None), started_at (datetime), completed_at (datetime|None)
> - CompletionResult: output_data (dict), risk_classification (str), summary (dict)
>
> `reference.py`:
> - ReferenceEntry: id (int|None), user_id (UUID), reference_key (str), run_id (UUID), date (datetime), value (Any)
>
> `user.py`:
> - UserProfile: id (UUID), display_name (str), age (int|None), sex (str|None), bodyweight (float|None), height (float|None), preferred_units (str), metadata (dict|None), created_at (datetime), updated_at (datetime)
>
> Write tests in `backend/tests/test_models.py` verifying each model validates correctly and rejects extra fields.
> Push to branch `step-02-models`.

**Key change from original design:** The `DocumentRecord` model now holds a `formio_schema` dict (the Form.io JSON) rather than a list of custom `Item` objects. Form.io owns the item/component definitions — we don't re-model them in Pydantic. Our models focus on what's unique to Process Wizard: runs, references, output rules, and the document wrapper.

**Verify at night:**
- [ ] All models import cleanly
- [ ] `extra="forbid"` is on every model
- [ ] Tests pass
- [ ] OutputRule expressions are strings (Python expressions), not JSON Logic dicts

---

### Step 3: Database Schema

**Prompt:**
> Create the database schema using SQLAlchemy 2.0 ORM models and an Alembic migration.
>
> `backend/src/process_wizard/db/tables.py` — SQLAlchemy ORM models for 6 tables:
>
> 1. `documents` — id (TEXT, UUID), schema_version, version, type, title, status, metadata (JSON), formio_schema (JSON), output_rules (JSON), data_requirements (JSON), created_at, updated_at
> 2. `runs` — id (TEXT, UUID), document_id (FK), document_version, user_id, status, input_method, collected_data (JSON), overrides (JSON), output_data (JSON), parent_run_id, started_at, completed_at
> 3. `reference_index` — id (INTEGER, autoincrement), user_id, reference_key, run_id (FK), date, value (JSON). Index on (user_id, reference_key, date DESC)
> 4. `run_analytics` — id (INTEGER), user_id, run_id (FK), document_id, date, component_key, value (REAL), text_value (TEXT), unit (TEXT). Index on (user_id, document_id, component_key, date DESC)
> 5. `lookup_tables` — id (TEXT), version, hit_policy, inputs (JSON), outputs (JSON), rules (JSON), created_at, updated_at
> 6. `user_profiles` — id (TEXT, UUID), display_name, age, sex, bodyweight, height, preferred_units, metadata (JSON), created_at, updated_at
>
> Use TEXT for UUIDs and JSON for complex fields (compatible with both SQLite and Postgres).
>
> `backend/src/process_wizard/db/session.py`:
> - Async session factory reading DATABASE_URL from environment
> - Default to SQLite: `sqlite+aiosqlite:///./process_wizard.db`
> - FastAPI dependency `get_session()` yielding async sessions
>
> Create the Alembic migration. Run it. Verify tables exist.
> Push to branch `step-03-schema`.

**Verify at night:**
- [ ] `alembic upgrade head` creates all 6 tables
- [ ] `alembic downgrade -1` works cleanly
- [ ] JSON columns work in SQLite (test a manual insert)
- [ ] The indexes are created

---

### Step 4: Database Repositories

**Prompt:**
> Create `backend/src/process_wizard/db/repositories.py` with async CRUD functions. Use SQLAlchemy 2.0 async patterns with `AsyncSession`.
>
> Documents:
> - `get_document(session, doc_id) -> DocumentRecord | None`
> - `create_document(session, doc) -> DocumentRecord`
>
> Runs:
> - `create_run(session, run) -> Run`
> - `get_run(session, run_id) -> Run | None`
> - `update_run_data(session, run_id, collected_data) -> Run` — full overwrite of collected_data
> - `complete_run(session, run_id, output_data) -> Run` — sets status=completed, completed_at=now, output_data
> - `abandon_run(session, run_id) -> Run`
>
> References:
> - `write_references(session, entries: list) -> None` — batch insert
> - `get_latest_references(session, user_id) -> dict[str, Any]` — most recent value per key. Use subquery with MAX(date) grouped by reference_key (works in both SQLite and Postgres)
> - `get_reference(session, user_id, key) -> Any | None`
>
> Analytics:
> - `write_analytics(session, rows: list) -> None` — batch insert
>
> Users:
> - `create_user(session, user) -> UserProfile`
> - `get_user(session, user_id) -> UserProfile | None`
>
> Write integration tests in `backend/tests/test_repositories.py` that test against SQLite. Test the full run lifecycle: create → update data → complete. Test reference retrieval returns latest values.
> Push to branch `step-04-repositories`.

**Verify at night:**
- [ ] All tests pass
- [ ] `get_latest_references` returns only the most recent value per key (test with two runs for the same user)
- [ ] `complete_run` rejects runs that aren't in_progress

---

### Step 5: GoRules Decision Tables

**Prompt:**
> Set up the GoRules Zen Engine integration.
>
> First, create the decision table JSON files in `data/`:
>
> `data/risk_stratification.json` — a GoRules JDM decision graph:
> - Input fields: parq_level (string), conditions_managed (string), pain_class (string), stress_class (string)
> - Output field: risk_classification (string)
> - Hit policy: first
> - Rules (evaluated top to bottom, first match wins):
>   1. parq_level == "high" → "high"
>   2. conditions_managed == "poorly_managed" → "high"
>   3. pain_class == "severe" → "high"
>   4. parq_level == "intermediate" → "intermediate"
>   5. conditions_managed == "partially_managed" → "intermediate"
>   6. pain_class == "moderate" → "intermediate"
>   7. stress_class == "high" → "intermediate"
>   8. (default) → "low"
>
> `data/medication_implications.json` — a GoRules JDM decision graph:
> - Input field: med_category (string)
> - Output fields: implication (string), action (string)
> - Hit policy: collect (return all matching rows)
> - Rules:
>   1. beta_blockers → "Blunted HR response", "Use RPE not HR"
>   2. diuretics → "Dehydration/electrolyte risk", "Emphasize hydration"
>   3. insulin_diabetes_meds → "Hypoglycemia risk", "Have fast carbs available"
>   4. anticoagulants → "Bleeding/bruising risk", "Avoid high-contact"
>   5. statins → "Possible myopathy", "Monitor muscle soreness"
>   6. corticosteroids → "Bone density reduction", "Emphasize weight-bearing"
>
> Then create `backend/src/process_wizard/engine/decisions.py`:
> - Class `DecisionEngine` that loads JDM files from `data/` directory on init
> - Method `evaluate_risk(parq_level, conditions_managed, pain_class, stress_class) -> str`
> - Method `evaluate_medication_implications(med_categories: list[str]) -> list[dict]`
> - Cache loaded decisions in memory (don't re-read files on every call)
>
> Write tests in `backend/tests/test_decisions.py`:
> - Risk: all low inputs → "low"
> - Risk: parq_level="high" → "high" regardless of other inputs
> - Risk: moderate pain + intermediate parq → "intermediate"
> - Medication: beta_blockers → returns HR implication
> - Medication: [beta_blockers, statins] → returns 2 implications
>
> Run tests. Push to branch `step-05-gorules`.

**Important:** The GoRules JDM JSON format has a specific structure with nodes and edges. If you're unsure of the exact schema, use the GoRules online editor (https://editor.gorules.io) to create the decision tables visually, then export the JSON. Alternatively, check the GoRules documentation for the JDM JSON schema.

**Verify at night:**
- [ ] `pip install zen-engine` works
- [ ] All decision table tests pass
- [ ] The JDM JSON files are valid (parseable by zen-engine)
- [ ] Risk stratification hit policy is "first" — verify ordering matters
- [ ] Medication implications hit policy is "collect" — verify multiple results returned

---

### Step 6: Form.io Intake Schema

**Prompt:**
> Create the Form.io wizard schema for the client intake form at `data/intake_form.json`.
>
> Read `docs/mvp_spec.md` for the full 7-step intake wireframe. Translate it into Form.io JSON format with `"display": "wizard"`.
>
> Form.io wizard structure:
> ```json
> {
>   "display": "wizard",
>   "components": [
>     {
>       "type": "panel",
>       "title": "Step 1: Consent & Pre-Screening",
>       "components": [ ... step 1 components ... ]
>     },
>     {
>       "type": "panel",
>       "title": "Step 2: PAR-Q+ Screening",
>       "components": [ ... step 2 components ... ]
>     },
>     ... steps 3-7 ...
>   ]
> }
> ```
>
> For each step, implement the components from the MVP spec:
>
> **Step 1: Consent & Pre-Screening**
> - HTML component with practitioner context text
> - Checkbox: informed_consent (required)
> - Signature: client_signature (required)
> - DateTime: intake_date (default today)
>
> **Step 2: PAR-Q+ Screening**
> - HTML component explaining PAR-Q+
> - 7 Checkboxes: parq_1_heart through parq_7_medically_supervised
> - Conditional fields: parq follow-up questions with `conditional.json` using JSON Logic — shown only when the relevant parq checkbox is true
> - Example: cardiac follow-up visible when `{"==": [{"var": "data.parq_1_heart"}, true]}`
>
> **Step 3: Health & Medical History**
> - Checkbox: has_chronic_conditions
> - Select (multiple): chronic_conditions (conditional on has_chronic_conditions)
> - Select: conditions_managed (well/partially/poorly)
> - Checkbox: takes_medications
> - Select (multiple): medication_categories (beta_blockers, diuretics, insulin_diabetes_meds, anticoagulants, statins, corticosteroids)
> - Checkbox: has_current_pain
> - Number (0-10): pain_at_rest, pain_during_activity (conditional on has_current_pain)
> - Select: pain_pattern (constant, intermittent, activity_only, morning_only)
> - Checkboxes: family_cardiac_early, family_metabolic, family_osteoporosis
>
> **Step 4: Fitness & Training History**
> - Select: activity_level (sedentary, lightly_active, moderately_active, very_active)
> - Number: training_days_per_week, session_duration_minutes, training_age_years, activity_consistency_months
> - Select (multiple): current_exercise_types, domain_experience
> - Checkbox: has_worked_with_trainer
> - Textarea: what_worked_before, what_didnt_work, what_needs_to_be_different
> - Select (multiple): adherence_barriers
>
> **Step 5: Lifestyle Patterns**
> - Number: available_training_days (1-7), available_session_minutes
> - Select: preferred_training_time (early_morning, morning, midday, afternoon, evening)
> - Select (multiple): available_days (mon-sun)
> - Select: occupation_type (sedentary_desk, light_active, moderately_active, heavy_labor)
> - Number: daily_sitting_hours, sleep_hours_weeknight, sleep_hours_weekend
> - Number (1-5): sleep_quality
> - Select: sleep_difficulty (none, falling_asleep, staying_asleep, both)
> - Checkbox: wakes_feeling_rested, snoring_or_apnea_signs
> - Number (1-10): stress_level
> - Select (multiple): stress_sources, stress_coping
> - Number: daily_step_count
> - Select: neat_level, primary_transportation
>
> **Step 6: Goals & Priorities**
> - Select (multiple): primary_goals (build_strength, build_muscle, lose_fat, improve_endurance, improve_health, sport_performance, reduce_pain, increase_mobility, general_fitness, mental_health)
> - Textarea: goal_description
> - Select: goal_timeline
> - Textarea: practitioner_goal_notes
>
> **Step 7: Practitioner Synthesis**
> - HTML component: "This section is completed after reviewing all collected data"
> - Select: risk_override (none, low, intermediate, high)
> - Textarea: risk_override_reason (conditional on risk_override != none)
> - Textarea: red_flags_summary, referral_decisions, program_design_priorities
>
> Use proper Form.io component keys matching the field names in the MVP spec.
> Include appropriate validation: required fields, min/max on numbers, conditional required-if.
> Add warning/error HTML components that show conditionally (e.g., severe pain warning when pain > 7).
>
> Push to branch `step-06-formio-schema`.

**This is the largest single step.** The resulting JSON will be 500-1000 lines. Claude may need to build it across multiple messages.

**Verify at night:**
- [ ] Load the JSON in the Form.io sandbox (https://formio.github.io/formio.js/app/sandbox) to visually verify
- [ ] Wizard navigation works (7 steps)
- [ ] Conditional visibility works (PAR-Q follow-ups show/hide correctly)
- [ ] Medication callouts appear when categories selected
- [ ] Pain warning appears at threshold
- [ ] All field keys match the names used in the output evaluator spec

---

### Step 7: Seed Script

**Prompt:**
> Create `backend/src/process_wizard/db/seed.py` with:
>
> - `async def seed_all(session)` — idempotent (check if data exists first)
> - Load `data/intake_form.json` and insert as a document record with:
>   - id: a fixed UUID (use `uuid5` with a namespace so it's deterministic)
>   - type: "form"
>   - title: "Comprehensive Client Intake"
>   - status: "published"
>   - formio_schema: the loaded JSON
>   - output section with referenceable spec: key="client_intake", fields=["risk_classification", "training_level", "volume_modifier", "intensity_ceiling", "sleep_score", "stress_level", "available_days", "available_minutes", "active_injuries", "medication_flags", "parq_expiry_date"]
> - Insert the two GoRules decision table records (risk_stratification, medication_implications)
> - Create a test user profile
>
> Also create `backend/src/process_wizard/__main__.py` so you can run: `python -m process_wizard.db.seed`
>
> Run the seed script. Verify data is in the database.
> Push to branch `step-07-seed`.

**Verify at night:**
- [ ] `python -m process_wizard.db.seed` runs without errors
- [ ] Running it twice doesn't create duplicates (idempotent)
- [ ] The document record contains the full Form.io schema

---

## Phase 2 — API + Form.io Integration (Week 2)

### Step 8: API Endpoints — Documents

**Prompt:**
> Create `backend/src/process_wizard/api/documents.py` with FastAPI router:
>
> - `GET /api/documents/{id}` — returns the document record including the Form.io schema
> - `GET /api/documents/{id}/wizard` — returns a WizardBundle: `{ document_id, title, formio_schema, decision_tables }`. The formio_schema is the raw Form.io JSON that the frontend will pass directly to the Form.io renderer. decision_tables is a dict of table_id → table_data for any tables the frontend might need.
>
> Register the router in main.py.
> Write tests that seed the database, fetch the document, and verify the Form.io schema comes back intact.
> Push to branch `step-08-api-documents`.

---

### Step 9: API Endpoints — Runs

**Prompt:**
> Create `backend/src/process_wizard/api/runs.py` with FastAPI router:
>
> - `POST /api/runs` — body: `{ document_id, user_id }`. Creates a run with status=in_progress, input_method=wizard. Resolves data requirements (for intake this returns empty dict, but wire up the resolver). Returns `{ run_id, resolved_context }`.
>
> - `PUT /api/runs/{id}/data` — body: `{ collected_data }`. Overwrites the run's collected_data. This is a checkpoint save called periodically during the wizard. Returns 200.
>
> - `POST /api/runs/{id}/complete` — triggers completion (placeholder for now: just sets status=completed, completedAt=now). Returns `{ output_data, summary }`.
>
> - `POST /api/runs/{id}/abandon` — sets status=abandoned.
>
> - `GET /api/runs/{id}` — returns the run (for resuming an in-progress wizard).
>
> Add validation: can't complete a non-in_progress run, can't checkpoint a completed run.
> Write integration tests for the full lifecycle: create → checkpoint → complete.
> Push to branch `step-09-api-runs`.

---

### Step 10: API Endpoints — References, Users, Health

**Prompt:**
> Create remaining API endpoints:
>
> `backend/src/process_wizard/api/references.py`:
> - `GET /api/users/{user_id}/references` — all current reference values (latest per key)
> - `GET /api/users/{user_id}/references/{key}` — specific reference value
>
> `backend/src/process_wizard/api/users.py`:
> - `POST /api/users` — create user profile
> - `GET /api/users/{user_id}` — fetch user profile
> - `GET /api/users/{user_id}/profile-summary` — placeholder (returns empty for now)
>
> Register all routers in main.py with `/api` prefix.
> Verify OpenAPI docs at `/docs` show all endpoints.
> Push to branch `step-10-api-remaining`.

---

### Step 11: Frontend — Form.io Wizard Shell

**Prompt:**
> Create the Form.io wizard integration in the frontend.
>
> `frontend/src/api/client.ts`:
> - `fetchWizardBundle(documentId: string): Promise<WizardBundle>`
> - `createRun(documentId: string, userId: string): Promise<{ run_id: string, resolved_context: object }>`
> - `saveRunData(runId: string, collectedData: object): Promise<void>`
> - `completeRun(runId: string): Promise<CompletionResult>`
> - `fetchRun(runId: string): Promise<Run>`
> - `fetchUserReferences(userId: string): Promise<object>`
> - `fetchProfileSummary(userId: string): Promise<object>`
> - Base URL from `VITE_API_URL` environment variable (default http://localhost:8000)
>
> `frontend/src/types/index.ts`:
> - TypeScript types for: WizardBundle, Run, CompletionResult
>
> `frontend/src/components/wizard/IntakeWizard.tsx`:
> - Props: documentId, userId
> - On mount: fetchWizardBundle(documentId), then createRun(documentId, userId)
> - Renders Form.io wizard using `@formio/react` Form component with:
>   - `form={wizardBundle.formio_schema}` — the Form.io JSON
>   - `onSubmit` handler that calls completeRun
>   - `onChange` handler that debounces checkpoint saves (every 30 seconds or on page change)
>   - `options={{ readOnly: false }}` — editable wizard mode
> - Loading state while fetching
> - Error state on API failures
>
> `frontend/src/App.tsx`:
> - Simple routing:
>   - `/wizard/:documentId` → IntakeWizard
>   - `/profile/:userId` → placeholder
>   - `/` → landing page with "Start Intake" button
> - Hard-code the test user ID for MVP
>
> Start both backend and frontend. Navigate to the wizard route. Verify the Form.io wizard renders with all 7 steps.
> Push to branch `step-11-formio-wizard`.

**Verify at night:**
- [ ] The wizard renders all 7 steps with correct fields
- [ ] Conditional visibility works (toggle PAR-Q checkboxes, verify follow-ups show/hide)
- [ ] Navigation between steps works
- [ ] Validation prevents advancing with missing required fields
- [ ] Console shows no errors

---

### Step 12: Frontend — Wizard Data Flow

**Prompt:**
> Wire up the data flow between the Form.io wizard and the backend API.
>
> In `IntakeWizard.tsx`:
> - On Form.io `onChange` event: extract the submission data, debounce-save to `PUT /api/runs/{id}/data` every 30 seconds
> - On Form.io page change (`onNextPage`, `onPrevPage`): save checkpoint immediately
> - On Form.io `onSubmit` event: call `POST /api/runs/{id}/complete`, navigate to completion screen
> - Add a "Save & Exit" button that saves current data and navigates to landing page
> - Add resume capability: if a run_id is in the URL, fetch the run and pre-populate Form.io with the saved collected_data using the `submission` prop
>
> Test the following flow:
> 1. Start the wizard, fill steps 1-3, close the browser
> 2. Reopen with the run_id in the URL — data is restored
> 3. Continue to step 7, submit — completion endpoint is called
>
> Push to branch `step-12-data-flow`.

**Verify at night:**
- [ ] Checkpoint saves work (check the database for saved collected_data)
- [ ] Resume works after page reload
- [ ] Submission triggers the complete endpoint
- [ ] The collected_data matches Form.io's submission format

---

## Phase 3 — Backend Pipeline (Weeks 3–4)

### Step 13: Data Requirement Resolver

**Prompt:**
> Create `backend/src/process_wizard/services/resolver.py`:
>
> `async def resolve_data_requirements(session, document, user_id) -> dict`:
> - If document.data_requirements is None, return empty dict
> - For "user.profile" fields: query user_profiles table
> - For "user.references" mapping: query reference_index for most recent value per key
> - Return flat dict with namespaced keys: `{"user.age": 35, "ref.client_intake.risk_classification": "low"}`
>
> For the intake MVP, this returns `{}`. But implement the full logic now — every future document needs it.
>
> Write tests:
> - No data requirements → empty dict
> - User profile fields → resolved from user_profiles table
> - Reference fields → resolved from reference_index (latest values)
> - Missing references → None values (not errors)
>
> Push to branch `step-13-resolver`.

---

### Step 14: Output Rule Evaluator

**Prompt:**
> Create `backend/src/process_wizard/services/output_evaluator.py`:
>
> `async def evaluate_output_rules(document, collected_data, resolved_context, decision_engine) -> dict`:
>
> This evaluates the document's output rules against the collected data to produce derived values. For the intake document, the output rules compute:
>
> 1. `parq_risk_level` — if any PAR-Q questions are YES, classify as intermediate; if chest pain or medically supervised, high; if all NO, low
> 2. `pain_severity_class` — based on pain_during_activity: 0=none, 1-4=mild, 5-6=moderate, 7-10=severe
> 3. `training_level` — based on training_age_years: <1=beginner, <3=intermediate, else=advanced
> 4. `sleep_score` — based on hours + quality + rested: thresholds for good/moderate/poor
> 5. `stress_classification` — based on stress_level: 1-3=low, 4-6=moderate, 7-10=high
> 6. `risk_classification` — calls DecisionEngine.evaluate_risk() with (parq_risk_level, conditions_managed, pain_severity_class, stress_classification)
> 7. `final_risk` — uses risk_override if set, otherwise risk_classification
> 8. `volume_modifier` — 1.0 × sleep_factor (good=1.0, moderate=0.9, poor=0.8) × stress_factor (low=1.0, moderate=0.9, high=0.8)
> 9. `intensity_ceiling` — high→"low", intermediate→"low_to_moderate", low→"unrestricted"
> 10. `medication_flags` — list of selected medication categories
> 11. `medication_implications` — calls DecisionEngine.evaluate_medication_implications()
> 12. `active_injuries` — extracted from affected_body_regions
> 13. `parq_expiry_date` — intake_date + 12 months
> 14. `available_days` and `available_minutes` — directly from collected data
> 15. `primary_goals` — directly from collected data
>
> Implementation approach: since we're using Python (not a generic expression engine), write the evaluation as clear Python functions. Each output field is computed by a named function. This is more readable and debuggable than expression evaluation. The document's OutputSection defines which fields to compute — the evaluator maps field names to computation functions.
>
> Write thorough tests:
> - Healthy client → risk=low, volume_modifier=1.0, intensity=unrestricted
> - High-risk client → risk=high, volume_modifier=0.64, intensity=low
> - Intermediate client → risk=intermediate, volume_modifier=0.81, intensity=low_to_moderate
> - Edge cases: missing fields, null values
>
> Push to branch `step-14-output-evaluator`.

**Verify at night — this is the most important review in the project:**
- [ ] Each computation function matches the clinical logic from the MVP spec
- [ ] PAR-Q risk classification logic is correct (desk-check against the book's decision tree)
- [ ] Volume modifier math: 1.0 × 0.8 × 0.8 = 0.64 for worst case
- [ ] GoRules integration works for risk stratification
- [ ] All test scenarios produce the correct output

---

### Step 15: Reference Index Writer

**Prompt:**
> Create `backend/src/process_wizard/services/reference_writer.py`:
>
> `async def write_references(session, user_id, run_id, completed_at, output_data, referenceable) -> None`:
> - For each field in referenceable.fields:
>   - Key = f"{referenceable.key}.{field}" (e.g., "client_intake.risk_classification")
>   - Value = output_data[field]
>   - Insert into reference_index table
> - Batch insert for efficiency
>
> Write tests that verify:
> - All 11 expected references are written after intake completion
> - Values are correctly extracted from output_data
> - A second run for the same user creates new entries (not overwrites — the latest query handles priority)
>
> Push to branch `step-15-reference-writer`.

---

### Step 16: Analytics Denormalizer

**Prompt:**
> Create `backend/src/process_wizard/services/analytics.py`:
>
> `async def denormalize_run(session, run) -> None`:
> - Flatten collected_data dict into rows in run_analytics table
> - For each key-value:
>   - Numbers → store in `value` column
>   - Strings → store in `text_value` column
>   - Booleans → store as 1.0/0.0 in `value`
>   - Lists → store as JSON string in `text_value`
> - Batch insert all rows
>
> Write tests. Push to branch `step-16-analytics`.

---

### Step 17: Completion Orchestrator

**Prompt:**
> Create `backend/src/process_wizard/services/completion.py`:
>
> `async def complete_run(session, run_id, decision_engine) -> CompletionResult`:
> 1. Fetch the run (verify status = in_progress)
> 2. Fetch the document
> 3. Resolve data requirements (empty for intake, but call it)
> 4. Evaluate output rules → output_data
> 5. Update run: status=completed, completed_at=now(), output_data
> 6. If document has referenceable spec: write references
> 7. Denormalize analytics
> 8. Return CompletionResult with output_data and summary
>
> All steps 5-7 should be in the same database transaction.
>
> Update `POST /api/runs/{id}/complete` to call this orchestrator instead of the placeholder.
>
> Write integration tests:
> - Create run → save realistic collected data for healthy client → complete → verify:
>   - Run status is completed
>   - Output data has all expected fields
>   - Reference index has 11 entries
>   - Analytics table has rows for collected data
> - Repeat for high-risk client scenario
>
> Push to branch `step-17-completion`.

**Verify at night:**
- [ ] The full pipeline executes without errors
- [ ] Output data is correct for both healthy and high-risk scenarios
- [ ] References are indexed correctly
- [ ] Transaction is atomic (if reference writing fails, the run isn't marked complete)

---

## Phase 4 — Profile & Integration (Week 4–5)

### Step 18: Profile Summary Endpoint

**Prompt:**
> Implement `GET /api/users/{user_id}/profile-summary`:
>
> - Query reference_index for all keys starting with "client_intake." for the user
> - Get most recent value for each key
> - Look up medication_implications from GoRules for any medication_flags values
> - Assemble and return structured profile:
>   ```json
>   {
>     "risk_classification": "low",
>     "training_level": "intermediate",
>     "volume_modifier": 0.9,
>     "intensity_ceiling": "unrestricted",
>     "sleep_score": "moderate",
>     "stress_level": 4,
>     "available_days": 4,
>     "available_minutes": 60,
>     "active_injuries": [],
>     "medication_flags": [],
>     "medication_implications": [],
>     "primary_goals": ["build_strength"],
>     "parq_expiry_date": "2027-03-02",
>     "days_until_parq_expiry": 365,
>     "last_intake_date": "2026-03-02"
>   }
>   ```
>
> Write tests. Push to branch `step-18-profile-summary`.

---

### Step 19: Completion Summary Screen

**Prompt:**
> Create `frontend/src/components/completion/CompletionSummary.tsx`:
>
> Shown after the wizard completes. Receives the CompletionResult from the API.
>
> Display:
> - Risk classification as a large colored badge (green=low, amber=intermediate, red=high)
> - Volume modifier as a percentage with explanation
> - Intensity ceiling
> - Training level
> - Sleep score and stress level
> - Active injuries (if any)
> - Medication flags with implications
> - Primary goals
> - Available schedule
> - "View Full Profile" button → navigates to /profile/{userId}
> - "Start New Intake" button → navigates to wizard
>
> Use Tailwind for styling. Make it look professional — this is what the practitioner sees after completing a 30-minute intake.
>
> Wire it up: after `completeRun` succeeds in IntakeWizard, navigate to the completion summary with the output data.
> Push to branch `step-19-completion-screen`.

---

### Step 20: Client Profile Page

**Prompt:**
> Create `frontend/src/components/profile/ClientProfile.tsx`:
>
> - Fetches profile summary from `GET /api/users/{id}/profile-summary`
> - Displays the one-page client profile:
>   - Risk badge (colored, prominent)
>   - Two-column layout on desktop, single column on mobile
>   - Left: health/risk factors (risk, pain, injuries, medications with implications)
>   - Right: training/lifestyle (training level, schedule, sleep, stress, goals)
>   - PAR-Q expiry with countdown (highlight if < 30 days)
>   - "Re-administer Intake" button
>   - "Last completed" date
> - Handle case where no intake exists: show "No intake completed" message
>
> Add the route to App.tsx.
> Push to branch `step-20-profile-page`.

---

### Step 21: Reference Resolution Proof

**Prompt:**
> This step proves the entire reference pipeline works end-to-end.
>
> Create a minimal test document in `data/reference_test.json`:
> - A simple Form.io form (not a wizard) with 2 HTML components
> - data_requirements that reference client_intake values:
>   ```json
>   {
>     "user": {
>       "references": {
>         "risk": "client_intake.risk_classification",
>         "volume_mod": "client_intake.volume_modifier",
>         "intensity_cap": "client_intake.intensity_ceiling"
>       }
>     }
>   }
>   ```
>
> Write an integration test:
> 1. Seed the intake document and test user
> 2. Create and complete an intake run with known data
> 3. Seed the reference test document
> 4. Create a run for the reference test document for the same user
> 5. Verify the resolved_context returned by POST /api/runs contains:
>    - ref.client_intake.risk_classification = expected value
>    - ref.client_intake.volume_modifier = expected value
>    - ref.client_intake.intensity_ceiling = expected value
>
> This proves: intake → output rules → reference index → data requirement resolution → downstream document. When this test passes, the architecture is validated.
>
> Push to branch `step-21-reference-proof`.

**Verify at night — this is the second most important review:**
- [ ] The test creates an intake run and completes it
- [ ] References are indexed
- [ ] A second document's run resolves those references
- [ ] The values match what the intake computed

---

## Phase 5 — End-to-End & Polish (Week 5)

### Step 22: End-to-End Integration Test

**Prompt:**
> Create `backend/tests/test_e2e.py` that tests the full system:
>
> **Scenario A: Healthy Client**
> 1. Seed database
> 2. Create user
> 3. POST /api/runs (intake document)
> 4. PUT /api/runs/{id}/data with healthy client data:
>    - All PAR-Q NO
>    - No conditions, no medications, no pain
>    - Training age: 2 years, 4 days/week
>    - Sleep: 7.5 hrs, quality 4, rested
>    - Stress: 3
>    - Goals: build_strength, improve_health
> 5. POST /api/runs/{id}/complete
> 6. Assert: risk=low, volume_modifier=1.0, intensity=unrestricted, training_level=intermediate
> 7. GET /api/users/{id}/references — 11 entries
> 8. GET /api/users/{id}/profile-summary — correct profile
>
> **Scenario B: High-Risk Client**
> - PAR-Q YES on chest pain + medically supervised
> - Poorly managed diabetes, beta blockers + statins
> - Severe pain (8/10), poor sleep (5 hrs, quality 2), high stress (9)
> - Assert: risk=high, volume_modifier=0.64, intensity=low, 2 medication implications
>
> **Scenario C: Resume**
> - Create run, save data for steps 1-3, fetch run, verify data persists
>
> **Scenario D: Re-Intake**
> - Complete intake, then complete a second intake with different data
> - Verify profile-summary returns the latest values
>
> Push to branch `step-22-e2e-tests`.

---

### Step 23: Error Handling

**Prompt:**
> Add comprehensive error handling across the stack:
>
> Backend:
> - Global exception handler returning structured JSON errors
> - 404 for missing documents, runs, users
> - 409 for invalid state transitions (completing an abandoned run)
> - 422 for validation errors (Pydantic catches these automatically)
> - Log errors with context (run_id, document_id)
>
> Frontend:
> - Toast notifications for save errors (non-blocking)
> - Retry logic for checkpoint saves (3 attempts with backoff)
> - Error boundary around the Form.io wizard
> - Clear error message on completion failure with retry button
> - "Unsaved changes" warning on browser back/close
>
> Push to branch `step-23-error-handling`.

---

### Step 24: Landing Page & Navigation

**Prompt:**
> Create a polished landing page and navigation:
>
> `frontend/src/components/LandingPage.tsx`:
> - "Process Wizard" header
> - "Start New Intake" button (prominent, primary color)
> - "View Client Profile" button (if intake exists)
> - "Resume Intake" button (if in-progress run exists — check via API)
>
> `frontend/src/components/Layout.tsx`:
> - Simple nav bar with app name
> - Back navigation where appropriate
>
> Update App.tsx routing:
> - `/` → LandingPage
> - `/wizard/:documentId` → IntakeWizard
> - `/wizard/:documentId/run/:runId` → IntakeWizard (resume mode)
> - `/completion/:runId` → CompletionSummary
> - `/profile/:userId` → ClientProfile
>
> Push to branch `step-24-navigation`.

---

### Step 25: Polish & Accessibility

**Prompt:**
> Final polish pass:
>
> - Form.io theme: apply clean styling (custom CSS or Form.io theme). The default Bootstrap theme is fine for MVP but ensure it's readable on tablets.
> - Responsive design: test wizard at iPad viewport (768px). Ensure all fields are usable.
> - Loading states: spinner on wizard load, button loading states during save/complete.
> - Tab/keyboard navigation through Form.io fields.
> - Print-friendly profile page (basic @media print styles).
> - Clean up console warnings.
> - Remove any debug logging.
> - Add favicon.
>
> Push to branch `step-25-polish`.

### Step 26:# Process Wizard MVP — Post-Implementation Next Steps

## 1. User Acceptance Testing (UAT)
- Recruit real users (coaches, clients, stakeholders) to run through the full intake and workflow.
- Collect feedback on usability, clarity, and any missing features or pain points.

## 2. Bug Fixes & Polish
- Address issues or UX friction found during UAT.
- Refine error handling, loading states, and validation messages.

## 3. Security & Compliance Review
- Review authentication, authorization, and data privacy (especially for health data).
- Ensure sensitive data is protected and audit logging is in place if needed.

## 4. Performance & Scalability
- Test with larger datasets and multiple users.
- Profile API and database performance; optimize queries and indexes as needed.

## 5. Documentation
- Write clear user and developer documentation (setup, API usage, workflows).
- Add onboarding guides for new users.

## 6. Deployment Readiness
- Prepare production infrastructure (Postgres, environment variables, HTTPS, backups).
- Set up CI/CD for automated testing and deployment.

## 7. MVP Launch & Feedback Loop
- Deploy to a staging or production environment.
- Monitor usage, collect feedback, and iterate quickly on improvements.
---

## Verification Checklist

Before declaring the MVP done, verify these from your desktop:

### Manual Walkthrough

- [ ] **Healthy client path:** Start intake → consent → all PAR-Q NO → no conditions → 2yr training → good sleep/low stress → complete → risk=low, volume=1.0
- [ ] **High-risk path:** PAR-Q YES chest pain → conditions poorly managed → beta blockers + statins → pain 8/10 → poor sleep/high stress → complete → risk=high, volume=0.64
- [ ] **Conditional visibility:** PAR-Q follow-ups show/hide, medication callouts appear, pain warning at threshold
- [ ] **Resume flow:** Fill 3 steps, close browser, reopen with run_id → data restored → continue
- [ ] **Re-intake:** Complete intake, start new one → profile updates to latest values
- [ ] **Profile page:** All values displayed correctly, PAR-Q expiry date shown
- [ ] **Reference resolution:** Test document resolves intake values

### Automated Tests
- [ ] `pytest backend/tests/` — all pass against SQLite
- [ ] GitHub Actions CI — all pass against Postgres
- [ ] Decision table tests pass
- [ ] E2E integration tests pass

### Technical
- [ ] No console errors in browser
- [ ] API returns proper error codes (404, 409, 422)
- [ ] Checkpoint saves work silently in background
- [ ] Form.io wizard handles all input types correctly

---

## Timing Summary

| Phase | Steps | Calendar Time | Hours (est.) |
|---|---|---|---|
| Phase 1: Foundation | Steps 1–7 | Week 1 | 12–16 hrs |
| Phase 2: API + Form.io | Steps 8–12 | Week 2 | 10–14 hrs |
| Phase 3: Backend Pipeline | Steps 13–17 | Weeks 3–4 | 12–16 hrs |
| Phase 4: Profile & Integration | Steps 18–21 | Week 4–5 | 8–12 hrs |
| Phase 5: Polish | Steps 22–25 | Week 5 | 8–10 hrs |
| **Total** | **25 steps** | **~5 weeks** | **50–68 hrs** |

---

## Phone Session Tips

1. **One step per session.** Each step is scoped to produce a mergeable PR. Don't try to combine steps.

2. **Reference CLAUDE.md in every prompt.** Start with "Read CLAUDE.md for project context and conventions" — Claude loads it automatically, but reminding it focuses attention.

3. **Be specific about the branch name.** Claude Code pushes to whatever branch you specify. Consistent naming (`step-XX-description`) makes PR review easy.

4. **Don't hover.** These steps are designed for autonomous execution. Check in once to verify Claude understood the task, then let it run. Review the PR at night.

5. **Update CLAUDE.md's "Current Step" after each merge.** This is your persistent state between sessions. When you merge step 14's PR, update CLAUDE.md to say "Working on: Step 15."

6. **If Claude gets stuck on Form.io JSON format**, point it to:
   - https://formio.github.io/formio.js/app/sandbox (interactive sandbox)
   - https://help.form.io/userguide/forms/form-building/logic-and-conditions (conditional logic docs)
   - https://github.com/formio/formio.js/wiki/Components-JSON-Schema (component schema reference)

7. **If Claude gets stuck on GoRules JDM format**, point it to:
   - https://editor.gorules.io (visual editor — create tables here, export JSON)
   - https://docs.gorules.io/reference/overview (SDK docs)
