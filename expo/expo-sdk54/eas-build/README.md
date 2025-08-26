# EAS Build Configuration for SDK 54
<!-- spongebug 2025.1 -->
<!-- File: expo-sdk54/eas-build/README.md -->
<!-- Purpose: Complete EAS Build configuration guide for SDK 54 -->
<!-- Dependencies: EAS CLI, Xcode 16.1+, Android SDK 36+ -->

## Overview

Expo SDK 54 introduces new build requirements and optimizations that require specific EAS Build configurations. This guide covers all necessary setup for successful builds.

## Prerequisites

### EAS CLI Update
```bash
# Update to latest EAS CLI
npm install -g @expo/eas-cli@latest

# Verify version
eas --version

# Login to your Expo account
eas login
```

### Project Setup
```bash
# Initialize EAS Build in your project
eas build:configure

# Verify project configuration
eas build:list --limit 1
```

## Build Images and Requirements

### iOS Build Requirements

#### Xcode 16.1 Configuration
```json
// eas.json
{
  "build": {
    "development": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-medium"
      }
    },
    "preview": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-medium",
        "distribution": "internal"
      }
    },
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-large"
      }
    }
  }
}
```

#### iOS 26 Features Support
```json
// eas.json - For iOS 26 features
{
  "build": {
    "ios26-preview": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0",
        "resourceClass": "m1-large",
        "env": {
          "EXPO_IOS_LIQUID_GLASS_ENABLED": "true",
          "IPHONEOS_DEPLOYMENT_TARGET": "26.0"
        }
      }
    }
  }
}
```

### Android Build Requirements

#### Android 16 (API 36) Configuration
```json
// eas.json
{
  "build": {
    "development": {
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "medium",
        "env": {
          "ANDROID_COMPILE_SDK": "36",
          "ANDROID_TARGET_SDK": "36"
        }
      }
    },
    "preview": {
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "large",
        "distribution": "internal"
      }
    },
    "production": {
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "large"
      }
    }
  }
}
```

## App Configuration for Builds

### iOS Configuration
```json
// app.json
{
  "expo": {
    "ios": {
      "deploymentTarget": "13.4",
      "buildNumber": "1",
      "icon": "./assets/app.icon", // For iOS 26 Liquid Glass icons
      "fallbackIcon": "./assets/icon.png" // Fallback for older iOS
    },
    "plugins": [
      [
        "expo-build-properties",
        {
          "ios": {
            "newArchEnabled": true,
            "buildReactNativeFromSource": false // Temporary for App Store submission
          }
        }
      ]
    ]
  }
}
```

### Android Configuration
```json
// app.json
{
  "expo": {
    "android": {
      "compileSdkVersion": 36,
      "targetSdkVersion": 36,
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundImage": "./assets/adaptive-icon-background.png"
      },
      "androidNavigationBar": {
        "enforceContrast": false
      },
      "predictiveBackGestureEnabled": true
    }
  }
}
```

## Build Profiles

### Complete eas.json Example
```json
{
  "cli": {
    "version": ">= 5.9.0",
    "requireCommit": false
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-medium"
      },
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "medium",
        "gradleCommand": ":app:assembleDebug"
      },
      "channel": "development",
      "env": {
        "NODE_ENV": "development"
      }
    },
    
    "preview": {
      "distribution": "internal",
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-medium",
        "simulator": false
      },
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "large",
        "buildType": "apk"
      },
      "channel": "preview",
      "env": {
        "NODE_ENV": "preview"
      }
    },
    
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "resourceClass": "m1-large",
        "autoIncrement": "buildNumber"
      },
      "android": {
        "image": "ubuntu-22.04-android-34",
        "resourceClass": "large",
        "autoIncrement": "versionCode"
      },
      "channel": "production",
      "env": {
        "NODE_ENV": "production"
      }
    },

    "ios26-features": {
      "extends": "production",
      "ios": {
        "image": "macos-sequoia-15.5-xcode-26.0",
        "resourceClass": "m1-large",
        "env": {
          "EXPO_IOS_LIQUID_GLASS_ENABLED": "true",
          "IPHONEOS_DEPLOYMENT_TARGET": "26.0"
        }
      }
    }
  },
  
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCD123456"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production"
      }
    }
  }
}
```

## Known Issues and Workarounds

### App Store Submission Issue (iOS)

#### Problem
React Native 0.81.0 precompiled XCFrameworks cannot be submitted to App Store.

#### Temporary Workaround
```json
// app.json - Until React Native 0.81.1 is available
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "ios": {
            "buildReactNativeFromSource": false // Disable precompiled frameworks
          }
        }
      ]
    ]
  }
}
```

#### Alternative Build Profile
```json
// eas.json - Separate profile for App Store submission
{
  "build": {
    "app-store-submission": {
      "extends": "production",
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "env": {
          "EXPO_BUILD_REACT_NATIVE_FROM_SOURCE": "true"
        }
      }
    }
  }
}
```

### Build Script Configuration
```bash
#!/bin/bash
# build-with-workaround.sh

echo "🏗️ Building for App Store submission with workaround..."

# Temporarily modify app.json
cp app.json app.json.backup

# Add workaround
cat app.json | jq '.expo.plugins = (.expo.plugins // []) + [["expo-build-properties", {"ios": {"buildReactNativeFromSource": false}}]]' > app.json.tmp
mv app.json.tmp app.json

# Build
eas build --platform ios --profile production

# Restore original configuration
mv app.json.backup app.json

echo "✅ Build complete with App Store submission workaround"
```

## Environment Variables

### Build-time Environment Variables
```json
// eas.json
{
  "build": {
    "production": {
      "env": {
        "NODE_ENV": "production",
        "EXPO_PUBLIC_API_URL": "https://api.production.com",
        "EXPO_PUBLIC_ANALYTICS_ID": "prod-analytics-id",
        
        // SDK 54 specific variables
        "EXPO_USE_EXPERIMENTAL_IMPORT_SUPPORT": "true",
        "EXPO_UNSTABLE_LIVE_BINDINGS": "true",
        "REACT_COMPILER_ENABLED": "true",
        
        // iOS 26 features
        "EXPO_IOS_LIQUID_GLASS_ENABLED": "true",
        
        // Performance monitoring
        "EXPO_PERFORMANCE_MONITORING": "true"
      }
    }
  }
}
```

### Secrets Management
```bash
# Set build secrets
eas secret:create --scope project --name API_SECRET_KEY --value "your-secret-key"
eas secret:create --scope project --name GOOGLE_SERVICES_KEY --value "$(cat google-services.json)"

# Use in builds
eas secret:list
```

## Build Optimization

### Resource Class Selection
```json
// eas.json - Optimize build times with appropriate resource classes
{
  "build": {
    "development": {
      "ios": {
        "resourceClass": "m1-medium" // Faster for development builds
      },
      "android": {
        "resourceClass": "medium"
      }
    },
    "production": {
      "ios": {
        "resourceClass": "m1-large" // Maximum performance for production
      },
      "android": {
        "resourceClass": "large"
      }
    }
  }
}
```

### Caching Configuration
```json
// eas.json - Optimize build caching
{
  "build": {
    "production": {
      "cache": {
        "disabled": false,
        "key": "sdk54-production-v1",
        "paths": [
          "node_modules",
          ".expo",
          "ios/Pods", // iOS dependencies
          "android/.gradle" // Android Gradle cache
        ]
      }
    }
  }
}
```

## Advanced Build Configurations

### Custom Build Hooks
```json
// eas.json
{
  "build": {
    "production": {
      "prebuild": {
        "scripts": [
          {
            "name": "Pre-build setup",
            "command": "./scripts/pre-build.sh"
          }
        ]
      },
      "postbuild": {
        "scripts": [
          {
            "name": "Post-build cleanup", 
            "command": "./scripts/post-build.sh"
          }
        ]
      }
    }
  }
}
```

#### Pre-build Script Example
```bash
#!/bin/bash
# scripts/pre-build.sh

echo "🚀 Pre-build setup for SDK 54..."

# Verify SDK 54 requirements
echo "Checking SDK version..."
SDK_VERSION=$(cat package.json | jq -r '.dependencies.expo' | sed 's/[^0-9]//g' | cut -c1-2)
if [ "$SDK_VERSION" != "54" ]; then
  echo "❌ SDK 54 required, found SDK $SDK_VERSION"
  exit 1
fi

# Verify Node.js version
NODE_VERSION=$(node --version | sed 's/v//' | cut -d'.' -f1)
if [ "$NODE_VERSION" -lt "20" ]; then
  echo "❌ Node.js 20+ required"
  exit 1
fi

# Clean dependencies for fresh build
echo "Cleaning dependencies..."
rm -rf node_modules/.cache
rm -rf .expo

echo "✅ Pre-build setup complete"
```

#### Post-build Script Example
```bash
#!/bin/bash
# scripts/post-build.sh

echo "🏁 Post-build processing..."

# Upload build artifacts to custom storage
if [ "$EAS_BUILD_PROFILE" = "production" ]; then
  echo "Archiving production build..."
  # Custom archival logic
fi

# Send build notifications
echo "Sending build completion notification..."
curl -X POST "https://hooks.slack.com/your-webhook" \
  -H "Content-Type: application/json" \
  -d "{\"text\":\"SDK 54 build completed for $EAS_BUILD_PLATFORM\"}"

echo "✅ Post-build processing complete"
```

### Conditional Configuration
```json
// eas.json - Platform-specific configurations
{
  "build": {
    "production": {
      "ios": {
        "image": "macos-sequoia-15.5-xcode-16.1",
        "env": {
          "EXPO_IOS_SPECIFIC_CONFIG": "true"
        }
      },
      "android": {
        "image": "ubuntu-22.04-android-34",
        "env": {
          "EXPO_ANDROID_SPECIFIC_CONFIG": "true"
        },
        "gradleCommand": ":app:bundleRelease"
      }
    }
  }
}
```

## Build Monitoring and Debugging

### Build Status Monitoring
```bash
#!/bin/bash
# monitor-builds.sh

echo "📊 Monitoring EAS builds..."

# Get latest builds
eas build:list --limit 5 --json > latest-builds.json

# Check for failures
failures=$(cat latest-builds.json | jq '[.[] | select(.status == "errored")] | length')

if [ "$failures" -gt 0 ]; then
  echo "❌ Found $failures failed builds"
  
  # Get error details
  cat latest-builds.json | jq '.[] | select(.status == "errored") | {id: .id, platform: .platform, error: .error}'
else
  echo "✅ All recent builds successful"
fi

# Build time analysis
echo "📈 Build time analysis:"
cat latest-builds.json | jq '.[] | select(.completedAt != null) | {
  platform: .platform,
  duration: (.completedAt | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime) - (.createdAt | strptime("%Y-%m-%dT%H:%M:%S.%fZ") | mktime)
} | .duration /= 60 | {platform: .platform, minutes: (.duration | floor)}'
```

### Debug Builds
```json
// eas.json - Debug configuration
{
  "build": {
    "debug": {
      "extends": "development",
      "ios": {
        "simulator": true,
        "env": {
          "EXPO_DEBUG": "true",
          "NODE_ENV": "development"
        }
      },
      "android": {
        "gradleCommand": ":app:assembleDebug",
        "env": {
          "EXPO_DEBUG": "true",
          "NODE_ENV": "development"
        }
      }
    }
  }
}
```

## Testing Builds

### Build Validation Script
```bash
#!/bin/bash
# validate-build.sh

BUILD_ID=$1

if [ -z "$BUILD_ID" ]; then
  echo "Usage: ./validate-build.sh <build-id>"
  exit 1
fi

echo "🔍 Validating build $BUILD_ID..."

# Get build details
eas build:view $BUILD_ID --json > build-details.json

# Check build status
status=$(cat build-details.json | jq -r '.status')
platform=$(cat build-details.json | jq -r '.platform')

if [ "$status" = "finished" ]; then
  echo "✅ Build $BUILD_ID ($platform) completed successfully"
  
  # Download and validate build artifact
  artifact_url=$(cat build-details.json | jq -r '.artifacts.buildUrl')
  
  if [ "$platform" = "ios" ]; then
    echo "📱 iOS build artifact: $artifact_url"
    # Additional iOS-specific validation
  else
    echo "🤖 Android build artifact: $artifact_url"
    # Additional Android-specific validation
  fi
else
  echo "❌ Build $BUILD_ID failed with status: $status"
  exit 1
fi
```

## Common Build Commands

### Essential Commands
```bash
# Build for all platforms
eas build --platform all --profile production

# Build with specific profile
eas build --platform ios --profile ios26-features

# Clear cache and rebuild
eas build --platform all --profile production --clear-cache

# Auto-submit after successful build
eas build --platform ios --profile production --auto-submit

# Monitor build progress
eas build:list --limit 10

# View detailed build logs
eas build:view <build-id>

# Cancel running build
eas build:cancel <build-id>
```

### Troubleshooting Commands
```bash
# Check project configuration
eas build:configure

# Validate eas.json
eas config --print

# Test credentials
eas credentials

# View build queue
eas build:list --status in-queue

# Download build logs
eas build:view <build-id> --logs > build-logs.txt
```

---

**Next Steps:**
1. Update your eas.json with SDK 54 requirements
2. Test builds on both platforms with new configurations
3. Implement workarounds for known issues
4. Monitor build performance and optimization
5. Set up automated build validation
