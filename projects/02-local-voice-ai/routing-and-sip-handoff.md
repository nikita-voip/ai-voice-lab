# Deterministic Routing and SIP Handoff

> This note preserves the original routing and SIP findings. MultiCall retains
> deterministic routing and isolates routing/CRM state per call; see
> [Legacy → MultiCall](legacy-to-multicall.md) for the newer stage.

## Conversation and Actions Have Different Owners

The LLM handles conversation. A deterministic backend resolves destinations and
performs routing/telecom actions. The LLM must not invent telephone numbers,
transfers or announce that an action succeeded before the backend has confirmed it.

## Employee Search and Routing State

The project uses normalization, RU/KZ folding and fuzzy employee matching.
A similarity threshold and confidence margin separate a credible match from
an ambiguous one. When the evidence is insufficient, the agent asks for
clarification instead of guessing.

Pending intent persists across utterances so a clarification can complete an
earlier request. Routing-state lifecycle and reset matter equally: a pending
selection must not leak into an unrelated request or survive a terminated call.
No private directory, employee names, destination numbers or numeric matching
thresholds are included.

## Bitrix Hierarchy Becomes Telecom Routing Data

The routing relationship is:

```text
employee -> department -> fallback group
```

CRM/HR hierarchy is operational routing input, not merely descriptive metadata.
Destination resolution and preservation of the resulting data are separate
responsibilities.

### Lost Fallback Data Across Layers

**Expected:** correctly calculated `employee_fallback` data reaches the layer
that performs handoff/fallback decisions.

**Observed:** the data was calculated correctly but lost when another layer
rebuilt a dictionary. The failure was at the data boundary, not in the hierarchy
calculation itself.

The fix is to preserve the resolved fallback field through that reconstruction.
Validation must follow the field from calculation to its consumer, rather than
checking only the first function's output. The project summary does not specify
the complete dictionary schema or provide a code diff.

## SIP Outcomes Must Control the Result

| Observation | Project interpretation / action |
|---|---|
| SIP 480 | Do not automatically tell the caller the destination is busy. Use “currently unavailable” unless more specific evidence supports another explanation. |
| SIP 484 during AI handoff | The project investigation identified a missing ECSS route, rather than LiveKit or AI, as the cause. The corrective boundary was ECSS routing. |
| LiveKit `participant_connected` | This event alone did not prove final SIP transfer success. |
| Final SIP outcome | Use it to determine whether handoff succeeded before reporting success. |

These are local project findings, not a universal diagnosis of every SIP 484.
The public summary does not specify the missing route's configuration or include
its signaling trace, so neither is reconstructed here.

## Original Caller Identity

The original caller number was obtained from SIP participant attributes and used
on the outbound leg to the employee/group. AI handoff must preserve existing
telephony semantics, including caller identity, rather than replacing it with an
arbitrary AI identity.

Attribute names, destination numbers and production header examples are omitted.
The documented result does not imply interoperability with every carrier or PBX.

## Acceptance Checks

These describe validation targets; the update does not claim a newly executed suite.

| Scenario | Expected behavior |
|---|---|
| Ambiguous employee match | Ask for clarification; do not choose the nearest name automatically |
| Clarification on the next utterance | Complete the pending intent using preserved context |
| Dictionary reconstruction | Retain `employee_fallback` through to the routing consumer |
| Employee / department / fallback resolution | Use directory-derived destinations rather than LLM-generated numbers |
| SIP failure after an application event | Report the final failure; do not infer success from `participant_connected` |
| SIP 480 without more specific evidence | Describe the destination as currently unavailable |
| AI-to-employee/group handoff | Preserve the original caller identity on the outbound leg |
| Call termination or routing reset | Clear pending routing state; do not reuse a stale destination |

## Lesson

Successful conversation does not prove successful telephony. Preserve resolved
business data between layers, validate the actual SIP outcome and retain the
caller's identity throughout handoff.

[Project overview](README.md) · [Call lifecycle and post-call processing](lifecycle-and-post-call.md)
