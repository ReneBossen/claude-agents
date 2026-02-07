# Test Maintenance

## Scope

Keeping tests in sync with source code changes. Applies to both backend (.NET) and frontend (React Native) test suites.

## The Core Problem

Source code changes without corresponding test updates cause:
1. **Compilation failures** (backend) - tests reference old signatures
2. **Silent logic failures** (frontend) - tests pass but verify old behavior
3. **False confidence** - broken tests that never run give the illusion of coverage

## After Changing Constructors or Method Signatures

### Backend (.NET)

When a service or controller constructor changes:

```bash
# Find all test files that instantiate the changed class
grep -rn "new UserService(" tests/
grep -rn "new UsersController(" tests/
```

Update every match. This typically means:
1. Add new `Mock<IDependency>` field
2. Initialize in constructor or setup
3. Pass `.Object` to the constructor in the correct position

### Frontend (TypeScript)

When a function signature changes:

```bash
# Find all test files that call or mock the changed function
grep -rn "createGroup\|mockCreateGroup" src/ --include="*.test.*"
```

Update mock return values and call expectations to match new parameters.

## After Changing API Contracts

### Backend: Request/Response DTOs

When adding a field to a DTO:
- Tests that construct the DTO need the new field
- Tests that assert on the response need updated expectations
- Mock setups returning the DTO need the new field

### Frontend: API Client Functions

When the implementation adds default values, timeout configs, or new fields:
- Test mock expectations must include the new values
- Use `expect.objectContaining()` for partial matching when appropriate

## After Changing UI Behavior

When a screen's behavior changes (e.g., from external links to modals):

1. Read the current source component
2. Identify what the user action now triggers
3. Rewrite test assertions to verify current behavior
4. Remove mocks for old behavior (e.g., `Linking.openURL`)
5. Add mocks/queries for new behavior (e.g., modal visibility)

## After Changing Error Handling

When error messages change in controllers or services:
- Update all test assertions that check error message strings
- Prefer generic messages over leaked internal details

## Pre-Commit Checklist

Before committing any source code change:

```bash
# Backend
dotnet build tests/Stepper.UnitTests/Stepper.UnitTests.csproj --verbosity quiet
dotnet test tests/Stepper.UnitTests/Stepper.UnitTests.csproj --verbosity quiet

# Frontend
cd Stepper.Mobile && npx jest --passWithNoTests
```

If tests fail, determine if the failure is:
- **Compilation error** -> constructor or signature changed, fix test setup
- **Assertion failure** -> behavior changed, fix test expectations
- **Mock error** -> dependency added or return type changed, fix mock configuration

## Warning Signs

| Symptom | Likely Cause |
|---------|-------------|
| `CS7036: There is no argument given that corresponds to...` | Constructor parameter added to source, not to test |
| `Expected X but received Y` (different exception type) | Service execution order changed, mock returns null before reaching expected code |
| `Expected call count 1, received 0` | Method was renamed, moved, or implementation path changed |
| `TypeError: Cannot read properties of undefined` | Component compound property not mocked (e.g., `Dialog.Title`) |
| `toHaveBeenCalledWith` mismatch | Implementation now sends additional fields or config |
