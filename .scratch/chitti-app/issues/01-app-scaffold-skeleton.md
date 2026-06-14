Status: ready-for-human

# 01 — Walking skeleton: Compose app + Hilt + Room + foreground service

## What to build

The greenfield foundation every other Chitti slice builds on. A Kotlin +
Jetpack Compose Android app that launches to a placeholder `KidHomeScreen`, with
Hilt dependency injection wired up, a Room database initialised with the root
`ChildProfile` entity and a `ProfileRepository`, a `NotificationService`
(`ForegroundService`) that shows a persistent "Chitti is running" notification,
and an `AlarmScheduler` utility skeleton that all later alarm registrations route
through.

The Room schema is multi-child ready from day one (`ChildProfile` is the root;
later entities carry a `childId` foreign key) but the UI is single-child for MVP.

This is a HITL slice: it requires the project to be created, the Gradle/SDK
toolchain chosen, and a first run on a device confirmed by the developer.

## Acceptance criteria

- [ ] App builds and launches to a placeholder `KidHomeScreen` composable
- [ ] Hilt is configured (`@HiltAndroidApp` application, an injectable repository)
- [ ] Room DB created with `ChildProfile` entity + `ProfileRepository`; schema is
      designed multi-child (foreign-key ready) per PRD `LocalDatabase`
- [ ] `NotificationService` runs as a `ForegroundService` with a persistent
      "Chitti is running" notification
- [ ] `AlarmScheduler` utility exists as the single entry point for alarm
      registration (skeleton; concrete alarms added in later slices)
- [ ] Key dependencies declared: `lottie-compose`, Room, Hilt, Retrofit+OkHttp,
      Jetpack Security

## Blocked by

None - can start immediately.

## Open questions

- **Build/test environment (blocking):** Confirm how slices are verified —
  developer-on-device (per the design doc's "agent writes code, developer
  tests") or an automated build in CI. The remote agent environment may not have
  the Android SDK/Gradle to compile, which affects every downstream slice's
  verification story.
- **min/target SDK:** Not specified in the PRD. Required before manifest work —
  the `FOREGROUND_SERVICE_*` subtype permission (slice 03) depends on the target
  SDK level. Please pin both.
