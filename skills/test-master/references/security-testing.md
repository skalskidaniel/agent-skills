# Security Testing

## Authentication Tests

```python
def test_rejects_invalid_credentials(client):
    response = client.post(
        "/api/login",
        json={"email": "user@test.com", "password": "wrong"},
    )
    assert response.status_code == 401


def test_rejects_expired_tokens(client):
    expired_token = create_expired_token()
    response = client.get(
        "/api/protected",
        headers={"Authorization": f"Bearer {expired_token}"},
    )
    assert response.status_code == 401


def test_rejects_tampered_tokens(client, valid_token):
    tampered_token = valid_token[:-5] + "xxxxx"
    response = client.get(
        "/api/protected",
        headers={"Authorization": f"Bearer {tampered_token}"},
    )
    assert response.status_code == 401


def test_enforces_rate_limiting_on_login(client):
    for _ in range(6):
        client.post(
            "/api/login",
            json={"email": "user@test.com", "password": "wrong"},
        )

    response = client.post(
        "/api/login",
        json={"email": "user@test.com", "password": "correct"},
    )
    assert response.status_code == 429
```

## Authorization Tests

```python
def test_denies_access_to_other_users_resources(client, user_a_token):
    response = client.get(
        "/api/users/other-user-id/data",
        headers={"Authorization": f"Bearer {user_a_token}"},
    )
    assert response.status_code == 403


def test_denies_admin_routes_to_regular_users(client, regular_user_token):
    response = client.delete(
        "/api/admin/users/123",
        headers={"Authorization": f"Bearer {regular_user_token}"},
    )
    assert response.status_code == 403
```

## Input Validation Tests

```python
def test_rejects_sql_injection_attempts(client):
    response = client.get(
        "/api/users",
        params={"search": "'; DROP TABLE users; --"},
    )
    assert response.status_code == 400


def test_rejects_xss_in_input_fields(client):
    response = client.post(
        "/api/posts",
        json={"title": '<script>alert("xss")</script>'},
    )
    assert response.status_code == 201
    assert "<script>" not in response.json()["title"]


def test_validates_file_upload_types(client):
    response = client.post(
        "/api/upload",
        files={"file": ("malicious.exe", b"not-a-real-binary", "application/octet-stream")},
    )
    assert response.status_code == 400
```

## Security Headers Test

```python
def test_sets_security_headers(client):
    response = client.get("/")

    assert response.headers["x-content-type-options"] == "nosniff"
    assert response.headers["x-frame-options"] == "DENY"
    assert "strict-transport-security" in response.headers
```

## Security Test Checklist

| Category       | Tests                                  |
| -------------- | -------------------------------------- |
| **Auth**       | Invalid creds, token expiry, tampering |
| **Input**      | SQL injection, XSS, command injection  |
| **Access**     | IDOR, privilege escalation             |
| **Rate Limit** | Brute force, API abuse                 |
| **Headers**    | CSP, HSTS, X-Frame-Options             |
| **Data**       | PII exposure, error messages           |

## Quick Reference

| Vulnerability | Test Approach                 |
| ------------- | ----------------------------- |
| SQL Injection | `'; DROP TABLE--` in inputs   |
| XSS           | `<script>alert(1)</script>`   |
| IDOR          | Access other user's resources |
| CSRF          | Missing/invalid tokens        |
| Auth Bypass   | Missing auth, expired tokens  |
