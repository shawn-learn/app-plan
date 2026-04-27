# 🏋️ Workout Logging System – User Stories

This document outlines user stories and requirements for a workout logging system designed to support diverse exercise modalities and flexible training structures. The system must handle:

- Strength workouts (sets, reps, weight, tempo, RPE)
- Supersets, circuits, AMRAP, EMOM, and pyramid sets
- Cardio and endurance (intervals, distance, time, pace)
- HIIT, CrossFit, bodyweight, and assistance/resistance equipment
- Yoga, mobility flows, and static holds
- Flexibility for future extensions

The core architecture will use a hybrid SQL + JSON model to ensure performance, consistency, and adaptability.

---


## ✅ MVP Functionality

### 🧱 Workout Templates

- **[MVP-01]** As a user, I want to create a workout template with a list of exercises, sets, reps, weight, rest, and optional metadata so I can reuse it regularly.
- **[MVP-02]** As a user, I want to define supersets, circuits, or interval structures within a workout template so I can support diverse training methods.
- **[MVP-03]** As a user, I want to edit existing workout templates so I can make updates as needed.
- **[MVP-04]** As a user, I want to duplicate a template and modify it to quickly create variations.

### 📝 Workout Logging

- **[MVP-05]** As a user, I want to select a template and log a workout so that I can record my activity quickly.
- **[MVP-06]** As a user, I want to modify the workout during logging in case I change sets, reps, or exercises.
- **[MVP-07]** As a user, I want to log free-form workouts without using a template to allow flexibility.
- **[MVP-08]** As a user, I want to track rest, tempo, RPE, duration, or distance per exercise or set as appropriate.

### 📊 Progress Tracking

- **[MVP-09]** As a user, I want to view my performance history for a specific exercise so I can track improvements.
- **[MVP-10]** As a user, I want to see basic charts of volume, weight, or estimated 1RM over time.
- **[MVP-11]** As a user, I want to filter my workout history by date and exercise to identify trends.

### 📦 Developer/Admin Tools

- **[MVP-12]** As a developer, I want to seed the database with common exercises across modalities (strength, cardio, mobility, etc.).
- **[MVP-13]** As a developer, I want to normalize exercise names to keep data clean and queryable.
- **[MVP-14]** As a developer, I want to log user activity and input errors for debugging and audit purposes.

---

## 🌱 Future Functionality

### 🗓 Workout Scheduling

- **[FUT-01]** As a user, I want to schedule upcoming workouts using templates so I can plan my training week.
- **[FUT-02]** As a user, I want to set recurring workouts (e.g., "Push Day every Monday") so I don’t have to re-enter them.

### 🧩 Customization

- **[FUT-03]** As a user, I want to tag exercises and workouts by muscle group, goal, or complexity (e.g., beginner, intermediate) to improve organization.
- **[FUT-04]** As a user, I want to add personal notes to individual sets for subjective feedback.

### 🧠 Intelligence Features

- **[FUT-05]** As a user, I want the app to suggest weight or rep adjustments based on past performance to help me progress.
- **[FUT-06]** As a user, I want to get feedback if I’m plateauing so I can adjust my routine.

### 📱 Platform Integration

- **[FUT-07]** As a user, I want to receive reminders for scheduled workouts.
- **[FUT-08]** As a user, I want to sync my completed workouts to my calendar or Apple Health.
- **[FUT-09]** As a user, I want to export my workout history to CSV or JSON for external use or backup.

---

## ⚙️ Future Considerations

- Coach/trainer view to manage multiple users
- Mobile app for offline logging
- AI chatbot for conversational workout building and logging
- Support for heart rate zones, wearable data, and effort scoring
