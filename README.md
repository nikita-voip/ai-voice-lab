# AI Voice Lab

Hands-on notes and experiments at the boundary between voice infrastructure and
realtime AI. The focus is the complete path—telephony, SIP signaling, media,
AI processing and transfer behavior—not a standalone chatbot demo.

## Featured Projects

- [Local Voice AI: RU/KZ Telephony PoC](projects/02-local-voice-ai/README.md)
  — CPU-only LiveKit SIP, local STT/LLM/TTS, language state, deterministic Bitrix
  routing and SIP handoff; Legacy-to-MultiCall evolution with two concurrent calls
  validated, per-call isolation and disk-backed CRM post-processing.
  Limited pilot / ongoing validation. Not production-ready.

- [SIP-Connected Voice AI and Realtime Translation](projects/01-sip-connected-voice-ai/README.md)
  — Vapi, Twilio, OpenAI, ECSS, SIP registration, Russian/English translation and
  trace-based transfer testing

## Notes

1. [AI Is Not the Hard Part](ai-voice-notes/01-ai-is-not-the-hard-part.md)
2. [Why AI Voice Is Mostly Telephony](ai-voice-notes/02-why-ai-voice-is-mostly-telephony.md)
3. [Vapi SIP Integration](ai-voice-notes/03-vapi-sip-integration.md)
4. [AI Voice Translator](ai-voice-notes/04-ai-voice-translator.md)
5. [OpenAI Realtime Notes](ai-voice-notes/05-openai-realtime-notes.md)
6. [Common AI Voice Failures](ai-voice-notes/06-common-ai-voice-failures.md)
7. [Kamailio for AI Voice](ai-voice-notes/07-kamailio-for-ai-voice.md)
8. [FreeSWITCH Conference Notes](ai-voice-notes/08-freeswitch-conference-notes.md)
9. [AI Call Transfer](ai-voice-notes/09-ai-call-transfer.md)
10. [Realtime Voice Translation](ai-voice-notes/10-realtime-voice-translation.md)
11. [Azure Realtime and ElevenLabs SIP Integration](ai-voice-notes/11-azure-realtime-elevenlabs-sip.md)

## Topics

- SIP ↔ AI Integration
- AI Voice Agents
- OpenAI Realtime API
- Vapi
- Twilio Voice
- FreeSWITCH
- Real-time Voice Translation
- AI Call Transfer
- WebRTC
- AI Voice Troubleshooting
- Local RU/KZ Voice AI
- Deterministic CRM Routing and Async Call Lifecycle

## Scope

These are implementation notes and engineering observations, not vendor benchmarks.
Deployment-specific credentials, endpoints and customer data are not published.

## Related Production Case

- [Vapi to ECSS SIP Call Transfer](https://github.com/nikita-voip/voice-integration-projects/tree/main/projects/01-vapi-ecss-sip-transfer)

## Portfolio Navigation

- [Voice Integration Projects](https://github.com/nikita-voip/voice-integration-projects)
- [VoIP Notes](https://github.com/nikita-voip/voip-notes)
- [ECSS-10 Training](https://github.com/nikita-voip/ecss10-training)

---

Author: Nikita Maksimenko

Senior VoIP / Voice Infrastructure Engineer
