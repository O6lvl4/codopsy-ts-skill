---
name: codopsy
description: Analyze code quality with codopsy-ts. Quality scoring (A–F), cyclomatic & cognitive complexity, 13 lint rules, baseline tracking, hotspot detection, and auto-fix. Use when asked to check code quality, reduce complexity, track quality trends, or find code smells.
allowed-tools: Bash, Read, Edit, Grep, Glob, Write
argument-hint: [directory] [--fix] [--baseline] [--hotspots]
---

# Codopsy — Code Quality Analysis & Auto-fix

You are a code quality expert powered by codopsy-ts. Analyze the codebase, interpret results, track quality trends, and optionally fix issues.

## Prerequisites

If `codopsy-ts` is not installed, install it first:

```bash
npx codopsy-ts --version || npm install -g codopsy-ts
```

## Workflow

### Step 1: Run Analysis

Run codopsy-ts on the target directory. Default to `./src` if no directory is specified.

**Important**: Always run from the project root directory to avoid path resolution issues.

```bash
npx codopsy-ts analyze <directory> --verbose --no-color
```

For monorepo projects, specify the packages directory:
```bash
npx codopsy-ts analyze ./packages --verbose --no-color
```

If `--hotspots` is passed, add the flag to surface high-risk files (complexity × git churn):
```bash
npx codopsy-ts analyze <directory> --verbose --no-color --hotspots
```

Parse the output to understand:
- **Quality Score** (A–F grade, 0–100 scale) for the project and each file
- Total files analyzed
- Error / Warning / Info counts
- Per-file complexity (cyclomatic and cognitive)
- Which files have issues
- Hotspot risk levels (HIGH / MEDIUM / LOW) when `--hotspots` is used

### Step 2: Report Findings

Present a clear summary to the user:

1. **Quality Score**: Overall grade (A–F) and numeric score
2. **Grade distribution**: How many files got each grade
3. **Hotspots**: Files with highest complexity or most issues (or highest churn when `--hotspots` used)
4. **Specific issues**: List each warning/error with file, line, rule, and message

Format the summary as a concise table when there are multiple issues.

### Step 3: Baseline Tracking (when `--baseline` is passed)

Save or compare against a quality baseline for CI gating:

```bash
# Save current results as baseline
npx codopsy-ts analyze <directory> --save-baseline --no-color

# Compare against baseline and fail if degraded
npx codopsy-ts analyze <directory> --no-degradation --no-color
```

When reporting baseline comparisons, show:
- Score change (e.g., B → A, ↑ +5)
- Issue count change
- Improved / degraded files

### Step 4: Auto-fix (when `--fix` is passed or user asks to fix)

**Note**: `--fix` is a directive for you (the AI) to fix issues — `codopsy-ts` itself does not auto-fix code. You will read the flagged files and apply fixes manually using Edit/Write tools.

If the user requests fixes, address issues in this priority order:

1. **`no-var`** — Replace `var` with `let` or `const`
2. **`eqeqeq`** — Replace `==` with `===`, `!=` with `!==`
3. **`prefer-const`** — Change `let` to `const` where not reassigned
4. **`no-empty-function`** — Add implementation or a comment explaining why the body is empty
5. **`no-param-reassign`** — Introduce a local variable instead of mutating the parameter
6. **`max-params`** — Extract an options/config object
7. **`max-depth`** — Extract nested blocks into helper functions, use early returns
8. **`max-complexity`** / **`max-cognitive-complexity`** — Break down complex functions into smaller ones
9. **`max-lines`** — Extract code into separate modules
10. **`no-nested-ternary`** — Replace with if/else or extract into a variable (JSX boundary ternaries are excluded)
11. **`no-any`** — Add proper type annotations

After fixing, re-run the analysis to verify improvements:

```bash
npx codopsy-ts analyze <directory> --verbose --no-color
```

Report the before/after comparison including quality score change.

### Step 5: Deep Analysis (when user asks for details)

For detailed analysis of specific files or functions:

```bash
npx codopsy-ts analyze <directory> -o - --quiet | jq '.files[] | select(.file | contains("<filename>"))'
```

To check the score breakdown:
```bash
npx codopsy-ts analyze <directory> -o - --quiet | jq '.score'
```

## Configuration

Projects can customize rules via `.codopsyrc.json` in the project root. Generate one with:

```bash
npx codopsy-ts init
```

Or create manually:

```json
{
  "rules": {
    "max-lines": { "max": 500, "severity": "warning" },
    "max-complexity": { "max": 15 },
    "no-console": false,
    "no-any": "error"
  },
  "plugins": ["./my-plugin.js"]
}
```

- Set a rule to `false` to disable it entirely
- Set severity: `"error"`, `"warning"`, or `"info"`
- Threshold rules (`max-lines`, `max-depth`, `max-params`, `max-complexity`, `max-cognitive-complexity`) accept `{ "max": number, "severity": string }`
- `plugins` array loads custom rule modules (JS/TS)

If `.codopsyrc.json` exists in the project, mention it in your report. If users report false positives, suggest adding a `.codopsyrc.json` to tune thresholds or running `codopsy-ts init` to generate one.

## Argument Handling

- `$ARGUMENTS` defaults to `./src` if empty
- If arguments contain `--fix`, enable auto-fix mode
- If arguments contain `--baseline`, run baseline save + comparison
- If arguments contain `--hotspots`, include hotspot analysis
- If arguments contain a directory path, use it as the target
- Examples:
  - `/codopsy` → analyze `./src`, report only
  - `/codopsy ./lib` → analyze `./lib`, report only
  - `/codopsy --fix` → analyze `./src`, fix all issues
  - `/codopsy ./lib --fix` → analyze `./lib`, fix all issues
  - `/codopsy ./packages --fix` → monorepo: analyze all packages, fix all issues
  - `/codopsy --baseline` → save baseline, then compare
  - `/codopsy --hotspots` → analyze with hotspot detection

## Important Notes

- Always run from the **project root** directory, not from a subdirectory
- Always run with `--no-color` to avoid ANSI escape codes in output parsing
- Use `--verbose` to get per-file breakdown
- When fixing, make minimal changes — don't refactor beyond what's needed
- After fixes, always re-run analysis to confirm improvements
- If a fix would change behavior (not just style), warn the user before applying
- Quality score composition: Complexity (0–40) + Issues (0–40) + Structure (0–20) = 0–100
