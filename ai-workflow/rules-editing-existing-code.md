# Rules for editing existing codebases

These rules apply when editing, extending, or fixing an existing codebase. They do not apply when starting a project or file from scratch, where there is no existing design, convention, or code to preserve.

## Frontend / UI design
- Do not change existing UI design, style, fonts, layout, spacing, icons, sizes, or component structure.
- New elements must match the existing design system: reuse existing components, classes, and tokens before introducing new ones.

## Comments
- Add comments and separators where they aid readability; skip them for self-explanatory code.
- Do not use numbered headings in comments.
- JSX context (between tags, e.g. between two `<div>`s): use `{/* ... */}`.
- Inside a function body (e.g. inside `onDrop` or `onClick`): use `//`.

## Handling changes with wider impact
Always ask first for:
- adding a new dependency, library, or external service
- a change with backward-compatibility or breaking-change implications, for existing behavior
- anything destructive: deleting files, dropping data, force-pushing, overwriting migrations
- running a command that installs packages, touches infrastructure, or has side effects outside the repo

For these, describe the options and trade-offs and wait for a decision. Do not implement a fix for these on your own judgment.

## General editing
- Change only what the current request explicitly covers. Do not refactor, reformat, rename, or "clean up" surrounding code, even if it looks improvable.
- Match the file's existing conventions (naming, formatting, structure) rather than introducing new patterns.
- Preserve existing comments and logic in code you are not asked to change.
- Do not add new dependencies, libraries, or config options without asking first.

## Reporting back
- After completing an edit, respond with a concise test plan covering every change made. No other commentary.
- If nothing was changed (blocked, needs clarification, plan-only), say so plainly instead of returning a test plan.