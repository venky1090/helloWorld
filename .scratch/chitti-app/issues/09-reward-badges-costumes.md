Status: needs-info

# 09 — Reward system: badges, costumes, costume selector

## What to build

The motivation layer on top of the points ledger. Badge and costume unlocks are
evaluated against the **current balance** (single-balance model — no separate
lifetime-earned total), so redeeming points can drop the balance below a
threshold and remove access to a higher-tier unlock. `checkNewBadges()` returns
newly crossed badges; `getUnlockedCostumes()` returns costumes whose milestone
the current balance meets; `setActiveCostume()` persists the child's selection on
`ChildProfile` and only succeeds for a currently-unlocked costume. `KidHomeScreen`
shows badge count, current level, and a costume selector.

## Acceptance criteria

- [ ] Awarding points across a badge threshold makes `checkNewBadges()` return
      the new badge
- [ ] Points reaching a costume milestone makes `getUnlockedCostumes()` include
      that costume
- [ ] Single-balance rule: after a redemption drops the balance below a
      milestone, `getUnlockedCostumes()` no longer returns that costume
- [ ] `setActiveCostume()` persists the selection and succeeds only for a
      currently-unlocked costume
- [ ] `KidHomeScreen` shows badge count, level, and a working costume selector

## Blocked by

- 05 — points ledger (`awardPoints`, `getBalance`)
- 08 — Chitti character (costume Lottie assets)

## Open questions

- **Badge & costume tables (blocking):** the PRD gives only examples
  ("50pts = Bronze badge, 200pts = Silver"). The full static tables are required
  to implement this slice:
  - Badge tiers: names + point thresholds (complete list).
  - Level rules: how levels map to points.
  - Costume milestones: point thresholds → Lottie costume asset names.
  Until these are supplied, the unlock logic cannot be finalised. This is why the
  status is `needs-info`. Acceptance-criteria *logic* can be built against a
  placeholder table, but the real values must be confirmed before merge.
