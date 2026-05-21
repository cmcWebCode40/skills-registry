---
name: axios-setup
category: scaffold
stack: [axios, react-native, typescript]
keywords: [http-client, interceptor, authentication, error-handling, singleton]
source-files: [libs/services/index.ts]
---

# Axios Setup

## Problem
You need a singleton Axios instance with auth interceptors, error handling, and a consistent API response format across your app.

## When to Use
- Initializing API layer for a new project
- Adding request interceptors for auth headers
- Standardizing API error responses
- Setting up a base URL and headers for all requests

## Implementation

### Dependencies
```json
{
  "axios": "^1.6.0"
}
```

### Code

Create `libs/services/index.ts`:

```typescript
import axios, { AxiosError, isAxiosError, AxiosInstance } from 'axios';
import { ConfigKeys } from 'libs/config';
import { AUTH_TOKEN, getFromSecureStore, saveToSecureStore } from 'libs/utils';

export const ApiErrorCodes = {
  '401': 'unauthorized',
  '409': 'Duplicate Transaction',
  '500': 'Server Error',
};

const headers = {
  Accept: 'application/json',
};

// Singleton instance
const serverApi: AxiosInstance = axios.create({
  baseURL: `${ConfigKeys.SERVER_API_URL}/api/v1`,
  headers,
});

// Error extraction
export const apiErrorHandler = (error: unknown) => {
  if (isAxiosError(error)) {
    if (error.response) {
      return error?.response?.data?.message || 'An error occurred';
    } else {
      return error?.message;
    }
  }
  if (error instanceof Error) {
    return error.message;
  }
  return 'Unknown error';
};

export const getServerErrorCode = (error: unknown) => {
  if (isAxiosError(error)) {
    if (error.response) {
      const rawCode = error?.response?.data?.statusCode ?? error?.response?.status;
      const customerErrorCode = error?.response?.data?.errCode;
      const errorCode = rawCode != null ? String(rawCode) : '';

      if (errorCode) {
        const message = Object.prototype.hasOwnProperty.call(ApiErrorCodes, errorCode)
          ? ApiErrorCodes[errorCode as keyof typeof ApiErrorCodes]
          : 'unknown';
        return { code: errorCode, message, customerErrorCode };
      }
    } else {
      return { code: error.code, message: 'unknown' };
    }
  }
};

// Request interceptor: attach auth token
serverApi.interceptors.request.use(
  async (config) => {
    const authToken = await getFromSecureStore(AUTH_TOKEN);
    if (authToken) {
      config.headers.Authorization = `Bearer ${authToken}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

export default serverApi;
```

**Usage in components/hooks:**

```typescript
import serverApi, { apiErrorHandler } from 'libs/services';
import { useMutation } from '@tanstack/react-query';
import Toast from 'react-native-toast-message';

const useLogin = () => {
  return useMutation({
    mutationFn: (payload: LoginPayload) => {
      return serverApi.post('/auth/signin', payload);
    },
    onError: (error) => {
      Toast.show({
        type: 'error',
        text1: apiErrorHandler(error),
      });
    },
  });
};
```

## Usage Example

**GET request:**

```typescript
// Will automatically include Authorization header
const { data } = await serverApi.get('/users/profile');
```

**POST request with error handling:**

```typescript
const { Login } = useLogin();

await Login.mutateAsync(
  { email, password },
  {
    onSuccess: (response) => {
      console.log('Logged in:', response.data);
    },
    onError: (error) => {
      const errorMsg = apiErrorHandler(error);
      Toast.show({ type: 'error', text1: errorMsg });
    },
  }
);
```

## Gotchas

- **Token availability**: Auth token is fetched from SecureStore on every request. If the store is empty (fresh install), requests without auth will fail.
- **Base URL**: Ensure `ConfigKeys.SERVER_API_URL` is set in your environment. Missing it will cause network errors.
- **Response shape**: This assumes responses follow the pattern `{ statusCode, message, data, meta, timestamp }`. Adjust `apiErrorHandler` if your API differs.
- **Error interceptor order**: Token refresh logic (not shown here, see token-refresh skill) comes after the request interceptor.
- **Timeout**: Axios has a default timeout of 0 (no limit). Consider adding `timeout: 10000` (10s) to the create config.
