# SIP-Connected Voice AI and Realtime Translation

## Objective

Connect Voice AI services to real telephony infrastructure and validate the whole
call path: SIP registration, signaling, media, realtime Russian/English translation
and transfer behavior.

## Components Used

- Vapi
- Twilio
- OpenAI realtime voice capabilities
- SIP and RTP
- ECSS-10 integration

This is a sanitized engineering summary. It does not contain customer credentials,
telephone numbers, private addresses or proprietary configuration.

## Logical Architecture

```text
PSTN / caller
      ↓
Telephony provider / SIP path
      ↓
Voice AI application
      ↓
Realtime speech and translation
      ↓
ECSS / transfer destination
```

## SIP Registration Validation

The integration exercised the standard digest-authentication sequence:

```text
REGISTER
  ← 401 Unauthorized + challenge
REGISTER + Authorization
  ← 200 OK
```

The first `401 Unauthorized` is expected in this flow. Registration is proven by the
authenticated retry and final `200 OK`, not by suppressing the challenge.

## Capabilities Tested

- SIP registration from the Voice AI integration
- AI voice-agent call handling
- Russian to English and English to Russian realtime translation
- SIP call path through the connected voice infrastructure
- Blind transfer using SIP REFER
- Experimental warm-transfer behavior

## Transfer Findings

Blind REFER was confirmed by SIP trace and completed as expected. The experimental
warm-transfer mode did not create the expected second SIP INVITE on ECSS, so it is
recorded as an investigation result rather than a completed feature.

This distinction matters: an application can report a transfer action while the
telephony trace shows that no second dialog was created.

## Engineering Checks

1. Confirm the complete REGISTER challenge/response sequence.
2. Follow dialog identifiers and Request-URI changes across the SIP path.
3. Inspect SDP and RTP separately from application events.
4. Measure translation behavior across both language directions.
5. Confirm each expected transfer leg in the SIP trace.
6. Treat blind and warm transfer as different call architectures.

## Lessons Learned

- Voice AI quality depends on the telephony and media path around the model.
- `401 Unauthorized` can be a normal authentication challenge or a real configuration
  fault; the surrounding SIP sequence determines which.
- Transfer validation requires signaling evidence for every expected call leg.
- Failed experiments are useful when the trace clearly identifies the missing step.

Detailed transfer artifacts are kept in the related
[production-style integration case](https://github.com/nikita-voip/voice-integration-projects/tree/main/projects/01-vapi-ecss-sip-transfer)
to avoid duplicating captures and screenshots.
