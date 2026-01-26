---
name: exhaustive-coverage-tests
description: Write high-signal, contract-style tests with clear human-readable intent, modular helpers/fixtures, and exhaustive branch coverage—portable across repos.
metadata:
  Version: 1.4.0
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

Generate tests in a **contract-test style**:
- “Contract tests” specify behavior we rely on and want to keep stable.
- Optional “desired behavior” tests can describe improvements without breaking CI (mark as `xfail`/`todo`/skipped with a clear reason).

Do not assume any existing files, naming conventions, or frameworks beyond what you can infer from the repo itself.

## 0 Ask the developer what to cover (required)

Before writing any tests, ask the developer to specify scope. Do not proceed until you have clear answers.

Ask:
- Which **files** should be covered (paths)?
- Which **modules/classes/functions** should be covered (names/symbols)?
- Which **behaviors/contracts** must be guaranteed (success + failure cases)?
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

### 1.1 Follow a testing pyramid (required)

Aim for the “pyramid” shape that senior engineers typically optimize for:
- Many fast **unit tests** (cheap, deterministic, exhaustive branching).
- Fewer **integration tests** (high-value end-to-end flows that catch wiring/state issues).
- Avoid “three copies” of the same contract (unit + integration + legacy/duplicate file).

Default policy:
- For any single behavior/contract, prefer to test it at **one** level.
- Add a second-level test only when it catches a different class of regression (see “Integration must earn its cost” below).

## 2 Choose the right test level (unit vs integration)

Pick the lowest-cost test that still provides real confidence.

Prefer **unit tests** when:
- Behavior is deterministic and local (parsing, validation, formatting, branching logic).
- Dependencies can be replaced cleanly (DB calls, caches, clocks, external services).

Prefer **integration tests** when:
- Confidence depends on wiring across layers (routing → handler → DB → cache, middleware, signals/hooks, auth).
- Mocking would “mock away” the risk (content-type parsing, session/cookies, serialization, cache invalidation).

### 2.1 Integration must earn its cost (required)

Before adding an integration test, answer these explicitly (in your working notes):
- What specific regression could happen that a unit test would not catch?
- Which real components must be exercised (DB transaction semantics, cache invalidation, serialization, middleware, signals/hooks, crypto verification)?
- What is the smallest end-to-end scenario that proves it?

If you cannot answer these, prefer a unit test instead.

### 2.2 Risk model → invariants → test matrix (required)

To avoid “tests for the sake of tests”, drive coverage from *realistic risks* and *invariants*.

#### 2.2.1 Build a risk model (required)

Write a short list of realistic failure modes for the scoped code. Examples (adapt to the domain):
- Security: authorization bypass, privilege escalation, replay, open redirects, sensitive-data leakage.
- Correctness: wrong parsing/validation, inconsistent error codes, wrong state transitions.
- Reliability: cache/DB/service outages, timeouts, partial failures.
- Concurrency: double-submit, race conditions, duplicate consumption, inconsistent writes.
- Interop: case/encoding differences, content-type quirks, backwards compatibility.

#### 2.2.2 Define invariants (required)

For each major risk, write 1–3 invariants in “must always” form. Examples:
- “A consumed one-time token can never be used again (even concurrently).”
- “When authorization cannot be verified, the system denies access (fail closed).”
- “Errors never include secrets or raw tokens.”

#### 2.2.3 Create a test matrix (required)

Create a small table in your working notes mapping:
- Invariant → tests that prove it → test level (unit/integration/e2e) → why that level is necessary.

Rules:
- Each invariant should have at least one test.
- Avoid two tests that fail for the same regression unless they catch different failure classes.
- Prefer unit tests by default; integration tests must earn their cost (see 2.1).

## 3 Exhaustiveness checklist (what to cover)

For each function/endpoint/story, cover at least:
- Success path (happy contract).
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

### 3.1 Use parametrization and property tests for input families (recommended)

Use parametrization/table-driven tests for families of similar cases.

When testing parsers/normalizers/validators, prefer “one test + many cases” over many near-identical tests:
- Use table-driven/parameterized tests for known edge-case families (whitespace, encoding, case, empty inputs).
- If the repo/tooling supports it, consider property-based tests for invariants such as:
  - “Parsing then re-encoding is stable/canonical.”
  - “Invalid inputs never crash; they return a safe error.”
  - “Normalization does not widen permissions.”

Do not introduce heavy new dependencies unless the repo already uses them.

## 4 Contract style: how tests should read

### 4.1 File header: what this module covers (required)

At the top of **each** test file/module, add a short description block that answers:
- What this test module is for (scope/purpose).
- What it covers (contracts/invariants/endpoints/functions).
- What it explicitly does **not** cover (non-goals, out-of-scope behaviors, covered elsewhere).

Keep this block brief and scannable. Prefer 3–8 bullet lines.

Template (adapt to the test runner’s conventions):

```text
# Purpose: <1 sentence>
# Covers:
# - <contract/invariant/area>
# - <contract/invariant/area>
# Does not cover:
# - <explicit non-goal / covered by other layer>
# - <explicit non-goal / covered by other layer>
```

### 4.2 One-line intent comment
Start each test with exactly **one** plain-English comment describing the contract:
- `# Missing required fields should return a structured error.`
- `# On success, returns a redirect including code and preserves state.`
- `# If the cache backend fails, fail closed (deny) rather than allow.`
- `# Current behavior: <behavior>; Desired behavior: <better behavior>.`

Avoid long docstrings inside tests unless your framework requires them.

### 4.3 Arrange–Act–Assert, kept tight
Keep each test short and scannable:
- Arrange: setup inputs and fakes
- Act: call the unit/endpoint
- Assert: verify status/result + key side-effects

If a test goes long due to repeated setup, extract a helper/fixture.

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

### 5.1 Folder organization (required)

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

### 5.2 Avoid duplicate tests (required)

Define “duplicate” as: two tests that would fail for the same underlying regression.

Rules:
- Do not duplicate a basic validation contract (e.g., “missing required fields returns invalid_request”) at both unit and integration levels unless the integration test is specifically about cross-layer wiring of that validation.
- Do not keep legacy tests that re-check what newer suites already cover, unless they provide unique coverage.
- Do not keep tests with no meaningful assertions (e.g., string concatenation, asserting on unrelated details, or only asserting “status_code is 400” without checking error type/shape when that matters).

When you find duplicates, keep the cheaper, more deterministic test unless the integration version provides unique wiring/state coverage.

## 6 Mocking / patching rules (critical for correctness)

General:
- Prefer **fakes/stubs** over deep mocks when possible.
- Keep mocking local to a test (context manager / fixture), and assert the minimal set of calls that matter.

For languages with import-based patching (e.g., Python):
- Patch the symbol at the **lookup location used by the code under test** (where it’s imported/used), not where it’s originally defined.

Avoid:
- Mocking the function you are trying to test.
- Over-asserting implementation details that will cause brittle tests.

## 7 Isolation and determinism

Make tests stable:
- Control time (freeze/patch clocks) for expiration logic.
- Clear or namespace caches between tests when caching exists.
- Use temporary directories/files and clean them up.
- Avoid real network calls; stub them unless the repo is explicitly end-to-end.

### 7.1 Quality bar: what seniors will push back on

Avoid these common “looks like coverage but isn’t” patterns:
- Skipped tests with no plan to re-enable (prefer deleting or converting into a clear `todo/xfail` spec with a reason).
- Tests that only assert “it errors” without asserting *which* error and *why* (when the error type is part of the contract).
- Tests that are effectively no-ops (assertions that always pass, asserting on unrelated logs, etc.).
- Integration tests that primarily duplicate unit validation (they add runtime/flake risk without extra confidence).
- Tests that are not tied to any explicit risk/invariant (add the invariant or remove the test).

## 8 “Desired behavior” tests policy (non-blocking specs)

If you want to describe a future improvement without failing CI:
- Mark the test as expected-to-fail using the idiom of the repo’s test runner:
  - pytest: `@pytest.mark.xfail(strict=True, reason="Desired: ...")`
  - jest/vitest: `test.todo(...)` or `test.skip(...)` with a reason
  - others: equivalent “pending/ignored” mechanism
- The test should read like a spec and include a short reason explaining what must change.

Do not use xfail/skip to hide flaky tests.

## 9 Output requirements (what you deliver)

Deliver:
- New/updated test files implementing the contracts above.
- Minimal changes outside tests (only add utilities if the repo’s conventions require it).

Also include:
- A short “test matrix” in your working notes (not in code) mapping behaviors → tests, so coverage is intentional.
- The exact command(s) to run the new tests in this repo (the repo’s standard test command when available).

## 10 Minimal pytest starter (use only when pytest is the project’s runner)

If the repo uses pytest, follow this skeleton style (adapt names to the domain):

```python
# Contract tests: behavior we rely on and want to keep stable.
# Desired-behavior tests (if any) are marked xfail/todo so they don't block CI.

import pytest

def _json(resp):
    return resp.json()  # or json.loads(resp.content) depending on stack

def test_example():
    # Missing required fields should return a structured error.
    ...
```
