---
name: project-folder-structure
category: scaffold
stack: [react-native, typescript, expo]
keywords: [folder-structure, architecture, organization, modules, libs, components, icons]
source-files: [folder tree structure]
---

# Project Folder Structure

## Problem
You need a scalable folder structure that separates infrastructure (`libs`), features (`modules`), shared UI (`components`), and screens (`app`) to maintain clarity as the codebase grows.

## When to Use
- Starting a new Expo project
- Understanding where to place new files, features, or utilities
- Onboarding new developers

## Implementation

### Directory Structure

```
my-expo-app/
├── app/                              # Expo Router screens (file-based routes)
│   ├── _layout.tsx                  # Root layout — providers only, no routing logic
│   ├── (onboarding)/                # Route group: onboarding flow
│   │   ├── _layout.tsx
│   │   ├── index.tsx                # Startup routing gate
│   │   ├── intro.tsx
│   │   ├── setup-day-end.tsx
│   │   ├── setup-reminder.tsx
│   │   └── setup-notifications.tsx
│   ├── (auth)/                      # Route group: unauthenticated screens
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   └── signup.tsx
│   └── (tabs)/                      # Route group: tab navigator
│       ├── _layout.tsx
│       ├── index.tsx
│       └── settings.tsx
│
├── components/                       # Shared, domain-agnostic UI
│   ├── icons/                       # All icons live here — never import from libraries directly
│   │   ├── Icon.tsx                 # Unified icon component wrapping Ionicons
│   │   ├── types.ts                 # IconProps extends SvgProps + typed name
│   │   └── index.ts
│   ├── layouts/                     # Screen layout wrappers
│   │   └── ScreenLayout.tsx
│   └── ui/                          # Design system primitives
│       ├── heading/
│       │   ├── Heading.tsx
│       │   └── types.ts
│       ├── paragraph/
│       │   ├── Paragraph.tsx
│       │   └── types.ts
│       ├── button/
│       │   ├── Button.tsx
│       │   └── types.ts
│       └── index.ts
│
├── libs/                            # Infrastructure (no business logic)
│   ├── config/
│   │   └── index.ts
│   ├── constants/
│   │   ├── theme.ts                 # SINGLE SOURCE OF TRUTH for colors, spacing, typography
│   │   └── index.ts
│   ├── context/
│   │   ├── ThemeContext.tsx
│   │   ├── AuthContext.tsx
│   │   ├── AppContext.tsx
│   │   └── index.ts
│   ├── hooks/
│   │   ├── useTheme.ts
│   │   ├── useThemedStyles.ts       # Returns { theme: Theme, isDark }
│   │   ├── useNotifications.ts
│   │   ├── usePagination.ts
│   │   ├── useTimer.ts
│   │   └── index.ts
│   ├── services/
│   │   ├── index.ts                 # Axios singleton
│   │   ├── type.ts
│   │   ├── sessionManager.ts
│   │   └── queryClient.ts
│   └── utils/
│       ├── keyStorage.ts            # MMKV (fastStorage) + SecureStore
│       ├── imageHandlers.ts
│       ├── fileDownloader.ts
│       ├── shareReceipt.ts
│       ├── ToastConfig.tsx
│       └── index.ts
│
├── modules/                         # Feature modules (domain-driven)
│   ├── onboarding/                  # Example feature
│   │   ├── components/              # Reusable UI for this feature
│   │   ├── store/                   # MMKV-backed state
│   │   ├── types/
│   │   └── index.ts
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── schema/
│   │   ├── services/
│   │   ├── types/
│   │   └── index.ts
│   └── index.ts                     # Re-exports all modules
│
├── assets/
│   ├── fonts/                       # TTF files organized by family
│   │   ├── Inter/static/
│   │   ├── Playfair_Display/static/
│   │   └── Roboto_Mono/static/
│   └── images/
│
├── app.config.ts                    # Expo config + expo-font plugin
├── tailwind.config.js               # NativeWind tokens (must mirror theme.ts)
├── global.css                       # NativeWind base import
├── tsconfig.json
├── package.json
└── .env.example
```

## Import Examples

```typescript
// UI primitives
import { Heading, Paragraph, Button } from '@/components/ui';

// Icons — always from here, never from @expo/vector-icons directly
import { Icon } from '@/components/icons';

// Layout
import { ScreenLayout } from '@/components/layouts/ScreenLayout';

// Theme
import { useThemedStyles } from '@/libs/hooks/useThemedStyles';
import { Theme } from '@/libs/constants/theme';

// Storage
import { fastStorage, FAST_KEYS, devClearAllStorage } from '@/libs/utils/keyStorage';

// Feature module
import { SetupMasthead, completeOnboarding } from '@/modules/onboarding';
```

## Gotchas

- **`app/` = screens only**: No logic, hooks, or components defined here — just Expo Router screen files that import from `modules/` and `components/`.
- **`modules/` = no `screens/` subfolder**: Module files that render a full screen belong in `app/(route-group)/`. If it's in a module, it's a reusable component or logic unit.
- **`components/icons/` = the only icon source**: Never import `Ionicons` or any icon library directly in a screen or feature component. All icons go through `Icon` from `@/components/icons`.
- **`libs/constants/theme.ts` = single source of truth**: Never define hex values, spacing numbers, or font family strings anywhere else. `tailwind.config.js` must mirror it.
- **`_layout.tsx` = providers only**: Root and route-group layouts never contain `router.replace` or any routing logic.
