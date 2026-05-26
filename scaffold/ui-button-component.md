---
name: ui-button-component
category: scaffold
stack: [react-native, nativewind, typescript]
keywords: [button, Pressable, variant, loading, Broadsheet, full-width, 52px]
source-files: [components/ui/button/Button.tsx]
---

# Button Component

## Problem
You need a reusable button that enforces the Broadsheet Editorial design system: full-width, 52px height, 0px border radius, with primary and outlined variants.

## When to Use
- Primary actions (full-width, solid fill)
- Secondary/ghost actions (full-width, outlined)
- Loading state during async operations

## Design Rules

- **Height**: 52px for primary, 44px for secondary/ghost
- **Width**: Full-width (`w-full`)
- **Border radius**: 0px — never use `rounded-*`
- **Primary**: `bg-primary` background, `text-on_primary` label
- **Outlined**: transparent background, `border border-outline_variant`, `text-secondary` label
- **Label**: `Paragraph` component with `font="inter-semibold"`, `size="sm"`, `uppercase tracking-widest`
- **No raw `Text`**: Button labels use `Paragraph` from `components/ui`, never the RN `Text` component

## Implementation

### Code

`components/ui/button/types.ts`:

```typescript
import { PressableProps } from 'react-native';

export type ButtonVariant = 'primary' | 'outlined' | 'ghost';

export interface ButtonProps extends PressableProps {
  variant?: ButtonVariant;
  isLoading?: boolean;
  children: React.ReactNode;
  className?: string;
}
```

`components/ui/button/Button.tsx`:

```typescript
import { Paragraph } from '@/components/ui/paragraph/Paragraph';
import { Theme } from '@/libs/constants/theme';
import { useThemedStyles } from '@/libs/hooks/useThemedStyles';
import React, { useMemo } from 'react';
import { ActivityIndicator, Pressable, StyleSheet } from 'react-native';
import { ButtonProps } from './types';

export function Button({
  variant = 'primary',
  isLoading = false,
  disabled,
  children,
  className = '',
  ...rest
}: ButtonProps) {
  const { theme } = useThemedStyles();
  const styles = useMemo(() => createStyles(theme), [theme]);

  const containerClass = {
    primary:  `h-[52px] w-full items-center justify-center bg-primary ${disabled ? 'opacity-50' : ''} ${className}`,
    outlined: `h-[44px] w-full items-center justify-center border border-outline_variant ${disabled ? 'opacity-50' : ''} ${className}`,
    ghost:    `h-[44px] w-full items-center justify-center ${disabled ? 'opacity-50' : ''} ${className}`,
  }[variant];

  const labelClass = {
    primary:  'uppercase tracking-widest text-on_primary',
    outlined: 'uppercase tracking-widest text-secondary',
    ghost:    'uppercase tracking-widest text-on_surface_variant',
  }[variant];

  return (
    <Pressable disabled={disabled || isLoading} className={containerClass} {...rest}>
      {isLoading ? (
        <ActivityIndicator color={variant === 'primary' ? theme.colors.on_primary : theme.colors.on_surface} />
      ) : (
        <Paragraph size="sm" font="inter-semibold" className={labelClass}>
          {children}
        </Paragraph>
      )}
    </Pressable>
  );
}

function createStyles(theme: Theme) {
  return StyleSheet.create({
    pressedOverlay: {
      opacity: 0.85,
    },
  });
}
```

`components/ui/index.ts`:

```typescript
export { Button } from './button/Button';
export type { ButtonProps } from './button/types';
```

## Usage Example

```typescript
import { Button } from '@/components/ui';

export function SetupFooter({ onNext, onSkip }: { onNext: () => void; onSkip: () => void }) {
  return (
    <View className="px-md pb-lg pt-md">
      <Button variant="primary" onPress={onNext}>
        NEXT →
      </Button>
      <Button variant="outlined" onPress={onSkip} className="mt-sm">
        SKIP FOR NOW
      </Button>
    </View>
  );
}
```

## Gotchas

- **No `rounded-*`**: The Broadsheet design system uses 0px border radius everywhere.
- **No raw `Text` inside Button**: The label is a `Paragraph` with `font="inter-semibold"`. Never use `<Text>` directly.
- **`disabled` vs `isLoading`**: Both disable the press event. `isLoading` also replaces the label with a spinner.
- **Color via className**: Button variants do not accept a `color` prop. Colors come from NativeWind tokens (`bg-primary`, `text-on_primary`).
- **`ActivityIndicator` color**: Must come from `theme.colors.*` via `useThemedStyles` — never hardcoded.
