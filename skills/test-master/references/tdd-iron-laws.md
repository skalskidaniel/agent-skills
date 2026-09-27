# TDD Iron Laws

---

## The Fundamental Principle

> **NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

This is non-negotiable. If you wrote production code before writing a failing test, delete it and start over. No exceptions.

---

## The Three Iron Laws

### Iron Law 1: The Fundamental Rule

> "You shall not write any production code unless it is to make a failing test pass."

Every line of production code must have a corresponding test that:

1. Was written first
2. Was observed to fail
3. Now passes because of that code

### Iron Law 2: Proof Through Observation

> "If you didn't watch the test fail, you don't know if it tests the right thing."

Mandatory verification steps:

- Write the test
- Run it and **observe the failure**
- Verify the failure message is meaningful
- Only then implement the fix

A test you've never seen fail proves nothing.

### Iron Law 3: The Final Rule

> "Production code exists → A test exists that failed first. Otherwise → It's not TDD."

There is no middle ground. Code written without a prior failing test is not test-driven development, regardless of how many tests exist afterward.

---

## The RED-GREEN-REFACTOR Cycle

### RED: Write One Minimal Failing Test

```python
def test_sum_returns_0_for_empty_list():
    assert sum_numbers([]) == 0
# Run: FAIL — sum_numbers is not defined
```

**Requirements:**

- One test at a time
- Minimal scope
- Clear failure message
- Observe the red

### GREEN: Implement Simplest Passing Code

```python
def sum_numbers(numbers: list[int]) -> int:
    return 0
# Run: PASS
```

**Requirements:**

- Simplest possible implementation
- No extra features
- No optimization
- Just make it pass

### REFACTOR: Improve While Keeping Tests Green

```python
def sum_numbers(numbers: list[int]) -> int:
    return sum(numbers)
# Run: PASS (still)
```

**Requirements:**

- Tests must stay green
- Remove duplication
- Improve clarity
- No new functionality

---

## Common Rationalizations to Reject

These thoughts indicate you're about to violate TDD:

| Rationalization                                        | Why It's Wrong                                   |
| ------------------------------------------------------ | ------------------------------------------------ |
| "I can manually test this quickly"                     | Manual testing doesn't prevent regression        |
| "I'll write tests after to save time"                  | You'll skip edge cases and test implementation   |
| "This is too simple to need a test"                    | Simple code changes; tests document expectations |
| "I've already written the code, I can't delete it now" | Sunk cost fallacy; delete it                     |
| "I know this works, I've done it before"               | Your memory isn't documentation                  |
| "We're in a hurry"                                     | Technical debt costs more than TDD               |

---

## Practical Application

### Starting a New Feature

```python
# 1. RED: Write failing test for simplest behavior
def test_validate_email_rejects_empty_email():
    assert validate_email("") is False


# 2. GREEN: Implement minimal passing code
def validate_email(email: str) -> bool:
    return len(email) > 0


# 3. RED: Add next failing test
def test_validate_email_rejects_email_without_at():
    assert validate_email("invalid") is False


# 4. GREEN: Extend to pass both tests
def validate_email(email: str) -> bool:
    return len(email) > 0 and "@" in email


# Continue cycle...
```

### Fixing a Bug

```python
# 1. RED: Write test that exposes the bug
def test_sum_handles_negative_numbers():
    assert sum_numbers([-1, -2, -3]) == -6
# Run: FAIL — got 0 instead of -6


# 2. GREEN: Fix the bug
def sum_numbers(numbers: list[int]) -> int:
    return sum(numbers)
# Run: PASS

# Bug is now fixed AND protected against regression
```

---

## Verification Checklist

Before claiming any code is complete:

- [ ] Every production function has corresponding tests
- [ ] Each test was written before its implementation
- [ ] Each test was observed to fail first
- [ ] Tests verify behavior, not implementation
- [ ] Refactoring kept all tests green
- [ ] No production code exists without a test

---

_Content adapted from [obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent (@obra), MIT License._
