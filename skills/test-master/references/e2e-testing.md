# E2E Testing

## E2E Test Strategy

```python
# Critical user paths to test
critical_paths = [
    "User registration and login",
    "Core product/service workflow",
    "Payment/checkout flow",
    "Settings and profile management",
]
```

## User Flow Testing

```python
import re

import pytest
from playwright.sync_api import Page, expect


def test_complete_registration(page: Page):
    page.goto("/register")

    page.get_by_label("Email").fill("new@example.com")
    page.get_by_label("Password").fill("SecurePass123!")
    page.get_by_label("Confirm Password").fill("SecurePass123!")
    page.get_by_role("button", name="Register").click()

    expect(page).to_have_url(re.compile(r"dashboard"))
    expect(page.get_by_text("Welcome")).to_be_visible()


def test_shows_validation_errors(page: Page):
    page.goto("/register")

    page.get_by_label("Email").fill("invalid")
    page.get_by_role("button", name="Register").click()

    expect(page.get_by_text("Invalid email")).to_be_visible()
```

## Checkout Flow

```python
import re

from playwright.sync_api import Page, expect


def test_complete_purchase(page: Page):
    # Add to cart
    page.goto("/products/123")
    page.get_by_role("button", name="Add to Cart").click()
    expect(page.get_by_test_id("cart-count")).to_have_text("1")

    # Checkout
    page.goto("/cart")
    page.get_by_role("button", name="Checkout").click()

    # Payment
    page.get_by_label("Card Number").fill("4242424242424242")
    page.get_by_label("Expiry").fill("12/25")
    page.get_by_label("CVC").fill("123")
    page.get_by_role("button", name="Pay").click()

    # Confirmation
    expect(page).to_have_url(re.compile(r"order-confirmation"))
    expect(page.get_by_text("Order Confirmed")).to_be_visible()
```

## Test Data Management

```python
import pytest
from playwright.sync_api import Page

# fixtures/test_data.py
test_users = {
    "standard": {
        "email": "standard@test.com",
        "password": "TestPass123!",
    },
    "admin": {
        "email": "admin@test.com",
        "password": "AdminPass123!",
    },
}


@pytest.fixture(autouse=True)
def seed_and_cleanup(page: Page):
    page.request.post("/api/test/seed")
    yield
    page.request.post("/api/test/cleanup")
```

## Cross-Browser Testing

```python
import pytest

DEVICES = [
    "Desktop Chrome",
    "Desktop Firefox",
    "Desktop Safari",
    "Pixel 5",
    "iPhone 13",
]


@pytest.fixture(params=DEVICES)
def device_page(playwright, browser, request):
    device = playwright.devices[request.param]
    context = browser.new_context(**device)
    page = context.new_page()
    yield page
    context.close()
```

Desktop browsers can also be selected with pytest-playwright:

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = """
  --browser chromium
  --browser firefox
  --browser webkit
"""
```

## Quick Reference

| Pattern        | When to Use                 |
| -------------- | --------------------------- |
| Happy path     | Critical user journeys      |
| Error handling | Form validation, API errors |
| Edge cases     | Empty states, max limits    |
| Cross-browser  | Before major releases       |
| Mobile         | Responsive features         |

| Priority | Test Coverage                     |
| -------- | --------------------------------- |
| **P0**   | Registration, login, core feature |
| **P1**   | Payment, settings, common flows   |
| **P2**   | Edge cases, admin features        |
| **P3**   | Rare scenarios                    |
