# CallMyCall OpenClaw Skill

Focused OpenClaw skill for operating the CallMyCall API from chat.

Docs and portal:

- https://api.callmycall.com

## Install (local)

1. Clone or copy this repo into your OpenClaw skills folder.
2. Restart OpenClaw (or refresh skills) so it detects the new skill.

Common locations:

- `<workspace>/skills/callmycall-openclaw`
- `~/.openclaw/skills/callmycall-openclaw`

## Usage

Examples:

- "Call +46700000000 and confirm my appointment for Tuesday."
- "Show my recent calls."
- "End call 1."
- "Show results for call 2."

## Files

- `SKILL.md` — primary instructions and workflows
- `references/api.md` — API reference subset
- `examples/prompts.md` — example prompts and expected actions

## API Key

You need a CallMyCall API key. The skill will ask for it if not present.

## Notes

This skill is pull based. It does not rely on webhook callbacks; results are fetched on demand.
