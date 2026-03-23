# Java Spring Boot Test Guide

Code patterns and conventions for generating JUnit 5 tests in Spring Boot projects.

## Dependencies

Expect these in `pom.xml` or `build.gradle`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

This includes: JUnit 5, AssertJ, Mockito, Spring Test.

## Test class structure

```java
package com.example.service;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("User Registration Service")
@Tag("requirement:user-registration-age")
class UserRegistrationServiceTest {

    // Traceability constants
    static final String REQUIREMENT = "User must be aged 18-65 to register";
    static final String SOURCE = "test-cases/user-registration-age.md";
    static final String LEVEL = "Unit Test";

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserRegistrationService registrationService;

    // Test methods go here, grouped by @Nested classes
}
```

## Traceability in test reports

Every test must include metadata that appears in the JUnit report, linking
the test back to the requirement. Use these mechanisms:

- `@DisplayName` — shows TC-ID and description in the report
- `@Tag` — enables filtering by requirement, technique, or level
- **Constants** — define `REQUIREMENT`, `SOURCE`, `LEVEL` at the class level
- **Comments** — include requirement, technique, and source in the Given section

```java
@Test
@DisplayName("UT-01: Accept registration when age is at minimum boundary (18)")
@Tag("technique:BVA")
@Tag("level:unit-test")
void should_acceptRegistration_when_ageIsAtMinimumBoundary() {
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Test Case: Age at minimum boundary (18)
    // Technique: BVA (Min boundary) | Source: test-cases/user-registration-age.md
    // Level: Unit Test
    ...
}
```

This produces a report like:
```
✓ User Registration Service [requirement:user-registration-age]
  ✓ Age Validation (BVA) [technique:BVA]
    ✓ UT-01: Accept registration when age is at minimum boundary (18)
    ✓ UT-02: Reject registration when age is below minimum boundary (17)
```

## Test method patterns

### Single test case → single test method

```java
@Test
@DisplayName("UT-01: Accept registration when age is at minimum boundary (18)")
@Tag("technique:BVA")
void should_acceptRegistration_when_ageIsAtMinimumBoundary() {
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Test Case: Age at minimum boundary (18)
    // Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
    var request = new RegistrationRequest("Sarah Lee", 18, "sarah@test.com");

    // When
    var result = registrationService.register(request);

    // Then
    assertThat(result.isAccepted()).isTrue();
    assertThat(result.getMessage()).isEqualTo("Registration accepted, welcome page shown");
}
```

### Multiple similar test cases → @ParameterizedTest

When multiple UT rows test the same behavior with different values (e.g., all valid
BVA boundary points that should be accepted), combine them:

```java
@ParameterizedTest(name = "UT-{0}: Accept registration when age is {1} (valid boundary)")
@CsvSource({
    "02, 18, Min boundary",
    "03, 19, Above min",
    "04, 64, Below max",
    "05, 65, Max boundary"
})
@DisplayName("Accept registration for valid age boundaries")
void should_acceptRegistration_when_ageIsWithinValidRange(
        String utId, int age, String description) {
    // Given - Valid BVA boundary values for age
    var request = new RegistrationRequest("Test User", age, "test@test.com");

    // When
    var result = registrationService.register(request);

    // Then
    assertThat(result.isAccepted())
        .as("UT-%s: Age %d (%s) should be accepted", utId, age, description)
        .isTrue();
}
```

### Invalid cases with expected errors

```java
@Test
@DisplayName("UT-06: Reject registration when age is above maximum boundary (66)")
void should_rejectRegistration_when_ageIsAboveMaximumBoundary() {
    // Given - UT-06: Age above maximum boundary (66)
    // Technique: C1:BVA AboveMax
    var request = new RegistrationRequest("Mark Hall", 66, "mark@test.com");

    // When
    var result = registrationService.register(request);

    // Then
    assertThat(result.isAccepted()).isFalse();
    assertThat(result.getMessage()).isEqualTo("Age must be between 18 and 65");
}
```

### Exception-based validation

If the service throws exceptions instead of returning result objects:

```java
@Test
@DisplayName("UT-01: Throw ValidationException when age is below minimum (17)")
void should_throwValidationException_when_ageIsBelowMinimum() {
    // Given - UT-01: Age below minimum boundary (17)
    var request = new RegistrationRequest("Tom Park", 17, "tom@test.com");

    // When & Then
    assertThatThrownBy(() -> registrationService.register(request))
        .isInstanceOf(ValidationException.class)
        .hasMessage("Age must be between 18 and 65");
}
```

## Grouping with @Nested

### BVA test grouping

```java
@Nested
@DisplayName("Age Validation (BVA)")
class AgeValidation {

    @ParameterizedTest
    @CsvSource({"18", "19", "64", "65"})
    void should_accept_when_ageIsValid(int age) { ... }

    @ParameterizedTest
    @CsvSource({"17", "66"})
    void should_reject_when_ageIsInvalid(int age) { ... }
}
```

### EP test grouping

```java
@Nested
@DisplayName("File Type Validation (EP)")
class FileTypeValidation {

    @Nested
    @DisplayName("Valid Partitions")
    class ValidPartitions {
        @ParameterizedTest
        @CsvSource({"'.pdf'", "'.doc'", "'.xlsx'"})
        void should_accept_when_fileTypeIsSupported(String fileType) { ... }
    }

    @Nested
    @DisplayName("Invalid Partitions")
    class InvalidPartitions {
        @ParameterizedTest
        @CsvSource({"'.exe'", "'.bat'", "''"})
        void should_reject_when_fileTypeIsNotSupported(String fileType) { ... }
    }
}
```

### Decision Table test grouping

```java
@Nested
@DisplayName("Loan Application (Decision Table)")
class LoanApplication {

    @Nested
    @DisplayName("R1: All conditions valid")
    class AllValid {
        @Test void should_accept_when_allConditionsAreMet_minBoundaries() { ... }
        @Test void should_accept_when_allConditionsAreMet_maxBoundaries() { ... }
    }

    @Nested
    @DisplayName("R2-R3: Single condition invalid")
    class SingleInvalid {
        @Test void should_reject_when_amountIsBelowMinimum() { ... }
        @Test void should_reject_when_membershipIsInvalid() { ... }
    }
}
```

### Sequential Condition test grouping

```java
@Nested
@DisplayName("Loan Validation (Sequential)")
class LoanValidation {

    @Nested
    @DisplayName("Fails at C1: Age")
    class FailsAtAge {
        @Test void should_rejectAge_when_ageBelowMinimum() { ... }
        @Test void should_rejectAge_when_ageAboveMaximum() { ... }
    }

    @Nested
    @DisplayName("Fails at C2: Membership (age passed)")
    class FailsAtMembership {
        @Test void should_rejectMembership_when_membershipIsInvalid() { ... }
    }

    @Nested
    @DisplayName("All conditions pass")
    class AllPass {
        @Test void should_accept_when_allConditionsPass() { ... }
    }
}
```

### State Transition test grouping

```java
@Nested
@DisplayName("Order State Transitions")
class OrderStateTransitions {

    @Nested
    @DisplayName("Valid Transitions")
    class ValidTransitions {
        @Test void should_changeToSubmitted_when_draftOrderIsSubmitted() { ... }
        @Test void should_changeToApproved_when_submittedOrderIsApproved() { ... }
    }

    @Nested
    @DisplayName("Invalid Transitions")
    class InvalidTransitions {
        @Test void should_blockApproval_when_orderIsDraft() { ... }
        @Test void should_blockShipping_when_orderIsNotApproved() { ... }
    }

    @Nested
    @DisplayName("Path Tests")
    class PathTests {
        @Test void should_completeFullLifecycle_happyPath() { ... }
        @Test void should_handleRejectionAndResubmit() { ... }
    }
}
```

## Mocking patterns

### Service with repository dependency

```java
@Test
void should_saveUser_when_registrationIsValid() {
    // Given
    var request = new RegistrationRequest("Sarah Lee", 25, "sarah@test.com");
    when(userRepository.existsByEmail("sarah@test.com")).thenReturn(false);

    // When
    registrationService.register(request);

    // Then
    verify(userRepository).save(argThat(user ->
        user.getName().equals("Sarah Lee") &&
        user.getAge() == 25
    ));
}
```

### Service with external API dependency

```java
@Test
void should_callPaymentGateway_when_loanIsApproved() {
    // Given
    var application = new LoanApplication(30, "Gold", new BigDecimal("500000"));
    when(paymentGateway.validateAccount(any())).thenReturn(true);

    // When
    loanService.apply(application);

    // Then
    verify(paymentGateway).validateAccount(any());
}
```

## Test method naming convention

Format: `should_<expectedBehavior>_when_<condition>`

Examples:
- `should_acceptRegistration_when_ageIsAtMinimumBoundary`
- `should_rejectRegistration_when_ageIsBelowMinimum`
- `should_throwValidationException_when_emailIsInvalid`
- `should_changeStatusToApproved_when_orderIsSubmitted`
- `should_blockShipping_when_orderIsNotApproved`

## AssertJ assertion patterns

```java
// Boolean
assertThat(result.isAccepted()).isTrue();
assertThat(result.isAccepted()).isFalse();

// String
assertThat(result.getMessage()).isEqualTo("Expected message");
assertThat(result.getMessage()).contains("partial match");
assertThat(result.getMessage()).startsWith("Error:");

// Number
assertThat(result.getAmount()).isEqualTo(new BigDecimal("30000.00"));
assertThat(result.getAge()).isBetween(18, 65);

// Object
assertThat(result.getStatus()).isEqualTo(OrderStatus.APPROVED);

// Collection
assertThat(result.getErrors()).hasSize(1);
assertThat(result.getErrors()).contains("Age must be between 18 and 65");

// Exception
assertThatThrownBy(() -> service.process(input))
    .isInstanceOf(ValidationException.class)
    .hasMessage("Invalid input");
```
