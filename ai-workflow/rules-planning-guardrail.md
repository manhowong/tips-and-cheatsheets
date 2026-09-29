# Planning-language guardrail

This is a safety net, not the default workflow. It exists for cases where the model doesn't reliably track mode-switching, or the user forgot to switch modes.

## Trigger
- Request uses explicit planning or design language: "plan," "design," "what would you do," "how would you approach," "propose," or similar.

## If triggered
- Read-only: investigate and explain, do not edit any file.
- Return a plan or proposal.
- Wait for explicit approval before making any change.

## If not triggered
- Proceed normally: make the requested edit directly.
- No separate planning step, no extra confirmation.

## When genuinely uncertain
- Multiple valid interpretations: list them briefly and ask which one applies, rather than choosing.
- Unclear scope: ask instead of guessing, even when the guess seems reasonable.