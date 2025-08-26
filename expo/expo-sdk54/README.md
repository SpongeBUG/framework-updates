# Expo SDK 54 Beta - Comprehensive Documentation
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/README.md -->
<!-- Purpose: Main overview of Expo SDK 54 features and changes -->
<!-- Dependencies: React Native 0.81, React 19.1.0 -->

## Overview

Expo SDK 54 Beta represents a major leap forward in React Native development, introducing React Native 0.81 with React 19.1.0, significant performance improvements, and enhanced platform capabilities. This release focuses on build performance, modern iOS features, Android edge-to-edge compliance, and developer experience improvements.

## Key Highlights

### 🚀 **Performance Revolution**
- **90% faster iOS builds** with precompiled XCFrameworks
- React Native build times reduced from 120s to 10s on M4 Max
- Significant improvements for all project sizes

### 🎨 **Modern Platform Features**
- **iOS 26 Liquid Glass** icons and UI components
- **Android 16 (API 36)** target with mandatory edge-to-edge
- **Apple TV & Android TV** expanded support

### ⚡ **Developer Experience**
- **React Compiler** enabled by default
- Enhanced **Expo Updates** with progress tracking
- Improved **Expo CLI** with better error handling
- **New Architecture** transition (final Legacy Architecture support)

### 📦 **New Packages**
- `expo-app-integrity` - App store verification
- `expo/blob` - Binary large object handling
- Enhanced `expo-file-system` (new default API)
- `expo-sqlite` with localStorage web API

## Architecture Support Notice

**⚠️ Important:** SDK 54 is the **final release** to support Legacy Architecture. SDK 55 will require New Architecture migration.

## Quick Start

```bash
# New project
npx create-expo-app@latest --template default@next

# Existing project upgrade
npx expo install expo@next --fix
```

## Documentation Structure

This documentation is organized into focused categories for easy navigation:

### 📋 [Core Platform Changes](./core-platform-changes/README.md)
React Native 0.81, React 19.1.0, and fundamental platform updates

### 🆕 [New Features](./new-features/README.md) 
New packages, APIs, and capabilities introduced in SDK 54

### ⚡ [Performance Improvements](./performance-improvements/README.md)
Build optimization, runtime enhancements, and developer tools

### 💔 [Breaking Changes](./breaking-changes/README.md)
Required migrations, deprecations, and compatibility notes

### 🛠️ [Development Tools](./development-tools/README.md)
Expo CLI updates, debugging improvements, and workflow enhancements

### 📱 [Platform-Specific Updates](./platform-specific/README.md)
iOS 26, Android 16, Apple TV, and Android TV specific features

### 🔄 [Migration Guide](./migration-guide/README.md)
Step-by-step upgrade instructions and troubleshooting

### 📚 [API Reference](./api-reference/README.md)
Detailed API documentation for new and updated packages

## Release Information

- **Release Type:** Beta (stable release TBD)
- **React Native Version:** 0.81.0
- **React Version:** 19.1.0
- **Beta Duration:** ~2 weeks
- **Minimum Requirements:**
  - Node.js 20.19.x+
  - Xcode 16.1+ (iOS)
  - Android SDK 36+ (Android)

## Beta Testing

During the beta period, continuous fixes and improvements will be released, potentially including breaking changes. Join the [Expo Discord](https://chat.expo.dev/) for office hours and community support.

## Known Issues & Limitations

- iOS 18.4+ simulator hotkeys may not work reliably
- React Native 0.81.0 precompiled XCFrameworks submission issue (fixed in 0.81.1)
- Expo CLI Live Bindings edge cases with default-only import cycles

## Next Steps

1. Review [Breaking Changes](./breaking-changes/README.md) for your project
2. Follow the [Migration Guide](./migration-guide/README.md) for upgrading
3. Explore [New Features](./new-features/README.md) for enhancement opportunities
4. Check [Platform-Specific Updates](./platform-specific/README.md) for your target platforms

---

**Last Updated:** August 2025 | **SDK Version:** 54 Beta | **Maintainer:** Framework Updates Team
