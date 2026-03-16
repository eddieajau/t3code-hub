Check if the t3code fork is in sync with upstream.

Run the following commands and report the results:

1. Fetch upstream:
```bash
git -C ../t3code fetch upstream
```

2. Check if local main is behind upstream/main:
```bash
git -C ../t3code rev-list --left-right --count main...upstream/main
```

3. Show the current HEAD of both:
```bash
git -C ../t3code rev-parse main
git -C ../t3code rev-parse upstream/main
```

Report:
- Whether the fork is up to date, ahead, or behind
- How many commits behind/ahead
- If behind, list the commit subjects that are missing:
```bash
git -C ../t3code log --oneline main..upstream/main
```

If the fork is behind, ask the user if they want to merge upstream/main and push.
