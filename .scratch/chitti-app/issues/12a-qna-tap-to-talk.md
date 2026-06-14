Status: needs-info

# 12a — Q&A with Chitti (tap-to-talk)

## What to build

Multi-turn voice Q&A, using tap-on-character as the always-available entry point
(wake word is slice 12b). Tapping Chitti opens a `QnAEngine` session; Android
`SpeechRecognizer` captures the child's utterance. Online, the conversation
history (last 6 exchanges) plus the new utterance is sent to the Claude API with
a kid-safe system prompt; the response text is fed to Chitti's TTS. The session
ends after 60 seconds of silence or a "bye Chitti" utterance.

Offline / API-error / timeout path: pick from a bundled pool of pre-written
fallback responses (≥ 20, covering curiosity, homework encouragement,
feelings/emotions, and a generic "I don't know, let's find out together"). The
category is chosen by a simple on-device keyword heuristic over the utterance,
with the generic category as the fallback when nothing matches.

Claude API integration: Retrofit + OkHttp; system prompt enforces age-appropriate
language, no adult content, no personal-data collection, and responses ≤ 3
sentences; 8-second timeout → offline fallback. API key injected via
`BuildConfig` at build time (never committed; `CLAUDE_API_KEY` kept out of source
control).

## Acceptance criteria

- [ ] Tapping the Chitti character starts a Q&A session
- [ ] Utterance captured via `SpeechRecognizer`
- [ ] Online: last 6 exchanges + utterance sent to Claude with the kid-safe
      prompt; response spoken via TTS; responses ≤ 3 sentences
- [ ] 8s timeout / offline / error → keyword-categorised fallback response, with
      generic fallback when no category matches
- [ ] Bundled pool has ≥ 20 responses across the four categories
- [ ] Session ends after 60s silence or "bye Chitti"
- [ ] API key sourced from `BuildConfig`; not present in source control

## Blocked by

- 08 — Chitti character (TTS, talking animation)

## Open questions

- **Claude API (blocking):** the exact model id, how the dev/test API key is
  provisioned, and the exact kid-safe system-prompt text are not specified.
  Needed before the online path can be implemented and tested. (The offline
  fallback path can proceed independently.)
