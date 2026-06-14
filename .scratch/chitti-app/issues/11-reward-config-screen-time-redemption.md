Status: ready-for-agent

# 11 — Reward config + screen-time redemption

## What to build

The optional closed loop that turns earned points into extra screen time. A
`RewardConfig` Room entity holds the parent's settings: an enable/disable toggle
for screen-time bonus redemption and the conversion rate (how many points equal
how many extra minutes). A Parent Reward Config UI edits these. When enabled, the
child taps a "Redeem" action on `KidHomeScreen`:
`RewardSystem.redeemForScreenTime(points)` computes minutes at the configured
rate, calls `ScreenTimeRestrictions.creditExtraMinutes(minutes)` (which moves the
hard-cutoff alarm forward), and records a **negative** ledger row so the balance
decreases by the redeemed points. Chitti celebrates the redemption.

Redemption is unavailable (or hidden) when the parent toggle is off.

## Acceptance criteria

- [ ] Parent can toggle screen-time bonus on/off and set the points→minutes rate
- [ ] With the toggle off, redemption is not available to the child
- [ ] "Redeem" computes the correct minutes for the balance and configured rate
- [ ] `creditExtraMinutes()` extends the effective cutoff time (cutoff alarm
      rescheduled)
- [ ] A negative ledger row is recorded; `getBalance()` decreases by the redeemed
      points (ledger stays append-only)
- [ ] Chitti celebrates on successful redemption

## Blocked by

- 09 — reward balance + ledger semantics
- 10 — `creditExtraMinutes` / cutoff alarm

## Open questions

- **Default rate & bounds (non-blocking):** the PRD does not give a default
  points→minutes rate or min/max bounds. Implement with a sensible default and
  validation; confirm values with the product owner.
