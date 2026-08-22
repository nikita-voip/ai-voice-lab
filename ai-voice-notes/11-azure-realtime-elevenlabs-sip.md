# Azure Realtime and ElevenLabs SIP Integration

## Scenario

An AI voice integration was evaluated across a SIP trunk, Azure-hosted realtime
processing and ElevenLabs voice services. The deployment context included a Malaysia
region requirement, making network path and end-to-end latency part of the design.

## Logical Architecture

```text
Caller
  |
SIP trunk / voice gateway
  |
Realtime application layer
  |-- Azure-hosted realtime model
  `-- ElevenLabs voice service
```

The exact media topology depends on the chosen gateway and API transport. The key
engineering requirement is to identify every point where audio is buffered,
transcoded, detected or sent across regions.

## Latency Budget

Measure the complete conversational loop instead of quoting a single API latency:

- Caller-to-gateway media transport.
- Audio buffering and turn detection.
- Speech/model processing.
- First audio generated for the response.
- Return media transport to the caller.

Regional placement matters, but the shortest geographic path is not automatically
the lowest conversational latency. Measure from the actual SIP ingress point.

## VAD and False Interruption

Voice activity detection can interpret background noise, echo or short acknowledgments
as a new user turn. In a live call this may stop the assistant too early.

Useful validation scenarios include:

- Speech over background noise.
- Short responses such as “yes” or “okay”.
- Caller speech while synthesized audio is playing.
- Echo and audio leakage from speakerphone use.
- Natural pauses inside a sentence.

Tune silence thresholds, interruption behavior and audio preprocessing together.
Aggressive interruption may feel responsive in a quiet test but unstable on the
public telephone network.

## SIP-Side Checks

- Confirm codec selection and avoid unnecessary transcoding.
- Track RTP continuity before and after every conversational turn.
- Separate SIP signaling latency from media and model latency.
- Validate timeout behavior when an upstream AI service is slow or unavailable.
- Preserve correlation identifiers without logging caller-sensitive content.

## Takeaway

A realtime voice agent is an end-to-end media system. Model choice is only one part;
SIP routing, region, buffering, VAD and interruption policy determine how the call
actually feels to the user.
