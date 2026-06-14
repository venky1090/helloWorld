Status: ready-for-agent

# 07 — Task reminders: alarms + tiered notification + escalation

## What to build

The full task reminder flow. When a task is saved or modified, `TaskEngine`
registers/cancels an exact alarm via `AlarmScheduler`
(`setExactAndAllowWhileIdle()` for Doze). On fire, `NotificationService` posts a
high-priority task-reminder notification. If the child does not acknowledge
within 5 minutes, a secondary escalation alarm (owned by `NotificationService`)
launches `ChittiFullScreenActivity` with the task prompt and a "Mark Done" button.
`TaskEngine.markComplete()` cancels the pending escalation alarm.

Single owner for escalation: `NotificationService` schedules and cancels the
5-minute escalation; `TaskEngine` does not. Edge cases: device off at task time →
fire on next boot, but **skip if more than 2 hours past due**; two tasks at the
same time fire as independent alarms. A `RECEIVE_BOOT_COMPLETED` BootReceiver
re-registers all active task alarms and restarts the foreground service on boot.

Data flow: `AlarmManager` → `NotificationService` posts notification → (5 min, no
ack) → escalation alarm → `ChittiFullScreenActivity` → `markComplete()` →
`awardPoints()` → (celebration handled in slice 08).

## Acceptance criteria

- [ ] Saving/modifying a task registers/cancels its exact alarm via
      `AlarmScheduler` using `setExactAndAllowWhileIdle()`
- [ ] On alarm fire, a high-priority task-reminder notification is posted
- [ ] If unacknowledged after 5 min, escalation launches
      `ChittiFullScreenActivity` with a "Mark Done" button
- [ ] `markComplete()` cancels the pending escalation alarm
- [ ] A task missed by > 2 hours on boot is skipped, not fired
- [ ] Two tasks at the same time fire independently
- [ ] BootReceiver re-registers active task alarms and restarts the foreground
      service after reboot

## Blocked by

- 05 — kid home, `markComplete`, reward award

## Open questions

None — escalation ownership and edge cases are specified in PRD `TaskEngine` /
`NotificationService`.
