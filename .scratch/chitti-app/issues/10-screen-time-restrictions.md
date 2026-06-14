Status: ready-for-agent

# 10 — Screen-time restrictions: parent config + warning + block

## What to build

Device-wide, clock-based screen-time windows. A `ScreenTimeRule` Room entity
(start time, end time, days active) with a `RestrictionRepository`, a Parent
Screen Time Rules UI to configure windows, and automatic enforcement via
`AlarmScheduler`: a 10-minute warning alarm and a hard-cutoff alarm per active
window. The 10-minute warning fires a high-priority notification with
`setFullScreenIntent` → `ChittiWarningActivity` (countdown + encouraging wrap-up
message). The hard cutoff launches `ChittiBlockActivity` full-screen with a
positive time's-up message.

`checkCurrentlyRestricted()` returns true only inside a configured window on a
matching day, evaluated correctly for windows spanning midnight (e.g. 10pm–7am).
Deactivated rules are not evaluated. The BootReceiver from slice 07 is extended
to re-register screen-time alarms on reboot.

**Known MVP limitation:** without `SYSTEM_ALERT_WINDOW`, the child can exit
`ChittiBlockActivity` via the Home button — this is a soft deterrent only, and
parents are told so during onboarding. Full enforcement (overlay) is v2.

## Acceptance criteria

- [ ] Parent can configure a window (start, end, active days), persisted in Room
- [ ] 10-min warning fires a `setFullScreenIntent` notification →
      `ChittiWarningActivity` with countdown + wrap-up message
- [ ] Hard cutoff launches `ChittiBlockActivity` full-screen with a positive
      time's-up message
- [ ] `checkCurrentlyRestricted()` is true only within a window on a matching day
- [ ] A window spanning midnight is evaluated correctly on both sides
- [ ] Deactivated rules are not evaluated
- [ ] Screen-time alarms re-register on boot

## Blocked by

- 04 — parent area shell
- 08 — Chitti character used in the warning/block activities

## Open questions

- **Re-launch policy (clarify):** without `PACKAGE_USAGE_STATS`, the block can
  only fire at the cutoff alarm. If the child dismisses `ChittiBlockActivity` via
  Home, the PRD's "soft deterrent" is silent on whether/when Chitti re-launches
  (e.g. on app foreground, on a repeating alarm, or not at all). Confirm the
  intended MVP behaviour.
- **Design-vs-PRD conflict (note):** `plans/chitti-design.md` and
  `plans/base-idea.md` describe an *activity gate* (eye exercises / jumping jacks
  before the timer unlocks) on the block screen. The PRD explicitly lists the
  activity gate as **out of scope (v2)**. This slice follows the PRD (no gate);
  raised here in case the product owner intends otherwise.
