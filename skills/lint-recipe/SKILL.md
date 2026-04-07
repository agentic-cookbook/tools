---
name: lint-recipe
version: "1.1.0"
description: "Lint a cookbook recipe file against the template, conventions, and completeness checks. Triggers on 'lint this recipe', 'check my recipe', 'review this recipe', or /lint-recipe."
argument-hint: "[path-or-name]"
allowed-tools: Read, Glob, Grep, Bash(wc *), AskUserQuestion
context: fork
model: sonnet
---

# Lint Recipe v1.1.0

Validate a cookbook recipe file for structural completeness, frontmatter correctness, requirement quality, test vector coverage, and convention compliance.

## Startup

lint-recipe v1.1.0

**Version check**: Read `${CLAUDE_SKILL_DIR}/SKILL.md` from disk and extract the `version:` field from frontmatter. Compare to this skill's version (1.1.0). If they differ, print:

> ⚠ This skill is running v1.1.0 but vA.B.C is installed. Restart the session to use the latest version.

Then continue running.

If `$ARGUMENTS` is `--version`, respond with exactly:
> lint-recipe v1.1.0

Then stop.

## Argument Resolution

Resolve `$ARGUMENTS` to a recipe `.md` file path using this flow:

### If `$ARGUMENTS` is provided:

1. **Path check**: If `$ARGUMENTS` contains `/` or ends with `.md`, treat it as a file path.
   - If the file exists, use it.
   - If not, print "File not found: <path>" and stop.

2. **Search string**: Otherwise, treat `$ARGUMENTS` as a search string. Use Glob to find `cookbook/recipes/**/*.md` (excluding `_template.md` and `INDEX.md`). Filter to files whose name contains the search string (case-insensitive).
   - **1 match** → Use it. Print: "Found: <path>"
   - **Multiple matches** → Show up to 4 matches with AskUserQuestion. Each option label is the filename, description is the relative path. The user picks one.
   - **0 matches** → Print "No recipes matching '<string>'" and stop.

### If `$ARGUMENTS` is empty:

1. **Session context**: Check if a recipe file was recently created, edited, or read in this conversation. If so, offer it:
   - Use AskUserQuestion: "Lint <filename>?" with options "Yes" and "No, choose another".
   - If "Yes", use that file.

2. **Prompt**: If no recent recipe or user declined, use AskUserQuestion: "Which recipe? Enter a name or path." The user's response re-enters the search string flow above.

## References

Before running checks, read:
1. `cookbook/recipes/_template.md` — the canonical recipe template
2. `cookbook/conventions.md` — format and frontmatter rules

If either is missing, warn but continue with what's available.

## Checks

Read the target recipe file completely. Run all checks below. For each check, emit one of:

```
[PASS] <ID>: <title>
[WARN] <ID>: <title>
       -> <recommendation>
[FAIL] <ID>: <title>
       -> <what's wrong>
```

---

### FRONTMATTER (F01–F10)

| ID | Check | PASS | FAIL |
|----|-------|------|------|
| F01 | YAML frontmatter block present | `---` delimiters found | No frontmatter |
| F02 | `id` field is a valid UUID | UUID v4 format | Missing or malformed |
| F03 | `domain` matches file path | Domain derived from path matches `domain:` value | Mismatch |
| F04 | `type` is `recipe` | Exactly `recipe` | Wrong or missing |
| F05 | `version` is valid semver | Matches `X.Y.Z` pattern | Missing or malformed — do NOT flag the version field's existence |
| F06 | Required fields present | `id`, `title`, `domain`, `type`, `version`, `status`, `language`, `created`, `modified`, `author`, `copyright`, `license`, `summary`, `platforms` all present | Any missing |
| F07 | `license` is `MIT` | Exactly `MIT` | Other value |
| F08 | `copyright` includes year and name | Contains 4-digit year and a name | Missing or malformed |
| F09 | `created` and `modified` are valid dates | ISO 8601 date format | Invalid format |
| F10 | `summary` is non-empty | At least 10 characters | Empty or too short |

---

### SECTIONS (S01–S04)

| ID | Check | PASS | FAIL/WARN |
|----|-------|------|-----------|
| S01 | All 17 template sections present | Every `## Section` heading from the template appears | Missing sections — list them |
| S02 | No empty sections | Every section has content beyond its heading | WARN for sections with only "Not applicable" without a reason; FAIL for truly empty |
| S03 | No TODO/placeholder text | No `TODO`, `TBD`, `FIXME`, `placeholder`, `fill in` | Found placeholder text |
| S04 | Change History section present and populated | Has at least one row in the table | Missing or empty |

---

### REQUIREMENTS (R01–R05)

| ID | Check | PASS | FAIL |
|----|-------|------|------|
| R01 | At least one behavioral requirement | `## Behavioral Requirements` has at least one `**kebab-case**:` entry | No requirements |
| R02 | Requirements use kebab-case names | All requirement names match `[a-z0-9]+(-[a-z0-9]+)*` | Names with underscores, spaces, or uppercase |
| R03 | Requirements use RFC 2119 keywords | Each requirement contains MUST, SHOULD, or MAY (uppercase) | Missing keyword |
| R04 | No duplicate requirement names | All names are unique within the file | Duplicates found |
| R05 | Requirements are specific and testable | WARN if any requirement is vague (e.g., "should work well", "must be good") | Subjective language detected |

---

### TEST VECTORS (T01–T03)

| ID | Check | PASS | FAIL |
|----|-------|------|------|
| T01 | Test vectors section exists and has entries | `## Conformance Test Vectors` table has at least one row | Empty or missing |
| T02 | Test vector IDs are unique | All IDs in the table are distinct | Duplicates |
| T03 | Test vectors reference requirements | Each vector's Requirements column references a valid requirement name from the Behavioral Requirements section | Orphan references |

---

### COMPLETENESS (K01–K07)

| ID | Check | PASS | WARN/FAIL |
|----|-------|------|-----------|
| K01 | States table populated | At least 2 states in the States table | WARN if only 1, FAIL if none |
| K02 | Appearance values specified | At least 3 concrete values (dimensions, colors, fonts, spacing) in Appearance section | WARN if fewer than 3 |
| K03 | Edge cases listed | At least 2 edge cases in the Edge Cases section | WARN if fewer |
| K04 | Logging events defined | At least 1 log event in the Logging section, OR "Not applicable" with reason | FAIL if section empty with no explanation |
| K05 | Accessibility requirements present | At least 1 item in Accessibility section, OR "Not applicable" with reason | FAIL if empty |
| K06 | Compliance table present | `## Compliance` section has at least one row | WARN if missing |
| K07 | Design Decisions section populated | At least 1 design decision documented | WARN if empty |

---

## Summary

After all checks, print:

```
=== SUMMARY ===
Pass: <n> | Warn: <n> | Fail: <n>
```

If there are any FAIL items, print:

```
⚠ Recipe has <n> failing checks. Fix these before considering the recipe complete.
```

If all checks pass (zero FAIL, zero or more WARN):

```
✓ Recipe passes all checks.
```

## Guards

- **Read-only** — do not modify the recipe file or any other file.
- **No version flagging** — never flag the `version` frontmatter field as a warning or error. It is maintained by convention per `rules/skill-versioning.md`.
- **Console only** — print the report to stdout; do not write report files.
