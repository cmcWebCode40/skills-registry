---
name: theming-hooks
category: styling
stack: [react, react-native, context, memoization]
keywords: [useTheme, useThemedStyles, Theme, createStyles, memoization, dark-mode]
source-files: [libs/hooks/useTheme.ts, libs/hooks/useThemedStyles.ts]
---

# Theming Hooks

## Problem
You need a consistent pattern to access the theme and produce `StyleSheet`-based styles that automatically update when the user switches color scheme.

## When to Use
- Creating theme-aware `StyleSheet.create` styles in any component
- Accessing `theme.colors.*`, `theme.spacing.*`, or `theme.typography.*` inside a component
- Avoiding hardcoded colors, spacing, or font strings in `StyleSheet` calls

## Implementation

### Code

`libs/hooks/useTheme.ts` — reads color scheme from context:

```typescript
import { useContext } from 'react';
import { ThemeContext } from '../context/ThemeContext';

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeContextProvider');
  }
  return context;
}
```

`libs/hooks/useThemedStyles.ts` — resolves the `Theme` object:

```typescript
import { useMemo } from 'react';
import { darkTheme, lightTheme, Theme } from '../constants/theme';
import { useTheme } from './useTheme';

export function useThemedStyles() {
  const { colorScheme } = useTheme();

  return useMemo(() => {
    const theme: Theme = colorScheme === 'dark' ? darkTheme : lightTheme;
    return { theme, isDark: colorScheme === 'dark' };
  }, [colorScheme]);
}
```

## Usage Pattern (mandatory in every component that uses StyleSheet)

```typescript
import { Theme } from '@/libs/constants/theme';
import { useThemedStyles } from '@/libs/hooks/useThemedStyles';
import { useMemo } from 'react';
import { StyleSheet, View } from 'react-native';
import { Paragraph } from '@/components/ui';

export function NotificationRow({ title, description }: { title: string; description: string }) {
  const { theme } = useThemedStyles();
  const styles = useMemo(() => createStyles(theme), [theme]);

  return (
    <View style={styles.row}>
      <Paragraph size="sm" font="inter-semibold">{title}</Paragraph>
      <Paragraph size="xs" className="text-secondary">{description}</Paragraph>
    </View>
  );
}

function createStyles(theme: Theme) {
  return StyleSheet.create({
    row: {
      paddingVertical: theme.spacing.md,
      borderBottomWidth: 1,
      borderBottomColor: theme.colors.outline_variant,
      backgroundColor: theme.colors.surface_container,
    },
  });
}
```

## Rules

- `createStyles` is always a hoisted `function` declaration placed **after** the component. Never a `const` at module level or before the component.
- Call `useThemedStyles()` once per component: `const { theme } = useThemedStyles()`.
- Wrap the call to `createStyles` in `useMemo`: `const styles = useMemo(() => createStyles(theme), [theme])`.
- Access `theme.colors.*` for color tokens, `theme.spacing.*` for spacing, `theme.typography.*` for font properties.
- Never pass raw colors, spacing integers, or font strings directly into `StyleSheet.create`. Everything comes from `theme`.

## Gotchas

- `lightTheme`/`darkTheme` are module-level constants — same object reference when scheme doesn't change, keeping `useMemo` stable.
- `useThemedStyles` depends on `ThemeContextProvider` being an ancestor. It will throw if used outside the provider.
- Dynamic Animated values (e.g. `interpolate` outputRange) should use `theme.colors.*` directly in the component body — they are recalculated on each render intentionally.
- Do NOT call `useThemedStyles` inside `createStyles`. It's a hook — it only works in the component body.
