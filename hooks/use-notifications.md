---
name: use-notifications
category: hooks
stack: [expo-notifications, react, react-native]
keywords: [push-notifications, permissions, deep-linking, listener, token]
source-files: [libs/hooks/useNotifications.ts]
---

# useNotifications

## Problem
You need a hook to handle push notification registration, listener setup, and deep linking to specific screens when notifications are tapped.

## When to Use
- Registering push tokens on app start
- Listening for incoming notifications
- Deep linking to relevant screens when users tap notifications
- Handling permission requests for notifications

## Implementation

### Dependencies
```json
{
  "expo-notifications": "^13.0.0",
  "expo-device": "^6.0.0",
  "expo-constants": "^16.0.0"
}
```

### Code

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import Constants from 'expo-constants';
import * as Device from 'expo-device';
import * as Notifications from 'expo-notifications';
import { AndroidNotificationPriority } from 'expo-notifications';
import { router } from 'expo-router';
import { colors } from 'libs/constants';
import serverApi from 'libs/services';
import { ApiResponsePayload } from 'libs/services/type';
import { useEffect, useRef, useState } from 'react';
import { Platform } from 'react-native';
import Toast from 'react-native-toast-message';

// Configure notification handler
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldPlaySound: false,
    shouldSetBadge: false,
    shouldShowBanner: true,
    shouldShowList: true,
    priority: AndroidNotificationPriority.HIGH,
  }),
});

export async function registerForPushNotificationsAsync() {
  // Android: Create notification channel
  if (Platform.OS === 'android') {
    try {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: colors.dark.primary[100],
      });
    } catch {
      return;
    }
  }

  // Check if device
  if (!Device.isDevice) {
    Toast.show({
      type: 'error',
      text1: 'Notifications unavailable on emulator',
    });
    return;
  }

  // Request permissions
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    Toast.show({
      type: 'error',
      text1: 'Permission Denied',
      text2: 'Enable push notifications in settings',
    });
    return;
  }

  // Get project ID
  const projectId = Constants?.expoConfig?.extra?.eas?.projectId ?? Constants?.easConfig?.projectId;

  if (!projectId) {
    console.error('EAS Project ID not found');
    return;
  }

  // Get push token
  try {
    const pushTokenString = (
      await Notifications.getExpoPushTokenAsync({
        projectId,
      })
    ).data;
    return pushTokenString;
  } catch (e: unknown) {
    console.error('Failed to get push token:', e);
    return;
  }
}

const useNotification = () => {
  const [pushToken, setPushToken] = useState<string | undefined>(undefined);
  const [notification, setNotification] = useState<Notifications.Notification | undefined>();
  const notificationListener = useRef<Notifications.EventSubscription>(null);
  const responseListener = useRef<Notifications.EventSubscription>(null);
  const queryClient = useQueryClient();

  // Register push token with backend
  const RegisterPushNotification = useMutation({
    mutationFn: (payload: { pushToken: string }) => {
      return serverApi.post(`/settings/notifications/push-token`, payload);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['profile'] });
    },
  });

  useEffect(() => {
    if (!Device.isDevice) return;

    let isMounted = true;

    // Register for push token
    registerForPushNotificationsAsync().then((token) => {
      if (isMounted && token) {
        setPushToken(token);
        RegisterPushNotification.mutate({ pushToken: token });
      }
    });

    // Listen for incoming notifications
    notificationListener.current = Notifications.addNotificationReceivedListener((notification) => {
      if (!isMounted) return;
      setNotification(notification);
      // Refresh relevant data
      queryClient.invalidateQueries({
        predicate: (query) => /^notification/i.test(query.queryKey[0] as string),
      });
    });

    // Handle notification tap (deep link)
    responseListener.current = Notifications.addNotificationResponseReceivedListener((data) => {
      if (!isMounted) return;

      const responsePayload = data.notification?.request?.content?.data;

      // Route based on notification type
      if (responsePayload?.type === 'chat_message') {
        router.replace(`/chats/${responsePayload.metadata?.conversationId}`);
        return;
      }
      if (responsePayload?.type === 'enrollment_status') {
        router.replace('/enrollments');
        return;
      }
      router.replace('/settings/notification');
    });

    return () => {
      isMounted = false;
      notificationListener.current?.remove();
      responseListener.current?.remove();
    };
  }, [queryClient]);

  return {
    pushToken,
    notification,
    RegisterPushNotification,
  };
};

export default useNotification;
```

## Usage Example

```typescript
import useNotification from 'libs/hooks/useNotifications';

export default function App() {
  const { pushToken, notification } = useNotification();

  useEffect(() => {
    console.log('Push token:', pushToken);
  }, [pushToken]);

  useEffect(() => {
    if (notification) {
      console.log('Notification received:', notification);
    }
  }, [notification]);

  return <View>{/* Your app */}</View>;
}
```

## Gotchas

- **EAS Project ID required**: Must be set in app.config.ts for push tokens to work.
- **Physical device only**: Emulators can't receive push notifications.
- **Permissions**: Always request permissions; don't assume they're granted.
- **Token registration**: Register token with backend immediately after obtaining it.
- **Deep linking**: Ensure routes exist before navigating via notification data.
- **Notification data structure**: Design a consistent schema for notification payload to handle all notification types.
