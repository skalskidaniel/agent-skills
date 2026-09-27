---
name: test-master
description: Generates test files, creates mocking strategies, analyzes code coverage, designs test architectures, and produces test plans and defect reports across functional, performance, and security testing disciplines. Use when writing unit tests, integration tests, or E2E tests.
---

# Test Master

Comprehensive testing specialist ensuring software quality through functional, performance, and security testing.

## Core Workflow

1. **Define scope** — Identify what to test and which testing types apply
2. **Create strategy** — Plan the test approach across functional, performance, and security perspectives
3. **Write tests** — Implement tests with proper assertions (see example below)
4. **Execute** — Run tests and collect results
   - If tests fail: classify the failure (assertion error vs. environment/flakiness), fix root cause, re-run
   - If tests are flaky: isolate ordering dependencies, check async handling, add retry or stabilization logic
5. **Report** — Document findings with severity ratings and actionable fix recommendations
   - Verify coverage targets are met before closing; flag gaps explicitly

## Quick-Start Example

A minimal pytest unit test illustrating the key patterns this skill enforces:

```python
import pytest


def test_calculate_discount_applies_10_percent_for_premium_users():
    result = calculate_discount(price=100, user_tier="premium")
    assert result == 90  # specific outcome, not just truthy


def test_calculate_discount_raises_on_negative_price():
    with pytest.raises(ValueError, match="Price must be non-negative"):
        calculate_discount(price=-1, user_tier="standard")
```

Use the same structure in other frameworks: a plain-language test name, one behavior per test, and an assertion on the exact outcome.

## Reference Guide

Load detailed guidance based on context:

<!-- TDD Iron Laws and Testing Anti-Patterns adapted from obra/superpowers by Jesse Vincent (@obra), MIT License -->

| Topic                 | Reference                             | Load When                                                        |
| --------------------- | ------------------------------------- | ---------------------------------------------------------------- |
| Unit Testing          | `references/unit-testing.md`          | pytest patterns                                                  |
| Integration           | `references/integration-testing.md`   | API testing, httpx                                               |
| E2E                   | `references/e2e-testing.md`           | E2E strategy, user flows                                         |
| Performance           | `references/performance-testing.md`   | Locust, load testing                                             |
| Security              | `references/security-testing.md`      | Security test checklist                                          |
| Reports               | `references/test-reports.md`          | Report templates, findings                                       |
| QA Methodology        | `references/qa-methodology.md`        | Manual testing, quality advocacy, shift-left, continuous testing |
| TDD Iron Laws         | `references/tdd-iron-laws.md`         | TDD methodology, test-first development, red-green-refactor      |
| Testing Anti-Patterns | `references/testing-anti-patterns.md` | Test review, mock issues, test quality problems                  |

## Constraints

**MUST DO**

- Test happy paths AND error/edge cases (e.g., empty input, null, boundary values)
- Mock external dependencies — never call real APIs or databases in unit tests
- Use meaningful `test_…` names that read as plain-English specifications
- Assert specific outcomes (`assert result == 90`), not just truthiness
- Run tests in CI/CD; document and remediate coverage gaps

**MUST NOT**

- Skip error-path testing (e.g., don't test only the success branch of a try/except)
- Use production data in tests — use fixtures or factories instead
- Create order-dependent tests — each test must be independently runnable
- Ignore flaky tests — quarantine and fix them; don't just re-run until green
- Test implementation details (internal method calls) — test observable behaviour

## Output Templates

When creating test plans, provide:

1. Test scope and approach
2. Test cases with expected outcomes
3. Coverage analysis
4. Findings with severity (Critical/High/Medium/Low)
5. Specific fix recommendations

[Documentation](https://jeffallan.github.io/claude-skills/skills/quality/test-master/)
