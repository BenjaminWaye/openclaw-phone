---
name: openclaw-phone
description: Use CallMyCall API to start, end, and check AI phone calls, and return results in chat. Use when the user asks to call someone, schedule a call, end a call, or fetch call results.
homepage: https://api.callmycall.com
---

# CallMyCall (OpenClaw Skill)

This skill helps you operate CallMyCall from chat. It is pull based (no webhook callbacks): you start a call, store the returned call ID in state, and later fetch status and results on demand.

## API Key Resolution (OpenClaw best practice)

Resolve credentials in this order:

1. Environment variable: `CALLMYCALL_API_KEY` (preferred)
2. OpenClaw user config: `~/.openclaw/openclaw.json` under `skills.openclaw-phone.apiKey`
3. If still missing, prompt user once for the key and ask permission to persist it in `~/.openclaw/openclaw.json`

Persistence rules:

- Never store API keys in `SKILL.md`, examples, references, or memory/state files.
- Do not write API keys into `recent_calls` or any conversation-visible output.
- If user declines persistence, use the provided key for the current task only.

## How This Skill Should Work

1. Resolve API key using the order in "API Key Resolution (OpenClaw best practice)".
2. Collect required call info using a layered gating flow (below).
3. Present a short review summary and ask for confirmation.
4. On confirmation, call `POST /v1/start-call`.
5. Store the returned `sid` in state as a recent call.

## Required Auth

Send the API key in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

Never echo the API key back to the user or include it in logs/summaries.

## Stateful Calls List (required)

Maintain a list (last 10) of recent calls in state:

- `recent_calls`: array of objects
  - `id` (call SID)
  - `phone_number`
  - `task`
  - `started_at`
  - `status` (optional, updated when you fetch)

Use this list to let the user say "end call 1" or "show results for call 2".

## Layered Gating Flow (copy from web app)

Do not rely on a single validation step. Use all layers below.

### Layer 1: Structured collection contract

Do not finalize a task until all required fields exist:

- `phone_number`
- `language`
- `call_brief` (background + goals)

### Layer 2: Task gap analysis

When the user gives the initial request, analyze what is missing. Then ask only for missing info. If the user answers partially, repeat analysis and keep asking for the remaining gaps.

### Layer 3: Prompt level enforcement

While missing info exists, continue gathering required fields. Do not proceed to the call until all required fields are present.

### Layer 4: Runtime validation before finalizing

Before sending the call request:

- Ensure phone exists and is E.164
- Block emergency or premium numbers
- Ensure `from_number` is not the same as `phone_number`
- Normalize `language`; normalize `openaiVoice` only if provided
- If scheduling is present, parse and clamp to a valid time

### Layer 5: Human review gate

Present a short review summary:

- Phone number
- Call brief (background + goals)
- Language (and voice if provided)
- Any schedule

Ask: "Confirm and place the call?" Do not proceed without explicit confirmation.

## Workflows

### Start a Call

1. Collect required fields using the layered gating flow.
2. Call `POST /v1/start-call`.
3. Store the returned `sid` in `recent_calls`.
4. Reply with confirmation and the call ID.

If the user asks to schedule a call for later, include the schedule in the task or decline if scheduling is not supported by the API.

### List Recent Calls

1. Read `recent_calls` from state.
2. For each call, fetch status via `GET /v1/calls/:callId` if needed.
3. Display a numbered list.

### End a Call

If the user says "end the call" without specifying which, list recent calls and ask which one.

If there is only one active call, confirm and end it.

Call:

- `POST /v1/end-call` with `{ callSid }`.

### Get Results

When the user asks for call results:

1. Fetch status via `GET /v1/calls/:callId`.
2. If available, fetch transcript via `GET /v1/calls/:callId/transcripts/stream`.
3. If the call was recorded, fetch recording URL via `GET /v1/calls/:callSid/recording`.

Return:

- Status (completed, failed, canceled)
- Short summary (1 to 3 bullets)
- Transcript excerpt (first few lines)
- Recording URL (if present)

## Safety and UX

- If user input is ambiguous, ask a clarification question.
- Never expose secrets or store API keys in transcript.
- If a request fails, show the HTTP error and suggest next steps.

## References

- Full API reference: `references/api.md`
- Examples: `examples/prompts.md`
