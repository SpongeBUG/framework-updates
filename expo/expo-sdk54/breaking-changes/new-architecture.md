# New Architecture Migration
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/breaking-changes/new-architecture.md -->
<!-- Purpose: Comprehensive guide for New Architecture migration -->
<!-- Dependencies: React Native 0.81, Expo SDK 54 -->

## Critical Timeline

**⚠️ Important:** SDK 54 is the **final release** to support Legacy Architecture.

### Timeline Overview
- **SDK 54 (Current):** Last release with Legacy Architecture support
- **SDK 55 (Q4 2025):** New Architecture required, no opt-out available
- **React Native 0.82:** Legacy Architecture completely removed

### Current Adoption
- **75% of SDK 53 projects** already use New Architecture on EAS Build
- **Growing library ecosystem** requires New Architecture
- **Performance benefits** demonstrated in production apps

## Understanding New Architecture

### Core Components

**Fabric (UI Layer):**
- New rendering system replacing the legacy UI manager
- Synchronous layout and rendering
- Better performance and reduced memory usage
- Type-safe native component interface

**TurboModules (Native Modules):**
- New native module system with lazy loading
- Type-safe JavaScript-native interface
- Better performance through code generation
- Gradual migration path from legacy modules

**Codegen:**
- Automatically generates type-safe interfaces
- Compile-time validation of native specifications
- Reduced boilerplate code
- Better debugging experience

### Architecture Benefits

```typescript
// Legacy Architecture: Bridge-based communication
// JavaScript → Bridge → Native (async, serialized)

// New Architecture: Direct interface
// JavaScript ↔ Native (sync when needed, type-safe)
```

#### Performance Improvements
- **30-50% faster** startup times
- **Reduced memory usage** by 20-30%
- **Better frame rates** during complex animations
- **Faster native module calls** with TurboModules

#### Developer Experience
- **Better error messages** with type safety
- **Faster debugging** with synchronous operations
- **Improved tooling** support
- **Future-proof** architecture

## Enabling New Architecture

### Step 1: Check Current Status
```bash
# Check if New Architecture is already enabled
npx expo config --type introspect | grep newArchEnabled

# Check your current React Native version
npm list react-native
# Should be 0.81.0+ for best experience
```

### Step 2: Enable New Architecture
Add to your `app.json`:

```json
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "newArchEnabled": true
        }
      ]
    ]
  }
}
```

### Step 3: Install Dependencies
```bash
# Ensure all dependencies are compatible
npx expo install --fix

# Prebuild with New Architecture
npx expo prebuild --clean
```

### Step 4: Test Builds
```bash
# Test on both platforms
npm run ios
npm run android

# Check for any compatibility issues
# Look for "New Architecture" in logs
```

## Library Compatibility

### Libraries Requiring New Architecture
These popular libraries now require New Architecture in their latest versions:

```json
{
  "dependencies": {
    "react-native-reanimated": "^4.0.0",  // New Architecture only
    "@shopify/flash-list": "^2.0.0"       // New Architecture only
  }
}
```

### Migration Strategies

#### Strategy 1: Enable New Architecture (Recommended)
```bash
# Enable New Architecture and update libraries
npm install react-native-reanimated@^4.0.0
npm install @shopify/flash-list@^2.0.0

# Add to app.json
{
  "expo": {
    "plugins": [
      ["expo-build-properties", { "newArchEnabled": true }]
    ]
  }
}
```

#### Strategy 2: Temporary Legacy Support
```bash
# Stay on Legacy Architecture temporarily
npm install react-native-reanimated@^3.15.0
npm install @shopify/flash-list@^1.x.x

# But plan migration for SDK 55!
```

## Verifying New Architecture

### Build-time Verification
```bash
# Check build logs for New Architecture indicators
npm run ios 2>&1 | grep -i "new architecture"
npm run android 2>&1 | grep -i "new architecture"
```

### Runtime Verification
```typescript
import { Platform } from 'react-native';

// Check if New Architecture is active
function checkNewArchitecture() {
  if (__DEV__) {
    // Check for TurboModules
    const isTurboModuleEnabled = global.__turboModuleProxy != null;
    
    // Check for Fabric
    const isFabricEnabled = global.nativeFabricUIManager != null;
    
    console.log('New Architecture Status:', {
      turboModules: isTurboModuleEnabled,
      fabric: isFabricEnabled,
      platform: Platform.OS
    });
    
    return isTurboModuleEnabled && isFabricEnabled;
  }
  
  return false;
}

// Use in your app
export default function App() {
  React.useEffect(() => {
    if (__DEV__) {
      const isNewArch = checkNewArchitecture();
      console.log('Running with New Architecture:', isNewArch);
    }
  }, []);
  
  // Your app content
  return <YourAppContent />;
}
```

## Common Migration Issues

### 1. Native Module Compatibility

**Problem:** Some native modules don't support New Architecture

**Solution:**
```bash
# Check library compatibility
npm list react-native-reanimated
npm list @react-native-community/datetimepicker
# etc.

# Update incompatible libraries
npm install compatible-alternative

# Or temporarily disable New Architecture for that library
```

**Check Community Support:**
- [New Architecture Library Status](https://github.com/reactwg/react-native-new-architecture/discussions/167)
- Library GitHub issues and README files

### 2. Build Failures

**Problem:** Build fails after enabling New Architecture

**Solution:**
```bash
# Clear all caches and rebuild
npx expo r -c
rm -rf node_modules
npm install
npx expo prebuild --clean

# Check for specific error messages
npm run ios 2>&1 | tee ios-build.log
npm run android 2>&1 | tee android-build.log
```

### 3. Runtime Crashes

**Problem:** App crashes on launch with New Architecture

**Solution:**
```bash
# Check device logs for specific errors
# iOS: Xcode → Window → Devices and Simulators → View Device Logs
# Android: adb logcat | grep ReactNative

# Common fixes:
# 1. Update all Expo modules
npx expo install --fix

# 2. Check for unsupported native code
# Look for direct bridge usage in custom native code

# 3. Verify Hermes is enabled (recommended with New Architecture)
{
  "expo": {
    "jsEngine": "hermes"
  }
}
```

### 4. Performance Regressions

**Problem:** App feels slower after enabling New Architecture

**Solution:**
```typescript
// Profile performance before and after
import { Profiler } from 'react';

function ProfiledApp() {
  const onRenderCallback = (id, phase, actualDuration) => {
    console.log('Render performance:', {
      component: id,
      phase, // 'mount' or 'update'
      duration: actualDuration
    });
  };

  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <App />
    </Profiler>
  );
}

// Check specific performance issues:
// 1. Ensure React Compiler is enabled
// 2. Update to latest library versions
// 3. Profile with React DevTools
```

## Advanced Configuration

### Selective Module Migration

For gradual migration, you can control which modules use New Architecture:

```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Configure New Architecture features
config.resolver.unstable_enablePackageExports = true;
config.resolver.unstable_conditionNames = ['react-native', 'browser', 'require'];

// TurboModule configuration
config.transformer.unstable_allowRequireContext = true;

module.exports = config;
```

### Custom Native Components

If you have custom native components, update them for New Architecture:

#### iOS (Objective-C/Swift)
```objc
// Legacy Architecture
#import <React/RCTBridgeModule.h>
@interface MyModule : NSObject <RCTBridgeModule>

// New Architecture (TurboModule)
#import <ReactCommon/RCTTurboModule.h>
#import "MyModuleSpec.h"
@interface MyModule : NSObject <MyModuleSpec>
```

#### Android (Java/Kotlin)
```java
// Legacy Architecture
public class MyModule extends ReactContextBaseJavaModule {
    
// New Architecture (TurboModule)
public class MyModule extends ReactContextBaseJavaModule implements MyModuleSpec {
```

## Testing New Architecture

### Test Plan
1. **Functionality Testing:**
   - Core app features work correctly
   - Navigation functions properly
   - Native modules respond correctly

2. **Performance Testing:**
   - Startup time improvements
   - Animation smoothness
   - Memory usage patterns

3. **Compatibility Testing:**
   - All libraries work correctly
   - No runtime warnings or errors
   - Platform-specific features work

### Automated Testing
```typescript
// __tests__/new-architecture.test.ts
import { render } from '@testing-library/react-native';
import App from '../App';

describe('New Architecture Compatibility', () => {
  it('should render without crashes', () => {
    const { getByTestId } = render(<App />);
    // Test core functionality
  });

  it('should have TurboModules enabled', () => {
    // Only test in development
    if (__DEV__) {
      expect(global.__turboModuleProxy).toBeDefined();
    }
  });

  it('should have Fabric enabled', () => {
    if (__DEV__) {
      expect(global.nativeFabricUIManager).toBeDefined();
    }
  });
});
```

## Rollback Strategy

If you encounter issues, you can rollback:

### Immediate Rollback
```json
// app.json - disable New Architecture
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "newArchEnabled": false
        }
      ]
    ]
  }
}
```

```bash
# Downgrade libraries if needed
npm install react-native-reanimated@^3.15.0
npm install @shopify/flash-list@^1.x.x

# Rebuild
npx expo prebuild --clean
npm run ios
npm run android
```

### Long-term Planning
Since SDK 55 will require New Architecture:

1. **Identify Problematic Libraries:** Make a list of incompatible dependencies
2. **Find Alternatives:** Research New Architecture compatible alternatives
3. **Contact Library Maintainers:** Request New Architecture support
4. **Plan Timeline:** Prepare for SDK 55 migration deadline

## Migration Checklist

### Pre-Migration
- [ ] Audit all dependencies for New Architecture compatibility
- [ ] Backup current working version
- [ ] Test plan prepared
- [ ] Team informed of potential issues

### Migration Process
- [ ] Enable New Architecture in app.json
- [ ] Update compatible libraries to latest versions
- [ ] Clean prebuild: `npx expo prebuild --clean`
- [ ] Test on both iOS and Android
- [ ] Verify performance improvements
- [ ] Check for runtime warnings/errors

### Post-Migration
- [ ] Monitor app performance in production
- [ ] Update documentation
- [ ] Plan for SDK 55 (New Architecture only)
- [ ] Share learnings with team

### Emergency Rollback
- [ ] Disable New Architecture in app.json
- [ ] Downgrade libraries if needed
- [ ] Clean rebuild
- [ ] Verify app functionality
- [ ] Plan re-migration approach

## Support and Resources

### Documentation
- [React Native New Architecture](https://reactnative.dev/docs/new-architecture-intro)
- [Expo New Architecture Guide](https://docs.expo.dev/guides/new-architecture/)
- [TurboModule Documentation](https://reactnative.dev/docs/new-architecture-library-intro)

### Community
- [New Architecture Working Group](https://github.com/reactwg/react-native-new-architecture)
- [Expo Discord #new-architecture](https://chat.expo.dev/)
- [Library Compatibility Tracking](https://github.com/reactwg/react-native-new-architecture/discussions/167)

### Getting Help
When reporting New Architecture issues:

1. **Specify Architecture:** Mention you're using New Architecture
2. **Library Versions:** List all native library versions
3. **Error Details:** Include full error messages and stack traces
4. **Environment:** Node, Expo, React Native versions
5. **Reproduction:** Provide minimal reproduction case

---

**Remember:** New Architecture migration is not optional for SDK 55. Start planning and testing now to ensure a smooth transition.
