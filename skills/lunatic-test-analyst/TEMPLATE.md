# <Short Title>

Date: `DD-MM-YYYY`

Assumed Scope:
- Files: `<path/to/file>`, `<path/to/file>`
- Symbols: `<symbol1>`, `<symbol2>`

Runner/Framework Detected: `<pytest|jest|vitest|...>` (evidence: `<config file>`)

Repo Conventions Detected:
- Layout: `<test folder layout>`
- Naming: `<test file naming pattern>`
- Utilities: `<existing fixtures/helpers>`

---

## File: `<path/to/file>`

### A. Architectural Footprint

- Purpose: `<1-3 line description>` @`<file>:<symbol>`
- Public surface area: `<exports/routes/handlers>` @`<file>:<symbol>`
- Dependencies touched: `<db/fs/cache/network/framework>` @`<file>:<symbol>`
- Integration triggers: `<External State|Framework Plumbing|Boundary Serialization|Side Effects|Protocol Assumptions>` @`<file>:<symbol>`

### B. Proposed Tests

#### UNIT

1. **Behavior/Invariant:** `<what behavior is being tested>`
   - Test name: `<test_function_name>`
   - Risk category: `<Security|Concurrency|Reliability|...>`
   - Level: UNIT
   - Status: TODO
   - Tags: `[happy_path|negative|fail_closed, validation|authn|authz|...]`
   - Rationale: `<why this level, what regression it catches>`
   - Setup notes: `<what must be real vs mocked>`

2. **Behavior/Invariant:** `<additional unit test>`
   - Test name: `<test_function_name>`
   - Risk category: `<...>`
   - Level: UNIT
   - Status: TODO
   - Tags: `[negative, validation]`
   - Rationale: `<...>`
   - Setup notes: `<...>`

#### INTEGRATION

3. **Behavior/Invariant:** `<integration test behavior>`
   - Test name: `<test_function_name>`
   - Risk category: `<...>`
   - Level: INTEGRATION
   - Status: TODO
   - Tags: `[happy_path, side_effects, business_rule]`
   - Rationale: `<why integration is needed (wiring that can't be mocked)>`
   - Setup notes: `<real DB|cache|queue required>`

#### E2E

4. **Behavior/Invariant:** `<e2e smoke test behavior>`
   - Test name: `<test_function_name>`
   - Risk category: `<...>`
   - Level: E2E
   - Status: TODO
   - Tags: `[happy_path, critical_flow]`
   - Rationale: `<critical business flow verification>`
   - Setup notes: `<full stack required>`

### C. Integration Justification (only if INTEGRATION items exist)

For each INTEGRATION item:

- What a unit test would miss: `<describe the wiring/behavior mocks would hide>` @`<file>:<symbol>`
- What real component must be exercised: `<DB|cache|queue|auth middleware|etc.>` @`<file>:<symbol>`
- Minimal harness recommendation: `<repo-native test DB|in-memory|testcontainers|etc.>`

### D. Coverage Sanity Check (Mandatory)

- Count by Level:
  - UNIT: N
  - INTEGRATION: M
  - E2E: P
- Count by Outcome:
  - happy_path: X
  - negative: Y
  - fail_closed: Z
- Intentional gaps:
  - `<description of gap>`: `<reason/risk accepted>`

---

## File: `<path/to/file>` (optional - repeat for each file)

### A. Architectural Footprint

- Purpose: `<...>` @`<file>:<symbol>`
- Public surface area: `<...>` @`<file>:<symbol>`
- Dependencies touched: `<...>` @`<file>:<symbol>`
- Integration triggers: `<...>` @`<file>:<symbol>`

### B. Proposed Tests

#### UNIT

5. **Behavior/Invariant:** `<...>`
   - Test name: `<test_function_name>`
   - Risk category: `<...>`
   - Level: UNIT
   - Status: TODO
   - Tags: `[...]`
   - Rationale: `<...>`
   - Setup notes: `<...>`

#### INTEGRATION

6. **Behavior/Invariant:** `<...>`
   - Test name: `<test_function_name>`
   - Risk category: `<...>`
   - Level: INTEGRATION
   - Status: TODO
   - Tags: `[...]`
   - Rationale: `<...>`
   - Setup notes: `<...>`

### C. Integration Justification

- What a unit test would miss: `<...>` @`<file>:<symbol>`
- What real component must be exercised: `<...>`
- Minimal harness recommendation: `<...>`

### D. Coverage Sanity Check

- Count by Level: UNIT: N, INTEGRATION: M, E2E: P
- Count by Outcome: happy_path: X, negative: Y, fail_closed: Z
- Intentional gaps: `<none|description with reason>`

---

## Summary

| Level        | Count | Key Risks Covered                          |
|--------------|-------|--------------------------------------------|
| UNIT         | N     | validation, authz, business rules          |
| INTEGRATION  | M     | DB constraints, serialization, middleware  |
| E2E          | P     | critical business flows                    |
| **Total**    | N+M+P |                                            |

### Risk Coverage Checklist

- [ ] Security: authn/authz bypass, field leaks, fail-open, injection, unsafe deserialization
- [ ] Concurrency: races, double-writes, non-idempotent handlers, deadlocks
- [ ] Reliability: timeouts, partial failures, malformed data, retry storms
- [ ] Data Integrity: constraint violations, orphaned records, inconsistent state
- [ ] Error Handling: graceful degradation, error propagation, boundary failures
- [ ] Observability: logging coverage, metrics emission, trace context
- [ ] Resilience: circuit breaker behavior, fallback paths, recovery procedures
- [ ] Performance: N+1 queries, memory leaks, unbounded resource usage
- [ ] State Management: stale cache, session handling, state transitions
- [ ] Compatibility: backwards compatibility, API versioning, schema migrations
- [ ] Business Logic: edge cases, boundary values, invariant violations

### Tag Distribution

| Tag          | Count | Notes                                      |
|--------------|-------|--------------------------------------------|
| happy_path   | N     |                                            |
| negative     | M     |                                            |
| fail_closed  | P     |                                            |
| authz        | Q     | if applicable                              |
| side_effects | R     | if applicable                              |

---

## How To Run

```bash
<copy/paste test command to run ONLY mentioned tests>
```