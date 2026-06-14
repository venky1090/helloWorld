Status: ready-for-agent

# 04 — Parent task CRUD + scheduling model

## What to build

The parent-facing task management surface and its data model. `Task` and
`TaskSchedule` Room entities (with a `TaskRepository`) and a Parent Task List +
Task Editor UI that lets a parent create, edit, delete, and toggle tasks
active/inactive. Each task has a name, estimated duration, a point value, and a
schedule: repeat daily, on specific weekdays, or on custom days, plus a fixed
time of day.

This slice owns the schedule model and CRUD only. Alarm registration on save is
added in slice 07 (routed through `AlarmScheduler`); this slice may call into the
scheduler as a no-op stub.

## Acceptance criteria

- [ ] Parent can create a task with name, duration, point value, schedule, and
      fixed time
- [ ] Schedule supports daily, specific-weekday, and custom-day repetition
- [ ] Parent can edit and delete tasks
- [ ] Parent can toggle a task active/inactive without deleting it
- [ ] Tasks and schedules persist in Room via `TaskRepository`
- [ ] Task list reflects active/inactive state

## Blocked by

- 01 — app scaffold (Room, Hilt)
- 02 — parent area (PIN gate, navigation)

## Open questions

None — covered by PRD user stories 17–22 and the `TaskEngine` interface.
