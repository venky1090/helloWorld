Status: ready-for-agent

# 13 — Parent dashboard: history, weekly summary, periodic jobs

## What to build

The parent's visibility into the child's progress, plus the background jobs that
keep derived data fresh. Inside the PIN-gated parent area: a Child History screen
(completed vs. missed tasks), a Weekly Summary of completed/missed tasks for
informed conversations, and the child's total points and badges earned.
Non-time-critical periodic work runs via `WorkManager`: a daily badge check, a
periodic history cleanup, and weekly summary generation. (Time-critical alarms
remain on `AlarmManager` per the scheduling approach.)

## Acceptance criteria

- [ ] Child History screen shows completed and missed tasks
- [ ] Weekly Summary aggregates completed/missed tasks for the week
- [ ] Parent dashboard shows total points and badges earned
- [ ] `WorkManager` runs a daily badge check, history cleanup, and weekly summary
      generation
- [ ] All of the above are behind the PIN gate

## Blocked by

- 05 — task completions + points ledger
- 09 — badges

## Open questions

- **History cleanup retention (non-blocking):** the retention window for history
  cleanup is unspecified. Implement with a sensible default and confirm.
