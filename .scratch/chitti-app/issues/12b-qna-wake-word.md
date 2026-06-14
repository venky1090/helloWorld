Status: needs-info

# 12b — Q&A wake word ("Hey Chitti")

## What to build

Passive wake-word activation layered on top of the tap-to-talk Q&A (12a). A
dedicated on-device wake-word engine detects "Hey Chitti" / "Hi Chitti" and opens
a `QnAEngine` session; Android `SpeechRecognizer` then captures the utterance (it
cannot reliably spot always-on wake words itself). Wake-word listening is active
only while the screen is on, falling back to tap-to-talk when the app is
backgrounded — to limit battery drain.

## Acceptance criteria

- [ ] Saying "Hey Chitti" / "Hi Chitti" with the screen on opens a Q&A session
- [ ] After the wake word, the utterance is captured via `SpeechRecognizer` and
      flows through the same Q&A path as 12a
- [ ] Wake-word listening is disabled when the app is backgrounded; tap-to-talk
      remains available
- [ ] Battery impact is bounded (listening only while screen on)

## Blocked by

- 12a — Q&A session pipeline

## Open questions

- **Wake-word engine (blocking):** the PRD names Porcupine / Picovoice only as
  examples ("e.g."). Required decisions before this can be built:
  - Which engine, and its licensing / access-key model (Picovoice's free tier has
    usage limits and the access key must be provisioned and kept out of source
    control like the Claude key).
  - Custom "Hey Chitti" / "Hi Chitti" keyword files must be trained/obtained for
    the chosen engine.
  This is why the status is `needs-info`. Consider whether wake word is required
  for the MVP launch or can ship after, since tap-to-talk (12a) already provides
  the full Q&A experience.
