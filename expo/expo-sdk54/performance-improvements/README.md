# Performance Improvements
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/performance-improvements/README.md -->
<!-- Purpose: Build optimization, runtime enhancements, and performance gains -->
<!-- Dependencies: React Native 0.81, Xcode 16.1+ -->

## iOS Build Performance Revolution

### Precompiled XCFrameworks
React Native 0.81 introduces precompiled XCFrameworks for iOS, delivering unprecedented build speed improvements.

#### Performance Metrics
- **RNTester Build Time:** 120s → 10s (M4 Max)
- **Improvement Factor:** ~12x faster (90% reduction)
- **Impact Scale:** Greater improvements in smaller projects, noticeable gains in large projects

#### Technical Details
```bash
# Before: React Native compiled from source
Building React Native dependencies: ~100-120s

# After: Precompiled XCFrameworks
Building React Native dependencies: ~8-10s
```

#### Architecture Impact
- **Clean Builds:** Most dramatic improvement
- **Incremental Builds:** Moderate improvement
- **CI/CD Pipelines:** Significant time savings
- **Developer Iteration:** Faster development cycles

### Swift Package Manager Readiness
Precompiled frameworks move Expo closer to Swift Package Manager adoption:

```swift
// Future SPM support preparation
.package(url: "https://github.com/expo/expo-spm", from: "54.0.0")
```

**Benefits:**
- Modern dependency management
- Better Xcode integration
- Improved build reproducibility
- Enhanced caching mechanisms

## Metro Bundler Enhancements

### Experimental Import Support
Enhanced Metro import support enabled by default:

```javascript
// metro.config.js
module.exports = {
  resolver: {
    // experimentalImportSupport now default: true
    unstable_enableSymlinks: true,
    unstable_enablePackageExports: true
  }
};
```

#### Performance Benefits
- **Tree Shaking:** Better dead code elimination
- **React Compiler:** Improved compatibility
- **ESM Support:** Step toward full ES modules
- **Live Bindings:** Enhanced module interoperability

### Import Stack Traces
Default enabled import debugging:

```bash
# Enhanced error messages show import chain
Error: Cannot resolve module 'missing-package'
Import stack:
  /src/components/Button.tsx
  /src/screens/HomeScreen.tsx
  /src/App.tsx
```

#### Developer Benefits
- **Faster Debugging:** Immediate issue location
- **AI Agent Compatible:** Better for automated debugging
- **Reduced Resolution Time:** Clear dependency chains

## CSS Processing Optimization

### Lightning CSS Integration
Rust-based CSS processing replaces Node.js alternatives:

```javascript
// metro.config.js
module.exports = {
  transformer: {
    // lightningcss enabled by default for CSS autoprefixing
    lightningcssEnabled: true
  }
};
```

#### Performance Comparison
- **lightningcss (Rust):** ~10x faster than autoprefixer
- **Memory Usage:** Significantly lower
- **Feature Support:** Modern CSS features
- **Build Integration:** Seamless Metro integration

### Browserslist Support
```json
// package.json
{
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```

## React Compiler Performance

### Automatic Optimization
React Compiler enabled by default in new projects:

```javascript
// Automatic memoization without manual optimization
function ExpensiveComponent({ data, filters }) {
  // React Compiler automatically optimizes this
  const processedData = data.filter(filters.condition)
    .map(item => complexTransformation(item))
    .sort(filters.sortFn);
  
  return <DataView data={processedData} />;
}
```

#### Performance Gains
- **Automatic Memoization:** No manual `useMemo`/`useCallback`
- **Render Optimization:** Fewer unnecessary re-renders
- **Memory Usage:** Intelligent caching strategies
- **Bundle Size:** Optimized component compilation

### Development Tools
Monitor React Compiler in development:

```bash
# Expo CLI shows compiler status
npx expo start
> Experimental React Compiler is enabled.

# Press J to open component inspector
# See "Experimental React Compiler is enabled" in component panel
```

## Autolinking Performance

### Unified Resolution System
Expo Autolinking now handles both Expo and React Native modules consistently:

#### Transitive Dependencies
```json
// Your app only needs direct dependencies
{
  "dependencies": {
    "my-feature-library": "1.0.0"
    // my-feature-library's native deps auto-linked
  }
}
```

#### Benefits
- **Reduced Configuration:** Libraries manage their own dependencies
- **Better Monorepo Support:** Proper dependency isolation
- **Consistent Behavior:** Unified linking across module types
- **Performance:** Faster dependency resolution

### Build Verification
```bash
# Verify autolinking performance
npx expo-modules-autolinking verify -v

# Shows:
# - Linked modules
# - Resolution time
# - Dependency graph
# - Potential optimizations
```

## Runtime Performance

### React 19.1 Enhancements
- **Concurrent Rendering:** Better frame rate consistency
- **Suspense Improvements:** Smoother loading states
- **Memory Management:** Reduced memory footprint
- **Error Recovery:** Faster error boundary recovery

### Hermes JavaScript Engine
- **Startup Time:** Faster app initialization
- **Memory Usage:** Lower RAM consumption
- **Garbage Collection:** More efficient cleanup
- **Bytecode:** Smaller bundle sizes

## Development Workflow Improvements

### Enhanced Error Handling
```javascript
// Unhandled promise rejections now surface as errors
Promise.reject(new Error('Background task failed'))
  .catch(error => {
    // Previously silently failed, now properly logged
    console.error('Caught background error:', error);
  });
```

#### Developer Benefits
- **Error Visibility:** Hidden issues now surface
- **Debugging Speed:** Immediate problem identification
- **Production Reliability:** Catch issues in development

### Owner Stack Traces
Enhanced React error reporting:

```bash
# Before: Generic component error
Error in Component: Cannot read property 'map' of undefined

# After: Full component ownership chain
Error in UserProfile (created by UserScreen)
  in UserScreen (created by StackNavigator)
  in StackNavigator (created by App)
Cannot read property 'map' of undefined at line 23
```

## EAS Build Performance

### Xcode 26 Beta Support
```json
// eas.json
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

#### Build Time Benefits
- **iOS 26 Features:** Access latest platform features
- **Precompiled Frameworks:** Full build speed benefits
- **CI/CD Integration:** Faster pipeline execution

### Apple TV Build Optimization
tvOS builds leverage precompiled frameworks:
- **Significant Build Time Reduction:** Similar to iOS improvements
- **Memory Usage:** Lower build server resource usage
- **Parallel Builds:** Better resource utilization

## Performance Monitoring

### Development Metrics
Monitor performance during development:

```javascript
// Performance API available
const navigationStart = performance.now();
// ... navigation logic
const navigationEnd = performance.now();
console.log('Navigation took:', navigationEnd - navigationStart, 'ms');
```

### Production Insights
Enhanced `expo-insights` package:
- **Apple TV Support:** Performance tracking on tvOS
- **Memory Metrics:** Runtime memory usage
- **Render Performance:** Component render timing
- **Network Performance:** Request timing and caching

## Optimization Recommendations

### Build Performance
1. **Enable Precompiled Frameworks:** Use React Native 0.81 defaults
2. **Update Xcode:** Use Xcode 16.1+ for best performance
3. **Clean Builds:** Periodic clean builds for optimal performance
4. **CI/CD Caching:** Cache node_modules and Xcode build artifacts

### Runtime Performance
1. **Enable React Compiler:** Default in new projects
2. **Use New Architecture:** Better performance than Legacy
3. **Optimize Images:** Use `expo-image` for better caching
4. **Bundle Optimization:** Enable tree shaking and minification

### Development Workflow
1. **Use Fast Refresh:** Avoid full reloads when possible
2. **Enable Import Traces:** Faster debugging with stack traces
3. **Optimize Metro:** Use experimental features for faster bundling
4. **Monitor Memory:** Use React DevTools for memory profiling

---

**Measurement Tools:**
- React DevTools Profiler
- Xcode Instruments
- Android Studio Profiler
- Expo CLI performance metrics

**Next Steps:**
- Review [Development Tools](../development-tools/README.md) for enhanced tooling
- Check [Migration Guide](../migration-guide/README.md) for upgrade optimizations
