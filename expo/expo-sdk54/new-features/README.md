# New Features
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/new-features/README.md -->
<!-- Purpose: New packages, APIs, and capabilities in SDK 54 -->
<!-- Dependencies: Expo SDK 54, React Native 0.81 -->

## New Packages

### `expo-app-integrity` - App Store Verification
Verify app authenticity using platform-specific APIs.

#### Platforms
- **iOS:** DeviceCheck (`DCAppAttestService`)
- **Android:** Play Integrity API

#### Use Cases
- Confirm App Store/Play Store installation
- Verify device authenticity 
- Prevent tampering detection
- Enterprise security compliance

```typescript
import * as AppIntegrity from 'expo-app-integrity';

// Check app integrity
const result = await AppIntegrity.checkIntegrityAsync();
console.log('App is genuine:', result.isGenuine);
```

**Status:** Beta - Package name will change from `@expo/app-integrity` to `expo-app-integrity`

### `expo/blob` - Binary Large Objects
Web-compatible Blob API for React Native platforms.

#### Features
- **W3C Specification Compliant** - Familiar web APIs
- **Cross-platform** - iOS and Android support
- **Type Safety** - Full TypeScript support
- **Streaming Support** - Handle large files efficiently

```typescript
import { Blob } from 'expo/blob';

// Create blob from text
const textBlob = new Blob(['Hello, world!'], { 
  type: 'text/plain' 
});

// Create blob from array buffer
const buffer = new ArrayBuffer(1024);
const bufferBlob = new Blob([buffer], { 
  type: 'application/octet-stream' 
});

// Read blob content
const text = await textBlob.text();
const arrayBuffer = await textBlob.arrayBuffer();
```

**Status:** Beta - Not included in Expo Go yet

### Enhanced `expo-file-system` (New Default API)

#### Object-Oriented API
The new API provides intuitive file and directory handling:

```typescript
import { Directory, File } from 'expo-file-system';

// Directory operations
const documentsDir = Directory.documentDirectory;
const customDir = documentsDir.createDirectory('myApp');

// File operations
const configFile = customDir.createFile('config.json');
await configFile.write(JSON.stringify({ version: '1.0' }));
const content = await configFile.readAsString();
```

#### New Capabilities
- **SAF URI Support** (Android) - Storage Access Framework
- **Bundled Assets** - Access app bundle files on both platforms
- **Better Error Handling** - More descriptive error messages
- **Type Safety** - Full TypeScript support

#### Migration
```typescript
// Legacy API (still available at 'expo-file-system/legacy')
import * as FileSystem from 'expo-file-system/legacy';

// New API (default export)
import { Directory, File } from 'expo-file-system';
```

### `expo-sqlite` localStorage API
Web-compatible localStorage implementation using SQLite.

```typescript
import { localStorage } from 'expo-sqlite/kv-store';

// Standard localStorage API
localStorage.setItem('user', JSON.stringify(userData));
const user = JSON.parse(localStorage.getItem('user') || '{}');
localStorage.removeItem('user');
localStorage.clear();

// With async variants
import { localStorageAsync } from 'expo-sqlite/kv-store';
await localStorageAsync.setItem('data', value);
```

#### Benefits
- **Cross-platform Storage** - Same API on web and native
- **Persistent Storage** - SQLite backend ensures reliability
- **Performance** - Optimized for mobile platforms
- **Code Sharing** - Share storage logic between platforms

## Enhanced Existing Packages

### `expo-maps` Improvements

#### Google Maps JSON Styling
```typescript
import MapView from 'expo-maps';

const mapStyle = [
  {
    elementType: 'geometry',
    stylers: [{ color: '#1d2c4d' }]
  },
  {
    featureType: 'water',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#8ec3b9' }]
  }
];

<MapView 
  provider="google"
  customMapStyle={mapStyle}
/>
```

#### Google Cloud Map IDs
```typescript
<MapView 
  provider="google"
  googleCloudMapId="your-cloud-map-id"
/>
```

#### Apple Maps POI Filtering
```typescript
<MapView 
  provider="apple"
  pointsOfInterestFilter={{
    includeCategories: ['restaurant', 'gasStation'],
    excludeCategories: ['school']
  }}
/>
```

### `expo-sqlite` Extensions

#### SQLite Extensions Support
```typescript
import { loadExtensionAsync } from 'expo-sqlite';

// Load custom extension
await loadExtensionAsync(db, 'path/to/extension.so');

// Load bundled sqlite-vec extension
await loadExtensionAsync(db, 'sqlite-vec');
```

#### sqlite-vec Integration
Vector processing for AI and ML workloads:

```typescript
import { openDatabase } from 'expo-sqlite';
import { withSQLiteVecExtension } from 'expo-sqlite/with-sqlite-vec';

const db = openDatabase('vectors.db');

// Enable sqlite-vec in config plugin
{
  "expo": {
    "plugins": [
      ["expo-sqlite", {
        "withSQLiteVecExtension": true
      }]
    ]
  }
}

// Use vector operations
await db.execAsync(`
  CREATE VIRTUAL TABLE embeddings USING vec0(
    id INTEGER PRIMARY KEY,
    vector FLOAT[768]
  );
`);
```

## iOS 26 Liquid Glass Features

### Icon Composer Support
Create Liquid Glass icons with Apple's Icon Composer app:

```json
// app.json configuration
{
  "ios": {
    "icon": "./assets/app.icon"
  },
  "android": {
    "adaptiveIcon": {
      "foregroundImage": "./assets/adaptive-icon.png",
      "backgroundImage": "./assets/adaptive-icon-background.png"
    }
  }
}
```

**Requirements:**
- macOS with Icon Composer app
- Outputs `.icon` file format
- Automatic fallback on iOS ≤ 19

### Expo UI Liquid Glass Components (Beta)

#### Glass Effects
```typescript
import { Host, HStack, Text } from "@expo/ui/swift-ui";
import { glassEffect, padding } from '@expo/ui/build/swift-ui/modifiers';

<Host matchContents>
  <HStack
    alignment="center"
    modifiers={[
      padding({ all: 16 }),
      glassEffect({
        glass: { variant: 'regular' }
      })
    ]}
  >
    <Text>Regular glass effect</Text>
  </HStack>
</Host>
```

#### Glass Button Variants
```typescript
import { Button, Host, VStack } from "@expo/ui/swift-ui";

<Host matchContents>
  <VStack spacing={8}>
    <Button
      variant="glassProminent"
      color="orange"
      systemImage="questionmark.circle"
    >
      Help & support
    </Button>
    <Button 
      variant="glass" 
      systemImage="person"
    >
      View profile
    </Button>
  </VStack>
</Host>
```

**Coming Soon:** GlassContainerEffect for combining multiple views

## Apple TV & Android TV Enhancements

### Expanded Package Support
New Apple TV support added to:
- `expo-sqlite`
- `expo-background-task` 
- `expo-task-manager`
- `expo-insights`
- `expo-image-loader`
- `expo-image-manipulator`
- `expo-video-thumbnails`

### Development Client Support
- **Apple TV:** Experimental support (authentication pending)
- **Android TV:** Full support available
- **Build Performance:** Leverages precompiled frameworks

## Expo Router Enhancements

### iOS View Controller Previews
```typescript
import { Link } from 'expo-router';

<Link 
  href="/profile" 
  preview={true}
  previewComponent={<UserPreview />}
  contextActions={[
    { title: 'Message', action: () => openMessage() },
    { title: 'Call', action: () => makeCall() }
  ]}
>
  User Profile
</Link>
```

### Native Tabs (Beta)
```typescript
// Import with unstable prefix (will be removed when stable)
import { Tabs } from 'expo-router/unstable-native-tabs';

<Tabs>
  <Tabs.Screen 
    name="home" 
    options={{
      title: 'Home',
      tabBarIcon: 'house.fill',
      liquidGlass: true // iOS 26 feature
    }} 
  />
</Tabs>
```

**Features:**
- Liquid glass effects on iOS
- Automatic scrolling on tab press
- Native platform behaviors
- Better performance than JS implementation

### Enhanced Web Modals
Modals now emulate iPad/iPhone behavior instead of full-screen pages:
- Proper modal backdrop
- Swipe-to-dismiss gestures
- Modal presentation animations
- Better responsive behavior

### Server Middleware (Experimental)
```typescript
// middleware.ts
export function middleware(request: Request) {
  // Run before route handlers
  const response = checkAuth(request);
  if (!response.authorized) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  return NextResponse.next();
}
```

## Runtime Enhancements

### Streaming Support
New streaming APIs for AI and data processing:

```typescript
// TextDecoderStream and TextEncoderStream available
const decoder = new TextDecoderStream();
const encoder = new TextEncoderStream();

// Use with fetch streaming
const response = await fetch('/api/stream');
const stream = response.body
  ?.pipeThrough(decoder)
  ?.pipeThrough(new TransformStream({
    transform(chunk, controller) {
      // Process streaming data
      controller.enqueue(processChunk(chunk));
    }
  }));
```

---

**Next Steps:**
- Review [API Reference](../api-reference/README.md) for detailed implementation
- Check [Platform-Specific Updates](../platform-specific/README.md) for platform features
- See [Breaking Changes](../breaking-changes/README.md) for migration requirements
