# `pw-domain-contracts`

## Purpose

The single source of truth for all shared Pydantic v2 domain types used across the ProcessWizard backend: documents, runs, users, groups, assignments, references, stylesheets, workflows, units, and all enumerations. Every other backend library and the application itself depends on this package; it depends on almost nothing else.

**Not responsible for:** database ORM mappings (that is `pw-data-access`), HTTP request/response models (those live in `pw-api-runtime`), or any business logic.

---

## Status

**Extract** from `backend/src/process_wizard/models/`.

---

## Source location

All 10 modules move verbatim:

```
backend/src/process_wizard/models/
├── enums.py          ← DocumentType, LifecycleStatus, RunStatus, InputMethod,
│                        HitPolicy, OwnerType, GroupRole, StylesheetTarget,
│                        StylesheetScope
├── document.py       ← DocumentMetadata, OutputRule, ReferenceableSpec,
│                        OutputSection, WorkflowTrigger, DocumentRecord,
│                        FormLibraryItem, WizardBundle
├── run.py            ← Run, RecordItem, CompletionResult
├── user.py           ← UserProfile, UserProfileUpdate
├── assignment.py     ← assignment request/response models
├── group.py          ← Group, GroupMember, GroupRole request models
├── reference.py      ← ReferenceEntry
├── stylesheet.py     ← Stylesheet, StylesheetRecord
├── workflow.py       ← WorkflowStepSummary, WorkflowLibraryItem, request models
└── units.py          ← UnitCategory, UnitDefinition, UnitPreferences
```

After extraction the app imports from `pw_domain_contracts` instead of `process_wizard.models`.

---

## Public API / Exports

```python
# Enumerations
from pw_domain_contracts import (
    DocumentType, LifecycleStatus, RunStatus, InputMethod,
    HitPolicy, OwnerType, GroupRole,
    StylesheetTarget, StylesheetScope,
)

# Document domain
from pw_domain_contracts import (
    DocumentMetadata, OutputRule, ReferenceableSpec,
    OutputSection, WorkflowTrigger, DocumentRecord,
    FormLibraryItem, WizardBundle,
)

# Run domain
from pw_domain_contracts import Run, RecordItem, CompletionResult

# User domain
from pw_domain_contracts import UserProfile, UserProfileUpdate

# Group and assignment domain
from pw_domain_contracts import (
    Group, GroupMember,          # group models
    # assignment request/response models
)

# Reference domain
from pw_domain_contracts import ReferenceEntry

# Stylesheet domain
from pw_domain_contracts import Stylesheet, StylesheetRecord

# Workflow domain
from pw_domain_contracts import WorkflowStepSummary, WorkflowLibraryItem

# Unit domain
from pw_domain_contracts import UnitCategory, UnitDefinition, UnitPreferences
```

All models use `model_config = ConfigDict(extra="forbid")`.

---

## Dependencies

**External (pip):**
- `pydantic>=2.0`

**Internal pw-* dependencies:** none.

---

## Out of scope

- SQLAlchemy ORM table classes — `pw-data-access`
- FastAPI request/response models, error schemas — `pw-api-runtime`
- Any computation, database access, or HTTP logic

---

## Acceptance criteria

1. `pip install pw-domain-contracts` in a fresh virtualenv installs successfully with only `pydantic` as a transitive dependency.
2. `from pw_domain_contracts import DocumentRecord, Run, UserProfile` works without importing any process_wizard app code.
3. All existing `pytest` tests in the main app pass unchanged (the app's `process_wizard.models` namespace re-exports from `pw_domain_contracts`).
4. `DocumentRecord.model_validate(fixture_dict)` round-trips correctly for at least one intake document fixture.
5. No circular imports between any two modules in the package.

---

## Phase

**Phase A** — first library to extract; everything else builds on top of it.
