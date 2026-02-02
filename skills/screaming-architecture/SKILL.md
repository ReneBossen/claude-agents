# Screaming Architecture

## Core Principle

> "Your architecture should scream the intent of the system, not the frameworks you used."
> — Robert C. Martin

When you look at a project structure, you should immediately see the **business domain**, not technical layers.

## Vertical Slice Organization

Each feature is a **vertical slice** containing everything needed:

```
{FeatureName}/
├── {FeatureName}Controller.cs   # HTTP endpoints (thin)
├── {FeatureName}Service.cs      # Business logic
├── I{FeatureName}Service.cs     # Service interface
├── {FeatureName}Repository.cs   # Data access
├── I{FeatureName}Repository.cs  # Repository interface
├── {DomainModel}.cs             # Domain model (clean)
├── {DomainModel}Entity.cs       # Database entity (internal)
└── DTOs/                        # Request/Response models
```

## Benefits

- **Feature cohesion**: Everything related to a feature is in one place
- **Easy navigation**: Developers find code by business concept
- **Clear boundaries**: Each slice is independent and focused
- **Minimal coupling**: Features interact through well-defined interfaces
- **Easier onboarding**: New developers immediately understand the system

## Dependency Rules

### Within a Feature Slice
```
Controller → Service → Repository → Database Client
         ↓
       DTOs
```

### Cross-Feature Dependencies
- Features should be **loosely coupled**
- If Feature A needs data from Feature B:
  - Option 1: Service-to-Service via interface
  - Option 2: Direct database queries (read-only)
  - Option 3: Event-based communication

### Common Dependencies
- All features can depend on `/Common`
- `/Common` should NOT depend on any feature
- Shared infrastructure goes in `/Common`

## Adding New Features

1. Create a new folder at root level: `{ProjectName}/{FeatureName}/`
2. Add vertical slice components:
   - Controller (HTTP endpoints)
   - Service interface + implementation (business logic)
   - Repository interface + implementation (data access)
   - Domain models
   - DTOs
3. Register services in dependency injection
4. Add database tables/migrations if needed
5. Add tests in corresponding test folders

## Anti-Patterns to Avoid

- **Technical layering at root**: Don't organize by `Controllers/`, `Services/`, `Repositories/`
- **Fat controllers**: Controllers should only handle HTTP, delegate to services
- **Anemic services**: Business logic belongs in services, not scattered around
- **Cross-feature coupling**: Features should not directly reference each other's internals
- **God objects**: Each class should have a single responsibility
