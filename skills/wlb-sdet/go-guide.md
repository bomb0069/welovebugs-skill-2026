# Go Test Guide

Code patterns for generating tests with the `testing` package and testify.

## Dependencies

```go
// go.mod
require github.com/stretchr/testify v1.9.0
```

If the project doesn't use testify, use standard `testing` package only.

## Test file structure

```go
package service

import (
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"
)

// Mock
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) FindByEmail(email string) (*User, error) {
    args := m.Called(email)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*User), args.Error(1)
}

func (m *MockUserRepository) Save(user *User) error {
    args := m.Called(user)
    return args.Error(0)
}
```

## Traceability in test reports

Use `t.Run` subtest names and `t.Log` to show traceability in `go test -v` output:

```go
const (
    requirement = "User must be aged 18-65 to register"
    source      = "test-cases/user-registration-age.md"
    level       = "Unit Test"
)

func logTraceability(t *testing.T, tcID, testCase, technique string) {
    t.Helper()
    t.Logf("Requirement: %s", requirement)
    t.Logf("TC-ID: %s | Test Case: %s", tcID, testCase)
    t.Logf("Technique: %s | Source: %s | Level: %s", technique, source, level)
}
```

This produces in `go test -v`:
```
=== RUN   TestUserRegistrationService/Age_Validation_(BVA)/UT-01:_accept_when_age_is_18
    Requirement: User must be aged 18-65 to register
    TC-ID: UT-01 | Test Case: Age at minimum boundary (18)
    Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md | Level: Unit Test
--- PASS: TestUserRegistrationService/Age_Validation_(BVA)/UT-01:_accept_when_age_is_18
```

## Test method patterns

### Single test

```go
func TestUserRegistrationService_Register(t *testing.T) {
    t.Run("UT-01: should accept registration when age is at minimum boundary (18)", func(t *testing.T) {
        logTraceability(t, "UT-01", "Age at minimum boundary (18)", "BVA (C1:Min)")
        mockRepo := new(MockUserRepository)
        mockRepo.On("Save", mock.Anything).Return(nil)
        service := NewUserRegistrationService(mockRepo)

        request := RegistrationRequest{
            Name:  "Sarah Lee",
            Age:   18,
            Email: "sarah@test.com",
        }

        // When
        result, err := service.Register(request)

        // Then
        require.NoError(t, err)
        assert.True(t, result.Accepted)
        assert.Equal(t, "Registration accepted, welcome page shown", result.Message)
    })
}
```

### Table-driven tests (parameterized)

```go
func TestUserRegistrationService_Register_ValidAgeBoundaries(t *testing.T) {
    tests := []struct {
        utID        string
        age         int
        description string
    }{
        {"UT-02", 18, "Min boundary"},
        {"UT-03", 19, "Above min"},
        {"UT-04", 64, "Below max"},
        {"UT-05", 65, "Max boundary"},
    }

    for _, tt := range tests {
        t.Run(fmt.Sprintf("%s: age %d (%s)", tt.utID, tt.age, tt.description), func(t *testing.T) {
            // Given - Valid BVA boundary values
            mockRepo := new(MockUserRepository)
            mockRepo.On("Save", mock.Anything).Return(nil)
            service := NewUserRegistrationService(mockRepo)

            request := RegistrationRequest{Name: "Test User", Age: tt.age, Email: "test@test.com"}

            // When
            result, err := service.Register(request)

            // Then
            require.NoError(t, err)
            assert.True(t, result.Accepted, "%s: Age %d (%s) should be accepted", tt.utID, tt.age, tt.description)
        })
    }
}
```

### Error test

```go
t.Run("UT-01: should return error when age is below minimum (17)", func(t *testing.T) {
    // Given - UT-01: Age below minimum boundary (17)
    service := NewUserRegistrationService(new(MockUserRepository))
    request := RegistrationRequest{Name: "Tom Park", Age: 17, Email: "tom@test.com"}

    // When
    result, err := service.Register(request)

    // Then
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "Age must be between 18 and 65")
    assert.False(t, result.Accepted)
})
```

## Grouping with t.Run subtests

```go
func TestUserRegistrationService(t *testing.T) {

    t.Run("Age Validation (BVA)", func(t *testing.T) {
        t.Run("valid boundaries", func(t *testing.T) {
            // table-driven tests for valid ages
        })
        t.Run("invalid boundaries", func(t *testing.T) {
            // table-driven tests for invalid ages
        })
    })

    t.Run("File Type Validation (EP)", func(t *testing.T) {
        t.Run("valid partitions", func(t *testing.T) { ... })
        t.Run("invalid partitions", func(t *testing.T) { ... })
    })

    t.Run("State Transitions", func(t *testing.T) {
        t.Run("valid transitions", func(t *testing.T) { ... })
        t.Run("invalid transitions", func(t *testing.T) { ... })
    })
}
```

## Mocking patterns (testify/mock)

```go
// Setup
mockRepo := new(MockUserRepository)
mockRepo.On("FindByEmail", "test@test.com").Return(nil, nil)
mockRepo.On("Save", mock.MatchedBy(func(u *User) bool {
    return u.Name == "Sarah Lee" && u.Age == 18
})).Return(nil)

// Verify
mockRepo.AssertCalled(t, "Save", mock.Anything)
mockRepo.AssertNumberOfCalls(t, "Save", 1)
mockRepo.AssertExpectations(t)
```

## Assertion patterns (testify/assert)

```go
// Boolean
assert.True(t, result.Accepted)
assert.False(t, result.Accepted)

// String
assert.Equal(t, "Expected message", result.Message)
assert.Contains(t, result.Message, "partial match")

// Number
assert.Equal(t, 30000, result.Amount)
assert.GreaterOrEqual(t, result.Age, 18)

// Error
assert.NoError(t, err)
assert.Error(t, err)
assert.EqualError(t, err, "expected error message")

// Nil
assert.Nil(t, result)
assert.NotNil(t, result)

// require (stops test on failure)
require.NoError(t, err)
require.NotNil(t, result)
```

## Test file location

- Source: `internal/service/user_registration.go`
- Test: `internal/service/user_registration_test.go` (same package)

## Naming convention

`TestServiceName_MethodName` or `TestServiceName` with `t.Run` subtests
