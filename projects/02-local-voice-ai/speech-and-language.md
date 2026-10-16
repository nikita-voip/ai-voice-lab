# RU/KZ Speech and Conversation State

> This note preserves the Legacy speech-handling work. MultiCall retains RU/KZ
> scenarios with per-call language state; see the
> [transition](legacy-to-multicall.md) and
> [later diagnostic lessons](multicall-engineering-lessons.md).

## Session Language Instead of Per-Utterance Guessing

Independent language detection on every utterance is unreliable for short or
ambiguous Russian/Kazakh input. A recognition result should not silently change
the language of the entire conversation.

The project uses session language state with forced RU/KZ modes and explicit
language switching. Recognition confidence, `avg_logprob`, minimum utterance
length and Kazakh-script evidence inform how an utterance is interpreted.
Script evidence is one signal, not a replacement for conversation context.

STT language and LLM response language are separate concerns. A response-language
guard keeps the generated reply aligned with the actual conversation state rather
than assuming that the recognizer's latest language label controls the LLM.
No numeric thresholds or language-accuracy results are published here.

## Whisper Hallucination Protection

**Expected:** silence and service sounds must not become actionable user requests.

**Observed:** Whisper could produce text from silence, noise, short utterances and
the ECSS recording beep. Plausible text alone was insufficient evidence of a
caller request.

The protection combines VAD, duration checks, confidence, a blacklist and session
context. The application must carry STT uncertainty into downstream business
logic; low-trust recognition must not become a confident employee selection or
telecom action merely because the LLM can produce a fluent interpretation.

This is layered filtering, not a claim that hallucinations have been eliminated.
The public summary does not include the private recordings, blacklist entries or
a measured false-positive rate.

## Reliable Barge-In

A VAD event is not necessarily caller speech. Own TTS, echo and service sounds
can trigger activity detection. Immediately stopping audio on every event creates
false interruptions.

The project introduces a confirmation window before interrupting playback.
The relevant decision is whether there is credible caller speech, not simply
whether VAD fired. This replaces naive `if speech: stop_audio()` behavior.
The window duration and detailed confirmation algorithm are not specified in the
provided project summary and are not invented here.

## TTS Text Normalization

Names and brands need a normalization/transliteration layer before synthesis.
The project adds this layer with RU/KZ pronunciation considerations instead of
passing every generated string directly to Piper. Text generation and spoken
pronunciation are distinct boundaries; no private employee names, brand-specific
rules or unsupported claim of complete pronunciation coverage are published.

## Acceptance Checks

These are checks for the documented behavior, not recorded test results.

| Input / condition | Expected behavior |
|---|---|
| Short or ambiguous utterance | Preserve session language; use uncertainty instead of an automatic language switch |
| Forced RU or KZ mode | Keep STT interpretation and generated replies consistent with the selected mode |
| Explicit language switch | Update conversation state and align subsequent replies |
| Kazakh-script evidence | Consider it together with confidence, duration and session context |
| Silence, noise or recording beep | Do not promote hallucinated text into a routing action |
| Own TTS, echo or service sound | Do not interrupt solely because VAD reports activity |
| Credible caller speech during TTS | Confirm barge-in before interrupting playback |
| Name or brand in spoken output | Apply the normalization layer with the conversation language in mind |

## Lesson

Recognition text, recognition confidence, conversation language and output
pronunciation are separate pieces of state. Preserving those distinctions is
necessary before speech can safely drive deterministic business actions.

[Project overview](README.md) · [Routing and SIP handoff](routing-and-sip-handoff.md)
