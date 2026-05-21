---
name: theming-config
category: scaffold
stack: [react-native, typescript, dark-mode, design-tokens]
keywords: [theme, colors, typography, tokens, dark-mode]
source-files: [libs/constants/theme.ts, libs/context/ThemeContext.tsx]
---

# Theming Config

## Problem
You need a centralized theme object with color tokens, typography settings, and dark mode support without hardcoding colors throughout the codebase.

## When to Use
- Setting up a new Expo project with consistent colors and typography
- Implementing dark mode support
- Creating a single source of truth for design tokens
- Enabling theme switching at runtime

## Implementation

### Dependencies
```json
{
  "react-native": "^0.81.0",
  "typescript": "^5.0.0"
}
```

### Code

Create `libs/constants/theme.ts`:

```typescript
const tintColorLight = '#DB2777';
const tintColorDark = '#fff';

export const commonColors = {
  text: {
    primary: '#1F2937',
    secondary: '#6B7280',
  },
  base: {
    white: '#FFFFFF',
    black: '#000000',
  },
  primary: {
    30: '#FFFEFA',
    50: '#FAF2E1',
    100: '#C6D8FF',
    200: '#E0BFA0',
    300: '#C78C5F',
    500: '#3B2513',
    600: '#D4A67F',
    700: '#0C3E90',
    800: '#5B391E',
    900: '#001D4C',
  },
  error: {
    50: '#FFE9E9',
    100: '#FFC4C4',
    500: '#DC3030',
    900: '#450303',
  },
  success: {
    50: '#E7FFF8',
    500: '#10FFC0',
    900: '#007245',
  },
};

export const colors = {
  light: {
    background: '#f8fafc',
    tint: tintColorLight,
    tabIconDefault: '#f3f3f3',
    tabIconSelected: tintColorLight,
    ...commonColors,
  },
  dark: {
    background: '#030712',
    tint: tintColorDark,
    tabIconDefault: '#f3f3f3',
    tabIconSelected: tintColorDark,
    ...commonColors,
  },
};

export const appTheme = {
  colors: colors.light,
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  borderRadius: {
    sm: 4,
    md: 8,
    lg: 16,
    full: 9999,
  },
  shadows: {
    sm: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 1 },
      shadowOpacity: 0.1,
      shadowRadius: 2,
      elevation: 2,
    },
    md: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.15,
      shadowRadius: 4,
      elevation: 4,
    },
  },
};

export type Theme = typeof appTheme;
```

Create `libs/context/ThemeContext.tsx`:

```typescript
import React, { createContext, useContext, useEffect, useState } from 'react';
import { useColorScheme } from 'react-native';
import { appTheme, colors } from 'libs/constants/theme';
import { USER_COLOR_PREFERENCE, saveToAsyncStore, getFromAsyncStore } from 'libs/utils';

interface ThemeContextType {
  theme: typeof appTheme;
  mode: 'light' | 'dark';
  toggleColorScheme: () => Promise<void>;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeContextProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const colorScheme = useColorScheme();
  const [mode, setMode] = useState<'light' | 'dark'>(colorScheme || 'light');
  const [theme, setTheme] = useState(appTheme);

  useEffect(() => {
    bootstrapTheme();
  }, []);

  const bootstrapTheme = async () => {
    const savedMode = await getFromAsyncStore(USER_COLOR_PREFERENCE);
    if (savedMode) {
      setMode(savedMode as 'light' | 'dark');
      updateTheme(savedMode as 'light' | 'dark');
    } else {
      const systemMode = colorScheme || 'light';
      setMode(systemMode);
      updateTheme(systemMode);
    }
  };

  const updateTheme = (newMode: 'light' | 'dark') => {
    const newTheme = {
      ...appTheme,
      colors: newMode === 'light' ? colors.light : colors.dark,
    };
    setTheme(newTheme);
  };

  const toggleColorScheme = async () => {
    const newMode = mode === 'light' ? 'dark' : 'light';
    setMode(newMode);
    updateTheme(newMode);
    await saveToAsyncStore(USER_COLOR_PREFERENCE, newMode);
  };

  return (
    <ThemeContext.Provider value={{ theme, mode, toggleColorScheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeContextProvider');
  }
  return context;
};

export default ThemeContext;
```

## Usage Example

**Access theme in StyleSheet:**

```typescript
import { useTheme } from 'libs/hooks';
import { StyleSheet } from 'react-native';

export default function MyComponent() {
  const { theme } = useTheme();

  const styles = StyleSheet.create({
    container: {
      backgroundColor: theme.colors.background,
      padding: theme.spacing.md,
      borderRadius: theme.borderRadius.lg,
    },
    text: {
      color: theme.colors.text.primary,
      fontSize: 16,
    },
  });

  return <View style={styles.container} />;
}
```

**With useThemedStyles (memoized):**

```typescript
import useThemedStyles from 'libs/hooks/useThemedStyles';

const useStyles = () =>
  useThemedStyles((theme) => ({
    container: {
      backgroundColor: theme.colors.background,
      padding: theme.spacing.md,
    },
  }));

export default function MyComponent() {
  const styles = useStyles();
  return <View style={styles.container} />;
}
```

**Toggle dark mode:**

```typescript
const { mode, toggleColorScheme } = useTheme();

<Button onPress={toggleColorScheme}>
  {mode === 'light' ? '🌙' : '☀️'}
</Button>
```

## Gotchas

- **Color scale numbers**: Use 50, 100, 200, etc. for consistency. Don't use random numbers.
- **useColorScheme()**: Returns the system preference on first render. Save user preference to persist across app restarts.
- **Dark mode colors**: Ensure sufficient contrast for accessibility. Test with `black: #000` and `white: #FFF`.
- **Shadow elevation**: iOS uses shadowOffset/shadowOpacity; Android uses elevation. Define both for cross-platform consistency.
- **Theme updates**: Changing the theme object reference triggers re-renders. Use useMemo in providers if you have nested contexts.
