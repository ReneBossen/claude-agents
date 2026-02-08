# Health Connect (Android)

## Scope

Android Health Connect integration for step tracking using `react-native-health-connect` in a React Native Expo app.

## Library

- Package: `react-native-health-connect` (v3.x)
- Bridge to Android's Health Connect API (successor to Google Fit)
- Key imports: `initialize`, `requestPermission`, `readRecords`, `getSdkStatus`, `SdkAvailabilityStatus`

## Expo Plugin Setup

### The Built-in Plugin Is Incomplete

The library's `app.plugin.js` **only** adds an intent filter to `AndroidManifest.xml`. It does NOT register the permission delegate in `MainActivity.kt`.

Without the fix below, requesting permissions crashes with:
```
kotlin.UninitializedPropertyAccessException: lateinit property requestPermission has not been initialized
```

### Required Custom Config Plugin

Create `plugins/withHealthConnectPermissions.js`:

```javascript
const { withMainActivity } = require('@expo/config-plugins');

const withHealthConnectPermissions = (config) => {
  return withMainActivity(config, async (config) => {
    let mainActivity = config.modResults.contents;

    // Add import
    if (!mainActivity.includes('HealthConnectPermissionDelegate')) {
      mainActivity = mainActivity.replace(
        'import com.facebook.react.ReactActivity',
        'import dev.matinzd.healthconnect.permissions.HealthConnectPermissionDelegate\nimport com.facebook.react.ReactActivity'
      );
    }

    // Register permission delegate in onCreate
    if (!mainActivity.includes('HealthConnectPermissionDelegate.setPermissionDelegate')) {
      mainActivity = mainActivity.replace(
        'super.onCreate(null)',
        'super.onCreate(null)\n    HealthConnectPermissionDelegate.setPermissionDelegate(this)'
      );
    }

    config.modResults.contents = mainActivity;
    return config;
  });
};

module.exports = withHealthConnectPermissions;
```

### app.json Configuration

```json
{
  "plugins": [
    ["expo-build-properties", {
      "android": { "minSdkVersion": 26 }
    }],
    ["react-native-health-connect", {
      "requestPermissionsRationale": "App needs access to health data to track steps."
    }],
    "./plugins/withHealthConnectPermissions"
  ],
  "android": {
    "permissions": ["android.permission.ACTIVITY_RECOGNITION"]
  }
}
```

**Order matters:** `react-native-health-connect` plugin first (adds manifest entries), then `withHealthConnectPermissions` (adds MainActivity code).

## Health Connect App Requirement

| Android Version | Health Connect Status |
|-----------------|----------------------|
| Android 14+ | Built-in, no action needed |
| Android 9-13 | Must install from Google Play Store |

If not installed, `getSdkStatus()` returns `SDK_UNAVAILABLE` or `SDK_UNAVAILABLE_PROVIDER_UPDATE_REQUIRED`.

Play Store package: `com.google.android.apps.healthdata`

## SDK Availability Check

```typescript
import { getSdkStatus, SdkAvailabilityStatus } from 'react-native-health-connect';

async isAvailable(): Promise<boolean> {
  try {
    const status = await getSdkStatus();
    return status === SdkAvailabilityStatus.SDK_AVAILABLE;
  } catch (error) {
    console.warn('Health Connect SDK check failed:', error);
    return false;
  }
}
```

## Initialization

**Must call `initialize()` before any other Health Connect operation.**

```typescript
import { initialize } from 'react-native-health-connect';

const initialized = await initialize();
if (!initialized) {
  // Health Connect is not properly set up
}
```

## Permission Flow

### Requesting Permissions

```typescript
import { requestPermission } from 'react-native-health-connect';

const permissions = await requestPermission([
  { accessType: 'read', recordType: 'Steps' },
  { accessType: 'read', recordType: 'Distance' },
]);

// Verify specific permissions
const hasSteps = permissions.some(
  (p) => p.recordType === 'Steps' && p.accessType === 'read'
);
```

### Permission Types

Health Connect permissions are **separate** from regular Android permissions:

| Permission | What It Grants |
|-----------|---------------|
| `ACTIVITY_RECOGNITION` (Android) | Access to device motion sensors directly |
| Health Connect Steps (read) | Access to step data in Health Connect data store |

Users must grant **both**: the Android permission AND the Health Connect data permission.

### Where Permissions Are Managed

- Health Connect permissions: Through the Health Connect app > App permissions
- NOT through regular Android Settings > App permissions
- `Linking.openSettings()` opens Android Settings, NOT Health Connect — consider linking to Health Connect directly

## Reading Step Data

```typescript
import { readRecords } from 'react-native-health-connect';

const result = await readRecords('Steps', {
  timeRangeFilter: {
    operator: 'between',
    startTime: startDate.toISOString(),
    endTime: endDate.toISOString(),
  },
});

// Records have: startTime, endTime, count
// Aggregate by date for daily totals
for (const record of result.records) {
  const date = new Date(record.startTime).toISOString().split('T')[0];
  // Sum record.count per date
}
```

### Reading Distance

```typescript
const distanceResult = await readRecords('Distance', {
  timeRangeFilter: { ... },
});

// Distance records have: distance.inMeters
for (const record of distanceResult.records) {
  const meters = record.distance?.inMeters ?? 0;
}
```

## Architecture Pattern

```
healthProviderFactory.ts  →  googleFitService.ts (implements HealthDataProvider)
                          →  healthKitService.ts (iOS equivalent)

unifiedStepTrackingService.ts  →  wraps platform provider
                               →  manages sync state
                               →  calls backend API
```

- `HealthDataProvider` interface: `isAvailable()`, `getAuthorizationStatus()`, `requestAuthorization()`, `getStepData()`, `disconnect()`
- Factory pattern: `createGoogleFitService()` / `createHealthKitService()`
- Platform detection: `Platform.OS === 'android'` in factory
- Data format: `{ date: 'YYYY-MM-DD', stepCount, distanceMeters, source: 'googlefit' }`

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Missing permission delegate | `lateinit property requestPermission` crash | Add custom config plugin (see above) |
| Health Connect not installed | `isAvailable()` returns false on Android < 14 | Install Health Connect from Play Store |
| Initialize not called | Various crashes or silent failures | Always call `initialize()` before other operations |
| Wrong permission type | User enables "Physical Activity" but still denied | Health Connect permissions are separate from Android permissions |
| Config plugin not rebuilt | Fix applied but crash persists | Config plugin changes require `eas build`, not just reload |
| `setPermissionDelegate` with wrong package | Permission dialog doesn't appear | Call without second arg to use default Google Health Connect provider |
