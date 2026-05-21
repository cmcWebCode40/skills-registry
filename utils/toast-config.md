---
name: toast-config
category: utils
stack: [react-native-toast-message, react-native, typescript]
keywords: [toast, notification, config, variants, styling]
source-files: [libs/utils/ToastConfig.tsx]
---

# Toast Config

## Problem
You need to configure toast notifications with custom variants (success, error, primary, info) that match your design system.

## When to Use
- Setting up the toast/notification system for your app
- Creating custom toast styles
- Displaying messages with consistent branding
- Supporting multiple toast types (success, error, warning, info)

## Implementation

### Dependencies
```json
{
  "react-native-toast-message": "^2.0.0"
}
```

### Code

Create `libs/utils/ToastConfig.tsx`:

```typescript
import { appTheme } from 'libs/constants';
import React from 'react';
import { StyleSheet, Text, TouchableOpacity, View } from 'react-native';
import { BaseToastProps, ToastConfigParams } from 'react-native-toast-message';
import { fontPixel, pixelSizeHorizontal, pixelSizeVertical } from './sizing';
import { CloseRounded } from 'components/icons';

export const DEFAULT_TOAST_DURATION = 6000;

export const toastConfig = {
  success: (props: ToastConfigParams<BaseToastProps>) => (
    <View style={[styles.container, styles.success]}>
      <View style={styles.content}>
        <Text style={[styles.typography, styles.successText]}>{props.text1}</Text>
      </View>
      <TouchableOpacity style={styles.closeIcon} onPress={() => props.hide()}>
        <CloseRounded color={appTheme.colors.text.secondary} size={24} />
      </TouchableOpacity>
    </View>
  ),
  error: (props: ToastConfigParams<BaseToastProps>) => (
    <View style={[styles.container, styles.error]}>
      <View style={styles.content}>
        <Text style={[styles.typography, styles.errorText]}>{props.text1}</Text>
      </View>
      <TouchableOpacity style={styles.closeIcon} onPress={() => props.hide()}>
        <CloseRounded color={appTheme.colors.text.secondary} size={24} />
      </TouchableOpacity>
    </View>
  ),
  primary: (props: ToastConfigParams<BaseToastProps>) => (
    <View style={[styles.container, styles.primary]}>
      <View style={styles.content}>
        <Text style={[styles.typography, styles.primaryText]}>{props.text1}</Text>
        {props.text2 && (
          <Text style={[styles.typographySecondary, styles.primaryText]}>
            {props.text2}
          </Text>
        )}
      </View>
      <TouchableOpacity style={styles.closeIcon} onPress={() => props.hide()}>
        <CloseRounded color={appTheme.colors.base.white} size={24} />
      </TouchableOpacity>
    </View>
  ),
  info: (props: ToastConfigParams<BaseToastProps>) => (
    <View style={[styles.container, styles.info]}>
      <View style={styles.content}>
        <Text style={[styles.typography, styles.infoText]}>{props.text1}</Text>
      </View>
      <TouchableOpacity style={styles.closeIcon} onPress={() => props.hide()}>
        <CloseRounded color={appTheme.colors.base.white} size={24} />
      </TouchableOpacity>
    </View>
  ),
};

const styles = StyleSheet.create({
  container: {
    borderRadius: 10,
    flexDirection: 'row',
    elevation: 2,
    justifyContent: 'space-between',
    paddingVertical: pixelSizeVertical(12),
    paddingHorizontal: pixelSizeHorizontal(12),
  },
  content: {
    flexBasis: '80%',
  },
  typography: {
    fontWeight: '500',
    fontSize: fontPixel(14),
    marginBottom: pixelSizeVertical(4),
  },
  typographySecondary: {
    fontWeight: '400',
    fontSize: fontPixel(12),
  },
  success: {
    backgroundColor: appTheme.colors.success[50],
  },
  successText: {
    color: appTheme.colors.success[900],
  },
  error: {
    backgroundColor: appTheme.colors.error[50],
  },
  errorText: {
    color: appTheme.colors.error[500],
  },
  primary: {
    backgroundColor: appTheme.colors.primary[500],
  },
  primaryText: {
    color: appTheme.colors.base.white,
  },
  info: {
    backgroundColor: appTheme.colors.primary[100],
  },
  infoText: {
    color: appTheme.colors.primary[500],
  },
  closeIcon: {
    paddingTop: 0,
  },
});
```

**Setup in Root Layout** (`app/_layout.tsx`):

```typescript
import Toast from 'react-native-toast-message';
import { toastConfig } from 'libs/utils/ToastConfig';

export default function RootLayout() {
  return (
    <>
      {/* Your app layout */}
      <Toast config={toastConfig} />
    </>
  );
}
```

## Usage Example

```typescript
import Toast from 'react-native-toast-message';

// Success
Toast.show({
  type: 'success',
  text1: 'Profile updated successfully',
  duration: 4000,
});

// Error
Toast.show({
  type: 'error',
  text1: 'Failed to update profile',
});

// Primary (with description)
Toast.show({
  type: 'primary',
  text1: 'Info',
  text2: 'This is additional information',
});

// Info
Toast.show({
  type: 'info',
  text1: 'Please wait...',
});
```

## Gotchas

- **Toast position**: By default at top. Configure via `Toast.show({ position: 'bottom' })`.
- **Duration**: Default is 3000ms. Use `duration` prop to override.
- **Multiple toasts**: Only one toast visible at a time by default. Configure via `topOffset`, `bottomOffset` if needed.
- **Text2**: Optional secondary text for detailed messages. Not available on all variants.
- **Auto-hide**: Toasts auto-hide after duration. Close button allows manual dismissal.
