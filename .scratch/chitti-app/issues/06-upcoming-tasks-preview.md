Status: ready-for-agent

# 06 — Upcoming tasks preview

## What to build

A child-facing preview of upcoming tasks for the next few days so the child can
plan ahead. `TaskEngine.getUpcoming(days: Int)` computes the scheduled tasks for
the next 7 days from the schedule rules, surfaced as an "upcoming" tab on
`KidHomeScreen`.

## Acceptance criteria

- [ ] `getUpcoming(7)` returns scheduled tasks for the next 7 days derived from
      schedule rules
- [ ] `KidHomeScreen` has an upcoming tab showing these grouped by day
- [ ] Inactive tasks are excluded from the upcoming list

## Blocked by

- 05 — kid home + task engine

## Open questions

None.
