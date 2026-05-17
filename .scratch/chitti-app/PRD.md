Status: ready-for-agent

# PRD: Chitti — Kid Companion Android App (MVP)

## Problem Statement

Parents struggle to enforce healthy digital habits for their children aged 8–12 in a way that feels positive rather than punitive. Existing parental control apps block and restrict, but do not engage — children feel policed, not guided. There is no companion that reframes screen time limits as opportunities, encourages physical activity and task completion, and answers children's questions in a safe, age-appropriate way.

---

## Solution

Chitti is an Android companion app featuring a cartoon character that lives on the child's device. Chitti:
- Reminds children about parent-configured tasks (study, music practice, etc.) with warmth and encouragement
- Reframes screen time restrictions positively ("We've got 10 mins left — let's make it count!")
- Gates screen time resumption behind short physical activities (eye exercises, jumping jacks)
- Awards points for task completion, unlocking badges and optional extra screen time
- Answers children's questions using an AI model with a kid-safe system prompt
- Escalates gently: soft notification first, full-screen Chitti intervention only when needed

Parents configure everything through a PIN-gated section of the same app. No separate parent app required.

---

## User Stories

### Child — Task Management
1. As a child, I want to see my tasks for today on Chitti's home screen, so that I know what I need to do.
2. As a child, I want Chitti to remind me when a task is due, so that I don't forget.
3. As a child, I want to tap "Done" when I finish a task, so that I can earn my reward points.
4. As a child, I want Chitti to celebrate with me when I complete a task, so that I feel proud of my effort.
5. As a child, I want to see how many points I have earned today, so that I know my progress.
6. As a child, I want to see my badges and level, so that I feel motivated to keep completing tasks.

### Child — Screen Time
7. As a child, I want Chitti to warn me 10 minutes before my screen time ends, so that I can wrap up what I'm doing.
8. As a child, I want to see a countdown of time remaining during my screen session, so that I can pace myself.
9. As a child, I want Chitti to guide me through a short exercise when my time is up, so that I can earn my next session.
10. As a child, I want Chitti to speak to me in a friendly, encouraging voice, so that restrictions don't feel like punishment.
11. As a child, I want to earn extra screen time through task completion, so that good habits are rewarded.

### Child — Q&A with Chitti
12. As a child, I want to ask Chitti questions by speaking aloud, so that I can get answers without typing.
13. As a child, I want Chitti to answer in a way I can understand, so that I actually learn from the interaction.
14. As a child, I want Chitti to give me a helpful response even when there's no internet, so that the app always works.

### Child — Activities & Exercises
15. As a child, I want Chitti to guide me through eye exercises step by step, so that I know what to do.
16. As a child, I want Chitti to count my jumping jacks with me, so that exercising feels fun.
17. As a child, I want to see a completion animation when I finish an exercise, so that I feel accomplished.

### Parent — Task Configuration
18. As a parent, I want to create tasks with a name, duration, and schedule, so that I can set expectations for my child.
19. As a parent, I want to schedule tasks to repeat daily, on specific weekdays, or on custom days, so that recurring habits are automated.
20. As a parent, I want to set a fixed time for each task, so that Chitti reminds my child at the right moment.
21. As a parent, I want to assign point values to tasks, so that I can weight more important habits higher.
22. As a parent, I want to edit or delete tasks, so that I can adjust as my child's schedule changes.
23. As a parent, I want to toggle tasks active or inactive, so that I can pause tasks without deleting them.

### Parent — Screen Time Rules
24. As a parent, I want to set time windows during which certain apps or all apps are restricted, so that I control when my child uses the device.
25. As a parent, I want Chitti to enforce these windows automatically, so that I don't have to intervene manually.
26. As a parent, I want to configure which physical activity Chitti uses as the activity gate, so that I can match my child's ability.

### Parent — Rewards Configuration
27. As a parent, I want to optionally enable screen time bonus points, so that task completion translates directly into earned screen time.
28. As a parent, I want to set how many points equal how many extra minutes of screen time, so that I control the reward rate.
29. As a parent, I want to see my child's task completion history, so that I can have informed conversations about their habits.
30. As a parent, I want to see my child's total points and badges earned, so that I can celebrate their progress.

### Parent — Account & Setup
31. As a parent, I want to set a PIN to protect the parent configuration screens, so that my child cannot change their own settings.
32. As a parent, I want to be walked through granting the required permissions during onboarding, so that setup is straightforward.
33. As a parent, I want to create a child profile with a name and age, so that Chitti personalises interactions.

---

## Implementation Decisions

### Module Architecture

**ChittiCharacter**
- Lottie animation player managing named states: `idle`, `talking`, `celebrating`, `exercising`, `warning`
- Adaptive tone engine: rule-based context mapping (rest/wind-down → calm, activity/celebration → energetic, task reminder → encouraging). Default tone: calm & wise.
- TTS wrapper around Android `TextToSpeech` with a swappable interface for future cloud TTS. Caches frequently used phrases as audio files.
- Single interface: `speak(text: String, context: ChittiContext)` and `animate(state: ChittiState)`

**KidHomeScreen**
- Primary child-facing UI built in Jetpack Compose
- Displays: Chitti character (Lottie), today's pending task checklist, points balance, badge count
- Task row: name, estimated duration, "Start" / "Done" button
- "Ask Chitti" voice button triggers STT → QnAEngine pipeline
- Receives task completion events and triggers ChittiCharacter celebration sequence

**TaskEngine**
- Manages task lifecycle: create, read, update, delete, toggle active
- Computes "due today" list from schedule rules and current date
- Registers/cancels AlarmManager alarms when tasks are saved or modified
- On alarm fire: posts tiered notification (immediate), registers a 5-minute delayed alarm for full-screen escalation if not acknowledged
- `markComplete(taskId)`: records completion timestamp, triggers RewardSystem point award, cancels any pending escalation alarm
- Exposes: `getPendingToday(): List<Task>`, `markComplete(taskId)`, `createTask(task)`, `updateTask(task)`, `deleteTask(taskId)`

**RewardSystem**
- Maintains points ledger in Room database (append-only transaction log)
- Badge/level rules: defined as a static table of thresholds (e.g. 50pts = Bronze badge, 200pts = Silver)
- Screen time bonus: if parent toggle enabled, converts points at configured rate into extra screen time minutes. Communicates with ScreenTimeRestrictions to credit extra time.
- Exposes: `awardPoints(taskId, points)`, `getBalance(): Int`, `checkNewBadges(): List<Badge>`, `getHistory(): List<PointTransaction>`

**ScreenTimeRestrictions**
- Stores parent-configured time windows in Room (start time, end time, days active, activity gate type)
- AlarmManager schedules: 10-minute warning alarm + hard cutoff alarm for each active window
- On 10-min warning: fires high-priority notification with `setFullScreenIntent` pointing to `ChittiWarningActivity`
- On hard cutoff: launches `ChittiBlockActivity` (full-screen, non-dismissible without completing ActivityGate)
- Screen time bonus integration: adjusts cutoff alarm time when RewardSystem credits extra minutes
- Exposes: `setRestriction(restriction)`, `checkCurrentlyRestricted(): Boolean`, `creditExtraMinutes(minutes)`

**ActivityGate**
- Full-screen Compose UI presented inside `ChittiBlockActivity`
- Gate types: `EYE_EXERCISES` (5-step guided sequence with Chitti animation), `JUMPING_JACKS` (count-based with Chitti counting aloud)
- Chitti guides each step with voice + animation. On completion: fires callback to ScreenTimeRestrictions to lift block, triggers celebration sequence.
- Exposes: `startGate(type: GateType, onComplete: () -> Unit)`

**QnAEngine**
- STT: Android `SpeechRecognizer` captures child's voice input
- Online path: sends transcribed text to Claude API with a hardcoded kid-safe system prompt. Response text fed to ChittiCharacter TTS.
- Offline/error path: selects from a bundled pool of generic fallback responses by topic category (curiosity, homework, feelings)
- Exposes: `ask(onResult: (String) -> Unit)` — handles full STT → API → response pipeline internally

**ParentDashboard**
- PIN-gated entry point: `PINGuard` module validates PIN before showing any parent screen
- Screens: Task List, Task Editor, Screen Time Rules, Reward Config, Child History
- `PINGuard`: stores hashed PIN in encrypted SharedPreferences; shows PIN entry dialog on any parent screen entry

**LocalDatabase (Room)**
- Entities: `ChildProfile`, `Task`, `TaskSchedule`, `TaskCompletion`, `PointTransaction`, `Badge`, `ScreenTimeRule`, `RewardConfig`
- `ChildProfile` is the root entity; all other entities have a `childId` foreign key — multi-child schema from day one, single-child UI for MVP
- Repositories: one per aggregate root (`TaskRepository`, `RewardRepository`, `RestrictionRepository`, `ProfileRepository`)

**NotificationService**
- `ForegroundService` that survives device reboot (registered via `RECEIVE_BOOT_COMPLETED`)
- On boot: re-registers all active AlarmManager alarms from the database
- Manages notification channels: task reminders, screen time warnings
- Escalation logic: on notification post, schedules a secondary alarm 5 minutes later; cancelled if child acknowledges the notification

### Scheduling Approach
- `AlarmManager.setExactAndAllowWhileIdle()` for time-critical alarms (task due, screen time cutoff)
- `WorkManager` for non-time-critical periodic work (e.g. daily badge check, history cleanup)
- All alarm registrations go through a single `AlarmScheduler` utility to avoid drift and duplication

### PIN Storage
- PIN stored as a bcrypt hash in Android `EncryptedSharedPreferences` (Jetpack Security)
- No plain-text PIN ever written to disk

### Claude API Integration
- HTTP client: Retrofit with OkHttp
- System prompt enforces: age-appropriate language, no adult content, no personal data collection, response length ≤ 3 sentences
- API key stored in `BuildConfig` (injected at build time, not in source control)
- Timeout: 8 seconds. On timeout → offline fallback path

### Data Flow: Task Reminder
```
AlarmManager fires
  → NotificationService posts high-priority notification
  → If not acknowledged in 5 min → AlarmManager fires escalation
  → ChittiBlockActivity launches full-screen
  → Child sees task prompt + "Mark Done" button
  → TaskEngine.markComplete() called
  → RewardSystem.awardPoints() called
  → ChittiCharacter.celebrate()
```

### Data Flow: Screen Time Restriction
```
AlarmManager fires (10-min warning)
  → Notification with setFullScreenIntent → ChittiWarningActivity
AlarmManager fires (hard cutoff)
  → ChittiBlockActivity (non-dismissible)
  → ActivityGate presented
  → On gate complete → block lifted
  → RewardSystem checks for screen time bonus → creditExtraMinutes()
```

---

## Testing Decisions

**What makes a good test:**
Tests should verify observable external behaviour through the module's public interface, not implementation details. A test should remain valid if the internal implementation is refactored. Avoid testing private methods, Room DAO internals, or Android framework internals directly — use fakes/in-memory databases instead.

**TaskEngine**
- Test that `getPendingToday()` returns only tasks scheduled for today's day-of-week at or before current time
- Test that `markComplete()` records a completion entry and does NOT re-surface the task as pending for the same day
- Test that `createTask()` with a daily schedule generates the correct AlarmManager registration
- Test escalation: verify a 5-minute follow-up alarm is registered when a reminder notification fires, and is cancelled when `markComplete()` is called

**RewardSystem**
- Test that `awardPoints()` correctly appends to the ledger and `getBalance()` reflects the new total
- Test badge threshold logic: awarding points across a threshold triggers `checkNewBadges()` to return the new badge
- Test screen time bonus calculation: given a point balance and a configured rate, verify `creditExtraMinutes()` is called with the correct value
- Test that the ledger is append-only: past transactions are never modified

**ScreenTimeRestrictions**
- Test that `checkCurrentlyRestricted()` returns `true` only within a configured time window on a matching day
- Test that `creditExtraMinutes()` correctly extends the effective cutoff time
- Test that a restriction spanning midnight (e.g. 10pm–7am) is correctly evaluated on both sides of midnight
- Test that deactivated restrictions are not evaluated

---

## Out of Scope (MVP)

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
- iOS support — not planned

---

## Further Notes

- **Device recommendation for development:** Use a stock Android device (Pixel or Android One) during MVP development. Manufacturer skin issues (MIUI, OneUI) affect background service survival and are a v2 concern.
- **Lottie assets:** MVP can use a free placeholder Lottie character. Custom Chitti artwork can be swapped in without code changes — Lottie file names are the only coupling point.
- **Claude API key:** Must be injected via `BuildConfig` at build time. Never committed to source control. Add `CLAUDE_API_KEY` to `.gitignore`-protected local properties.
- **Offline fallback pool:** Ship with a minimum of 20 pre-written fallback responses covering: curiosity questions, homework encouragement, feelings/emotions, and generic "I don't know, let's find out together" responses.
- **AlarmManager reliability:** `setExactAndAllowWhileIdle()` is required for Doze mode. All alarms must be re-registered on boot via `RECEIVE_BOOT_COMPLETED` broadcast receiver.
