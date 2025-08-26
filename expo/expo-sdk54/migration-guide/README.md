# Migration Guide
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/migration-guide/README.md -->
<!-- Purpose: Step-by-step upgrade instructions and troubleshooting -->
<!-- Dependencies: Node.js 20.19+, Xcode 16.1+, React Native 0.81 -->

## Pre-Migration Checklist

### Environment Requirements
```bash
# 1. Check Node.js version (required: 20.19+)
node --version

# 2. Update Node.js if needed
# Via nvm (recommended)
nvm install 20.19.0
nvm use 20.19.0

# Via direct download
# Download from https://nodejs.org/
```

### Development Tools
```bash
# 3. Update Xcode (iOS development)
# Install Xcode 16.1+ from App Store or Developer Portal
xcode-select --install

# 4. Update Android SDK (Android development)
# Update via Android Studio SDK Manager to API 36
```

### Project Backup
```bash
# 5. Backup your project
git add . && git commit -m "Pre-SDK54 backup"
git checkout -b sdk54-migration
```

## Migration Steps

### Step 1: Update Expo SDK
```bash
# Update to SDK 54 beta
npx expo install expo@next --fix

# Verify installation
npx expo --version
```

### Step 2: Handle Breaking Changes

#### A. File System API Migration
**Required Action:** Update all `expo-file-system` imports

```typescript
// Find all imports in your project
grep -r "expo-file-system" src/

// Option 1: Quick fix (use legacy API)
// Replace all:
import * as FileSystem from 'expo-file-system';
// With:
import * as FileSystem from 'expo-file-system/legacy';

// Option 2: Migrate to new API (recommended long-term)
import { Directory, File } from 'expo-file-system';
```

#### B. New Architecture Check
```bash
# Check if New Architecture is enabled
npx expo config --type introspect | grep newArchEnabled

# If false, enable it (recommended)
# Add to app.json:
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

#### C. Reanimated Version Check
```bash
# Check Reanimated version
npm list react-native-reanimated

# If using New Architecture:
npm install react-native-reanimated@^4.0.0

# If staying on Legacy Architecture (temporary):
npm install react-native-reanimated@^3.15.0
```

#### D. JSC Engine Migration
```javascript
// Check current engine in app.json
{
  "expo": {
    "jsEngine": "jsc" // If present, needs action
  }
}

// Option 1: Switch to Hermes (recommended)
{
  "expo": {
    "jsEngine": "hermes"
  }
}

// Option 2: Use community JSC
npm install @react-native-community/jsc
// Requires manual native configuration
```

### Step 3: Update Deprecated APIs

#### A. SafeAreaView Replacement
```typescript
// Find usage
grep -r "SafeAreaView.*react-native" src/

// Replace:
import { SafeAreaView } from 'react-native';

// With:
import { SafeAreaView, SafeAreaProvider } from 'react-native-safe-area-context';

// Wrap app with provider:
function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView>
        {/* Your app */}
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

#### B. Notification Configuration
```json
// Remove from app.json:
{
  "expo": {
    "notification": {
      "icon": "./assets/notification-icon.png"
    }
  }
}

// Add to plugins:
{
  "expo": {
    "plugins": [
      ["expo-notifications", {
        "icon": "./assets/notification-icon.png"
      }]
    ]
  }
}
```

#### C. Build Properties Update
```json
// Update in app.json:
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "android": {
          "enableMinifyInReleaseBuilds": true
        }
      }]
    ]
  }
}
```

### Step 4: Test Build Process

#### A. Clean and Prebuild
```bash
# Clear caches
npx expo r -c

# Clean prebuild
npx expo prebuild --clean

# Test builds
npm run ios
npm run android
```

#### B. Handle Build Issues

**Common Issue: Store Submission (iOS)**
```json
// Temporary workaround until React Native 0.81.1
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "ios": {
          "buildReactNativeFromSource": false
        }
      }]
    ]
  }
}
```

**Common Issue: Autolinking Problems**
```bash
# Verify autolinking
npx expo-modules-autolinking verify -v

# If issues, try legacy config:
{
  "expo": {
    "autolinking": {
      "legacy_shallowReactNativeLinking": true,
      "searchPaths": ["../../node_modules", "node_modules"]
    }
  }
}
```

### Step 5: Optimize for New Features

#### A. Enable React Compiler
```javascript
// babel.config.js - should be enabled by default
module.exports = {
  presets: [
    ['babel-preset-expo', {
      reactCompiler: true // Verify this is true
    }]
  ]
};
```

#### B. Update Development Workflow
```bash
# Test new CLI features
npx expo start

# Verify import stack traces work
# Introduce an import error and check error output

# Test React Compiler
# Press J in CLI and check Components panel
```

## Platform-Specific Migrations

### iOS 26 Features (Optional)

#### A. Liquid Glass Icons
```bash
# If you have macOS and Icon Composer:
# 1. Download Icon Composer from Mac App Store
# 2. Create .icon file from your existing icon
# 3. Update app.json:
{
  "ios": {
    "icon": "./assets/app.icon"
  }
}
```

#### B. Expo UI Components (Beta)
```bash
# Install Expo UI (if desired)
npx expo install @expo/ui

# Update Xcode version for iOS 26 features
# Use Xcode 16.1+ and iOS 26 beta
```

### Android 16 Compliance (Required)

#### A. Edge-to-Edge Implementation
```typescript
// Ensure proper edge-to-edge handling
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { StatusBar } from 'expo-status-bar';

function App() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{
      flex: 1,
      paddingTop: insets.top,
      paddingBottom: insets.bottom
    }}>
      <StatusBar style="auto" />
      {/* Content */}
    </View>
  );
}
```

#### B. Predictive Back Gesture (New Projects)
```json
// Enable in app.json for existing projects:
{
  "expo": {
    "android": {
      "predictiveBackGestureEnabled": true
    }
  }
}
```

## Post-Migration Testing

### Automated Testing
```bash
# Run existing tests
npm test

# Test on both architectures (if applicable)
npm run ios
npm run android

# Test EAS builds
eas build --platform all --profile preview
```

### Manual Testing Checklist
- [ ] App launches without errors
- [ ] Navigation works correctly
- [ ] File operations function (if using expo-file-system)
- [ ] Notifications work (if used)
- [ ] Platform-specific features work
- [ ] Performance feels improved
- [ ] No new warnings in console

### Performance Validation
```bash
# Check build times (should be faster on iOS)
time npm run ios

# Monitor React Compiler optimizations
# Press J in Expo CLI → Components panel

# Verify autolinking performance
npx expo-modules-autolinking verify -v
```

## Troubleshooting Common Issues

### Build Failures

#### Node.js Version Issues
```bash
# Error: Node version not supported
node --version # Should be 20.19+

# Solution: Update Node.js
nvm install 20.19.0 && nvm use 20.19.0
```

#### Xcode Version Issues
```bash
# Error: Xcode version incompatible
xcode-select -p

# Solution: Update Xcode to 16.1+
# Install from App Store or Developer Portal
```

### Runtime Errors

#### File System API Errors
```typescript
// Error: FileSystem.readAsStringAsync is not a function

// Quick fix:
import * as FileSystem from 'expo-file-system/legacy';

// Or migrate to new API:
import { Directory } from 'expo-file-system';
const file = Directory.documentDirectory.createFile('data.json');
```

#### Reanimated Compatibility
```bash
# Error: Reanimated requires New Architecture

# Solution 1: Enable New Architecture
# Add to app.json plugins: expo-build-properties with newArchEnabled: true

# Solution 2: Downgrade Reanimated temporarily
npm install react-native-reanimated@^3.15.0
```

#### Autolinking Issues
```bash
# Error: Native module not found

# Check autolinking:
npx expo-modules-autolinking verify -v

# Reset autolinking:
npx expo prebuild --clean
```

### Development Issues

#### Metro Bundler Problems
```bash
# Clear Metro cache
npx expo r -c

# Reset bundler
rm -rf node_modules/.cache
npm install
```

#### iOS Simulator Hotkeys
```bash
# If hotkeys don't work in iOS 18.4+ simulators:
# Use Expo CLI interactive prompt instead
npx expo start
# Press r for reload, m for dev menu
```

## Rollback Strategy

### If Migration Fails
```bash
# 1. Return to backup branch
git checkout main
git branch -D sdk54-migration

# 2. Or rollback packages
npx expo install expo@~53.0.0
npm install

# 3. Restore previous configuration
git checkout HEAD~1 app.json
git checkout HEAD~1 package.json
```

### Gradual Migration
```typescript
// Migrate incrementally:
// 1. Update SDK but keep legacy APIs
// 2. Test thoroughly
// 3. Migrate to new APIs one by one
// 4. Enable new features last

// Example: File system gradual migration
import * as LegacyFS from 'expo-file-system/legacy';
import { Directory } from 'expo-file-system';

// Keep both APIs during transition
const useNewFileSystem = __DEV__ ? true : false;
```

## Getting Help

### Resources
- [GitHub Issues](https://github.com/expo/expo/issues) - Report bugs
- [Discord](https://chat.expo.dev/) - Community support
- [Forums](https://forums.expo.dev/) - Discussion and help
- [Documentation](https://docs.expo.dev/) - Official guides

### When Reporting Issues
Include:
- SDK version: `npx expo --version`
- Node version: `node --version`
- Platform: iOS/Android
- Error messages and stack traces
- Minimal reproduction steps

### Beta Testing Support
During SDK 54 beta period:
- Tag issues with "SDK 54 beta"
- Join Discord office hours
- Help prioritize critical issues
- Get recognition in release notes

---

**Migration Complete!** 
Your app should now be running on Expo SDK 54 with improved performance and access to the latest platform features.

**Next Steps:**
- Explore [New Features](../new-features/README.md) for enhancement opportunities
- Review [Performance Improvements](../performance-improvements/README.md) for optimization benefits