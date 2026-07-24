# AI Call Transfer

One of the hardest parts of an AI voice system isn't speech recognition.

It's transferring the call without breaking the conversation.

## Typical flow

Caller

↓

AI Agent

↓

Intent detected

↓

Human operator

↓

Conversation continues

## Things that matter

- Context transfer
- SIP REFER or bridge logic
- RTP continuity
- Conversation history
- Fast handoff

## Lesson

The user shouldn't notice the transfer.

Only the technology should.
