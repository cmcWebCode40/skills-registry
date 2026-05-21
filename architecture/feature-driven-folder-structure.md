---
name: feature-driven-folder-structure
category: architecture
stack: [react-native, typescript]
keywords: [module, feature, folder-structure, domain-driven, organization]
source-files: [modules/ folder structure]
---

# Feature-Driven Folder Structure

## Problem
You need to organize code by feature (auth, chats, profile) rather than type (components, hooks, utils) to improve maintainability and modularity.

## When to Use
- Organizing a new feature
- Scaling a codebase beyond a few screens
- Enabling different team members to own different features
- Making features independently testable and deployable

## Implementation

### Code

Recommended module structure:

```
modules/auth/
├── hooks/
│   ├── useAuth.ts              # Main feature hook
│   ├── useLogin.ts             # Specific operation hook
│   └── index.ts                # Barrel export
├── components/
│   ├── LoginForm.tsx
│   ├── SignupForm.tsx
│   └── index.ts
├── login/                      # Feature sub-section
│   ├── LoginForm.tsx
│   └── index.ts
├── signup/
│   └── index.ts
├── schema/                     # Validation
│   ├── loginSchema.ts
│   └── index.ts
├── types/
│   ├── index.ts                # LoginPayload, AuthResponse, etc.
├── utils/
│   ├── authHelpers.ts
│   └── index.ts
├── services/                   # Feature API (optional)
│   ├── auth.service.ts
│   └── index.ts
├── constants.ts                # Query keys, constants
├── index.ts                    # Barrel export (public API)
```

**Module barrel export** (`modules/auth/index.ts`):

```typescript
export { default as useAuth } from './hooks/useAuth';
export { LoginForm } from './login';
export * from './types';
export { authQueryKeys } from './constants';
```

**Usage in screens:**

```typescript
// From another module
import { useAuth, authQueryKeys } from 'modules/auth';

// Imports are clean; implementation details hidden
```

## Usage Example

**Organizing a chat feature:**

```
modules/chats/
├── hooks/
│   ├── useChat.ts              # Fetch messages
│   ├── useChatList.ts          # Fetch conversations
│   └── useSendMessage.ts       # Send message
├── components/
│   ├── ChatBubble.tsx
│   ├── ChatInput.tsx
│   └── ConversationCard.tsx
├── services/
│   ├── chat.service.ts         # API calls
│   ├── chat.socket.ts          # MQTT socket
│   └── localMessageStore.ts    # Offline queue
├── types/
│   ├── index.ts
├── constants.ts                # chatQueryKeys
└── index.ts
```

## Gotchas

- **No cross-module imports**: Don't import from other modules directly. If you need shared logic, move it to libs/.
- **Barrel exports**: Use index.ts to export only the public API. Hide implementation details.
- **Circular dependencies**: Modules should not import each other. If they need shared data, use a shared context in libs/ or pass via props.
- **Services per module**: Some modules (chats) have a service layer. This is optional; prefer calling serverApi directly in hooks.
