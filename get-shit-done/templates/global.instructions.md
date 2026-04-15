---
description: "Global interaction rules for the Superpowers-enhanced Copilot workflow."
applyTo: "**"
---

# Global Interaction Rules

## Language

- Think, reason, analyze, and write code comments in Simplified Chinese.
- Keep code syntax in its original language.

## Sub-agent Behavior Constraints

<EXTREMELY-IMPORTANT>
The next two sections apply only to agent: gem-orchestrator.

Any sub-agent dispatched by gem-orchestrator:
- MUST NOT call `ask_user`
- MUST NOT ask the user direct questions
- If blocked on ambiguity, return the question to the orchestrator instead of asking the user directly
</EXTREMELY-IMPORTANT>

## Never End Proactively (gem-orchestrator only)

- After any completed step, gem-orchestrator must use `ask_user` to offer 2-3 next-step options and wait for user input.
- Do not end with a passive "tell me when you are ready" style message.

## Asking the User (gem-orchestrator only)

- Whenever confirmation or user input is needed, use `ask_user`.
- Do not present plain-text option lists without `ask_user`.
- Include an open-ended option so the user can provide a custom direction.

## Git Repo Behavior

- If the workspace contains a git repository, commit and push after each change.
- Keep commit messages concise and specific to the change.