# MultiCall Engineering Lessons

These are project-owner-reported diagnostic issues from the Legacy-to-MultiCall
stage. The supplied summary identifies the problems but does not specify every
patch or per-issue retest. Missing details are marked explicitly; corrective
directions below are not claims of unreported implementation or test success.

## Working-Directory Permissions After Adding the Background Worker

- **Symptom:** working-directory permission problems appeared after the background
  processor was connected.
- **Diagnosis:** access to working directories was identified as the integration
  boundary. Exact failing operations and process identities are not published.
- **Fix:** the required correction concerns directory access for the participating
  processes; exact ownership/permission changes are not supplied.
- **Result:** CRM post-processing is reported as checked at the stage level;
  a separate permission-fix retest is not documented.
- **Remaining limits:** no blanket permission commands or claim of validated
  filesystem security is derived from this report.

## False Interruptions and Speakerphone Echo

- **Symptom:** false barge-in and echo were observed during speakerphone use.
- **Diagnosis:** VAD activity was not sufficient evidence of caller speech;
  playback/echo could cross the interruption boundary. The
  [Legacy confirmation-window lesson](speech-and-language.md#reliable-barge-in)
  remains relevant.
- **Fix:** MultiCall adds protection against stale responses after interruption.
  That output-lifecycle protection is distinct from echo suppression; exact new
  VAD/echo adjustments are not supplied.
- **Result:** the call architecture progressed to two validated concurrent calls;
  elimination of false interruptions is not a confirmed result.
- **Remaining limits:** noise, echo and speakerphone robustness remain roadmap work.

## Russian/Kazakh Switching Errors

- **Symptom:** conversation language switched incorrectly between Russian and Kazakh.
- **Diagnosis:** language recognition and response-language behavior must be
  evaluated against conversation state, not treated as one signal. No single
  code-level cause is specified for these later errors.
- **Fix:** MultiCall isolates language state per call and retains the earlier
  session-language handling. Specific switching-rule corrections are not supplied.
- **Result:** RU/KZ scenarios are reported as checked; error-free switching and
  full Legacy equivalence are not established.
- **Remaining limits:** dual RU/KK recognition adds CPU work; broader language and
  mixed-input acceptance tests are still needed.

## Kazakh Phrases Truncated by Whisper's Output Limit

- **Symptom:** Kazakh phrases were cut off in recognition output.
- **Diagnosis:** the project identified a Whisper output limitation as the cause;
  a truncated transcript must not automatically be blamed on SIP/RTP loss.
- **Fix:** the corrective boundary is the recognition output limit. The changed
  parameter, value and exact patch are not supplied and are not reconstructed.
- **Result:** no separate before/after transcript or completion rate is provided.
- **Remaining limits:** longer Kazakh utterances need explicit completeness checks;
  no claim of universal truncation removal is made.

## Overly Generic Replies After LLM Guard Changes

- **Symptom:** protective LLM changes were followed by excessively general replies.
- **Diagnosis:** the regression appeared after guard changes. The report does not
  isolate a specific prompt, decoding setting or guard rule as the sole cause.
- **Fix:** guard behavior needs to retain useful grounded answers while preserving
  deterministic action ownership. Exact subsequent changes are not supplied.
- **Result:** main dialogs are reported as checked at the stage level, without a
  separate response-specificity comparison for this issue.
- **Remaining limits:** specificity, factual grounding and action safety need joint
  acceptance criteria; removing guards is not established as the solution.

## Cross-Layer Lesson

Independent rooms solve a state-ownership problem; they do not automatically solve
recognition cost, echo, transcript completeness or response quality. Follow each
symptom to its responsible layer and record its own fix and validation evidence.

[Project overview](README.md) · [Architecture, validation and roadmap](legacy-to-multicall.md)
