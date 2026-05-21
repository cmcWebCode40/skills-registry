---
name: expo-router-navigation
category: scaffold
stack: [expo, react-native, expo-router, typescript]
keywords: [routing, navigation, route-groups, tab-navigator, file-based]
source-files: [app/_layout.tsx, app/(auth)/_layout.tsx, app/(tabs)/_layout.tsx]
---

# Expo Router Navigation

## Problem
You need to implement file-based routing with route groups, authenticated routes, and role-based tab navigation without manual route configuration.

## When to Use
- Setting up navigation structure for a new Expo app
- Creating protected routes that require authentication
- Implementing role-based tab visibility
- Organizing routes by feature groups (auth, tabs, etc.)

## Implementation

### Dependencies
```json
{
  "expo-router": "^6.0.0",
  "expo": "^54.0.0"
}
```

### Code

**Root Layout** (`app/_layout.tsx`):

```typescript
import React, { useEffect } from 'react';
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { QueryClientProvider } from '@tanstack/react-query';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import Toast from 'react-native-toast-message';
import ErrorBoundaryWrapper from 'modules/error-boundary';
import { queryClient } from 'libs/services/queryClient';
import { AppContextProvider } from 'libs/context/AppContext';
import { AuthContextProvider } from 'libs/context/AuthContext';
import { ThemeContextProvider } from 'libs/context/ThemeContext';
import { SheetProvider } from 'react-native-actions-sheet';
import { toastConfig } from 'libs/utils/ToastConfig';

export default function RootLayout() {
  return (
    <ErrorBoundaryWrapper>
      <QueryClientProvider client={queryClient}>
        <AppContextProvider>
          <AuthContextProvider>
            <ThemeContextProvider>
              <GestureHandlerRootView style={{ flex: 1 }}>
                <SafeAreaProvider>
                  <SheetProvider>
                    <Stack
                      screenOptions={{
                        headerShown: false,
                      }}
                    />
                    <Toast config={toastConfig} />
                  </SheetProvider>
                </SafeAreaProvider>
              </GestureHandlerRootView>
            </ThemeContextProvider>
          </AuthContextProvider>
        </AppContextProvider>
      </QueryClientProvider>
    </ErrorBoundaryWrapper>
  );
}
```

**Auth Routes** (`app/(auth)/_layout.tsx`):

```typescript
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack
      screenOptions={{
        headerShown: false,
        animationEnabled: false,
      }}
    />
  );
}
```

**Tabs Layout** (`app/(tabs)/_layout.tsx`):

```typescript
import { BottomTabNavigationOptions } from '@react-navigation/bottom-tabs';
import { Tabs } from 'expo-router';
import { useAuth } from 'libs/context';
import { HomeIcon, ReportsIcon, ChatsIcon, SettingsIcon } from 'components/icons';

const allTabs = [
  {
    name: 'index',
    title: 'Home',
    icon: HomeIcon,
    roles: ['parent', 'caregiver'],
  },
  {
    name: 'reports',
    title: 'Reports',
    icon: ReportsIcon,
    roles: ['parent', 'caregiver'],
  },
  {
    name: 'chats',
    title: 'Chats',
    icon: ChatsIcon,
    roles: ['parent', 'caregiver'],
  },
  {
    name: 'settings',
    title: 'Settings',
    icon: SettingsIcon,
    roles: ['parent', 'caregiver'],
  },
];

export default function TabsLayout() {
  const { user } = useAuth();

  const canAccessTab = (tabRoles: string[]) => {
    if (!user?.roles) return false;
    return user.roles.some((role) => tabRoles.includes(role));
  };

  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarActiveTintColor: '#3B2513',
      }}
    >
      {allTabs.map((tab) => {
        const Icon = tab.icon;
        const screenOptions: BottomTabNavigationOptions = {
          title: tab.title,
          tabBarIcon: ({ color }) => <Icon size={24} color={color} />,
        };

        // Hide tab if user doesn't have the required role
        if (!canAccessTab(tab.roles)) {
          screenOptions.href = null;
        }

        return (
          <Tabs.Screen
            key={tab.name}
            name={tab.name}
            options={screenOptions}
          />
        );
      })}
    </Tabs>
  );
}
```

**Protected Route Wrapper** (`modules/auth/private/ProtectedRoute.tsx`):

```typescript
import { Redirect } from 'expo-router';
import { useAuth } from 'libs/context';
import { ActivitySpinner } from 'components/ui';

export const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { hasSession, isLoading } = useAuth();

  if (isLoading) {
    return <ActivitySpinner />;
  }

  if (!hasSession) {
    return <Redirect href="/(auth)/login" />;
  }

  return <>{children}</>;
};
```

**Usage** (`app/(tabs)/_layout.tsx` wrapper):

```typescript
export default function TabsLayout() {
  return (
    <ProtectedRoute>
      <Tabs>{/* ... tabs content ... */}</Tabs>
    </ProtectedRoute>
  );
}
```

## Usage Example

File structure that routes automatically:

```
app/
  _layout.tsx          ← Root provider wrapper
  (auth)/
    _layout.tsx
    login.tsx          → /login
    signup.tsx         → /signup
    otp.tsx            → /otp
  (tabs)/
    _layout.tsx
    index.tsx          → / (home)
    reports.tsx        → /reports
    chats/
      [chatId].tsx     → /chats/:chatId
    settings.tsx       → /settings
```

## Gotchas

- **Route groups** (parentheses) don't appear in URLs; they're for organizing layouts
- **Dynamic segments** (`[id]`) extract params via `useLocalSearchParams()`
- **href: null** hides a tab; it's still rendered but not touchable
- **Navigation.replace()** used in auth flows to prevent back-stack accumulation
- **Animations disabled** in auth group to prevent flashing during redirects
- **Provider order** matters: ensure QueryClient, AuthContext, ThemeContext are outside Tabs/Stack
