---
name: api-services
category: api
stack: [axios, react-native, typescript]
keywords: [singleton, interceptor, auth, error-extraction, baseurl]
source-files: [libs/services/index.ts]
---

# API Services

## Problem
You need a centralized API layer with auth token injection, error extraction, and standardized response handling.

## When to Use
- Setting up API communication for a new project
- Adding auth headers to all requests
- Extracting error messages for user display
- Creating a single HTTP client for the entire app

## Implementation

### Code

```typescript
import axios, { AxiosInstance, isAxiosError } from 'axios';
import { ConfigKeys } from 'libs/config';
import { AUTH_TOKEN, getFromSecureStore } from 'libs/utils';

const serverApi: AxiosInstance = axios.create({
  baseURL: `${ConfigKeys.SERVER_API_URL}/api/v1`,
  headers: { Accept: 'application/json' },
});

// Request: Attach token
serverApi.interceptors.request.use(
  async (config) => {
    const token = await getFromSecureStore(AUTH_TOKEN);
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  }
);

// Error extraction
export const apiErrorHandler = (error: unknown): string => {
  if (isAxiosError(error) && error.response) {
    return error.response.data?.message || 'An error occurred';
  }
  return error instanceof Error ? error.message : 'Unknown error';
};

export default serverApi;
```

## Usage Example

```typescript
const { data } = await serverApi.get('/users/profile');
// Authorization header automatically added
```

## Gotchas

- **Circular dependency**: Avoid importing context in interceptors. Use sessionManager pattern instead.
- **Response interceptor**: Token refresh logic goes here (see token-refresh skill).
