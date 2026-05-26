---
name: key-storage-handler
category: scaffold
stack: [expo-secure-store, react-native-mmkv, react-native, typescript]
keywords: [secure-storage, tokens, MMKV, keychain, FAST_KEYS, fastStorage, devClearAllStorage]
source-files: [libs/utils/keyStorage.ts]
---

# Key Storage Handler

## Problem
You need a two-tier storage system: encrypted storage (`SecureStore`) for sensitive tokens, and fast synchronous MMKV storage for app state — with typed key constants and a dev utility for resetting storage during testing.

## When to Use
- Setting up auth token storage
- Persisting onboarding completion, user preferences, or feature flags
- Separating sensitive data (tokens) from non-sensitive data (state, preferences)

## Implementation

### Dependencies
```json
{
  "expo-secure-store": "^13.0.0",
  "react-native-mmkv": "^3.0.0"
}
```

### Code

Create `libs/utils/keyStorage.ts`:

```typescript
import * as SecureStore from 'expo-secure-store';
import { MMKV } from 'react-native-mmkv';

export const fastStorage = new MMKV({ id: 'fast' });
export const secureStorageInstance = new MMKV({ id: 'secure', encryptionKey: 'app-secure-key' });

export const FAST_KEYS = {
  ONBOARDING_COMPLETE: 'ONBOARDING_COMPLETE',
  DAY_END_TIME: 'DAY_END_TIME',
  MORNING_REMINDER_ENABLED: 'MORNING_REMINDER_ENABLED',
  MORNING_REMINDER_TIME: 'MORNING_REMINDER_TIME',
  USER_COLOR_PREFERENCE: 'USER_COLOR_PREFERENCE',
} as const;

export const SECURE_KEYS = {
  AUTH_TOKEN: 'AUTH_TOKEN',
  AUTH_SESSION: 'AUTH_SESSION',
} as const;

export const saveToSecureStore = async (key: string, value: string): Promise<void> => {
  await SecureStore.setItemAsync(key, value);
};

export const getFromSecureStore = async (key: string): Promise<string | null> => {
  return SecureStore.getItemAsync(key);
};

export const deleteFromSecureStore = async (key: string): Promise<void> => {
  await SecureStore.deleteItemAsync(key);
};

export const clearAllStorage = async (): Promise<void> => {
  await deleteFromSecureStore(SECURE_KEYS.AUTH_TOKEN);
  await deleteFromSecureStore(SECURE_KEYS.AUTH_SESSION);
  fastStorage.clearAll();
};

export const devClearAllStorage = (): void => {
  if (__DEV__) {
    fastStorage.clearAll();
  }
};
```

## Usage Example

**Read/write fast state (MMKV — synchronous):**

```typescript
import { FAST_KEYS, fastStorage } from '@/libs/utils/keyStorage';

fastStorage.set(FAST_KEYS.ONBOARDING_COMPLETE, true);

const isComplete = fastStorage.getBoolean(FAST_KEYS.ONBOARDING_COMPLETE);
```

**Read/write secure tokens (SecureStore — async):**

```typescript
import { SECURE_KEYS, saveToSecureStore, getFromSecureStore } from '@/libs/utils/keyStorage';

await saveToSecureStore(SECURE_KEYS.AUTH_TOKEN, accessToken);

const token = await getFromSecureStore(SECURE_KEYS.AUTH_TOKEN);
```

**Logout (clear everything):**

```typescript
import { clearAllStorage } from '@/libs/utils/keyStorage';

await clearAllStorage();
router.replace('/(auth)/login');
```

**Dev testing (reset onboarding):**

```typescript
import { devClearAllStorage } from '@/libs/utils/keyStorage';

devClearAllStorage();
```

## Gotchas

- **MMKV is synchronous**: No `await` needed for `fastStorage` operations. The old `saveToAsyncStore` naming is misleading — use `fastStorage.set()` directly.
- **MMKV is NOT encrypted by default**: Never store tokens in `fastStorage`. Use `SecureStore` for `AUTH_TOKEN` and `AUTH_SESSION`.
- **`devClearAllStorage` is `__DEV__` guarded**: Safe to call on startup during development. Does nothing in production.
- **`FAST_KEYS` as const**: Typed constants prevent typos and enable autocomplete. Add new keys here rather than using bare strings.
- **Platform differences**: `SecureStore` uses iOS Keychain and Android Keystore. Values are not visible in debuggers. Test on device.
