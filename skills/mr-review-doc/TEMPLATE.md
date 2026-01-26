# <Short Title>

Branch: `<branch-name>`  
Date: `DD-MM-YYYY`  
Related:
- Issue: <link>
- MR: <link>

## 1. Summary

- Behavior: <what user/test now observes> @`<entry point>` or `@<test path>`
- Scope: <code path(s) affected> @`<file>:<symbol>`
- Mechanism: <what changed mechanically> @`<file>:<symbol>`
- Risk: <compat/behavior caveat, if any> @`<env var>` or `@<file>:<symbol>`

## 2. Problem / Context

### Issue 1: <Issue title>

- Problem: <one sentence: what breaks, in plain English> @`<observed output>`
- Repro:
  - command:
    ```bash
    <copy/paste>
    ```
- Observed:
  - output:
    ```text
    <copied 1–3 lines>
    ```
- Why: <one sentence: high-level why (env precedence, stale image, wrong default, etc.)> @`<file>` or `@<command>`
- Impact: <who/what breaks> @`<suite>` or `@<entry point>`

#### Details

- Entry point: <UI path / endpoint / job / command> @`<endpoint/UI path>` or `@<command>`
- Prereqs: <env/config/data state, if any> @`<env var>` or `@<file>`
- Expected: <expected value/output> @`<test name>` or `@<expected output>`
- Actual: <actual value/output> @`<observed output>`
- Constraint: <config/precedence/runtime rule that explains surprising behavior> @`<file>` or `@<command>`

### Issue 2: <Issue title> (optional)

- <repeat Issue 1 shape>

## 3. Root Cause / Findings

### Issue 1: <Issue title>

- Cause: <one-liner root cause summary> @`<file>:<symbol>`
- Consequence: <why the repro fails; tie back to Section 2> @`<test path>:<test name>` or `@<observed output>`

#### Details

- Cause: <concrete incorrect behavior + exact location> @`<file>:<symbol>`
- Consequence: <what the incorrect behavior causes> @`<test path>:<test name>` or `@<observed output>`
- Constraint: <runtime/config factor, if relevant> @`<file>` or `@<command>`
- Finding: <confirmed invariant/calculation, if relevant> @`<observed output>`

### Issue 2: <Issue title> (optional)

- <repeat Issue 1 shape>

## 4. Requirements / Acceptance Criteria

- AC: <checkable Given/When/Then> @`<test path>:<test name>`
- AC: When running `<command>`, expect `<output>` @`<command>`

## 5. Decision Log

### Decision 1: <Decision title>

- Options for @`<file>:<symbol>`
  - Option 1: <Option A> @`<file>:<symbol>` (or `@runtime`/`@command`)
  - Option 2: <Option B> @`<file>:<symbol>` (or `@runtime`/`@command`)
- Chosen (Option X): <Option X> @`<file>:<symbol>`
- Why: <references a specific AC bullet> @`AC#` or `@<AC bullet>`
- Trade-off: <limitation/risk introduced> @`<file>:<symbol>` or `@<runtime>`
- Non-goal: <explicitly out of scope> @`<issue/test path>`

### Decision 2: <Decision title> (optional)

- <repeat Decision 1 shape>

## 6. Implementation Notes

- Files touched: @`git diff --name-only`
  - `<path/to/file>`
  - `<path/to/file>`
- Symbols changed: @`<path>:<symbol>`
  - `<path>:<symbol>`
  - `<path>:<symbol>`
- Operational: <env vars / settings / DB alias / deploy notes> @`<env var>` or `@<file>`
- Data impact: <migrations/backfills/none> @`<migration/model>`
- Logging: <new/changed log lines, if any> @`<log text>` or `@<path>:<symbol>`

## 7. How To Validate

### Issue 1: <Issue title>

- Validate (before):
  - command:
    ```bash
    <copy/paste>
    ```
- Observed (before):
  - output:
    ```text
    <copied failure line(s)>
    ```
- Validate (after):
  - command:
    ```bash
    <copy/paste>
    ```
- Expected (after):
  - output:
    ```text
    <copied pass summary>
    ```

### Issue 2: <Issue title> (optional)

- <repeat Issue 1 shape>

### General

- Failure modes:
  - <common failure + what it looks like> @`<output>`

## 8. Rollback / Safety

- Rollback: <revert strategy> @`git revert …` or `@<paths>`
- Disable: <feature flag/env toggle, if any> @`<env var>` or `@<settings path>`
- Data impact: <migrations/backfills/none> @`<migration/model>`
- Risk: <what to monitor after rollback> @`<log/output>`

## 9. Open Questions / Follow-ups

- Follow-up: <actionable next step> @`<file>:<symbol>` or `@<issue link>`
- Non-goal: <explicitly deferred behavior> @`<test/issue>`
