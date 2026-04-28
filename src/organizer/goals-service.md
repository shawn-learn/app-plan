# goals-service

## 1. Purpose

`goals-service` owns goal-tracking on top of the activity stream: planned milestones embedded in training periods, the milestone log that records when each milestone was hit, and the badge catalog that ties achievements to milestone completions.

## 2. Scope

**In scope**
- Planned milestones (`/milestones`) inside a period.
- Milestone log (a milestone completion record).
- Badges and the rules that award them on milestone completion.
- Read APIs that surface "next milestone" / "recent achievements" for the dashboard.

**Out of scope**
- The activity log that *evidences* a milestone (→ `activity-service`); a milestone references logs but does not own them.
- Period boundaries (→ `calendar-service`).
- Personal records (PRs) — these are a metric concept owned by `activity-service`.

## 3. Current Capabilities

Router: `app/routers/milestones.py`. The README data-model section names `badges`, `planned_milestones`, and `milestone_log` as the supporting tables. Frontend exposure today is minimal — milestones surface in the dashboard and progress views (`Dashboard.jsx`, `MetricsPage.jsx`).

## 4. Public API Surface

- HTTP: `GET/POST/PATCH /milestones`, `POST /milestones/{id}/complete`, `GET /badges`, `GET /users/me/badges`.
- Python: `award_badges(user, milestone_log_entry)`, `next_milestone(user)`.

## 5. Data Model

Owns: `badges`, `planned_milestones`, `milestone_log`. Reads from: `periods` (calendar-service), `activity_logs` (activity-service).

## 6. Dependencies

- Depends on: `activity-service`, `user-service`, `auth-service`, `shared-schemas`. Reads (but does not depend tightly on) `calendar-service`.
- Must NOT depend on: `team-service` (sharing of achievements happens through team-service consuming this lib's read API).

## 7. Cross-Cutting Concerns

- **Determinism**: badge-award rules must be deterministic — replaying the same milestone-log sequence yields the same badges.
- **Visibility**: badge visibility is governed by user-service privacy settings.

## 8. Open Questions / Known Gaps

- No formal "personal record" surface; `User_Stories.md` MVP-09/10 imply this lives in metrics, not goals — boundary needs confirmation.
- Notifications when a milestone is hit (FUT-07) are not yet routed.
- The relationship between badges and team-broadcast announcements (in messaging-service) is unspecified.
