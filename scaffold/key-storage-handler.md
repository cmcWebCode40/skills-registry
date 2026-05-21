---
name: key-storage-handler
category: scaffold
stack: [expo-secure-store, react-native-mmkv, react-native, typescript]
keywords: [secure-storage, tokens, persistent-state, two-tier, keychain]
source-files: [libs/utils/keyStorage.ts]
---

# Key Storage Handler

## Problem
You need a two-tier storage system: encrypted storage (SecureStore) for sensitive tokens, and fast synchronous storage (MMKV) for app state, with unified helpers.

## When to Use
- Setting up secure token storage
- Persisting app state across app restarts
- Separating sensitive data (tokens) from non-sensitive data (theme, preferences)
- Providing a unified storage API throughout the app

## Implementation

### Dependencies
```json
{
  "expo-secure-store": "^13.0.0",
  "react-native-mmkv": "^2.0.0"
}
```

### Code

Create `libs/utils/keyStorage.ts`:

```typescript
import * as SecureStore from 'expo-secure-store';
import { MMKV } from 'react-native-mmkv';

// Storage keys
export const AUTH_TOKEN = 'AUTH_TOKEN';
export const AUTH_SESSION = 'AUTH_SESSION';
export const HAS_USER_ONBOARD = 'HAS_USER_ONBOARD';
export const PUSH_NOTIFICATION_REGISTRATION = 'PUSH_NOTIFICATION_REGISTRATION';
export const CHILD_ENROLLMENT_DRAFT = 'CHILD_ENROLLMENT_DRAFT';
export const PENDING_CHAT_MESSAGES = 'PENDING_CHAT_MESSAGES';
export const USER_COLOR_PREFERENCE = 'USER_COLOR_PREFERENCE';

// MMKV instance for fast sync storage
const mmkvStorage = new MMKV();

// SecureStore: encrypted, async, platform-specific (iOS Keychain, Android Keystore)
export const saveToSecureStore = async (key: string, value: string): Promise<void> => {
  try {
    await SecureStore.setItemAsync(key, value);
  } catch (error) {
    console.error(`Failed to save to SecureStore: ${key}`, error);
    throw error;
  }
};

export const getFromSecureStore = async (key: string): Promise<string | null> => {
  try {
    const value = await SecureStore.getItemAsync(key);
    return value || null;
  } catch (error) {
    console.error(`Failed to read from SecureStore: ${key}`, error);
    return null;
  }
};

export const deleteFromSecureStore = async (key: string): Promise<void> => {
  try {
    await SecureStore.deleteItemAsync(key);
  } catch (error) {
    console.error(`Failed to delete from SecureStore: ${key}`, error);
  }
};

// MMKV: fast, sync storage for app state (not encrypted)
export const saveToAsyncStore = async (key: string, value: string | boolean | number): Promise<void> => {
  try {
    if (typeof value === 'string') {
      mmkvStorage.set(key, value);
    } else if (typeof value === 'boolean') {
      mmkvStorage.set(key, value);
    } else if (typeof value === 'number') {
      mmkvStorage.set(key, value);
    }
  } catch (error) {
    console.error(`Failed to save to MMKV: ${key}`, error);
    throw error;
  }
};

export const getFromAsyncStore = async (key: string): Promise<string | boolean | number | null> => {
  try {
    const value = mmkvStorage.getString(key) || mmkvStorage.getBoolean(key) || mmkvStorage.getNumber(key);
    return value || null;
  } catch (error) {
    console.error(`Failed to read from MMKV: ${key}`, error);
    return null;
  }
};

export const deleteFromAsyncStore = async (key: string): Promise<void> => {
  try {
    mmkvStorage.delete(key);
  } catch (error) {
    console.error(`Failed to delete from MMKV: ${key}`, error);
  }
};

// Batch clear (logout)
export const clearAllStorage = async (): Promise<void> => {
  try {
    // Clear SecureStore
    await deleteFromSecureStore(AUTH_TOKEN);
    await deleteFromSecureStore(AUTH_SESSION);
    // Clear MMKV
    mmkvStorage.clearAll();
  } catch (error) {
    console.error('Failed to clear all storage', error);
  }
};
```

## Usage Example

**Saving tokens (SecureStore):**

```typescript
import { saveToSecureStore, AUTH_TOKEN, AUTH_SESSION } from 'libs/utils/keyStorage';

// After login
await saveToSecureStore(AUTH_TOKEN, newAccessToken);
await saveToSecureStore(AUTH_SESSION, JSON.stringify(sessionPayload));
```

**Reading tokens:**

```typescript
import { getFromSecureStore, AUTH_TOKEN } from 'libs/utils/keyStorage';

const token = await getFromSecureStore(AUTH_TOKEN); // null if not set
if (token) {
  // Token exists, continue
}
```

**Saving app state (MMKV):**

```typescript
import { saveToAsyncStore, USER_COLOR_PREFERENCE } from 'libs/utils/keyStorage';

await saveToAsyncStore(USER_COLOR_PREFERENCE, 'dark'); // string
await saveToAsyncStore(HAS_USER_ONBOARD, true); // boolean
```

**Logout:**

```typescript
import { clearAllStorage } from 'libs/utils/keyStorage';

const handleLogout = async () => {
  await clearAllStorage();
  router.replace('/(auth)/login');
};
```

## Gotchas

- **SecureStore is async**: Always await. Don't assume tokens are available immediately.
- **MMKV is sync but named async**: The function names are `saveToAsyncStore` for consistency, but MMKV is actually synchronous.
- **MMKV unencrypted**: Don't store tokens in MMKV. Use SecureStore for anything sensitive.
- **Batch clear**: `clearAllStorage()` removes all keys. Consider selective clearing if needed.
- **Platform differences**: iOS uses Keychain; Android uses Keystore. Both are platform-specific. Test on both.
- **Debugging**: SecureStore values are invisible in native debuggers. Don't expect to see them in Chrome DevTools.
