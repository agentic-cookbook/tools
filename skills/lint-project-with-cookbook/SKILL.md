---
name: lint-project-with-cookbook
version: "1.1.0"
description: "Lint an implementation against cookbook guidelines or a specific recipe. Combines guideline review and recipe conformance checking."
argument-hint: "[guidelines|recipe] [recipe-path] [implementation-path] [--version]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git diff *), Bash(git log *), Bash(wc *), Agent, AskUserQuestion
context: fork
---

# Lint with Cookbook v1.1.0

## Startup

**First action**: If `$ARGUMENTS` is `--version`, print `lint-project-with-cookbook v1.1.0` and stop — do not run the skill.

Otherwise, print `lint-project-with-cookbook v1.1.0` as the first line of output, then proceed.

**Version check**: Read `${CLAUDE_SKILL_DIR}/SKILL.md` from disk and extract the `version:` field from frontmatter. If it differs from this skill's version (1.1.0), print:

> ⚠ This skill is running v1.1.0 but vA.B.C is installed. Restart the session to use the latest version.

Continue running — do not stop.

## Overview

Lint an implementation against the agentic cookbook. Two modes:

- **guidelines** — Review against the full guideline checklist, engineering principles, and recipe requirements. Produces a structured pass/warn/fail report.
- **recipe** — Compare against a specific recipe's conformance requirements line by line. Produces a coverage report with PASS/FAIL/PARTIAL per requirement.

## Usage

```
/lint-project-with-cookbook guidelines cookbook/recipes/ui/component/empty-state.md ../my-app/Sources/EmptyState/
/lint-project-with-cookbook recipe empty-state ../my-app/Sources/EmptyState/
/lint-project-with-cookbook recipe empty-state
/lint-project-with-cookbook
```

## Step 1: Determine Mode

Parse `$ARGUMENTS`:

- **Starts with `guidelines`**: Guidelines mode. Remaining arguments are `<recipe-path> [implementation-path]`.
- **Starts with `recipe`**: Recipe mode. Remaining arguments are `<recipe-path> [implementation-path]`.
- **Empty or unrecognized first word**: Ask the user using `AskUserQuestion`:

```
What would you like to lint?

1. **guidelines** — Review against the full cookbook guideline checklist, engineering principles, and recipe requirements
2. **recipe** — Compare against a specific recipe's conformance requirements (requirements, states, appearance, test vectors, logging)
```

After the mode is determined, proceed to the corresponding section below.

---

## Mode: Guidelines

Review a generated implementation against the agentic cookbook guidelines, recipes, and engineering principles.

### Inputs

After the mode keyword, arguments are: `<recipe-path> [implementation-path]`

1. **Recipe path**: Path to the component or recipe (e.g., `cookbook/recipes/ui/component/empty-state.md` or just the component name like `empty-state`)
2. **Implementation path**: Path to the implementation directory or files (defaults to current working directory if not provided)

If either argument is missing, ask the user to provide it.

### Fallbacks

| Situation | Action |
|-----------|--------|
| Recipe path doesn't exist | Search `cookbook/recipes/` recursively for a matching filename. If not found, stop with error. |
| Implementation directory is empty | Stop with error: "No implementation files found at {path}" |
| `cookbook/guidelines/` not found | Stop with error: "Cannot locate cookbook guidelines directory." |
| `cookbook/principles/` not found | Skip principles review, note in report |
| A review agent fails | Report partial results from completed agents, note the failure |
| More than 50 implementation files | Prioritize files matching the recipe's component name and related types. Note truncation in report. |

### Phase 1: Read the recipe and implementation

1. Read the recipe from the recipe path. If the path is a bare name (no path separator), search `cookbook/recipes/` recursively for `<name>.md`.
2. Read `CLAUDE.md` and `cookbook/conventions.md` for repo-level context.
3. Read the guideline index at `cookbook/guidelines/INDEX.md` and then the core guidelines file `cookbook/guidelines/general.md`.
4. Read engineering principles from `cookbook/principles/` (all files).
5. Read the guideline checklist from `${CLAUDE_SKILL_DIR}/references/guideline-checklist.md`.
6. Read implementation files from the implementation path (or current directory). Use Glob to find source files (*.swift, *.kt, *.tsx, *.ts). Cap at 50 files — if more exist, prioritize files whose names match types referenced in the recipe.

### Phase 2: Systematic review

Launch up to 3 review agents in parallel. Each agent reviews the items assigned to it from `${CLAUDE_SKILL_DIR}/references/guideline-checklist.md`:

**Agent 1 — UI Quality & Behavior**
Categories: UI Quality & Behavior

**Agent 2 — Quality Assurance**
Categories: Quality Assurance, Linting

**Agent 3 — Accessibility, i18n, Privacy, Engineering Principles**
Categories: Accessibility, i18n, Privacy, Engineering Principles

### Phase 3: Compile report

Combine all agent findings into a single structured report.

### Output Format

```
# Cookbook Guideline Lint: [Component Name]

Recipe: [path to recipe]
Implementation: [path to implementation]
Platform: [detected platform]
Date: [today]
Files reviewed: [count]

## Summary
[1-2 sentence overall assessment]

## Results

| Guideline | Status | Finding |
|-----------|--------|---------|
| Native controls | ✅/⚠️/❌ | [detail] |
| Open-source preference | ✅/⚠️/❌/N/A | [detail] |
| Responsiveness & progress | ✅/⚠️/❌ | [detail] |
| Main thread / concurrency | ✅/⚠️/❌ | [detail] |
| Unit testing | ✅/⚠️/❌ | [detail] |
| Design decisions | ✅/⚠️/❌ | [detail] |
| Atomic commits | ✅/⚠️/❌ | [detail] |
| Post-generation verification | ✅/⚠️/❌ | [detail] |
| Logging | ✅/⚠️/❌ | [detail] |
| Deep linking | ✅/⚠️/❌/N/A | [detail] |
| Scriptability | ✅/⚠️/❌/N/A | [detail] |
| Accessibility | ✅/⚠️/❌ | [detail] |
| Localization | ✅/⚠️/❌ | [detail] |
| RTL support | ✅/⚠️/❌ | [detail] |
| Accessibility display options | ✅/⚠️/❌ | [detail] |
| Privacy & security | ✅/⚠️/❌ | [detail] |
| Feature flags | ✅/⚠️/❌ | [detail] |
| Analytics | ✅/⚠️/❌/N/A | [detail] |
| A/B testing | ✅/⚠️/❌/N/A | [detail] |
| Debug mode | ✅/⚠️/❌ | [detail] |
| ... | | |

## Critical Issues
[Any ❌ findings that must be fixed]

## Warnings
[Any ⚠️ findings that should be addressed]

## Recommendations
[Suggestions for improvement]

## Limitations
[Any skipped checks due to missing files, agent failures, or file count caps]
```

Status meanings:
- ✅ **Pass**: Fully compliant
- ⚠️ **Warning**: Partially compliant or could be improved
- ❌ **Fail**: Non-compliant, must be fixed
- N/A: Rule does not apply to this component

### Important Notes (Guidelines Mode)

- Be thorough but fair — don't flag things that genuinely don't apply
- Check the actual code, not just file presence. A test file with no assertions is ⚠️ not ✅
- For the logging guideline, compare recipe log messages character-by-character against implementation
- For the localization guideline, grep for hardcoded strings in the implementation
- For the privacy guideline, grep for PII patterns in log statements
- Reference the cookbook guidelines and reference docs by name when flagging compliance issues

---

## Mode: Recipe

Compare an implementation against a specific recipe's conformance requirements, line by line.

### Inputs

After the mode keyword, arguments are: `<recipe-path> [implementation-path]`

1. **Recipe path**: Path to the recipe, or a bare name (no `/`) — search `../agentic-cookbook/cookbook/recipes/` recursively for `<name>.md`. If not found, print an error listing the search paths tried and stop.
2. **Implementation path**: Path to the implementation directory. If empty, use the current working directory. If the implementation path contains no source files (*.swift, *.kt, *.tsx, *.ts, *.py, *.cs), print an error and stop.

### Step R1: Read the Recipe

Read the recipe file completely. Extract every section:

- **Behavioral requirements** (named, kebab-case) with their RFC 2119 keywords (MUST, SHOULD, MAY)
- **States table** — every row
- **Appearance values** — dimensions, colors, fonts, spacing
- **Conformance test vectors** — every row
- **Logging messages** — exact strings
- **Edge cases** — every item
- **Accessibility requirements** — every item

Print a header:

```
=== RECIPE LINT: <recipe-name> ===
Recipe: <path>
Implementation: <path>
Requirements: <count>
Test vectors: <count>
```

### Step R2: Read the Implementation

Read implementation files from the implementation path. Use Glob to find source files matching `*.swift`, `*.kt`, `*.tsx`, `*.ts`, `*.py`, `*.cs`. Cap at 50 files — if more exist, prioritize files whose names match types referenced in the recipe. Note any truncation. Read each file.

### Step R3: Compare

For each recipe section, compare against the implementation:

**Requirements**: For each requirement, determine if the implementation satisfies the described behavior. Mark PASS, FAIL, or PARTIAL.

**States**: For each state in the states table, check if the implementation handles it.

**Appearance**: Check if specified values (dimensions, colors, fonts, spacing) match.

**Test vectors**: Check if tests exist that correspond to each test vector row.

**Logging**: Compare logging messages character by character against the recipe's logging section.

**Edge cases**: Check if each edge case is handled.

**Accessibility**: Check if accessibility requirements are implemented.

### Step R4: Print the Report

```
--- REQUIREMENTS ---
[PASS] ordered-list: <description>
[FAIL] scroll-to-bottom: <description>
       -> <what's wrong>
[PARTIAL] user-input-focus: <description>
       -> <what's implemented vs missing>

--- STATES ---
[PASS] Default state: implemented
[FAIL] Error state: not found in implementation

--- APPEARANCE ---
[PASS] Corner radius: 12pt matches recipe
[FAIL] Padding: implementation uses 8pt, recipe specifies 12pt

--- TEST VECTORS ---
[PASS] comp-001: test exists at <test-file>
[FAIL] comp-002: no corresponding test found

--- LOGGING ---
[PASS] "ComponentName: appeared" — matches exactly
[FAIL] "ComponentName: action completed" — not found in implementation

--- EDGE CASES ---
[PASS] Empty data: handled at <file:line>
[FAIL] RTL layout: not addressed

--- ACCESSIBILITY ---
[PASS] VoiceOver label: present
[FAIL] Keyboard navigation: not implemented

=== CONFORMANCE SUMMARY ===
Pass: <n> | Fail: <n> | Partial: <n>
Coverage: <pass/(pass+fail+partial)>%

Top issues:
1. [FAIL] <most critical>
2. [FAIL] <next>
```

---

## Guards

- **Read-only** — do not modify any files.
- **Console only** — print the report to stdout; do not write report files.
- **50-file cap** — if more than 50 implementation files match, prioritize by recipe relevance and note the truncation.
