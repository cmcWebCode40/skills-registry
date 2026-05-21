---
name: session-management
category: api
stack: [react-native, typescript]
keywords: [callbacks, register-unregister, force-logout, session-update, decoupled]
source-files: [libs/services/sessionManager.ts]
---

# Session Management

## Problem
You need to decouple session state changes (token updates, forced logout) from the Axios layer and auth context, allowing them to communicate without circular dependencies.

## When to Use
- Triggering auth context updates from Axios interceptors
- Forcing logout from API errors
- Showing server error modals from the HTTP layer
- Avoiding circular imports between services and context

## Implementation

### Code

```typescript
import { router } from 'expo-router';
import Toast from 'react-native-toast-message';
import { LoginResponsePayload } from 'modules/auth/types';

type ClearUserFn = () => Promise<void>;
type UpdatedSessionFn = (payload: LoginResponsePayload) => void;
type ShowServerErrorFn = () => void;

let clearUserRef: ClearUserFn | null = null;
let updatedSessionRef: UpdatedSessionFn | null = null;
let showServerErrorRef: ShowServerErrorFn | null = null;
let isLoggingOut = false;

// Register callbacks (called in AuthContext)
export const registerClearUser = (fn: ClearUserFn) => {
  clearUserRef = fn;
};

export const registerUpdatedSession = (fn: UpdatedSessionFn) => {
  updatedSessionRef = fn;
};

export const registerShowServerError = (fn: ShowServerErrorFn) => {
  showServerErrorRef = fn;
};

// Unregister (cleanup on unmount)
export const unregisterClearUser = () => {
  clearUserRef = null;
};

export const unregisterUpdatedSession = () => {
  updatedSessionRef = null;
};

export const unregisterShowServerError = () => {
  showServerErrorRef = null;
};

// Call from Axios (token refresh success)
export const callUpdatedSession = (payload: LoginResponsePayload) => {
  updatedSessionRef?.(payload);
};

// Call from Axios (server error 501)
export const callShowServerError = () => {
  showServerErrorRef?.();
};

// Call from Axios (force logout on 401 failure or 502)
export const forceLogout = async (reason: string) => {
  if (isLoggingOut) return; // Prevent concurrent logouts
  isLoggingOut = true;

  try {
    if (clearUserRef) {
      await clearUserRef();
    }
    Toast.show({
      type: 'error',
      text1: 'Session Expired',
      text2: reason,
    });
    router.replace('/(auth)/login');
  } finally {
    isLoggingOut = false;
  }
};
```

**Register in AuthContext:**

```typescript
useEffect(() => {
  registerClearUser(clearUser);
  registerUpdatedSession(updatedSession);
  registerShowServerError(showServerError);

  return () => {
    unregisterClearUser();
    unregisterUpdatedSession();
    unregisterShowServerError();
  };
}, []);
```

## Usage Example

**In Axios interceptor:**

```typescript
import { callUpdatedSession, forceLogout } from 'libs/services/sessionManager';

// Token refresh success
const newPayload = await fetchNewToken();
callUpdatedSession(newPayload); // Calls AuthContext.updatedSession()

// Token refresh failure
catch (error) {
  await forceLogout('Session expired'); // Calls AuthContext.clearUser()
}
```

## Gotchas

- **Circular imports**: This pattern avoids them by using callbacks instead of imports.
- **Cleanup**: Unregister on component unmount to prevent stale references.
- **Concurrent logouts**: The `isLoggingOut` flag prevents multiple concurrent logout calls.
- **Ref vs state**: Uses refs (`clearUserRef`) not state, as this runs outside React.
