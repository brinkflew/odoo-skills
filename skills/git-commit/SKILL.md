---
name: git-commit-message-format
description: >-
  Defines git commits: header [TAG][TASK] module: summary (whole line ≤80 characters), body explaining why.
  Use when drafting, amending, or rewording commits; building messages from staged files or diffs; or git commit wording help.
alwaysApply: false
---

# Commit message format

## Header line (required)

Single line, exactly this shape (spaces as shown):

```text
[TAG][TASK] module: Short but meaningful description
```

**Hard rule:** the entire first line is **at most 80 characters**, including brackets, spaces, and the colon after `module`.

If the line is too long, shorten the description first. NEVER shorten or abbreviate `[TASK]`.

## `[TAG]`

Pick one tag that matches the change:

| TAG | Typical use |
|-----|-------------|
| `[ADD]` | Adding new modules |
| `[DOC]` | Documentation only |
| `[FIX]` | Bug fix or regression fix |
| `[I18N]` | Changes in translation files |
| `[IMP]` | New behavior, new feature, improvement of existing feature |
| `[LINT]` | Linting passes |
| `[MOV]` | Moving files: use git move and do not change content of moved file |
| `[PERF]` | Performance patches |
| `[REF]` | Refactor without intended behavior change |
| `[REL]` | Release commits and commits limited to version bump |
| `[REM]` | Removing resources: removing dead code, removing views, removing modules, … |
| `[REV]` | Reverting commits: if a commit causes issues or is not wanted reverting it is done using this tag |

## `[TASK]`

Put the ticket or issue id in brackets, e.g. `[123]`. Take it from the user, the branch name, or the tracking link they provide.

When there is **no** task id, use **`[NOTASK]`** so the header shape stays consistent and length stays predictable.

## `module:`

Use the primary affected Odoo addon directory name (e.g. `sale`, `account`), then a colon and a space before the summary.

If several modules change, pick the main one, or the one the user names as primary.

## Body (required)

- Leave **one blank line** after the header.
- Be **verbose** when useful; multiple paragraphs are fine.
- Focus on **why** the change was made (motivation, constraints, tradeoffs, risk), not only **what** changed—the diff already shows what.

## Workflow

1. Inspect `git diff` / staged paths / the user’s summary of intent.
2. Choose `[TAG]`, `[TASK]`, and `module`.
3. Draft the header line; **count characters** on line 1.
4. If over 80 characters, shorten the summary.
5. Write the body explaining why.

## Examples

**Example 1**

```text
[FIX][NOTASK] sale: round refund taxes per line

Refunds reused invoice line aggregates that were computed with different
rounding order than POS expects. Aligning per-line rounding avoids one-cent
mismatches when reconciling with the payment terminal.
```

**Example 2**

```text
[ADD][123456] account: add intra-community VAT helper

We need a single place to derive EU VAT for EC sales so customs reports stay
consistent between invoicing and declared totals. Centralizing avoids copying
domain fragments across reports.
```
