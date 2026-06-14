Status: ready-for-agent

# 05 — Kid home: today's tasks + mark done + points (honor system)

## What to build

The core child loop. `KidHomeScreen` shows today's pending task checklist; the
child taps "Done" to complete a task (honor system), which records a completion
and immediately awards points. The screen shows two distinct figures: points
earned today and current points balance.

`TaskEngine.getPendingToday()` returns **all** tasks scheduled for today's
day-of-week that are not yet completed, regardless of time of day (so the child
sees later-today tasks too). `markComplete(taskId)` records a completion
timestamp and must not re-surface the same task as pending for the same day.
`RewardSystem` maintains a single append-only points ledger (`PointTransaction`):
`awardPoints` writes positive rows, `getBalance()` is the running sum,
`getEarnedToday()` sums only positive rows dated today.

The "today" boundary is midnight (00:00) rollover.

## Acceptance criteria

- [ ] `getPendingToday()` returns all of today's not-yet-completed scheduled
      tasks regardless of time of day
- [ ] Each task row shows name, estimated duration, and a "Done" button
- [ ] Tapping "Done" calls `markComplete`, records a `TaskCompletion`, and awards
      the task's points via `awardPoints`
- [ ] A completed task does not re-appear as pending for the same day
- [ ] Home screen shows points-earned-today and current balance as two figures
- [ ] Ledger is append-only; `getBalance()` reflects the running sum;
      `getEarnedToday()` counts only today's positive rows

## Blocked by

- 04 — task model + CRUD

## Open questions

None.
