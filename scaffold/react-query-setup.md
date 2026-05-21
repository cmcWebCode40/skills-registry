---
name: react-query-setup
category: scaffold
stack: [react-query, react-native, typescript]
keywords: [tanstack-query, server-state, caching, provider, querykey]
source-files: [libs/services/queryClient.ts, app/_layout.tsx]
---

# React Query Setup

## Problem
You need to configure TanStack React Query for server state management with proper cache configuration, providers, and query key factories.

## When to Use
- Setting up a new Expo project
- Adding server state management with automatic caching and refetching
- Creating query key factories for type-safe cache invalidation
- Configuring QueryClient with optimal default settings

## Implementation

### Dependencies
```json
{
  "@tanstack/react-query": "^5.0.0"
}
```

### Code

Create `libs/services/queryClient.ts`:

```typescript
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 10, // 10 minutes (was cacheTime)
      retry: 1,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
    mutations: {
      retry: 1,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});
```

**Root Layout** (`app/_layout.tsx`):

```typescript
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from 'libs/services/queryClient';

export default function RootLayout() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* Rest of your app */}
    </QueryClientProvider>
  );
}
```

**Query Key Factory Pattern** (`modules/profile/constants.ts`):

```typescript
export const profileQueryKeys = {
  all: () => ['profile'] as const,
  profile: () => [...profileQueryKeys.all(), 'profile'] as const,
  childProfiles: () => [...profileQueryKeys.all(), 'childProfiles'] as const,
  childProfile: (childId: string) => [...profileQueryKeys.childProfiles(), childId] as const,
};
```

**Module Hook** (`modules/profile/hooks/useProfile.ts`):

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import serverApi from 'libs/services';
import { profileQueryKeys } from 'modules/profile/constants';
import Toast from 'react-native-toast-message';
import { apiErrorHandler } from 'libs/services';

const useProfile = () => {
  const queryClient = useQueryClient();

  const Profile = useQuery({
    queryKey: profileQueryKeys.profile(),
    queryFn: () => serverApi.get('/users/profile'),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });

  const UpdateProfile = useMutation({
    mutationFn: (payload: ProfilePayload) => {
      return serverApi.patch('/users/profile', payload);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: profileQueryKeys.profile() });
      Toast.show({ type: 'success', text1: 'Profile updated' });
    },
    onError: (error) => {
      Toast.show({ type: 'error', text1: apiErrorHandler(error) });
    },
  });

  return { Profile, UpdateProfile };
};

export default useProfile;
```

## Usage Example

**In a screen:**

```typescript
import useProfile from 'modules/profile/hooks/useProfile';
import { ActivitySpinner, ErrorComponent } from 'components/ui';

export default function ProfileScreen() {
  const { Profile, UpdateProfile } = useProfile();

  if (Profile.isLoading) return <ActivitySpinner />;
  if (Profile.error) return <ErrorComponent error={Profile.error} />;

  const user = Profile.data?.data?.data;

  return (
    <View>
      <Text>{user?.name}</Text>
      <Button
        onPress={() =>
          UpdateProfile.mutate({
            name: 'New Name',
          })
        }
        isLoading={UpdateProfile.isPending}
      >
        Update Profile
      </Button>
    </View>
  );
}
```

## Gotchas

- **gcTime vs cacheTime**: React Query v5 renamed `cacheTime` to `gcTime` (garbage collection time). Unused queries are removed after this period.
- **staleTime**: Data is considered stale after this duration, triggering a refetch. Set it appropriately (too low = constant refetches, too high = stale data).
- **retry logic**: Mutation failures won't retry by default. useQuery retries once. Adjust based on your needs.
- **Query key immutability**: Use `as const` on query key factory returns for type safety and cache coherence.
- **Cache invalidation scope**: `invalidateQueries({ queryKey: ['profile'] })` also invalidates `['profile', 'childProfiles']`. Be specific to avoid over-invalidation.
