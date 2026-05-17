Status: ready-for-agent

# PRD: Chitti — Kid Companion Android App (MVP)

## Problem Statement

Parents struggle to enforce healthy digital habits for their children aged 8–12 in a way that feels positive rather than punitive. Existing parental control apps (Google Family Link, Bark) block and restrict, but do not engage — children feel policed, not guided. There is no companion that reframes screen time limits as opportunities, encourages task completion, and answers children's questions in a safe, age-appropriate way.

---

## Solution

Chitti is an Android companion app featuring a cartoon character that lives on the child's device. Unlike enforcement-only parental control tools, Chitti acts as a positive companion — not a warden:
- Reminds children about parent-configured tasks (study, music practice, etc.) with warmth and encouragement
- Reframes device-wide screen time restrictions positively ("We've got 10 mins left — let's make it count!")
- Awards points for task completion, unlocking badges, costume changes, and optional extra screen time
- Answers children's questions in multi-turn voice conversations using a kid-safe AI
- Escalates gently: soft notification first, full-screen Chitti intervention only when needed

Parents configure everything through a PIN-gated section of the same app. No separate parent app required. App-specific controls (restricting individual apps) are the domain of dedicated tools like StayFree — Chitti manages device-wide time windows and habit formation.

---

## User Stories

### Child — Task Management
1. As a child, I want to see my tasks for today on Chitti's home screen, so that I know what I need to do.
2. As a child, I want to see upcoming tasks for the next few days, so that I can plan ahead.
3. As a child, I want Chitti to remind me when a task is due, so that I don't forget.
4. As a child, I want to tap "Done" when I finish a task, so that I can earn my reward points.
5. As a child, I want Chitti to celebrate with me when I complete a task, so that I feel proud of my effort.
6. As a child, I want to see how many points I have earned today, so that I know my progress.
7. As a child, I want to see my badges and level, so that I feel motivated to keep completing tasks.
8. As a child, I want to see my unlocked Chitti costumes and choose which one Chitti wears, so that earning points feels meaningful.

### Child — Screen Time
9. As a child, I want Chitti to warn me 10 minutes before my screen time ends, so that I can wrap up what I'm doing.
10. As a child, I want to see how much screen time I have remaining, so that I can pace myself.
11. As a child, I want Chitti to show me a positive message when my screen time ends, so that restrictions don't feel like punishment.
12. As a child, I want to redeem my earned points for extra screen time, so that I can use my rewards when I want them.

### Child — Q&A with Chitti
13. As a child, I want to start a conversation with Chitti by saying "Hey Chitti" or tapping the character, so that I can ask questions naturally.
14. As a child, I want Chitti to remember the context of our conversation so I can ask follow-up questions, so that conversations feel natural.
15. As a child, I want Chitti to answer in a way I can understand, so that I actually learn from the interaction.
16. As a child, I want Chitti to give me a helpful response even when there's no internet, so that the app always works.

### Parent — Task Configuration
17. As a parent, I want to create tasks with a name, duration, and schedule, so that I can set expectations for my child.
18. As a parent, I want to schedule tasks to repeat daily, on specific weekdays, or on custom days, so that recurring habits are automated.
19. As a parent, I want to set a fixed time for each task, so that Chitti reminds my child at the right moment.
20. As a parent, I want to assign point values to tasks, so that I can weight more important habits higher.
21. As a parent, I want to edit or delete tasks, so that I can adjust as my child's schedule changes.
22. As a parent, I want to toggle tasks active or inactive, so that I can pause tasks without deleting them.

### Parent — Screen Time Rules
23. As a parent, I want to set device-wide time windows during which the device is restricted, so that I control when my child has screen time.
24. As a parent, I want Chitti to enforce these windows automatically, so that I don't have to intervene manually.

### Parent — Rewards Configuration
25. As a parent, I want to optionally enable screen time bonus redemption, so that task completion can translate directly into earned screen time.
26. As a parent, I want to set how many points equal how many extra minutes of screen time, so that I control the reward rate.
27. As a parent, I want to see a weekly summary of my child's completed and missed tasks, so that I can have informed conversations about their habits.
28. As a parent, I want to see my child's total points and badges earned, so that I can celebrate their progress.

### Parent — Account & Setup
29. As a parent, I want to set a PIN to protect the parent configuration screens, so that my child cannot change their own settings.
30. As a parent, I want to be walked through granting the required app permissions during onboarding, so that setup is straightforward.
31. As a parent, I want Chitti to detect if required permissions are missing and guide me to re-enable them, so that the app works reliably after setup.
32. As a parent, I want to create a child profile with a name and age, so that Chitti personalises interactions.

---

## Implementation Decisions

### Module Architecture

**ChittiCharacter**
- Lottie animation player managing named states: `idle`, `talking`, `celebrating`, `warning`
- Adaptive tone engine: rule-based mapping from an explicit `ChittiContext` enum to tone + animation state:
  ```kotlin
  enum class ChittiContext {
      TASK_REMINDER,   // → encouraging tone, attentive animation
      TASK_COMPLETE,   // → energetic tone, celebration animation
      SCREEN_WARNING,  // → calm tone, gentle warning animation
      SCREEN_CUTOFF,   // → calm tone, time's-up animation
      QNA,             // → calm/curious tone, talking animation
      IDLE             // → calm & wise (default), idle animation
  }
  ```
- TTS wrapper around Android `TextToSpeech` with a swappable interface for future cloud TTS. Caches frequently used phrases as audio files.
- Interface: `speak(text: String, context: ChittiContext)` and `animate(state: ChittiState)`

**KidHomeScreen**
- Primary child-facing UI built in Jetpack Compose
- Displays: Chitti character (Lottie), today's pending task checklist, upcoming tasks tab, points balance, badge count, costume selector, screen time remaining indicator
- Task row: name, estimated duration, "Done" button
- Wake word detection ("Hey Chitti", "Hi Chitti") and tap-on-character both trigger QnAEngine session
- Receives task completion events → triggers ChittiCharacter celebration sequence

**TaskEngine**
- Manages task lifecycle: create, read, update, delete, toggle active
- Computes "due today" list from schedule rules and current date (midnight rollover boundary)
- Computes "upcoming" list for next 7 days for child preview
- Registers/cancels AlarmManager alarms when tasks are saved or modified
- On alarm fire: posts tiered notification (immediate), registers 5-minute delayed alarm for full-screen escalation if not acknowledged
- `markComplete(taskId)`: records completion timestamp, triggers RewardSystem point award, cancels pending escalation alarm
- Calendar-like scheduling edge cases:
  - Device off at task time → fire on next boot; skip if > 2 hours past due
  - Two tasks at same time → independent alarms, both fire
  - "Today" boundary → midnight (00:00) rollover
- Exposes: `getPendingToday(): List<Task>`, `getUpcoming(days: Int): List<ScheduledTask>`, `markComplete(taskId)`, `createTask(task)`, `updateTask(task)`, `deleteTask(taskId)`

**RewardSystem**
- Maintains points ledger in Room database (append-only transaction log)
- Badge/level rules: static threshold table (e.g. 50pts = Bronze badge, 200pts = Silver)
- Costume unlocks: static table mapping point milestones to Lottie costume asset names
- Screen time bonus: child manually redeems points from KidHomeScreen via a "Redeem" action. `redeemForScreenTime(points)` calls ScreenTimeRestrictions to credit extra minutes at the parent-configured rate.
- Exposes: `awardPoints(taskId, points)`, `getBalance(): Int`, `checkNewBadges(): List<Badge>`, `getUnlockedCostumes(): List<Costume>`, `redeemForScreenTime(points)`, `getHistory(): List<PointTransaction>`

**ScreenTimeRestrictions**
- Stores parent-configured device-wide time windows in Room (start time, end time, days active)
- AlarmManager schedules: 10-minute warning alarm + hard cutoff alarm per active window
- On 10-min warning: fires high-priority notification with `setFullScreenIntent` → `ChittiWarningActivity` (countdown + positive wrap-up message)
- On hard cutoff: launches `ChittiBlockActivity` full-screen (positive time's-up message, no activity gate in MVP)
- **Known MVP limitation:** child can exit `ChittiBlockActivity` via the Home button — soft deterrent only. Full enforcement requires v2 overlay.
- `creditExtraMinutes(minutes)`: adjusts the hard cutoff alarm time forward by the given amount
- Exposes: `setRestriction(restriction)`, `checkCurrentlyRestricted(): Boolean`, `creditExtraMinutes(minutes)`

**QnAEngine**
- Activation: passive wake word detection ("Hey Chitti", "Hi Chitti") via Android `SpeechRecognizer`; also activated by tap on Chitti character. Wake word listening active only while screen is on.
- Multi-turn: maintains a session context window of the last 6 exchanges, included in each Claude API call
- Session ends after 60 seconds of silence or "bye Chitti" utterance
- Online path: sends conversation history + new utterance to Claude API with kid-safe system prompt. Response text fed to ChittiCharacter TTS.
- Offline/error path: selects from bundled pool of generic fallback responses by category (curiosity, homework, feelings)
- Exposes: `startSession()`, `endSession()`, `ask(utterance: String, onResult: (String) -> Unit)`

**ParentDashboard**
- PIN-gated entry point: `PINGuard` validates PIN before showing any parent screen
- Screens: Task List, Task Editor, Screen Time Rules, Reward Config, Child History + Weekly Summary, Permission Health Check
- `PINGuard`: hashed PIN stored in `EncryptedSharedPreferences`; PIN entry dialog on every parent screen entry

**LocalDatabase (Room)**
- Entities: `ChildProfile`, `Task`, `TaskSchedule`, `TaskCompletion`, `PointTransaction`, `Badge`, `CostumeUnlock`, `ScreenTimeRule`, `RewardConfig`
- `ChildProfile` is the root entity; all other entities carry a `childId` foreign key — multi-child schema from day one, single-child UI for MVP
- Repositories: one per aggregate root (`TaskRepository`, `RewardRepository`, `RestrictionRepository`, `ProfileRepository`)

**NotificationService**
- `ForegroundService` with persistent notification text: "Chitti is running"
- Survives device reboot via `RECEIVE_BOOT_COMPLETED` broadcast receiver
- On boot: re-registers all active AlarmManager alarms from the database
- Manages notification channels: task reminders, screen time warnings
- Escalation logic: on notification post, schedules a secondary alarm 5 minutes later; cancelled on child acknowledgement

### Scheduling Approach
- `AlarmManager.setExactAndAllowWhileIdle()` for all time-critical alarms (task due, screen time cutoff)
- `WorkManager` for non-time-critical periodic work (daily badge check, history cleanup, weekly summary generation)
- All alarm registrations go through a single `AlarmScheduler` utility to prevent drift and duplication

### PIN Storage
- PIN stored as a bcrypt hash in Android `EncryptedSharedPreferences` (Jetpack Security)
- No plain-text PIN ever written to disk
- Default PIN on first install: `0000`
- MVP PIN recovery: app reinstall resets to `0000`. Full recovery (email/security questions) deferred to v2.

### Claude API Integration
- HTTP client: Retrofit with OkHttp
- System prompt enforces: age-appropriate language, no adult content, no personal data collection, response length ≤ 3 sentences
- Conversation history: last 6 exchanges included per API call
- API key stored in `BuildConfig` (injected at build time, not in source control)
- Timeout: 8 seconds → offline fallback path on timeout

### Data Flow: Task Reminder
```
AlarmManager fires
  → NotificationService posts high-priority notification
  → If not acknowledged in 5 min → AlarmManager fires escalation
  → ChittiFullScreenActivity launches (task prompt + "Mark Done" button)
  → TaskEngine.markComplete() called
  → RewardSystem.awardPoints() called
  → ChittiCharacter.speak(celebration, ChittiContext.TASK_COMPLETE)
```

### Data Flow: Screen Time Restriction
```
AlarmManager fires (10-min warning)
  → Notification with setFullScreenIntent → ChittiWarningActivity
  → Chitti shows countdown + encouraging wrap-up message
AlarmManager fires (hard cutoff)
  → ChittiBlockActivity launches full-screen
  → Chitti shows positive time's-up message ("Great session! See you next time.")
  → Child can exit via Home button (known MVP limitation — v2 overlay will enforce)
```

### Data Flow: Screen Time Redemption
```
Child taps "Redeem" on KidHomeScreen
  → RewardSystem.redeemForScreenTime(points)
  → ScreenTimeRestrictions.creditExtraMinutes(calculatedMinutes)
  → Cutoff alarm rescheduled to new time
  → ChittiCharacter celebrates redemption
```

---

## Testing Decisions

**What makes a good test:**
Tests should verify observable external behaviour through the module's public interface, not implementation details. A test should remain valid if the internal implementation is refactored. Avoid testing private methods, Room DAO internals, or Android framework internals — use fakes/in-memory databases instead.

**TaskEngine**
- `getPendingToday()` returns only tasks scheduled for today's day-of-week at or before current time
- `markComplete()` records a completion entry and does NOT re-surface the task as pending for the same day
- `createTask()` with a daily schedule generates the correct AlarmManager registration
- Escalation: 5-minute follow-up alarm is registered when reminder fires; cancelled when `markComplete()` is called
- Late-fire rule: task missed by > 2 hours on boot is skipped, not fired

**RewardSystem**
- `awardPoints()` appends to the ledger and `getBalance()` reflects the new total
- Badge threshold logic: awarding points across a threshold triggers `checkNewBadges()` returning the new badge
- Costume unlock: points reaching a milestone returns the correct costume in `getUnlockedCostumes()`
- `redeemForScreenTime()`: given balance and configured rate, correct minutes calculated and `creditExtraMinutes()` called
- Ledger is append-only: past transactions are never modified

**ScreenTimeRestrictions**
- `checkCurrentlyRestricted()` returns `true` only within a configured window on a matching day
- `creditExtraMinutes()` correctly extends the effective cutoff time
- Restriction spanning midnight (e.g. 10pm–7am) is correctly evaluated on both sides
- Deactivated restrictions are not evaluated

---

## Out of Scope (MVP)

- **App-specific restrictions** — device-wide time windows only; per-app controls handled by external tools (e.g. StayFree)
- **Activity gate (exercise as screen time unlock)** — deferred to v2
- True app overlay (`SYSTEM_ALERT_WINDOW`) — deferred to v2
- Usage-based screen time tracking (`PACKAGE_USAGE_STATS`) — deferred to v2
- Soft floating Chitti nudges over other apps — deferred to v2
- AI proactive nudges before hard cutoffs — deferred to v2
- Hybrid task completion with timer and partial points — deferred to v2
- Time-window scheduling ("anytime between 7–9 AM") — deferred to v2
- Streak tracking — deferred to v2
- Cloud data sync — deferred to v2
- Multi-child UI — deferred to v2 (schema is multi-child ready)
- Cloud TTS for Chitti's voice — deferred to v2
- COPPA / GDPR-K compliance work — deferred with cloud sync
- PIN recovery via email / security questions — deferred to v2
- iOS support — not planned

---

## Further Notes

- **Known MVP limitation — Screen time bypass:** Without `SYSTEM_ALERT_WINDOW`, ChittiBlockActivity can be dismissed via the Android Home button. Screen time enforcement in MVP is a soft deterrent, not a hard block. Parents should be informed of this during onboarding. Full enforcement arrives in v2 with the overlay.
- **Wake word battery consideration:** Continuous STT listening drains battery. Wake word detection should be active only when the screen is on. Fall back to tap-to-talk when the app is in the background.
- **Device recommendation for development:** Use a stock Android device (Pixel or Android One). Manufacturer skin issues (MIUI, OneUI) affect background service survival and are a v2 concern.
- **Lottie assets:** MVP can use a free placeholder Lottie character. Custom Chitti artwork and costume variants can be swapped in without code changes — Lottie file names are the only coupling point.
- **Claude API key:** Injected via `BuildConfig` at build time. Never committed to source control. Add `CLAUDE_API_KEY` to `.gitignore`-protected local properties.
- **Offline fallback pool:** Ship with a minimum of 20 pre-written fallback responses covering: curiosity questions, homework encouragement, feelings/emotions, and generic "I don't know, let's find out together" responses.
- **AlarmManager reliability:** `setExactAndAllowWhileIdle()` is required for Doze mode. All alarms must be re-registered on boot via `RECEIVE_BOOT_COMPLETED`.
