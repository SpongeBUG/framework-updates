# Best Practices for Expo SDK 54
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/BEST_PRACTICES.md -->
<!-- Purpose: Comprehensive best practices guide for SDK 54 development -->
<!-- Dependencies: Expo SDK 54, TypeScript, React Native 0.81 -->

## Development Environment Setup

### Essential Requirements
```bash
# Required versions for optimal SDK 54 experience
Node.js: 20.19.0+
Xcode: 16.1+ (iOS development)
Android SDK: API 36+ (Android development)
EAS CLI: Latest version
```

### Recommended Project Structure
```
project/
├── src/
│   ├── components/         # Reusable UI components
│   ├── features/          # Feature-based organization
│   │   └── auth/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── services/
│   │       └── types/
│   ├── hooks/             # Global custom hooks
│   ├── services/          # API and external services
│   ├── types/             # Global TypeScript types
│   └── utils/             # Utility functions
├── assets/
│   ├── images/
│   ├── fonts/
│   └── icons/
├── app.json
├── eas.json
└── package.json
```

## TypeScript Configuration

### Optimal tsconfig.json
```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "moduleResolution": "NodeNext",
    
    // Path mapping for cleaner imports
    "baseUrl": "./src",
    "paths": {
      "@/components/*": ["./components/*"],
      "@/features/*": ["./features/*"],
      "@/hooks/*": ["./hooks/*"],
      "@/services/*": ["./services/*"],
      "@/types/*": ["./types/*"],
      "@/utils/*": ["./utils/*"]
    }
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.ts",
    "expo-env.d.ts"
  ]
}
```

### Type Safety Best Practices
```typescript
// ✅ Always specify explicit return types for public functions
export function processUserData(data: UserData): ProcessedUser {
  return {
    id: data.id,
    name: data.name,
    isActive: data.status === 'active'
  };
}

// ✅ Use branded types for IDs
type UserId = string & { readonly brand: unique symbol };
type ProductId = string & { readonly brand: unique symbol };

// ✅ Leverage noUncheckedIndexedAccess
function getUserById(users: User[], id: string): User | undefined {
  // TypeScript will enforce null checking with noUncheckedIndexedAccess
  return users.find(user => user.id === id);
}

// ✅ Never use any - prefer unknown
function parseResponse(response: unknown): ParsedData {
  if (isValidResponse(response)) {
    return response;
  }
  throw new Error('Invalid response');
}
```

## React Compiler Optimization

### Writing Compiler-Friendly Code
```typescript
// ✅ Keep components pure and focused
function UserCard({ user, onPress }: UserCardProps) {
  // React Compiler can automatically optimize this
  const displayName = user.firstName + ' ' + user.lastName;
  const isVip = user.tier === 'premium';
  
  return (
    <TouchableOpacity onPress={() => onPress(user.id)}>
      <View style={isVip ? styles.vipCard : styles.regularCard}>
        <Text>{displayName}</Text>
        <Text>{user.email}</Text>
      </View>
    </TouchableOpacity>
  );
}

// ✅ Split complex components
function UserProfile({ user }: UserProfileProps) {
  return (
    <ScrollView>
      <UserHeader user={user} />
      <UserDetails user={user} />
      <UserActions user={user} />
    </ScrollView>
  );
}

// ✅ Use stable references for configuration
const CHART_OPTIONS = {
  responsive: true,
  plugins: { legend: { position: 'top' as const } }
};

function Chart({ data }: ChartProps) {
  return <MyChart data={data} options={CHART_OPTIONS} />;
}
```

### Manual Optimization When Needed
```typescript
// If React Compiler doesn't optimize correctly, add manual optimization
import { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ 
  data, 
  filters 
}: ExpensiveComponentProps) {
  const processedData = useMemo(() => {
    return data
      .filter(item => filters.includes(item.category))
      .sort((a, b) => a.priority - b.priority);
  }, [data, filters]);

  const handleItemPress = useCallback((item: Item) => {
    // Handle press
  }, []);

  return (
    <FlatList 
      data={processedData}
      renderItem={({ item }) => (
        <ItemCard item={item} onPress={handleItemPress} />
      )}
    />
  );
});
```

## New Architecture Adoption

### Enabling New Architecture
```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "newArchEnabled": true,
          "proguardMinifyEnabled": true,
          "android": {
            "enableMinifyInReleaseBuilds": true
          }
        }
      ]
    ]
  }
}
```

### Migration Strategy
```typescript
// Gradual migration approach
class ModernComponent {
  private useNewArchitecture = this.checkNewArchitecture();
  
  private checkNewArchitecture(): boolean {
    return global.__turboModuleProxy != null && 
           global.nativeFabricUIManager != null;
  }
  
  public performNativeOperation() {
    if (this.useNewArchitecture) {
      // Use TurboModule APIs
      return this.performTurboModuleOperation();
    } else {
      // Fallback to Legacy Architecture
      return this.performLegacyOperation();
    }
  }
}
```

## File System Best Practices

### Migration to New API
```typescript
// ✅ Use new file system API for new code
import { Directory, File } from 'expo-file-system';

class FileManager {
  private dataDirectory = Directory.documentDirectory;
  
  async saveConfig(config: AppConfig): Promise<void> {
    const configFile = this.dataDirectory.createFile('config.json');
    await configFile.write(JSON.stringify(config, null, 2));
  }
  
  async loadConfig(): Promise<AppConfig | null> {
    const configFile = this.dataDirectory.createFile('config.json');
    
    if (!(await configFile.exists())) {
      return null;
    }
    
    const content = await configFile.readAsString();
    return JSON.parse(content);
  }
  
  async createBackup(filename: string): Promise<File> {
    const sourceFile = this.dataDirectory.createFile(filename);
    const backupDir = this.dataDirectory.createDirectory('backups');
    const backupFile = backupDir.createFile(`${filename}.backup`);
    
    await sourceFile.copy(backupFile);
    return backupFile;
  }
}

// ✅ Gradual migration approach
import * as LegacyFS from 'expo-file-system/legacy';

class HybridFileManager {
  private useNewAPI = __DEV__; // Test new API in development
  
  async saveData(filename: string, data: any): Promise<void> {
    if (this.useNewAPI) {
      const file = Directory.documentDirectory.createFile(filename);
      await file.write(JSON.stringify(data));
    } else {
      const uri = LegacyFS.documentDirectory + filename;
      await LegacyFS.writeAsStringAsync(uri, JSON.stringify(data));
    }
  }
}
```

## Performance Optimization

### Build Performance
```bash
#!/bin/bash
# optimize-builds.sh

# Enable all SDK 54 performance features
export EXPO_USE_EXPERIMENTAL_IMPORT_SUPPORT=true
export EXPO_UNSTABLE_LIVE_BINDINGS=true
export REACT_COMPILER_ENABLED=true

# Clean environment for optimal builds
npx expo r -c
rm -rf node_modules/.cache

# Use optimal build commands
npx expo prebuild --clean
npm run ios # Benefits from precompiled frameworks
```

### Runtime Performance
```typescript
// ✅ Optimize component structure for React Compiler
function OptimizedListScreen({ data, searchQuery }: ListScreenProps) {
  // Simple, compiler-friendly data transformations
  const filteredData = data.filter(item => 
    item.name.toLowerCase().includes(searchQuery.toLowerCase())
  );
  
  const sortedData = filteredData.sort((a, b) => 
    a.priority - b.priority
  );

  return (
    <FlatList
      data={sortedData}
      renderItem={({ item }) => <ListItem item={item} />}
      keyExtractor={item => item.id}
      removeClippedSubviews={true} // Performance optimization
      maxToRenderPerBatch={10}
      windowSize={10}
    />
  );
}

// ✅ Use proper image optimization
import { Image } from 'expo-image';

function OptimizedImage({ source, ...props }: ImageProps) {
  return (
    <Image
      source={source}
      placeholder={{ uri: 'data:image/png;base64,placeholder...' }}
      contentFit="cover"
      transition={200}
      cachePolicy="memory-disk"
      {...props}
    />
  );
}
```

## Platform-Specific Best Practices

### iOS 26 Liquid Glass
```typescript
import { Platform } from 'react-native';
import { Host, VStack } from "@expo/ui/swift-ui";
import { glassEffect, padding } from '@expo/ui/build/swift-ui/modifiers';

// ✅ Progressive enhancement for iOS 26 features
function AdaptiveCard({ children }: { children: React.ReactNode }) {
  const isLiquidGlassAvailable = Platform.OS === 'ios' && 
    parseInt(Platform.Version) >= 26;

  if (isLiquidGlassAvailable) {
    return (
      <Host matchContents>
        <VStack modifiers={[
          padding({ all: 16 }),
          glassEffect({ glass: { variant: 'regular' }})
        ]}>
          {children}
        </VStack>
      </Host>
    );
  }

  // Graceful fallback for older versions
  return (
    <View style={{
      padding: 16,
      backgroundColor: 'rgba(255, 255, 255, 0.9)',
      borderRadius: 12,
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 4,
    }}>
      {children}
    </View>
  );
}
```

### Android 16 Edge-to-Edge
```typescript
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { StatusBar } from 'expo-status-bar';

// ✅ Proper edge-to-edge implementation
function EdgeToEdgeScreen({ children }: { children: React.ReactNode }) {
  const insets = useSafeAreaInsets();
  
  return (
    <View style={{ flex: 1 }}>
      <StatusBar style="auto" translucent />
      <View style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }}>
        {children}
      </View>
    </View>
  );
}

// ✅ Handle predictive back gesture
import { BackHandler } from 'react-native';
import { useFocusEffect } from '@react-navigation/native';

function FormScreen() {
  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);
  
  useFocusEffect(
    useCallback(() => {
      const onBackPress = () => {
        if (hasUnsavedChanges) {
          Alert.alert(
            'Unsaved Changes',
            'You have unsaved changes. Are you sure you want to leave?',
            [
              { text: 'Cancel', style: 'cancel' },
              { text: 'Leave', style: 'destructive', onPress: () => navigation.goBack() }
            ]
          );
          return true; // Prevent default back behavior
        }
        return false; // Allow default back behavior
      };

      const subscription = BackHandler.addEventListener('hardwareBackPress', onBackPress);
      return subscription.remove;
    }, [hasUnsavedChanges])
  );

  // Component implementation
}
```

## State Management Best Practices

### Context and State Colocation
```typescript
// ✅ Keep state close to where it's used
function UserProfile() {
  return (
    <View>
      <UserHeader />
      <UserDetailsSection />
      <UserPreferences />
    </View>
  );
}

function UserDetailsSection() {
  const [isEditing, setIsEditing] = useState(false);
  
  // State is colocated with the component that uses it
  return (
    <View>
      {isEditing ? <EditForm /> : <DisplayDetails />}
      <Button 
        onPress={() => setIsEditing(!isEditing)}
        title={isEditing ? 'Save' : 'Edit'}
      />
    </View>
  );
}

// ✅ Use context for truly global state
const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = useCallback(async (credentials: LoginCredentials) => {
    const user = await authService.login(credentials);
    setUser(user);
    setIsAuthenticated(true);
  }, []);

  const value = useMemo(() => ({
    user,
    isAuthenticated,
    login,
  }), [user, isAuthenticated, login]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
```

## Error Handling and Debugging

### Error Boundaries
```typescript
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }: ErrorFallbackProps) {
  return (
    <View style={{ flex: 1, justifyContent: 'center', padding: 20 }}>
      <Text style={{ fontSize: 18, marginBottom: 16 }}>
        Something went wrong:
      </Text>
      <Text style={{ color: 'red', marginBottom: 16 }}>
        {error.message}
      </Text>
      <Button title="Try again" onPress={resetErrorBoundary} />
    </View>
  );
}

// ✅ Wrap your app with error boundaries
export default function App() {
  return (
    <SafeAreaProvider>
      <ErrorBoundary
        FallbackComponent={ErrorFallback}
        onError={(error, errorInfo) => {
          // Log to crash reporting service
          console.error('Error boundary caught an error:', error, errorInfo);
        }}
      >
        <AuthProvider>
          <Navigation />
        </AuthProvider>
      </ErrorBoundary>
    </SafeAreaProvider>
  );
}
```

### Development Debugging
```typescript
// ✅ Use proper development utilities
if (__DEV__) {
  // React DevTools integration
  import('./ReactotronConfig').then(() => console.log('Reactotron configured'));
  
  // Performance monitoring in development
  const originalConsoleError = console.error;
  console.error = (...args) => {
    if (args[0]?.includes?.('Warning:')) {
      // Handle React warnings appropriately
      originalConsoleError(...args);
    } else {
      originalConsoleError(...args);
    }
  };
}

// ✅ Add debugging helpers
function useDebugValue(value: any, formatter?: (value: any) => any) {
  React.useDebugValue(value, formatter);
  return value;
}

function useWhyDidYouUpdate(name: string, props: Record<string, any>) {
  const previous = useRef<Record<string, any>>();
  
  useEffect(() => {
    if (previous.current) {
      const allKeys = Object.keys({ ...previous.current, ...props });
      const changedProps: Record<string, { from: any; to: any }> = {};
      
      allKeys.forEach(key => {
        if (previous.current![key] !== props[key]) {
          changedProps[key] = {
            from: previous.current![key],
            to: props[key]
          };
        }
      });
      
      if (Object.keys(changedProps).length) {
        console.log('[why-did-you-update]', name, changedProps);
      }
    }
    
    previous.current = props;
  });
}
```

## Testing Best Practices

### Unit Testing Setup
```typescript
// __tests__/utils/test-utils.tsx
import { render } from '@testing-library/react-native';
import { SafeAreaProvider } from 'react-native-safe-area-context';

const AllTheProviders = ({ children }: { children: React.ReactNode }) => {
  return (
    <SafeAreaProvider
      initialMetrics={{
        insets: { top: 0, left: 0, right: 0, bottom: 0 },
        frame: { x: 0, y: 0, width: 0, height: 0 },
      }}
    >
      {children}
    </SafeAreaProvider>
  );
};

export const renderWithProviders = (ui: React.ReactElement, options?: any) =>
  render(ui, { wrapper: AllTheProviders, ...options });
```

### Component Testing
```typescript
// __tests__/components/UserCard.test.tsx
import { renderWithProviders } from '../utils/test-utils';
import { UserCard } from '../../src/components/UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('renders user information correctly', () => {
    const { getByText } = renderWithProviders(
      <UserCard user={mockUser} onPress={jest.fn()} />
    );

    expect(getByText('John Doe')).toBeTruthy();
    expect(getByText('john@example.com')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const onPress = jest.fn();
    const { getByTestId } = renderWithProviders(
      <UserCard user={mockUser} onPress={onPress} />
    );

    fireEvent.press(getByTestId('user-card'));
    expect(onPress).toHaveBeenCalledWith(mockUser.id);
  });
});
```

## Security Best Practices

### App Integrity Implementation
```typescript
import * as AppIntegrity from 'expo-app-integrity';

class SecurityService {
  private integrityCache = new Map<string, boolean>();
  
  async verifyAppIntegrity(): Promise<boolean> {
    try {
      const available = await AppIntegrity.isAvailableAsync();
      if (!available) {
        // Log for monitoring but don't block functionality
        console.warn('App integrity verification not available');
        return true;
      }

      const result = await AppIntegrity.checkIntegrityAsync({
        timeoutMs: 5000,
      });

      const isSecure = result.isGenuine && 
                      result.deviceTrusted && 
                      result.appStoreInstalled;

      if (!isSecure) {
        // Log security concern
        console.error('App integrity check failed', result);
      }

      return isSecure;
    } catch (error) {
      console.error('App integrity check error:', error);
      return false; // Fail secure
    }
  }

  async secureApiCall(endpoint: string, data: any) {
    const isSecure = await this.verifyAppIntegrity();
    
    const headers: HeadersInit = {
      'Content-Type': 'application/json',
    };

    if (isSecure) {
      // Add additional security headers for verified apps
      headers['X-App-Verified'] = 'true';
    }

    return fetch(endpoint, {
      method: 'POST',
      headers,
      body: JSON.stringify(data),
    });
  }
}
```

## Deployment and Monitoring

### EAS Build Configuration
```json
// eas.json
{
  "cli": {
    "version": ">= 5.9.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview"
    },
    "production": {
      "channel": "production",
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production"
      }
    }
  }
}
```

### Performance Monitoring
```typescript
// Monitor app performance in production
import { Profiler } from 'react';

function AppWithMonitoring() {
  const onRenderCallback = useCallback((
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number,
    baseDuration: number,
    startTime: number,
    commitTime: number
  ) => {
    if (!__DEV__ && actualDuration > 100) {
      // Log slow renders in production
      analytics.track('slow_render', {
        component: id,
        phase,
        duration: actualDuration,
      });
    }
  }, []);

  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <App />
    </Profiler>
  );
}
```

## Checklist for Production Readiness

### Pre-Release Checklist
- [ ] New Architecture enabled and tested
- [ ] React Compiler enabled and optimized
- [ ] File system migrated to new API (or legacy imports used)
- [ ] iOS 26 features implemented with fallbacks
- [ ] Android 16 edge-to-edge compliance verified
- [ ] Performance benchmarks meet requirements
- [ ] Error boundaries implemented
- [ ] App integrity verification configured
- [ ] EAS Build profiles optimized
- [ ] Testing coverage sufficient
- [ ] Analytics and monitoring configured

### Post-Release Monitoring
- [ ] Monitor build performance improvements
- [ ] Track runtime performance metrics
- [ ] Monitor error rates and crashes
- [ ] Validate New Architecture benefits
- [ ] Review React Compiler optimizations
- [ ] Check platform-specific feature adoption

---

**Remember:** SDK 54 is the final release to support Legacy Architecture. Plan your New Architecture migration for SDK 55 compatibility.
