# Core Platform Changes
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/core-platform-changes/README.md -->
<!-- Purpose: React Native and core platform updates documentation -->
<!-- Dependencies: React Native 0.81, React 19.1.0 -->

## React Native 0.81 Integration

### Major Updates
- **React Native Version:** 0.81.0
- **React Version:** 19.1.0
- **Architecture:** New Architecture as default (final Legacy support)

### React 19.1 Features
React 19.1.0 brings significant improvements to React development:

- **React Compiler Integration** - Automatic optimization enabled by default
- **Enhanced Concurrent Features** - Better Suspense and concurrent rendering
- **Improved Error Boundaries** - More reliable error handling patterns
- **Server Components Ready** - Better SSR/SSG compatibility

### New Architecture Transition

**Critical Notice:** SDK 54 is the **final release** to support Legacy Architecture.

#### Current Status
- **75% of SDK 53 projects** already use New Architecture on EAS
- **SDK 55** will require New Architecture (no opt-out available)
- **Interop layer** remains for Legacy Architecture libraries

#### Library Requirements
Several popular libraries now require New Architecture:
- `react-native-reanimated` v4+ (New Architecture only)
- `@shopify/flash-list` v2+ (New Architecture only)

## JavaScript Engine Changes

### JSC Support Removal
React Native 0.81 removes built-in JavaScriptCore (JSC) support:

#### Migration Options
1. **Hermes (Recommended)** - Default and optimized engine
2. **Community JSC** - Use `@react-native-community/jsc` package

```json
// For community JSC (manual config required)
{
  "dependencies": {
    "@react-native-community/jsc": "latest"
  }
}
```

**Note:** Community JSC doesn't provide config plugin yet, requiring manual native project modifications.

## Module Resolution Improvements

### Metro 0.83 Changes
Internal Metro import path changes affect library developers:

```javascript
// ❌ Old (deprecated)
import something from 'metro/src/internal-module';

// ✅ New (required)
import something from 'metro/private/internal-module';
```

**Impact:** Most app developers unaffected, but library updates may be required.

## Build System Evolution

### Precompiled Frameworks
React Native 0.81 introduces precompiled XCFrameworks for iOS:

#### Benefits
- **90% faster build times** - 120s → 10s on M4 Max
- **Swift Package Manager readiness** - Moving away from CocoaPods
- **Consistent compilation** - Reduced build variations

#### Known Limitations
- React Native 0.81.0 release builds can't be submitted to App Store
- **Fixed in 0.81.1** (available week of August 25, 2025)

### Workaround for Store Submission
```json
// expo-build-properties config
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "ios": {
            "buildReactNativeFromSource": false
          }
        }
      ]
    ]
  }
}
```

## Platform Target Updates

### iOS Requirements
- **Minimum Xcode:** 16.1
- **iOS Target:** Supports iOS 26 features
- **Architecture:** arm64, x86_64 (simulators)

### Android Requirements
- **Target API:** 36 (Android 16)
- **Minimum API:** 21 (Android 5.0)
- **Architecture:** arm64-v8a, armeabi-v7a, x86_64

## Breaking Changes Summary

### Immediate Actions Required
1. **Update Xcode** to 16.1+ for iOS development
2. **Review JSC usage** - migrate to Hermes or community JSC
3. **Update dependencies** that use internal Metro imports
4. **Plan New Architecture migration** for SDK 55 readiness

### Timeline Considerations
- **SDK 54 Beta:** 2-week testing period with continuous fixes
- **SDK 55 ETA:** Q4 2025 (New Architecture only)
- **React Native 0.82:** Will remove Legacy Architecture opt-out

## Development Environment Setup

### Recommended Configuration
```json
// package.json engines
{
  "engines": {
    "node": ">=20.19.0",
    "npm": ">=10.0.0"
  }
}
```

### Xcode Setup
- Install Xcode 16.1+ from App Store or Developer Portal
- Update Command Line Tools: `xcode-select --install`
- Verify installation: `xcode-select -p`

### Android Setup
- Android Studio with SDK 36+
- Update build tools and platform tools
- Verify emulator compatibility with API 36

## Performance Characteristics

### Build Performance
- **iOS:** 90% improvement with precompiled frameworks
- **Android:** Standard compilation times maintained
- **Metro:** Improved module resolution with experimental features

### Runtime Performance  
- **Hermes:** Enhanced performance and memory usage
- **React 19:** Better concurrent rendering and updates
- **New Architecture:** Improved bridge performance

---

**Next Steps:**
- Review [New Features](../new-features/README.md) for SDK 54 additions
- Check [Breaking Changes](../breaking-changes/README.md) for migration requirements
- See [Migration Guide](../migration-guide/README.md) for upgrade instructions
