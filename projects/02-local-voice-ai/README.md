# Local Voice AI: RU/KZ Telephony PoC

A working local Russian/Kazakh Voice AI proof of concept connected to an
ECSS/SBC telephony path. The engineering focus is the boundary between SIP,
RTP/media, VAD/STT, the LLM, deterministic routing, CRM, asynchronous call
lifecycle and CPU scheduling.

This extends hands-on VoIP and voice infrastructure engineering into Voice AI.
**Current status: Limited pilot / ongoing validation. Not production-ready.**

The original CPU-only, single-session PoC is retained here as the **Legacy** stage.
The completed Legacy-to-**MultiCall** transition adds independent LiveKit rooms
and per-call state. Two independent simultaneous calls have been validated;
the STT queue remains sequential.

## Project Evolution

| Stage | Engineering result | Validation boundary |
|---|---|---|
| Legacy, documented October 2, 2026 | Real SIP calls, local RU/KZ STT/LLM/TTS, deterministic routing and Bitrix integration on CPU | Shared state and a single-session-oriented pipeline |
| MultiCall, documented October 16, 2026 | Independent rooms and isolated call contexts, streamed LLM-to-TTS output, background CRM work and disk-backed post-processing | Two concurrent calls confirmed; sequential STT; ongoing validation |

Start with [Legacy → MultiCall](legacy-to-multicall.md) for the current architecture,
validation results and roadmap. The earlier notes below preserve the diagnostic
history; they are not claims of complete Legacy/MultiCall equivalence.

## Logical Architecture

```text
Phone / PSTN <-> ECSS / SBC <-> SIP / RTP <-> LiveKit SIP + LiveKit Server
                                                      |
                                             VAD -> local STT
                                                      |
                                                 local LLM
                                                      |
                                                 local TTS
                                                      |
                                      LiveKit -> SIP / RTP -> caller

Conversation intent -> deterministic backend -> Bitrix employee / department data
                                            -> employee / group SIP handoff

Post-call audio -> STT segments -> grounded extraction -> structured CRM result
```

The diagram shows functional boundaries, not a customer deployment topology.

## Environment

| Component | Project environment |
|---|---|
| Host | Ubuntu 22.04; Intel i7-13700; 32 GB RAM |
| Compute | CPU-only; no discrete GPU |
| Realtime platform | LiveKit + Redis in Docker; LiveKit SIP |
| STT | Kazakh Whisper Large-v3 Turbo; CTranslate2 INT8 |
| LLM | Qwen3.5-4B-Kazakh; GGUF Q4_K_M; llama.cpp |
| TTS | Piper |
| Business integration | Bitrix employee, department and CRM data |

## Engineering Notes

| Note | New project results |
|---|---|
| [Legacy → MultiCall](legacy-to-multicall.md) | Per-call isolation, output lifecycle, CRM background work, recordings, disk queue, bounded concurrency results and roadmap |
| [MultiCall engineering lessons](multicall-engineering-lessons.md) | Directory permissions, echo and false interruptions, language switching, Whisper truncation and LLM response regressions |
| [RU/KZ speech and conversation state](speech-and-language.md) | Session language, hallucination filtering, confirmed barge-in, TTS normalization and response-language guard |
| [Deterministic routing and SIP handoff](routing-and-sip-handoff.md) | Employee matching, fallback data preservation, SIP outcome interpretation and original caller identity |
| [Call lifecycle, CPU priority and post-call processing](lifecycle-and-post-call.md) | Dial-task cancellation, background process-group pause/resume, backpressure and grounded CRM extraction |

## Evidence and Validation Scope

These notes document project-owner-reported observations, fixes and engineering
results. Source code, raw SIP traces, audio and logs are not included.
The [MultiCall validation section](legacy-to-multicall.md#reported-validation)
separates reported checks and a limited STT timing experiment from unverified
capacity claims. Tables in the earlier notes describe acceptance checks, not a
newly executed or fully passed test suite.

The concrete observations include the lost `employee_fallback` field, missing ECSS
route behind a SIP 484 response, continued outbound ringing after caller hangup,
and post-call Whisper contention with the next live call. The later timing
experiment does not establish end-to-end call latency, throughput,
recognition accuracy or production reliability.

## Current Limitations and Pending Work

- The Legacy shared-state limitation motivated MultiCall. Independent rooms and
  isolated history, language, routing and CRM context are now implemented; the
  internal class layout is not published or inferred from that result.
- STT recognizes completed utterances and still uses a sequential queue; it is
  not full streaming STT. Two-call validation does not establish larger capacity.
- False interruptions and speakerphone echo remain validation concerns.
- Security hardening remains pending: stronger/rotated LiveKit secrets, minimum
  Bitrix permissions, secrets outside source, and allowlisting/safety controls.
- Full Legacy equivalence, automatic deal creation and physical recovery after
  power loss are not confirmed. See the [roadmap](legacy-to-multicall.md#roadmap)
  for further performance and acceptance work.

## Relationship to Existing Work

The [SIP-connected Voice AI project](../01-sip-connected-voice-ai/README.md)
covers the earlier Vapi/Twilio/OpenAI integration and Russian/English translation.
Its [Vapi-to-ECSS transfer case](https://github.com/nikita-voip/voice-integration-projects/tree/main/projects/01-vapi-ecss-sip-transfer)
remains the canonical location for that project's transfer artifacts. Those
artifacts do not validate this Local Voice AI PoC.

The existing [realtime integration note](../../ai-voice-notes/11-azure-realtime-elevenlabs-sip.md)
introduces false VAD interruption. The local project adds a confirmation window
and interaction with local STT, language state and telecom actions.

[Back to AI Voice Lab](../../README.md)
