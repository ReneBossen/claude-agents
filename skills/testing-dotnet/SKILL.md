# Testing .NET Backend

## Scope

Patterns for xUnit + Moq + FluentAssertions testing in .NET backend services following Screaming Architecture.

## Constructor Signatures and Test Synchronization

When a service constructor changes (new dependencies added), **every test file** that instantiates the service will fail to compile.

### Pattern: Full Constructor Setup

Always construct services with all required dependencies, even if the test doesn't use them:

```csharp
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _mockUserRepo;
    private readonly Mock<IStepRepository> _mockStepRepo;
    // ... ALL dependencies as mock fields
    private readonly UserService _sut;

    public UserServiceTests()
    {
        _mockUserRepo = new Mock<IUserRepository>();
        _mockStepRepo = new Mock<IStepRepository>();
        // ... initialize ALL mocks
        _sut = new UserService(
            _mockUserRepo.Object,
            _mockStepRepo.Object,
            // ... pass ALL mocks in constructor order
        );
    }
}
```

### Rule: After Changing a Constructor

When adding or removing a service/controller constructor parameter:

1. Search for all test files that instantiate the class: `grep -rn "new ClassName(" tests/`
2. Update every constructor call in every test file
3. Add mock fields and initialization for any new dependencies
4. Verify tests compile: `dotnet build tests/Stepper.UnitTests/Stepper.UnitTests.csproj`

## Moq Default Behavior

Unconfigured Moq mocks return **default values**:

| Type | Default Return |
|------|---------------|
| Reference types | `null` |
| `int`, `long` | `0` |
| `bool` | `false` |
| `Guid` | `Guid.Empty` |
| `Task<T>` | `Task.FromResult(default(T))` |
| Collections | `null` (not empty list) |

### Trap: Null Returns Cause Wrong Exceptions

If a service method does this:

```csharp
public async Task UpdateAsync(Guid userId, UpdateRequest request)
{
    var entity = await _repository.GetByIdAsync(userId);  // Returns null from unconfigured mock
    if (entity == null) throw new KeyNotFoundException();  // Throws HERE
    ValidateRequest(request);  // Never reaches validation
}
```

A test expecting `ArgumentException` from validation will get `KeyNotFoundException` instead.

### Rule: Configure Mocks for the Happy Path First

For validation tests, set up the mock to return a valid object so the service reaches the validation code:

```csharp
[Fact]
public async Task Update_WithInvalidName_ThrowsArgumentException()
{
    var userId = Guid.NewGuid();
    var existingEntity = new User { Id = userId, DisplayName = "Valid" };

    // Configure mock so service gets past the null check
    _mockRepo.Setup(x => x.GetByIdAsync(userId))
        .ReturnsAsync(existingEntity);

    var request = new UpdateRequest { DisplayName = "A" }; // Too short

    var act = async () => await _sut.UpdateAsync(userId, request);

    await act.Should().ThrowAsync<ArgumentException>();
    _mockRepo.Verify(x => x.GetByIdAsync(userId), Times.Once);
}
```

## Controller Error Message Testing

Controllers should return **generic error messages** in catch-all handlers to avoid leaking internal details.

### Pattern: Test the Actual Message

```csharp
// The controller returns:
catch (Exception ex)
{
    _logger.LogError(ex, "...");
    return StatusCode(500, ApiResponse<T>.ErrorResponse(
        "An unexpected error occurred. Please try again later."));
}

// The test should match exactly:
[Fact]
public async Task Action_WhenServiceThrows_ReturnsInternalServerError()
{
    _mockService.Setup(x => x.DoSomethingAsync(It.IsAny<Guid>()))
        .ThrowsAsync(new Exception("Database connection failed"));

    var result = await _sut.Action();

    var objectResult = result.Result.Should().BeOfType<ObjectResult>().Subject;
    objectResult.StatusCode.Should().Be(500);
    var response = objectResult.Value.Should().BeOfType<ApiResponse<T>>().Subject;

    // Match the ACTUAL generic message, not the internal exception
    response.Errors.Should().Contain("An unexpected error occurred. Please try again later.");
}
```

### Anti-pattern: Never expect internal details in error responses

```csharp
// WRONG - expects leaked exception message
response.Errors.Should().Contain("An error occurred: Database connection failed");

// CORRECT - expects the generic user-facing message
response.Errors.Should().Contain("An unexpected error occurred. Please try again later.");
```

## Tuple Return Types in Mocks

When mocking methods that return named tuples, match the types exactly:

```csharp
// Interface method returns:
Task<(List<Notification> Notifications, int TotalCount, int UnreadCount)> GetAllAsync(...);

// Mock setup must match all tuple element types:
_mock.Setup(x => x.GetAllAsync(userId, It.IsAny<int>(), It.IsAny<int>()))
    .ReturnsAsync((new List<Notification>(), 0, 0));  // int, not bool

// WRONG - bool instead of int for UnreadCount:
    .ReturnsAsync((new List<Notification>(), 0, false));  // Compilation error
```

## FluentAssertions Async Patterns

```csharp
// For async methods that should throw:
var act = async () => await _sut.MethodAsync(args);
await act.Should().ThrowAsync<ArgumentException>()
    .WithMessage("Expected message*");  // Wildcard matching

// For sync constructors that should throw:
var act = () => new Service(null!);
act.Should().Throw<ArgumentNullException>();
```

## Verify Call Counts Accurately

Update `Times.Never` / `Times.Once` assertions to match the actual execution path:

```csharp
// If validation happens AFTER a repository call, the repo IS called
_mockRepo.Verify(x => x.GetByIdAsync(userId), Times.Once);  // Not Times.Never

// If validation happens BEFORE a repository call (e.g., empty GUID check)
_mockRepo.Verify(x => x.GetByIdAsync(It.IsAny<Guid>()), Times.Never);
```
