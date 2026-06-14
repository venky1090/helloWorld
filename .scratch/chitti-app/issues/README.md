# Chitti MVP — implementation issues

Derived from `../PRD.md` as vertical tracer-bullet slices. Each issue file has a
`Status:` line: `ready-for-agent` (fully specified, AFK-runnable),
`ready-for-human` (needs human setup/decisions), or `needs-info` (blocked on an
open question — see that issue's `## Open questions`).

## Issue index & dependencies

| # | Slice | Status | Blocked by | Stories |
|---|---|---|---|---|
| 01 | App scaffold / walking skeleton | ready-for-human | — | foundation |
| 02 | Parent PIN gate + child profile | ready-for-agent | 01 | 29, 32 |
| 03 | Permission onboarding + health check | ready-for-agent | 01, 02 | 30, 31 |
| 04 | Parent task CRUD + scheduling model | ready-for-agent | 01, 02 | 17–22 |
| 05 | Kid home: today's tasks + done + points | ready-for-agent | 04 | 1, 4, 6 |
| 06 | Upcoming tasks preview | ready-for-agent | 05 | 2 |
| 07 | Task reminders: alarms + escalation | ready-for-agent | 05 | 3 |
| 08 | ChittiCharacter: Lottie + TTS | needs-info | 01, 05 | 5 |
| 09 | Reward: badges, costumes, selector | needs-info | 05, 08 | 7, 8 |
| 10 | Screen-time restrictions | ready-for-agent | 04, 08 | 9, 10, 11, 23, 24 |
| 11 | Reward config + screen-time redemption | ready-for-agent | 09, 10 | 12, 25, 26 |
| 12a | Q&A tap-to-talk (Claude + offline) | needs-info | 08 | 13, 14, 15, 16 |
| 12b | Q&A wake word ("Hey Chitti") | needs-info | 12a | 13 |
| 13 | Parent dashboard: history + summary | ready-for-agent | 05, 09 | 27, 28 |

All 32 PRD user stories are covered. Dependencies form a DAG (build in numeric
order).

## Flagged ambiguities (blocking — resolve before the noted issue can merge)

1. **Build/test environment + min/target SDK** (issue 01) — how slices are
   verified (device vs CI) and the pinned SDK levels.
2. **Badge / costume / level tables** (issue 09) — only examples in the PRD; need
   the full threshold tables + costume asset names.
3. **Claude API** (issue 12a) — model id, dev API-key provisioning, exact kid-safe
   system-prompt text.
4. **Wake-word engine** (issue 12b) — engine choice, licensing/access key, custom
   "Hey Chitti" keyword training.
5. **Lottie placeholder asset** (issue 08) — confirm the agent may pick a free
   placeholder and document the state mapping.

## Non-blocking flags (sensible defaults; confirm with product owner)

- Chitti phrase copy (08), points→minutes default rate/bounds (11), screen-time
  re-launch policy after Home-dismiss (10), history-cleanup retention (13).

## Note: design-doc vs PRD conflict

`plans/chitti-design.md` and `plans/base-idea.md` describe an **activity gate**
(eye exercises / jumping jacks before the screen-time timer unlocks). The PRD
lists this as **out of scope (v2)**. These issues follow the PRD (no gate); see
issue 10 if the product owner intends otherwise.
