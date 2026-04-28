# Features and Capabilities

This document provides an overview of the main functionality offered by the workout logging application. It complements the architecture details in the [README](../README.md) and other documentation.

## Core Features

- **Template Builder** – Design reusable workout templates with sections, supersets, circuits and progression rules. Templates are stored in JSON for easy version control.
- **Workout Planning** – Schedule personalized sessions derived from templates or created ad-hoc. Recurrence rules and periods allow for long-term training plans.
- **Activity Logging** – Record sets, reps, weight, duration and other metrics using structured tables plus JSON fields for flexibility.
- **Progress Tracking** – Visualize historical performance with charts and graphs in the React frontend.
- **Milestones and Badges** – Define goals and automatically award badges when milestones are logged.
- **Exercise Management** – Maintain a catalog of exercises with muscle groups, equipment requirements and complexity ratings. Names and alternate spellings are stored in a dedicated alias table and resolved via the `/exercises/lookup` endpoint.
- **Team Collaboration** – Share templates, workouts and achievements with team members. Permissions are managed through share tables.
- **Messaging & Announcements** – Real-time chats, groups, team broadcasts and practitioner announcements with E2EE for physiotherapist↔patient conversations.
- **Import and Export** – Command line tools allow templates to be loaded from or saved to disk for backup and sharing.
- **Authentication** – OAuth2 with JWT secures the API. Users can manage personal settings and parameter visibility.
- **Offline Sync** – Helper functions enable synchronization with an offline database so workouts can be logged without connectivity.

## Frontend Highlights

- **React SPA** – Built with Vite, Material UI, React Hook Form and Recharts.
- **Template Editing** – Load a template from disk, tweak the title or structure and export it back.
- **Calendar Views** – Browse workouts by day, week, month or year.
- **Activity History** – Filter and review completed sessions.
- **Settings Page** – Customize visible exercise parameters and other preferences.

These capabilities will expand as the project evolves. See the [Feature TODO list](TODO_features.md) for planned enhancements.
