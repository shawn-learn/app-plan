# Planned Workout Editor Technical Support Manual

This document explains how the planned workout editor selects which parameters to display for each exercise and outlines troubleshooting steps for support staff.

## Parameter Selection Workflow

1. **Fetch Parameter Rules**  
   When the editor initializes, it requests parameter rule groups from `/users/me/parameter-rules/details`. Each rule group defines a primary parameter (e.g., reps, duration) and the additional parameters visible for each exercise category.

2. **Map Visible Parameters**  
   The returned rule groups are transformed into a `visibleParamMap`. Only rules marked as visible are included. The map indexes extra parameter definitions by exercise category and primary parameter.

3. **Select Editor & Apply Parameters**  
   For every exercise in the workout, `WorkoutItemEditor` chooses an editor component based on the exercise's `primaryParameter` (reps, duration, distance, or elevation). It then looks up extra parameters for the exercise's category in `visibleParamMap` and passes them to the editor as `extraParams`.

4. **Render Additional Fields**  
   Specialized editors such as `ExerciseRepsEditor` or `ExerciseDurationEditor` iterate over `extraParams` to render the corresponding input fields alongside the primary parameter input.

## Troubleshooting Tips

- **Parameters Missing**: Verify the API response contains the expected rule groups and that the rules are marked `visible`.
- **Incorrect Editor**: Confirm each exercise has the correct `primaryParameter`. Mismatched types cause the wrong editor component to load.
- **Extra Fields Not Saving**: Ensure the editor passes `extraParams` through to the save handler and that the backend accepts those fields.

Understanding this workflow helps diagnose why certain parameters appear or are missing when editing planned workouts.
