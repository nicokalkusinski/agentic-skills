---
name: lunatic-test-coverage
description: Generate behavior-first tests that lock in expected behavior and make regressions hard. Start by requiring scoped targets and expected behavior, then drive coverage from realistic risks and invariants. Prefer a focused unit-heavy suite; add a small number of integration tests only where wiring, serialization, DB semantics, caching, auth, or middleware would otherwise be untested. Keep tests deterministic and repo-native, reuse the project’s runner and conventions, avoid flaky snapshots and redundant coverage, include negative cases (including fail-closed security behavior), and optionally add non-blocking “desired behavior” tests (xfail/todo/skip with a reason) to document future improvements without breaking CI.
metadata:
  version: 1.6.0
  last updated: Jan 26, 2026

  author(s):
    - Nico Kalkusinski (for Multiverse Computing)

  short-description: "Write behavior-first tests: spec-readable, unit-first, edge-complete, repo-native."
  
  versioning:
    scheme: Semantic Versioning (MAJOR.MINOR.PATCH)
    bump MAJOR When:
      - The skill’s contract/output format changes in a breaking way (existing usage no longer works without edits).
      - The skill’s scope changes materially (e.g., drops a supported runner/language or reverses key defaults).
    bump MINOR When:
      - New capabilities/guidance are added in a backwards-compatible way.
      - New optional sections/rules are added that improve outputs without breaking existing usage.
    bump PATCH When:
      - Fix typos/clarity, reorder wording, or make small non-behavioral improvements.
      - Tighten ambiguous instructions without changing expected deliverables.
---

Generate tests in a **behavioral stability** style:
- “Behavioral stability tests” specify behavior you rely on and want to keep stable (unit, integration, and e2e).
- Optional “desired behavior” tests describe improvements without breaking CI (mark as `xfail`/`todo`/skipped with a clear reason).

Do not assume any existing files, naming conventions, frameworks, or languages beyond what you can infer from the repo itself.

## Priority levels

- **Mandatory**: do this unless the developer explicitly opts out.
- **Recommended**: do this by default; skip only with a clear reason.
- **Optional**: do this only when it buys unique confidence.

## Quick workflow checklist (mandatory)

1. Ask for scope and constraints (section 0).
2. Detect the runner and conventions (section 1).
3. Build risks → invariants → a test matrix (section 2.2).
4. Choose the right test level per invariant (section 2.3).
5. Generate cases systematically (section 3).
6. Write tests in a behavior-first style (sections 4–8).
7. Run the tests and do a final self-review before finishing (section 10).

## 0 Ask the developer what to cover (mandatory)

Before writing any tests, ask the developer to specify scope. If answers are incomplete, proceed by inferring scope from the repo and state assumptions.

Ask:
- Which **files** should be covered (paths)?
- Which **modules/classes/functions** should be covered (names/symbols)?
- Which **behaviors/invariants** must be guaranteed (success + failure cases)?
- Any constraints: “no DB”, “no network”, “unit-only”, “integration required”, “must be deterministic”, etc.
- The command they expect to use in Docker/CI/local to run tests (if known).
- Any other comments or extra instructions from the developer that you should respect when writing tests.
- Preferred test strategy:
  - “Unit-heavy + thin integration layer” (default recommendation)
  - “Integration-heavy” (only when system wiring is the primary risk)
- Budget constraints:
  - Max acceptable runtime for the test suite (CI minutes)
  - Flakiness tolerance (ideally zero; ask what is acceptable)

## 1 Detect the project’s testing setup

1. Identify the test runner and conventions by scanning the repo (examples):
   - Python: `pytest.ini`, `pyproject.toml`, `tox.ini`, `conftest.py`, `requirements*.txt`
   - JS/TS: `package.json` (jest/vitest), `playwright.config.*`
   - Go: `go.mod` + `_test.go`
   - Java: `pom.xml` / `build.gradle`
2. Use the project’s existing framework **if present**. If multiple frameworks exist, pick the one that is clearly primary.
3. If the repo has **no tests at all**, create the minimal harness consistent with the stack (do not add heavy tooling).
4. Identify the project’s **test root** (where tests live). Examples:
   - Python: `tests/`, `<package>/tests/`
   - JS/TS: `test/`, `tests/`, `__tests__/`
   - Go: package directories containing `*_test.go`
   - Java: `src/test/`

### 1.1 Follow a testing pyramid (recommended)

Aim for the “pyramid” shape that senior engineers typically optimize for:
- Many fast **unit tests** (cheap, deterministic, exhaustive branching).
- Fewer **integration tests** (high-value end-to-end flows that catch wiring/state issues).
- Avoid “three copies” of the same behavior (unit + integration + legacy/duplicate file).

Default policy:
- For any single behavior/invariant, prefer to test it at **one** level.
- Add a second-level test only when it catches a different class of regression (see “Integration must earn its cost” below).

### 1.2 Monorepos and multi-runner repos (mandatory when applicable)

Some repos contain multiple packages, languages, or test runners.

Rules:
1. Detect scope boundaries first.
   - Identify which package or service owns the code under test.
   - Prefer the test runner that package uses, even if other runners exist elsewhere in the repo.

2. Follow how CI runs tests.
   - If CI runs per-package commands, mirror that structure.
   - Do not introduce a new repo-wide runner unless the repo already standardizes on one.

3. Keep changes local.
   - Add tests and small helpers within the owning package’s test root.
   - Avoid adding root-level dependencies or configs when only one package needs tests.

4. When multiple runners exist in the same package:
   - Pick the runner referenced by that package’s scripts (for example `package.json` scripts) or the runner used by existing tests in that package.
   - If still ambiguous, pick the runner invoked in CI for that package.

5. Document the command precisely.
   - Provide the exact per-package command to run the new tests, plus any required working directory.


## 2 Choose the right test level (unit vs integration)

Pick the lowest-cost test that still provides real confidence.

Prefer **unit tests** when:
- Behavior is deterministic and local (parsing, validation, formatting, branching logic).
- Dependencies can be replaced cleanly (DB calls, caches, clocks, external services).

Prefer **integration tests** when:
- Confidence depends on wiring across layers (routing → handler → DB → cache, middleware, signals/hooks, auth).
- Mocking would “mock away” the risk (content-type parsing, session/cookies, serialization, cache invalidation).

### 2.1 Integration must earn its cost (mandatory)

Before adding an integration test, answer these explicitly (in your working notes):
- What specific regression could happen that a unit test would not catch?
- Which real components must be exercised (DB transaction semantics, cache invalidation, serialization, middleware, signals/hooks, crypto verification)?
- What is the smallest end-to-end scenario that proves it?

If you cannot answer these, prefer a unit test instead.

### 2.2 Risks → invariants → test matrix (recommended)

To avoid “tests for the sake of tests”, drive coverage from *realistic risks* and *invariants*.

#### 2.2.1 Build a risk model (recommended)

Write a short list of realistic failure modes for the scoped code. Examples (adapt to the domain):
- Security: authorization bypass, privilege escalation, replay, open redirects, sensitive-data leakage.
- Correctness: wrong parsing/validation, inconsistent error codes, wrong state transitions.
- Reliability: cache/DB/service outages, timeouts, partial failures.
- Concurrency: double-submit, race conditions, duplicate consumption, inconsistent writes.
- Interop: case/encoding differences, content-type quirks, backwards compatibility.

#### 2.2.2 Define invariants (mandatory)

For each major risk, write 1–3 invariants in “must always” form. Examples:
- “A consumed one-time token can never be used again (even concurrently).”
- “When authorization cannot be verified, the system denies access (fail closed).”
- “Errors never include secrets or raw tokens.”

#### 2.2.3 Create a test matrix (recommended)

Create a small table in your working notes mapping:
- Invariant → tests that prove it → test level (unit/integration/e2e) → why that level is necessary.

Rules:
- Each invariant should have at least one test.
- Avoid two tests that fail for the same regression unless they catch different failure classes.
- Prefer unit tests by default; integration tests must earn their cost (see 2.1).

### 2.3 Test level selection rules (mandatory)

Use this section to choose the RIGHT LEVEL (unit vs integration vs e2e) for each behavior/invariant.
It defines what each level is responsible for, so you avoid duplicate coverage across levels.
After choosing the level, use section 3 to generate the concrete test cases for that behavior.

#### Unit tests should test (at minimum when applicable)
Core logic and behaviors, with exhaustive branching on meaningful decision points:
- Input validation and error mapping
  - Missing fields, wrong types, invalid formats, invalid enums
  - Structured error shape and stable error codes (if applicable)
- Branching and boundary conditions
  - Off-by-one limits, empty vs whitespace, casing, unicode normalization
  - Min, max, just under, just over for lengths, ranges, counts
- Security checks when local to the unit
  - Permission decisions, policy evaluation, scope checks
  - Fail-closed behavior when verification input is absent or invalid
- Time and expiration logic
  - At expiry, just before, just after, clock skew handling (if implemented)
- Idempotency and deduplication logic
  - Same request repeated, same idempotency key, replay inputs
- Serialization and normalization
  - Canonicalization does not widen permissions
  - Parsing never crashes on invalid input, returns safe errors
- Cache key construction and caching decisions (logic only)
  - Key stability, namespace separation, negative caching decisions
- Retry/backoff selection logic (if present)
  - No infinite loops, correct max attempts, correct classification of retryable errors
- “No secret leakage” in returned errors or logs (where testable)
  - Errors should not contain tokens, passwords, secrets, raw credentials

Preferred style:
- Parameterized tests for input families (invalid formats, whitespace, unicode, boundary values).
- Property-based tests only if the repo already uses them, or if they are lightweight and high value.

#### Integration tests should test (only when they earn their cost)
Wiring across real components where unit tests would miss regressions:
- Request parsing and framework integration
  - Content-Type parsing, JSON decoding, form parsing, headers, cookies, middleware
- Auth wiring
  - Middleware/guard integration, session/cookie parsing, signature verification wiring
  - Fail closed when verification cannot run (key missing, verifier error)
- DB and transaction semantics
  - Unique constraints, foreign keys, migrations assumptions, rollback behavior
  - Concurrency-sensitive invariants (best-effort, deterministic)
- Cache integration
  - Cache hit and miss behavior across layers
  - Invalidation paths for updates, namespace isolation
- Serialization boundaries
  - Actual API response schema, status codes, and important headers
- Background job enqueueing (if present)
  - Correct job payload and enqueue call, without running real workers unless repo already does it
- External service boundaries, using stubs not real network
  - Contract with stub server or fake client, timeouts and error mapping

If you add integration tests around a boundary that can fail, include at least one “negative integration” when it provides unique confidence (recommended):
- Dependency outage/exception yields safe errors.
- Security-relevant decisions fail closed (deny) when verification cannot be performed.

Regardless of test level, cover dependency failures when they change externally observed behavior or security posture (recommended when applicable):
- auth verifier/signature key fetch
- DB transaction/write
- cache read/write
- outbound HTTP/service call
- background job enqueue


#### E2E tests (allowed to bootstrap infrastructure, but must earn their cost)

E2E tests are OPTIONAL and should be added only when they catch a class of regression that unit + integration will not.
If e2e infrastructure does not exist, you MAY create it, but only via the Bootstrap Policy below.

##### When to add E2E
Add 1–5 e2e smoke tests only if at least one is true:
- The critical risk is cross-process or cross-layer behavior that integration cannot realistically prove
  (example: auth redirect flow, cookie/session persistence, full request chain through proxy/middleware).
- The product has a small number of “must never break” journeys that define usability or revenue.
- The system is a web app where browser behavior is part of the stable behavior surface (not just APIs).

If none apply, do NOT add e2e tests, keep the suite unit-heavy with targeted integration tests.

##### E2E Bootstrap Policy (required if infra is missing)
If you create e2e infrastructure:
1. Keep it minimal, reversible, and aligned with the repo’s stack.
   - Prefer API-level e2e over browser e2e when it gives the same confidence.
   - Use the existing language ecosystem and runner where possible.
2. Avoid heavy new dependencies unless unavoidable.
   - Only introduce Playwright/Cypress when browser behavior is truly required.
   - If browser is required, default to Playwright (headless) unless repo already uses another tool.
3. Make it hermetic and deterministic:
   - No real network, stub external services.
   - Use ephemeral ports, no hardcoded localhost ports.
   - Control time and randomness (fixed seeds).
4. Handle data and state safely:
   - Use an ephemeral DB strategy (in-memory if possible).
   - If a real DB is required, use lightweight containers ONLY if the repo already uses Docker in dev/CI,
     otherwise provide a fallback mode that skips e2e by default and documents how to enable it.
5. Keep runtime small:
   - Target < 2–5 minutes for all e2e tests combined in CI.
   - Tag e2e tests as "slow" and make it easy to run separately.
6. Update TEST INVENTORY:
   - Explain why e2e was needed and what unique regressions it catches.
   - Document the exact command to run e2e locally and in CI.

##### Default e2e scope
- Implement 1–3 top user journeys (happy path).
- Add 1 fail-closed security scenario if auth exists (unauthorized cannot access).
- Add 1 resilience scenario if critical (dependency down does not allow access, safe error).

### Snapshot tests policy
Avoid snapshot tests unless the repo already relies on them and the output is stable.
Prefer explicit assertions on key fields instead of brittle snapshots.

## 3 Exhaustiveness checklist (case generation within the chosen level)

After selecting the test level using section 2.3, use this checklist to generate the concrete cases for each function/endpoint/story at that level.

Exhaustive coverage applies only to decision points that define behaviors and invariants.
Cover every meaningful branch where the externally observed behavior can differ, including:
- validation and error mapping
- authorization and policy decisions (fail closed)
- state transitions and idempotency
- time and expiration boundaries
- parsing, normalization, and serialization decisions
- caching decisions that affect correctness or security

Non-goal:
- Do NOT chase branches in trivial wrappers, logging, framework glue, or third-party libraries.
- Do NOT add tests whose only purpose is to increase line or branch metrics.
- If a branch does not change externally observed behavior, it does not need its own test.

### Exhaustive coverage scope (recommended)
“Exhaustive branch coverage” applies to decision logic that affects behaviors and invariants, for example validation, auth/permissions, state transitions, parsing/normalization, idempotency, time/expiry, and boundary handling.
Do NOT chase exhaustive branches for trivial wrappers, logging, framework boilerplate, or third-party library behavior.

For each function/endpoint/story, cover at least:
- Success path (happy path).
- Missing required inputs (validation errors).
- Unsupported/invalid inputs (type/format/enum violations).
- Security-sensitive failures (fail closed):
  - auth/permission mismatch
  - replay protection
  - dependency outage (cache/DB/service) should not silently allow access
- Boundary and regression-prone cases:
  - whitespace, encoding, case sensitivity, normalization
  - time/expiration handling
  - idempotency (repeating requests)
  - caching behavior (cached result stability; cache invalidation)

### 3.1 Case design methods (recommended)
Generate cases systematically using these techniques (pick what applies):
- Equivalence partitions: groups of inputs that should behave the same (valid, invalid-format, missing).
- Boundary values: null/none, empty collections, zero, negatives, capacity limits, min, max, just-under, just-over (lengths, limits, expiry).
- Decision tables: combinations where behavior changes based on multiple inputs/flags (role × state × feature flag).
- State transitions: workflows and lifecycles (issued → used → revoked), including replay.
- Pairwise coverage: when many options exist, cover pairs instead of all combinations.

Prefer parameterization/table-driven tests for these case families.

## 4 Behavior-first style: how tests should read

### 4.1 File header: what this module covers (recommended)

At the top of **each** test file/module, add a short description block that answers:
- What this test module is for (scope/purpose).
- What it covers (behaviors/invariants/endpoints/functions).
- What it explicitly does **not** cover (non-goals, out-of-scope behaviors, covered elsewhere).

Keep this block brief and scannable. Prefer 3–8 bullet lines.

Template (adapt to the test runner’s conventions):

```text
# Purpose: <1 sentence>
# Covers:
# - <behavior/invariant/area>
# - <behavior/invariant/area>
# Does not cover:
# - <explicit non-goal / covered by other layer>
# - <explicit non-goal / covered by other layer>
```

### 4.2 One-line intent comment
Start each test with a plain-English comment describing the behavior/invariant:
- `# Missing required fields should return a structured error.`
- `# On success, returns a redirect including code and preserves state.`
- `# If the cache backend fails, fail closed (deny) rather than allow.`
- `# Current behavior: <behavior>; Desired behavior: <better behavior>.`

You MAY add short inline comments only when they clarify a non-obvious setup step (for example: “freeze time at expiry boundary”, “simulate verifier failure”), but do not add extra narrative blocks.
Parameterized/table-driven tests still begin with a single intent line, and each case should have a short case label/ID.

### 4.3 Arrange–Act–Assert, kept tight
Keep each test short and scannable:
- Arrange: setup inputs and fakes
- Act: call the unit/endpoint
- Assert: verify status/result + key side-effects

If a test goes long due to repeated setup, extract a helper/fixture.

### 4.4 Assertions: what to verify (mandatory)
Assertions must prove the behavior/invariant, not just “it failed”.

When testing API/handler-like code:
- On success: assert status/result plus the minimum stable fields that define the behavior (do not snapshot entire payloads).
- On error: assert status + machine-readable error code/type (if present) + error shape (required fields) + that secrets are not present.
- If side-effects are part of the behavior: assert the side-effect (DB write, cache invalidation, job enqueued). Otherwise, avoid over-asserting internals.

Avoid asserting on full error strings unless the repo treats them as stable API.

## 5 Modularity rules (helpers + fixtures)

Make repetition cheap:
- Add tiny helpers at the top of the file (local to that test file), e.g.:
  - `_json(resp)` / `_body(resp)` / `_status(resp)` depending on stack
  - `issue_token(...)`, `create_user(...)`, `make_request(...)` for repeated flows
- Use fixtures/factories for repeated setup:
  - users/resources/clients/config objects
  - temporary files, in-memory stores
  - “autouse” cleanup fixtures when the system caches aggressively

Rules:
- Helpers must be small, named for intent, and used by multiple tests.
- Tests must not share mutable global state.
- Tests must pass in any order and in parallel (unless the repo explicitly forbids it).

### 5.1 Folder organization (recommended)

Keep tests modular **and** organized into folders that match the system’s shape.

Rules:
- Create one folder per feature/module/behavior area under the repo’s **test root**.
- Put only closely related tests in the same folder.
- Inside a folder, file names should be specific and avoid redundant prefixes when the folder already provides context.

Example (Python/pytest-style naming):
- If you have multiple integration tests currently named like `test_oauth_integration_*`, place them under:
  - `tests/test_oauth_integration/`
  - and split by topic, e.g. `test_authorize.py`, `test_refresh_rotation.py`, `test_revoke_introspect_userinfo.py`.

If the repo already uses a different convention (e.g. `__tests__/oauth/integration/`), follow it.

### 5.2 Avoid duplicate tests (mandatory)

Define “duplicate” as: two tests that would fail for the same underlying regression.

Rules:
- Do not duplicate a basic validation behavior (e.g., “missing required fields returns invalid_request”) at both unit and integration levels unless the integration test is specifically about cross-layer wiring of that validation.
- Do not keep legacy tests that re-check what newer suites already cover, unless they provide unique coverage.
- Do not keep tests with no meaningful assertions (e.g., string concatenation, asserting on unrelated details, or only asserting “status_code is 400” without checking error type/shape when that matters).

When you find duplicates, keep the cheaper, more deterministic test unless the integration version provides unique wiring/state coverage.

## 6 Test doubles (recommended)

General:
- Prefer **fakes/stubs/in-memory implementations** over deep mocks when possible.
- Replace dependencies at the seam actually used by the code under test (dependency injection, interface/trait boundary, module-level singleton, service locator, etc.).
- Keep doubles local to a test (fixture/setup/teardown), and assert only the minimal interactions that define the behavior.

Avoid:
- Mocking the unit you are trying to test.
- Over-asserting implementation details that cause brittle tests.

## 7 Isolation and determinism (mandatory)

Make tests stable:
- Control time (freeze/patch clocks) for expiration logic.
- Clear or namespace caches between tests when caching exists.
- Use temporary directories/files and clean them up.
- Avoid real network calls; stub them unless the repo is explicitly end-to-end.

### 7.1 Quality bar: what seniors will push back on

Avoid these common “looks like coverage but isn’t” patterns:
- Skipped tests with no plan to re-enable (prefer deleting or converting into a clear `todo/xfail` spec with a reason).
- Tests that only assert “it errors” without asserting *which* error and *why* (when the error type is part of the stable behavior/API).
- Tests that are effectively no-ops (assertions that always pass, asserting on unrelated logs, etc.).
- Integration tests that primarily duplicate unit validation (they add runtime/flake risk without extra confidence).
- Tests that are not tied to any explicit risk/invariant (add the invariant or remove the test).

## 8 “Desired behavior” tests policy (optional non-blocking specs)

If you want to describe a future improvement without failing CI:
- Mark the test as expected-to-fail using the idiom of the repo’s test runner:
  - pytest: `@pytest.mark.xfail(strict=True, reason="Desired: ...")`
  - jest/vitest: `test.todo(...)` or `test.skip(...)` with a reason
  - others: equivalent “pending/ignored” mechanism
- The test should read like a spec and include a short reason explaining what must change.

Do not use xfail/skip to hide flaky tests.

## 9 Output requirements (what you deliver)

Deliver:
- New/updated test files implementing the behaviors/invariants above.
- Minimal changes outside tests (only add utilities if the repo’s conventions require it).
- The exact command to run the new tests (and run it unless the developer opts out).

## 10 Final self-review checklist (mandatory)

Before finishing, double-check:
- Scope is explicit (what you tested, what you did not, and any assumptions).
- Tests run with the repo’s runner/conventions, and you provide the exact command you ran.
- Tests are deterministic (no real network, controlled time/randomness, isolated state).
- Each test maps to an explicit behavior/invariant (no metric-chasing filler).
- No redundant coverage (duplicates removed unless they catch different regressions).
- Negative/failure behavior is covered where it matters (especially security: fail closed).

If you cannot run tests in this environment, say exactly why and still provide the command the developer should run.
