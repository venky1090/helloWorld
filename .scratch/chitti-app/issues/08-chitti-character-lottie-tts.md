Status: needs-info

# 08 — ChittiCharacter: Lottie states + TTS + celebration

## What to build

The Chitti character module: a Lottie animation player managing named states
(`idle`, `talking`, `celebrating`, `warning`), a rule-based adaptive-tone engine
mapping the `ChittiContext` enum to a tone + animation state, and a TTS wrapper
around Android `TextToSpeech` behind a swappable interface (for future cloud TTS)
that caches frequently used phrases as audio files. On task completion, Chitti
plays the celebration sequence with voice.

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

Interface: `speak(text: String, context: ChittiContext)` and
`animate(state: ChittiState)`.

## Acceptance criteria

- [ ] Lottie player renders and switches between idle/talking/celebrating/warning
- [ ] `ChittiContext` maps to the correct tone + animation state per the table
- [ ] TTS wrapper speaks text behind a swappable interface; frequently used
      phrases are cached as audio files
- [ ] Task completion triggers the celebration animation + voice
      (`ChittiContext.TASK_COMPLETE`)

## Blocked by

- 01 — app scaffold (`lottie-compose`, TTS init)
- 05 — task completion event source

## Open questions

- **Lottie assets (non-blocking):** which placeholder Lottie character file and
  how its named states map to the four states. PRD permits a free placeholder
  with file names as the only coupling point — implement against a chosen
  placeholder and document the mapping. (Custom artwork swaps in later without
  code changes.)
- **Phrase strings (non-blocking):** exact reminder/celebration/warning/cutoff
  copy is unspecified. Author age-appropriate defaults; confirm wording with the
  product owner.

> Status is `needs-info` only pending confirmation of the placeholder asset to
> build against. If the team is happy for the implementing agent to pick a free
> placeholder and document it, this can be flipped to `ready-for-agent`.
