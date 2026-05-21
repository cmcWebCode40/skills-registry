---
name: ui-typography-component
category: scaffold
stack: [react-native, nativewind, typescript]
keywords: [typography, heading, paragraph, text, semantic, size]
source-files: [components/ui/heading/Heading.tsx, components/ui/paragraph/Paragraph.tsx]
---

# Typography Component

## Problem
You need semantic typography components (Heading, Paragraph) with predefined sizes that enforce design system consistency.

## When to Use
- Creating text hierarchy in your app (h1, h2, h3, body, caption, etc.)
- Ensuring font families, weights, and sizes match your design system
- Reducing duplication of text styling across components
- Supporting dark mode with semantic color names

## Implementation

### Dependencies
```json
{
  "react-native": "^0.81.0",
  "nativewind": "^4.0.0"
}
```

### Code

Create `components/ui/heading/types.ts`:

```typescript
import { TextProps } from 'react-native';

export type HeadingSize = 'h1' | 'h2' | 'h3' | 'h4';

export interface HeadingProps extends TextProps {
  size?: HeadingSize;
  color?: 'primary' | 'secondary' | 'error' | 'success';
  children: React.ReactNode;
  className?: string;
}
```

Create `components/ui/heading/Heading.tsx`:

```typescript
import React from 'react';
import { Text } from 'react-native';
import { HeadingProps } from './types';

const Heading = React.forwardComponent<Text, HeadingProps>(
  ({ size = 'h1', color = 'primary', children, className, ...rest }, ref) => {
    const sizeStyles = {
      h1: 'text-4xl font-averta-bold leading-tight',
      h2: 'text-3xl font-averta-bold',
      h3: 'text-2xl font-averta-semibold',
      h4: 'text-xl font-averta-semibold',
    };

    const colorStyles = {
      primary: 'text-text-primary',
      secondary: 'text-text-secondary',
      error: 'text-error-500',
      success: 'text-success-500',
    };

    return (
      <Text
        ref={ref}
        className={`
          ${sizeStyles[size]}
          ${colorStyles[color]}
          ${className || ''}
        `}
        {...rest}
      >
        {children}
      </Text>
    );
  }
);

Heading.displayName = 'Heading';

export default Heading;
```

Create `components/ui/paragraph/types.ts`:

```typescript
import { TextProps } from 'react-native';

export type ParagraphSize = 'lg' | 'base' | 'sm' | 'xs';

export interface ParagraphProps extends TextProps {
  size?: ParagraphSize;
  color?: 'primary' | 'secondary' | 'error' | 'success' | 'muted';
  weight?: 'regular' | 'medium' | 'semibold' | 'bold';
  children: React.ReactNode;
  className?: string;
}
```

Create `components/ui/paragraph/Paragraph.tsx`:

```typescript
import React from 'react';
import { Text } from 'react-native';
import { ParagraphProps } from './types';

const Paragraph = React.forwardComponent<Text, ParagraphProps>(
  (
    { size = 'base', color = 'primary', weight = 'regular', children, className, ...rest },
    ref
  ) => {
    const sizeStyles = {
      lg: 'text-lg leading-relaxed',
      base: 'text-base leading-relaxed',
      sm: 'text-sm leading-normal',
      xs: 'text-xs leading-normal',
    };

    const colorStyles = {
      primary: 'text-text-primary',
      secondary: 'text-text-secondary',
      error: 'text-error-500',
      success: 'text-success-500',
      muted: 'text-gray-400',
    };

    const weightStyles = {
      regular: 'font-averta-regular',
      medium: 'font-averta-semibold',
      semibold: 'font-averta-semibold',
      bold: 'font-averta-bold',
    };

    return (
      <Text
        ref={ref}
        className={`
          ${sizeStyles[size]}
          ${colorStyles[color]}
          ${weightStyles[weight]}
          ${className || ''}
        `}
        {...rest}
      >
        {children}
      </Text>
    );
  }
);

Paragraph.displayName = 'Paragraph';

export default Paragraph;
```

Create `components/ui/index.ts` (barrel export):

```typescript
export { default as Heading } from './heading/Heading';
export { default as Paragraph } from './paragraph/Paragraph';
export { default as Button } from './button/Button';
export * from './heading/types';
export * from './paragraph/types';
export * from './button/types';
```

## Usage Example

```typescript
import { Heading, Paragraph, Button } from 'components/ui';

export default function MyScreen() {
  return (
    <View className="p-4">
      <Heading size="h1">Welcome</Heading>
      <Paragraph size="base" color="secondary" className="mt-2">
        This is a description
      </Paragraph>
      <Paragraph size="sm" color="muted" className="mt-1">
        Updated 2 hours ago
      </Paragraph>
      <Button variant="contained" className="mt-4">
        Get Started
      </Button>
    </View>
  );
}
```

## Gotchas

- **Font family**: Ensure `font-averta-*` classes are defined in tailwind.config.js and fonts are loaded via `expo-font`.
- **Line height**: Adjust `leading-*` values based on your design system. `leading-relaxed` = 1.625; `leading-normal` = 1.5.
- **Color inheritance**: Text color doesn't inherit in React Native. Always specify color explicitly.
- **forwardRef**: Use `forwardRef` if you need to access the underlying Text component (e.g., for testing).
