# Development Tools
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/development-tools/README.md -->
<!-- Purpose: Expo CLI updates, debugging improvements, and workflow enhancements -->
<!-- Dependencies: Expo CLI, React Native 0.81 -->

## Expo CLI Enhancements

### Import Stack Traces (Default Enabled)
Enhanced debugging with complete import chains:

```bash
Error: Cannot resolve module 'missing-package'
Import stack:
  /src/components/Button.tsx
  /src/screens/HomeScreen.tsx  
  /src/App.tsx
```

#### Benefits
- **AI Agent Compatible:** Automated tools can resolve import issues
- **Faster Debugging:** Immediate source location identification
- **Better Error Context:** Complete dependency chain visibility

### Enhanced Metro Import Support
Experimental import support now enabled by default:

```javascript
// metro.config.js - now default behavior
module.exports = {
  resolver: {
    experimentalImportSupport: true, // Default: true
    unstable_enableSymlinks: true,
    unstable_enablePackageExports: true
  }
};
```

#### Features
- **React Compiler:** Better compatibility
- **Tree Shaking:** Improved dead code elimination  
- **ESM Support:** Step toward full ES modules
- **Live Bindings:** Enhanced module interoperability

### CSS Processing Improvements
Rust-based `lightningcss` autoprefixing:

```javascript
// metro.config.js - automatic configuration
module.exports = {
  transformer: {
    lightningcssEnabled: true // Default: true
  }
};
```

#### Performance Benefits
- **10x Faster:** Rust implementation vs Node.js
- **Lower Memory:** Significant memory usage reduction
- **Remove Dependencies:** Can remove `autoprefixer` from `postcss.config.mjs`
- **Browserslist Support:** Standard browser targeting

### Babel Enhancements
New transforms included by default:

```javascript
// babel.config.js - automatic inclusion
module.exports = {
  presets: ['babel-preset-expo'],
  // @babel/plugin-transform-class-static-block now included
};
```

#### Static Class Blocks Support
```javascript
class MyComponent {
  static {
    // Static initialization block now supported
    this.defaultProps = {
      theme: 'light'
    };
  }
}
```

## React Compiler Integration

### Default Enablement
React Compiler enabled in new projects:

```bash
npx expo start
> Experimental React Compiler is enabled.
```

#### Development Monitoring
```bash
# Press J in Expo CLI to access developer menu
# Navigate to Components panel
# See React Compiler optimization status
```

#### Component Inspector
View which components are automatically memoized:
- **Optimized Components:** Show "Memo" indicator
- **Render Count:** Track re-render frequency
- **Performance Impact:** Real-time optimization metrics

### Manual Configuration
```javascript
// babel.config.js - if customization needed
module.exports = {
  presets: [
    ['babel-preset-expo', {
      reactCompiler: true // Can be disabled if needed
    }]
  ]
};
```

## Enhanced Error Handling

### Owner Stack Traces (Default Enabled)
Improved React component error reporting:

```bash
# Before: Generic error
Error: Cannot read property 'map' of undefined

# After: Component ownership chain  
Error in UserProfile (created by UserScreen)
  in UserScreen (created by StackNavigator)
  in StackNavigator (created by App)
Cannot read property 'map' of undefined
  at UserProfile.render (UserProfile.tsx:23)
```

#### Human & AI Benefits
- **Faster Issue Location:** Exact component hierarchy
- **AI Debugging:** Better context for automated tools
- **Development Speed:** Immediate problem source identification

### Unhandled Promise Rejection Logging
Promises now surface as proper errors:

```javascript
// Previously silently failed
Promise.reject(new Error('Background task failed'));

// Now properly logged as error
Promise.reject(new Error('Background task failed'))
  .catch(error => {
    console.error('Unhandled promise rejection:', error);
  });
```

## Autolinking Resolution (Experimental)

### Sticky Resolution
Prevent module resolution mismatches:

```bash
# Enable experimental autolinking resolution
EXPO_USE_STICKY_RESOLVER=1 npx expo start
```

#### Benefits
- **Consistency:** JavaScript and native module alignment
- **Conflict Resolution:** Handle dependency duplicates
- **Monorepo Support:** Better workspace handling
- **Build Reliability:** Fewer module resolution issues

### Verification Tools
```bash
# Check autolinking configuration
npx expo-modules-autolinking verify -v

# Detailed output shows:
# - Linked native modules
# - JavaScript module mapping
# - Potential conflicts
# - Resolution strategies
```

## Development Client Improvements

### UI Rewrite
Complete rebuild of `expo-dev-client` interface:

#### Improvements
- **Simplified Interface:** Cleaner, more intuitive UI
- **Better React Native Integration:** Reduced complexity
- **Enhanced Hermes Debugging:** Improved debugging experience
- **Modernized Design:** Updated look and feel

### Apple TV Support (Experimental)
```bash
# Install development client on Apple TV
npx expo install expo-dev-client

# Note: Authentication with Expo account not yet available
```

#### Features
- **Basic Functionality:** Load and test projects
- **Development Workflow:** Hot reloading and debugging
- **Platform Testing:** Test Apple TV specific features

### Android TV Support (Full)
Complete support for Android TV development:
- **Full Authentication:** Expo account integration
- **Complete Feature Set:** All development client features
- **Performance Optimization:** Optimized for TV interfaces

## Prebuild Template Improvements

### Bundled Templates
Templates now included in `expo` package:

```bash
# Templates version-locked to Expo version
npx expo prebuild # Uses bundled template

# Custom template still supported
npx expo prebuild --template custom-template
```

#### Benefits
- **Version Consistency:** Template matches SDK version
- **Reliability:** No network dependency for default template
- **Predictability:** Consistent behavior across environments
- **Faster Prebuild:** No download required

## Live Bindings Support

### Enhanced Module Interoperability
Live bindings enabled by default for better compatibility:

```bash
# Disable live bindings if needed
EXPO_UNSTABLE_LIVE_BINDINGS=false npx expo start
```

#### Use Cases
- **Import Cycles:** Better handling of circular dependencies
- **Module Updates:** Real-time module state synchronization
- **ESM Compatibility:** Improved ES module support

#### Known Limitations
- **Default-only Cycles:** Still fail with only default imports
- **Performance:** Slight overhead for binding management

## Hotkey Improvements

### iOS Simulator Issue
Known issue with iOS 18.4+ simulators:
- **Reload Hotkey:** May not fire reliably  
- **Dev Menu:** Inconsistent response

#### Workaround
```bash
# Use interactive Expo CLI prompt instead
npx expo start
# Press corresponding keys in CLI for actions:
# r - Reload
# m - Dev menu
# j - Open developer menu
```

## Enhanced Logging

### Promise Rejection Visibility
Previously silent failures now logged:

```javascript
// Background promise failures now visible
fetch('/api/background-sync')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
  })
  // These rejections now appear in logs
  .catch(error => {
    console.error('Background sync failed:', error);
  });
```

### Development Environment Debugging
Improved development server logging:
- **Request Timing:** Track Metro bundling performance
- **Error Context:** Better error message context
- **Module Resolution:** Detailed resolution logging
- **Performance Metrics:** Build and reload timing

## VSCode Integration

### Expo Tools Extension
Enhanced VSCode support:

#### Features
- **Better IntelliSense:** Improved autocomplete
- **Error Highlighting:** Real-time error detection
- **Config Validation:** app.json/eas.json validation
- **Debugging Support:** Integrated debugging tools

#### Known Issue
Icon Composer validation warnings:
```bash
# Temporary warning for .icon files (will be resolved)
Warning: iOS icon validation for .icon format
```

## Development Workflow Optimization

### Fast Refresh Improvements
Better hot reloading experience:
- **Component State Preservation:** Maintain state across updates
- **Error Recovery:** Faster recovery from syntax errors
- **Import Updates:** Handle import/export changes better

### Metro Caching
Enhanced Metro bundler caching:
- **Dependency Caching:** Cache resolved dependencies
- **Transform Caching:** Cache babel transformations
- **Asset Caching:** Cache processed assets
- **Clear Cache:** `npx expo r -c` for cache clearing

## Testing & Debugging Tools

### React DevTools Integration
Enhanced React DevTools support:
- **Component Inspector:** View component hierarchy
- **Profiler:** Performance profiling tools
- **Props/State Viewer:** Real-time state inspection
- **React Compiler Visibility:** See optimization status

### Network Debugging
Improved network request debugging:
- **Request/Response Logging:** Detailed HTTP logging
- **Error Context:** Better network error messages
- **Performance Metrics:** Request timing analysis

### Memory Profiling
Enhanced memory debugging:
- **Hermes Profiling:** JavaScript heap analysis
- **Native Memory:** Platform-specific memory tools
- **Memory Leaks:** Detection and prevention tools

---

**Next Steps:**
- Explore [Performance Improvements](../performance-improvements/README.md) for optimization benefits
- Review [Migration Guide](../migration-guide/README.md) for upgrade workflow
