# Local Voice AI: RU/KZ Telephony PoC

A working local Russian/Kazakh Voice AI proof of concept connected to an
ECSS/SBC telephony path. The engineering focus is the boundary between SIP,
RTP/media, VAD/STT, the LLM, deterministic routing, CRM, asynchronous call
lifecycle and CPU scheduling.

This extends hands-on VoIP and voice infrastructure engineering into Voice AI.
It is a low-volume, single-session-oriented PoC, **not a production-ready platform**.

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
| [RU/KZ speech and conversation state](speech-and-language.md) | Session language, hallucination filtering, confirmed barge-in, TTS normalization and response-language guard |
| [Deterministic routing and SIP handoff](routing-and-sip-handoff.md) | Employee matching, fallback data preservation, SIP outcome interpretation and original caller identity |
| [Call lifecycle, CPU priority and post-call processing](lifecycle-and-post-call.md) | Dial-task cancellation, background process-group pause/resume, backpressure and grounded CRM extraction |

## Evidence and Validation Scope

These notes document project-owner-reported observations, fixes and engineering
results. The public update is based on that project summary; source code, raw SIP
traces, audio, logs and measured benchmarks are not included. Validation tables
describe acceptance checks, not a newly executed or fully passed test suite.

The concrete observations include the lost `employee_fallback` field, missing ECSS
route behind a SIP 484 response, continued outbound ringing after caller hangup,
and post-call Whisper contention with the next live call. No throughput, latency,
recognition-accuracy or production reliability figures are claimed.

## Current Limitations and Pending Work

- Shared/global handoff state, the realtime STT queue and audio source do not yet
  form a true multi-session architecture. A per-call `CallSession` object is a
  future direction, not an implemented result.
- CPU scheduling protects live callers from heavy background work within this
  PoC; it does not establish capacity for concurrent realtime sessions.
- Security hardening remains pending: stronger/rotated LiveKit secrets, minimum
  Bitrix permissions, secrets outside source, and allowlisting/safety controls.
- Production readiness and comprehensive failure/concurrency coverage have not
  been established.

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
