# Chitti — App Design Document

## Concept
Chitti is a kid companion Android app (ages 8–12) that manages digital wellness, encourages healthy habits, and makes parental restrictions feel positive rather than punitive. A cartoon character named Chitti guides children through screen breaks, physical activities, and tasks — while answering their questions like a friendly companion.

---

## MVP Scope

### Platform
- Android only
- Native Kotlin + Jetpack Compose
- Agent-driven development (Claude writes code, developer tests and reports back)

### User Model
- Single app, two modes:
  - **Parent mode:** PIN-gated setup screens
  - **Child mode:** Everything else, voice+visual primary interface

---

## Core Features (MVP)

### 1. Chitti Character
- Lottie animations (`lottie-compose`)
- Adaptive personality, rule-based context switching:
  - Activity/celebration context → energetic tone
  - Rest/wind-down context → calm & wise (default)
  - Task reminder context → encouraging tone
- Voice: Android built-in TTS (architecture swappable to cloud TTS in v2)
- Child input: Android built-in STT

### 2. Task System
- Parent creates tasks with:
  - Name, duration, icon
  - Schedule: daily / weekly / custom days + fixed time
  - Reward points value
- Task reminder flow (tiered):
  1. Push notification at scheduled time
  2. If not acknowledged in 5 minutes → full-screen Chitti escalation
- Completion: Honor system (child taps "Done")
- Chitti celebrates with animation + voice on completion
- Points awarded immediately on completion

### 3. Reward System
- **In-app (always on):** Points → badges, Chitti costume unlocks, level-up animations
- **Screen time bonus (optional parent toggle):** Points convert to extra screen time
- Parents handle real-world rewards separately

### 4. Screen Time Restrictions
- Clock-based only (parent sets time windows, e.g. "no games after 8pm")
- Restriction flow (tiered):
  1. Push notification 10 minutes before limit (`setFullScreenIntent`) → full-screen Chitti soft warning
  2. At hard limit → full-screen `ChittiBlockActivity` launches
- Activity gate on full-screen block: child must complete activity (eye exercises / jumping jacks) before timer unlocks
- Chitti reframes restriction positively ("We've got 10 mins left — let's make it count!")

### 5. Chitti Q&A
- Child speaks to Chitti via STT → routed to Claude API with kid-safe system prompt
- Offline / API failure → graceful generic fallback responses (pre-scripted, on-device)

### 6. Parent Dashboard (PIN-gated)
- Task management (create, edit, delete, toggle active)
- Screen time schedule configuration
- Reward point configuration (screen time bonus toggle, point values per task)
- Child's points, badges, task history (done / not done)
- Data model designed for multi-child from day one (single-child UI for MVP)

### 7. Permissions (MVP — clean profile)
| Permission | Purpose |
|---|---|
| `POST_NOTIFICATIONS` | Task reminders, time warnings |
| `FOREGROUND_SERVICE` | Background timer/reminder service |
| `RECEIVE_BOOT_COMPLETED` | Restart services on device reboot |
| `RECORD_AUDIO` | Child voice input (STT) |

No manual settings navigation required for MVP onboarding.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | Jetpack Compose |
| Character animation | Lottie (`lottie-compose`) |
| Local database | Room (multi-child schema from day one) |
| Scheduling | AlarmManager + WorkManager |
| Background services | ForegroundService |
| TTS | Android built-in (`TextToSpeech`) |
| STT | Android built-in (`SpeechRecognizer`) |
| LLM Q&A | Claude API (Retrofit HTTP client) |
| DI | Hilt (agent-written, developer tests) |

---

## V2 Roadmap

| Feature | Complexity |
|---|---|
| True overlay (`SYSTEM_ALERT_WINDOW`) | High |
| Usage-based tracking (`PACKAGE_USAGE_STATS`) | High |
| Soft floating Chitti nudges | High |
| AI proactive nudges (Layer C) | Medium |
| Hybrid task completion (timer + partial points) | Medium |
| Time-window scheduling + streak tracking | Medium |
| Cloud TTS for Chitti's voice | Low |
| Cloud sync + cross-device parent dashboard | High |
| Multi-child UI | Medium |
| COPPA/GDPR-K compliance for cloud | High |

---

## Key Design Decisions & Rationale

| Decision | Choice | Reason |
|---|---|---|
| Platform | Android only | Overlay APIs, target demographic |
| Tech stack | Native Kotlin | Single codebase, full API access, best for agent-driven dev |
| Overlay for MVP | Full-screen activity + notification intent | Avoids SYSTEM_ALERT_WINDOW complexity, cleaner onboarding |
| Usage tracking for MVP | Clock-based only | Avoids PACKAGE_USAGE_STATS, ship faster |
| Task verification | Honor system | MVP simplicity, extensible to timer+partial points |
| Data storage | Local-only (Room) | COPPA-clean, no cloud compliance burden |
| LLM | Claude API | Q&A quality, kid-safe system prompt |
| Voice | On-device TTS+STT | Free, offline-capable, swappable |
| Rewards | Badges + optional screen time bonus | Works out of box, powerful closed loop when enabled |
| Character animation | Lottie | Industry standard, swappable artwork |
