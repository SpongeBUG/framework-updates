# API Reference
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/api-reference/README.md -->
<!-- Purpose: Detailed API documentation for new and updated packages -->
<!-- Dependencies: Expo SDK 54, TypeScript definitions -->

## New Package APIs

### `expo-app-integrity`

Verify app authenticity using platform-specific integrity APIs.

```typescript
import * as AppIntegrity from 'expo-app-integrity';

// Type definitions
interface IntegrityResult {
  isGenuine: boolean;
  deviceTrusted: boolean;
  appStoreInstalled: boolean;
  details?: {
    attestationToken?: string;
    riskScore?: number;
    timestamp: number;
  };
}

interface IntegrityOptions {
  challenge?: string;
  timeoutMs?: number;
  includeDeviceDetails?: boolean;
}
```

#### Methods

**`checkIntegrityAsync(options?: IntegrityOptions): Promise<IntegrityResult>`**

Performs app integrity verification.

```typescript
// Basic integrity check
const result = await AppIntegrity.checkIntegrityAsync();
console.log('App is genuine:', result.isGenuine);

// With challenge and options
const challengeResult = await AppIntegrity.checkIntegrityAsync({
  challenge: 'unique-server-challenge',
  timeoutMs: 10000,
  includeDeviceDetails: true
});

if (challengeResult.isGenuine) {
  // App is verified as genuine
  sendSecureRequest(challengeResult.details.attestationToken);
}
```

**`isAvailableAsync(): Promise<boolean>`**

Check if integrity checking is available on the device.

```typescript
const available = await AppIntegrity.isAvailableAsync();
if (!available) {
  console.log('Integrity checking not supported on this device');
}
```

#### Platform Support
- **iOS:** DeviceCheck (`DCAppAttestService`)
- **Android:** Play Integrity API
- **Web:** Not supported

---

### `expo/blob`

Web-compatible Blob API for React Native platforms.

```typescript
import { Blob } from 'expo/blob';

// Constructor
new Blob(array?: BlobPart[], options?: BlobPropertyBag)

// Types
type BlobPart = string | ArrayBuffer | ArrayBufferView | Blob;

interface BlobPropertyBag {
  type?: string;
  endings?: 'transparent' | 'native';
}
```

#### Properties

**`size: number`** - Size in bytes
**`type: string`** - MIME type

#### Methods

**`text(): Promise<string>`**

Read blob content as text.

```typescript
const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
const text = await blob.text();
console.log(text); // "Hello, world!"
```

**`arrayBuffer(): Promise<ArrayBuffer>`**

Read blob content as ArrayBuffer.

```typescript
const buffer = new ArrayBuffer(1024);
const blob = new Blob([buffer], { type: 'application/octet-stream' });
const result = await blob.arrayBuffer();
console.log(result.byteLength); // 1024
```

**`slice(start?: number, end?: number, contentType?: string): Blob`**

Create a new blob with subset of data.

```typescript
const originalBlob = new Blob(['Hello, world!']);
const slicedBlob = originalBlob.slice(0, 5);
const text = await slicedBlob.text();
console.log(text); // "Hello"
```

**`stream(): ReadableStream`**

Get readable stream of blob data.

```typescript
const blob = new Blob(['streaming data']);
const stream = blob.stream();
const reader = stream.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log(new TextDecoder().decode(value));
}
```

---

### Enhanced `expo-file-system`

New object-oriented API for file and directory operations.

```typescript
import { Directory, File, Asset } from 'expo-file-system';

// Base classes
class Directory {
  readonly path: string;
  readonly name: string;
  
  // Static properties
  static readonly documentDirectory: Directory;
  static readonly cacheDirectory: Directory;
  static readonly bundleDirectory: Directory;
}

class File {
  readonly path: string;
  readonly name: string;
  readonly directory: Directory;
}
```

#### Directory Methods

**`createDirectory(name: string): Directory`**

Create a subdirectory.

```typescript
const documentsDir = Directory.documentDirectory;
const appDir = documentsDir.createDirectory('MyApp');
const dataDir = appDir.createDirectory('data');
```

**`createFile(name: string): File`**

Create a file in this directory.

```typescript
const configFile = appDir.createFile('config.json');
const logFile = appDir.createFile('app.log');
```

**`exists(): Promise<boolean>`**

Check if directory exists.

```typescript
const exists = await dataDir.exists();
if (!exists) {
  await dataDir.create();
}
```

**`list(): Promise<(File | Directory)[]>`**

List directory contents.

```typescript
const contents = await appDir.list();
for (const item of contents) {
  if (item instanceof File) {
    console.log('File:', item.name);
  } else {
    console.log('Directory:', item.name);
  }
}
```

**`delete(): Promise<void>`**

Delete directory and all contents.

```typescript
await tempDir.delete();
```

#### File Methods

**`write(content: string | ArrayBuffer | Blob): Promise<void>`**

Write content to file.

```typescript
// Write text
await configFile.write(JSON.stringify({ version: '1.0' }));

// Write binary data
const buffer = new ArrayBuffer(1024);
await dataFile.write(buffer);

// Write blob
const blob = new Blob(['file content']);
await textFile.write(blob);
```

**`readAsString(encoding?: 'utf8' | 'base64'): Promise<string>`**

Read file as string.

```typescript
const config = await configFile.readAsString();
const data = JSON.parse(config);

// Read as base64
const base64Data = await imageFile.readAsString('base64');
```

**`readAsArrayBuffer(): Promise<ArrayBuffer>`**

Read file as ArrayBuffer.

```typescript
const buffer = await binaryFile.readAsArrayBuffer();
const view = new DataView(buffer);
```

**`copy(destination: File): Promise<File>`**

Copy file to another location.

```typescript
const backupFile = backupDir.createFile('config-backup.json');
await configFile.copy(backupFile);
```

**`move(destination: File): Promise<File>`**

Move file to another location.

```typescript
const newLocation = otherDir.createFile('moved-file.txt');
await originalFile.move(newLocation);
```

#### Asset Handling

**`Asset.fromModule(moduleId: number): Asset`**

Access bundled assets.

```typescript
import logoAsset from './assets/logo.png';

const logo = Asset.fromModule(logoAsset);
const logoFile = await logo.downloadAsync();
const logoData = await logoFile.readAsArrayBuffer();
```

#### SAF URI Support (Android)

```typescript
// Handle Storage Access Framework URIs on Android
import { Directory } from 'expo-file-system';

const safUri = 'content://com.android.externalstorage.documents/...';
const externalDir = new Directory(safUri);
const files = await externalDir.list();
```

---

### Enhanced `expo-sqlite`

#### localStorage API

Web-compatible localStorage implementation.

```typescript
import { localStorage, localStorageAsync } from 'expo-sqlite/kv-store';

// Synchronous API (web-compatible)
localStorage.setItem('user', JSON.stringify({ name: 'John', age: 30 }));
const user = JSON.parse(localStorage.getItem('user') || '{}');
localStorage.removeItem('user');
localStorage.clear();

// Get all keys
const keys = localStorage.keys();
const length = localStorage.length;

// Async API (recommended for React Native)
await localStorageAsync.setItem('data', 'value');
const data = await localStorageAsync.getItem('data');
await localStorageAsync.removeItem('data');
await localStorageAsync.clear();
```

#### SQLite Extensions

**`loadExtensionAsync(database: Database, extensionPath: string): Promise<void>`**

Load SQLite extension.

```typescript
import { openDatabase, loadExtensionAsync } from 'expo-sqlite';

const db = openDatabase('app.db');

// Load custom extension
await loadExtensionAsync(db, 'path/to/extension.so');

// Load bundled sqlite-vec extension
await loadExtensionAsync(db, 'sqlite-vec');
```

**sqlite-vec Integration**

Vector processing for AI workloads.

```typescript
// Enable in config plugin
{
  "expo": {
    "plugins": [
      ["expo-sqlite", {
        "withSQLiteVecExtension": true
      }]
    ]
  }
}

// Usage
await db.execAsync(`
  CREATE VIRTUAL TABLE embeddings USING vec0(
    id INTEGER PRIMARY KEY,
    vector FLOAT[768]
  );
`);

// Insert vectors
await db.runAsync(
  'INSERT INTO embeddings (vector) VALUES (?)',
  [JSON.stringify(new Array(768).fill(0.5))]
);

// Vector similarity search
const results = await db.getAllAsync(`
  SELECT id, vec_distance_cosine(vector, ?) as distance
  FROM embeddings
  ORDER BY distance
  LIMIT 10
`, [JSON.stringify(queryVector)]);
```

---

## Updated Package APIs

### `expo-updates`

#### Header Overrides

**`setUpdateRequestHeadersOverride(headers: Record<string, string>): void`**

Override update request headers at runtime.

```typescript
import * as Updates from 'expo-updates';

// Set custom headers for employee builds
Updates.setUpdateRequestHeadersOverride({
  'X-Employee-Channel': 'beta',
  'X-Feature-Flag': 'new-ui-enabled'
});

// Fetch updates with new headers
await Updates.fetchUpdateAsync();
```

#### Download Progress

**`useUpdates(): UseUpdatesReturnType`**

Hook with download progress tracking.

```typescript
import { useUpdates } from 'expo-updates';

function UpdateComponent() {
  const {
    isUpdateAvailable,
    isUpdatePending,
    downloadProgress, // New property
    lastUpdateInfo
  } = useUpdates();

  if (downloadProgress && downloadProgress.totalBytesExpected > 0) {
    const percentage = (downloadProgress.totalBytesWritten / downloadProgress.totalBytesExpected) * 100;
    
    return (
      <View>
        <Text>Downloading update: {percentage.toFixed(0)}%</Text>
        <ProgressBar progress={percentage / 100} />
      </View>
    );
  }

  // Rest of component
}

// Types
interface DownloadProgress {
  totalBytesExpected: number;
  totalBytesWritten: number;
}
```

#### Reload Screen Configuration

**`reloadAsync(options?: ReloadAsyncOptions): Promise<void>`**

Reload app with custom screen.

```typescript
interface ReloadScreenOptions {
  backgroundColor?: string;
  image?: ImageSource;
  imageResizeMode?: 'cover' | 'contain' | 'center';
  imageFullScreen?: boolean;
  fade?: boolean;
}

// Custom reload screen
await Updates.reloadAsync({
  reloadScreenOptions: {
    backgroundColor: '#1a1a1a',
    image: require('./assets/reload-screen.jpg'),
    imageResizeMode: 'cover',
    imageFullScreen: true,
    fade: true
  }
});
```

**Development Testing**

```typescript
// Test reload screen in development
Updates.showReloadScreen({
  backgroundColor: '#fa0000',
  image: require('./assets/reload.jpg'),
  fade: true
});

// Hide after testing
setTimeout(() => {
  Updates.hideReloadScreen();
}, 3000);
```

### `expo-maps`

#### Google Maps Styling

**JSON-based Styling**

```typescript
import MapView from 'expo-maps';

const darkMapStyle = [
  {
    elementType: 'geometry',
    stylers: [{ color: '#212121' }]
  },
  {
    elementType: 'labels.icon',
    stylers: [{ visibility: 'off' }]
  },
  {
    featureType: 'water',
    elementType: 'geometry',
    stylers: [{ color: '#000255' }]
  }
];

<MapView 
  provider="google"
  customMapStyle={darkMapStyle}
  style={styles.map}
/>
```

**Google Cloud Map IDs**

```typescript
<MapView 
  provider="google"
  googleCloudMapId="your-cloud-based-map-id"
  style={styles.map}
/>
```

#### Apple Maps POI Filtering

```typescript
interface POIFilter {
  includeCategories?: POICategory[];
  excludeCategories?: POICategory[];
}

type POICategory = 
  | 'airport'
  | 'amusementPark' 
  | 'bank'
  | 'beach'
  | 'brewery'
  | 'cafe'
  | 'campground'
  | 'carRental'
  | 'evCharger'
  | 'fireStation'
  | 'fitnessCenter'
  | 'foodMarket'
  | 'gasStation'
  | 'hospital'
  | 'hotel'
  | 'laundry'
  | 'library'
  | 'marina'
  | 'movieTheater'
  | 'museum'
  | 'nationalPark'
  | 'nightlife'
  | 'park'
  | 'parking'
  | 'pharmacy'
  | 'police'
  | 'postOffice'
  | 'publicTransport'
  | 'restaurant'
  | 'restroom'
  | 'school'
  | 'stadium'
  | 'store'
  | 'theater'
  | 'university'
  | 'winery'
  | 'zoo';

<MapView 
  provider="apple"
  pointsOfInterestFilter={{
    includeCategories: ['restaurant', 'gasStation', 'hospital'],
    excludeCategories: ['school', 'university']
  }}
/>
```

### Expo Router Enhancements

#### Link Previews (iOS)

```typescript
import { Link } from 'expo-router';

function UserCard({ user }) {
  return (
    <Link 
      href={`/users/${user.id}`}
      preview={true}
      previewComponent={
        <UserPreview user={user} />
      }
      contextActions={[
        {
          title: 'Message',
          systemImage: 'message.fill',
          action: () => openMessage(user.id)
        },
        {
          title: 'Call',
          systemImage: 'phone.fill', 
          action: () => makeCall(user.phone)
        },
        {
          title: 'Favorite',
          systemImage: 'heart.fill',
          action: () => toggleFavorite(user.id)
        }
      ]}
    >
      <UserCard user={user} />
    </Link>
  );
}
```

#### Native Tabs (Beta)

```typescript
import { Tabs } from 'expo-router/unstable-native-tabs';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: 'blue',
        liquidGlass: true, // iOS 26 feature
      }}
    >
      <Tabs.Screen 
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="home" focused={focused} color={color} size={size} />
          ),
          tabBarBadge: unreadCount > 0 ? unreadCount : undefined,
        }}
      />
      <Tabs.Screen 
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="person" focused={focused} color={color} size={size} />
          ),
        }}
      />
    </Tabs>
  );
}
```

#### Server Middleware (Experimental)

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  // Authentication check
  const authToken = request.headers.get('authorization');
  
  if (!authToken && request.nextUrl.pathname.startsWith('/api/protected')) {
    return new NextResponse('Unauthorized', { status: 401 });
  }
  
  // Rate limiting
  const ip = request.ip || 'unknown';
  if (isRateLimited(ip)) {
    return new NextResponse('Rate limited', { status: 429 });
  }
  
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');
  
  return response;
}

export const config = {
  matcher: ['/api/:path*', '/protected/:path*']
};
```

---

## iOS 26 Liquid Glass APIs

### Expo UI Components (Beta)

```typescript
import { Host, HStack, VStack, Text, Button } from "@expo/ui/swift-ui";
import { 
  glassEffect, 
  padding, 
  cornerRadius, 
  shadow 
} from '@expo/ui/build/swift-ui/modifiers';

// Glass effect variants
type GlassVariant = 'regular' | 'thin' | 'thick' | 'ultraThin';

interface GlassEffectConfig {
  glass: {
    variant: GlassVariant;
    tint?: {
      color: string;
      intensity: number; // 0.0 - 1.0
    };
  };
  background?: {
    blur: 'systemMaterial' | 'prominent' | 'regular' | 'thin';
    opacity: number;
  };
}

// Button variants
type GlassButtonVariant = 'glass' | 'glassProminent' | 'glassSecondary';

// Example usage
<Host matchContents>
  <VStack spacing={12} modifiers={[padding({ all: 16 })]}>
    <HStack 
      modifiers={[
        glassEffect({
          glass: { variant: 'regular' },
          background: { blur: 'systemMaterial', opacity: 0.8 }
        }),
        cornerRadius(12),
        padding({ horizontal: 16, vertical: 12 })
      ]}
    >
      <Text>Glass Card Content</Text>
    </HStack>
    
    <Button
      variant="glassProminent"
      color="blue"
      systemImage="plus.circle"
    >
      Add Item
    </Button>
  </VStack>
</Host>
```

---

## Streaming APIs

### TextDecoderStream & TextEncoderStream

```typescript
// Available in native runtime for AI and streaming support
const decoder = new TextDecoderStream('utf-8');
const encoder = new TextEncoderStream();

// Use with fetch streaming
async function processStream(url: string) {
  const response = await fetch(url);
  
  if (!response.body) {
    throw new Error('No response body');
  }
  
  const stream = response.body
    .pipeThrough(decoder)
    .pipeThrough(new TransformStream({
      transform(chunk, controller) {
        // Process each text chunk
        const processed = processTextChunk(chunk);
        controller.enqueue(processed);
      }
    }));
    
  const reader = stream.getReader();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    console.log('Processed chunk:', value);
  }
}
```

---

**TypeScript Support:** All new APIs include full TypeScript definitions for better development experience and type safety.

**Platform Compatibility:** Check individual API documentation for platform-specific availability and requirements.