---
name: lint-compliance
version: "1.1.0"
description: "Evaluate a cookbook recipe or guideline against applicable compliance checks. Triggers on 'check compliance', 'compliance check', 'lint compliance', or /lint-compliance."
argument-hint: "<path-to-recipe-or-guideline> [--version]"
allowed-tools: Read, Glob, Grep, AskUserQuestion
context: fork
model: sonnet
---

# Lint Compliance v1.1.0

## Startup

**First action**: If `$ARGUMENTS` is `--version`, print `lint-compliance v1.1.0` and stop.

Otherwise, print `lint-compliance v1.1.0` as the first line of output, then proceed.

**Version check**: Read `${CLAUDE_SKILL_DIR}/SKILL.md` from disk and extract the `version:` field from frontmatter. If it differs from this skill's version (1.1.0), print:

> ⚠ This skill is running v1.1.0 but vA.B.C is installed. Restart the session to use the latest version.

Continue running — do not stop.

## Overview

Evaluates a cookbook recipe or guideline against the compliance framework defined in `cookbook/compliance/`. For each applicable compliance category, determines which checks apply to the target, evaluates them, and produces a structured PASS/WARN/FAIL report.

This skill serves two modes:
- **Audit mode**: For items that already have a `## Compliance` section — verifies completeness and accuracy
- **Guidance mode**: For items without a `## Compliance` section — identifies applicable checks and recommends what should be evaluated

## Guards

- **Read-only**: Do NOT modify any files
- **Console only**: Print the report to stdout
- **Fail fast**: If the target path is invalid, stop with a clear error

## Step 1: Resolve the Target

Resolve `$ARGUMENTS` to a recipe or guideline `.md` file.

### If `$ARGUMENTS` is provided:

1. **Path check**: If `$ARGUMENTS` contains `/` or ends with `.md`, treat it as a file path.
   - If the file exists, use it directly.
   - If the path points to a directory, look for the primary `.md` file in it.
   - If not found, print "File not found: <path>" and stop.

2. **Search string**: Otherwise, treat `$ARGUMENTS` as a search string. Use Glob to find `cookbook/recipes/**/*.md` and `cookbook/guidelines/**/*.md` (excluding `_template.md` and `INDEX.md`). Filter to files whose name contains the search string (case-insensitive).
   - **1 match** → Use it. Print: "Found: <path>"
   - **Multiple matches** → Show up to 4 matches with AskUserQuestion. Each option label is the filename, description is the relative path.
   - **0 matches** → Print "No recipes or guidelines matching '<string>'" and stop.

### If `$ARGUMENTS` is empty:

1. **Session context**: Check if a recipe or guideline file was recently created, edited, or read in this conversation. If so, offer it with AskUserQuestion: "Check compliance for <filename>?" with options "Yes" and "No, choose another".

2. **Prompt**: If no recent file or user declined, use AskUserQuestion: "Which recipe or guideline? Enter a name or path." The user's response re-enters the search string flow above.

## Step 2: Locate Compliance Definitions

Locate the compliance directory. Check in order:
1. `cookbook/compliance/` (running from within the cookbook repo)
2. `../agentic-cookbook/cookbook/compliance/` (running from a consuming project)

If not found, stop with: "Cannot locate compliance definitions. Expected `cookbook/compliance/` or `../agentic-cookbook/cookbook/compliance/`."

Read `cookbook/compliance/INDEX.md` to get the list of all compliance categories.

## Step 3: Read the Target

Read the target file in full. Extract:
1. **Frontmatter**: type, platforms, tags, domain
2. **Content sections**: which sections are present (Behavioral Requirements, Accessibility, Privacy, Logging, etc.)
3. **Existing Compliance section**: if present, parse the check table (check name, status, category)

## Step 4: Determine Applicable Categories

For each of the 10 compliance categories, read the category file and evaluate applicability based on the target's content:

| Category | Applicable when... |
|----------|-------------------|
| Security | Target handles authentication, user input, credentials, tokens, network requests, or stores data |
| User Safety | Target involves user-generated content, content display, or social features |
| Performance | Target has a UI, animations, data loading, or resource management |
| Best Practices | Always applicable — testing, linting, and code quality apply to everything |
| Access Patterns | Target involves network communication, API calls, or data synchronization |
| Accessibility | Target has a user interface or defines visual/interactive elements |
| Privacy & Data | Target collects, stores, transmits, or processes personal or sensitive data |
| Platform Compliance | Target targets specific platforms (check `platforms:` frontmatter) |
| Reliability | Target handles errors, manages state, or communicates over networks |
| Internationalization | Target contains user-visible text, dates, numbers, or locale-sensitive content |

For each applicable category, read the full compliance file and identify which individual checks within it apply to this specific target.

## Step 5: Evaluate Checks

For each applicable check, evaluate against the target content:

- **passed**: The target's content addresses this concern adequately
- **failed**: The target's content contradicts this check or explicitly omits a required behavior
- **partial**: The target partially addresses this concern but has gaps
- **missing**: The check applies but the target doesn't address it at all (no relevant section or mention)

### Evaluation criteria:

For a check to **pass**, the target must:
- Have a section or requirement that addresses the check's concern, OR
- The check's concern is inherently satisfied by the target's design (e.g., a read-only display component inherently passes input-sanitization)

For a check to **fail**, the target must:
- Explicitly define behavior that contradicts the check (e.g., logging PII when the check says MUST NOT)

For a check to be **missing**, the target must:
- Have no mention of the concern at all, AND the check is applicable based on the target's content

## Step 6: Detect Mode and Report

### Audit mode (target HAS a `## Compliance` section)

Compare the existing compliance table against your evaluation:

1. **Missing checks**: Applicable checks not listed in the existing table
2. **Extra checks**: Checks listed as applicable that shouldn't be (false applicability)
3. **Status disagreements**: Checks where your evaluation differs from the listed status
4. **Missing categories**: Entire applicable categories not represented

### Guidance mode (target has NO `## Compliance` section)

Generate a recommended compliance evaluation listing all applicable checks.

## Step 7: Print the Report

```
=== COMPLIANCE: <target filename> ===
Type: <recipe|guideline>
Mode: <audit|guidance>
Domain: <domain from frontmatter>
```

For each applicable category, print:

```
--- <CATEGORY NAME> ---
[PASS] check-name
       <one-line justification — what in the target satisfies this>

[FAIL] check-name
       <what's wrong or missing>

[WARN] check-name (missing)
       <this check applies but the target doesn't address it>

[SKIP] check-name
       <why this check doesn't apply to this target>
```

In audit mode, also print discrepancies:

```
--- AUDIT DISCREPANCIES ---
[MISSING] check-name — applicable but not in Compliance section
[DISAGREE] check-name — listed as "passed" but evaluation found "partial"
[EXTRA] check-name — listed but not applicable to this target
```

### Summary

```
=== SUMMARY ===
Categories evaluated: N/10
Checks: N passed, N failed, N partial, N missing
Audit discrepancies: N (audit mode only)
```

### Recommendations

After the summary, print prioritized recommendations:

1. **Failed checks** — highest priority, these contradict compliance requirements
2. **Missing checks** — the target should address these concerns
3. **Partial checks** — gaps that should be filled
4. **Audit discrepancies** — the Compliance section should be updated

For guidance mode, also print the recommended `## Compliance` section table that could be added to the target:

```
Recommended Compliance section:

| Check | Status | Category |
|-------|--------|----------|
| [check-name](agentic-cookbook://compliance/category#check-name) | passed | Category |
...
```
