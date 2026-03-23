# Kotlin Spring Boot Test Guide

Code patterns for generating JUnit 5 tests in Kotlin Spring Boot projects.

## Dependencies

```kotlin
// build.gradle.kts
testImplementation("org.springframework.boot:spring-boot-starter-test")
testImplementation("io.mockk:mockk:1.13.+")
testImplementation("com.ninja-squad:springmockk:4.0.+")
```

## Test class structure

```kotlin
package com.example.service

import io.mockk.*
import io.mockk.impl.annotations.InjectMockKs
import io.mockk.impl.annotations.MockK
import io.mockk.junit5.MockKExtension
import org.assertj.core.api.Assertions.assertThat
import org.assertj.core.api.Assertions.assertThatThrownBy
import org.junit.jupiter.api.*
import org.junit.jupiter.api.extension.ExtendWith
import org.junit.jupiter.params.ParameterizedTest
import org.junit.jupiter.params.provider.CsvSource

@ExtendWith(MockKExtension::class)
@DisplayName("User Registration Service")
@Tag("requirement:user-registration-age")
class UserRegistrationServiceTest {

    companion object {
        const val REQUIREMENT = "User must be aged 18-65 to register"
        const val SOURCE = "test-cases/user-registration-age.md"
        const val LEVEL = "Unit Test"
    }

    @MockK
    private lateinit var userRepository: UserRepository

    @InjectMockKs
    private lateinit var registrationService: UserRegistrationService
}
```

## Traceability in test reports

Same as Java — use `@DisplayName`, `@Tag`, and comments:

```kotlin
@Test
@DisplayName("UT-01: Accept registration when age is at minimum boundary (18)")
@Tag("technique:BVA")
fun should_acceptRegistration_when_ageIsAtMinimumBoundary() {
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Test Case: Age at minimum boundary (18)
    // Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
    ...
}
```

## Test method patterns

### Single test

```kotlin
@Test
@DisplayName("UT-01: Accept registration when age is at minimum boundary (18)")
@Tag("technique:BVA")
fun should_acceptRegistration_when_ageIsAtMinimumBoundary() {
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Technique: BVA (C1:Min)
    val request = RegistrationRequest("Sarah Lee", 18, "sarah@test.com")

    // When
    val result = registrationService.register(request)

    // Then
    assertThat(result.isAccepted).isTrue()
    assertThat(result.message).isEqualTo("Registration accepted, welcome page shown")
}
```

### Parameterized test

```kotlin
@ParameterizedTest(name = "UT-{0}: Accept when age is {1} ({2})")
@CsvSource(
    "02, 18, Min boundary",
    "03, 19, Above min",
    "04, 64, Below max",
    "05, 65, Max boundary"
)
fun should_acceptRegistration_when_ageIsValid(utId: String, age: Int, description: String) {
    // Given - Valid BVA boundary values
    val request = RegistrationRequest("Test User", age, "test@test.com")

    // When
    val result = registrationService.register(request)

    // Then
    assertThat(result.isAccepted)
        .`as`("UT-$utId: Age $age ($description) should be accepted")
        .isTrue()
}
```

### Exception test

```kotlin
@Test
fun should_throwValidationException_when_ageIsBelowMinimum() {
    // Given - UT-01: Age below minimum (17)
    val request = RegistrationRequest("Tom Park", 17, "tom@test.com")

    // When & Then
    assertThatThrownBy { registrationService.register(request) }
        .isInstanceOf(ValidationException::class.java)
        .hasMessage("Age must be between 18 and 65")
}
```

## MockK patterns

```kotlin
// Stubbing
every { userRepository.existsByEmail("test@test.com") } returns false
every { userRepository.save(any()) } returns mockUser

// Verification
verify { userRepository.save(match { it.name == "Sarah Lee" && it.age == 18 }) }
verify(exactly = 0) { paymentGateway.charge(any()) }

// Relaxed mock (returns defaults)
@MockK(relaxed = true)
private lateinit var auditService: AuditService
```

## Grouping with @Nested

```kotlin
@Nested
@DisplayName("Age Validation (BVA)")
inner class AgeValidation {

    @Nested
    @DisplayName("Valid boundaries")
    inner class ValidBoundaries {
        @ParameterizedTest
        @CsvSource("18", "19", "64", "65")
        fun should_accept_when_ageIsValid(age: Int) { ... }
    }

    @Nested
    @DisplayName("Invalid boundaries")
    inner class InvalidBoundaries {
        @ParameterizedTest
        @CsvSource("17", "66")
        fun should_reject_when_ageIsInvalid(age: Int) { ... }
    }
}
```

## Test file location

- Source: `src/main/kotlin/com/example/service/UserRegistrationService.kt`
- Test: `src/test/kotlin/com/example/service/UserRegistrationServiceTest.kt`

## Naming convention

Same as Java: `should_<expectedBehavior>_when_<condition>`
