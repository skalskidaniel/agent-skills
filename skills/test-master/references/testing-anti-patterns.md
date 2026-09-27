# Testing Anti-Patterns

---

## Core Principle

> **"Test what the code does, not what the mocks do."**

When tests verify mock behavior instead of actual functionality, they provide false confidence while catching zero real bugs.

---

## The Five Anti-Patterns

### Anti-Pattern 1: Testing Mock Behavior

**The Problem:** Verifying that mocks exist and were called, rather than testing actual component output.

```python
from unittest.mock import AsyncMock

# BAD: Testing the mock, not the behavior
def test_should_call_the_api():
    mock_api = AsyncMock(return_value={"data": "test"})
    service = UserService(mock_api)

    service.get_user(1)

    mock_api.assert_called_with(1)  # Testing the mock, not the result
```

```python
# GOOD: Testing actual behavior
async def test_returns_user_data_from_api():
    mock_api = AsyncMock(return_value={"id": 1, "name": "Alice"})
    service = UserService(mock_api)

    user = await service.get_user(1)

    assert user["name"] == "Alice"  # Testing actual output
```

**Solution:** Test the genuine component output. If you can only verify mock calls, reconsider whether the test adds value.

---

### Anti-Pattern 2: Test-Only Methods in Production

**The Problem:** Adding methods to production classes solely for test setup or cleanup.

```python
# BAD: Production code polluted with test concerns
class UserCache:
    def __init__(self):
        self._cache: dict[int, User] = {}

    def get_user(self, user_id: int) -> User | None:
        return self._cache.get(user_id)

    # This method exists ONLY for tests
    def _reset_for_testing(self) -> None:
        self._cache.clear()
```

```python
# GOOD: Test utilities separate from production
# production/user_cache.py
class UserCache:
    def __init__(self):
        self._cache: dict[int, User] = {}

    def get_user(self, user_id: int) -> User | None:
        return self._cache.get(user_id)


# test/helpers.py
def create_fresh_cache() -> UserCache:
    return UserCache()  # Fresh instance per test
```

**Solution:** Relocate cleanup logic to test utility functions. Use fresh instances per test instead of reset methods.

---

### Anti-Pattern 3: Mocking Without Understanding

**The Problem:** Over-mocking without grasping side effects, leading to tests that pass but hide real issues.

```python
from unittest.mock import patch

# BAD: Mocking everything without understanding
async def test_should_process_order():
    with (
        patch("app.inventory"),
        patch("app.payment"),
        patch("app.shipping"),
        patch("app.notifications"),
    ):
        result = await process_order(order)

    assert result.success is True  # What did we actually test?
```

```python
# GOOD: Strategic mocking with real components where possible
async def test_processes_order_with_real_inventory_check():
    # Real inventory service against a test database
    inventory = InventoryService(test_db)

    # Mock only external services
    payment = mock_payment_gateway()

    processor = OrderProcessor(inventory, payment)
    result = await processor.process(order)

    assert result.success is True
    assert await inventory.get_stock(order.item_id) == original_stock - 1
```

**Solution:** Run tests with real implementations first to understand behavior. Then mock at the appropriate level — external services, not internal logic.

---

### Anti-Pattern 4: Incomplete Mocks

**The Problem:** Partial mock responses missing downstream fields that production code expects.

```python
from unittest.mock import AsyncMock

# BAD: Incomplete mock response
mock_user_api = AsyncMock(return_value={
    "id": 1,
    "name": "Test User",
    # Missing: email, created_at, permissions, settings...
})

# Test passes, but production crashes when accessing user["email"]
```

```python
# GOOD: Complete mock matching a real API response
mock_user_api = AsyncMock(return_value={
    "id": 1,
    "name": "Test User",
    "email": "test@example.com",
    "created_at": "2024-01-01T00:00:00Z",
    "permissions": ["read", "write"],
    "settings": {
        "theme": "light",
        "notifications": True,
    },
})

# Or use a factory
mock_user_api = AsyncMock(
    return_value=create_mock_user(name="Test User")  # Factory fills defaults
)
```

**Solution:** Mirror the complete real API response structure. Use factories to generate complete mock objects with sensible defaults.

---

### Anti-Pattern 5: Integration Tests as Afterthought

**The Problem:** Treating testing as optional follow-up work rather than integral to development.

```python
# BAD: "We'll add tests later"
# Day 1: Write 500 lines of code
# Day 2: Write 500 more lines
# Day 3: "We need to ship, tests can wait"
# Day 30: Catastrophic bug in production
# Day 31: "Why didn't we have tests?"
```

```python
import pytest


# GOOD: Tests are part of implementation
# Write failing test
async def test_rejects_duplicate_usernames():
    await create_user({"username": "alice"})

    with pytest.raises(ValueError, match="Username already exists"):
        await create_user({"username": "alice"})


# Make it pass
async def create_user(data: dict) -> dict:
    existing = await db.users.find_by_username(data["username"])
    if existing:
        raise ValueError("Username already exists")
    return await db.users.create(data)


# Feature AND test ship together
```

**Solution:** Follow TDD — testing is implementation, not documentation. No feature is "done" without tests.

---

## Detection Checklist

Review your tests for these warning signs:

| Warning Sign                                          | Anti-Pattern                  |
| ----------------------------------------------------- | ----------------------------- |
| `assert_called_with()` without testing output         | Testing mock behavior         |
| Methods named `_reset` or `for_testing` in production | Test-only methods             |
| Every dependency is mocked                            | Mocking without understanding |
| Mocks return `{"success": True}` only                 | Incomplete mocks              |
| Test files added weeks after the feature ships        | Tests as afterthought         |

---

## Quick Reference

| Anti-Pattern          | Symptom                                 | Fix                          |
| --------------------- | --------------------------------------- | ---------------------------- |
| Testing mocks         | Only mock assertions, no behavior tests | Assert on actual output      |
| Test-only methods     | `_reset()`, `_set_for_test()` in prod   | Use fresh instances          |
| Over-mocking          | 10+ mocks per test                      | Test with real deps first    |
| Incomplete mocks      | Minimal stub responses                  | Use factories, match reality |
| Tests as afterthought | Features ship untested                  | TDD from the start           |

---

_Content adapted from [obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent (@obra), MIT License._
