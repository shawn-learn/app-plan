# team-service

## 1. Purpose

`team-service` owns named groups of users — teams — together with their memberships, tags, and the share-permission tables that let teams (rather than individuals) be granted access to templates, exercises, exercise templates, and (eventually) workouts and achievements. It is the authoritative source for "is user U a member of team T?" and "what does team T see?"

## 2. Scope

**In scope**
- Teams (`/teams`): create, rename, delete, list members.
- Team memberships and roles.
- Team tags and the link table to teams.
- Share-permission resolution against team membership (used by `exercise-service` and `template-service`).
- Coach/practitioner role flagging on memberships (foundation for messaging E2EE rules).

**Out of scope**
- Cross-team messaging (→ `messaging-service`, which depends on this lib).
- The shared assets themselves (templates / exercises) — only the team side of the share rule lives here.

## 3. Current Capabilities

Routers: `app/routers/teams.py`, `team_tags.py`. Service: `app/services/teams.py`. Schemas: `app/schemas/teams.py`, `team_tags.py`. Tables (per README): `teams`, `team_memberships`, `team_tags`, `team_tag_links`. Frontend: `TeamManagerPage.jsx`.

## 4. Public API Surface

- HTTP: `/teams`, `/teams/{id}/members`, `/team-tags`.
- Python: `is_member(user, team)`, `members_of(team)`, `teams_for(user)`, `evaluate_team_share(asset, user)`.

## 5. Data Model

Owns: `teams`, `team_memberships`, `team_tags`, `team_tag_links`. Reads (but does not own): the share tables in `exercise-service` and `template-service`.

## 6. Dependencies

- Depends on: `user-service`, `auth-service`.
- Must NOT depend on: `exercise-service`, `template-service`, `messaging-service`.

## 7. Cross-Cutting Concerns

- **Roles**: a membership has a role (member, coach, practitioner) — coaches and practitioners gate features in other libs.
- **Audit**: membership changes should be logged via `content-service`'s `system_log`.
- **Privacy**: a user's team list is visible only to teammates and the user themselves.

## 8. Open Questions / Known Gaps

- The role taxonomy is informal today; messaging E2EE rules depend on it being formalized.
- "Team broadcast" (one-way announcements) is named in `FEATURES_OVERVIEW.md` but its data model lives in `messaging-service`, with team-service supplying only the audience.
