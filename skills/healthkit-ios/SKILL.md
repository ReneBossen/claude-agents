# HealthKit (iOS)

## Scope

iOS HealthKit integration for step tracking using `react-native-health` in a React Native Expo app.

## Library

- Package: `react-native-health` (v1.x)
- Bridge to Apple's HealthKit framework
- **Callback-based API** — must wrap in Promises for async/await

## Expo Plugin Setup

### app.json Plugin

```json
["react-native-health", {
  "healthSharePermission": "App needs access to your health data to track steps.",
  "healthUpdatePermission": "App needs permission to save health data."
}]
```

### iOS Configuration in app.json

```json
"ios": {
  "bundleIdentifier": "com.your.app",
  "infoPlist": {
    "NSHealthShareUsageDescription": "App needs access to health data to track steps.",
    "NSHealthUpdateUsageDescription": "App needs permission to save step data.",
    "UIBackgroundModes": ["processing"]
  },
  "entitlements": {
    "com.apple.developer.healthkit": true,
    "com.apple.developer.healthkit.background-delivery": true
  }
}
```

Both `infoPlist` descriptions AND the plugin `healthSharePermission`/`healthUpdatePermission` are required.

## Availability Check

HealthKit is only available on iOS devices (not iPad simulators without Health app).

```typescript
import AppleHealthKit from 'react-native-health';

async isAvailable(): Promise<boolean> {
  return new Promise((resolve) => {
    try {
      AppleHealthKit.isAvailable((error, available) => {
        if (error) { resolve(false); return; }
        resolve(available);
      });
    } catch (error) {
      resolve(false);
    }
  });
}
```

## Permissions

### Configuration

```typescript
import AppleHealthKit, { HealthKitPermissions } from 'react-native-health';

const permissions: HealthKitPermissions = {
  permissions: {
    read: [
      AppleHealthKit.Constants.Permissions.StepCount,
      AppleHealthKit.Constants.Permissions.DistanceWalkingRunning,
    ],
    write: [],  // Only request what you actually need
  },
};
```

Only request `write` permissions if the app needs to write data. Unnecessary write requests reduce user trust.

### Authorization — Apple Privacy Model

**This is the most important thing to understand about HealthKit.**

```typescript
async requestAuthorization(): Promise<AuthorizationStatus> {
  return new Promise((resolve) => {
    AppleHealthKit.initHealthKit(permissions, (error) => {
      if (error) {
        resolve('not_available');
        return;
      }
      resolve('authorized');
    });
  });
}
```

**Critical limitations:**
1. `initHealthKit` **always succeeds** even if the user denies ALL read permissions
2. Apple does NOT allow checking read permission status — only write permission status can be queried
3. If the user denies read access, data queries simply return empty results (no error)
4. There is no way to programmatically determine if the user granted read access

**Implication:** Track initialization state with an internal `isInitialized` flag. Accept that "authorized" really means "the permission dialog was shown" — not that access was actually granted.

## Reading Step Data

### Daily Step Count

```typescript
import AppleHealthKit, { HealthInputOptions, HealthValue } from 'react-native-health';

const options: HealthInputOptions = {
  startDate: startDate.toISOString(),
  endDate: endDate.toISOString(),
  includeManuallyAdded: true,
};

AppleHealthKit.getDailyStepCountSamples(options, (error, results: HealthValue[]) => {
  if (error) { /* handle */ return; }
  // results: Array<{ startDate: string, value: number, ... }>
  // IMPORTANT: May return multiple samples per day — must aggregate
});
```

### Daily Distance

```typescript
import { HealthUnit } from 'react-native-health';

const options: HealthInputOptions = {
  startDate: startDate.toISOString(),
  endDate: endDate.toISOString(),
  unit: HealthUnit.meter,
  includeManuallyAdded: true,
};

AppleHealthKit.getDailyDistanceWalkingRunningSamples(options, (error, results) => {
  // Same structure as steps — aggregate by date
});
```

### Date Aggregation Pattern

HealthKit may return multiple samples per day. Always aggregate:

```typescript
const stepsByDate = new Map<string, number>();

for (const sample of results) {
  const date = sample.startDate.split('T')[0]; // Extract YYYY-MM-DD
  const current = stepsByDate.get(date) ?? 0;
  stepsByDate.set(date, current + (sample.value ?? 0));
}
```

### Parallel Fetching

Fetch steps and distance simultaneously:

```typescript
const [steps, distances] = await Promise.all([
  this.fetchDailySteps(startDate, endDate),
  this.fetchDailyDistance(startDate, endDate),
]);
```

## Disconnect Behavior

- No programmatic way to revoke HealthKit permissions
- Users manage permissions via **iOS Settings > Privacy & Security > Health**
- Disconnect only resets internal state:

```typescript
async disconnect(): Promise<void> {
  this.isInitialized = false;
  // No actual revocation possible
}
```

## Architecture Pattern

```
healthProviderFactory.ts  →  healthKitService.ts (implements HealthDataProvider)
                          →  googleFitService.ts (Android equivalent)

unifiedStepTrackingService.ts  →  wraps platform provider
                               →  manages sync state
                               →  calls backend API
```

- `HealthDataProvider` interface: `isAvailable()`, `getAuthorizationStatus()`, `requestAuthorization()`, `getStepData()`, `disconnect()`
- Factory pattern: `createHealthKitService()`
- Data format: `{ date: 'YYYY-MM-DD', stepCount, distanceMeters, source: 'healthkit' }`

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Callback API | Type errors or hanging promises | Wrap every HealthKit call in `new Promise()` |
| Read permission check | Trying to verify read access | Impossible by design — Apple privacy model |
| `initHealthKit` "succeeds" on denial | App thinks it has access but gets empty data | Accept this behavior; empty data = likely denied |
| Multiple samples per day | Step count is much higher than expected | Always aggregate by date using a Map |
| Date format | Mismatched dates or off-by-one errors | Extract date with `split('T')[0]` from ISO timestamps |
| Background delivery | Background sync not working | Need both `UIBackgroundModes: ["processing"]` AND `healthkit.background-delivery` entitlement |
| Missing write permission request | Asking for write when only reading | Only request write in `permissions.write` if actually needed |
