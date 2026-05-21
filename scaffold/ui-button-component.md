---
name: ui-button-component
category: scaffold
stack: [react-native, nativewind, typescript]
keywords: [button, component, variant, size, loading, icon]
source-files: [components/ui/button/Button.tsx]
---

# Button Component

## Problem
You need a reusable Button component with variants (contained, outlined, text), sizes, loading state, and icon support while maintaining design system consistency.

## When to Use
- Creating a standard button for your app's UI
- Supporting multiple button styles without duplicating code
- Adding loading, disabled, and icon support
- Ensuring consistency with design tokens

## Implementation

### Dependencies
```json
{
  "react-native": "^0.81.0",
  "nativewind": "^4.0.0"
}
```

### Code

Create `components/ui/button/types.ts`:

```typescript
import { PressableProps } from 'react-native';

export type ButtonVariant = 'contained' | 'outlined' | 'text' | 'ghost-primary' | 'outlined-danger';
export type ButtonSize = 'xs' | 'sm' | 'md' | 'lg';

export interface ButtonProps extends PressableProps {
  variant?: ButtonVariant;
  size?: ButtonSize;
  isLoading?: boolean;
  startIcon?: React.ReactNode;
  endIcon?: React.ReactNode;
  children: React.ReactNode;
  className?: string;
}
```

Create `components/ui/button/Button.tsx`:

```typescript
import React from 'react';
import { Pressable, Text, View, ActivityIndicator } from 'react-native';
import { ButtonProps } from './types';
import { useTheme } from 'libs/hooks';

const Button = React.forwardComponent<View, ButtonProps>(
  (
    {
      variant = 'contained',
      size = 'md',
      isLoading = false,
      startIcon,
      endIcon,
      children,
      disabled,
      className,
      ...rest
    },
    ref
  ) => {
    const { theme } = useTheme();

    const baseStyles = 'flex-row items-center justify-center rounded';

    const variantStyles = {
      contained: `bg-primary-500 ${disabled ? 'opacity-50' : ''}`,
      outlined: `border border-gray-300 ${disabled ? 'opacity-50' : ''}`,
      text: `${disabled ? 'opacity-50' : ''}`,
      'ghost-primary': `${disabled ? 'opacity-50' : ''}`,
      'outlined-danger': `border border-error-500 ${disabled ? 'opacity-50' : ''}`,
    };

    const sizeStyles = {
      xs: 'px-2 py-1',
      sm: 'px-3 py-2',
      md: 'px-4 py-3',
      lg: 'px-6 py-4',
    };

    const textColorStyles = {
      contained: 'text-white',
      outlined: 'text-gray-900',
      text: 'text-primary-500',
      'ghost-primary': 'text-primary-500',
      'outlined-danger': 'text-error-500',
    };

    const textSizeStyles = {
      xs: 'text-xs',
      sm: 'text-sm',
      md: 'text-base',
      lg: 'text-lg',
    };

    return (
      <Pressable
        ref={ref}
        disabled={disabled || isLoading}
        className={`
          ${baseStyles}
          ${variantStyles[variant]}
          ${sizeStyles[size]}
          ${className || ''}
        `}
        {...rest}
      >
        {isLoading ? (
          <ActivityIndicator color={theme.colors.primary[500]} />
        ) : (
          <>
            {startIcon && <View className="mr-2">{startIcon}</View>}
            <Text
              className={`
                font-averta-semibold
                ${textColorStyles[variant]}
                ${textSizeStyles[size]}
              `}
            >
              {children}
            </Text>
            {endIcon && <View className="ml-2">{endIcon}</View>}
          </>
        )}
      </Pressable>
    );
  }
);

Button.displayName = 'Button';

export default Button;
```

## Usage Example

```typescript
import Button from 'components/ui/button';
import { PlusIcon } from 'components/icons';

export default function MyScreen() {
  const [isLoading, setIsLoading] = React.useState(false);

  const handlePress = async () => {
    setIsLoading(true);
    // Do something
    setIsLoading(false);
  };

  return (
    <>
      {/* Contained button */}
      <Button variant="contained" size="md" onPress={handlePress}>
        Save
      </Button>

      {/* Outlined button with icon */}
      <Button variant="outlined" startIcon={<PlusIcon />}>
        Add Item
      </Button>

      {/* Loading state */}
      <Button isLoading={isLoading} onPress={handlePress}>
        Uploading...
      </Button>

      {/* Danger variant */}
      <Button variant="outlined-danger" onPress={() => handleDelete()}>
        Delete
      </Button>
    </>
  );
}
```

## Gotchas

- **Disabled state**: Set both `disabled` prop and visual opacity. The prop prevents press events; opacity provides feedback.
- **Loading state**: Automatically disables the button. Wrap isLoading check to prevent double-submissions.
- **Icon spacing**: Use `mr-2`/`ml-2` for icon margins. Adjust based on your design system.
- **Text weight**: Button text uses `font-averta-semibold` from your Tailwind config. Ensure the font is loaded.
- **Accessible hit slop**: React Native buttons have a minimum 44x44 tap target. Adjust padding if needed for accessibility.
