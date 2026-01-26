# Docs Notes

This folder stores short decision/context documents created per MR/branch to preserve:

- Problem statements and root cause analysis
- Rationale behind decisions and trade-offs
- Operational notes that do not belong in inline code comments or docstrings
- Validation steps and rollback notes
- Follow-ups and open questions

## Naming

Create one document per relevant change/issue, named as:

`[branch-name]_[daystamp].md`

Where `daystamp` is `DDMMYYYY`.

Example:

`319-permission-granted-email-not-sent-when-adding-new-function-to-existing-user_15012026.md`

## Template

Use `TEMPLATE.md` as the starting point for consistent structure and headings.

## Writing Guidelines (Style + Information Density)

These docs are intended to be read months later by engineers and LLM tools with no prior context.

### Style Rules

- Prefer facts over narrative: state what happened, where, and why.
- Every non-trivial statement should include at least one of: entry point (URL/endpoint), file path + symbol name, example payload, or observed log output.
- Use short bullets; avoid paragraphs unless they add concrete details that cannot be bulleted.
- Avoid vague language ("improved", "robust", "clean"); replace with measurable statements (e.g., "one email per user per transaction").
- Separate behavior from mechanism:
  - Behavior: what the system does (user-visible).
  - Mechanism: how it is implemented (signals, transactions, batching).

### Section Expectations

- **Problem / Context**: include a minimal repro (exact path + input) and expected vs actual.
- **Root Cause / Findings**: point to exact missing/incorrect link(s) with file paths + symbols.
- **Decision Log**: list options considered, chosen option, and trade-offs tied to acceptance criteria.
- **Implementation Notes**: list the key files/symbols touched and any operational requirements (env vars, credentials, migrations).
- **How To Validate**: include copy/paste commands (curl) and expected log lines / outcomes.