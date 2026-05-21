---
name: expo-app-config
category: scaffold
stack: [expo, react-native, typescript, eas-build]
keywords: [app.config.ts, EAS, environment, build, configuration]
source-files: [app.config.ts, .env.example]
---

# Expo App Config

## Problem
You need to configure your Expo app with environment-specific settings, EAS Build profiles, and dynamic environment variables without hardcoding secrets.

## When to Use
- Setting up a new Expo project
- Adding environment-specific configurations (dev, staging, production)
- Configuring EAS Build profiles for Android and iOS
- Managing API URLs, app identifiers, and build settings

## Implementation

### Dependencies
```json
{
  "expo": "^54.0.0",
  "expo-constants": "^16.0.0"
}
```

### Code

Create `app.config.ts`:

```typescript
import * as dotenv from 'dotenv';

dotenv.config();

const getEnv = () => {
  const env = process.env.APP_ENV || 'development';
  return {
    ENVIRONMENT: env,
    SERVER_API_URL: process.env[`${env.toUpperCase()}_API_URL`] || '',
    EAS_PROJECT_ID: process.env.EAS_PROJECT_ID || '',
  };
};

const env = getEnv();

export default {
  expo: {
    name: 'YourAppName',
    slug: 'your-app-slug',
    version: '1.0.0',
    orientation: 'portrait',
    icon: './assets/icon.png',
    userInterfaceStyle: 'automatic',
    splash: {
      image: './assets/splash.png',
      resizeMode: 'contain',
      backgroundColor: '#ffffff',
    },
    ios: {
      supportsTabletMode: true,
      bundleIdentifier: 'com.yourcompany.app',
      buildNumber: '1.0.0',
      infoPlist: {
        NSPhotoLibraryUsageDescription:
          'This app needs access to your photo library.',
        NSCameraUsageDescription: 'This app needs access to your camera.',
      },
    },
    android: {
      adaptiveIcon: {
        foregroundImage: './assets/adaptive-icon.png',
        backgroundColor: '#ffffff',
      },
      package: 'com.yourcompany.app',
      permissions: [
        'android.permission.READ_EXTERNAL_STORAGE',
        'android.permission.WRITE_EXTERNAL_STORAGE',
        'android.permission.CAMERA',
      ],
    },
    web: {
      bundler: 'metro',
      output: 'static',
      favicon: './assets/favicon.png',
    },
    plugins: [
      [
        'expo-build-properties',
        {
          ios: {
            minSdkVersion: '13.0',
          },
          android: {
            minSdkVersion: 21,
            compileSdkVersion: 34,
            targetSdkVersion: 34,
          },
        },
      ],
    ],
    extra: {
      eas: {
        projectId: env.EAS_PROJECT_ID,
      },
      apiUrl: env.SERVER_API_URL,
      environment: env.ENVIRONMENT,
    },
  },
};
```

Create `.env.example`:

```bash
APP_ENV=development
DEVELOPMENT_API_URL=http://localhost:3000
STAGING_API_URL=https://staging-api.example.com
PRODUCTION_API_URL=https://api.example.com
EAS_PROJECT_ID=your-eas-project-id
```

Access in code:

```typescript
import Constants from 'expo-constants';

const apiUrl = Constants?.expoConfig?.extra?.apiUrl;
const projectId = Constants?.expoConfig?.extra?.eas?.projectId;
```

## Usage Example

```typescript
// In app.config.ts, EAS Build profiles
export default {
  expo: {
    // ... config
    extra: {
      eas: {
        projectId: process.env.EAS_PROJECT_ID,
      },
    },
  },
};

// In your app
import Constants from 'expo-constants';

const API_URL = Constants?.expoConfig?.extra?.apiUrl;

// Use in API calls
fetch(`${API_URL}/api/users`);
```

## Gotchas

- **Environment variables in app.config.ts:** Only environment variables set before the build are available. Use `.env` files or CI/CD secrets.
- **EAS_PROJECT_ID:** Required for Expo push notifications and EAS Build. Get it from `eas.json` or the Expo dashboard.
- **iOS bundle identifier:** Must be unique in the App Store. Use reverse domain naming (e.g., `com.company.appname`).
- **Android package name:** Must be unique on Google Play. Different from iOS bundle identifier but should be similar.
- **Build number vs version:** On iOS, increment `buildNumber` for each build; `version` follows semver. Android uses `versionCode` (integer) and `versionName` (semver).
