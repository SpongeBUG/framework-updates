# Glossary of Terms
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/GLOSSARY.md -->
<!-- Purpose: Definitions of terms and concepts used in SDK 54 documentation -->
<!-- Dependencies: SDK 54 terminology and concepts -->

## Core Concepts

### **API Level (Android)**
The version number for Android SDK features. SDK 54 targets API 36 (Android 16).

### **App Integrity**
Security verification system using platform-specific APIs (DeviceCheck on iOS, Play Integrity on Android) to confirm app authenticity and device security.

### **Architecture (New vs Legacy)**
- **Legacy Architecture:** React Native's original bridge-based communication system
- **New Architecture:** Modern React Native architecture with TurboModules and Fabric

### **Autolinking**
Automatic linking system that connects JavaScript modules to their native implementations without manual configuration.

### **babel-preset-expo**
Babel preset that configures transformations and optimizations for Expo projects, including React Compiler integration.

### **Blob**
Binary Large Object - a data structure for handling binary data with Web API compatibility.

## Build and Development

### **Clean Build**
Building a project from scratch without using any cached artifacts, ensuring all dependencies are freshly compiled.

### **Development Client**
Custom version of Expo Go that includes your app's native dependencies, allowing testing of features not available in standard Expo Go.

### **EAS (Expo Application Services)**
Expo's hosted services for building, submitting, and updating apps.

### **EAS Build**
Expo's cloud-based build service that compiles your app for iOS and Android.

### **Edge-to-Edge**
Android design approach where app content extends to screen edges, behind system UI elements like status bar and navigation bar.

### **Expo CLI**
Command-line interface for Expo development tasks like starting development servers and managing projects.

### **Expo Go**
Development app for iOS and Android that allows testing Expo projects without building native apps.

### **Hot Reload / Fast Refresh**
Development feature that updates your app instantly when code changes are saved.

### **Metro**
JavaScript bundler used by React Native and Expo to bundle your app's JavaScript code.

### **Prebuild**
Process of generating native iOS and Android projects from your Expo configuration.

### **Precompiled XCFrameworks**
Pre-built iOS frameworks that reduce compilation time by eliminating the need to compile React Native from source.

## React and JavaScript

### **JSC (JavaScriptCore)**
JavaScript engine that was previously built into React Native but removed in 0.81.0.

### **Hermes**
JavaScript engine optimized for React Native, providing better performance and smaller bundle sizes.

### **React Compiler**
Tool that automatically optimizes React components, eliminating the need for manual `useMemo` and `useCallback` usage.

### **React 19**
Latest version of React included in SDK 54, bringing improved performance and new features.

### **TurboModules**
New system for JavaScript-native communication in React Native's New Architecture, providing better performance than the legacy bridge.

### **Fabric**
New rendering system in React Native's New Architecture that provides synchronous layout and better performance.

## Platform Features

### **Liquid Glass**
iOS 26 design system featuring glass-like visual effects with blur, transparency, and depth.

### **Icon Composer**
Apple's macOS app for creating Liquid Glass icons that output .icon files.

### **Predictive Back Gesture**
Android feature that shows preview of destination screen during back navigation gesture.

### **Safe Area**
Screen regions that are guaranteed to not be obscured by system UI elements.

### **Safe Area Insets**
Measurements of how much content should be inset from screen edges to avoid system UI overlap.

### **Status Bar**
Top system UI showing time, battery, signal strength, etc.

### **Navigation Bar (Android)**
Bottom system UI on Android containing navigation buttons or gesture indicators.

## File System

### **Document Directory**
App-specific directory for storing user documents and data that persists across app updates.

### **Cache Directory**
App-specific directory for storing temporary files that may be cleared by the system.

### **Bundle Directory**
Read-only directory containing files included with your app at build time.

### **SAF (Storage Access Framework)**
Android system for accessing files from external storage providers with user permission.

### **URI vs Path**
- **URI:** Universal Resource Identifier (e.g., `content://` on Android)
- **Path:** Traditional file system path (e.g., `/path/to/file`)

## Performance and Optimization

### **Bundle Size**
Total size of JavaScript code and assets included in your app.

### **Cold Start**
Time it takes for an app to start from a completely closed state.

### **Frame Rate**
Number of screen updates per second, ideally 60 FPS for smooth animations.

### **Memory Usage**
Amount of RAM your app consumes during operation.

### **Tree Shaking**
Build optimization that removes unused code from your bundle.

### **Code Splitting**
Technique of splitting your app's code into smaller chunks that can be loaded on demand.

## TV Development

### **Apple TV**
Apple's set-top box device running tvOS, a variant of iOS optimized for television.

### **Android TV**
Google's smart TV platform based on Android, optimized for television interfaces.

### **Focus Management**
System for handling navigation between UI elements using remote control or directional input.

### **Remote Control Navigation**
Navigation paradigm for TV interfaces using directional keys instead of touch input.

## Build Configuration

### **Build Profile**
Named configuration in eas.json defining build settings for different scenarios (development, preview, production).

### **Resource Class**
Computational resources allocated for EAS Build (e.g., m1-medium, m1-large).

### **Build Image**
Virtual machine environment used for building your app (specifies OS and tool versions).

### **Artifact**
Output file from a build process (e.g., .ipa for iOS, .apk/.aab for Android).

### **Code Signing**
Process of digitally signing your app to verify its authenticity and integrity.

## Version and Release Management

### **SDK Version**
Version number of Expo SDK (e.g., SDK 54) that determines available features and compatibility.

### **Semantic Versioning (SemVer)**
Version numbering scheme using MAJOR.MINOR.PATCH format.

### **Beta Release**
Pre-release version for testing before stable release.

### **LTS (Long Term Support)**
Versions that receive extended support and security updates.

### **Deprecation**
Process of phasing out old features, typically with warnings before removal.

## Security and Integrity

### **DeviceCheck (iOS)**
Apple's framework for verifying app integrity and device authenticity.

### **Play Integrity API (Android)**
Google's service for verifying app integrity and detecting tampered devices.

### **Attestation Token**
Cryptographic proof of app and device integrity.

### **Certificate Pinning**
Security technique that validates server certificates against known good certificates.

## Development Workflow

### **Development Build**
Custom build of your app that includes development tools and debugging capabilities.

### **Preview Build**
Build for internal testing that mimics production but may include additional debugging features.

### **Production Build**
Optimized build ready for app store submission and end-user distribution.

### **OTA (Over-the-Air) Updates**
System for updating app content without going through app stores.

### **Expo Updates**
Expo's service for delivering OTA updates to your published apps.

## Platform-Specific Terms

### **iOS Terms**
- **Xcode:** Apple's IDE for iOS development
- **App Store Connect:** Apple's portal for managing App Store submissions
- **TestFlight:** Apple's beta testing service
- **Provisioning Profile:** Certificate that allows app installation on specific devices
- **Bundle ID:** Unique identifier for iOS apps

### **Android Terms**
- **Android Studio:** Google's IDE for Android development
- **Google Play Console:** Google's portal for managing Play Store submissions
- **APK:** Android Package file format
- **AAB (Android App Bundle):** Modern Android app publishing format
- **Package Name:** Unique identifier for Android apps

## Debugging and Testing

### **Error Boundary**
React component that catches JavaScript errors anywhere in the component tree.

### **Stack Trace**
List of function calls that led to an error, useful for debugging.

### **Profiler**
Tool for measuring app performance, particularly component render times.

### **DevTools**
Browser-based or standalone tools for debugging and profiling React applications.

### **Simulator/Emulator**
Software that mimics mobile device behavior for testing without physical devices.

## Networking and APIs

### **API Endpoint**
URL where your app sends requests to communicate with backend services.

### **GraphQL**
Query language and runtime for APIs, alternative to REST.

### **REST API**
Architectural style for web services using HTTP requests.

### **WebSocket**
Protocol for real-time, bidirectional communication between client and server.

### **Fetch API**
Modern JavaScript API for making HTTP requests.

## State Management

### **Context API**
React's built-in state management solution for sharing data across components.

### **Redux**
Popular state management library for JavaScript applications.

### **Zustand**
Lightweight state management library alternative to Redux.

### **State Colocation**
Practice of keeping state close to where it's used rather than in global state.

## Common Abbreviations

| Abbreviation | Full Form | Description |
|-------------|-----------|-------------|
| **API** | Application Programming Interface | Set of rules for software communication |
| **CI/CD** | Continuous Integration/Continuous Deployment | Automated build and deployment process |
| **CLI** | Command Line Interface | Text-based interface for running commands |
| **DOM** | Document Object Model | Programming interface for web documents |
| **IDE** | Integrated Development Environment | Software for writing and debugging code |
| **JSON** | JavaScript Object Notation | Data interchange format |
| **NPM** | Node Package Manager | Package manager for JavaScript |
| **OTA** | Over-the-Air | Updates delivered without app store |
| **SDK** | Software Development Kit | Tools and libraries for development |
| **UI/UX** | User Interface/User Experience | App design and usability |
| **URL** | Uniform Resource Locator | Web address |
| **UUID** | Universally Unique Identifier | Unique identifier standard |

## File Extensions

| Extension | Description | Usage |
|-----------|-------------|--------|
| **.tsx** | TypeScript React component | React components with TypeScript |
| **.ts** | TypeScript file | TypeScript code files |
| **.jsx** | JavaScript React component | React components with JavaScript |
| **.js** | JavaScript file | JavaScript code files |
| **.json** | JSON file | Configuration and data files |
| **.md** | Markdown file | Documentation files |
| **.yaml/.yml** | YAML file | Configuration files |
| **.icon** | Icon Composer file | iOS 26 Liquid Glass icons |

## Configuration Files

### **app.json / app.config.js**
Main Expo configuration file defining app metadata, build settings, and plugin configurations.

### **eas.json**
Configuration file for EAS Build and Submit services.

### **package.json**
Node.js package configuration file listing dependencies and scripts.

### **tsconfig.json**
TypeScript compiler configuration file.

### **babel.config.js**
Babel transformer configuration, includes React Compiler settings.

### **metro.config.js**
Metro bundler configuration file.

### **.env**
Environment variables file for storing configuration values.

### **google-services.json (Android)**
Firebase configuration file for Android.

### **GoogleService-Info.plist (iOS)**
Firebase configuration file for iOS.

---

**Need more clarification on any term?**
- Check the specific documentation section for detailed explanations
- Search this glossary with Ctrl/Cmd+F
- Ask in [Expo Discord](https://chat.expo.dev/) for community help
- Reference official documentation at [docs.expo.dev](https://docs.expo.dev/)

**Contributing to this glossary:**
If you encounter terms not defined here, please suggest additions through GitHub issues or community discussions.
