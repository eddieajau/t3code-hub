# Agent Instructions

## Read first

1. TODO.md — evaluate what we are working on next

## Dev environment

TBD

## Architecture rules (never violate these)

TBD

## Key commands

TBD

## Package hygiene

Never hand-write package.json. Use npm install / npm pkg set.

## Change review

Always show proposed changes and get approval before editing files. Exception: trivial, unambiguous changes (e.g. adding a single config line) can be made directly.

## Working style

- One step at a time — implement, then stop for review
- If anything is ambiguous or has meaningful trade-offs, ask first

## Verify steps

TODO items marked "Verify:" require manual testing in a running browser/server.
Never tick these off — leave them unchecked and list them for the user to confirm.
After each step, remind the user which verify steps to run and what to look for,
based on the specific changes made (not just a copy of the TODO text).

## Commits

Never commit unless the user explicitly asks. After finishing a step, stop and
wait — do not stage or commit automatically.
