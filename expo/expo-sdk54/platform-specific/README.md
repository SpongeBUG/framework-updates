# Platform-Specific Updates
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/platform-specific/README.md -->
<!-- Purpose: iOS 26, Android 16, Apple TV, and Android TV specific features -->
<!-- Dependencies: iOS 26 SDK, Android 16 API 36, Xcode 16.1+ -->

## iOS 26 Features

### Liquid Glass Design System
iOS 26 introduces Liquid Glass visual effects, bringing premium system-level aesthetics to your apps.

#### Icon Composer Integration
```json
// app.json
{
  "ios": {
    "icon": "./assets/app.icon"
  }
}
```

**Requirements:**
- macOS with Icon Composer app (macOS only)
- Outputs `.icon` file format
- Automatic fallback on iOS ≤ 19

#### Creating Liquid Glass Icons
1. Download Icon Composer from Mac App Store
2. Import your base icon design
3. Configure Liquid Glass properties
4. Export as `.icon` file
5. Reference in app.json

### Expo UI Liquid Glass Components (Beta)

#### Glass Effect Modifiers
```typescript
import { Host, HStack, Text } from "@expo/ui/swift-ui";
import { glassEffect, padding } from '@expo/ui/build/swift-ui/modifiers';

function GlassCard({ children }) {
  return (
    <Host matchContents>
      <HStack
        alignment="center"
        modifiers={[
          padding({ all: 16 }),
          glassEffect({
            glass: { 
              variant: 'regular' // 'regular', 'thin', 'thick'
            }
          })
        ]}
      >
        {children}
      </HStack>
    </Host>
  );
}
```

#### Glass Button Variants
```typescript
import { Button, Host, VStack } from "@expo/ui/swift-ui";

function GlassButtons() {
  return (
    <Host matchContents>
      <VStack spacing={8}>
        <Button
          variant="glassProminent"
          color="orange"
          systemImage="questionmark.circle"
        >
          Help & Support
        </Button>
        
        <Button 
          variant="glass" 
          systemImage="person"
          color="blue"
        >
          View Profile
        </Button>
        
        <Button
          variant="glassSecondary"
          systemImage="gear"
        >
          Settings
        </Button>
      </VStack>
    </Host>
  );
}
```

#### Advanced Glass Effects
```typescript
import { glassEffect, cornerRadius, shadow } from '@expo/ui/build/swift-ui/modifiers';

const advancedGlassModifiers = [
  cornerRadius(12),
  glassEffect({
    glass: { 
      variant: 'thick',
      tint: { color: 'blue', intensity: 0.3 }
    },
    background: {
      blur: 'systemMaterial',
      opacity: 0.8
    }
  }),
  shadow({
    color: 'black',
    radius: 10,
    x: 0,
    y: 5
  })
];
```

### Glass Container Effects (Coming Soon)
Support for `GlassContainerEffect` to combine multiple views with unified glass appearance:

```typescript
// Expected API (in development)
import { GlassContainer, GlassItem } from "@expo/ui/swift-ui";

<GlassContainer effect="unified">
  <GlassItem priority="primary">
    <Text>Primary Content</Text>
  </GlassItem>
  <GlassItem priority="secondary">
    <Text>Secondary Content</Text>
  </GlassItem>
</GlassContainer>
```

### EAS Build Xcode 26 Support
```json
// eas.json
{
  "build": {
    "preview": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0"
      }
    },
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0"
      }
    }
  }
}
```

#### Build Requirements
- **Required for iOS 26 features:** Liquid Glass, new APIs
- **Backwards Compatible:** Builds work on older iOS versions
- **Performance:** Benefits from precompiled frameworks

## Android 16 (API 36) Changes

### Mandatory Edge-to-Edge
Edge-to-edge is now **always enabled** and cannot be disabled:

```javascript
// No longer configurable - always enabled
import { StatusBar } from 'expo-status-bar';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function App() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ 
      flex: 1,
      paddingTop: insets.top,
      paddingBottom: insets.bottom 
    }}>
      <StatusBar style="auto" />
      {/* Your content */}
    </View>
  );
}
```

#### Migration from react-native-edge-to-edge
```bash
# If using react-native-edge-to-edge config plugin only:
npm uninstall react-native-edge-to-edge

# If using the package directly:
npx expo install react-native-edge-to-edge
```

#### Navigation Bar Contrast
```json
// app.json - replaces enforceNavigationBarContrast config
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

### Predictive Back Gesture
Enabled by default in new projects, opt-in for existing:

```json
// app.json
{
  "expo": {
    "android": {
      "predictiveBackGestureEnabled": true
    }
  }
}
```

#### Implementation Guidelines
```typescript
import { BackHandler } from 'react-native';
import { useEffect } from 'react';

function Screen() {
  useEffect(() => {
    const backAction = () => {
      // Handle predictive back gesture
      // Return true to prevent default behavior
      // Return false to allow default behavior
      
      if (hasUnsavedChanges) {
        showSaveDialog();
        return true; // Prevent navigation
      }
      
      return false; // Allow navigation
    };

    const backHandler = BackHandler.addEventListener(
      'hardwareBackPress',
      backAction
    );

    return () => backHandler.remove();
  }, []);

  // Component content
}
```

### Build Configuration
```json
// app.json - Android 16 target
{
  "expo": {
    "android": {
      "compileSdkVersion": 36,
      "targetSdkVersion": 36,
      "buildToolsVersion": "36.0.0"
    }
  }
}
```

## Apple TV Platform Support

### Expanded Package Compatibility
New Apple TV support in core packages:

#### Newly Supported Packages
- **expo-sqlite:** Full database functionality
- **expo-background-task:** Background processing
- **expo-task-manager:** Task scheduling and management
- **expo-insights:** Analytics and performance monitoring
- **expo-image-loader:** Image loading and caching
- **expo-image-manipulator:** Image processing
- **expo-video-thumbnails:** Video thumbnail generation

#### Usage Examples
```typescript
// expo-sqlite on Apple TV
import { openDatabase } from 'expo-sqlite';

const db = openDatabase('tv-app.db');

// Create TV-specific tables
await db.execAsync(`
  CREATE TABLE IF NOT EXISTS tv_preferences (
    id INTEGER PRIMARY KEY,
    channel TEXT,
    last_watched INTEGER
  );
`);

// expo-insights on Apple TV
import * as Insights from 'expo-insights';

await Insights.initialize({
  platform: 'tvOS',
  deviceType: 'appleTV'
});
```

### Development Client Support (Experimental)
```bash
# Install development client on Apple TV
npx expo install expo-dev-client

# Known limitations:
# - No Expo account authentication yet
# - Basic project loading and testing
# - Hot reloading works
```

#### Current Capabilities
- **Project Loading:** Load projects via QR code or manual URL
- **Hot Reloading:** Fast refresh during development
- **Error Display:** Runtime error reporting
- **Basic Debugging:** Console output and basic debugging

### Build Performance Optimization
tvOS builds leverage precompiled frameworks:

```bash
# Build times significantly reduced
# Similar to iOS improvements (90% faster)
Before: ~100-120 seconds
After: ~10-15 seconds
```

### TV-Specific Development
```typescript
import { Platform } from 'react-native';
import { useFocusEffect } from '@react-navigation/native';

function TVScreen() {
  // Detect Apple TV platform
  const isAppleTV = Platform.OS === 'ios' && Platform.isTV;
  
  // Handle TV focus behavior
  useFocusEffect(
    useCallback(() => {
      if (isAppleTV) {
        // TV-specific focus handling
        setupTVFocus();
      }
      
      return () => {
        if (isAppleTV) {
          cleanupTVFocus();
        }
      };
    }, [isAppleTV])
  );

  return (
    <View>
      {isAppleTV ? <TVOptimizedUI /> : <MobileUI />}
    </View>
  );
}
```

## Android TV Platform Support

### Full Development Client Support
Complete Android TV support with all features:

```bash
# Full feature set available
npx expo install expo-dev-client

# Includes:
# - Expo account authentication
# - Complete development workflow
# - All debugging features
# - Performance optimization
```

#### Features
- **Account Integration:** Full Expo account support
- **Project Management:** Complete project workflow
- **Debugging Tools:** All debugging capabilities
- **Performance Monitoring:** Real-time performance metrics

### TV Interface Optimization
```typescript
import { Platform } from 'react-native';
import { useTVEventHandler } from 'react-native';

function AndroidTVScreen() {
  const [tvEventHandler, setTVEventHandler] = useState(null);
  
  useEffect(() => {
    if (Platform.isTV && Platform.OS === 'android') {
      const handler = (component, data) => {
        // Handle Android TV remote events
        switch (data.eventType) {
          case 'select':
            handleTVSelect();
            break;
          case 'up':
          case 'down':
          case 'left':
          case 'right':
            handleTVNavigation(data.eventType);
            break;
        }
      };
      
      setTVEventHandler(handler);
    }
  }, []);

  return (
    <View accessible={true} focusable={true}>
      <TVOptimizedContent />
    </View>
  );
}
```

## Cross-Platform Considerations

### Platform Detection
```typescript
import { Platform } from 'react-native';

const PlatformInfo = {
  isIOS: Platform.OS === 'ios',
  isAndroid: Platform.OS === 'android',
  isTV: Platform.isTV,
  isAppleTV: Platform.OS === 'ios' && Platform.isTV,
  isAndroidTV: Platform.OS === 'android' && Platform.isTV,
  
  // Version checks
  isiOS26Plus: Platform.OS === 'ios' && 
               parseInt(Platform.Version) >= 26,
  isAndroid16Plus: Platform.OS === 'android' && 
                   Platform.Version >= 36,
};

// Usage in components
function ResponsiveComponent() {
  if (PlatformInfo.isiOS26Plus) {
    return <LiquidGlassComponent />;
  }
  
  if (PlatformInfo.isAndroid16Plus) {
    return <EdgeToEdgeComponent />;
  }
  
  return <StandardComponent />;
}
```

### TV-Optimized Layouts
```typescript
import { Platform, Dimensions } from 'react-native';

function TVLayout({ children }) {
  const { width, height } = Dimensions.get('screen');
  const isTV = Platform.isTV;
  
  const layoutStyles = {
    container: {
      flex: 1,
      // TV-specific margins for safe viewing area
      paddingHorizontal: isTV ? width * 0.05 : 16,
      paddingVertical: isTV ? height * 0.05 : 16,
    },
    content: {
      // Larger touch targets for TV remote
      minHeight: isTV ? 60 : 44,
      // Better focus visibility
      borderRadius: isTV ? 8 : 4,
    }
  };

  return (
    <View style={layoutStyles.container}>
      <View style={layoutStyles.content}>
        {children}
      </View>
    </View>
  );
}
```

---

**Platform-Specific Resources:**
- [iOS 26 Documentation](https://developer.apple.com/ios/)
- [Android 16 Migration Guide](https://developer.android.com/about/versions/16)
- [Apple TV Development](https://developer.apple.com/tvos/)
- [Android TV Development](https://developer.android.com/tv)

**Next Steps:**
- Review [Migration Guide](../migration-guide/README.md) for platform-specific updates
- Check [API Reference](../api-reference/README.md) for detailed implementation guides