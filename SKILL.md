---
name: running-angular-tests
description: Use when running Angular tests with ng test to capture complete output and search for failures or specific results
---

# Running Angular Tests

## Overview

Pipe-based approaches (`ng test 2>&1 | tail -20`) silently truncate output and miss failures. Always redirect to a file so the full output is available for searching.

## Process

### 1. Before Running

- Identify what you're looking for (specific test failures, a suite, coverage info).
- If you only need a subset of tests, use `--include` or `--test-file` to limit scope — faster and less noise.

### 2. Run Tests — Redirect to File

Always redirect both stdout and stderr. Use `--no-progress` to suppress the Karma progress spinner, and `NO_COLOR=1` to strip ANSI escape codes that pollute the file.

```bash
NO_COLOR=1 npx ng test --no-watch --no-progress > /tmp/ng-test-output.txt 2>&1
```

To run a specific spec file:
```bash
NO_COLOR=1 npx ng test --no-watch --no-progress --include='**/foo.spec.ts' > /tmp/ng-test-output.txt 2>&1
```

**Before running**, verify you can write and will be able to delete the file:
```bash
touch /tmp/ng-test-output.txt && echo "ok"
```

### 3. Search the Output File

Use Grep on the file — do NOT pipe or tail. Search for what matters:

```bash
# Find failures
grep -n "FAILED\|Error\|✗\|✕\|FAIL" /tmp/ng-test-output.txt

# Find a specific describe/it block
grep -n "my component" /tmp/ng-test-output.txt

# Get context around a failure
grep -n -A 10 "FAILED" /tmp/ng-test-output.txt
```

### 4. Clean Up

Delete the file after you have the information you need:
```bash
rm /tmp/ng-test-output.txt
```

## Common Mistakes

| Mistake | Fix |
|--------|-----|
| `ng test \| tail -20` | Truncates output — use file redirect |
| Grepping while tests run | Wait for exit, then grep the file |
| Forgetting `--no-watch` | Tests run forever; always add `--no-watch` |
| Leaving output file behind | Always `rm` after reading |
| Reading ANSI codes in file | Use `NO_COLOR=1` to strip them |
