# Agent Instructions

## Read first

1. TODO.md — evaluate what we are working on next

## Multi-repo workflow

This hub repo (`t3code-hub`) holds docs, plans, and TODO tracking.
Code changes go in the sibling `../t3code` repo on feature branches.

- **Do not create branches or edit files in `../t3code` unless the user
  explicitly says to start a coding task.** Planning, spike docs, and TODO
  updates all happen here in t3code-hub first.
- When starting a coding task, the TODO item will name the branch.

## Dev environment

See INSTALL.md.

## Key commands

Run from `../t3code`:

- `bun run typecheck` — full monorepo type check
- `bun run test` — run test suite

## Package hygiene

Never hand-write package.json. Use npm install / npm pkg set.

## Change review

Always show proposed changes and get approval before editing files.
Exception: trivial, unambiguous changes (e.g. adding a single config line)
can be made directly.

## Working style

- **One task at a time.** Complete one TODO task, stop, and wait for the
  user to review and say "commit" or "next". Do not start the next task
  on your own.
- If anything is ambiguous or has meaningful trade-offs, ask first.

## Verify steps

TODO items marked "Verify:" require manual testing in a running browser/server.
Never tick these off — leave them unchecked and list them for the user to confirm.
After each step, remind the user which verify steps to run and what to look for,
based on the specific changes made (not just a copy of the TODO text).

## Commits

Never commit unless the user explicitly asks. After finishing a step, stop and
wait — do not stage or commit automatically.
