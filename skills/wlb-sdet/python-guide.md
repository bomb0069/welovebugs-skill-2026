# Python Test Guide

Code patterns for generating tests with pytest.

## Dependencies

```
# requirements.txt or pyproject.toml
pytest>=7.0
pytest-mock>=3.0
```

## Test file structure

```python
import pytest
from unittest.mock import MagicMock, patch
from app.services.user_registration_service import UserRegistrationService


class TestUserRegistrationService:
    """User Registration Service tests."""

    # Traceability
    REQUIREMENT = "User must be aged 18-65 to register"
    SOURCE = "test-cases/user-registration-age.md"
    LEVEL = "Unit Test"

    def setup_method(self):
        """Set up test fixtures."""
        self.mock_repository = MagicMock()
        self.service = UserRegistrationService(self.mock_repository)
```

## Traceability in test reports

Use docstrings and `@pytest.mark` to show traceability in pytest output.
The docstring appears in `pytest -v` output and in HTML reports.

```python
import pytest

# Custom markers (register in pyproject.toml or conftest.py)
# [tool.pytest.ini_options]
# markers = [
#     "requirement: links test to a requirement",
#     "technique: test design technique used",
# ]

@pytest.mark.requirement("User must be aged 18-65 to register")
@pytest.mark.technique("BVA")
def test_should_accept_when_age_is_at_minimum_boundary(self):
    """UT-01: Age at minimum boundary (18)
    Requirement: User must be aged 18-65 to register
    Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
    Level: Unit Test
    """
    ...
```

This produces in `pytest -v`:
```
PASSED test_user_registration.py::TestAgeValidation::test_should_accept_when_age_is_at_minimum_boundary
  UT-01: Age at minimum boundary (18)
  Requirement: User must be aged 18-65 to register
  Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
```

## Test method patterns

### Single test

```python
@pytest.mark.requirement("User must be aged 18-65 to register")
@pytest.mark.technique("BVA")
def test_should_accept_registration_when_age_is_at_minimum_boundary(self):
    """UT-01: Age at minimum boundary (18)
    Requirement: User must be aged 18-65 to register
    Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
    Level: Unit Test
    """
    # Given - UT-01: Age at minimum boundary (18)
    request = RegistrationRequest(name="Sarah Lee", age=18, email="sarah@test.com")

    # When
    result = self.service.register(request)

    # Then
    assert result.is_accepted is True
    assert result.message == "Registration accepted, welcome page shown"
```

### Parameterized test

```python
@pytest.mark.parametrize(
    "ut_id, age, description",
    [
        ("UT-02", 18, "Min boundary"),
        ("UT-03", 19, "Above min"),
        ("UT-04", 64, "Below max"),
        ("UT-05", 65, "Max boundary"),
    ],
)
def test_should_accept_registration_when_age_is_valid(self, ut_id, age, description):
    """Accept registration for valid age boundaries."""
    # Given - Valid BVA boundary values
    request = RegistrationRequest(name="Test User", age=age, email="test@test.com")

    # When
    result = self.service.register(request)

    # Then
    assert result.is_accepted is True, f"{ut_id}: Age {age} ({description}) should be accepted"
```

### Exception test

```python
def test_should_raise_validation_error_when_age_is_below_minimum(self):
    """UT-01: Age below minimum (17)."""
    # Given - UT-01: Age below minimum boundary (17)
    request = RegistrationRequest(name="Tom Park", age=17, email="tom@test.com")

    # When & Then
    with pytest.raises(ValidationError, match="Age must be between 18 and 65"):
        self.service.register(request)
```

## Grouping with inner classes

```python
class TestUserRegistrationService:

    class TestAgeValidation:
        """Age Validation (BVA)."""

        class TestValidBoundaries:
            @pytest.mark.parametrize("age", [18, 19, 64, 65])
            def test_should_accept_when_age_is_valid(self, service, age):
                ...

        class TestInvalidBoundaries:
            @pytest.mark.parametrize("age", [17, 66])
            def test_should_reject_when_age_is_invalid(self, service, age):
                ...

    class TestFileTypeValidation:
        """File Type Validation (EP)."""

        class TestValidPartitions:
            ...

        class TestInvalidPartitions:
            ...
```

## Mocking patterns

```python
# With unittest.mock
from unittest.mock import MagicMock, patch

mock_repo = MagicMock()
mock_repo.find_by_email.return_value = None
mock_repo.save.return_value = saved_user

# Verify
mock_repo.save.assert_called_once()
mock_repo.save.assert_called_with(
    pytest.approx(expected_user, rel=None, abs=None)
)

# With pytest-mock (mocker fixture)
def test_example(self, mocker):
    mock_gateway = mocker.patch("app.services.PaymentGateway")
    mock_gateway.return_value.validate.return_value = True
```

## Assertion patterns

```python
# Boolean
assert result.is_accepted is True
assert result.is_accepted is False

# String
assert result.message == "Expected message"
assert "partial match" in result.message

# Number
assert result.amount == 30000
assert 18 <= result.age <= 65

# Object
assert result == expected_result
assert result.status == OrderStatus.APPROVED

# List
assert len(result.errors) == 1
assert "Age must be between 18 and 65" in result.errors

# Exception
with pytest.raises(ValidationError, match="Invalid input"):
    service.process(input_data)
```

## Test file location

- Source: `app/services/user_registration_service.py`
- Test: `tests/services/test_user_registration_service.py`
- Or: `tests/test_user_registration_service.py`

## Naming convention

`test_should_<expected_behavior>_when_<condition>` (snake_case)
