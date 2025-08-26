# React Compiler Integration
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/development-tools/react-compiler.md -->
<!-- Purpose: Detailed guide for React Compiler in SDK 54 -->
<!-- Dependencies: React 19.1.0, babel-preset-expo -->

## Overview

React Compiler is enabled by default in new Expo SDK 54 projects, providing automatic optimization of React components without manual memoization.

## What React Compiler Does

### Automatic Optimizations
React Compiler automatically handles:

- **Component Memoization:** Eliminates unnecessary re-renders
- **Hook Dependencies:** Optimizes `useMemo`, `useCallback` usage
- **Prop Drilling:** Reduces prop change impacts
- **Expensive Calculations:** Caches computation results

### Before vs After

```typescript
// ❌ Before: Manual optimization required
function ExpensiveComponent({ items, filter, sort }) {
  // Manual memoization
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);

  const sortedItems = useMemo(() => {
    return filteredItems.sort((a, b) => {
      return sort === 'name' ? a.name.localeCompare(b.name) : a.id - b.id;
    });
  }, [filteredItems, sort]);

  // Manual callback memoization
  const handleItemPress = useCallback((item) => {
    console.log('Selected:', item.name);
  }, []);

  const renderItem = useCallback(({ item }) => (
    <TouchableOpacity onPress={() => handleItemPress(item)}>
      <Text>{item.name}</Text>
    </TouchableOpacity>
  ), [handleItemPress]);

  return (
    <FlatList
      data={sortedItems}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}

// ✅ After: React Compiler handles optimization automatically
function OptimizedComponent({ items, filter, sort }) {
  // React Compiler automatically optimizes these operations
  const filteredItems = items.filter(item => item.category === filter);
  
  const sortedItems = filteredItems.sort((a, b) => {
    return sort === 'name' ? a.name.localeCompare(b.name) : a.id - b.id;
  });

  const handleItemPress = (item) => {
    console.log('Selected:', item.name);
  };

  const renderItem = ({ item }) => (
    <TouchableOpacity onPress={() => handleItemPress(item)}>
      <Text>{item.name}</Text>
    </TouchableOpacity>
  );

  return (
    <FlatList
      data={sortedItems}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
}
```

## Configuration

### Default Setup (New Projects)
React Compiler is enabled by default in new SDK 54 projects:

```javascript
// babel.config.js - default configuration
module.exports = function(api) {
  api.cache(true);
  return {
    presets: [
      ['babel-preset-expo', {
        reactCompiler: true // Default: true
      }]
    ],
  };
};
```

### Manual Enablement (Existing Projects)
For existing projects, enable React Compiler:

```javascript
// babel.config.js
module.exports = function(api) {
  api.cache(true);
  return {
    presets: [
      ['babel-preset-expo', {
        reactCompiler: true // Add this
      }]
    ],
  };
};
```

### Advanced Configuration
For custom React Compiler options:

```javascript
// babel.config.js
module.exports = function(api) {
  api.cache(true);
  return {
    presets: [
      ['babel-preset-expo', {
        reactCompiler: {
          compilationMode: 'annotation', // 'all' | 'annotation'
          panicThreshold: 'all_errors',   // 'all_errors' | 'critical_errors' | 'none'
          
          // Advanced options
          sources: (filename) => {
            // Only compile files in src/ directory
            return filename.includes('/src/');
          },
          
          // Development vs production
          ...(process.env.NODE_ENV === 'development' && {
            compilationMode: 'annotation',
          }),
        }
      }]
    ],
  };
};
```

## Monitoring Optimization

### Development Console
Check if React Compiler is running:

```bash
npx expo start
# Look for this message:
> Experimental React Compiler is enabled.
```

### Component Inspector
View optimization status in development:

```bash
# Press J in Expo CLI
# Navigate to Components tab
# See which components are optimized
```

### Performance Profiling
Use React DevTools to see optimization impact:

```typescript
// Enable Profiler in development
import { Profiler } from 'react';

function ProfiledApp() {
  return (
    <Profiler
      id="App"
      onRender={(id, phase, actualDuration) => {
        console.log('Component render:', {
          id,
          phase,
          duration: actualDuration
        });
      }}
    >
      <App />
    </Profiler>
  );
}
```

## Writing Compiler-Friendly Code

### Do's ✅

```typescript
// ✅ Pure functional components
function UserCard({ user, onPress }) {
  return (
    <TouchableOpacity onPress={() => onPress(user.id)}>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </TouchableOpacity>
  );
}

// ✅ Clear data transformations
function useProcessedData(rawData, filters) {
  // Compiler can optimize this chain
  const filtered = rawData.filter(item => filters.includes(item.type));
  const sorted = filtered.sort((a, b) => a.priority - b.priority);
  const grouped = groupBy(sorted, 'category');
  
  return grouped;
}

// ✅ Stable object patterns
function useUserSettings(userId) {
  const [settings, setSettings] = useState({
    theme: 'light',
    notifications: true,
    privacy: 'public'
  });
  
  const updateSetting = (key, value) => {
    setSettings(prev => ({ ...prev, [key]: value }));
  };
  
  return { settings, updateSetting };
}
```

### Don'ts ❌

```typescript
// ❌ Avoid complex side effects in render
function ProblematicComponent({ data }) {
  // Compiler may not optimize side effects well
  useEffect(() => {
    // Complex side effect logic
    data.forEach(item => {
      localStorage.setItem(item.id, JSON.stringify(item));
      analytics.track('item_viewed', item);
      updateExternalCache(item);
    });
  }, [data]);
  
  return <View>...</View>;
}

// ❌ Avoid dynamic property access patterns
function DynamicComponent({ config }) {
  // Dynamic property access is harder to optimize
  const dynamicStyle = {};
  Object.keys(config).forEach(key => {
    dynamicStyle[key] = config[key];
  });
  
  return <View style={dynamicStyle} />;
}

// ❌ Avoid complex conditional rendering
function ConditionalComponent({ type, data }) {
  // Complex conditional logic may not optimize well
  let content;
  
  switch (type) {
    case 'A':
      content = processTypeA(data);
      break;
    case 'B':
      content = processTypeB(data);
      break;
    default:
      content = processDefault(data);
  }
  
  return <View>{content}</View>;
}
```

## Performance Patterns

### Component Splitting
Split large components for better optimization:

```typescript
// ✅ Split into smaller, focused components
function UserProfile({ user }) {
  return (
    <View>
      <UserHeader user={user} />
      <UserDetails user={user} />
      <UserActions user={user} />
    </View>
  );
}

function UserHeader({ user }) {
  return (
    <View>
      <Image source={{ uri: user.avatar }} />
      <Text>{user.name}</Text>
    </View>
  );
}

function UserDetails({ user }) {
  return (
    <View>
      <Text>{user.email}</Text>
      <Text>{user.joinDate}</Text>
    </View>
  );
}
```

### State Colocation
Keep state close to where it's used:

```typescript
// ✅ Colocated state is easier to optimize
function SearchableList({ items }) {
  return (
    <View>
      <SearchInput />
      <ItemList items={items} />
    </View>
  );
}

function SearchInput() {
  const [query, setQuery] = useState('');
  
  return (
    <TextInput
      value={query}
      onChangeText={setQuery}
      placeholder="Search..."
    />
  );
}

// Instead of hoisting search state to parent
```

### Stable References
Use stable object and function references:

```typescript
// ✅ Stable configuration objects
const CHART_CONFIG = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: 'top' }
  }
};

function ChartComponent({ data }) {
  // Stable reference helps optimization
  return <Chart data={data} options={CHART_CONFIG} />;
}
```

## Debugging Optimization Issues

### Component Not Optimizing
If a component isn't being optimized:

```typescript
// Add compilation directive (annotation mode)
'use optimize'; // At top of component file

function MyComponent() {
  // Component implementation
}

// Or disable for debugging
'use no-optimize';

function ProblematicComponent() {
  // This component won't be optimized
}
```

### Performance Regression
If you suspect compiler issues:

```javascript
// Temporarily disable compiler
// babel.config.js
module.exports = {
  presets: [
    ['babel-preset-expo', {
      reactCompiler: false // Disable temporarily
    }]
  ]
};
```

### Measuring Impact
Compare performance with and without compiler:

```typescript
// Performance measurement component
function PerformanceTest({ iterations = 1000 }) {
  const [results, setResults] = useState([]);
  
  const runTest = () => {
    const start = performance.now();
    
    // Simulate heavy computation
    for (let i = 0; i < iterations; i++) {
      const result = heavyCalculation(i);
      processResult(result);
    }
    
    const duration = performance.now() - start;
    setResults(prev => [...prev, duration]);
  };
  
  return (
    <View>
      <Button title="Run Test" onPress={runTest} />
      <Text>Average: {results.reduce((a, b) => a + b, 0) / results.length}ms</Text>
    </View>
  );
}
```

## Migration Strategy

### Gradual Adoption
For existing large projects:

1. **Enable in Development First:**
   ```javascript
   // babel.config.js
   module.exports = {
     presets: [
       ['babel-preset-expo', {
         reactCompiler: process.env.NODE_ENV === 'development'
       }]
     ]
   };
   ```

2. **Use Annotation Mode:**
   ```javascript
   reactCompiler: {
     compilationMode: 'annotation'
   }
   ```

3. **Annotate Components Gradually:**
   ```typescript
   'use optimize';
   
   function OptimizedComponent() {
     // This component will be optimized
   }
   ```

4. **Monitor Performance:**
   - Use React DevTools Profiler
   - Measure render times
   - Check for regressions

5. **Enable in Production:**
   ```javascript
   reactCompiler: true // Full enablement
   ```

## Troubleshooting

### Common Issues

**Build Errors with Compiler:**
```bash
# Clear babel cache
rm -rf node_modules/.cache/babel-loader

# Restart Metro
npx expo r -c
```

**Runtime Errors:**
```javascript
// Check compilation mode
reactCompiler: {
  compilationMode: 'annotation',
  panicThreshold: 'critical_errors'
}
```

**Performance Regressions:**
```typescript
// Add selective compilation
reactCompiler: {
  sources: (filename) => {
    // Only compile specific files
    return filename.includes('/optimized/');
  }
}
```

---

**Next Steps:**
- Enable React Compiler in your project
- Monitor optimization in React DevTools
- Gradually migrate manual optimizations
- Measure performance improvements
