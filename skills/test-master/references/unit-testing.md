# Unit Testing

## pytest Pattern

```python
import pytest
from unittest.mock import AsyncMock, Mock


class TestUserService:
    @pytest.fixture
    def mock_repo(self):
        repo = Mock()
        repo.find_by_id = AsyncMock()
        repo.save = AsyncMock()
        return repo

    @pytest.fixture
    def service(self, mock_repo):
        return UserService(mock_repo)

    @pytest.fixture(autouse=True)
    def _reset_mocks(self, mock_repo):
        yield
        mock_repo.reset_mock()

    class TestGetUser:
        async def test_returns_user_when_found(self, service, mock_repo):
            user = {"id": "1", "name": "Test"}
            mock_repo.find_by_id.return_value = user

            result = await service.get_user("1")

            assert result == user
            mock_repo.find_by_id.assert_awaited_once_with("1")

        async def test_raises_not_found_when_user_missing(self, service, mock_repo):
            mock_repo.find_by_id.return_value = None

            with pytest.raises(NotFoundError):
                await service.get_user("1")
```

## Mocking Patterns

```python
from unittest.mock import AsyncMock, Mock, patch

# Mock functions
mock_fn = Mock(return_value="value")
async_fn = AsyncMock(return_value="async value")
failing_fn = AsyncMock(side_effect=RuntimeError("error"))

# Mock modules
with patch("app.database.query") as query:
    query.return_value = []

# Spy on an existing callable
with patch("builtins.print") as mocked_print:
    mocked_print.side_effect = lambda *args, **kwargs: None
```

## Test Organization

```python
class TestFeature:
    class TestHappyPath:
        def test_does_expected_behavior(self):
            pass

    class TestEdgeCases:
        def test_handles_empty_input(self):
            pass

        def test_handles_max_values(self):
            pass

    class TestErrorCases:
        def test_raises_on_invalid_input(self):
            pass
```

## Quick Reference

| Pattern                  | Use Case                     |
| ------------------------ | ---------------------------- |
| Test class or module     | Group related tests          |
| `def test_…`             | Single test case             |
| `@pytest.fixture`        | Setup before each test       |
| `Mock()` / `AsyncMock()` | Create a mock function       |
| `return_value`           | Mock an async or sync return |
| `pytest.raises`          | Assert an exception          |
