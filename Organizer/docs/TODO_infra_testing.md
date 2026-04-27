## User Interface Tests

- [x] Guest session automatically starts on first visit and stores the token in `localStorage`.
  1. Clear browser storage and visit the home page.
  2. Confirm a guest token has been created in `localStorage`.
  3. Verify anonymous API requests succeed using this token.

- [x] Signup form validates email format and matching passwords with inline errors.
  1. Enter an invalid email and blur the field to trigger validation.
  2. Expect an `Invalid email` message under the input.
  3. Provide mismatched passwords and submit the form.
  4. Expect the password mismatch error to appear.

- [x] Successful signup creates an account and logs the user in.
  1. Fill out the signup form with valid details.
  2. Submit and wait for redirect to the dashboard.
  3. The new user's avatar should display in the header.

- [x] Login with valid credentials grants access and displays the user avatar.
  1. Navigate to `/login`.
  2. Enter valid credentials and submit.
  3. Verify you are redirected to the dashboard with the avatar visible.

- [x] Invalid login attempts show an appropriate error message.
  1. Attempt to log in with wrong credentials.
  2. The login page should show `Invalid credentials` without redirecting.

- [x] Sidebar navigation works across pages.
  1. Click each menu item in the sidebar.
  2. Confirm the expected page loads and the URL changes accordingly.

- [x] Calendar month view loads events and allows adding a planned activity.
  1. Open the calendar page and ensure existing workouts appear.
  2. Click on a day cell to add a new planned activity.
  3. Save and verify the new activity is visible.

- [x] Fitness Planner can add periods, workouts and exercises correctly.
  1. Create a new plan in the planner.
  2. Add a period, add a workout inside it, then add an exercise.
  3. Save and reopen the plan to ensure the hierarchy persists.

- [x] Settings page saves profile changes and uploads a user image.
  1. Navigate to the settings page.
  2. Change the display name and upload an avatar image.
  3. Save and refresh to confirm the changes persist.

- [x] Change Password page updates the password when confirmation matches.
  1. Enter the current password and a new password twice.
  2. Submit and verify a success message appears.

- [x] Muscle browser opens from the Help menu and filters by body region.
  1. Open the Help menu and select `Muscle browser`.
  2. Choose a body region filter such as `Legs`.
  3. Verify only exercises for that region are shown.

- [x] Server downtime triggers the error screen when API requests fail.
  1. Simulate server failure by returning `500` errors from the API.
  2. Navigate through the app and confirm the global error screen appears.

## Production Infrastructure

### CI/CD Pipeline
- [ ] Set up GitHub Actions for automated tests and builds. (MVP)
- [ ] Build Docker images and push to a container registry. (MVP)
- [ ] Deploy to staging and production with environment-specific workflows. (MVP)

### Monitoring and Observability
- [ ] Centralize application logs using ELK or Grafana Loki.
- [ ] Collect metrics with Prometheus and visualize them in Grafana.
- [ ] Configure alerts for errors, latency, and resource usage.

### Database Backups & Disaster Recovery
- [ ] Schedule automated backups for PostgreSQL/TimescaleDB. (MVP)
- [ ] Store encrypted backups with retention policies.
- [ ] Document restore procedures and verify recovery regularly.

### Secrets Management & Environment Configuration
- [ ] Manage secrets using a vault service across environments. (MVP)
- [ ] Load configuration from environment variables per deployment. (MVP)
- [ ] Document required variables and secure storage practices. (MVP)

### Infrastructure as Code & Scaling
- [ ] Define infrastructure with Terraform or Kubernetes manifests.
- [ ] Enable horizontal scaling and load balancing.
- [ ] Use caching layers where appropriate.

## Quality Assurance

### Comprehensive Testing
- [ ] Increase unit test coverage across modules.
- [ ] Add end-to-end tests for critical user flows.
- [ ] Run static analysis and security scans in CI.

