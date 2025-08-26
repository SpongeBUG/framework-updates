# Quick Reference & Cheat Sheet
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/QUICK_REFERENCE.md -->
<!-- Purpose: Quick lookup for common SDK 54 operations and commands -->
<!-- Dependencies: Expo SDK 54 -->

## Installation & Upgrade

```bash
# New Project
npx create-expo-app@latest --template default@next

# Upgrade Existing
npx expo install expo@next --fix

# Verify Installation  
npx expo --version
```

## Breaking Changes Quick Fix

### File System API
```typescript
// Quick fix: Use legacy API
import * as FileSystem from 'expo-file-system/legacy';

// Long-term: Migrate to new API
import { Directory, File } from 'expo-file-system';
const file = Directory.documentDirectory.createFile('data.json');
```

### SafeAreaView
```typescript
// Replace deprecated SafeAreaView
import { SafeAreaView, SafeAreaProvider } from 'react-native-safe-area-context';
```

### Notification Config
```json
// Move from app.json to plugins
{
  "expo": {
    "plugins": [
      ["expo-notifications", { "icon": "./assets/notification-icon.png" }]
    ]
  }
}
```

## Environment Requirements

| Component | Minimum Version |
|-----------|----------------|
| Node.js | 20.19.0+ |
| Xcode | 16.1+ |
| Android SDK | API 36+ |
| React Native | 0.81.0 |
| React | 19.1.0 |

## New Package Quick Start

### expo-app-integrity
```typescript
import * as AppIntegrity from 'expo-app-integrity';
const result = await AppIntegrity.checkIntegrityAsync();
console.log('App is genuine:', result.isGenuine);
```

### expo/blob
```typescript
import { Blob } from 'expo/blob';
const blob = new Blob(['Hello'], { type: 'text/plain' });
const text = await blob.text();
```

### localStorage API
```typescript
import { localStorage } from 'expo-sqlite/kv-store';
localStorage.setItem('key', 'value');
const value = localStorage.getItem('key');
```

## Platform Detection

```typescript
import { Platform } from 'react-native';

const PlatformInfo = {
  isIOS: Platform.OS === 'ios',
  isAndroid: Platform.OS === 'android',
  isTV: Platform.isTV,
  isiOS26Plus: Platform.OS === 'ios' && parseInt(Platform.Version) >= 26,
  isAndroid16Plus: Platform.OS === 'android' && Platform.Version >= 36,
  isAppleTV: Platform.OS === 'ios' && Platform.isTV,
  isAndroidTV: Platform.OS === 'android' && Platform.isTV,
};
```

## iOS 26 Liquid Glass

```typescript
import { Host, VStack, Button } from "@expo/ui/swift-ui";
import { glassEffect, padding } from '@expo/ui/build/swift-ui/modifiers';

<Host matchContents>
  <VStack modifiers={[
    padding({ all: 16 }),
    glassEffect({ glass: { variant: 'regular' }})
  ]}>
    <Button variant="glassProminent" color="blue">Action</Button>
  </VStack>
</Host>
```

## Android 16 Edge-to-Edge

```typescript
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function EdgeToEdgeScreen() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{
      flex: 1,
      paddingTop: insets.top,
      paddingBottom: insets.bottom
    }}>
      {/* Content */}
    </View>
  );
}
```

## EAS Build Configuration

### iOS 26 Features
```json
{
  "build": {
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0"
      }
    }
  }
}
```

### Store Submission Workaround
```json
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "ios": { "buildReactNativeFromSource": false }
      }]
    ]
  }
}
```

## Development Tools

### React Compiler Check
```bash
npx expo start
# Look for: "Experimental React Compiler is enabled."
# Press J → Components panel to see optimization status
```

### Import Stack Traces
```bash
# Enabled by default - shows import chains for missing modules
Error: Cannot resolve module 'missing-package'
Import stack:
  /src/components/Button.tsx
  /src/screens/HomeScreen.tsx
  /src/App.tsx
```

### Autolinking Verification  
```bash
npx expo-modules-autolinking verify -v
```

## Common Commands

```bash
# Clean everything
npx expo r -c

# Prebuild clean
npx expo prebuild --clean  

# Check config
npx expo config --type introspect

# Install dependencies
npx expo install --fix

# Start with specific platform
npx expo start --ios
npx expo start --android

# EAS Build
eas build --platform all --profile preview
```

## Performance Features

| Feature | Impact | Status |
|---------|---------|---------|
| Precompiled XCFrameworks | 90% faster iOS builds | Default |
| React Compiler | Auto-optimization | Default in new projects |
| lightningcss | 10x faster CSS processing | Default |
| New Architecture | Better runtime performance | 75% adoption |
| Import stack traces | Faster debugging | Default |

## Troubleshooting Quick Fixes

### Build Failures
```bash
# Node version issues
nvm install 20.19.0 && nvm use 20.19.0

# Xcode issues  
xcode-select --install

# Cache issues
npx expo r -c && npm install
```

### Runtime Errors
```bash
# Autolinking issues
npx expo prebuild --clean

# Metro cache
rm -rf node_modules/.cache

# Development client
npx expo install expo-dev-client
```

### iOS Simulator Issues
```bash
# Hotkeys not working in iOS 18.4+
# Use Expo CLI interactive prompt instead:
# r - reload, m - dev menu, j - developer menu
```

## Migration Checklist

- [ ] Update Node.js to 20.19+
- [ ] Update Xcode to 16.1+ (iOS)  
- [ ] Fix expo-file-system imports
- [ ] Replace SafeAreaView
- [ ] Update notification config
- [ ] Check Reanimated version
- [ ] Enable New Architecture
- [ ] Test edge-to-edge on Android
- [ ] Verify React Compiler enabled
- [ ] Test builds on both platforms

## New Architecture Transition

```json
// Enable New Architecture
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "newArchEnabled": true
      }]
    ]
  }
}
```

**Important:** SDK 54 is the final release with Legacy Architecture support. SDK 55 will require New Architecture.

## Support Resources

- **GitHub Issues:** https://github.com/expo/expo/issues
- **Discord:** https://chat.expo.dev/
- **Documentation:** https://docs.expo.dev/
- **Forums:** https://forums.expo.dev/

---

**Print this reference for quick lookup during development!**
