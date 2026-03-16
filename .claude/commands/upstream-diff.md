Show what changed in upstream t3code since the last review.

The last review was at commit `4b811dae61b3042e2bdd053b52f9a75aac46fcb8` (2026-03-16).

If the user provides a different commit hash as $ARGUMENTS, use that instead.

Steps:

1. Fetch upstream:
```bash
git -C ../t3code fetch upstream
```

2. Show commits since the review baseline:
```bash
git -C ../t3code log --oneline <baseline>..upstream/main
```

3. Show a diffstat summary:
```bash
git -C ../t3code diff --stat <baseline>..upstream/main
```

4. Highlight changes in key areas:
- Provider abstraction: `apps/server/src/provider/`
- Orchestration: `apps/server/src/orchestration/`
- Contracts: `packages/contracts/src/`
- Shared: `packages/shared/src/`

For each area with changes, show the diffstat:
```bash
git -C ../t3code diff --stat <baseline>..upstream/main -- <path>
```

5. Report:
- Total commits and files changed
- Which key areas were affected
- Whether any changes impact planned work (Ollama provider spike, remediation items)
- Whether the review doc at `docs/t3code-review-2026-03-16.md` needs updating
