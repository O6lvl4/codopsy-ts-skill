<h1 align="center">codopsy-ts-skill</h1>

<p align="center">
  <strong>Claude Code plugin for <a href="https://github.com/O6lvl4/codopsy-ts">codopsy-ts</a></strong>
</p>

<p align="center">
  Analyze code quality and auto-fix issues, right from Claude Code.
</p>

---

## What it does

This plugin adds the `/codopsy` skill to Claude Code. It runs [codopsy-ts](https://github.com/O6lvl4/codopsy-ts) on your codebase, interprets the results, and can automatically fix the issues it finds.

**Analyze** — cyclomatic & cognitive complexity, 13 lint rules, per-file breakdown

**Fix** — Claude reads the analysis, understands each issue, and applies minimal fixes

---

## Install

```
/plugin install codopsy@O6lvl4/codopsy-ts-skill
```

> Requires [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) (`npm install -g codopsy-ts`)

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
```

---

## What happens

### Report mode (default)

1. Runs `codopsy-ts analyze` with `--verbose`
2. Presents a summary: file count, error/warning/info counts, hotspots
3. Lists each issue with file, line, rule, and message

### Fix mode (`--fix`)

1. Runs analysis
2. Fixes issues in priority order:
   - `no-var` → replace with `let`/`const`
   - `eqeqeq` → replace `==` with `===`
   - `prefer-const` → change `let` to `const`
   - `no-param-reassign` → introduce local variable
   - `max-params` → extract options object
   - `max-depth` → extract helpers, early returns
   - `max-complexity` / `max-cognitive-complexity` → break down functions
   - `max-lines` → extract into modules
   - `no-nested-ternary` → replace with if/else
   - `no-any` → add type annotations
3. Re-runs analysis to verify improvements
4. Reports before/after comparison

---

## Example

```
> /codopsy

=== Codopsy Analysis ===

Files: 12 | Errors: 0 | Warnings: 3 | Info: 5

| File             | Line | Rule           | Message                                    |
|------------------|------|----------------|--------------------------------------------|
| src/parser.ts    | 42   | max-complexity | Function "parse" has complexity 14 (max 10) |
| src/utils.ts     | 8    | no-var         | Unexpected var, use let or const            |
| src/handler.ts   | 15   | eqeqeq         | Expected "===" but found "=="              |

Hotspot: src/parser.ts (complexity: 14, cognitive: 11)

> /codopsy --fix

Fixed 3 issues:
  ✓ src/utils.ts:8 — var → const
  ✓ src/handler.ts:15 — == → ===
  ✓ src/parser.ts:42 — extracted helper functions

Re-analysis: 0 errors, 0 warnings, 5 info
```

---

## Requirements

- [Claude Code](https://claude.ai/code)
- [codopsy-ts](https://www.npmjs.com/package/codopsy-ts) v1.0.0+
- Node.js 18+

---

## Links

- [codopsy-ts](https://github.com/O6lvl4/codopsy-ts) — the CLI tool
- [npm package](https://www.npmjs.com/package/codopsy-ts)

## License

ISC
