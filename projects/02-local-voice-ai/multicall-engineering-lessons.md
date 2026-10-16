# MultiCall Engineering Lessons

These are project-owner-reported fixes and checks from the Legacy-to-MultiCall
stage. Automated checks, saved-audio comparisons and live-call observations are
distinguished below. They were not rerun for this documentation update. Service
identities and company-specific response text are anonymized.

## Working-Directory Permissions After Adding the Background Worker

- **Symptom:** incoming calls stopped receiving a response after post-processing
  was integrated.
- **Diagnosis:** worker processes ran under a dedicated service account, but the
  post-processing directory belonged to root with mode `0700`. The runtime account
  could not access that working directory.
- **Fix:** queue, state and recording directories were assigned to the runtime
  account; the post-processing service was moved to the same account. Access for
  other users was not broadened.
- **Result:** checks as that account covered WAV creation/finalization, queue-job
  publication, file-lock acquisition and dependency availability. After integration
  was restored, incoming calls received responses again.
- **Remaining limits:** these checks establish the repaired access path, not a
  complete filesystem-security or crash-recovery audit.

## False Interruptions and Speakerphone Echo

- **Symptom:** false barge-in and echo were observed during speakerphone use.
- **Diagnosis:** playback echo could be mistaken for caller speech. Echo references
  needed continuity across sequential playback fragments, and sequence timing
  needed correct alignment when quiet frames were present.
- **Fix:** retained echo references across fragments, corrected timing alignment
  and strengthened speech confirmation. Two short initial greetings were protected
  from interruption. Barge-in requires a level at least 20 dB above the preceding
  background. The Legacy prefilter was also restored: a weak candidate is not sent
  to STT when **all three** conditions hold: background difference below 20 dB,
  mean level below −60 dBFS and peak below −40 dBFS.
- **Result:** checks blocked matching synthetic echo, admitted an independent
  signal, blocked interruption without an echo reference and rejected stale entries
  after cancellation. The acoustic filter blocked one saved false trigger; normal
  and quiet speech above background were checked for admission. The user reported
  improved speakerphone behavior, but a subsequent test still stopped the greeting
  in **6 of 10 calls**. Diagnostics found no application-internal audio-feed gaps
  in 67 completed audio fragments.
- **Remaining limits:** false interruption is not eliminated. The 67-fragment
  observation does not exclude loss farther along the audio path. These thresholds
  describe this project configuration, not universal acoustic tuning guidance.

## Russian/Kazakh Switching Errors

- **Symptom:** Kazakh text was sometimes recognized correctly while the application
  selected Russian mode and a Russian voice.
- **Diagnosis:** a correct transcript did not guarantee correct conversation-language
  selection; decoder language and application state needed separate handling.
- **Fix:** added language evidence independent of the forced decoder language,
  Kazakh greeting handling and more explicit switch-command variants. A single
  noisy word no longer locks the language. Language-change requests are handled
  separately from routing and receive a fixed reply in the requested voice. On a
  language change, LLM history that could retain the old language is cleared while
  routing state is preserved.
- **Result:** automated checks covered Kazakh greetings, RU→KZ and KZ→RU switches,
  Russian greetings and commands in both languages. Live tests confirmed successful
  switches followed by continued dialog.
- **Remaining limits:** some distorted short greetings were still misrecognized.
  Error-free automatic language detection is not claimed; dual RU/KK recognition
  also adds CPU work.

## Kazakh Phrases Truncated by Whisper's Output Limit

- **Symptom:** a Kazakh utterance about purchased faulty equipment was truncated
  before the fault description, causing selection of a commercial route.
- **Diagnosis:** the prior duration-dependent Whisper output limit, capped at
  96 tokens, cut off semantically important information.
- **Fix:** created a separate MultiCall Whisper profile with a 192-token output
  limit in its own service. The earlier STT service and Legacy code were preserved.
  An additional routing safeguard asks for clarification when recognition contains
  only a purchase history without a clear reason for calling.
- **Result:** on a saved fragment, the old limit stopped within the purchase phrase;
  the larger limit recognized the subsequent fault description. Processing took
  2.59 seconds versus 2.24 seconds in that comparison. Separate-service startup and
  model readiness were checked; the user installed the profile and obtained
  confirmation that the services were active.
- **Remaining limits:** this is one saved-fragment comparison, not a performance
  benchmark or a guarantee against all STT truncation and routing errors. Service
  activity alone does not establish complete end-to-end correctness.

## Overly Generic Replies After LLM Guard Changes

- **Symptom:** after hallucination-protection changes, ordinary questions such as
  “Where have I called?” and “Which company is this?” received a generic clarification.
- **Diagnosis:** protective behavior was too broad for identification questions,
  which have an approved factual answer rather than requiring open-ended generation.
- **Fix:** added fixed company-identification replies in Russian and Kazakh.
  Unclear requests receive a safe clarification instead of generated weather
  comments or invented employee restrictions. Requests to describe the company
  remain a separate scenario. The actual company name is not published here.
- **Result:** checks covered Kazakh formulations from the tests, absence of
  accidental transfer on an unclear request, and preservation of the
  “Tell me about the company” scenario. The latest identification-answer extensions
  passed text checks and module imports.
- **Remaining limits:** the available material contains no confirmed repeat call
  after those latest extensions. Text checks and successful imports are not
  presented as live-call validation.

## Cross-Layer Lesson

Independent rooms solve a state-ownership problem; they do not automatically solve
recognition cost, echo, transcript completeness or response quality. Follow each
symptom to its responsible layer and record its own fix and validation evidence.

[Project overview](README.md) · [Architecture, validation and roadmap](legacy-to-multicall.md)
