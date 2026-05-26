---
name: nativewind-tailwind-config
category: styling
stack: [nativewind, tailwind, react-native, expo]
keywords: [tailwind, nativewind, className, preset, design-tokens, fonts, PostScript]
source-files: [tailwind.config.js]
---

# NativeWind & Tailwind Config

## Problem
You need Tailwind CSS utilities in React Native with design tokens (colors, fonts, spacing, font sizes) that exactly mirror your `theme.ts` constants so that NativeWind classes and `StyleSheet.create` always produce the same values.

## When to Use
- Setting up styling for a new React Native/Expo project
- Defining custom color palette and font families from your design system
- Ensuring NativeWind classes (`bg-primary`, `text-on_surface`, `font-inter`) resolve to the same values as `theme.ts`

## Implementation

### Dependencies
```json
{
  "nativewind": "^4.0.0",
  "tailwindcss": "^3.0.0"
}
```

### Code

`tailwind.config.js`:

```javascript
module.exports = {
  content: [
    './app/**/*.{js,jsx,ts,tsx}',
    './components/**/*.{js,jsx,ts,tsx}',
    './libs/**/*.{js,jsx,ts,tsx}',
    './modules/**/*.{js,jsx,ts,tsx}',
  ],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {
      colors: {
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
        outline: '#1C1C1A',
        outline_variant: '#D4CCBC',
        secondary: '#777771',
        on_secondary: '#F5F0E8',

        tertiary: '#3D5A2A',
        on_tertiary: '#F5F0E8',
        tertiary_container: '#D4E8C2',

        error: '#BA1A1A',
        on_error: '#FFFFFF',
        error_container: '#FFDAD6',
        on_error_container: '#93000A',
      },
      fontFamily: {
        playfair:          ['PlayfairDisplay-Bold'],
        inter:             ['Inter18pt-Regular'],
        'inter-semibold':  ['Inter18pt-SemiBold'],
        'inter-bold':      ['Inter18pt-Bold'],
        mono:              ['RobotoMono-Medium'],
      },
      fontSize: {
        'display-lg':        ['48px', { lineHeight: '56px', fontWeight: '700' }],
        'headline-lg':       ['32px', { lineHeight: '40px', fontWeight: '700' }],
        'headline-lg-mobile':['28px', { lineHeight: '36px', fontWeight: '700' }],
        'headline-md':       ['24px', { lineHeight: '32px', fontWeight: '700' }],
        'headline-sm':       ['20px', { lineHeight: '28px', fontWeight: '700' }],
        'body-lg':           ['18px', { lineHeight: '28px' }],
        'body-md':           ['16px', { lineHeight: '24px' }],
        'body-sm':           ['14px', { lineHeight: '20px' }],
        'label-lg':          ['14px', { lineHeight: '20px', fontWeight: '600' }],
        'label-sm':          ['11px', { lineHeight: '16px', fontWeight: '600' }],
        'numeric-lg':        ['36px', { lineHeight: '44px', fontWeight: '500' }],
        'numeric-display':   ['48px', { lineHeight: '56px', fontWeight: '500' }],
        'numeric-md':        ['24px', { lineHeight: '32px', fontWeight: '500' }],
        'numeric-sm':        ['12px', { lineHeight: '16px' }],
        'button-text':       ['14px', { lineHeight: '20px', fontWeight: '600' }],
      },
      spacing: {
        xs:           '4px',
        sm:           '8px',
        md:           '16px',
        lg:           '24px',
        xl:           '32px',
        xxl:          '48px',
        gutter:       '16px',
        margin_mobile:'16px',
        section_gap:  '48px',
      },
    },
  },
  plugins: [],
};
```

## Usage in Components

NativeWind classes map directly to the token names above:

```typescript
import { Heading, Paragraph } from '@/components/ui';

export function ArticleCard() {
  return (
    <View className="bg-background px-gutter pt-lg">
      <Paragraph size="xs" className="mb-xs uppercase text-on_surface_variant">
        CATEGORY
      </Paragraph>
      <View className="h-px bg-on_surface mb-md" />
      <Heading level="h1" className="mb-sm">
        Headline text here
      </Heading>
      <Paragraph className="text-on_surface_variant">
        Body copy uses Inter18pt-Regular automatically via font-inter.
      </Paragraph>
    </View>
  );
}
```

Common patterns:
```
bg-background          → background color
text-on_surface        → primary text
text-on_surface_variant → muted/secondary text
text-secondary         → secondary brand color
bg-primary             → button fill
text-on_primary        → button text

font-playfair          → PlayfairDisplay-Bold (headlines)
font-inter             → Inter18pt-Regular (body)
font-inter-semibold    → Inter18pt-SemiBold (labels, caps)
font-mono              → RobotoMono-Medium (numbers, times)

px-md py-sm            → spacing tokens
h-[52px]               → arbitrary value when no token fits
```

## Gotchas

- **Font PostScript names**: `fontFamily` values must match PostScript names from the font files. `Inter18pt-Regular` not `Inter-Regular`. These are set via the `expo-font` config plugin — do NOT use `useFonts`.
- **No rounded corners**: The Broadsheet design system uses 0px border radius. Do not add `rounded-*` to the config or use it in classNames.
- **No shadows**: Do not add shadow tokens. The design system has no shadows or elevation.
- **Sync with theme.ts**: Every color in this file must match a value in `COLORS`/`DARK_COLORS` in `theme.ts`. Every spacing token must match `SPACING`. Every font family must match `TYPOGRAPHY` entries. They are not auto-synced — update both manually.
- **NativeWind v4 preset required**: `presets: [require('nativewind/preset')]` must be present.
- **`libs/` in content**: Include `./libs/**/*.{js,jsx,ts,tsx}` so that any className in lib files is processed.
