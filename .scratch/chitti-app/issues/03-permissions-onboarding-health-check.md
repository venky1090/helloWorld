Status: ready-for-agent

# 03 — Permission onboarding + Permission Health Check

## What to build

An onboarding flow that walks the parent through granting the MVP runtime/manifest
permissions, plus a Permission Health Check screen that detects any of them being
revoked later and guides the parent to re-enable them — so the app keeps working
reliably after setup.

MVP permissions:
- `POST_NOTIFICATIONS` — task reminders and screen time warnings
- `FOREGROUND_SERVICE` (+ the `FOREGROUND_SERVICE_*` subtype required by the
  target SDK) — persistent timer/reminder service
- `RECEIVE_BOOT_COMPLETED` — re-register alarms and restart the service on reboot
- `RECORD_AUDIO` — child voice input (wake word + utterance capture)

## Acceptance criteria

- [ ] Onboarding requests each permission with a child-friendly rationale
- [ ] Permission Health Check screen lists each permission's current granted state
- [ ] When a required permission is revoked, the screen flags it and routes the
      parent to re-enable it
- [ ] App degrades gracefully (does not crash) when a permission is missing

## Blocked by

- 01 — app scaffold (manifest, foreground service)
- 02 — parent area (health check lives behind the PIN gate)

## Open questions

- **target SDK (blocking, shared with 01):** the exact `FOREGROUND_SERVICE_*`
  subtype permission to declare depends on the pinned target SDK. Resolve in 01.
