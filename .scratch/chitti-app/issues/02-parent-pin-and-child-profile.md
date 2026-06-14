Status: ready-for-agent

# 02 — Parent PIN gate + child profile creation

## What to build

The PIN-gated entry to the parent area and first-run child profile setup. A
`PINGuard` validates a PIN before any parent screen is shown; the PIN is stored
as a bcrypt hash in `EncryptedSharedPreferences` (Jetpack Security), never in
plain text, defaulting to `0000` on first install. The parent can create a
`ChildProfile` with a name and age so Chitti can personalise interactions.

MVP PIN recovery is reinstall-only (resets to `0000`); email/security-question
recovery is out of scope.

## Acceptance criteria

- [ ] Parent area is gated by a PIN entry dialog shown on every parent-screen
      entry
- [ ] PIN stored as a bcrypt hash in `EncryptedSharedPreferences`; no plain-text
      PIN written to disk
- [ ] Default PIN is `0000` on first install
- [ ] Parent can set/change the PIN
- [ ] Parent can create a `ChildProfile` (name, age), persisted in Room
- [ ] Incorrect PIN is rejected; correct PIN unlocks the parent area

## Blocked by

- 01 — app scaffold (Room, Hilt, parent navigation shell)

## Open questions

None — covered by PRD "PIN Storage" section.
