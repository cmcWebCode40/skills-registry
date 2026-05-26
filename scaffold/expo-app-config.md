---
name: expo-app-config
category: scaffold
stack: [expo, react-native, typescript, eas-build]
keywords: [app.config.ts, EAS, environment, build, expo-font, fonts, config-plugin]
source-files: [app.config.ts, .env.example]
---

# Expo App Config

## Problem
Configure your Expo app with environment-specific settings, EAS Build profiles, font bundling via config plugin, and dynamic environment variables — without hardcoding secrets or using `useFonts`.

## When to Use
- Setting up a new Expo project
- Adding fonts via the `expo-font` config plugin (recommended over `useFonts`)
- Configuring environment-specific API URLs
- Setting up EAS Build for iOS and Android

## Implementation

### Dependencies
```json
{
  "expo": "^54.0.0",
  "expo-font": "^13.0.0",
  "expo-build-properties": "^0.13.0",
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
    icon: './assets/images/icon.png',
    userInterfaceStyle: 'automatic',
    splash: {
      image: './assets/images/splash.png',
      resizeMode: 'contain',
      backgroundColor: '#F5F0E8',
    },
    ios: {
      supportsTabletMode: false,
      bundleIdentifier: 'com.yourcompany.app',
      buildNumber: '1',
      infoPlist: {
        NSPhotoLibraryUsageDescription: 'App needs access to your photo library.',
        NSCameraUsageDescription: 'App needs access to your camera.',
        UIStatusBarStyle: 'dark',
      },
    },
    android: {
      adaptiveIcon: {
        foregroundImage: './assets/images/android-icon-foreground.png',
        backgroundColor: '#F5F0E8',
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
      favicon: './assets/images/favicon.png',
    },
    plugins: [
      [
        'expo-font',
        {
          fonts: [
            './assets/fonts/Playfair_Display/static/PlayfairDisplay-Bold.ttf',
            './assets/fonts/Inter/static/Inter_18pt-Regular.ttf',
            './assets/fonts/Inter/static/Inter_18pt-SemiBold.ttf',
            './assets/fonts/Inter/static/Inter_18pt-Bold.ttf',
            './assets/fonts/Roboto_Mono/static/RobotoMono-Medium.ttf',
          ],
          android: {
            fonts: [
              {
                fontFamily: 'PlayfairDisplay-Bold',
                fontDefinitions: [
                  { path: './assets/fonts/Playfair_Display/static/PlayfairDisplay-Bold.ttf', weight: 700 },
                ],
              },
              {
                fontFamily: 'Inter18pt-Regular',
                fontDefinitions: [
                  { path: './assets/fonts/Inter/static/Inter_18pt-Regular.ttf', weight: 400 },
                ],
              },
              {
                fontFamily: 'Inter18pt-SemiBold',
                fontDefinitions: [
                  { path: './assets/fonts/Inter/static/Inter_18pt-SemiBold.ttf', weight: 600 },
                ],
              },
              {
                fontFamily: 'Inter18pt-Bold',
                fontDefinitions: [
                  { path: './assets/fonts/Inter/static/Inter_18pt-Bold.ttf', weight: 700 },
                ],
              },
              {
                fontFamily: 'RobotoMono-Medium',
                fontDefinitions: [
                  { path: './assets/fonts/Roboto_Mono/static/RobotoMono-Medium.ttf', weight: 500 },
                ],
              },
            ],
          },
        },
      ],
      [
        'expo-build-properties',
        {
          ios: { minSdkVersion: '13.0' },
          android: {
            minSdkVersion: 21,
            compileSdkVersion: 34,
            targetSdkVersion: 34,
          },
        },
      ],
    ],
    extra: {
      eas: { projectId: env.EAS_PROJECT_ID },
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

## Usage Example

Access environment values in code:

```typescript
import Constants from 'expo-constants';

const apiUrl = Constants?.expoConfig?.extra?.apiUrl;
```

## Gotchas

- **No `useFonts`**: The `expo-font` config plugin bundles fonts into the native binary at build time. Do not use `useFonts` — fonts are available at startup without any JS-side loading or splash screen delay.
- **iOS font names**: On iOS, the font family name used in styles must match the font's PostScript name (e.g. `Inter18pt-Regular`, not `Inter-Regular`). The `fonts` array in the plugin handles iOS registration.
- **Android font names**: The `android.fonts` section with `fontDefinitions` overrides the default filename-based naming, giving Android the same `fontFamily` names as iOS. Both platforms must resolve to the same string.
- **Rebuild required**: After adding or changing fonts in `app.config.ts`, you must run a new native build (`npx expo run:ios` / `npx expo run:android`). Metro reload is not enough.
- **EAS_PROJECT_ID**: Required for Expo push notifications. Get it from `eas.json` or the Expo dashboard after running `eas build:configure`.
