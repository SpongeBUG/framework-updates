# Android 16 Features Deep Dive
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/platform-specific/android16-features.md -->
<!-- Purpose: Comprehensive guide to Android 16 features and requirements -->
<!-- Dependencies: Android 16 API 36, Expo SDK 54 -->

## Overview

Android 16 (API level 36) introduces significant changes that affect all apps, with mandatory edge-to-edge enforcement and enhanced predictive back gestures. Expo SDK 54 ensures full compliance with these requirements.

## Edge-to-Edge Enforcement

### What Changed
Android 16 **mandates** edge-to-edge display for all applications:

- **Cannot be disabled** - No opt-out mechanism
- **System enforced** - Operating system handles enforcement
- **Breaking change** - Apps must adapt their layouts
- **Universal requirement** - Affects all apps regardless of target SDK

### Implementation Requirements

#### Safe Area Handling
```typescript
import React from 'react';
import { View, Text, ScrollView, Platform } from 'react-native';
import { 
  useSafeAreaInsets, 
  SafeAreaProvider,
  SafeAreaView 
} from 'react-native-safe-area-context';
import { StatusBar } from 'expo-status-bar';

// Essential edge-to-edge setup
function EdgeToEdgeApp() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1 }}>
      <StatusBar style="auto" translucent />
      
      {/* Content that respects system UI */}
      <View style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }}>
        {/* Your app content */}
      </View>
    </View>
  );
}

// App setup with SafeAreaProvider
export default function App() {
  return (
    <SafeAreaProvider>
      <EdgeToEdgeApp />
    </SafeAreaProvider>
  );
}
```

#### Advanced Layout Patterns
```typescript
// Full-bleed content with safe content areas
function AdvancedEdgeToEdgeLayout() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1 }}>
      {/* Full-width header that extends to edges */}
      <View style={{
        backgroundColor: '#2196F3',
        paddingTop: insets.top + 16,
        paddingBottom: 16,
        paddingHorizontal: Math.max(insets.left, 16), // Respect notches/curves
      }}>
        <Text style={{
          color: 'white',
          fontSize: 20,
          fontWeight: 'bold',
          textAlign: 'center',
        }}>
          Edge-to-Edge Header
        </Text>
      </View>
      
      {/* Scrollable content with proper insets */}
      <ScrollView
        style={{ flex: 1 }}
        contentContainerStyle={{
          paddingHorizontal: Math.max(insets.left, 16),
          paddingBottom: Math.max(insets.bottom, 16),
        }}
        // Ensure scroll indicators respect safe areas
        scrollIndicatorInsets={{
          top: 0,
          left: insets.left,
          bottom: insets.bottom,
          right: insets.right,
        }}
      >
        {/* Your scrollable content */}
        <View style={{ padding: 16 }}>
          <Text>Content that respects safe areas</Text>
        </View>
      </ScrollView>
      
      {/* Bottom bar that extends to edges */}
      <View style={{
        backgroundColor: '#F5F5F5',
        borderTopWidth: 1,
        borderTopColor: '#E0E0E0',
        paddingTop: 16,
        paddingBottom: Math.max(insets.bottom, 16),
        paddingHorizontal: Math.max(insets.left, 16),
      }}>
        <Text style={{ textAlign: 'center', fontWeight: '500' }}>
          Bottom Navigation
        </Text>
      </View>
    </View>
  );
}
```

#### React Navigation Integration
```typescript
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const Stack = createStackNavigator();

function NavigationWithEdgeToEdge() {
  const insets = useSafeAreaInsets();
  
  return (
    <NavigationContainer>
      <Stack.Navigator
        screenOptions={{
          // Configure header to work with edge-to-edge
          headerStyle: {
            paddingTop: insets.top,
            height: 56 + insets.top, // Standard header height + status bar
          },
          // Ensure content doesn't overlap with system UI
          safeAreaInsets: insets,
        }}
      >
        <Stack.Screen 
          name="Home" 
          component={HomeScreen}
          options={{
            headerTitle: 'Edge-to-Edge App',
            headerTransparent: true, // Allow content behind header
            headerBackground: () => (
              <View style={{
                flex: 1,
                backgroundColor: 'rgba(255, 255, 255, 0.95)',
              }} />
            ),
          }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

### Navigation Bar Configuration

#### App Configuration
```json
// app.json - Configure navigation bar appearance
{
  "expo": {
    "android": {
      "androidNavigationBar": {
        "enforceContrast": false,
        "backgroundColor": "transparent",
        "barStyle": "dark-content"
      }
    }
  }
}
```

#### Migration from react-native-edge-to-edge
```bash
# If you were using react-native-edge-to-edge config plugin only:
npm uninstall react-native-edge-to-edge

# If you're using the package functionality:
npx expo install react-native-edge-to-edge

# Update configuration
# Remove from plugins if only using enforceNavigationBarContrast
# Use androidNavigationBar.enforceContrast in app.json instead
```

#### Runtime Navigation Bar Control
```typescript
import { NavigationBar } from 'expo-navigation-bar';

// Control navigation bar appearance at runtime
async function configureNavigationBar() {
  // Set background color
  await NavigationBar.setBackgroundColorAsync('transparent');
  
  // Set button style
  await NavigationBar.setButtonStyleAsync('dark');
  
  // Configure contrast (replaces enforceNavigationBarContrast)
  // This is now handled via app.json androidNavigationBar.enforceContrast
}
```

## Predictive Back Gesture

### Overview
Android 16 enhances the predictive back gesture system:

- **Visual Previews:** Users see destination before completing gesture
- **Smooth Animations:** Fluid transitions between screens
- **Gesture Cancellation:** Users can cancel navigation mid-gesture
- **Cross-app Consistency:** Standardized behavior across all apps

### Configuration

#### Enable for New Projects
```json
// app.json - Automatically enabled for new projects
{
  "expo": {
    "android": {
      "predictiveBackGestureEnabled": true
    }
  }
}
```

#### Enable for Existing Projects
Existing projects need manual enablement:

```json
// app.json - Add this to existing projects
{
  "expo": {
    "android": {
      "predictiveBackGestureEnabled": true
    }
  }
}
```

### Implementation

#### Basic Back Handler
```typescript
import { BackHandler, Alert } from 'react-native';
import { useEffect, useCallback } from 'react';
import { useFocusEffect } from '@react-navigation/native';

function ScreenWithBackHandler() {
  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);
  
  const handleBackPress = useCallback(() => {
    if (hasUnsavedChanges) {
      // Show confirmation dialog for predictive back
      Alert.alert(
        'Unsaved Changes',
        'You have unsaved changes. Are you sure you want to leave?',
        [
          { text: 'Cancel', style: 'cancel', onPress: () => false },
          { 
            text: 'Leave', 
            style: 'destructive', 
            onPress: () => {
              setHasUnsavedChanges(false);
              // Allow back navigation
              return false;
            }
          },
        ]
      );
      
      // Prevent default back behavior
      return true;
    }
    
    // Allow default back behavior
    return false;
  }, [hasUnsavedChanges]);

  useFocusEffect(
    useCallback(() => {
      const backSubscription = BackHandler.addEventListener(
        'hardwareBackPress',
        handleBackPress
      );

      return backSubscription.remove;
    }, [handleBackPress])
  );

  return (
    <View>
      {/* Your screen content */}
    </View>
  );
}
```

#### React Navigation Integration
```typescript
import { usePreventRemove } from '@react-navigation/native';

function FormScreen({ navigation }) {
  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);

  // Integrate with React Navigation's prevent remove
  usePreventRemove(hasUnsavedChanges, ({ data }) => {
    Alert.alert(
      'Discard changes?',
      'You have unsaved changes. Discard them and leave the screen?',
      [
        { text: "Don't leave", style: 'cancel', onPress: () => {} },
        {
          text: 'Discard',
          style: 'destructive',
          onPress: () => navigation.dispatch(data.action),
        },
      ]
    );
  });

  return (
    <View>
      {/* Form content */}
      <TextInput
        onChangeText={(text) => {
          setHasUnsavedChanges(text.length > 0);
        }}
      />
    </View>
  );
}
```

#### Advanced Gesture Handling
```typescript
// Custom hook for predictive back gesture
function usePredictiveBackGesture(onBackGesture: () => boolean) {
  useEffect(() => {
    const subscription = BackHandler.addEventListener('hardwareBackPress', () => {
      // Custom logic for predictive back
      const shouldPreventBack = onBackGesture();
      
      if (shouldPreventBack) {
        // Provide visual feedback that back was prevented
        // This helps users understand the predictive gesture
        console.log('Back navigation prevented');
        return true;
      }
      
      return false;
    });

    return subscription.remove;
  }, [onBackGesture]);
}

// Usage in component
function MediaPlayerScreen() {
  const [isPlaying, setIsPlaying] = useState(false);
  
  const handleBackGesture = useCallback(() => {
    if (isPlaying) {
      // Pause media and show predictive back preview
      setIsPlaying(false);
      
      // Show preview of previous screen
      // The system will handle the preview animation
      return false; // Allow back navigation after pausing
    }
    
    return false;
  }, [isPlaying]);

  usePredictiveBackGesture(handleBackGesture);

  return (
    <View>
      {/* Media player UI */}
    </View>
  );
}
```

## System UI Integration

### Status Bar Considerations
```typescript
import { StatusBar } from 'expo-status-bar';
import { Platform } from 'react-native';

function StatusBarConfiguration() {
  // Android 16 edge-to-edge requires careful status bar handling
  return (
    <>
      <StatusBar 
        style="auto" // Automatically adapt to content
        translucent={Platform.OS === 'android'} // Required for edge-to-edge
        backgroundColor="transparent" // Transparent for edge-to-edge
        animated={true}
      />
      
      {/* Your app content */}
    </>
  );
}

// Dynamic status bar changes based on content
function DynamicStatusBar({ isDark }) {
  return (
    <StatusBar 
      style={isDark ? 'light' : 'dark'}
      backgroundColor={isDark ? '#000' : 'transparent'}
      animated={true}
    />
  );
}
```

### System Gestures and Hotspots
```typescript
// Handle system gesture areas
function GestureAwareLayout() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1 }}>
      {/* Avoid interactive elements in gesture areas */}
      <View style={{
        flex: 1,
        // Avoid bottom gesture area for back navigation
        paddingBottom: Math.max(insets.bottom, 48), // 48dp minimum for gestures
        // Avoid side edges for drawer/navigation gestures
        paddingLeft: Math.max(insets.left, 16),
        paddingRight: Math.max(insets.right, 16),
      }}>
        {/* Interactive content safe from gesture conflicts */}
        <TouchableOpacity style={{ padding: 16 }}>
          <Text>Safe interactive area</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```

## Testing Android 16 Features

### Development Environment
```bash
# Ensure you have Android 16 (API 36) installed
# Via Android Studio SDK Manager:
# - Android 16.0 (API 36)
# - Google APIs Android 36
# - Android SDK Build-Tools 36.0.0

# Update environment variables
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

### Testing Checklist

#### Edge-to-Edge Compliance
- [ ] Status bar content doesn't overlap with app content
- [ ] Navigation bar area is properly handled
- [ ] Content adapts to different screen sizes and orientations
- [ ] Safe areas work correctly on devices with notches/cutouts
- [ ] Scrollable content respects system UI areas

#### Predictive Back Gesture
- [ ] Back gestures show appropriate previews
- [ ] Complex navigation scenarios work correctly
- [ ] Form data preservation works with back gestures
- [ ] Animation performance is smooth
- [ ] Gesture cancellation works properly

### Debug Utilities
```typescript
// Debug component to visualize safe areas
function SafeAreaDebugger() {
  const insets = useSafeAreaInsets();
  
  if (!__DEV__) return null;
  
  return (
    <View style={{
      position: 'absolute',
      top: 0,
      left: 0,
      right: 0,
      bottom: 0,
      pointerEvents: 'none',
    }}>
      {/* Top safe area */}
      <View style={{
        position: 'absolute',
        top: 0,
        left: 0,
        right: 0,
        height: insets.top,
        backgroundColor: 'rgba(255, 0, 0, 0.3)',
      }} />
      
      {/* Bottom safe area */}
      <View style={{
        position: 'absolute',
        bottom: 0,
        left: 0,
        right: 0,
        height: insets.bottom,
        backgroundColor: 'rgba(0, 255, 0, 0.3)',
      }} />
      
      {/* Side safe areas */}
      <View style={{
        position: 'absolute',
        top: 0,
        left: 0,
        width: insets.left,
        bottom: 0,
        backgroundColor: 'rgba(0, 0, 255, 0.3)',
      }} />
      
      <View style={{
        position: 'absolute',
        top: 0,
        right: 0,
        width: insets.right,
        bottom: 0,
        backgroundColor: 'rgba(0, 0, 255, 0.3)',
      }} />
    </View>
  );
}
```

## Performance Optimizations

### Edge-to-Edge Performance
```typescript
// Optimize edge-to-edge layouts for performance
const EdgeToEdgeStyles = StyleSheet.create({
  container: {
    flex: 1,
    // Use backgroundColor instead of View components when possible
    backgroundColor: 'white',
  },
  
  safeContent: {
    flex: 1,
    // Pre-calculate common inset values
    paddingTop: Platform.OS === 'android' ? StatusBar.currentHeight : 0,
  },
  
  // Memoize complex inset calculations
  fullBleedHeader: {
    backgroundColor: '#2196F3',
    // Avoid inline styles for better performance
    paddingVertical: 16,
  },
});

// Use StyleSheet for better performance
function OptimizedEdgeToEdgeLayout() {
  const insets = useSafeAreaInsets();
  
  // Memoize inset-dependent styles
  const dynamicStyles = useMemo(() => ({
    safeContent: {
      ...EdgeToEdgeStyles.safeContent,
      paddingTop: insets.top,
      paddingBottom: insets.bottom,
      paddingLeft: insets.left,
      paddingRight: insets.right,
    },
    fullBleedHeader: {
      ...EdgeToEdgeStyles.fullBleedHeader,
      paddingTop: insets.top + 16,
      paddingHorizontal: Math.max(insets.left, 16),
    },
  }), [insets]);

  return (
    <View style={EdgeToEdgeStyles.container}>
      <View style={dynamicStyles.fullBleedHeader}>
        <Text style={{ color: 'white', textAlign: 'center' }}>
          Optimized Header
        </Text>
      </View>
      
      <View style={dynamicStyles.safeContent}>
        {/* Content */}
      </View>
    </View>
  );
}
```

## Migration Examples

### Common Migration Patterns

#### Pattern 1: Simple App Migration
```typescript
// Before: Basic app layout
function OldAppLayout() {
  return (
    <View style={{ flex: 1, backgroundColor: 'white' }}>
      <View style={{ padding: 16 }}>
        <Text>My App Content</Text>
      </View>
    </View>
  );
}

// After: Android 16 compliant
function NewAppLayout() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1, backgroundColor: 'white' }}>
      <StatusBar style="auto" translucent />
      <View style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingHorizontal: Math.max(insets.left, 16),
      }}>
        <Text>My App Content</Text>
      </View>
    </View>
  );
}
```

#### Pattern 2: Modal Migration
```typescript
// Before: Standard modal
function OldModal({ visible, children }) {
  return (
    <Modal visible={visible} animationType="slide">
      <View style={{ flex: 1, padding: 20 }}>
        {children}
      </View>
    </Modal>
  );
}

// After: Edge-to-edge compliant modal
function NewModal({ visible, children }) {
  const insets = useSafeAreaInsets();
  
  return (
    <Modal visible={visible} animationType="slide">
      <StatusBar style="light" />
      <View style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingHorizontal: Math.max(insets.left, 20),
      }}>
        {children}
      </View>
    </Modal>
  );
}
```

#### Pattern 3: Tab Navigation Migration
```typescript
// Before: Standard bottom tabs
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

// After: Edge-to-edge aware tabs
function EdgeToEdgeTabNavigator() {
  const insets = useSafeAreaInsets();
  
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarStyle: {
          paddingBottom: insets.bottom,
          height: 49 + insets.bottom, // Standard tab height + bottom inset
        },
        tabBarItemStyle: {
          paddingBottom: insets.bottom > 0 ? 0 : 5, // Adjust for devices without bottom inset
        },
      }}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}
```

## Best Practices

### Do's ✅
- Always use `SafeAreaProvider` at the root of your app
- Use `useSafeAreaInsets()` for layout calculations
- Test on devices with different screen shapes (notches, cutouts)
- Handle predictive back gestures appropriately
- Use `translucent` StatusBar for proper edge-to-edge
- Provide visual feedback for prevented back gestures

### Don'ts ❌
- Don't ignore safe area insets
- Don't use fixed padding values for system UI areas
- Don't disable edge-to-edge (it's mandatory)
- Don't ignore predictive back gesture behavior
- Don't place critical UI in gesture areas
- Don't assume uniform insets across all devices

### Testing Strategy
1. **Device Variety:** Test on phones, tablets, foldables
2. **Orientation Testing:** Portrait and landscape modes
3. **Gesture Testing:** All back navigation scenarios
4. **Edge Cases:** Keyboards, modals, overlays
5. **Performance:** Smooth animations and transitions

---

**Next Steps:**
- Update your app's layout to respect safe areas
- Enable predictive back gestures
- Test on Android 16 devices/emulators
- Review navigation flows for gesture compatibility
- Plan for mandatory compliance timeline
