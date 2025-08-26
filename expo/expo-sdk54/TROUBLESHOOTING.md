# Troubleshooting Guide
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/TROUBLESHOOTING.md -->
<!-- Purpose: Common issues and solutions for SDK 54 migration and usage -->
<!-- Dependencies: Expo SDK 54, development environment -->

## Installation Issues

### Node.js Version Errors

**Error:** `Node version not supported` or `Unsupported Node.js version`

**Solution:**
```bash
# Check current version
node --version

# Update with nvm (recommended)
nvm install 20.19.0
nvm use 20.19.0
nvm alias default 20.19.0

# Or download from nodejs.org
# Verify after installation
node --version  # Should be 20.19.0+
```

**Troubleshooting nvm issues:**
```bash
# If nvm command not found
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# If nvm exists but doesn't work
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Expo CLI Update Failures

**Error:** `Failed to install expo@next` or dependency resolution errors

**Solution:**
```bash
# Clear npm cache
npm cache clean --force

# Clear yarn cache (if using yarn)
yarn cache clean

# Clear pnpm cache (if using pnpm)
pnpm store prune

# Remove node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Try installation again
npx expo install expo@next --fix
```

## Build Issues

### iOS Build Failures

**Error:** `Xcode version incompatible` or `Command line tools not found`

**Solution:**
```bash
# Update Xcode to 16.1+
# Install from App Store or Apple Developer Portal

# Update command line tools
xcode-select --install

# Verify installation
xcode-select -p
# Should output: /Applications/Xcode.app/Contents/Developer

# If wrong path, set manually
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

**Error:** `React Native build failed` or `Precompiled frameworks error`

**Solution:**
```bash
# Clean Xcode build
npx expo run:ios --clean

# If still failing, try building from source temporarily
# Add to app.json:
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "ios": {
          "buildReactNativeFromSource": true
        }
      }]
    ]
  }
}

# Rebuild
npx expo prebuild --clean
npx expo run:ios
```

### Android Build Failures

**Error:** `Android SDK not found` or `API level 36 not installed`

**Solution:**
```bash
# Open Android Studio
# Go to SDK Manager (Tools > SDK Manager)
# Install Android 16 (API level 36)
# Also install latest build tools

# Set ANDROID_HOME if not set
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

**Error:** `Edge-to-edge configuration error`

**Solution:**
```bash
# Remove react-native-edge-to-edge if only using config plugin
npm uninstall react-native-edge-to-edge

# Or install as direct dependency if using the package
npx expo install react-native-edge-to-edge

# Add navigation bar config to app.json
{
  "expo": {
    "android": {
      "androidNavigationBar": {
        "enforceContrast": false
      }
    }
  }
}
```

## Runtime Errors

### File System API Errors

**Error:** `FileSystem.readAsStringAsync is not a function` or similar

**Solution:**
```typescript
// Quick fix: Use legacy API
import * as FileSystem from 'expo-file-system/legacy';

// All your existing FileSystem calls will work
const content = await FileSystem.readAsStringAsync(fileUri);

// For long-term: Migrate to new API gradually
import { Directory, File } from 'expo-file-system';

// Convert file paths to new API
const fileUri = FileSystem.documentDirectory + 'data.json';
// Becomes:
const file = Directory.documentDirectory.createFile('data.json');
const content = await file.readAsString();
```

### SafeAreaView Deprecation

**Error:** `SafeAreaView from 'react-native' is deprecated`

**Solution:**
```typescript
// Remove old import
// import { SafeAreaView } from 'react-native';

// Add new imports
import { SafeAreaView, SafeAreaProvider } from 'react-native-safe-area-context';

// Wrap your app
function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={{ flex: 1 }}>
        {/* Your app content */}
      </SafeAreaView>
    </SafeAreaProvider>
  );
}

// Install if not already installed
npx expo install react-native-safe-area-context
```

### Reanimated Compatibility Issues

**Error:** `Reanimated requires New Architecture` or worklets errors

**Solution:**
```bash
# Option 1: Enable New Architecture (recommended)
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

# Install Reanimated v4
npm install react-native-reanimated@^4.0.0

# Option 2: Stay on Legacy Architecture temporarily
npm install react-native-reanimated@^3.15.0

# Remove Reanimated plugin from babel.config.js (Expo handles it)
// Remove this line:
// plugins: ['react-native-reanimated/plugin']

# Rebuild
npx expo prebuild --clean
```

**Error:** `NativeWind + Reanimated v4 conflict`

**Solution:**
```bash
# NativeWind doesn't support Reanimated v4 yet
# Use Reanimated v3 if you need NativeWind
npm install react-native-reanimated@^3.15.0

# Or wait for NativeWind v4 support
# Monitor: https://github.com/nativewind/nativewind
```

## Development Issues

### Metro Bundler Problems

**Error:** `Metro bundler failed to start` or `Cannot resolve module`

**Solution:**
```bash
# Clear all caches
npx expo r -c

# Clear Metro cache specifically
rm -rf node_modules/.cache

# Reset bundler
rm -rf .expo
npm install

# Start fresh
npx expo start --clear
```

**Error:** `Import cycle` or `Circular dependency`

**Solution:**
```bash
# Enable live bindings (usually fixes import cycles)
EXPO_UNSTABLE_LIVE_BINDINGS=true npx expo start

# If default-only import cycles still fail, refactor code:
// Instead of:
export default MyComponent;

// Use named exports:
export { MyComponent };
```

### Autolinking Issues

**Error:** `Native module not found` after upgrade

**Solution:**
```bash
# Verify autolinking
npx expo-modules-autolinking verify -v

# Check what's linked
npx expo-modules-autolinking list

# Reset autolinking
npx expo prebuild --clean

# If in monorepo, check search paths
{
  "expo": {
    "autolinking": {
      "searchPaths": ["../../node_modules", "node_modules"]
    }
  }
}
```

**Error:** `Too many/few modules linked`

**Solution:**
```bash
# Use legacy shallow linking if needed
{
  "expo": {
    "autolinking": {
      "legacy_shallowReactNativeLinking": true
    }
  }
}

# Or exclude specific modules
{
  "expo": {
    "autolinking": {
      "exclude": ["problematic-package"]
    }
  }
}
```

### iOS Simulator Issues

**Error:** Hotkeys (Cmd+R, Cmd+M) not working in iOS 18.4+ simulators

**Solution:**
```bash
# Use Expo CLI interactive prompt instead
npx expo start

# Then use keyboard shortcuts in terminal:
# r - reload app
# m - toggle dev menu  
# j - open developer menu
# shift+j - open React DevTools

# Or use simulator menu:
# Device > Shake Gesture (for dev menu)
# Hardware > Shake Gesture
```

### App Store Submission Issues

**Error:** `Cannot submit to App Store` with precompiled frameworks

**Solution:**
```bash
# Temporary workaround until React Native 0.81.1
# Add to app.json:
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

# Rebuild and submit
eas build --platform ios --profile production

# Remove workaround after React Native 0.81.1 release
```

## Performance Issues

### Slow Build Times

**Problem:** Builds taking too long despite precompiled frameworks

**Solution:**
```bash
# Ensure you're getting the benefits:
# 1. Clean build folder
npx expo run:ios --clean

# 2. Check Xcode version (must be 16.1+)
xcode-select -p

# 3. Verify React Native version
npm list react-native
# Should be 0.81.0+

# 4. Monitor build in Xcode to see what's slow
# Open .xcworkspace and build from Xcode

# 5. Check for excessive dependencies
npm ls --depth=0 | wc -l
# Consider removing unused packages
```

### Runtime Performance Issues

**Problem:** App feels slower after upgrade

**Solution:**
```bash
# 1. Check React Compiler is enabled
npx expo start
# Look for: "Experimental React Compiler is enabled"

# 2. Enable New Architecture if not already
{
  "expo": {
    "plugins": [
      ["expo-build-properties", { "newArchEnabled": true }]
    ]
  }
}

# 3. Profile with React DevTools
# Press j in Expo CLI → Profiler tab

# 4. Check for memory leaks
# Use React DevTools Profiler → Memory tab

# 5. Optimize images
npx expo install expo-image
# Replace Image components with expo-image
```

## TypeScript Issues

**Error:** Type errors after SDK upgrade

**Solution:**
```bash
# Update TypeScript definitions
npm install --save-dev typescript@latest
npx expo install --fix

# Check for conflicting type packages
npm ls @types/react
npm ls @types/react-native

# Clear TypeScript cache
rm -rf node_modules/.cache/typescript
npx tsc --build --clean

# Restart TypeScript server in VS Code
# Cmd+Shift+P → "TypeScript: Restart TS Server"
```

## EAS Build Issues

**Error:** `Build failed on EAS`

**Solution:**
```bash
# 1. Check EAS CLI is updated
npm install -g @expo/eas-cli@latest

# 2. Update eas.json for SDK 54
{
  "build": {
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0"
      },
      "android": {
        "image": "ubuntu-22.04-android-34"
      }
    }
  }
}

# 3. Clear EAS cache
eas build --clear-cache --platform all

# 4. Check build logs
eas build:list
# Click on failed build to see detailed logs

# 5. Test locally first
npx expo prebuild --clean
npm run ios
npm run android
```

## Getting Help

### Before Asking for Help

1. **Check the version:**
   ```bash
   npx expo --version
   node --version
   npm --version
   ```

2. **Create minimal reproduction:**
   ```bash
   npx create-expo-app@latest TestApp --template default@next
   cd TestApp
   # Add minimal code that reproduces the issue
   ```

3. **Clear all caches:**
   ```bash
   npx expo r -c
   rm -rf node_modules
   npm install
   ```

4. **Check known issues:**
   - [SDK 54 Known Issues](https://docs.expo.dev/versions/v54.0.0/)
   - [GitHub Issues](https://github.com/expo/expo/issues)

### Where to Get Help

- **GitHub Issues:** https://github.com/expo/expo/issues
- **Discord Community:** https://chat.expo.dev/
- **Expo Forums:** https://forums.expo.dev/
- **Stack Overflow:** Tag with `expo` and `react-native`

### Reporting Issues

Include this information:
```bash
# Environment info
npx expo --version
node --version
npm --version
xcode-select -p  # on macOS

# Platform info
# iOS/Android version
# Physical device or simulator

# Error logs
# Complete error message
# Stack trace
# Steps to reproduce

# Project info
npx expo config --type introspect
```

---

**Remember:** SDK 54 is in beta, so some issues are expected. The Expo team is actively fixing issues during the beta period.
