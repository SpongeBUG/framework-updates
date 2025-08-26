# Practical Examples
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/examples/README.md -->
<!-- Purpose: Practical implementation examples for SDK 54 features -->
<!-- Dependencies: Expo SDK 54, TypeScript -->

## Quick Start Examples

### File System Migration Example
Complete example showing migration from legacy to new file system API.

```typescript
// Before (Legacy API)
import * as FileSystem from 'expo-file-system/legacy';

async function legacyFileOperations() {
  const fileUri = FileSystem.documentDirectory + 'data.json';
  
  // Write file
  await FileSystem.writeAsStringAsync(fileUri, JSON.stringify({
    user: 'john',
    preferences: { theme: 'dark' }
  }));
  
  // Read file
  const content = await FileSystem.readAsStringAsync(fileUri);
  const data = JSON.parse(content);
  
  return data;
}

// After (New API)
import { Directory, File } from 'expo-file-system';

async function newFileOperations() {
  const documentsDir = Directory.documentDirectory;
  const dataFile = documentsDir.createFile('data.json');
  
  // Write file
  await dataFile.write(JSON.stringify({
    user: 'john',
    preferences: { theme: 'dark' }
  }));
  
  // Read file
  const content = await dataFile.readAsString();
  const data = JSON.parse(content);
  
  return data;
}

// Gradual migration approach
class FileManager {
  private useNewAPI = __DEV__; // Test new API in development
  
  async saveData(filename: string, data: any) {
    if (this.useNewAPI) {
      const file = Directory.documentDirectory.createFile(filename);
      await file.write(JSON.stringify(data));
    } else {
      const uri = FileSystem.documentDirectory + filename;
      await FileSystem.writeAsStringAsync(uri, JSON.stringify(data));
    }
  }
  
  async loadData(filename: string) {
    if (this.useNewAPI) {
      const file = Directory.documentDirectory.createFile(filename);
      const exists = await file.exists();
      if (!exists) return null;
      
      const content = await file.readAsString();
      return JSON.parse(content);
    } else {
      const uri = FileSystem.documentDirectory + filename;
      const info = await FileSystem.getInfoAsync(uri);
      if (!info.exists) return null;
      
      const content = await FileSystem.readAsStringAsync(uri);
      return JSON.parse(content);
    }
  }
}
```

### React Compiler Optimization Example
See how React Compiler automatically optimizes components.

```typescript
import React, { useState, useMemo, useCallback } from 'react';
import { View, Text, FlatList, TouchableOpacity } from 'react-native';

// ❌ Before: Manual optimization required
function ManuallyOptimizedList({ data, filters }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // Manual memoization
  const filteredData = useMemo(() => {
    return data.filter(item => {
      if (filters.category && item.category !== filters.category) return false;
      if (filters.search && !item.name.toLowerCase().includes(filters.search.toLowerCase())) return false;
      return true;
    }).sort((a, b) => {
      return filters.sortBy === 'name' ? a.name.localeCompare(b.name) : b.score - a.score;
    });
  }, [data, filters.category, filters.search, filters.sortBy]);
  
  // Manual callback memoization
  const handleItemPress = useCallback((id) => {
    setSelectedId(id);
  }, []);
  
  const renderItem = useCallback(({ item }) => (
    <TouchableOpacity onPress={() => handleItemPress(item.id)}>
      <Text style={{ fontWeight: selectedId === item.id ? 'bold' : 'normal' }}>
        {item.name}
      </Text>
    </TouchableOpacity>
  ), [selectedId, handleItemPress]);
  
  return (
    <FlatList
      data={filteredData}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}

// ✅ After: React Compiler handles optimization automatically
function AutoOptimizedList({ data, filters }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // React Compiler automatically optimizes these operations
  const filteredData = data.filter(item => {
    if (filters.category && item.category !== filters.category) return false;
    if (filters.search && !item.name.toLowerCase().includes(filters.search.toLowerCase())) return false;
    return true;
  }).sort((a, b) => {
    return filters.sortBy === 'name' ? a.name.localeCompare(b.name) : b.score - a.score;
  });
  
  const handleItemPress = (id) => {
    setSelectedId(id);
  };
  
  const renderItem = ({ item }) => (
    <TouchableOpacity onPress={() => handleItemPress(item.id)}>
      <Text style={{ fontWeight: selectedId === item.id ? 'bold' : 'normal' }}>
        {item.name}
      </Text>
    </TouchableOpacity>
  );
  
  return (
    <FlatList
      data={filteredData}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}
```

### App Integrity Implementation Example
Complete implementation with error handling and fallbacks.

```typescript
import * as AppIntegrity from 'expo-app-integrity';
import * as SecureStore from 'expo-secure-store';

class SecurityManager {
  private integrityCache = new Map<string, { result: any; timestamp: number }>();
  private cacheTimeout = 5 * 60 * 1000; // 5 minutes
  
  async verifyAppIntegrity(challenge?: string): Promise<{
    isSecure: boolean;
    canProceed: boolean;
    reason?: string;
  }> {
    try {
      // Check if integrity checking is available
      const available = await AppIntegrity.isAvailableAsync();
      if (!available) {
        return {
          isSecure: false,
          canProceed: true, // Allow on unsupported devices
          reason: 'Integrity checking not supported on this device'
        };
      }
      
      // Check cache
      const cacheKey = challenge || 'default';
      const cached = this.integrityCache.get(cacheKey);
      if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
        return this.processResult(cached.result);
      }
      
      // Perform integrity check
      const result = await AppIntegrity.checkIntegrityAsync({
        challenge,
        timeoutMs: 10000,
        includeDeviceDetails: true
      });
      
      // Cache result
      this.integrityCache.set(cacheKey, {
        result,
        timestamp: Date.now()
      });
      
      return this.processResult(result);
      
    } catch (error) {
      console.warn('App integrity check failed:', error);
      
      // Fallback strategy based on error type
      if (error.code === 'NETWORK_ERROR') {
        return {
          isSecure: false,
          canProceed: true,
          reason: 'Network error during integrity check'
        };
      }
      
      return {
        isSecure: false,
        canProceed: false,
        reason: 'Integrity verification failed'
      };
    }
  }
  
  private processResult(result: any) {
    const isSecure = result.isGenuine && result.deviceTrusted && result.appStoreInstalled;
    
    if (!result.isGenuine) {
      return {
        isSecure: false,
        canProceed: false,
        reason: 'App integrity compromised'
      };
    }
    
    if (!result.deviceTrusted) {
      return {
        isSecure: false,
        canProceed: true, // May allow with limitations
        reason: 'Device security status uncertain'
      };
    }
    
    return {
      isSecure: true,
      canProceed: true
    };
  }
  
  async secureAPICall(endpoint: string, data: any) {
    const challenge = generateChallenge();
    const integrity = await this.verifyAppIntegrity(challenge);
    
    if (!integrity.canProceed) {
      throw new Error(`Security check failed: ${integrity.reason}`);
    }
    
    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
    };
    
    if (integrity.isSecure) {
      // Add integrity token for secure requests
      const token = await this.getIntegrityToken(challenge);
      headers['X-Integrity-Token'] = token;
    }
    
    return fetch(endpoint, {
      method: 'POST',
      headers,
      body: JSON.stringify(data)
    });
  }
  
  private async getIntegrityToken(challenge: string): Promise<string> {
    // Get fresh integrity result with attestation token
    const result = await AppIntegrity.checkIntegrityAsync({ challenge });
    return result.details?.attestationToken || '';
  }
}

// Usage example
const securityManager = new SecurityManager();

async function performSecureAction() {
  try {
    const integrity = await securityManager.verifyAppIntegrity();
    
    if (integrity.canProceed) {
      // Proceed with sensitive operation
      const response = await securityManager.secureAPICall('/api/secure-data', {
        action: 'transfer',
        amount: 1000
      });
      
      console.log('Secure operation completed');
    } else {
      // Handle security failure
      showSecurityError(integrity.reason);
    }
  } catch (error) {
    console.error('Security check failed:', error);
  }
}

function generateChallenge(): string {
  return Math.random().toString(36).substring(2) + Date.now().toString(36);
}
```

### Liquid Glass UI Implementation
Complete iOS 26 Liquid Glass component examples.

```typescript
import React from 'react';
import { Platform } from 'react-native';
import { Host, VStack, HStack, Text, Button } from "@expo/ui/swift-ui";
import { 
  glassEffect, 
  padding, 
  cornerRadius, 
  shadow,
  background 
} from '@expo/ui/build/swift-ui/modifiers';

// Check if Liquid Glass is available
const isLiquidGlassAvailable = Platform.OS === 'ios' && 
  parseInt(Platform.Version) >= 26;

// Glass card component
function GlassCard({ children, variant = 'regular', prominent = false }) {
  if (!isLiquidGlassAvailable) {
    // Fallback for older iOS versions
    return (
      <View style={{
        backgroundColor: 'rgba(255, 255, 255, 0.1)',
        backdropFilter: 'blur(10px)',
        borderRadius: 12,
        padding: 16,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.1,
        shadowRadius: 8,
      }}>
        {children}
      </View>
    );
  }

  const glassModifiers = [
    padding({ all: 16 }),
    cornerRadius(12),
    glassEffect({
      glass: { 
        variant,
        tint: prominent ? { color: 'blue', intensity: 0.2 } : undefined
      },
      background: {
        blur: 'systemMaterial',
        opacity: prominent ? 0.9 : 0.8
      }
    }),
    shadow({
      color: 'black',
      radius: 8,
      x: 0,
      y: 4
    })
  ];

  return (
    <Host matchContents>
      <VStack modifiers={glassModifiers}>
        {children}
      </VStack>
    </Host>
  );
}

// Glass button variants
function GlassButtons() {
  if (!isLiquidGlassAvailable) {
    // Fallback buttons for older versions
    return (
      <View>
        <TouchableOpacity style={fallbackButtonStyle}>
          <Text>Primary Action</Text>
        </TouchableOpacity>
        <TouchableOpacity style={fallbackSecondaryStyle}>
          <Text>Secondary Action</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <Host matchContents>
      <VStack spacing={12}>
        <Button
          variant="glassProminent"
          color="blue"
          systemImage="plus.circle.fill"
        >
          Create New
        </Button>
        
        <Button
          variant="glass"
          color="green"
          systemImage="checkmark.circle"
        >
          Confirm
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

// Complex glass layout
function GlassInterface() {
  return (
    <Host matchContents>
      <VStack spacing={16} modifiers={[padding({ all: 20 })]}>
        {/* Header glass card */}
        <GlassCard variant="thin" prominent={true}>
          <HStack alignment="center" spacing={12}>
            <Text style={{ fontSize: 24, fontWeight: 'bold' }}>
              Dashboard
            </Text>
            <Button variant="glass" systemImage="bell">
              Notifications
            </Button>
          </HStack>
        </GlassCard>
        
        {/* Content cards */}
        <HStack spacing={16}>
          <GlassCard variant="regular">
            <VStack spacing={8}>
              <Text style={{ fontSize: 18, fontWeight: '600' }}>
                Statistics
              </Text>
              <Text style={{ fontSize: 32, fontWeight: 'bold', color: 'blue' }}>
                1,234
              </Text>
              <Text style={{ fontSize: 14, opacity: 0.7 }}>
                Total Users
              </Text>
            </VStack>
          </GlassCard>
          
          <GlassCard variant="thick">
            <VStack spacing={12}>
              <Text style={{ fontSize: 16, fontWeight: '600' }}>
                Quick Actions
              </Text>
              <GlassButtons />
            </VStack>
          </GlassCard>
        </HStack>
      </VStack>
    </Host>
  );
}

// Fallback styles for older iOS versions
const fallbackButtonStyle = {
  backgroundColor: 'rgba(0, 122, 255, 0.8)',
  padding: 12,
  borderRadius: 8,
  marginBottom: 8,
};

const fallbackSecondaryStyle = {
  backgroundColor: 'rgba(255, 255, 255, 0.2)',
  padding: 12,
  borderRadius: 8,
  marginBottom: 8,
};

export { GlassCard, GlassButtons, GlassInterface };
```

### Edge-to-Edge Android Implementation
Complete Android 16 edge-to-edge setup with safe areas.

```typescript
import React from 'react';
import { Platform, StatusBar, View, Text, ScrollView } from 'react-native';
import { useSafeAreaInsets, SafeAreaProvider } from 'react-native-safe-area-context';
import { StatusBar as ExpoStatusBar } from 'expo-status-bar';

// Edge-to-edge wrapper component
function EdgeToEdgeWrapper({ children, statusBarStyle = 'auto' }) {
  const insets = useSafeAreaInsets();
  
  // Android 16+ automatically enables edge-to-edge
  const isAndroid16Plus = Platform.OS === 'android' && Platform.Version >= 36;
  
  return (
    <View style={{ flex: 1 }}>
      <ExpoStatusBar style={statusBarStyle} />
      
      <View style={{
        flex: 1,
        // Apply top inset for status bar
        paddingTop: insets.top,
        // Apply bottom inset for gesture navigation
        paddingBottom: insets.bottom,
        // Side insets for landscape or notched devices
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }}>
        {children}
      </View>
    </View>
  );
}

// Content that extends to edges
function EdgeToEdgeScreen() {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1 }}>
      {/* Full-width header that extends to edges */}
      <View style={{
        backgroundColor: '#007AFF',
        paddingTop: insets.top + 16,
        paddingBottom: 16,
        paddingHorizontal: Math.max(insets.left, 16),
      }}>
        <Text style={{
          color: 'white',
          fontSize: 24,
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
          padding: 16,
          paddingLeft: Math.max(insets.left, 16),
          paddingRight: Math.max(insets.right, 16),
          paddingBottom: Math.max(insets.bottom, 16),
        }}
      >
        <Text style={{ fontSize: 16, lineHeight: 24, marginBottom: 16 }}>
          This content respects safe areas while allowing the header to extend to screen edges.
        </Text>
        
        {/* Generate content to test scrolling */}
        {Array.from({ length: 20 }, (_, i) => (
          <Text key={i} style={{ marginBottom: 12, fontSize: 14 }}>
            Content item {i + 1} - This text has proper margins and respects the safe areas.
          </Text>
        ))}
      </ScrollView>
      
      {/* Bottom bar that extends to edges */}
      <View style={{
        backgroundColor: '#F2F2F7',
        paddingTop: 12,
        paddingBottom: Math.max(insets.bottom, 12),
        paddingHorizontal: Math.max(insets.left, 16),
        borderTopWidth: 1,
        borderTopColor: '#E5E5EA',
      }}>
        <Text style={{
          textAlign: 'center',
          fontSize: 16,
          fontWeight: '500',
        }}>
          Bottom Navigation
        </Text>
      </View>
    </View>
  );
}

// App setup with proper providers
function App() {
  return (
    <SafeAreaProvider>
      <EdgeToEdgeWrapper statusBarStyle="dark">
        <EdgeToEdgeScreen />
      </EdgeToEdgeWrapper>
    </SafeAreaProvider>
  );
}

// Navigation bar contrast configuration
// Add to app.json:
/*
{
  "expo": {
    "android": {
      "androidNavigationBar": {
        "enforceContrast": false
      }
    }
  }
}
*/

export default App;
```

## Performance Optimization Examples

### Build Time Optimization
Tips for maximizing the performance improvements in SDK 54.

```bash
#!/bin/bash
# build-optimization.sh
# Script to optimize build performance

echo "🚀 Optimizing Expo SDK 54 build performance..."

# 1. Clean all caches
echo "Cleaning caches..."
npx expo r -c
rm -rf node_modules/.cache
rm -rf .expo

# 2. Update dependencies for optimal performance
echo "Updating dependencies..."
npx expo install --fix

# 3. Verify React Compiler is enabled
echo "Checking React Compiler status..."
if grep -q '"reactCompiler": true' babel.config.js; then
    echo "✅ React Compiler enabled"
else
    echo "⚠️ React Compiler not found in babel config"
fi

# 4. Check for New Architecture
echo "Checking New Architecture status..."
if npx expo config --type introspect | grep -q '"newArchEnabled": true'; then
    echo "✅ New Architecture enabled"
else
    echo "⚠️ New Architecture not enabled - consider enabling for better performance"
fi

# 5. Prebuild with optimization
echo "Prebuilding with optimizations..."
npx expo prebuild --clean

# 6. Measure build times
echo "📊 Measuring iOS build time..."
time npm run ios &

echo "📊 Measuring Android build time..."
time npm run android &

wait

echo "✅ Build optimization complete!"
echo "Expected improvements:"
echo "  - iOS builds: ~90% faster with precompiled frameworks"
echo "  - Metro bundling: Faster with experimental import support"
echo "  - Development: Better error reporting and debugging"
```

### Memory and Performance Monitoring
```typescript
// performance-monitor.ts
import { AppState } from 'react-native';

class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map();
  private startTimes: Map<string, number> = new Map();
  
  startTiming(label: string) {
    this.startTimes.set(label, performance.now());
  }
  
  endTiming(label: string) {
    const startTime = this.startTimes.get(label);
    if (!startTime) return;
    
    const duration = performance.now() - startTime;
    const existing = this.metrics.get(label) || [];
    existing.push(duration);
    this.metrics.set(label, existing);
    
    this.startTimes.delete(label);
    
    // Log if duration is concerning
    if (duration > 100) {
      console.warn(`⚠️ Slow operation: ${label} took ${duration.toFixed(2)}ms`);
    }
  }
  
  getMetrics(label: string) {
    const timings = this.metrics.get(label) || [];
    if (timings.length === 0) return null;
    
    const avg = timings.reduce((a, b) => a + b, 0) / timings.length;
    const min = Math.min(...timings);
    const max = Math.max(...timings);
    
    return { avg, min, max, count: timings.length };
  }
  
  measureComponent<T extends React.ComponentType<any>>(
    Component: T,
    displayName?: string
  ): T {
    const name = displayName || Component.displayName || Component.name || 'Anonymous';
    
    return React.forwardRef((props, ref) => {
      const renderStart = React.useRef<number>(0);
      
      React.useLayoutEffect(() => {
        renderStart.current = performance.now();
      });
      
      React.useLayoutEffect(() => {
        const renderTime = performance.now() - renderStart.current;
        if (renderTime > 16) { // More than one frame
          console.log(`🐌 Slow render: ${name} took ${renderTime.toFixed(2)}ms`);
        }
      });
      
      return React.createElement(Component, { ...props, ref });
    }) as T;
  }
  
  logMemoryUsage() {
    if (__DEV__ && (performance as any).memory) {
      const memory = (performance as any).memory;
      console.log('📊 Memory Usage:', {
        used: Math.round(memory.usedJSHeapSize / 1024 / 1024) + 'MB',
        total: Math.round(memory.totalJSHeapSize / 1024 / 1024) + 'MB',
        limit: Math.round(memory.jsHeapSizeLimit / 1024 / 1024) + 'MB'
      });
    }
  }
  
  startAppStateMonitoring() {
    AppState.addEventListener('change', (nextAppState) => {
      if (nextAppState === 'background') {
        this.logPerformanceSummary();
      }
    });
  }
  
  private logPerformanceSummary() {
    console.log('📈 Performance Summary:');
    for (const [label, timings] of this.metrics) {
      const metrics = this.getMetrics(label);
      if (metrics) {
        console.log(`  ${label}: avg ${metrics.avg.toFixed(2)}ms (${metrics.count} samples)`);
      }
    }
    this.logMemoryUsage();
  }
}

// Usage examples
const monitor = new PerformanceMonitor();

// Measure async operations
async function fetchData() {
  monitor.startTiming('data-fetch');
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } finally {
    monitor.endTiming('data-fetch');
  }
}

// Measure component render performance
const OptimizedComponent = monitor.measureComponent(
  function MyComponent({ data }) {
    // Component that might be slow
    const processedData = data.map(item => ({
      ...item,
      computed: expensiveCalculation(item)
    }));
    
    return (
      <View>
        {processedData.map(item => (
          <Text key={item.id}>{item.name}</Text>
        ))}
      </View>
    );
  },
  'MyComponent'
);

// Start monitoring
monitor.startAppStateMonitoring();
```

---

**Next Steps:**
- Copy examples to your project and modify for your needs
- Test performance improvements with measurement tools
- Experiment with new APIs in development environment
- Join [Expo Discord](https://chat.expo.dev/) for community examples
