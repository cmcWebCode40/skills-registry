---
name: project-folder-structure
category: scaffold
stack: [react-native, typescript, expo]
keywords: [folder-structure, architecture, organization, modules, libs]
source-files: [folder tree structure]
---

# Project Folder Structure

## Problem
You need a scalable folder structure that separates infrastructure (libs), features (modules), and shared UI (components) to maintain clarity as the codebase grows.

## When to Use
- Starting a new Expo project
- Organizing existing code for clarity and maintainability
- Understanding where to place new features, utilities, or components
- Onboarding new developers to the project

## Implementation

### Code

Recommended directory structure:

```
my-expo-app/
├── app/                          # Expo Router pages (file-based routes)
│   ├── _layout.tsx              # Root layout + providers
│   ├── (auth)/                  # Route group: unauthenticated screens
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   ├── signup.tsx
│   │   └── otp.tsx
│   └── (tabs)/                  # Route group: tab navigator
│       ├── _layout.tsx
│       ├── index.tsx            # Home tab
│       ├── reports.tsx
│       ├── chats/
│       │   └── [chatId].tsx     # Dynamic segment
│       └── settings.tsx
│
├── components/                   # Shared, domain-agnostic UI
│   ├── icons/                   # SVG icon components (PascalCase)
│   │   ├── HomeIcon.tsx
│   │   ├── ChatsIcon.tsx
│   │   └── types.ts
│   ├── layouts/                 # Layout wrappers
│   │   ├── ScreenLayout.tsx
│   │   └── ParallaxScrollView.tsx
│   ├── ui/                      # Design system primitives
│   │   ├── button/
│   │   │   ├── Button.tsx
│   │   │   └── types.ts
│   │   ├── form-group/
│   │   ├── heading/
│   │   ├── paragraph/
│   │   ├── modal/
│   │   └── index.ts             # Barrel export
│   ├── sheets/                  # Bottom sheets
│   │   └── ActionSheetModal.tsx
│   └── utils/                   # Image/file helpers (not logic utils)
│       ├── ImageForm.tsx
│       └── UploadedImageCard.tsx
│
├── libs/                        # Infrastructure (no business logic)
│   ├── config/                  # Configuration
│   │   ├── ConfigKeys.ts        # API URLs, env flags
│   │   └── index.ts
│   ├── constants/               # Global constants
│   │   ├── theme.ts            # Theme object
│   │   ├── fonts.ts
│   │   ├── url.ts
│   │   └── index.ts
│   ├── context/                 # React Contexts
│   │   ├── AuthContext.tsx
│   │   ├── ThemeContext.tsx
│   │   ├── AppContext.tsx
│   │   └── index.ts
│   ├── hooks/                   # Global reusable hooks
│   │   ├── useTheme.ts
│   │   ├── useThemedStyles.ts
│   │   ├── usePagination.ts
│   │   ├── useTimer.ts
│   │   ├── useNotifications.ts
│   │   └── index.ts
│   ├── services/                # API & session management
│   │   ├── index.ts            # Axios singleton
│   │   ├── type.ts             # API response types
│   │   ├── sessionManager.ts   # Auth callbacks
│   │   └── queryClient.ts      # React Query config
│   └── utils/                   # Pure utility functions
│       ├── keyStorage.ts       # SecureStore + MMKV
│       ├── imageHandlers.ts
│       ├── fileDownloader.ts
│       ├── shareReceipt.ts
│       ├── ToastConfig.tsx
│       ├── sizing.ts           # Responsive scaling
│       ├── formatters.ts
│       ├── permissions.ts
│       └── index.ts
│
├── modules/                     # Feature modules (domain-driven)
│   ├── auth/                   # Auth feature
│   │   ├── hooks/              # Feature-specific hooks
│   │   │   └── useAuth.ts
│   │   ├── login/              # Sub-features
│   │   │   ├── LoginForm.tsx
│   │   │   └── index.ts
│   │   ├── signup/
│   │   ├── schema/             # Yup/Zod validation
│   │   │   └── authSchema.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── utils/              # Feature-specific utils
│   │   │   └── authHelpers.ts
│   │   ├── components/         # Feature-specific components
│   │   ├── services/           # Feature-specific services
│   │   ├── constants.ts        # Query keys, constants
│   │   └── index.ts
│   ├── chats/
│   │   ├── hooks/
│   │   │   ├── useChat.ts
│   │   │   └── useChatList.ts
│   │   ├── components/
│   │   ├── services/
│   │   │   └── chat.service.ts
│   │   ├── types/
│   │   ├── constants.ts
│   │   └── index.ts
│   ├── profile/
│   ├── reports/
│   ├── dashboard/
│   └── ... (other features)
│
├── assets/                      # Static assets
│   ├── icon.png
│   ├── splash.png
│   └── images/
│
├── app.config.ts               # Expo config
├── tailwind.config.js          # NativeWind/Tailwind config
├── tsconfig.json               # TypeScript config
├── package.json
├── .env.example                # Environment variables template
└── README.md
```

## Usage Example

**Importing from each layer:**

```typescript
// From components (UI primitives)
import { Button, Heading, FormGroup } from 'components/ui';
import { HomeIcon } from 'components/icons';

// From libs (infrastructure)
import { useTheme } from 'libs/hooks';
import { serverApi } from 'libs/services';
import { AUTH_TOKEN } from 'libs/utils/keyStorage';

// From modules (features)
import { useAuth } from 'modules/auth/hooks';
import { LoginForm } from 'modules/auth/login';
import { chatQueryKeys } from 'modules/chats/constants';

// Cross-module imports are rare; use libs if you need shared data
```

## Gotchas

- **No cross-module imports**: Modules should not import from each other. Use libs for shared data.
- **Barrel exports (index.ts)**: Every folder exports its public API via index.ts. This enables clean imports.
- **components/utils vs libs/utils**: components/utils = UI helper components (ImageForm, UploadedImageCard). libs/utils = pure functions (formatters, keyStorage, imageHandlers).
- **Module-level services**: Some modules (chats) have a service.ts layer. This is inconsistent; prefer calling serverApi directly in hooks.
- **Features in app/ vs modules/**: Screens live in app/ (routes); logic lives in modules/ (hooks, components, services).
