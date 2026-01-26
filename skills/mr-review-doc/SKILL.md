---
name: mr-review-doc
description: Create a high-signal, low-noise MR review document to justify rationale behind your implementation.
metadata:
  Version: 1.0.0
  Last Updated: Jan 26, 2026

  Author(s):
    - Nico Kalkusinski (for Multiverse Computing)
  
  Versioning:
    Scheme: Semantic Versioning (MAJOR.MINOR.PATCH)
    Bump MAJOR When:
      - The skill’s contract/output format changes in a breaking way (existing usage no longer works without edits).
      - The skill’s scope changes materially (e.g., drops a supported runner/language or reverses key defaults).
    Bump MINOR When:
      - New capabilities/guidance are added in a backwards-compatible way.
      - New optional sections/rules are added that improve outputs without breaking existing usage.
    Bump PATCH When:
      - Fix typos/clarity, reorder wording, or make small non-behavioral improvements.
      - Tighten ambiguous instructions without changing expected deliverables.
---

Create **one** MR context document in `[root]/docs/mr-docs` that explains the changes in your current MR/branch.

Goal: a reader (engineer or LLM) can reproduce the problem + verify the fix using only this doc and the repo.

## 0. Inputs (must read)

- `skills/mr-review-doc/README.md`
- `skills/mr-review-doc/TEMPLATE.md`

Keep the same numbered headings `1..9` from `TEMPLATE.md` unless there is a strong reason not to.

## 0.1 Filename (mandatory)

- Path: `[root]/docs/mr-docs/[branch-name]_[DDMMYYYY].md`
- Example: `[root]/docs/mr-docs/319-permission-granted-email-not-sent-when-adding-new-function-to-existing-user_16012026.md`

## 0.2 Writing rules (mandatory)

- Write for a reader with **zero context**.
- Prefer facts over narrative. Avoid vague language (“improved/robust/clean”).
- Use present tense and active voice (“Computes…”, “Reads…”, “Fails…”).
- One claim per bullet.
- Prefer short top-level bullets; move details into nested bullets.

### Quick checklist (recommended)

- If the MR has more than one problem, define separate issues (see “Issue blocks”).
- Make every issue independently reproducible (repro + observed output).
- Use `@...` anchors for jump-to-code and evidence.

### Structure rules (mandatory)

- Use headings `## 1..9` exactly as in `TEMPLATE.md`.
- You may use `###` subheadings inside sections to create visual blocks (recommended).
- Content is primarily bullets, but you may also use:
  - label lines like `Symptom:`, `Expected:`, `Actual:` when they help scanning
  - fenced code blocks for commands, payloads, and output (recommended for long artifacts)

### Issue blocks (mandatory when >1 issue)

If the MR addresses multiple distinct issues, define issues once and reuse them consistently:

- Use explicit numbering in headings: `### Issue 1: ...`, `### Issue 2: ...`.
- In sections `## 2`, `## 3`, and `## 7`, group content by the same issue numbers and keep each issue’s bullets together.
- Avoid interleaving unrelated causes/repros across issues (one issue = one block).

### Artifact blocks (recommended)

If a command/output is longer than one short line, use a nested bullet + fenced block instead of a single long inline backtick:

- `Repro:` (or `Validate:`) parent bullet
  - command:
    ```bash
    <copy/paste>
    ```
- `Observed:` parent bullet
  - output:
    ```text
    <copied lines>
    ```

### Evidence / anchor rule (mandatory)

Anchors exist to make the doc verifiable and navigable.

- Use anchors when they add evidence or a jump target.
- Prefer at most one anchor per bullet.
- If a bullet already includes the full path/command in backticks, do not add a second anchor that repeats it.

Anchor formats:

- `@<endpoint/UI path>` (example: `@/admin/user_management/userfunction/add/`)
- `@<command>` (example: `@docker-compose run --rm admin_dashboard pytest -k create_token`)
- `@<file>:<symbol>` (example: `@sso_user_management/use_cases/manage_user.py:ManageUserUseCase.create_token`)
- `@<observed output/log line>` (example: `@AssertionError: 2025-01-04 != 2024-10-11`)

Required evidence:

- Section 2 includes at least one runnable repro command and at least one copied `Observed:` output line (per issue if multiple issues exist).
- Section 3 includes at least one concrete code pointer to the relevant path/symbol (`@<file>:<symbol>`).
- Section 7 includes before/after commands and copied before/after output lines (per issue if multiple issues exist).

### Bullet prefixes (mandatory)

Start each bullet with one of these prefixes (pick the closest match).

- Parenthetical qualifiers are allowed, e.g. `Validate (before):`, `Validate (after):`, `Observed (before):`, `Expected (after):`.

Core: `Behavior:` `Scope:` `Mechanism:` `Risk:` `Problem:` `Repro:` `Observed:` `Why:` `Impact:` `Cause:` `Consequence:` `Fix:` `Validate:` `AC:`

Optional (use when relevant): `Entry point:` `Prereqs:` `Expected:` `Actual:` `Constraint:` `Finding:` `Failure modes:` `Files touched:` `Symbols changed:` `Rollback:` `Disable:` `Data impact:` `Logging:` `Follow-up:` `Non-goal:` `Options:` `Option:` `Chosen:` `Trade-off:`

## 0.3 Pre-work checklist (collect before writing)

- Branch name: `git branch --show-current`
- Changed files: `git diff --name-only <base>...HEAD`
- Failing repro command **and** copied output lines (at least one concrete failure line)
- Passing validation command **and** copied output summary (e.g., `X passed`)

## 0.4 Header block (mandatory)

Add this header block at the top of the created MR doc (immediately under the `# <Short Title>` line, before `## 1. Summary`).

Branch: `<branch-name>`  
Date: `DD-MM-YYYY`  
Related:
- Issue: <link>
- MR: <link>

## 1. Summary (exact shape)

Write **3–4 bullets total**, in this order:

- `Behavior (What):` what a user/test now observes (include where it applies) @`<entry point>` or `@<test path>`
- `Scope (Where):` what code path(s) this change affects @`<file>:<symbol>`
- `Mechanism (How):` what changed mechanically (one line) @`<file>:<symbol>`
- `Risk:` compatibility or behavior caveat (if any) @`<env var>` or `@<file>:<symbol>`

### Summary readability rules (mandatory)

- Behavior (What): state the before/after delta in one clause (e.g. “Previously X; now Y”) and name the entry point @`<command>` or `@<observed output>`
- Scope (Where): name the surface area that changes and avoid describing runtime mechanics (details belong in `## 2` Details or `## 6` Operational) @`<file>:<symbol>`
- Mechanism (How): describe intent-level changes and avoid tool flags and long lists (move flag/config detail to `## 5` / `## 6`) @`<file>:<symbol>`
- Risk: include exactly one caveat; move additional caveats to `## 8` (operational) or `## 9` (follow-ups) @`<env var>` or `@<command>`

## 2. Problem / Context (exact shape)

For each issue, start with a 5-line short path (in this order):

- `Problem:` one sentence describing what breaks, in plain English (no deep mechanics) @`<observed output>`
- `Repro:` one copy/paste repro command @`<command>`
- `Observed:` 1–3 copied lines of real output (no paraphrase) @`<error/log/output>`
- `Why:` one sentence “high-level why” (env precedence, stale image, wrong default, etc.) @`<file>` or `@<command>`
- `Impact:` what breaks and for whom (tests/runtime/users) @`<suite>` or `@<entry point>`

If the MR has multiple issues, use `### Issue 1`, `### Issue 2`, … and repeat the same 5-line short path per issue.

If needed, add a small “Details (optional)” block under the issue with any of:

- `Entry point:` UI path / endpoint / job / command @`<endpoint/UI path>` or `@<command>`
- `Prereqs:` env/config/data state @`<env var>` or `@<file>`
- `Expected:` / `Actual:` when it clarifies the failure @`<test name>` or `@<observed output>`
- `Constraint:` runtime/config rule that explains surprising behavior (Docker `.env` override, settings module, DB alias, image-baked vs mounted code) @`<file>` or `@<command>`

Keep details skippable: the issue should still be understandable by reading only the 5 short-path bullets.

## 3. Root Cause / Findings (exact shape)

Write **3–10 bullets total**.

- Group by issue if multiple issues exist (`Issue 1`, `Issue 2`, …).
- Each issue must contain at least one “cause → consequence” pair.
- Each `Cause:` bullet includes a code pointer (`@<file>:<symbol>`) or a copied observed line.

Recommended per-issue bullets:

- `Cause:` concrete incorrect behavior + exact location @`<file>:<symbol>`
- `Consequence:` why the repro fails (tie back to Section 2) @`<test path>:<test name>` or `@<observed output>`
- `Constraint:` non-obvious runtime/config factor (optional) @`<file>` or `@<command>`
- `Finding:` confirmed invariant/calculation (optional) @`<observed output>`

## 4. Requirements / Acceptance Criteria (exact shape)

Write **checkable** bullets (prefer “Given/When/Then” phrasing). If the issue/ticket includes AC, copy it verbatim.

- `AC:` “When running `<command>`, expect `<output>`” @`<command>`
- `AC:` “Given `<setup>`, when `<action>`, then `<result>`” @`<test path>:<test name>`

If none exist:

- `Finding:` derived acceptance criteria from failing tests

## 5. Decision Log (strict per-decision template)

For each decision, use an explicit numbered heading (avoid `D1/D2` abbreviations):

- `### Decision 1: <title>`
- `### Decision 2: <title>`

Under each decision heading, include:

- `Options for:` @`<file>:<symbol>` (or `@N/A` if purely conceptual)
  - `Option 1:` <option A> @`<file>:<symbol>` (or `@runtime`/`@command`)
  - `Option 2:` <option B> @`<file>:<symbol>` (or `@runtime`/`@command`)
  - `Option ...n:` <option B> @`<file>:<symbol>` (or `@runtime`/`@command`)
- `Chosen (Option N):` <option X> @`<file>:<symbol>`
- `Why:` references a specific AC bullet (by number or quoted text) @`AC#` or `@<AC bullet>`
- `Trade-off:` limitation/risk introduced @`<file>:<symbol>` or `@<runtime>`
- `Non-goal:` what is explicitly not addressed in this MR @`<issue/test path>`

## 6. Implementation Notes (exact shape)

Write **6–12 bullets total**, with concrete pointers:

- `Files touched:` introduce the list (optionally include `git diff --name-only`) @`<command>`
  - `<path>`
  - `<path>`
- `Symbols changed:` introduce the list @`<path>:<symbol>`
  - `<path>:<symbol>`
  - `<path>:<symbol>`
- `Removed/moved:` removed/moved logic (if any) @`<path>:<symbol>`
- `Operational:` env vars / settings module / DB alias / migrations @`<path>` or `@<env var>`
- `Data impact:` persisted data changes (or “none”) @`<model>` or `@<migration>`
- `Logging:` new/changed log lines (if relevant) @`<log text>` or `@<path>:<symbol>`

## 7. How To Validate (must include before/after)

If this MR has a single issue, include:

- `Validate (before):` failing command (copy/paste) @`<command>`
- `Observed (before):` copied failure line(s) @`<output>`
- `Validate (after):` passing command (copy/paste) @`<command>`
- `Expected (after):` copied pass summary (e.g., `X passed`) @`<output>`

If this MR has multiple issues, group validation by the same issue headings (`Issue 1`, `Issue 2`, …) and include the above bullets per issue.

Also include:

- `Failure modes:` 2–4 common failures + what they look like (expired creds, docker daemon perms, missing env) @`<output>`

## 8. Rollback / Safety (exact shape)

Write **3–6 bullets total**:

- `Rollback:` safe revert strategy (files/commit) @`git revert …` or `@<paths>`
- `Disable:` feature flag/env toggle (if any) @`<env var>` or `@<settings path>`
- `Data impact:` migrations/backfills required or “none” @`<migration/model>`
- `Risk:` what to monitor after rollback @`<log/output>`

## 9. Open Questions / Follow-ups (strict)

Only include real follow-ups (not speculation). Each item must be actionable.

- `Follow-up:` what to do + where + why @`<file>:<symbol>` or `@<issue link>`
- `Non-goal:` explicitly deferred behavior not shipped in this MR @`<test/issue>`

## Definition of done (mandatory checklist)

- Doc exists at `[root]/docs/mr-docs/[branch-name]_[DDMMYYYY].md`
- Headings match `TEMPLATE.md` (1..9) `skills/mr-review-doc/TEMPLATE.md`
- Section 2 includes `Repro`, `Expected`, `Actual`, `Observed`
- Section 3 includes at least one `@<file>:<symbol>` root-cause pointer
- Section 7 includes before/after commands + copied output lines
