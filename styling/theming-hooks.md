---
name: theming-hooks
category: styling
stack: [react, react-native, context, memoization]
keywords: [useTheme, useThemedStyles, memoization, dynamic-styles]
source-files: [libs/hooks/useTheme.ts, libs/hooks/useThemedStyles.ts]
---

# Theming Hooks

## Problem
You need efficient hooks to access theme and create theme-dependent styles without re-rendering on every theme change.

## When to Use
- Accessing color tokens in components
- Creating dynamic styles based on theme
- Memoizing styles to prevent unnecessary recalculations
- Implementing dark mode support

## Implementation

### Code

**useTheme hook:**

```typescript
import { useContext } from 'react';
import { ThemeContext } from 'libs/context';

const useTheme = () => {
  return useContext(ThemeContext);
};

export default useTheme;
```

**useThemedStyles hook:**

```typescript
import { useMemo } from 'react';
import useTheme from './useTheme';
import { Theme } from 'libs/constants/theme';

type Styles<T extends Record<string, unknown>> = (theme: Theme) => T;

const useThemedStyles = <T extends Record<string, unknown>>(
  styles: Styles<T>
) => {
  const { theme } = useTheme();
  return useMemo(() => styles(theme), [styles, theme]);
};

export default useThemedStyles;
```

## Usage Example

**With useTheme:**

```typescript
import { useTheme } from 'libs/hooks';
import { View, Text } from 'react-native';

export default function MyComponent() {
  const { theme, mode, toggleColorScheme } = useTheme();

  return (
    <View style={{ backgroundColor: theme.colors.background }}>
      <Text style={{ color: theme.colors.text.primary }}>
        Current mode: {mode}
      </Text>
      <Button onPress={toggleColorScheme}>
        Toggle {mode === 'light' ? '🌙' : '☀️'}
      </Button>
    </View>
  );
}
```

**With useThemedStyles (memoized):**

```typescript
import useThemedStyles from 'libs/hooks/useThemedStyles';
import { StyleSheet } from 'react-native';

const createStyles = (theme) =>
  StyleSheet.create({
    container: {
      flex: 1,
      backgroundColor: theme.colors.background,
      padding: theme.spacing.md,
    },
    text: {
      color: theme.colors.text.primary,
      fontSize: 16,
    },
  });

export default function MyComponent() {
  const styles = useThemedStyles(createStyles);
  return <View style={styles.container} />;
}
```

## Gotchas

- **useMemo dependency**: Ensure `styles` function is memoized (defined outside component) or useCallback wrapped.
- **Context provider**: ThemeContext must wrap all components using these hooks.
- **Re-renders**: Changing theme triggers re-renders of all components using useTheme. Optimize with useMemo.
