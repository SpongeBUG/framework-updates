# Breaking Changes
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/breaking-changes/README.md -->
<!-- Purpose: Required migrations, deprecations, and compatibility changes -->
<!-- Dependencies: React Native 0.81, Expo SDK 54 -->

## Critical Architecture Changes

### 🚨 Final Legacy Architecture Support
**Impact:** High - Affects all projects using Legacy Architecture

SDK 54 is the **last release** to support Legacy Architecture:
- **SDK 55:** New Architecture required (no opt-out)
- **React Native 0.82:** Legacy Architecture completely removed
- **Current adoption:** 75% of SDK 53 projects already use New Architecture

#### Action Required
```bash
# Check your current architecture
npx expo config --type introspect | grep newArchEnabled

# Enable New Architecture (if not already)
npx expo install --fix
```

#### New Architecture Requirements
Libraries requiring New Architecture in SDK 54:
- `react-native-reanimated` v4+
- `@shopify/flash-list` v2+

### JSC Engine Removal
**Impact:** Medium - Affects projects using JavaScriptCore

React Native 0.81 removes built-in JSC support:

```bash
# ❌ No longer available by default
"reactNativeEngine": "jsc"

# ✅ Migration options
# Option 1: Use Hermes (recommended)
"reactNativeEngine": "hermes"

# Option 2: Use community JSC
npm install @react-native-community/jsc
```

**Manual Configuration Required:** Community JSC lacks config plugin.

## File System API Changes

### expo-file-system Default API
**Impact:** High - Affects all file system usage

The legacy API is now at `expo-file-system/legacy`:

```typescript
// ❌ Old imports (now legacy)
import * as FileSystem from 'expo-file-system';

// ✅ New imports
// Legacy API (temporary migration)
import * as FileSystem from 'expo-file-system/legacy';

// New API (recommended)
import { Directory, File } from 'expo-file-system';
```

#### Migration Steps
1. **Immediate Fix:** Replace all `expo-file-system` imports with `expo-file-system/legacy`
2. **Long-term:** Migrate to new object-oriented API gradually

#### New API Example
```typescript
// Legacy API
const fileUri = FileSystem.documentDirectory + 'data.json';
await FileSystem.writeAsStringAsync(fileUri, JSON.stringify(data));
const content = await FileSystem.readAsStringAsync(fileUri);

// New API
const documentsDir = Directory.documentDirectory;
const dataFile = documentsDir.createFile('data.json');
await dataFile.write(JSON.stringify(data));
const content = await dataFile.readAsString();
```

## Metro Bundler Changes

### Internal Import Paths
**Impact:** Low - Primarily affects library developers

Metro 0.83 changes internal import paths:

```javascript
// ❌ Deprecated internal paths
import something from 'metro/src/internal/module';

// ✅ New internal paths (if needed)
import something from 'metro/private/internal/module';
```

**Note:** Most app developers unaffected, but library updates may be required.

## Reanimated v4 Breaking Changes

### New Architecture Requirement
**Impact:** High - Affects Reanimated users

Reanimated v4 only supports New Architecture:

```bash
# If staying on Legacy Architecture temporarily
npm install react-native-reanimated@^3.15.0

# For New Architecture (recommended)
npm install react-native-reanimated@^4.0.0
```

### Worklets Integration
Reanimated v4 introduces `react-native-worklets`:

```javascript
// ❌ No longer needed in babel.config.js
// Expo handles this automatically
plugins: ['react-native-reanimated/plugin']

// ✅ Automatic configuration via babel-preset-expo
// No manual babel configuration required
```

### NativeWind Compatibility
**Important:** NativeWind doesn't support Reanimated v4 yet:
- Use Reanimated v3 if you need NativeWind
- Monitor [NativeWind roadmap](https://github.com/nativewind/nativewind) for v4 support

## Notification API Changes

### expo-notifications Exports
**Impact:** Medium - Affects deprecated function usage

Removed deprecated function exports:

```typescript
// ❌ Deprecated exports removed
import { 
  getExpoPushTokenAsync,  // Removed
  getDevicePushTokenAsync // Removed  
} from 'expo-notifications';

// ✅ Use class-based API
import * as Notifications from 'expo-notifications';

const token = await Notifications.getExpoPushTokenAsync();
const deviceToken = await Notifications.getDevicePushTokenAsync();
```

## Configuration Changes

### Build Properties
**Impact:** Low - Affects ProGuard configuration

Field renamed for consistency:

```json
// ❌ Deprecated
{
  "expo": {
    "plugins": [
      ["expo-build-properties", {
        "android": {
          "enableProguardInReleaseBuilds": true
        }
      }]
    ]
  }
}

// ✅ New field name
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

### Notification Configuration
**Impact:** Medium - Affects notification setup

App config field deprecated:

```json
// ❌ Deprecated app.json config
{
  "expo": {
    "notification": {
      "icon": "./assets/notification-icon.png"
    }
  }
}

// ✅ Use expo-notifications config plugin
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

## Component Deprecations

### SafeAreaView Replacement
**Impact:** Medium - Affects layout components

React Native's SafeAreaView deprecated:

```javascript
// ❌ Deprecated
import { SafeAreaView } from 'react-native';

// ✅ Use react-native-safe-area-context
import { SafeAreaProvider, SafeAreaView } from 'react-native-safe-area-context';

function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView>
        {/* Your app content */}
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

## Build System Changes

### Store Submission Issue
**Impact:** High - Affects App Store releases

React Native 0.81.0 precompiled frameworks can't be submitted to App Store:

```json
// Temporary workaround until 0.81.1
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

**Resolution:** Fixed in React Native 0.81.1 (available week of August 25, 2025)

## Autolinking Changes

### Dependency Resolution
**Impact:** Medium - Affects monorepos and complex setups

Autolinking now uses package.json dependencies instead of scanning node_modules:

#### Breaking Changes
- **Monorepos:** Only app dependencies linked, not hoisted packages
- **Transitive Linking:** Now works for React Native modules
- **Search Behavior:** More restrictive dependency discovery

#### Opt-out Configuration
```json
// Revert to legacy behavior if needed
{
  "expo": {
    "autolinking": {
      "legacy_shallowReactNativeLinking": true,
      "searchPaths": ["../../node_modules", "node_modules"]
    }
  }
}
```

## Version Requirements

### Minimum Versions
**Impact:** High - Affects development environment

Updated minimum requirements:

```json
{
  "engines": {
    "node": ">=20.19.0",
    "npm": ">=10.0.0"
  }
}
```

- **Xcode:** 16.1+ (iOS development)
- **Android SDK:** API 36+ target
- **React Native:** 0.81.0+

## Migration Priority Matrix

### High Priority (Required)
1. ✅ **Update Node.js** to 20.19+
2. ✅ **Update Xcode** to 16.1+
3. ✅ **Fix expo-file-system imports** 
4. ✅ **Plan New Architecture migration**
5. ✅ **Handle JSC removal** (if applicable)

### Medium Priority (Recommended)
1. 🔄 **Update SafeAreaView usage**
2. 🔄 **Migrate notification config**
3. 🔄 **Update build properties**
4. 🔄 **Test Reanimated v4** (if using)

### Low Priority (Optional)
1. 📋 **Review Metro internal imports** (library developers)
2. 📋 **Optimize autolinking** (monorepo projects)
3. 📋 **Update deprecated exports**

## Testing Breaking Changes

### Verification Steps
```bash
# 1. Check environment requirements
node --version # Should be 20.19+
xcode-select -p # Should show Xcode 16.1+

# 2. Verify autolinking
npx expo-modules-autolinking verify -v

# 3. Test builds
npx expo prebuild --clean
npm run ios
npm run android

# 4. Check for deprecated warnings
npx expo start # Look for deprecation warnings
```

### Common Issues
- **Build failures** - Usually Xcode/Node version related
- **Module not found** - Often autolinking configuration
- **Runtime errors** - Typically file system API changes
- **Store submission** - Precompiled frameworks issue

## Rollback Strategies

### If Migration Issues Occur
1. **Revert to SDK 53:** `npx expo install expo@~53.0.0`
2. **Use legacy APIs:** Import from `/legacy` paths
3. **Disable new features:** Use compatibility flags
4. **Seek help:** [GitHub Issues](https://github.com/expo/expo/issues) or [Discord](https://chat.expo.dev/)

---

**Next Steps:**
- Follow [Migration Guide](../migration-guide/README.md) for step-by-step instructions
- Review [Development Tools](../development-tools/README.md) for enhanced workflow
- Check [Platform-Specific Updates](../platform-specific/README.md) for platform features
