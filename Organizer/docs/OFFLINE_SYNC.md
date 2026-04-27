# Offline Synchronization Helpers

The backend exposes helper functions for keeping an offline database in sync with the server.

## `get_changes_since`

```python
from datetime import datetime, timezone
from sqlalchemy.orm import Session
from app.services.sync import get_changes_since

changes = get_changes_since(db, user_id=1, since=datetime(2025, 1, 1, tzinfo=timezone.utc))
```

Returns a dictionary keyed by table name containing serialized rows that have an `updated_at` value greater than the provided timestamp.

## `apply_user_changes`

```python
from app.services.sync import apply_user_changes
apply_user_changes(db, user_id=1, changes=changes)
```

Applies inserts and updates from an offline device. Records belonging to another user (determined by the `user_id` column) are ignored to prevent unauthorized modifications.
