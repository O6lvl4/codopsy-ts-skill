<h1 align="center">codopsy-ts-skill</h1>

<p align="center">
  <strong>Claude Code plugin for <a href="https://github.com/O6lvl4/codopsy-ts">codopsy-ts</a></strong>
</p>

<p align="center">
  Analyze code quality, track quality trends, and auto-fix issues &mdash; right from Claude Code.
</p>

<p align="center">
  <a href="README.ja.md">日本語</a>
</p>

---

## What it does

This plugin adds the `/codopsy` skill to Claude Code. It runs [codopsy-ts](https://github.com/O6lvl4/codopsy-ts) on your codebase, interprets the results, and can automatically fix the issues it finds.

**Quality Score** — A&ndash;F grade per file and project (0&ndash;100 scale)

**Analyze** — cyclomatic & cognitive complexity, 13 lint rules, per-file breakdown

**Baseline** — save a quality snapshot, fail CI if quality degrades

**Hotspots** — find high-risk files by combining complexity with git churn

**Fix** — Claude reads the analysis, understands each issue, and applies minimal fixes

---

## Install

```
/plugin install codopsy@O6lvl4/codopsy-ts-skill
```

> Requires [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) v1.1.0+ (`npm install -g codopsy-ts`)

---

## Usage

```bash
# Analyze ./src (default)
/codopsy

# Analyze a specific directory
/codopsy ./lib

# Analyze and auto-fix all issues
/codopsy --fix

# Analyze a specific directory and fix
/codopsy ./lib --fix

# Save baseline and compare
/codopsy --baseline

# Include hotspot analysis (complexity × git churn)
/codopsy --hotspots
```

---

## What happens

### Report mode (default)

1. Runs `codopsy-ts analyze` with `--verbose`
2. Shows **Quality Score** (A&ndash;F grade) and grade distribution
3. Presents a summary: file count, error/warning/info counts, hotspots
4. Lists each issue with file, line, rule, and message

### Baseline mode (`--baseline`)

1. Saves current quality metrics as a baseline snapshot
2. On subsequent runs, compares against baseline
3. Reports score change, issue delta, and improved/degraded files

### Hotspot mode (`--hotspots`)

1. Runs analysis with `--hotspots` flag
2. Combines complexity scores with git churn (commits, authors)
3. Reports risk levels: HIGH, MEDIUM, LOW

### Fix mode (`--fix`)

1. Runs analysis
2. Fixes issues in priority order:
   - `no-var` → replace with `let`/`const`
   - `eqeqeq` → replace `==` with `===`
   - `prefer-const` → change `let` to `const`
   - `no-empty-function` → add implementation or explanatory comment
   - `no-param-reassign` → introduce local variable
   - `max-params` → extract options object
   - `max-depth` → extract helpers, early returns
   - `max-complexity` / `max-cognitive-complexity` → break down functions
   - `max-lines` → extract into modules
   - `no-nested-ternary` → replace with if/else (JSX boundaries excluded)
   - `no-any` → add type annotations
3. Re-runs analysis to verify improvements
4. Reports before/after comparison with quality score change

---

## Example

```
> /codopsy

=== Codopsy Analysis ===

Quality Score: A (95/100)
Files: 12 | Errors: 0 | Warnings: 3 | Info: 5
Grade distribution: A: 9, B: 2, C: 1

| File             | Line | Rule           | Score | Message                                    |
|------------------|------|----------------|-------|--------------------------------------------|
| src/parser.ts    | 42   | max-complexity | B (78)| Function "parse" has complexity 14 (max 10) |
| src/utils.ts     | 8    | no-var         | A (92)| Unexpected var, use let or const            |
| src/handler.ts   | 15   | eqeqeq         | A (90)| Expected "===" but found "=="              |

> /codopsy --fix

Fixed 3 issues:
  ✓ src/utils.ts:8 — var → const
  ✓ src/handler.ts:15 — == → ===
  ✓ src/parser.ts:42 — extracted helper functions

Re-analysis: A (99/100), 0 errors, 0 warnings, 5 info
```

---

## Requirements

- [Claude Code](https://claude.ai/code)
- [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) v1.2.0+
- Node.js 18+

---

## Links

- [codopsy-ts](https://github.com/O6lvl4/codopsy-ts) — the CLI tool
- [npm package](https://www.npmjs.com/package/codopsy-ts)

## License

ISC
