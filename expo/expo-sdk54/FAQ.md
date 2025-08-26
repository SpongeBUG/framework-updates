# Frequently Asked Questions (FAQ)
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/FAQ.md -->
<!-- Purpose: Common questions and answers about SDK 54 -->
<!-- Dependencies: Expo SDK 54 knowledge base -->

## General Questions

### Q: What's new in Expo SDK 54?
**A:** SDK 54 introduces several major improvements:
- **90% faster iOS builds** with precompiled XCFrameworks
- **React Native 0.81** with React 19.1.0
- **React Compiler** enabled by default
- **iOS 26 Liquid Glass** support
- **Android 16** mandatory edge-to-edge
- **New Architecture** as recommended default
- **Final Legacy Architecture** support

### Q: Is SDK 54 a breaking change release?
**A:** Yes, SDK 54 has several breaking changes:
- New file system API is now default
- SafeAreaView deprecation
- JSC engine removal
- New Architecture strongly recommended
- Android 16 edge-to-edge mandatory

See [Breaking Changes](./breaking-changes/README.md) for complete details.

### Q: Should I upgrade to SDK 54 right away?
**A:** Consider these factors:
- **Yes, if:** You want performance improvements, new features, and future compatibility
- **Wait, if:** Your project has complex native dependencies or tight deadlines
- **Remember:** SDK 54 is the final release with Legacy Architecture support

## Migration Questions

### Q: How long does migration to SDK 54 take?
**A:** Migration time varies by project:
- **Small projects:** 2-4 hours
- **Medium projects:** 1-2 days
- **Large projects:** 3-7 days
- **Complex projects:** 1-2 weeks

Main time factors: New Architecture migration, custom native code updates, testing.

### Q: Can I migrate gradually?
**A:** Yes, several migration strategies:
- Use legacy file system API temporarily
- Enable New Architecture in development first
- Test new features in isolated branches
- Use feature flags for gradual rollout

### Q: What if my app breaks after upgrading?
**A:** Follow this troubleshooting process:
1. Check [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)
2. Verify [Breaking Changes](./breaking-changes/README.md) compliance
3. Use legacy APIs as temporary workarounds
4. Roll back if necessary and plan gradual migration

### Q: Do I need to update my development environment?
**A:** Yes, update these requirements:
- **Node.js:** 20.19.0+
- **Xcode:** 16.1+ (iOS development)
- **Android SDK:** API 36+ (Android development)
- **EAS CLI:** Latest version

## Performance Questions

### Q: Will my app really build 90% faster?
**A:** iOS build improvements vary by project:
- **Clean builds:** Up to 90% improvement
- **Incremental builds:** 30-50% improvement
- **Depends on:** Project size, dependency count, machine specs
- **Android:** Standard build times (no precompiled frameworks yet)

### Q: How does React Compiler affect my app?
**A:** React Compiler provides automatic optimizations:
- **Eliminates:** Manual useMemo/useCallback usage
- **Improves:** Render performance automatically
- **Reduces:** Bundle size through better optimization
- **Maintains:** Same development experience

### Q: Should I enable New Architecture?
**A:** Yes, for these reasons:
- **Performance:** 30-50% better runtime performance
- **Future-proof:** Required for SDK 55
- **Library support:** Growing ecosystem support
- **Stability:** 75% of apps already use it successfully

## Feature Questions

### Q: Can I use iOS 26 features on older iOS versions?
**A:** Limited support with fallbacks:
- **Liquid Glass icons:** Automatic fallbacks on iOS ≤ 19
- **Expo UI components:** Provide fallback implementations
- **Progressive enhancement:** Implement feature detection

```typescript
const isLiquidGlassAvailable = Platform.OS === 'ios' && 
  parseInt(Platform.Version) >= 26;
```

### Q: Is Android 16 edge-to-edge mandatory?
**A:** Yes, Android 16 mandates edge-to-edge:
- **Cannot be disabled** in Android 16+
- **Must implement** proper safe area handling
- **Update now** to prepare for user device upgrades

### Q: What's the new file system API like?
**A:** Object-oriented and more intuitive:
```typescript
// Old API
const uri = FileSystem.documentDirectory + 'file.txt';
await FileSystem.writeAsStringAsync(uri, content);

// New API  
const file = Directory.documentDirectory.createFile('file.txt');
await file.write(content);
```

## Compatibility Questions

### Q: Which libraries work with SDK 54?
**A:** Most libraries are compatible, but check:
- **Reanimated v4:** Requires New Architecture
- **Flash List v2:** Requires New Architecture  
- **Community packages:** Check individual compatibility

### Q: Can I still use Legacy Architecture?
**A:** Yes, but with important caveats:
- **SDK 54:** Final release to support Legacy Architecture
- **SDK 55:** Will require New Architecture
- **Timeline:** Plan migration before SDK 55 release (Q4 2025)

### Q: What about Expo Go compatibility?
**A:** SDK 54 apps work in Expo Go with limitations:
- Some new packages not included in Expo Go
- Use development builds for full feature access
- New Architecture benefits available in development builds

## Build Questions

### Q: Why can't I submit my iOS app to the App Store?
**A:** Known issue with React Native 0.81.0:
- **Problem:** Precompiled XCFrameworks submission issue
- **Workaround:** Set `buildReactNativeFromSource: false`
- **Fix:** Coming in React Native 0.81.1 (week of August 25)

### Q: How do I configure EAS Build for SDK 54?
**A:** Use updated build images:
```json
{
  "build": {
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1"
      },
      "android": {
        "image": "ubuntu-22.04-android-34"
      }
    }
  }
}
```

### Q: My builds are failing, what should I check?
**A:** Common build failure points:
1. **Node.js version:** Must be 20.19.0+
2. **Xcode version:** Must be 16.1+
3. **Dependencies:** Run `npx expo install --fix`
4. **Clean build:** Try `npx expo prebuild --clean`
5. **Architecture:** Check New vs Legacy Architecture settings

## Development Questions

### Q: Do Metro hotkeys work in iOS simulators?
**A:** Known issue with iOS 18.4+ simulators:
- **Problem:** Hotkeys (Cmd+R, Cmd+M) unreliable
- **Workaround:** Use Expo CLI interactive prompt (r, m, j keys)
- **Status:** Expo team investigating

### Q: How do I monitor React Compiler optimizations?
**A:** Several ways to monitor:
- **Console:** Look for "Experimental React Compiler is enabled"
- **DevTools:** Press J → Components panel
- **Profiler:** Use React DevTools Profiler

### Q: Can I disable React Compiler?
**A:** Yes, if needed:
```javascript
// babel.config.js
module.exports = {
  presets: [
    ['babel-preset-expo', {
      reactCompiler: false
    }]
  ]
};
```

## Troubleshooting Questions

### Q: My app crashes after enabling New Architecture
**A:** Common solutions:
1. Update all dependencies: `npx expo install --fix`
2. Check library compatibility
3. Clear caches: `npx expo r -c`
4. Rebuild: `npx expo prebuild --clean`

### Q: File system operations aren't working
**A:** Check your imports:
```typescript
// Quick fix - use legacy API
import * as FileSystem from 'expo-file-system/legacy';

// Long term - migrate to new API
import { Directory, File } from 'expo-file-system';
```

### Q: Autolinking issues after upgrade
**A:** Try these solutions:
```bash
# Verify autolinking
npx expo-modules-autolinking verify -v

# Reset autolinking
npx expo prebuild --clean

# Use legacy config if needed
{
  "expo": {
    "autolinking": {
      "legacy_shallowReactNativeLinking": true
    }
  }
}
```

## Future Planning Questions

### Q: When will SDK 55 be released?
**A:** Expected timeline:
- **SDK 55 Beta:** Q3 2025  
- **SDK 55 Stable:** Q4 2025
- **Key change:** New Architecture required (no Legacy Architecture support)

### Q: How should I prepare for SDK 55?
**A:** Preparation steps:
1. **Enable New Architecture** in SDK 54
2. **Test thoroughly** with New Architecture
3. **Update dependencies** to New Architecture compatible versions
4. **Plan migration timeline** before SDK 55 release

### Q: Will there be more breaking changes in SDK 55?
**A:** Expected changes:
- **New Architecture:** Mandatory (biggest change)
- **React Native 0.82:** Likely included
- **Library updates:** Continued ecosystem evolution
- **Platform updates:** iOS 27, Android 17 support

## Getting Help

### Q: Where can I get help with SDK 54?
**A:** Multiple support channels:
- **Documentation:** This comprehensive guide
- **GitHub Issues:** [expo/expo](https://github.com/expo/expo/issues)
- **Discord:** [Expo Community](https://chat.expo.dev/)
- **Forums:** [Expo Forums](https://forums.expo.dev/)

### Q: How do I report bugs in SDK 54?
**A:** Bug reporting process:
1. **Check existing issues** in GitHub
2. **Create minimal reproduction** case
3. **Include environment info** (versions, platform, etc.)
4. **Tag with "SDK 54 beta"** for priority
5. **Provide detailed steps** to reproduce

### Q: Can I get help with migration?
**A:** Yes, multiple resources:
- **Community:** Discord and forums for community help
- **Documentation:** Comprehensive migration guides
- **Professional:** Consider consulting services for complex migrations
- **Workshops:** Expo team occasionally hosts migration workshops

## Best Practices Questions

### Q: What's the recommended project structure for SDK 54?
**A:** Feature-based organization:
```
src/
├── features/
│   └── auth/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       └── types/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── types/
```

### Q: How should I handle environment variables?
**A:** Use Expo's built-in support:
```typescript
// Use EXPO_PUBLIC_ prefix for client-side variables
const apiUrl = process.env.EXPO_PUBLIC_API_URL;

// Use EAS secrets for sensitive data
// Set via: eas secret:create --name SECRET_KEY --value "value"
```

### Q: What's the best testing strategy for SDK 54?
**A:** Multi-layered approach:
- **Unit tests:** Component and utility testing
- **Integration tests:** Feature workflow testing  
- **E2E tests:** Critical user path testing
- **Performance tests:** Build and runtime benchmarks
- **Platform tests:** iOS and Android specific testing

---

**Still have questions?** Check:
- [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) for technical issues
- [Migration Guide](./migration-guide/README.md) for upgrade help
- [Best Practices](./BEST_PRACTICES.md) for development guidance
- [Expo Discord](https://chat.expo.dev/) for community support
