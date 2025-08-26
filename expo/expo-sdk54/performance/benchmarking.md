# Performance Benchmarking Guide
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/performance/benchmarking.md -->
<!-- Purpose: Comprehensive guide to measuring and optimizing SDK 54 performance -->
<!-- Dependencies: Expo SDK 54, Performance APIs, React DevTools -->

## Overview

Expo SDK 54 introduces significant performance improvements, including 90% faster iOS builds, React Compiler optimizations, and New Architecture benefits. This guide helps you measure and maximize these improvements.

## Build Performance Benchmarking

### iOS Build Time Measurement

#### Baseline Measurement Script
```bash
#!/bin/bash
# ios-build-benchmark.sh

echo "📊 iOS Build Performance Benchmark - SDK 54"
echo "==========================================="

# Clean environment for accurate measurement
echo "🧹 Cleaning build environment..."
npx expo r -c
rm -rf ios
rm -rf node_modules/.cache

# Install dependencies
echo "📦 Installing dependencies..."
npm install

# Measure cold prebuild
echo "⏱️  Measuring cold prebuild time..."
time npx expo prebuild --platform ios --clean

# Measure first build (with precompiled frameworks)
echo "⏱️  Measuring first iOS build time..."
start_time=$(date +%s)
npm run ios --silent > build.log 2>&1
end_time=$(date +%s)
build_time=$((end_time - start_time))

echo "✅ First build completed in: ${build_time} seconds"

# Measure incremental build
echo "⏱️  Measuring incremental build time..."
touch App.tsx # Touch file to trigger rebuild
start_time=$(date +%s)
npm run ios --silent > incremental-build.log 2>&1
end_time=$(date +%s)
incremental_time=$((end_time - start_time))

echo "✅ Incremental build completed in: ${incremental_time} seconds"

# Results summary
echo ""
echo "📈 Build Performance Results:"
echo "   First build: ${build_time}s"
echo "   Incremental build: ${incremental_time}s"
echo "   Expected improvement over SDK 53: ~90% for clean builds"

# Save results
cat > build-results.json << EOF
{
  "timestamp": "$(date -Iseconds)",
  "sdk_version": "54",
  "platform": "ios",
  "first_build_time": ${build_time},
  "incremental_build_time": ${incremental_time},
  "machine_info": "$(uname -a)"
}
EOF

echo "📁 Results saved to build-results.json"
```

#### Comparative Analysis
```bash
# Compare SDK 53 vs SDK 54 performance
# Run this script on both SDK versions

#!/bin/bash
# performance-comparison.sh

function benchmark_sdk() {
  local sdk_version=$1
  echo "Testing SDK $sdk_version..."
  
  # Switch SDK version
  npx expo install expo@$sdk_version
  
  # Clean build
  rm -rf ios node_modules/.cache
  npm install
  npx expo prebuild --platform ios --clean
  
  # Measure build time
  start_time=$(date +%s)
  npm run ios --silent > "build-${sdk_version}.log" 2>&1
  end_time=$(date +%s)
  
  build_time=$((end_time - start_time))
  echo "SDK $sdk_version build time: ${build_time}s"
  
  return $build_time
}

# Test both versions
benchmark_sdk "53"
sdk53_time=$?

benchmark_sdk "54"
sdk54_time=$?

# Calculate improvement
improvement=$(( (sdk53_time - sdk54_time) * 100 / sdk53_time ))
echo "Performance improvement: ${improvement}%"
```

### Android Build Measurement
```bash
#!/bin/bash
# android-build-benchmark.sh

echo "📊 Android Build Performance Benchmark"
echo "======================================"

# Clean environment
npx expo r -c
rm -rf android

# Measure Android build
echo "⏱️  Measuring Android build time..."
npx expo prebuild --platform android --clean

start_time=$(date +%s)
npm run android --silent > android-build.log 2>&1
end_time=$(date +%s)
android_build_time=$((end_time - start_time))

echo "✅ Android build completed in: ${android_build_time} seconds"

# Measure Gradle build specifically
echo "⏱️  Measuring Gradle build time..."
cd android
start_time=$(date +%s)
./gradlew assembleDebug > gradle-build.log 2>&1
end_time=$(date +%s)
gradle_time=$((end_time - start_time))

echo "✅ Gradle build completed in: ${gradle_time} seconds"
cd ..

echo "📈 Android Build Results:"
echo "   Total build: ${android_build_time}s"
echo "   Gradle only: ${gradle_time}s"
```

## Runtime Performance Monitoring

### React Compiler Impact Measurement
```typescript
// performance-monitor.tsx
import React, { Profiler, useState, useCallback } from 'react';
import { View, Text, FlatList, TouchableOpacity } from 'react-native';

// Test component for React Compiler benchmarking
function BenchmarkComponent({ data, filter, onItemPress }) {
  // React Compiler should optimize these operations automatically
  const filteredData = data.filter(item => 
    filter ? item.category === filter : true
  );
  
  const sortedData = filteredData.sort((a, b) => 
    a.priority - b.priority
  );
  
  const processedData = sortedData.map(item => ({
    ...item,
    displayName: `${item.name} (${item.category})`,
    isHighPriority: item.priority > 8
  }));

  const renderItem = useCallback(({ item }) => (
    <TouchableOpacity onPress={() => onItemPress(item.id)}>
      <View style={{ 
        padding: 16, 
        backgroundColor: item.isHighPriority ? '#ffeb3b' : 'white' 
      }}>
        <Text>{item.displayName}</Text>
      </View>
    </TouchableOpacity>
  ), [onItemPress]);

  return (
    <FlatList
      data={processedData}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}

// Performance measurement wrapper
function PerformanceBenchmark() {
  const [renderTimes, setRenderTimes] = useState<number[]>([]);
  const [selectedFilter, setSelectedFilter] = useState<string | null>(null);
  
  // Generate test data
  const testData = Array.from({ length: 1000 }, (_, i) => ({
    id: i.toString(),
    name: `Item ${i}`,
    category: ['A', 'B', 'C'][i % 3],
    priority: Math.floor(Math.random() * 10) + 1,
  }));

  const onRender = useCallback((id: string, phase: string, actualDuration: number) => {
    console.log(`Render: ${id} (${phase}) - ${actualDuration.toFixed(2)}ms`);
    setRenderTimes(prev => [...prev.slice(-9), actualDuration]);
  }, []);

  const handleItemPress = useCallback((itemId: string) => {
    console.log('Item pressed:', itemId);
  }, []);

  const averageRenderTime = renderTimes.length > 0 
    ? renderTimes.reduce((a, b) => a + b, 0) / renderTimes.length 
    : 0;

  return (
    <View style={{ flex: 1 }}>
      <View style={{ padding: 16, backgroundColor: '#f5f5f5' }}>
        <Text>Average Render Time: {averageRenderTime.toFixed(2)}ms</Text>
        <Text>React Compiler: {__DEV__ ? 'Enabled' : 'Production'}</Text>
        
        <View style={{ flexDirection: 'row', marginTop: 8 }}>
          {['A', 'B', 'C', null].map(filter => (
            <TouchableOpacity
              key={filter || 'all'}
              onPress={() => setSelectedFilter(filter)}
              style={{
                padding: 8,
                margin: 4,
                backgroundColor: selectedFilter === filter ? '#2196F3' : '#e0e0e0'
              }}
            >
              <Text style={{ color: selectedFilter === filter ? 'white' : 'black' }}>
                {filter || 'All'}
              </Text>
            </TouchableOpacity>
          ))}
        </View>
      </View>
      
      <Profiler id="BenchmarkList" onRender={onRender}>
        <BenchmarkComponent 
          data={testData} 
          filter={selectedFilter}
          onItemPress={handleItemPress}
        />
      </Profiler>
    </View>
  );
}
```

### Memory Usage Monitoring
```typescript
// memory-monitor.tsx
import { useEffect, useState, useRef } from 'react';

interface MemoryMetrics {
  usedJSHeapSize: number;
  totalJSHeapSize: number;
  jsHeapSizeLimit: number;
  timestamp: number;
}

function useMemoryMonitor(interval: number = 1000) {
  const [memoryMetrics, setMemoryMetrics] = useState<MemoryMetrics[]>([]);
  const intervalRef = useRef<NodeJS.Timeout>();

  useEffect(() => {
    if (!__DEV__ || !(performance as any).memory) {
      return;
    }

    const collectMemoryMetrics = () => {
      const memory = (performance as any).memory;
      const metrics: MemoryMetrics = {
        usedJSHeapSize: memory.usedJSHeapSize,
        totalJSHeapSize: memory.totalJSHeapSize,
        jsHeapSizeLimit: memory.jsHeapSizeLimit,
        timestamp: Date.now(),
      };

      setMemoryMetrics(prev => [...prev.slice(-59), metrics]); // Keep last 60 measurements
    };

    collectMemoryMetrics(); // Initial measurement
    intervalRef.current = setInterval(collectMemoryMetrics, interval);

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [interval]);

  const currentMemory = memoryMetrics[memoryMetrics.length - 1];
  const memoryTrend = memoryMetrics.length > 1 
    ? memoryMetrics[memoryMetrics.length - 1].usedJSHeapSize - memoryMetrics[0].usedJSHeapSize
    : 0;

  return {
    memoryMetrics,
    currentMemory,
    memoryTrend,
    isMemoryAvailable: __DEV__ && !!(performance as any).memory,
  };
}

// Memory monitoring component
function MemoryMonitor() {
  const { memoryMetrics, currentMemory, memoryTrend, isMemoryAvailable } = useMemoryMonitor();

  if (!isMemoryAvailable) {
    return (
      <View style={{ padding: 16 }}>
        <Text>Memory monitoring only available in development</Text>
      </View>
    );
  }

  const formatBytes = (bytes: number) => {
    return Math.round(bytes / 1024 / 1024 * 100) / 100 + ' MB';
  };

  return (
    <View style={{ padding: 16, backgroundColor: '#f5f5f5' }}>
      <Text style={{ fontSize: 16, fontWeight: 'bold' }}>Memory Usage</Text>
      
      {currentMemory && (
        <View style={{ marginTop: 8 }}>
          <Text>Used: {formatBytes(currentMemory.usedJSHeapSize)}</Text>
          <Text>Total: {formatBytes(currentMemory.totalJSHeapSize)}</Text>
          <Text>Limit: {formatBytes(currentMemory.jsHeapSizeLimit)}</Text>
          <Text style={{ 
            color: memoryTrend > 0 ? 'red' : 'green',
            marginTop: 4 
          }}>
            Trend: {memoryTrend > 0 ? '+' : ''}{formatBytes(memoryTrend)}
          </Text>
        </View>
      )}

      <Text style={{ fontSize: 12, marginTop: 8, opacity: 0.7 }}>
        Samples: {memoryMetrics.length}
      </Text>
    </View>
  );
}
```

## New Architecture Performance Testing

### Architecture Detection and Benchmarking
```typescript
// architecture-benchmark.tsx
import { Platform } from 'react-native';

interface ArchitectureInfo {
  isTurboModuleEnabled: boolean;
  isFabricEnabled: boolean;
  isNewArchitecture: boolean;
  platform: string;
}

function detectArchitecture(): ArchitectureInfo {
  const isTurboModuleEnabled = global.__turboModuleProxy != null;
  const isFabricEnabled = global.nativeFabricUIManager != null;
  
  return {
    isTurboModuleEnabled,
    isFabricEnabled,
    isNewArchitecture: isTurboModuleEnabled && isFabricEnabled,
    platform: Platform.OS,
  };
}

// Benchmark native module calls
async function benchmarkNativeModuleCalls(iterations: number = 100) {
  const startTime = performance.now();
  
  // Test multiple native module calls
  const promises = Array.from({ length: iterations }, async () => {
    // Example native module calls
    const promises = [
      Platform.constants,
      // Add more native module calls as needed
    ];
    
    return Promise.all(promises);
  });
  
  await Promise.all(promises);
  
  const endTime = performance.now();
  const totalTime = endTime - startTime;
  const averageTime = totalTime / iterations;
  
  return {
    totalTime,
    averageTime,
    iterations,
    callsPerSecond: 1000 / averageTime,
  };
}

// Architecture performance comparison component
function ArchitectureBenchmark() {
  const [architectureInfo, setArchitectureInfo] = useState<ArchitectureInfo | null>(null);
  const [benchmarkResults, setBenchmarkResults] = useState<any>(null);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    setArchitectureInfo(detectArchitecture());
  }, []);

  const runBenchmark = async () => {
    setIsRunning(true);
    try {
      const results = await benchmarkNativeModuleCalls(100);
      setBenchmarkResults(results);
    } catch (error) {
      console.error('Benchmark failed:', error);
    } finally {
      setIsRunning(false);
    }
  };

  return (
    <View style={{ padding: 16 }}>
      <Text style={{ fontSize: 18, fontWeight: 'bold', marginBottom: 16 }}>
        Architecture Performance
      </Text>
      
      {architectureInfo && (
        <View style={{ marginBottom: 16 }}>
          <Text>Platform: {architectureInfo.platform}</Text>
          <Text>TurboModules: {architectureInfo.isTurboModuleEnabled ? '✅' : '❌'}</Text>
          <Text>Fabric: {architectureInfo.isFabricEnabled ? '✅' : '❌'}</Text>
          <Text style={{ 
            color: architectureInfo.isNewArchitecture ? 'green' : 'orange',
            fontWeight: 'bold' 
          }}>
            New Architecture: {architectureInfo.isNewArchitecture ? 'Active' : 'Legacy'}
          </Text>
        </View>
      )}

      <TouchableOpacity
        onPress={runBenchmark}
        disabled={isRunning}
        style={{
          backgroundColor: isRunning ? '#ccc' : '#2196F3',
          padding: 12,
          borderRadius: 8,
          marginBottom: 16,
        }}
      >
        <Text style={{ color: 'white', textAlign: 'center' }}>
          {isRunning ? 'Running Benchmark...' : 'Run Native Module Benchmark'}
        </Text>
      </TouchableOpacity>

      {benchmarkResults && (
        <View style={{ backgroundColor: '#f5f5f5', padding: 12, borderRadius: 8 }}>
          <Text style={{ fontWeight: 'bold' }}>Benchmark Results:</Text>
          <Text>Total Time: {benchmarkResults.totalTime.toFixed(2)}ms</Text>
          <Text>Average per Call: {benchmarkResults.averageTime.toFixed(3)}ms</Text>
          <Text>Calls per Second: {benchmarkResults.callsPerSecond.toFixed(0)}</Text>
          <Text>Iterations: {benchmarkResults.iterations}</Text>
        </View>
      )}
    </View>
  );
}
```

## Bundle Size Analysis

### Bundle Size Monitoring
```bash
#!/bin/bash
# bundle-analysis.sh

echo "📦 Bundle Size Analysis - SDK 54"
echo "================================"

# Create production bundle
echo "🏗️  Building production bundle..."
npx expo export --platform all

# Analyze bundle sizes
echo "📊 Analyzing bundle sizes..."

if [ -d "dist" ]; then
  echo "Web bundle sizes:"
  find dist -name "*.js" -exec du -h {} \; | sort -hr
  
  echo -e "\nTotal web bundle size:"
  du -sh dist/
fi

# Analyze native bundle sizes (requires EAS Build)
echo -e "\n🔍 To analyze native bundle sizes:"
echo "1. Run: eas build --platform all --profile production"
echo "2. Download the builds"
echo "3. For iOS: Extract .ipa and analyze app bundle"
echo "4. For Android: Use 'bundletool' to analyze .aab file"

# Create bundle analysis report
cat > bundle-analysis.json << EOF
{
  "timestamp": "$(date -Iseconds)",
  "sdk_version": "54",
  "web_bundle_size": "$(du -s dist/ 2>/dev/null | cut -f1 || echo 'N/A')",
  "analysis_date": "$(date)"
}
EOF

echo -e "\n📁 Analysis saved to bundle-analysis.json"
```

### Metro Bundle Analysis
```javascript
// metro.config.js - Add bundle analysis
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Add bundle analysis in development
if (process.env.NODE_ENV !== 'production') {
  config.serializer = {
    ...config.serializer,
    createModuleIdFactory: () => (path) => {
      // Log module sizes for analysis
      console.log(`Module: ${path}`);
      return path;
    },
  };
}

// Production optimizations
if (process.env.NODE_ENV === 'production') {
  config.transformer = {
    ...config.transformer,
    minifierConfig: {
      // Optimize for smaller bundles
      mangle: true,
      compress: {
        drop_console: true, // Remove console.log in production
      },
    },
  };
}

module.exports = config;
```

## Performance Testing Automation

### Automated Performance Test Suite
```bash
#!/bin/bash
# automated-performance-tests.sh

echo "🤖 Automated Performance Test Suite"
echo "==================================="

# Test configuration
ITERATIONS=5
PLATFORMS=("ios" "android")

# Create results directory
mkdir -p performance-results
cd performance-results

# Function to run performance test
run_performance_test() {
  local platform=$1
  local iteration=$2
  
  echo "Running test $iteration for $platform..."
  
  # Clean environment
  cd ..
  npx expo r -c
  rm -rf $platform
  
  # Build and measure
  start_time=$(date +%s)
  npx expo prebuild --platform $platform --clean > /dev/null 2>&1
  npm run $platform > /dev/null 2>&1
  end_time=$(date +%s)
  
  build_time=$((end_time - start_time))
  
  # Save result
  echo "{\"platform\":\"$platform\",\"iteration\":$iteration,\"build_time\":$build_time,\"timestamp\":\"$(date -Iseconds)\"}" >> performance-results/${platform}-results.jsonl
  
  cd performance-results
}

# Run tests for all platforms
for platform in "${PLATFORMS[@]}"; do
  echo "Testing $platform performance..."
  
  for i in $(seq 1 $ITERATIONS); do
    run_performance_test $platform $i
  done
done

# Generate summary report
echo "📈 Generating performance summary..."
cat > performance-summary.md << 'EOF'
# Performance Test Results

## Test Configuration
- SDK Version: 54
- Iterations per platform: 5
- Platforms tested: iOS, Android

## Results Summary

### iOS Performance
EOF

# Calculate iOS averages
if [ -f "ios-results.jsonl" ]; then
  ios_avg=$(cat ios-results.jsonl | jq '.build_time' | awk '{sum+=$1; count++} END {print sum/count}')
  echo "- Average build time: ${ios_avg}s" >> performance-summary.md
fi

# Calculate Android averages
if [ -f "android-results.jsonl" ]; then
  android_avg=$(cat android-results.jsonl | jq '.build_time' | awk '{sum+=$1; count++} END {print sum/count}')
  echo -e "\n### Android Performance" >> performance-summary.md
  echo "- Average build time: ${android_avg}s" >> performance-summary.md
fi

echo "✅ Performance testing complete!"
echo "📁 Results saved in performance-results/"
```

## Continuous Performance Monitoring

### CI/CD Performance Integration
```yaml
# .github/workflows/performance-monitoring.yml
name: Performance Monitoring

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  performance-test:
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '20'
        cache: 'npm'
        
    - name: Setup Expo CLI
      run: npm install -g @expo/cli@latest
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run iOS build performance test
      run: |
        start_time=$(date +%s)
        npx expo prebuild --platform ios --clean
        npm run ios
        end_time=$(date +%s)
        build_time=$((end_time - start_time))
        echo "iOS build time: ${build_time}s"
        echo "ios_build_time=${build_time}" >> $GITHUB_OUTPUT
      id: ios_performance
      
    - name: Comment performance results
      uses: actions/github-script@v6
      with:
        script: |
          const buildTime = '${{ steps.ios_performance.outputs.ios_build_time }}';
          const comment = `
          ## 📊 Performance Results
          
          - iOS Build Time: ${buildTime}s
          - SDK Version: 54
          - Expected improvement: ~90% vs SDK 53
          
          *Automated performance monitoring*
          `;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: comment
          });
```

## Performance Optimization Checklist

### Build Performance
- [ ] React Native 0.81+ with precompiled frameworks
- [ ] Xcode 16.1+ for iOS development
- [ ] Clean build environment before major tests
- [ ] Monitor Metro bundler performance
- [ ] Use production builds for accurate bundle size testing

### Runtime Performance
- [ ] React Compiler enabled and monitored
- [ ] New Architecture enabled where possible
- [ ] Memory usage patterns analyzed
- [ ] Component render times optimized
- [ ] Native module call efficiency measured

### Optimization Strategies
- [ ] Enable React Compiler for automatic optimizations
- [ ] Use New Architecture for better performance
- [ ] Implement proper memoization where React Compiler doesn't help
- [ ] Optimize bundle size with tree shaking
- [ ] Monitor and fix memory leaks
- [ ] Profile and optimize expensive operations

---

**Next Steps:**
- Run baseline performance tests on your current setup
- Implement performance monitoring in your development workflow
- Compare SDK 54 improvements against previous versions
- Set up automated performance testing in CI/CD
- Monitor production performance metrics
