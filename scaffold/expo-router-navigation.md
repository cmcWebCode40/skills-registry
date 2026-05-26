---
name: expo-router-navigation
category: scaffold
stack: [expo, react-native, expo-router, typescript]
keywords: [routing, navigation, route-groups, tab-navigator, file-based, _layout, SplashScreen]
source-files: [app/_layout.tsx, app/(auth)/_layout.tsx, app/(tabs)/_layout.tsx]
---

# Expo Router Navigation

## Problem
You need file-based routing with route groups, authenticated routes, and role-based tab navigation — without putting navigation logic inside `_layout.tsx`.

## When to Use
- Setting up the navigation structure for a new Expo app
- Creating protected routes that require authentication
- Implementing role-based tab visibility
- Organizing routes by feature group (onboarding, auth, tabs)

## Implementation

### Dependencies
```json
{
  "expo-router": "^4.0.0",
  "expo": "^54.0.0",
  "react-native-safe-area-context": "^5.0.0",
  "react-native-gesture-handler": "^2.0.0"
}
```

### Code

**Root Layout** (`app/_layout.tsx`) — providers only, no routing logic:

```typescript
import '../global.css';

import { QueryClientProvider } from '@tanstack/react-query';
import { Stack } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import React, { useEffect } from 'react';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import Toast from 'react-native-toast-message';

import { queryClient } from '@/libs';
import { AppContextProvider } from '@/libs/context/AppContext';
import { AuthContextProvider } from '@/libs/context/AuthContext';
import { ThemeContextProvider } from '@/libs/context/ThemeContext';

SplashScreen.preventAutoHideAsync();

export const unstable_settings = {
  anchor: '(tabs)',
};

export default function RootLayout() {
  useEffect(() => {
    SplashScreen.hideAsync();
  }, []);

  return (
    <QueryClientProvider client={queryClient}>
      <AppContextProvider>
        <AuthContextProvider>
          <ThemeContextProvider>
            <GestureHandlerRootView className="flex-1">
              <SafeAreaProvider>
                <Stack screenOptions={{ headerShown: false }}>
                  <Stack.Screen name="(onboarding)" />
                  <Stack.Screen name="(auth)" />
                  <Stack.Screen name="(tabs)" />
                </Stack>
                <Toast />
              </SafeAreaProvider>
            </GestureHandlerRootView>
          </ThemeContextProvider>
        </AuthContextProvider>
      </AppContextProvider>
    </QueryClientProvider>
  );
}
```

**Onboarding Layout** (`app/(onboarding)/_layout.tsx`):

```typescript
import { Stack } from 'expo-router';

export default function OnboardingLayout() {
  return (
    <Stack screenOptions={{ headerShown: false, animation: 'fade' }} />
  );
}
```

**Onboarding index** (`app/(onboarding)/index.tsx`) — handles startup routing:

```typescript
import { FAST_KEYS, fastStorage } from '@/libs/utils/keyStorage';
import { router } from 'expo-router';
import { useEffect } from 'react';

export default function OnboardingGate() {
  useEffect(() => {
    const onboardingComplete = fastStorage.getBoolean(FAST_KEYS.ONBOARDING_COMPLETE);
    if (onboardingComplete) {
      router.replace('/(tabs)');
    } else {
      router.replace('/(onboarding)/intro');
    }
  }, []);

  return null;
}
```

**Auth Layout** (`app/(auth)/_layout.tsx`):

```typescript
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack screenOptions={{ headerShown: false, animation: 'fade' }} />
  );
}
```

**Tabs Layout** (`app/(tabs)/_layout.tsx`):

```typescript
import { Tabs } from 'expo-router';
import { useAuth } from '@/libs/context/AuthContext';
import { Icon } from '@/components/icons';

export default function TabsLayout() {
  return (
    <Tabs screenOptions={{ headerShown: false }}>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <Icon name="home-outline" size={24} color={color} />,
        }}
      />
    </Tabs>
  );
}
```

**Protected Route Guard** (`modules/auth/components/ProtectedRoute.tsx`):

```typescript
import { Redirect } from 'expo-router';
import { useAuth } from '@/libs/context/AuthContext';

export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { hasSession, isLoading } = useAuth();

  if (isLoading) return null;
  if (!hasSession) return <Redirect href="/(auth)/login" />;

  return <>{children}</>;
}
```

## File Structure

```
app/
  _layout.tsx               ← Providers only. No routing.
  (onboarding)/
    _layout.tsx
    index.tsx               ← Startup routing gate (checks ONBOARDING_COMPLETE)
    intro.tsx
    setup-day-end.tsx
    setup-reminder.tsx
    setup-notifications.tsx
  (auth)/
    _layout.tsx
    login.tsx
    signup.tsx
  (tabs)/
    _layout.tsx
    index.tsx               ← Home tab
    settings.tsx
```

## Gotchas

- **`_layout.tsx` is providers only**: Never add `router.replace`, `router.push`, or any `router` import to a `_layout.tsx`. Routing logic belongs in screen files or a gate screen.
- **`GestureHandlerRootView` uses `className`**: Use `className="flex-1"`, not `style={{ flex: 1 }}`.
- **Startup routing**: Check onboarding or auth status inside the first screen's `useEffect` or with a `<Redirect>` component — not in the root layout.
- **`SplashScreen.hideAsync()`**: Call in root layout's `useEffect`. With the `expo-font` config plugin, fonts are ready immediately — no need to delay.
- **`unstable_settings.anchor`**: Set to `'(tabs)'` so deep links resolve correctly.
- **Route groups** (parentheses) don't appear in URLs; they organize layouts only.
- **`animation: 'fade'`**: Use in group layouts to prevent flashing during auth redirects.
