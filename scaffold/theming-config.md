---
name: theming-config
category: scaffold
stack: [react-native, typescript, dark-mode, design-tokens]
keywords: [theme, colors, typography, spacing, tokens, dark-mode, Theme, useThemedStyles]
source-files: [libs/constants/theme.ts, libs/context/ThemeContext.tsx, libs/hooks/useTheme.ts, libs/hooks/useThemedStyles.ts]
---

# Theming Config

## Problem
You need a centralized theme object with color tokens, typography, and spacing that every `StyleSheet.create` call pulls from. No hardcoded hex values, pixel numbers, or font strings anywhere in the codebase.

## When to Use
- Setting up a new Expo project with consistent design tokens
- Implementing dark mode with a single source of truth
- Enabling `createStyles(theme)` to access `theme.colors.*`, `theme.spacing.*`, `theme.typography.*`

## Implementation

### Dependencies
```json
{
  "react-native-mmkv": "^3.0.0"
}
```

### Code

Create `libs/constants/theme.ts`:

```typescript
export const COLORS = {
  background: '#F5F0E8',
  surface: '#F5F0E8',
  surface_dim: '#DED9D2',
  surface_bright: '#FEFCF8',
  surface_container_lowest: '#FFFFFF',
  surface_container_low: '#F8F3EB',
  surface_container: '#EDE8DF',
  surface_container_high: '#E7E2DA',
  surface_container_highest: '#D4CCBC',

  on_surface: '#1C1C1A',
  on_surface_variant: '#474741',
  inverse_surface: '#1C1C1A',
  inverse_on_surface: '#F5F0E8',
  primary: '#1C1C1A',
  on_primary: '#F5F0E8',
  primary_container: '#1C1C1A',
  on_primary_container: '#F5F0E8',
  inverse_primary: '#C8C6C3',

  outline: '#1C1C1A',
  outline_variant: '#D4CCBC',

  secondary: '#777771',
  on_secondary: '#F5F0E8',
  surface_tint: '#777771',

  tertiary: '#3D5A2A',
  on_tertiary: '#F5F0E8',
  tertiary_container: '#D4E8C2',
  on_tertiary_container: '#1C3610',

  error: '#BA1A1A',
  on_error: '#FFFFFF',
  error_container: '#FFDAD6',
  on_error_container: '#93000A',
};

export const DARK_COLORS: typeof COLORS = {
  background: '#1C1C1A',
  surface: '#2A2A28',
  surface_dim: '#141412',
  surface_bright: '#323230',
  surface_container_lowest: '#0F0F0D',
  surface_container_low: '#1F1F1D',
  surface_container: '#272725',
  surface_container_high: '#313130',
  surface_container_highest: '#3C3C3A',

  on_surface: '#F5F0E8',
  on_surface_variant: '#B8B7B4',
  inverse_surface: '#F5F0E8',
  inverse_on_surface: '#1C1C1A',
  primary: '#F5F0E8',
  on_primary: '#1C1C1A',
  primary_container: '#F5F0E8',
  on_primary_container: '#1C1C1A',
  inverse_primary: '#474744',

  outline: '#F5F0E8',
  outline_variant: '#474741',

  secondary: '#888883',
  on_secondary: '#1C1C1A',
  surface_tint: '#888883',

  tertiary: '#A8D38A',
  on_tertiary: '#0A2100',
  tertiary_container: '#2C4F15',
  on_tertiary_container: '#C4EFA3',

  error: '#FFB4AB',
  on_error: '#93000A',
  error_container: '#93000A',
  on_error_container: '#FFDAD6',
};

export type ThemeColors = typeof COLORS;

export const TYPOGRAPHY = {
  display_lg:        { fontFamily: 'PlayfairDisplay-Bold',  fontSize: 48, lineHeight: 56, letterSpacing: -0.96 },
  headline_lg:       { fontFamily: 'PlayfairDisplay-Bold',  fontSize: 32, lineHeight: 40, letterSpacing: -0.32 },
  headline_lg_mobile:{ fontFamily: 'PlayfairDisplay-Bold',  fontSize: 28, lineHeight: 36, letterSpacing: -0.28 },
  headline_md:       { fontFamily: 'PlayfairDisplay-Bold',  fontSize: 24, lineHeight: 32 },
  headline_sm:       { fontFamily: 'PlayfairDisplay-Bold',  fontSize: 20, lineHeight: 28 },
  body_lg:           { fontFamily: 'Inter18pt-Regular',     fontSize: 18, lineHeight: 28 },
  body_md:           { fontFamily: 'Inter18pt-Regular',     fontSize: 16, lineHeight: 24 },
  body_sm:           { fontFamily: 'Inter18pt-Regular',     fontSize: 14, lineHeight: 20 },
  label_lg:          { fontFamily: 'Inter18pt-SemiBold',    fontSize: 14, lineHeight: 20, letterSpacing: 0.84 },
  label_sm:          { fontFamily: 'Inter18pt-SemiBold',    fontSize: 11, lineHeight: 16, letterSpacing: 0.88 },
  numeric_display:   { fontFamily: 'RobotoMono-Medium',     fontSize: 48, lineHeight: 56 },
  numeric_lg:        { fontFamily: 'RobotoMono-Medium',     fontSize: 36, lineHeight: 44 },
  numeric_md:        { fontFamily: 'RobotoMono-Medium',     fontSize: 24, lineHeight: 32 },
  numeric_sm:        { fontFamily: 'RobotoMono-Medium',     fontSize: 12, lineHeight: 16 },
  button_text:       { fontFamily: 'Inter18pt-SemiBold',    fontSize: 14, lineHeight: 20, letterSpacing: 0.84 },
};

export const SPACING = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
  gutter: 16,
  margin_mobile: 16,
  section_gap: 48,
};

export type Theme = {
  colors: ThemeColors;
  spacing: typeof SPACING;
  typography: typeof TYPOGRAPHY;
};

export const lightTheme: Theme = {
  colors: COLORS,
  spacing: SPACING,
  typography: TYPOGRAPHY,
};

export const darkTheme: Theme = {
  colors: DARK_COLORS,
  spacing: SPACING,
  typography: TYPOGRAPHY,
};
```

Create `libs/context/ThemeContext.tsx`:

```typescript
import React, { createContext, useState } from 'react';
import { MMKV } from 'react-native-mmkv';

type ColorScheme = 'light' | 'dark';

interface ThemeContextType {
  colorScheme: ColorScheme;
  toggleTheme: () => void;
}

const storage = new MMKV({ id: 'theme' });
export const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeContextProvider({ children }: { children: React.ReactNode }) {
  const [colorScheme, setColorScheme] = useState<ColorScheme>(
    () => (storage.getString('theme') as ColorScheme) ?? 'light'
  );

  function toggleTheme() {
    const next: ColorScheme = colorScheme === 'light' ? 'dark' : 'light';
    storage.set('theme', next);
    setColorScheme(next);
  }

  return (
    <ThemeContext.Provider value={{ colorScheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Create `libs/hooks/useTheme.ts`:

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

Create `libs/hooks/useThemedStyles.ts`:

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

## Usage Example

```typescript
import { Theme } from '@/libs/constants/theme';
import { useThemedStyles } from '@/libs/hooks/useThemedStyles';
import { useMemo } from 'react';
import { StyleSheet, View } from 'react-native';

export function Card() {
  const { theme } = useThemedStyles();
  const styles = useMemo(() => createStyles(theme), [theme]);

  return <View style={styles.container} />;
}

function createStyles(theme: Theme) {
  return StyleSheet.create({
    container: {
      backgroundColor: theme.colors.surface_container,
      padding: theme.spacing.md,
      borderBottomWidth: 1,
      borderBottomColor: theme.colors.outline_variant,
    },
    label: {
      fontFamily: theme.typography.label_sm.fontFamily,
      fontSize: theme.typography.label_sm.fontSize,
      color: theme.colors.on_surface_variant,
    },
  });
}
```

## Gotchas

- `Theme` has exactly three keys: `colors`, `spacing`, `typography`. Never add `borderRadius` or `shadows` to it (0px radius, no shadows in the Broadsheet design system).
- Font strings must match PostScript names embedded in the font files: `PlayfairDisplay-Bold`, `Inter18pt-Regular`, `Inter18pt-SemiBold`, `Inter18pt-Bold`, `RobotoMono-Medium`.
- `lightTheme` and `darkTheme` are module-level constants, never re-created on render. This keeps `useMemo` dependencies stable.
- `createStyles` is always a hoisted `function` declaration placed **after** the component, never before it or at module level with a `const`.
- `tailwind.config.js` color and font tokens must mirror `theme.ts` exactly — keep them in sync manually.
