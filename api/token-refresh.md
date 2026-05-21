---
name: token-refresh
category: api
stack: [axios, react-native, typescript, secure-store]
keywords: [401-refresh, queue-pattern, interceptor, silent-refresh, race-condition]
source-files: [libs/services/index.ts]
---

# Token Refresh

## Problem
You need to silently refresh auth tokens on 401 errors without concurrent refresh storms, while queuing failed requests until the token is available.

## When to Use
- Handling token expiration transparently to the user
- Preventing race conditions when multiple requests receive 401 simultaneously
- Retrying requests with a fresh token after refresh

## Implementation

### Code

```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';
import { ConfigKeys } from 'libs/config';
import { AUTH_SESSION, AUTH_TOKEN, getFromSecureStore, saveToSecureStore } from 'libs/utils';
import { LoginResponsePayload } from 'modules/auth/types';
import { forceLogout } from './sessionManager';

const serverApi = axios.create({
  baseURL: `${ConfigKeys.SERVER_API_URL}/api/v1`,
});

const AUTH_ROUTES = ['/auth/signin', '/auth/refresh-token'];
let isRefreshing = false;
type QueueEntry = { resolve: (token: string) => void; reject: (err: unknown) => void };
let refreshQueue: QueueEntry[] = [];

const flushQueue = (error: unknown, token: string | null = null) => {
  refreshQueue.forEach((cb) => (error ? cb.reject(error) : cb.resolve(token!)));
  refreshQueue = [];
};

serverApi.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const status = error.response?.status;
    const requestUrl = error.config?.url ?? '';
    const isAuthRoute = AUTH_ROUTES.some((route) => requestUrl.includes(route));
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    // 401: Silent refresh
    if (!isAuthRoute && status === 401) {
      if (isRefreshing) {
        // Queue request while refresh is in progress
        return new Promise((resolve, reject) => {
          refreshQueue.push({
            resolve: (token: string) => {
              originalRequest.headers['Authorization'] = `Bearer ${token}`;
              resolve(serverApi(originalRequest));
            },
            reject,
          });
        });
      }

      isRefreshing = true;
      try {
        const session = await getFromSecureStore(AUTH_SESSION);
        if (!session) throw new Error('No session');
        const { refreshToken } = JSON.parse(session) as LoginResponsePayload;
        const { data } = await axios.post(
          `${ConfigKeys.SERVER_API_URL}/api/v1/auth/refresh-token`,
          { refreshToken }
        );
        const newPayload: LoginResponsePayload = data.data;
        await saveToSecureStore(AUTH_TOKEN, newPayload.accessToken);
        await saveToSecureStore(AUTH_SESSION, JSON.stringify(newPayload));
        flushQueue(null, newPayload.accessToken);
        originalRequest.headers['Authorization'] = `Bearer ${newPayload.accessToken}`;
        return serverApi(originalRequest);
      } catch (refreshError) {
        flushQueue(refreshError);
        await forceLogout('Your session has expired. Please log in again.');
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    // 501: Server error modal (custom handling)
    if (status === 501) {
      // Trigger modal via sessionManager.callShowServerError()
      return Promise.reject(error);
    }

    // 502: Force logout
    if (!isAuthRoute && status === 502) {
      await forceLogout('The server is temporarily unavailable. Please try again later.');
    }

    return Promise.reject(error);
  }
);

export default serverApi;
```

## Usage Example

**Automatic refresh on 401:**

```typescript
// First request fails with 401
const response = await serverApi.get('/protected');
// Interceptor automatically refreshes token
// Request retried with new token
// User sees no error
```

## Gotchas

- **isRefreshing flag**: Critical to prevent parallel refresh requests. Don't remove.
- **Queue pattern**: Without this, multiple 401s trigger multiple refresh attempts. The queue ensures one refresh serves all.
- **Session validation**: Check that refresh token is valid. If it's expired too, force logout.
- **Auth route exclusion**: Don't refresh on /auth/signin, /auth/refresh-token itself to avoid loops.
- **Error handling**: If refresh fails, log out the user. Don't silently swallow the error.
