# C# / .NET Test Guide

Code patterns for generating tests with xUnit, FluentAssertions, and Moq.

## Dependencies

```xml
<!-- .csproj -->
<PackageReference Include="xunit" Version="2.*" />
<PackageReference Include="xunit.runner.visualstudio" Version="2.*" />
<PackageReference Include="FluentAssertions" Version="6.*" />
<PackageReference Include="Moq" Version="4.*" />
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
```

## Test class structure

```csharp
using Xunit;
using Moq;
using FluentAssertions;

namespace MyApp.Tests.Services;

[Trait("Requirement", "User must be aged 18-65 to register")]
[Trait("Source", "test-cases/user-registration-age.md")]
[Trait("Level", "Unit Test")]
public class UserRegistrationServiceTests
{
    private const string Requirement = "User must be aged 18-65 to register";
    private const string Source = "test-cases/user-registration-age.md";

    private readonly Mock<IUserRepository> _mockRepository;
    private readonly UserRegistrationService _service;

    public UserRegistrationServiceTests()
    {
        _mockRepository = new Mock<IUserRepository>();
        _service = new UserRegistrationService(_mockRepository.Object);
    }
}
```

## Traceability in test reports

Use `[Trait]` attributes for categorization and filtering, and display names
for readability in the test report:

```csharp
[Fact(DisplayName = "UT-01: Accept registration when age is at minimum boundary (18)")]
[Trait("Requirement", "User must be aged 18-65 to register")]
[Trait("Technique", "BVA")]
[Trait("TC-ID", "UT-01")]
[Trait("Level", "Unit Test")]
public void Should_AcceptRegistration_When_AgeIsAtMinimumBoundary()
{
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Test Case: Age at minimum boundary (18)
    // Technique: BVA (C1:Min) | Source: test-cases/user-registration-age.md
    ...
}
```

This produces in `dotnet test -v n`:
```
Passed  UT-01: Accept registration when age is at minimum boundary (18)
  Traits: Requirement=User must be aged 18-65 to register, Technique=BVA, TC-ID=UT-01
```

Filter by requirement: `dotnet test --filter "Requirement=User must be aged 18-65"`
Filter by technique: `dotnet test --filter "Technique=BVA"`

## Test method patterns

### Single test

```csharp
[Fact(DisplayName = "UT-01: Accept registration when age is at minimum boundary (18)")]
[Trait("Technique", "BVA")]
[Trait("TC-ID", "UT-01")]
public void Should_AcceptRegistration_When_AgeIsAtMinimumBoundary()
{
    // Requirement: User must be aged 18-65 to register
    // TC-ID: UT-01 | Technique: BVA (C1:Min)
    var request = new RegistrationRequest("Sarah Lee", 18, "sarah@test.com");

    // When
    var result = _service.Register(request);

    // Then
    result.IsAccepted.Should().BeTrue();
    result.Message.Should().Be("Registration accepted, welcome page shown");
}
```

### Parameterized test (Theory)

```csharp
[Theory]
[InlineData("UT-02", 18, "Min boundary")]
[InlineData("UT-03", 19, "Above min")]
[InlineData("UT-04", 64, "Below max")]
[InlineData("UT-05", 65, "Max boundary")]
public void Should_AcceptRegistration_When_AgeIsWithinValidRange(
    string utId, int age, string description)
{
    // Given - Valid BVA boundary values
    var request = new RegistrationRequest("Test User", age, "test@test.com");

    // When
    var result = _service.Register(request);

    // Then
    result.IsAccepted.Should().BeTrue(
        because: $"{utId}: Age {age} ({description}) should be accepted");
}
```

### Exception test

```csharp
[Fact]
public void Should_ThrowValidationException_When_AgeIsBelowMinimum()
{
    // Given - UT-01: Age below minimum (17)
    var request = new RegistrationRequest("Tom Park", 17, "tom@test.com");

    // When
    var act = () => _service.Register(request);

    // Then
    act.Should().Throw<ValidationException>()
       .WithMessage("Age must be between 18 and 65");
}
```

## Grouping with nested classes

```csharp
public class UserRegistrationServiceTests
{
    // shared setup...

    [Trait("Category", "BVA")]
    public class AgeValidation : UserRegistrationServiceTests
    {
        [Theory]
        [InlineData(18)]
        [InlineData(19)]
        [InlineData(64)]
        [InlineData(65)]
        public void Should_Accept_When_AgeIsValid(int age) { ... }

        [Theory]
        [InlineData(17)]
        [InlineData(66)]
        public void Should_Reject_When_AgeIsInvalid(int age) { ... }
    }

    [Trait("Category", "EP")]
    public class FileTypeValidation : UserRegistrationServiceTests
    {
        public class ValidPartitions : FileTypeValidation { ... }
        public class InvalidPartitions : FileTypeValidation { ... }
    }

    [Trait("Category", "StateTransition")]
    public class OrderStateTransitions : UserRegistrationServiceTests
    {
        public class ValidTransitions : OrderStateTransitions { ... }
        public class InvalidTransitions : OrderStateTransitions { ... }
    }
}
```

## Moq patterns

```csharp
// Setup
_mockRepository.Setup(r => r.FindByEmail("test@test.com"))
    .Returns((User?)null);

_mockRepository.Setup(r => r.Save(It.IsAny<User>()))
    .Returns(true);

// Verify
_mockRepository.Verify(r => r.Save(It.Is<User>(u =>
    u.Name == "Sarah Lee" && u.Age == 18
)), Times.Once);

_mockRepository.Verify(r => r.Save(It.IsAny<User>()), Times.Never);
```

## FluentAssertions patterns

```csharp
// Boolean
result.IsAccepted.Should().BeTrue();
result.IsAccepted.Should().BeFalse();

// String
result.Message.Should().Be("Expected message");
result.Message.Should().Contain("partial match");
result.Message.Should().StartWith("Error:");

// Number
result.Amount.Should().Be(30000m);
result.Age.Should().BeInRange(18, 65);

// Object
result.Status.Should().Be(OrderStatus.Approved);
result.Should().BeEquivalentTo(expectedResult);

// Collection
result.Errors.Should().HaveCount(1);
result.Errors.Should().Contain("Age must be between 18 and 65");

// Exception
act.Should().Throw<ValidationException>()
   .WithMessage("Invalid input");

// Async exception
await act.Should().ThrowAsync<ValidationException>();
```

## Test file location

- Source: `MyApp/Services/UserRegistrationService.cs`
- Test: `MyApp.Tests/Services/UserRegistrationServiceTests.cs`

## Naming convention

`Should_<ExpectedBehavior>_When_<Condition>` (PascalCase)
