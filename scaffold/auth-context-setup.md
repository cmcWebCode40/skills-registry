---
name: auth-context-setup
category: scaffold
stack: [react, context-api, typescript, secure-store]
keywords: [authentication, session, context, role-based, token]
source-files: [libs/context/AuthContext.tsx, libs/services/sessionManager.ts]
---

# Auth Context Setup

## Problem
You need a centralized auth state with session management, role-based routing, and automatic token bootstrap from secure storage.

## When to Use
- Setting up authentication for a new project
- Implementing session persistence across app restarts
- Creating role-based route protection
- Managing logout and token refresh callbacks

## Implementation

### Dependencies
```json
{
  "react": "^18.0.0",
  "react-native": "^0.81.0",
  "expo-secure-store": "^13.0.0"
}
```

### Code

Create `libs/context/AuthContext.tsx`:

```typescript
import React, { createContext, useContext, useEffect, useState } from 'react';
import { AUTH_SESSION, AUTH_TOKEN, getFromSecureStore, deleteFromSecureStore, saveToSecureStore } from 'libs/utils';
import { LoginResponsePayload } from 'modules/auth/types';
import { router } from 'expo-router';
import { registerClearUser, registerUpdatedSession } from 'libs/services/sessionManager';

interface AuthContextType {
  hasSession: boolean;
  isLoading: boolean;
  user: (LoginResponsePayload & { roles: string[] }) | null;
  clearUser: () => Promise<void>;
  updatedSession: (payload: LoginResponsePayload) => void;
  changeUpdateStatus: (status: boolean) => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthContextProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [hasSession, setHasSession] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [user, setUser] = useState<AuthContextType['user']>(null);

  useEffect(() => {
    bootstrapAsync();
  }, []);

  const bootstrapAsync = async () => {
    try {
      const session = await getFromSecureStore(AUTH_SESSION);
      if (session) {
        const parsedSession = JSON.parse(session) as LoginResponsePayload;
        setUser({
          ...parsedSession,
          roles: parsedSession.roles || [],
        });
        setHasSession(true);
        router.replace('/(tabs)');
      } else {
        setHasSession(false);
        router.replace('/(auth)/welcome');
      }
    } catch (error) {
      setHasSession(false);
      router.replace('/(auth)/welcome');
    } finally {
      setIsLoading(false);
    }
  };

  const clearUser = async () => {
    await deleteFromSecureStore(AUTH_TOKEN);
    await deleteFromSecureStore(AUTH_SESSION);
    setHasSession(false);
    setUser(null);
    router.replace('/(auth)/login');
  };

  const updatedSession = (payload: LoginResponsePayload) => {
    setUser({
      ...payload,
      roles: payload.roles || [],
    });
    setHasSession(true);
  };

  const changeUpdateStatus = (status: boolean) => {
    setIsLoading(status);
  };

  // Register session manager callbacks
  useEffect(() => {
    registerClearUser(clearUser);
    registerUpdatedSession(updatedSession);
  }, []);

  return (
    <AuthContext.Provider
      value={{
        hasSession,
        isLoading,
        user,
        clearUser,
        updatedSession,
        changeUpdateStatus,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthContextProvider');
  }
  return context;
};

export default AuthContext;
```

**Protected Route Wrapper:**

```typescript
import { useAuth } from 'libs/context';
import { Redirect } from 'expo-router';
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

## Usage Example

**In a screen:**

```typescript
import { useAuth } from 'libs/context';

export default function SettingsScreen() {
  const { user, clearUser } = useAuth();

  return (
    <View>
      <Text>Welcome, {user?.email}</Text>
      <Button
        onPress={() => clearUser()}
        variant="outlined-danger"
      >
        Logout
      </Button>
    </View>
  );
}
```

**Role-based rendering:**

```typescript
const { user } = useAuth();
const isParent = user?.roles?.includes('parent');

{isParent && <ParentOnlyFeature />}
```

## Gotchas

- **Token refresh integration**: Session update must be called by the Axios interceptor (see token-refresh skill). Register via `registerUpdatedSession()`.
- **Role extraction**: Roles come from the login response. Ensure your API returns `roles: string[]` in the LoginResponsePayload.
- **Circular navigation**: If you redirect from AuthContext on mount (e.g., to /welcome), ensure it doesn't conflict with conditional rendering in Root Layout.
- **Logout flow**: clearUser is called by sessionManager on force logout (401, 502 errors). Don't call it manually except for explicit user logout.
