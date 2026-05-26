---
name: ui-typography-component
category: scaffold
stack: [react-native, nativewind, typescript]
keywords: [typography, Heading, Paragraph, text, semantic, font, level, size]
source-files: [components/ui/heading/Heading.tsx, components/ui/paragraph/Paragraph.tsx]
---

# Typography Components

## Problem
You need semantic `Heading` and `Paragraph` wrappers that enforce the design system font scale and prevent raw `Text` usage anywhere in the codebase.

## When to Use
- Rendering any text in the app — always use `Heading` or `Paragraph`, never raw `Text`
- Creating typographic hierarchy (display, h1–h4, body, labels, numeric)
- Applying font family via the `font` prop rather than className conflicts

## Rules

- **Never use the raw React Native `Text` component** in any screen, component, or layout.
- Use `Heading` for display text and heading levels.
- Use `Paragraph` for all body copy, labels, captions, and numeric text.
- Color is controlled via `className` NativeWind tokens (`text-on_surface`, `text-secondary`, etc.) — never a `color` prop.

## Implementation

### Code

`components/ui/heading/types.ts`:

```typescript
import { TextStyle } from 'react-native';

export type HeadingLevel = 'display' | 'h1' | 'h2' | 'h3' | 'h4';

export interface HeadingProps {
  level?: HeadingLevel;
  children: React.ReactNode;
  className?: string;
  style?: TextStyle;
}
```

`components/ui/heading/Heading.tsx`:

```typescript
import React from 'react';
import { Text } from 'react-native';
import { HeadingProps } from './types';

const levelClasses: Record<string, string> = {
  display: 'font-playfair text-display-lg',
  h1:      'font-playfair text-headline-lg-mobile',
  h2:      'font-playfair text-headline-md',
  h3:      'font-playfair text-headline-sm',
  h4:      'font-inter-semibold text-body-lg',
};

export function Heading({ level = 'h1', children, className = '', style }: HeadingProps) {
  return (
    <Text className={`text-on_surface ${levelClasses[level]} ${className}`} style={style}>
      {children}
    </Text>
  );
}
```

`components/ui/paragraph/types.ts`:

```typescript
import { TextStyle } from 'react-native';

export type ParagraphSize = 'lg' | 'md' | 'sm' | 'xs';
export type ParagraphFont = 'inter' | 'inter-semibold' | 'inter-bold' | 'mono';

export interface ParagraphProps {
  size?: ParagraphSize;
  font?: ParagraphFont;
  children: React.ReactNode;
  className?: string;
  style?: TextStyle;
}
```

`components/ui/paragraph/Paragraph.tsx`:

```typescript
import React from 'react';
import { Text } from 'react-native';
import { ParagraphProps } from './types';

const sizeClasses: Record<string, string> = {
  lg: 'text-body-lg',
  md: 'text-body-md',
  sm: 'text-body-sm',
  xs: 'text-label-sm',
};

const fontClasses: Record<string, string> = {
  inter:           'font-inter',
  'inter-semibold':'font-inter-semibold',
  'inter-bold':    'font-inter-bold',
  mono:            'font-mono',
};

export function Paragraph({ size, font, children, className = '', style }: ParagraphProps) {
  const sizeClass = size ? sizeClasses[size] : '';
  const fontClass = font ? fontClasses[font] : (size === 'xs' ? 'font-inter-semibold' : 'font-inter');

  return (
    <Text className={`text-on_surface ${sizeClass} ${fontClass} ${className}`} style={style}>
      {children}
    </Text>
  );
}
```

`components/ui/index.ts`:

```typescript
export { Heading } from './heading/Heading';
export { Paragraph } from './paragraph/Paragraph';
export type { HeadingProps, HeadingLevel } from './heading/types';
export type { ParagraphProps, ParagraphSize, ParagraphFont } from './paragraph/types';
```

## Usage Example

```typescript
import { Heading, Paragraph } from '@/components/ui';

export function ArticleCard() {
  return (
    <View className="px-gutter pt-lg">
      <Paragraph size="xs" className="mb-xs uppercase text-on_surface_variant">
        CATEGORY
      </Paragraph>

      <Heading level="h1" className="mb-sm">
        The headline goes here
      </Heading>

      <Paragraph className="text-on_surface_variant">
        Body copy in Inter18pt-Regular. Color overridden via className.
      </Paragraph>

      <Paragraph size="sm" font="inter-semibold" className="uppercase tracking-widest text-secondary">
        LABEL TEXT
      </Paragraph>

      <Paragraph font="mono" className="text-numeric-lg text-on_surface">
        10:00
      </Paragraph>
    </View>
  );
}
```

## Gotchas

- **`font="mono"`** switches to `RobotoMono-Medium`. Use for times, scores, counters.
- **`font="inter-semibold"`** is for labels, button text, and uppercase caps.
- **`size="xs"`** defaults to `font-inter-semibold` automatically (label style).
- **Color is always via `className`**: Use `text-on_surface`, `text-secondary`, `text-on_primary`, etc. There is no `color` prop.
- **`Heading level="h4"`** uses `font-inter-semibold` (not Playfair) — it's a subheading, not a display level.
- **`Heading level="display"`** is for the app splash/logo only — 48px Playfair Bold.
- Font families resolve from `tailwind.config.js` and are loaded via the `expo-font` config plugin in `app.config.ts`.
