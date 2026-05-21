---
name: nativewind-tailwind-config
category: styling
stack: [nativewind, tailwind, react-native]
keywords: [tailwind, nativewind, css-in-js, className, preset]
source-files: [tailwind.config.js]
---

# NativeWind & Tailwind Config

## Problem
You need to use Tailwind CSS utilities in React Native while extending it with custom colors and fonts from your design system.

## When to Use
- Setting up styling for a new React Native project
- Adding custom colors to Tailwind config
- Defining custom font families
- Using Tailwind utilities instead of StyleSheet

## Implementation

### Dependencies
```json
{
  "nativewind": "^4.0.0",
  "tailwindcss": "^3.0.0"
}
```

### Code

Create `tailwind.config.js`:

```javascript
module.exports = {
  content: [
    './components/**/*.{js,jsx,ts,tsx}',
    './app/**/*.{js,jsx,ts,tsx}',
    './modules/**/*.{js,jsx,ts,tsx}',
  ],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {
      colors: {
        text: {
          primary: '#1F2937',
          secondary: '#6B7280',
        },
        base: {
          white: '#FFFFFF',
          black: '#000000',
        },
        primary: {
          50: '#FAF2E1',
          500: '#3B2513',
          900: '#001D4C',
        },
        error: {
          50: '#FFE9E9',
          500: '#DC3030',
        },
        success: {
          50: '#E7FFF8',
          500: '#10FFC0',
        },
      },
      fontFamily: {
        sans: ['AvertaRegular', 'sans-serif'],
        'averta': ['AvertaRegular'],
        'averta-semibold': ['AvertaSemibold'],
        'averta-bold': ['AvertaBold'],
      },
    },
  },
  plugins: [],
};
```

**Usage in components:**

```typescript
import { View, Text } from 'react-native';

export default function MyComponent() {
  return (
    <View className="bg-primary-50 p-4 rounded-lg">
      <Text className="text-primary-500 font-averta-semibold text-lg">
        Hello World
      </Text>
      <Text className="text-text-secondary text-sm mt-2">
        Subtitle
      </Text>
    </View>
  );
}
```

## Usage Example

Common utilities:

```typescript
// Layout
className="flex flex-row items-center justify-between p-4"

// Colors
className="bg-primary-500 text-white"

// Sizing
className="w-full h-12 rounded-lg"

// Spacing
className="mt-4 mb-2 px-3 py-2"

// Responsive (breakpoints)
// NativeWind doesn't use responsive prefixes like web; use conditional logic instead
```

## Gotchas

- **NativeWind preset required**: Must include `presets: [require('nativewind/preset')]`.
- **No responsive prefixes**: React Native doesn't support media queries. Use conditional rendering or dynamic styling.
- **Font loading**: Custom fonts must be loaded via `expo-font` before use.
- **StyleSheet still used**: Complex styles (shadows, animations) are still easier with StyleSheet.
- **Content path**: Ensure all component paths are in `content` array so Tailwind generates their utilities.
