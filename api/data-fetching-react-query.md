---
name: data-fetching-react-query
category: api
stack: [react-query, react-native, axios]
keywords: [useQuery, useMutation, caching, server-state, error-handling]
source-files: [modules/auth/hooks/useAuth.ts]
---

# Data Fetching with React Query

## Problem
You need to manage server state (fetch, cache, refetch) with a pattern that handles loading, error, and success states while preventing duplicate requests.

## When to Use
- Fetching data from an API
- Mutating server state (create, update, delete)
- Handling pagination and infinite queries
- Implementing optimistic updates

## Implementation

### Dependencies
```json
{
  "@tanstack/react-query": "^5.0.0",
  "axios": "^1.6.0"
}
```

### Code

**Module Hook Pattern** (`modules/auth/hooks/useAuth.ts`):

```typescript
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import serverApi, { apiErrorHandler } from 'libs/services';
import { authQueryKeys } from 'modules/auth/constants';
import Toast from 'react-native-toast-message';

interface LoginPayload {
  email: string;
  password: string;
}

const useAuth = () => {
  const queryClient = useQueryClient();

  // Query: Fetch user profile
  const Profile = useQuery({
    queryKey: authQueryKeys.profile(),
    queryFn: () => serverApi.get('/auth/me'),
    enabled: !!queryClient.getQueryData(authQueryKeys.isAuthenticated()),
  });

  // Mutation: Login
  const Login = useMutation({
    mutationFn: (payload: LoginPayload) => {
      return serverApi.post('/auth/signin', payload);
    },
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: authQueryKeys.profile() });
      Toast.show({ type: 'success', text1: 'Logged in successfully' });
    },
    onError: (error) => {
      Toast.show({ type: 'error', text1: apiErrorHandler(error) });
    },
  });

  // Mutation: Logout
  const Logout = useMutation({
    mutationFn: () => serverApi.post('/auth/signout'),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: authQueryKeys.profile() });
    },
  });

  return { Profile, Login, Logout };
};

export default useAuth;
```

**Usage in Screen:**

```typescript
import useAuth from 'modules/auth/hooks/useAuth';

export default function ProfileScreen() {
  const { Profile, Logout } = useAuth();

  if (Profile.isPending) return <ActivitySpinner />;
  if (Profile.isError) return <ErrorComponent error={Profile.error} />;

  const user = Profile.data?.data?.data;

  return (
    <View>
      <Text>{user?.email}</Text>
      <Button
        isLoading={Logout.isPending}
        onPress={() => Logout.mutate()}
      >
        Logout
      </Button>
    </View>
  );
}
```

## Usage Example

**Query Key Factory:**

```typescript
export const authQueryKeys = {
  all: () => ['auth'] as const,
  profile: () => [...authQueryKeys.all(), 'profile'] as const,
  isAuthenticated: () => [...authQueryKeys.all(), 'isAuthenticated'] as const,
};
```

**Optimistic Update (with rollback):**

```typescript
const UpdateProfile = useMutation({
  mutationFn: (payload) => serverApi.patch('/profile', payload),
  onMutate: async (newData) => {
    // Cancel ongoing queries
    await queryClient.cancelQueries({ queryKey: authQueryKeys.profile() });
    
    // Save old data for rollback
    const previousData = queryClient.getQueryData(authQueryKeys.profile());
    
    // Update cache optimistically
    queryClient.setQueryData(authQueryKeys.profile(), (old) => ({
      ...old,
      data: { ...old?.data, ...newData },
    }));
    
    return { previousData }; // Return context for onError
  },
  onError: (error, newData, context) => {
    // Rollback on error
    if (context?.previousData) {
      queryClient.setQueryData(authQueryKeys.profile(), context.previousData);
    }
  },
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: authQueryKeys.profile() });
  },
});
```

## Gotchas

- **Mutation vs Query**: useQuery is for fetching; useMutation is for modifying. Don't fetch in mutations.
- **Cache invalidation**: Too broad invalidation refetches unrelated data. Use specific query keys.
- **enabled flag**: Set to false to prevent queries from running until a condition is met (e.g., auth token available).
- **Stale time**: Data fresh for this duration; after, it's refetched on access. Set appropriately per endpoint.
