---
name: high-signal-comments
description: Add skimmable, plain-English comments and docs to code you changed on the current branch, focusing on intent and constraints, not narration.
metadata:
  author: Nico Kalkusinski
  last updated: Feb 3, 2026
  version: "1.1.0"
  argument-hint: <base-branch|optional>
---

Add high-signal comments/docs to **only** the files changed on the current branch, compared to the base branch. Comments must be skimmable, plain-English, and easy to understand at a glance. Avoid over-explaining. Do not narrate obvious code.

Goal: engineers, reviewers, and LLM tools with zero context can read the diff and quickly understand intent, constraints, edge cases, and behavior guarantees.

## 0. Inputs (must read)

- `AGENTS.md` (commenting philosophy)
- Repo style and doc conventions (JSDoc/TSDoc, Python docstrings, Go doc comments, etc.)

## 0.1 Scope Discovery (MANDATORY START)

**If the user has not already provided a specific list of files or functions, your first response MUST be to ask the following questions:**

### Required inputs (developer must specify)
1. Which **files** should be covered (paths)?
2. Which **modules/classes/functions** should be covered (names/symbols)?

If either is missing, ask follow-ups. If the developer cannot provide them, infer likely targets from the repo and clearly label that as an assumption.

## 0.2 Writing rules (mandatory)

Write comments that are fast to scan.

- Use plain English.
- Prefer clarity over cleverness.
- Keep comments short, usually one sentence, rarely two.
- Use “because” only when it adds real value.
- Avoid vague words (“improved”, “robust”, “clean”).
- Prefer measurable statements and guarantees:
  - “one email per user per transaction”
  - “cache TTL is 10 minutes”
  - “returns inclusive start, exclusive end”
  - “rejects requests without X header”
- Do not restate types, names, or the obvious “what”.
- If code can be made clearer with a better name, do that instead of commenting.

Audience rules:

- Write for engineers and LLM tools with zero context.
- Assume the reader has the repo, but not the mental model.

## 0.3 What to comment (mandatory)

Add docs/comments only where a reader would ask “why” or “what are the rules”.

Priority order:

1. Public surface changes:
   - exported functions, classes, modules
   - routes, handlers, jobs, CLI commands
   - Document behavior, inputs meaning, outputs meaning, side effects, failure modes
2. Rules and constraints in changed logic:
   - business rules and invariants
   - auth and permissions
   - retries, backoff, rate limits
   - caching rules and invalidation
   - pagination, batching, idempotency
   - ordering and concurrency constraints
   - external API quirks and backward compatibility
3. Risk hotspots:
   - places likely to be “simplified” later and broken
   - tricky edge cases that are not obvious from the code

Do NOT add comments for:

- simple control flow
- obvious calculations
- lines already explained by good naming

## 0.4 Comment formats (mandatory)

Use the smallest format that works.

### Inline comments (preferred for rules)

Use for “rule” statements that a reviewer can scan quickly.

Good:

```ts
// One email per user per transaction, prevents duplicate sends on retries.
```

Bad:

```ts
// This function is robust and improved.
```

### Doc comments for public APIs (only when needed)

Keep them short and rule-driven.

JSDoc/TSDoc example:

```ts
/**
 - Sends a verification email.
 - Guarantees: at most one send per user per transaction.
 */
```

Python example:

```py
def send_verification_email(user_id: str) -> None:
    """Send a verification email.

    Guarantee: sends at most once per user per transaction.
    """
```

If a doc would only repeat the function name or types, do not write it.

## 0.5 Fix bad comments in changed areas (mandatory)

In changed regions, if comments are:

- vague
- narration
- outdated
- misleading

Then remove them or rewrite them into a short rule or constraint.

Avoid comment-only churn outside changed context.

## 0.6 Output rules (mandatory)

Your output is a single, review-ready patch:

- Provide a unified diff
- Touch only the selected changed files
- No unrelated formatting changes
- No refactors unless needed to make the docs truthful and easy to maintain

If there is nothing worth commenting, output:

- `No comment changes needed`
- Include the list of files checked

## 1. Execution procedure (mandatory)

For each changed file:

1. Identify what needs explanation:
   - what rule is enforced
   - what constraint exists
   - what guarantee is intended
   - what failure mode matters
2. Add minimal docs/comments:
   - one line per rule or guarantee
   - keep wording simple and concrete
3. Remove or rewrite vague/narration comments in the changed regions.
4. Re-check the diff:
   - every new comment answers “what is the rule” or “why is this needed”
   - no new comment repeats what the code already says

## 2. Quality gates (mandatory)

- Comments match the actual behavior
- Comments are skimmable (short, plain English)
- No vague claims, use measurable statements where possible
- No secrets or sensitive data in comments
- No lint or formatting violations introduced

## 3. Acceptance criteria (exact shape)

- AC: Only files returned by specified by the user are edited.
- AC: Every changed file with non-trivial or public-facing changes has skimmable rule/constraint docs where needed
- AC: No new comment narrates obvious code, each new comment states a rule, constraint, guarantee, or failure mode @`<path>`
- AC: No unrelated refactors or formatting churn @`git diff`

## Definition of done (mandatory checklist)

- Scope of change chosen correctly
- Added skimmable docs/comments at rules and public surfaces
- Removed or rewrote vague/narration comments in changed regions
- Final output is a single unified diff patch touching only changed files