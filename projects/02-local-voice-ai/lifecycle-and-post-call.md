# Call Lifecycle, CPU Priority and Post-Call Processing

## Caller Hangup During Outbound Dialing

**Expected:** if the caller disconnects while an employee/group is being dialed,
the ringing leg must stop and fallback must not start.

**Observed:** the outbound human/group leg originally continued ringing after
the caller disconnected. The outbound dialing lifecycle was not sufficiently
bound to the caller lifecycle.

The project moved outbound dialing into a dedicated `asyncio` task. On caller
disconnect, the cancellation path cancels that dial task, removes the ringing
participant, clears routing state and prevents fallback. It does not cancel the
main AI worker.

The important boundary is cancellation of the work owned by the ended call,
without accidentally terminating the worker that manages the AI lifecycle.
Cancellation deserves the same validation attention as a successful handoff.

## CPU Contention Between Live and Post-Call Work

On the CPU-only host, post-call Whisper competed with realtime Whisper and Qwen.
The next live call became slower. No measured latency delta is available in this
public summary.

Checking for an active call only before launching background work is insufficient:
a new caller can arrive immediately after the check. A launch-time check cannot
protect the complete lifetime of a heavy job.

The project moved background processing into a separate process group:

- Pause the background group with `SIGSTOP` when a live call arrives.
- Resume it with `SIGCONT` after the last live caller disconnects.
- Track `live_caller_sids` separately from all active SIP participants; an
  outbound employee/group participant is not itself a live caller.
- Run heavy post-call jobs sequentially and apply backpressure.

This is the documented PoC scheduling approach. The summary does not specify
queue limits, signal-dispatch code or a stress-test result. Tracking live callers
for scheduling does not establish that the AI pipeline supports concurrent calls.

## Grounded Post-Call Processing

```text
audio -> STT segments -> grounded extraction -> structured CRM result
```

Post-call extraction must derive CRM facts from reliable transcript fragments.
Missing information must remain “not specified / no reliable fragment”; the LLM
must not fill gaps with plausible names, requests or other invented CRM facts.
This requirement also depends on the
[STT confidence boundary](speech-and-language.md#whisper-hallucination-protection).

The CRM Deal ID is captured as an immutable snapshot when the background job is
scheduled. A delayed job must retain its intended CRM destination even if mutable
conversation state changes before processing finishes.

No customer recordings, transcript fragments, Deal IDs or CRM payloads are
published. Grounding is an application requirement, not a claim of zero extraction
errors.

## Acceptance Checks

These are acceptance targets, not a published test execution record.

| Scenario | Expected behavior |
|---|---|
| Caller disconnects during outbound ringing | Cancel the dial task, remove the ringing participant, clear routing state and prevent fallback |
| Dial task cancellation | Leave the main AI worker intact |
| Live call arrives while post-call processing runs | Pause the background process group, including work already started |
| Last live caller disconnects | Resume the background group based on live caller state |
| Outbound SIP participant remains | Distinguish it from a live caller when making scheduling decisions |
| Several heavy post-call jobs are pending | Process sequentially with backpressure |
| Transcript lacks a reliable fact | Emit “not specified / no reliable fragment” rather than an invented CRM value |
| Mutable CRM context changes after scheduling | Use the job's captured Deal ID snapshot |

## Remaining Architecture Boundary

Shared/global handoff state, a realtime STT queue and an audio source remain
limitations. A per-call `CallSession` object is the future direction for isolating
language, pending intent, routing, audio and task ownership. It is not presented
as an implemented multi-session solution.

## Lesson

The lifecycle includes callers who hang up, jobs that wait, changing CRM context
and new calls arriving during background inference. Correctness requires ownership,
cancellation and scheduling across those boundaries, not only a working happy path.

[Project overview](README.md) · [Deterministic routing and SIP handoff](routing-and-sip-handoff.md)
