# Logs and Troubleshooting Guide

This document explains where to find backend and frontend logs, how to read them, and basic troubleshooting steps for the workout logging application.

## Backend Logs

The API server is launched with [Uvicorn](https://www.uvicorn.org/) via `scripts/run_server.sh`, `scripts/manage.py runserver`, or the Docker setup. Logging is configured in `app/log_config.py` and sends output to three places:

1. **Console** – all log messages appear in your terminal for quick debugging.
2. **Database** – entries are stored in the `system_log` table so you can view them via the API.
3. **Rotating file** – messages are also written to `system.log` in the project root with rotation (1&nbsp;MB, 3 backups) so you still have logs if the database is unavailable.

### Local development

- **Run server locally**
  ```bash
  ./scripts/run_server.sh --reload
  ```
  Logs appear directly in your terminal. They are also written to `system.log` and stored in the database. Uvicorn defaults to the `info` log level.

### Docker

- **Start containers**
  ```bash
  ./scripts/run_docker.sh
  ```
- **View logs**
  ```bash
  docker compose logs -f api
  ```

### Systemd service

If you install the API as a systemd service using `scripts/install_service.sh`, logs are captured by `journald`:

```bash
sudo journalctl -u workout-api -f
```

## Frontend Logs

The React frontend (located in `frontend/`) uses Vite for development.

- **Start the dev server**
  ```bash
  cd frontend
  npm install
  npm run dev
  ```
  Vite prints its progress and any build errors in the terminal. Runtime errors and `console.log()` output appear in your browser's developer console.

## Reading the Logs

Logs are plain text streamed to stdout/stderr and duplicated in `system.log`. When running under Docker or systemd, these outputs are captured by the container runtime or `journald` respectively. Use the commands above to tail the output or open the log file directly. Look for timestamps and log levels such as `INFO`, `WARNING`, or `ERROR` to diagnose problems.

## Troubleshooting Checklist

1. **Database connection** – ensure the `DATABASE_URL` environment variable points to your database. Run `alembic upgrade head` to create tables.
2. **Dependencies** – for the backend, run `./scripts/setup.sh` to create the virtual environment and install packages. For the frontend, run `npm install` in the `frontend/` directory.
3. **Server startup issues** – check terminal output or `journalctl` for stack traces. Confirm no other process is using the configured port (default 8000 for the API, 5173 for Vite).
4. **Frontend build problems** – delete `frontend/node_modules` and reinstall. Verify Node.js (v18+) and npm are installed.
5. **Running tests** – execute `pytest` from the repository root. All tests should pass as shown in `TEST_REPORT.md`.
6. **Python version** – use Python 3.11 or 3.12. Newer versions like Python 3.13 may not have wheels for all dependencies and can cause setup failures.
7. **`LocalProtocolError` in logs** – seeing an error such as `can't handle event type Response when role=SERVER and state=MUST_CLOSE` is usually a sign of an unsupported Python version, mismatched HTTP library versions, or non-HTTP traffic reaching Uvicorn. Ensure you are running Python 3.11/3.12, reinstall dependencies with `pip install -r requirements.txt` to update Uvicorn and `h11`, and deploy behind a reverse proxy (e.g., Nginx or Traefik) to filter malformed requests. After deploying, monitor `system.log` or container logs to confirm stack traces no longer appear.

Following these steps and reviewing the logs will resolve most issues encountered when working with the application.

