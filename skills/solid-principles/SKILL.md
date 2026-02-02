# SOLID Principles

## S - Single Responsibility Principle

> A class should have only one reason to change.

**Apply to:**
- **Controllers**: HTTP handling only
- **Services**: Business logic only
- **Repositories**: Data access only

**Example:**
```csharp
// BAD: Controller with business logic
public async Task<ActionResult> CreateOrder(OrderRequest request)
{
    // Validation logic here
    // Business rules here
    // Database calls here
    // Email sending here
}

// GOOD: Controller delegates to service
public async Task<ActionResult> CreateOrder(OrderRequest request)
{
    var result = await _orderService.CreateOrderAsync(request);
    return Ok(result);
}
```

## O - Open/Closed Principle

> Open for extension, closed for modification.

**Apply by:**
- Using interfaces and abstractions
- Adding new features via new classes, not modifying existing ones
- Using strategy/decorator patterns for varying behavior

**Example:**
```csharp
// GOOD: Extend via new implementations
public interface INotificationSender
{
    Task SendAsync(Notification notification);
}

public class EmailNotificationSender : INotificationSender { }
public class PushNotificationSender : INotificationSender { }  // Extension!
```

## L - Liskov Substitution Principle

> Subtypes must be substitutable for their base types.

**Apply by:**
- Not breaking interface contracts in implementations
- Derived classes honoring base class invariants
- Avoiding unexpected side effects in overrides

## I - Interface Segregation Principle

> Clients should not be forced to depend on methods they don't use.

**Apply by:**
- Preferring small, focused interfaces
- Splitting large interfaces into role-specific ones
- Creating interfaces based on client needs, not implementation convenience

**Example:**
```csharp
// BAD: Fat interface
public interface IUserService
{
    Task<User> GetUserAsync(Guid id);
    Task UpdateUserAsync(User user);
    Task DeleteUserAsync(Guid id);
    Task SendPasswordResetAsync(string email);
    Task ValidateEmailAsync(string token);
}

// GOOD: Segregated interfaces
public interface IUserReader { Task<User> GetUserAsync(Guid id); }
public interface IUserWriter { Task UpdateUserAsync(User user); Task DeleteUserAsync(Guid id); }
public interface IPasswordService { Task SendPasswordResetAsync(string email); }
```

## D - Dependency Inversion Principle

> Depend on abstractions, not concretions.

**Apply by:**
- Constructor injection only
- Depending on interfaces, not concrete classes
- High-level modules defining interfaces, low-level modules implementing them

**Example:**
```csharp
// GOOD: Depend on abstraction
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;  // Interface, not concrete

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;  // Injected, not created
    }
}
```

## Quick Reference

| Principle | One-liner | Check |
|-----------|-----------|-------|
| **S**ingle Responsibility | One reason to change | Can you describe the class purpose in one sentence? |
| **O**pen/Closed | Extend, don't modify | Adding features without changing existing code? |
| **L**iskov Substitution | Subtypes are substitutable | Can any implementation replace the interface? |
| **I**nterface Segregation | Small, focused interfaces | Do clients use all interface methods? |
| **D**ependency Inversion | Depend on abstractions | Are dependencies injected via interfaces? |
