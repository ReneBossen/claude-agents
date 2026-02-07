# Testing React Native Frontend

## Scope

Jest + React Native Testing Library patterns for the Expo mobile app, including mock setup for native modules and third-party libraries.

## Mock Locations

| Location | Purpose |
|----------|---------|
| `Stepper.Mobile/jest.setup.js` | Global mocks for native modules and libraries |
| `Stepper.Mobile/__mocks__/` | Manual mock files for specific packages |
| Test file `jest.mock(...)` | Test-specific mocks scoped to a single file |

### Priority Order

Jest resolves mocks in this order:
1. `jest.mock()` in the test file (highest priority)
2. `__mocks__/` directory manual mocks
3. `jest.setup.js` global mocks

## react-native-paper Compound Components

react-native-paper uses compound components where sub-components are properties of a parent:

```tsx
<Dialog visible={show} onDismiss={onClose}>
  <Dialog.Title>Title</Dialog.Title>
  <Dialog.Content>Content</Dialog.Content>
  <Dialog.Actions>Actions</Dialog.Actions>
</Dialog>
```

### Mock Pattern

Compound components must have their sub-components explicitly defined as properties:

```javascript
// In __mocks__/react-native-paper.js or jest.mock()
const React = require('react');
const RN = require('react-native');

const Dialog = (props) => React.createElement(RN.View, props, props.children);
Dialog.Title = (props) => React.createElement(RN.Text, props, props.children);
Dialog.Content = (props) => React.createElement(RN.View, props, props.children);
Dialog.Actions = (props) => React.createElement(RN.View, props, props.children);
```

Without this, tests fail with:
```
TypeError: Cannot read properties of undefined (reading 'Title')
```

### Full react-native-paper Mock

The `__mocks__/react-native-paper.js` file should mock all used components:

```javascript
const React = require('react');
const RN = require('react-native');

// Simple component factory
const createComponent = (name) => {
  const Component = (props) =>
    React.createElement(RN.View, { testID: props.testID, ...props }, props.children);
  Component.displayName = name;
  return Component;
};

// Compound components
const Dialog = createComponent('Dialog');
Dialog.Title = createComponent('Dialog.Title');
Dialog.Content = createComponent('Dialog.Content');
Dialog.Actions = createComponent('Dialog.Actions');

const Appbar = createComponent('Appbar');
Appbar.Header = createComponent('Appbar.Header');
Appbar.Content = createComponent('Appbar.Content');
Appbar.Action = createComponent('Appbar.Action');
Appbar.BackAction = createComponent('Appbar.BackAction');

module.exports = {
  Dialog,
  Portal: ({ children }) => React.createElement(React.Fragment, null, children),
  Button: createComponent('Button'),
  TextInput: createComponent('TextInput'),
  Appbar,
  // Add other components as needed
};
```

## Native Module Mocks

Native modules that aren't available in the Jest environment must be mocked.

### @react-native-community/slider

```javascript
// In jest.setup.js
jest.mock('@react-native-community/slider', () => {
  const React = require('react');
  const RN = require('react-native');
  return React.forwardRef((props, ref) =>
    React.createElement(RN.View, { ref, testID: props.testID, ...props })
  );
});
```

### Common Native Modules

Add mocks in `jest.setup.js` for any module that causes import errors:

```javascript
// expo-secure-store
jest.mock('expo-secure-store', () => ({
  getItemAsync: jest.fn(),
  setItemAsync: jest.fn(),
  deleteItemAsync: jest.fn(),
}));

// @react-native-async-storage/async-storage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);
```

## Zustand Store Mocking

### Pattern: Mock the Store Module

```javascript
jest.mock('../../../store/groupsStore', () => ({
  useGroupsStore: jest.fn(() => ({
    groups: [],
    isLoading: false,
    error: null,
    fetchMyGroups: jest.fn(),
    createGroup: jest.fn(),
    // Include ALL state properties the component reads
  })),
}));
```

### Trap: Missing Store Properties

If a component reads `store.featuredGroups` and the mock doesn't include it, the component will get `undefined` and may crash during rendering. Always check which properties the component destructures from the store.

## API Client Test Patterns

### Match Full Call Signatures

When the implementation passes extra options (timeout, headers), tests must match:

```typescript
// Implementation:
const response = await apiClient.get('/users/me/data-export', { timeout: 120000 });

// Test assertion must include the config:
expect(mockApiClient.get).toHaveBeenCalledWith('/users/me/data-export', { timeout: 120000 });

// NOT just the path:
expect(mockApiClient.get).toHaveBeenCalledWith('/users/me/data-export');  // FAILS
```

### Match Request Bodies With Defaults

If the implementation adds default values to requests:

```typescript
// Implementation adds maxMembers default:
const request = { name: data.name, maxMembers: data.max_members ?? 5 };

// Test must expect the default value:
expect(mockPost).toHaveBeenCalledWith('/groups', expect.objectContaining({
  name: 'Test Group',
  maxMembers: 5,
}));
```

## Behavior Change Detection

When source code changes behavior (e.g., URL opens -> modal shows), tests must be updated:

```typescript
// OLD behavior: opens external URL
expect(Linking.openURL).toHaveBeenCalledWith('https://example.com/terms');

// NEW behavior: shows modal
fireEvent.press(getByTestId('settings-terms'));
await waitFor(() => {
  expect(getByTestId('legal-modal')).toBeTruthy();
});
```

Read the source component before writing or fixing tests to verify current behavior.

## Flaky Test Awareness

Some tests pass individually but fail in the full suite due to shared state or timing:

- `tokenStorage.test.ts` - Known flaky in parallel execution
- Fix: ensure `beforeEach` properly resets all mocked state
- Use `jest.clearAllMocks()` or `jest.resetAllMocks()` in `beforeEach`
