---
name: feature-driven-folder-structure
category: architecture
stack: [react-native, typescript]
keywords: [module, feature, folder-structure, domain-driven, organization, barrel-export]
source-files: [modules/ folder structure]
---

# Feature-Driven Folder Structure

## Problem
You need to organize code by feature (auth, onboarding, chats) rather than by type (components, hooks, utils) to keep features cohesive and independently maintainable.

## When to Use
- Organizing a new feature
- Scaling a codebase beyond a few screens
- Enabling team members to own separate features
- Making features independently testable

## Rules

- **No `screens/` subfolder inside a module**. If it looks like a screen, it belongs in `app/(route-group)/`. Modules contain only reusable components, store logic, types, hooks, and utils.
- **No cross-module imports**. Modules must not import from each other. Shared logic moves to `libs/`.
- **Barrel exports** (`index.ts`): Every module exports its public API through a single `index.ts`. Keep it up to date when adding files.

## Implementation

### Module Structure

```
modules/onboarding/
├── components/
│   ├── SetupMasthead.tsx
│   ├── ProgressBar.tsx
│   ├── TimeDisplay.tsx
│   ├── ToggleRow.tsx
│   ├── NotificationRow.tsx
│   ├── OnboardingSlide.tsx
│   └── index.ts
├── store/
│   └── onboardingStore.ts
├── types/
│   └── index.ts
└── index.ts
```

```
modules/auth/
├── components/
│   ├── LoginForm.tsx
│   └── index.ts
├── hooks/
│   ├── useAuth.ts
│   └── index.ts
├── schema/
│   └── authSchema.ts
├── types/
│   └── index.ts
├── utils/
│   └── authHelpers.ts
├── services/
│   └── auth.service.ts
├── constants.ts
└── index.ts
```

### Barrel Export Example

`modules/onboarding/index.ts`:

```typescript
export { SetupMasthead } from './components/SetupMasthead';
export { ProgressBar } from './components/ProgressBar';
export { TimeDisplay } from './components/TimeDisplay';
export { ToggleRow } from './components/ToggleRow';
export { NotificationRow } from './components/NotificationRow';
export { OnboardingSlide } from './components/OnboardingSlide';
export { saveOnboardingData, completeOnboarding, isOnboardingComplete } from './store/onboardingStore';
export type { OnboardingState } from './types';
```

Usage in a screen:

```typescript
import { SetupMasthead, ProgressBar, completeOnboarding } from '@/modules/onboarding';
```

## Gotchas

- **No `screens/` in modules**: A file that renders a full screen belongs in `app/(route-group)/`. A module component is something reused across multiple screens or composed into a screen — not the screen itself.
- **Barrel exports stay current**: Every time you add a new component or util to a module, export it from `index.ts`. Stale barrels cause silent import failures.
- **Circular dependencies**: Modules never import each other. If two modules need the same data, it belongs in `libs/context` or `libs/utils`.
- **`store/` for feature state**: Module-level persistence lives in `modules/<feature>/store/`. Use `fastStorage` from `libs/utils/keyStorage` for MMKV access.
