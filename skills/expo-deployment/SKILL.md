# Expo Deployment & Device Testing

## Scope

EAS builds, dev client workflow, config plugins, and physical device testing for React Native Expo apps.

## EAS Build Setup

### Installation

```bash
npm install -g eas-cli
eas login
```

### eas.json Profiles

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

- `development`: Dev client build with debugging — use for local testing
- `preview`: Production-like build for internal testing
- `production`: Store-ready build

### Build Commands

```bash
eas build --profile development --platform android
eas build --profile development --platform ios
```

### Slug Mismatch Error

If you see: `Slug for project identified by "extra.eas.projectId" does not match the "slug" field`

**Fix:** Run `eas init` to re-link the project with the current slug. This creates a new EAS project matching your `app.json` slug.

## Dev Client Workflow

A development build is a **custom version of Expo Go** that includes all your native modules. Required when using libraries not in Expo Go (e.g., `react-native-health-connect`, `react-native-health`).

### Start the Dev Server

```bash
npx expo start --dev-client
```

**NOT** `npx expo start` — that targets Expo Go, not your dev build.

### Connect a Device

1. Phone and PC must be on the **same Wi-Fi network**
2. Open the installed dev client app on your phone
3. Scan the QR code from the terminal

## Config Plugins

When third-party libraries need native code modifications not covered by their Expo plugin, create a custom config plugin.

### Pattern

```javascript
// plugins/withXxx.js
const { withMainActivity } = require('@expo/config-plugins');

const withXxx = (config) => {
  return withMainActivity(config, async (config) => {
    let mainActivity = config.modResults.contents;

    // Guard for idempotency
    if (!mainActivity.includes('SomeImport')) {
      mainActivity = mainActivity.replace(
        'import com.facebook.react.ReactActivity',
        'import some.package.SomeImport\nimport com.facebook.react.ReactActivity'
      );
    }

    config.modResults.contents = mainActivity;
    return config;
  });
};

module.exports = withXxx;
```

### Reference in app.json

```json
{
  "plugins": [
    "./plugins/withXxx"
  ]
}
```

### Available Config Plugin APIs

| API | Modifies |
|-----|----------|
| `withMainActivity` | `MainActivity.kt` |
| `withMainApplication` | `MainApplication.kt` |
| `withAndroidManifest` | `AndroidManifest.xml` |
| `withAppDelegate` | `AppDelegate.m/swift` |
| `withInfoPlist` | `Info.plist` |
| `withEntitlementsPlist` | Entitlements |

### Idempotency

Always guard modifications with `.includes()` checks so the plugin is safe to run multiple times during prebuild.

## Android Cleartext Traffic

Android 9+ blocks HTTP (non-HTTPS) traffic by default. For local dev server access:

```json
["expo-build-properties", {
  "android": {
    "minSdkVersion": 26,
    "usesCleartextTraffic": true
  }
}]
```

This is a **native config change** — requires a new build.

## Physical Device Testing (Android)

### Installing the APK

| Method | When to Use |
|--------|-------------|
| ADB | Best if USB debugging is set up |
| USB file transfer | Reliable, no ADB needed |
| Google Drive | No cable needed |
| Browser download | Direct from EAS build URL |

### ADB Install

```bash
adb install path\to\your-app.apk
```

**Prerequisites:**
- USB debugging enabled (Settings > Developer Options > USB Debugging)
- Developer mode active (tap Build Number 7 times in Settings > About Phone)
- USB cable that supports data transfer (not charge-only)
- Phone set to File Transfer / MTP mode

**Troubleshooting `adb devices` empty list:**
1. Try a different USB cable (most common cause)
2. Try a different USB port (prefer USB-A, avoid hubs)
3. Check for "Allow USB debugging?" popup on phone
4. Install OEM USB drivers (Samsung, Xiaomi, etc.)

### USB File Transfer

1. Connect phone via USB, set to File Transfer mode
2. Phone appears in File Explorer as a drive
3. Copy `.apk` to Download folder
4. Open Files app on phone, tap the `.apk`
5. Enable "Install unknown apps" for the file manager if prompted

## Native vs JS Changes

| Change Type | Action Required |
|-------------|-----------------|
| TypeScript/React code | Hot reload (no rebuild) |
| `app.json` plugin config | Full rebuild via `eas build` |
| New native dependency | Full rebuild via `eas build` |
| Custom config plugin | Full rebuild via `eas build` |
| `expo-build-properties` change | Full rebuild via `eas build` |
| Environment variables | Restart dev server |

**Rule of thumb:** If it affects native code or `app.json` plugins, rebuild. If it's JS/TS only, hot reload works.

## Troubleshooting Checklist

- [ ] Is `eas.json` present with the correct build profiles?
- [ ] Does `app.json` slug match the EAS project ID?
- [ ] Are all required plugins listed in `app.json` plugins array?
- [ ] Did you rebuild after native config changes?
- [ ] Is the dev server running with `--dev-client` flag?
- [ ] Are phone and PC on the same Wi-Fi network?
- [ ] For Android: is `usesCleartextTraffic` enabled for local HTTP?
