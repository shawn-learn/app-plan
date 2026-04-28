# TODO

## Messaging Service

### Messaging service overview
The repository already models users and teams, so you can build conversations on top of those structures. For example, `User` already relates to teams through memberships, giving you the participant information needed for team chats. Teams and their members are defined via `Team` and `TeamMembership` models, which you can reuse for group conversations. The repository guidelines require any new tables to include timestamp columns and to consolidate schema changes into the existing initial migration.

### Backend implementation steps

#### Database schema
- [ ] Add tables to `app/models/tables.py`:
  - `Conversation`: identifier, optional name, `team_id` nullable for team channels, `is_group` flag, and `created_at`/`updated_at` using `TimestampMixin`.
  - `ConversationParticipant`: links users to conversations (`conversation_id`, `user_id`, optional `last_read_at`), also using `TimestampMixin`.
  - `Message`: stores `conversation_id`, `sender_id`, message body, optional attachments/metadata, and timestamps.
- [ ] Update `alembic/versions/0001_initial.py` to include these tables and relationships.
- [ ] Ensure inserts/updates call `utc_now()` for timestamps.

#### Pydantic schemas
- [ ] Create `app/schemas/messages.py` with payload/response models (e.g., `ConversationCreate`, `MessageCreate`, `MessageRead`).
- [ ] Follow existing schema style: snake_case fields and small payload models like in `app/schemas/teams.py`.

#### Service layer
- [ ] Add `app/services/messaging.py` with functions to:
  - Create conversations (DM, custom group, or team-based).
  - Add/remove participants.
  - Persist messages and mark `last_read`.
  - Fetch conversation lists and messages (with pagination).
  - Validate team membership before allowing team messages.

#### API router
- [ ] Create `app/routers/messaging.py` (prefix `/messages` or `/conversations`):
  - `POST /conversations` – create DM/group chat.
  - `GET /conversations` – list user’s conversations.
  - `GET /conversations/{id}/messages` – fetch messages.
  - `POST /conversations/{id}/messages` – send message.
  - Optionally `/teams/{team_id}/conversations` for team channels.
- [ ] Use `get_db` and `get_current_user` dependencies for auth and DB access.

#### Real-time delivery (optional)
- [ ] For live updates, add WebSocket endpoints under the same router (`/ws/conversations/{id}`) using FastAPI’s WebSocket support.
- [ ] Broadcast new messages to connected participants; fall back to polling if WebSockets aren’t used.

### Tests
- [ ] Under `tests/`, add API tests verifying:
  - Creating DM/group/team conversations.
  - Sending and retrieving messages.
  - Authorization rules (non-members cannot send team messages).
- [ ] Run `pytest -q` after adding tests and update `TEST_REPORT.md`.

### Frontend implementation steps

#### Project structure
- [ ] Create `frontend/src/features/messaging/` for messaging-specific code, mirroring the `features/exercises` pattern.
- [ ] Add shared UI components (message list, input box) under `frontend/src/components` if they will be reused.

#### State and API hooks
- [ ] Implement API helpers (e.g., `useConversations`, `useMessages`, `useSendMessage`) calling the new backend endpoints.
- [ ] Use React Query or similar for caching and automatic refetch.

#### UI components
- [ ] Conversation list view: show DMs, group chats, and team channels.
- [ ] Conversation detail view: render messages, sender info, timestamps, and a form for new messages using React Hook Form and MUI components.
- [ ] Team integration: allow navigation from a team page (`TeamManagerPage.jsx`) into its conversation.

#### Real-time updates
- [ ] If WebSockets are implemented, connect via `useEffect` to receive new messages and append them to state.
- [ ] Otherwise, poll periodically or rely on refetching after sends.

#### Frontend tests
- [ ] Create tests under `frontend/test/` verifying conversation rendering and message sending.
- [ ] Run `npm test` and update `TEST_REPORT.md`.

### Additional considerations
- [ ] Handle authorization carefully: verify that only conversation participants or team members can access messages.
- [ ] Support pagination and ordering by `created_at` to efficiently load older messages.
- [ ] Provide graceful error handling and validation messages for empty content or unauthorized access.

## Security and Account Management

### Email Verification After Sign-up
- [ ] Generate a verification token and store it with the new user.
  - [ ] Add `email_verification_token` and `email_verification_sent_at` columns via an Alembic migration.
  - [ ] Include these fields in the SQLAlchemy `User` model and Pydantic schemas.
  - [ ] Generate the token using `secrets.token_urlsafe` during signup.
- [ ] Create `/verify-email` endpoint to validate tokens.
  - [ ] Accept the token and look up the associated user.
  - [ ] Set `email_verified_at` when the token matches and has not expired.
  - [ ] Return success or failure messages for the frontend.
- [ ] Send verification emails via background tasks.
  - [ ] Render HTML email templates with the verification link.
  - [ ] Dispatch using FastAPI `BackgroundTasks` and a pluggable email provider.
- [ ] Update frontend to prompt verification status.
  - [ ] Display a banner when the account is not verified.
  - [ ] Add a button to resend the verification email.

### System Account Security Review
- [ ] Audit current privileges and usage of the `system` account.
  - [ ] List all API endpoints and background jobs executed by this account.
  - [ ] Review database access roles and service integrations.
- [ ] Remove unnecessary permissions or convert to service accounts.
  - [ ] Create dedicated service accounts for automated tasks.
  - [ ] Reassign permissions and revoke direct `system` account logins.
- [ ] Document risks and recommended mitigations.
  - [ ] Summarize findings in `docs/security.md` with recommended changes.
  - [ ] Schedule a quarterly review of privileged accounts.

### Token Handling Improvements
- [x] Load `SECRET_KEY` and `ACCESS_TOKEN_EXPIRE_MINUTES` from environment variables instead of hardcoding them.
- [ ] Rotate the token signing key and invalidate tokens signed with the previous key.
  - [ ] Introduce a `token_version` column on the `User` model to force logouts.
  - [ ] Update JWT creation to embed the current version.
- [ ] Shorten the default token lifetime to minimize exposure of compromised credentials.
  - [ ] Make the expiry configurable via environment variables.
  - [ ] Document recommended values for production and development.
- [x] Implement session persistence so users remain logged in after a page refresh.
- [ ] Run cleanup job: delete guest users inactive for >30 days.
- [x] Implement login and signup with OAuth2/JWT authentication.

### Role Management
- [ ] Create a `roles` table and associate users.
  - [ ] Define default roles such as `user`, `coach`, and `admin`.
  - [ ] Map roles to allowed API scopes.
- [ ] Add role-based access checks across APIs.
  - [ ] Implement a dependency that verifies the current user's role.
  - [ ] Cover protected endpoints with unit tests.

### Onboarding Flow
- [ ] Introduce a first-time setup screen with quick tips.
  - [ ] Display a modal on first login with a short tour of key pages.
  - [ ] Dismiss the modal forever once completed.
- [ ] Allow choosing default preferences and sample plans.
  - [ ] Let new users select units (kg/lb) and theme during setup.
  - [ ] Offer to import a starter training plan from templates.

### Two-Factor Authentication (2FA)
- [ ] Implement optional two-factor authentication using TOTP codes.
  - [ ] Generate and display a QR code for authenticator apps.
  - [ ] Store hashed TOTP secrets in the database.
- [ ] Provide backup codes and account recovery flows.
  - [ ] Allow generating one-time-use recovery codes.
  - [ ] Add a support contact process for locked-out users.
- [ ] Update login screens to prompt for verification codes.
  - [ ] Include "Remember this device" option with expiring cookies.

### Rate Limiting and Abuse Protection
- [ ] Add per-IP and per-user rate limiting middleware.
  - [ ] Evaluate `slowapi` or `fastapi-limiter` for implementation.
  - [ ] Apply sensible defaults for anonymous users and authenticated users.
- [ ] Store throttle counters in Redis or the database.
  - [ ] Use Redis in development and production with configurable TTLs.
  - [ ] Write Alembic migrations if database-backed limits are required.
- [ ] Return standardized errors when limits are exceeded.
  - [ ] Follow RFC 6585 `429 Too Many Requests` semantics.
  - [ ] Include retry-after headers where appropriate.


## Dashboards and Reporting

### Calendar and Periodization Views
- [x] Build a calendar view to visualize planned workouts and logged activities.
- [x] Add a periodization timeline view to visualize training periods and milestones.
  - [x] Add day, week, and year calendar views for detailed scheduling perspectives.
- [ ] Enhance the calendar with Today/This Week snapshots highlighting completed vs. planned workouts.

### Progress Dashboard with Recharts
- [ ] Add API endpoints summarizing recent workouts and volume.
- [ ] Query TimescaleDB for aggregated statistics.
- [ ] Build React components using Recharts.
- [ ] Integrate results into the dashboard page with tests.

### Exercise History View and Workout Summary
- [x] Create endpoints to fetch historical logs with date filters. (MVP)
  - Extend the API router to accept `start_date` and `end_date` query parameters validated by a Pydantic model.
  - Filter logs using SQLAlchemy with `>=` and `<=` comparisons on `ActivityLog.actual_start_time`.
  - Sort results chronologically and return them via the existing `ActivityLog` schema.
  - Add unit tests to cover various date range scenarios and invalid input.
  - Update the React activity history view to pass the date range and display results chronologically.
- [ ] Show a summary page after each session highlighting totals and personal records. (MVP)
- [ ] Provide charts of progress over time.

### Body Metrics Tracking
- [ ] Design database tables for muscle groups and weight records.
- [ ] Implement CRUD APIs for metrics.
- [ ] Visualize trends with line charts.

## User Settings and Preferences

### Settings Page Enhancements
- [x] Provide profile editing and avatar uploads.
- [ ] Add forms for preference settings (units, rest timers, themes).
- [ ] Implement notification management UI and API.
- [ ] Persist settings per user and seed defaults during onboarding.

### App-wide Settings
- [ ] Centralize unit selection, rest timers, and theme choices.
- [ ] Expose these settings via APIs and the settings page.

## Planning and Templates

### Template List, Detail, and Comparison Views
- [ ] Implement search/filtering for templates.
- [ ] Provide a detail screen with JSON inspection and version history.
- [ ] Add comparison logic to show differences between versions.

### Plan Management Pages
- [ ] Build list and detail views sorted by date. (MVP)
- [ ] Support creating/editing plans from templates. (MVP)
- [ ] Include recurrence handling and metadata display. (MVP)
- [x] Enable plan creation and editing from templates or scratch with scheduling and recurrence.

### Additional Tools
- [ ] Build template list and search pages showing metadata like name, version, and creator.
- [ ] Add a template detail view with JSON inspection and version control actions (clone, edit, export).
- [x] Support importing and exporting templates through a JSON UI.
- [x] Add a recurrence rules editor for scheduling periods.
- [x] Include a raw JSON viewer for advanced users.
- [ ] Implement template comparison to show differences between versions.
- [x] Add a help and troubleshooting section linking to documentation.

## Workout Logging

### Live Workout Screen
- [ ] Show exercise info, timers, and record buttons. (MVP)
- [ ] Track set entry history to display previous results. (MVP)
- [ ] Provide pause, skip, and end controls. (MVP)

### Activity Logs
- [x] List past workouts chronologically. (MVP) — added Activity Logs page in frontend
- [ ] Offer a detailed view with structured data and raw JSON. (MVP)
- [ ] Allow editing and deleting entries. (MVP)
- [ ] Add a Freeform activity type combining lists, agendas, and workout forms into a single activity.

### Progress Graphs and Compliance Dashboard
- [ ] Chart personal records and volume trends.
- [ ] Compare completed sessions against planned ones on a calendar.

### Exercise Catalog and Details
- [ ] Implement filtering by tags, equipment, and complexity.
- [ ] Show muscle groups, aliases, metadata, and media.
- [ ] Global exercise bank exposing public system exercises.
- [ ] Build template from custom exercise.
- [ ] Clone system exercise for user customization.
- [ ] Sync exercise tags with templates.

### Coaching Features
- [ ] Enable linking coaches/trainers to athletes.
- [ ] Provide client lists, plan views, and log views with permissions.
- [ ] Add sharing options for templates or workouts.

- [ ] Allow inviting other users to a planned activity.
  - [ ] Implement relationship-based permissions to authorize invitations.
  - [ ] Provide UI for coaches or trainers to invite their entire team easily.

### User Relationships
- [ ] Add follow and unfollow endpoints with pending request support.
- [ ] Display follower and following lists in user settings.
- [ ] Provide accept or decline actions for relationship requests.

### Planned Activity Invitations
- [ ] Create an `activity_invitations` table linking planned activities to invitees.
- [ ] Extend relationship permissions with an `invite_activity` scope.
- [ ] Implement `/planned-activities/{id}/invite` and `/activity-invitations/{id}` endpoints.
- [ ] Send email or in-app notifications when invitations are sent or updated.

## User Interface Enhancements

### Theme Toggle
- [ ] Persist light/dark mode preference in user settings.
- [ ] Update MUI theme configuration accordingly.

### Dashboard Charts
- [ ] Seed example data and render Recharts graphs on the dashboard.

### Responsive Layout
- [ ] Ensure navigation collapses properly on mobile.
- [ ] Test forms and charts for usability on small screens.


### Accessibility and Internationalization
- [ ] Ensure WCAG 2.1 AA compliance across the UI.
- [ ] Provide translations with language switcher support.
- [ ] Verify keyboard navigation and screen reader compatibility.

### Mobile and Offline Support
- [ ] Implement a service worker for offline access.
- [ ] Optimize components for touch devices.
- [ ] Explore React Native or PWA distribution.

### Gamification Features
- [ ] Add achievement badges, leaderboards, and social sharing.

### Milestones and Badges
- [ ] Create CRUD API endpoints for planned milestones and milestone logs.
- [ ] Allow adding and editing milestones from the timeline view.
- [ ] Display earned badges on the user profile page.

## News Feed
- [ ] Create `feed_events` table with `user_id`, `event_type`, `content`, `metadata` JSON, and timestamps.
- [ ] Add optional `feed_visibility` table to scope events to specific users.
- [ ] Generate SQLAlchemy models and Alembic migrations for the feed tables.
- [ ] Support event types such as `workout_log`, `milestone`, `new_friend`, `plan_shared`, and `test_result`.
- [ ] Query feed events with a pull model using follower relationships and pagination.
- [ ] Provide a `GET /feed` API endpoint returning personalized events ordered by time.
- [ ] Build a React feed component with infinite scroll and day-grouped sections.
- [ ] Render profile picture, username, event message, and like/comment/share buttons for each item.
- [ ] Implement per-type renderers and inline comments in the frontend.
- [ ] Plan for push-model fan-out, ranking, caching, and deduplication as future enhancements.
- [ ] Display sponsored posts or ads at configurable intervals.
- [ ] Show articles or blog posts curated by admins.
- [ ] Surface announcements from teams the user belongs to.
- [ ] Include activity updates from users the viewer follows.
- [ ] List upcoming milestones and recently earned badges.
- [ ] Provide a summary widget with the day's or week's activity breakdown.
## Team Management Features

### Team Functionality
- [ ] Add a `teams` table with metadata like name, description, and owner.
- [ ] Create a `user_team` association table with roles and join dates.
- [ ] CRUD endpoints and tests for managing teams and memberships.
- [ ] Update existing APIs to include team membership details.
- [ ] Build React pages listing teams and showing team details.
- [ ] Implement group chat using WebSockets for team channels.
- [ ] Add bulk invitation support so coaches can invite an entire team with one action.
- [ ] Build React forms using MUI and React Hook Form for selecting users or teams.
- [ ] Write API and UI tests covering invite creation, acceptance, and team invites.

- [ ] Announcements and alerts via push, email, and in-app notifications.
- [ ] Group chat with threads, read receipts, and media attachments.
- [ ] File and photo sharing organized by event or chat thread.
- [ ] Event scheduling with calendar sync (Google/iCal).
- [ ] RSVP and availability tracking with aggregate views.
- [ ] Auto-scheduling tool for recurring events.
- [ ] Volunteer assignments with notifications and availability management.
- [ ] Team lineups and roster management with automated lineup creation.
- [ ] Invoice creation and payment processing with Stripe or Square.
- [ ] Bulk billing at the team or organization level.
- [ ] Live game updates with real-time commentary and media posts.
- [ ] Statistics tracking with leaderboards and performance reports.
- [ ] Roster and profile management with CSV import and photos.
- [ ] Online registration forms with fee collection and waiver uploads.
- [ ] Website or team page builder for schedules and news.
- [ ] Team store integration for merchandise sales.
- [ ] Sponsorship and fundraising tools.
- [ ] Support and resources hub for guides and training content.
- [ ] Event notifications and reminders by SMS, push, or email.
- [ ] Background check integration for volunteers and admins.
- [ ] Document storage for policies, waivers, and images.
- [ ] API endpoints for calendar, registration, and analytics integration.

## Productivity Integrations

### Gmail
- [ ] Register the application in Google Cloud Console and enable the Gmail API.
- [ ] Implement OAuth 2.0 with minimal scopes (`gmail.readonly`, `gmail.modify`).
- [ ] Sync message threads and monitor inbox updates using Gmail push notifications.
- [ ] Send automated emails, manage drafts, and apply labels via the Gmail API.
- [ ] Store and refresh Gmail OAuth tokens securely.

### Google Calendar
- [ ] Enable the Google Calendar API and configure OAuth credentials.
- [ ] Authenticate with `calendar.events` and `calendar.readonly` scopes.
- [ ] List, create, edit, and delete calendar events through the API.
- [ ] Provide embedded calendar views or iCal subscription links.
- [ ] Detect calendar changes using push channels or polling.
- [ ] Decide on one-way versus full two-way sync for calendars.

### Outlook
- [ ] Register the app with Azure AD and request `Mail.Read` and `Calendars.ReadWrite` scopes.
- [ ] Use Microsoft Graph to sync emails and calendar events.
- [ ] Create Graph webhook subscriptions for real-time changes.
- [ ] Support CalDAV or iCal links for Outlook desktop users.
- [ ] Implement bi-directional calendar sync with conflict resolution and deduplication.
- [ ] Provide UI for selecting which calendars should sync across providers.

### Notifications and Tasks
- [ ] Add Slack, Teams, or Google Chat notifications for new messages or schedule updates.
- [ ] Offer Zapier or Make integrations for custom workflows.
- [ ] Sync tasks with Google Tasks, Microsoft To-Do, or Trello.
- [ ] Apply least-privilege scopes, secure token storage, and webhook retries across all integrations.
- [ ] Display sync status, last sync time, and connected accounts in user settings.

## Fitness Platform Integrations

### Strava
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes: `read`, `activity:read_all`.

### Garmin Connect
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes: `wellness`, `activity`, `profile`.

### Google Fit
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes: `fitness.activity.read`, `fitness.body.read`.

### Apple HealthKit
- [ ] Request read/write permissions for HealthKit data types such as workouts and body metrics.
- [ ] Document required entitlements and data type authorizations.

### MyFitnessPal
- [ ] Register the app and implement OAuth flow or other approved API access.
- [ ] Document required OAuth scopes for diary entries, weight data, and activity sync.

### Withings
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes: `user.metrics`, `user.activity`, `user.info`.

### Under Armour
- [ ] Register with the MapMyFitness developer program and implement OAuth flow.
- [ ] Document required OAuth scopes: `read`, `write`, `profile`.

### HeiaHeia
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes for activities, wellness data, and friends list.

### FitnessSyncer
- [ ] Register the app and implement OAuth flow.
- [ ] Document required OAuth scopes to pull aggregated activity and health data.

- [ ] Fetch and normalize workout and activity data across providers.
- [ ] Support bi-directional sync and handle conflicts, duplicates, and deletions.
- [ ] Implement webhooks or subscriptions for push updates with retry and deduplication.
- [ ] Store raw and normalized data separately and respect provider privacy policies.
- [ ] Schedule background sync jobs and offer a manual "Sync now" action showing the last sync time.
- [ ] Display connected accounts and imported activities in the UI with source tags.
- [ ] Aggregate data from multiple platforms and build analytics dashboards.
- [ ] Test each integration end-to-end, including token revocation and API limits.
- [ ] Ensure compliance with platform terms and regulations such as HIPAA/GDPR.
- [ ] Document setup steps, scopes, and known limitations for each platform.
- [ ] Plan phased rollout:
  1. OAuth and read-only sync for Strava, Garmin, and Google Fit.
  2. Add Fitbit, HealthKit, and MyFitnessPal with activity normalization.
  3. Implement write-back features, webhooks, and two-way sync.
  4. Extend to Withings, Under Armour, and HeiaHeia.
  5. Build hub-like features and analytics dashboards.

## Mobile App Development

### Offline API Support
- [ ] Provide endpoints to deliver user activities and logs in bulk or by date range.
- [ ] Include `updated_at` timestamps so the mobile app can fetch only new or modified records since the last sync.
- [ ] Secure endpoints with JWT-based authentication.

### Local Storage
- [ ] Choose a persistent store such as SQLite or Realm on the device.
- [ ] Mirror the server models for activities and logs locally.
- [ ] Populate and update this store on login and during periodic syncs.

### Sync Logic
- [ ] Create a sync module triggered when connectivity returns or via a manual "Sync" button.
- [ ] Fetch updates since `last_synced_at` and merge them into the local store.
- [ ] Queue failed syncs and gracefully handle token expiration.

### Data Versioning
- [ ] Track `last_synced_at` locally and filter server records using `updated_at`.
- [ ] Include deleted item markers so the mobile database stays consistent.

### Offline UI
- [ ] Display cached data when offline and clearly indicate offline mode.
- [ ] Provide a manual refresh option to retry synchronization.

### Background Sync (iOS and Android)
- [ ] Implement background tasks using iOS Background Fetch and Android WorkManager to sync data periodically.
- [ ] Respect battery usage and data limits, exposing settings to control frequency.

### Testing and Validation
- [ ] Test partial syncs, network interruptions, and token renewal scenarios.
- [ ] Verify local database updates correctly with inserts, updates, and deletions.
- [ ] Simulate extended offline periods to confirm cached data remains accessible.

### Platform Details
- [ ] Plan distribution through the Apple App Store and Google Play Store.
- [ ] Utilize Core Data or SQLite on iOS and Room or SQLite on Android.
- [ ] Document differences in push notification handling between APNs and FCM.
- [ ] Validate layouts across various screen sizes and OS versions.

### Cross-Platform Migration
- [ ] Wrap the existing React web app using Capacitor for early App Store deployment.
  - [ ] Configure iOS and Android build scripts in Capacitor.
  - [ ] Verify web features work within the Capacitor shell.
- [ ] Set up an Expo monorepo with React Native Web for long-term development.
  - [ ] Map current MUI components to React Native Paper or Tamagui equivalents.
  - [ ] Replace DOM layout (`div`, `span`, CSS classes) with `View` and `Text`.
  - [ ] Replace Recharts with Victory Native or React Native SVG Charts.
  - [ ] Replace FullCalendar with React Native Calendars and Expo DateTime Picker.
- [ ] Document component mapping and migration order starting with mobile-heavy pages.
