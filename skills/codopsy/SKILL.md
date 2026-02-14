---
name: codopsy
description: Analyze code quality with codopsy-ts. Measures cyclomatic & cognitive complexity, detects lint issues, and can auto-fix problems. Use when asked to check code quality, reduce complexity, or find code smells.
allowed-tools: Bash, Read, Edit, Grep, Glob, Write
argument-hint: [directory] [--fix]
---

# Codopsy — Code Quality Analysis & Auto-fix

You are a code quality expert powered by codopsy-ts. Analyze the codebase, interpret results, and optionally fix issues.

## Prerequisites

If `codopsy-ts` is not installed, install it first:

```bash
npx codopsy-ts --version || npm install -g codopsy-ts
```

## Workflow

### Step 1: Run Analysis

Run codopsy-ts on the target directory. Default to `./src` if no directory is specified.

```bash
npx codopsy-ts analyze <directory> --verbose --no-color
```

Parse the output to understand:
- Total files analyzed
- Error / Warning / Info counts
- Per-file complexity (cyclomatic and cognitive)
- Which files have issues

### Step 2: Report Findings

Present a clear summary to the user:

1. **Overall health**: Are there errors or warnings?
2. **Hotspots**: Files with highest complexity or most issues
3. **Specific issues**: List each warning/error with file, line, rule, and message

Format the summary as a concise table when there are multiple issues.

### Step 3: Auto-fix (when `--fix` is passed or user asks to fix)

If the user requests fixes, address issues in this priority order:

1. **`no-var`** — Replace `var` with `let` or `const`
2. **`eqeqeq`** — Replace `==` with `===`, `!=` with `!==`
3. **`prefer-const`** — Change `let` to `const` where not reassigned
4. **`no-param-reassign`** — Introduce a local variable instead of mutating the parameter
5. **`max-params`** — Extract an options/config object
6. **`max-depth`** — Extract nested blocks into helper functions, use early returns
7. **`max-complexity`** / **`max-cognitive-complexity`** — Break down complex functions into smaller ones
8. **`max-lines`** — Extract code into separate modules
9. **`no-nested-ternary`** — Replace with if/else or extract into a variable
10. **`no-any`** — Add proper type annotations

After fixing, re-run the analysis to verify improvements:

```bash
npx codopsy-ts analyze <directory> --verbose --no-color
```

Report the before/after comparison.

### Step 4: Deep Analysis (when user asks for details)

For detailed analysis of specific files or functions:

```bash
npx codopsy-ts analyze <directory> -o - --quiet | jq '.files[] | select(.file | contains("<filename>"))'
```

## Argument Handling

- `$ARGUMENTS` defaults to `./src` if empty
- If arguments contain `--fix`, enable auto-fix mode
- If arguments contain a directory path, use it as the target
- Examples:
  - `/codopsy` → analyze `./src`, report only
  - `/codopsy ./lib` → analyze `./lib`, report only
  - `/codopsy --fix` → analyze `./src`, fix all issues
  - `/codopsy ./lib --fix` → analyze `./lib`, fix all issues

## Important Notes

- Always run with `--no-color` to avoid ANSI escape codes in output parsing
- Use `--verbose` to get per-file breakdown
- When fixing, make minimal changes — don't refactor beyond what's needed
- After fixes, always re-run analysis to confirm improvements
- If a fix would change behavior (not just style), warn the user before applying
