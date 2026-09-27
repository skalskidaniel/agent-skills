# Integration Testing

## API Testing (httpx)

```python
def test_creates_user_with_valid_data(client):
    response = client.post(
        "/api/users",
        json={"email": "test@test.com", "name": "Test"},
    )

    assert response.status_code == 201
    body = response.json()
    assert body["email"] == "test@test.com"
    assert body["name"] == "Test"
    assert "id" in body


def test_returns_400_for_invalid_email(client):
    response = client.post(
        "/api/users",
        json={"email": "invalid", "name": "Test"},
    )

    assert response.status_code == 400
    assert "email" in response.json()["error"]


def test_returns_401_without_auth_token(client):
    response = client.get("/api/users/me")

    assert response.status_code == 401
```

## Authenticated Requests

```python
import pytest


class TestProtectedEndpoints:
    @pytest.fixture
    def auth_token(self, client):
        response = client.post(
            "/api/auth/login",
            json={"email": "test@test.com", "password": "password"},
        )
        return response.json()["token"]

    def test_accesses_protected_route(self, client, auth_token):
        response = client.get(
            "/api/users/me",
            headers={"Authorization": f"Bearer {auth_token}"},
        )

        assert response.status_code == 200
```

## Database Testing

```python
import pytest


class TestUserRepository:
    @pytest.fixture(autouse=True)
    def clean_users(self, db):
        db.execute("DELETE FROM users")
        yield

    def test_creates_and_retrieves_user(self, user_repo):
        user = user_repo.create(email="test@test.com", name="Test")

        found = user_repo.find_by_id(user.id)

        assert found == user
```

Provide `client` with `httpx.Client` or `httpx.AsyncClient` against the app under test. Mark async tests with `@pytest.mark.asyncio` when the client is async:

```python
import pytest
from httpx import AsyncClient


@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    response = await client.post("/api/users/", json={
        "email": "test@example.com",
        "name": "Test",
    })
    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"


@pytest.mark.asyncio
async def test_invalid_email(client: AsyncClient):
    response = await client.post("/api/users/", json={
        "email": "invalid",
        "name": "Test",
    })
    assert response.status_code == 422
```

## Quick Reference

| Pattern                            | Purpose                |
| ---------------------------------- | ---------------------- |
| `json=`                            | Send a JSON body       |
| `headers=`                         | Set a header           |
| `assert response.status_code`      | Assert the status code |
| `response.headers["content-type"]` | Assert a header        |
| `response.json()`                  | Parsed JSON body       |
