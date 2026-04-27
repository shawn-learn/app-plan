# Development Guidelines

## Timestamp Fields

Every table in the project must provide `created_at` and `updated_at` columns. Use the `TimestampMixin` from `app.models.tables` for SQLAlchemy models and add the same columns in Alembic migrations.

- **Inserts**: set `created_at` and `updated_at` using `utc_now()` from `app.utils`.
- **Updates**: refresh `updated_at` with `utc_now()` while leaving `created_at` unchanged.

These rules ensure that offline synchronization can rely on `updated_at` to detect changes.

## Testing

Run tests by functional area using the helper script. This makes it easy to execute only the relevant subsets during development.

```bash
# run only API endpoint tests
./scripts/run_tests.sh api

# run database model and seed tests
./scripts/run_tests.sh models

# run service-layer tests
./scripts/run_tests.sh services

# run messaging subsystem tests
./scripts/run_tests.sh messaging

# run frontend tests only
./scripts/run_tests.sh frontend

# run all backend and frontend tests
./scripts/run_tests.sh all
```

Always run the full suite (`all`) before committing changes.
