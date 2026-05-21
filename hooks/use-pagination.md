---
name: use-pagination
category: hooks
stack: [react-query, react, react-native]
keywords: [infinite-query, pagination, cursor, pagination-limit, fetch-more]
source-files: [libs/hooks/usePagination.ts]
---

# usePagination

## Problem
You need a hook that wraps React Query's useInfiniteQuery to simplify paginated API calls with automatic page flattening and refetch control.

## When to Use
- Fetching paginated data from an API
- Implementing infinite scroll or load-more functionality
- Flattening page data into a single array
- Managing pagination state (loading, refetching, fetching more)

## Implementation

### Dependencies
```json
{
  "@tanstack/react-query": "^5.0.0"
}
```

### Code

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';
import serverApi from 'libs/services';
import { ApiResponsePayload } from 'libs/services/type';
import { useState } from 'react';

type QueryFnResponse<TData = any[]> = {
  results: TData;
  next?: number;
};

type UsePaginationOptions = {
  queryKey: string[];
  apiUrl: string;
  defaultPage?: number;
  pageLimit?: number;
  enabled?: boolean;
  queryParams?: string;
  refetchInterval?: number | false;
  refetchIntervalInBackground?: boolean;
  transformResponse?: (response: any) => { count: number; results: any[] };
};

const usePagination = <TData>({
  apiUrl,
  queryKey,
  defaultPage = 1,
  pageLimit = 10,
  enabled = true,
  queryParams = '',
  refetchInterval,
  refetchIntervalInBackground,
  transformResponse,
}: UsePaginationOptions) => {
  const [totalCount, setTotalCount] = useState(0);
  const [isRefetching, setIsRefetching] = useState(false);
  const [isFetchingMoreData, setIsFetchingMoreData] = useState(false);

  const {
    data,
    error,
    hasNextPage,
    isFetching,
    fetchNextPage,
    isSuccess,
    refetch: onRefetch,
  } = useInfiniteQuery({
    queryKey: [...queryKey, queryParams],
    queryFn: async ({ pageParam = defaultPage }: { pageParam: number }) => {
      const endpoint = `${apiUrl}?limit=${pageLimit}&page=${pageParam}${queryParams}`;
      const response = await serverApi.get(endpoint);

      let responsePayload: { count: number; results: TData[] };

      if (transformResponse) {
        responsePayload = transformResponse(response);
      } else {
        const apiResponse = response as ApiResponsePayload<TData[]>;
        responsePayload = {
          count: apiResponse.data?.meta?.total ?? 0,
          results: apiResponse.data?.data ?? [],
        };
      }

      setTotalCount(responsePayload.count);
      return {
        results: responsePayload.results,
        next: responsePayload.count > pageParam * pageLimit ? pageParam + 1 : undefined,
      };
    },
    enabled: enabled,
    refetchInterval: refetchInterval,
    refetchIntervalInBackground: refetchIntervalInBackground,
    initialPageParam: defaultPage,
    getNextPageParam: (lastPage: QueryFnResponse) => lastPage.next,
  });

  const refetch = async () => {
    setIsRefetching(true);
    await onRefetch();
    setIsRefetching(false);
  };

  const fetchMore = async () => {
    if (!isFetching && hasNextPage) {
      setIsFetchingMoreData(true);
      await fetchNextPage();
      setIsFetchingMoreData(false);
    }
  };

  const flattenData = data?.pages
    ? data.pages.flatMap((page: QueryFnResponse<TData[]>) => page.results ?? [])
    : [];

  return {
    refetch,
    error,
    fetchMore,
    data: flattenData as TData[],
    isRefetching,
    totalCount,
    isSuccess,
    isLoading: isFetching && flattenData.length < 1,
    isFetchingMore: isFetchingMoreData,
    hasNextPage: !!hasNextPage,
  };
};

export default usePagination;
```

## Usage Example

```typescript
import usePagination from 'libs/hooks/usePagination';
import { FlatList, ActivityIndicator } from 'react-native';

const ChatListScreen = () => {
  const { data, fetchMore, isLoading, isFetchingMore } = usePagination({
    queryKey: ['chats'],
    apiUrl: '/chats',
    pageLimit: 20,
  });

  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <ChatCard chat={item} />}
      keyExtractor={(item) => item.id}
      onEndReached={() => fetchMore()}
      onEndReachedThreshold={0.5}
      ListEmptyComponent={isLoading ? <ActivitySpinner /> : <NoChatsFound />}
      ListFooterComponent={isFetchingMore ? <ActivitySpinner /> : null}
    />
  );
};
```

## Gotchas

- **onEndReachedThreshold**: Default 0.5 means fetch when user is 50% from end. Adjust for your UX.
- **totalCount**: Only accurate if API returns total count in meta. Some APIs may not.
- **transformResponse**: Use if your API response doesn't follow the standard { meta: { total }, data: [] } format.
- **Refetch interval**: Polls continuously if set. Useful for live data but can waste bandwidth.
- **queryParams**: Include filter/sort params here; don't append to apiUrl directly.
