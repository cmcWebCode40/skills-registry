# Skills Registry Index

One-line per skill. Use this to scan and identify relevant skills before reading the full files.

---

## Scaffold (Project Setup & Foundations)

- **expo-app-config** — Configure app.config.ts, EAS build profiles, and environment management
- **expo-router-navigation** — File-based routing, route groups, tab navigator with role-based visibility
- **project-folder-structure** — Recommended directory layout (libs/, modules/, components/, app/)
- **axios-setup** — Singleton Axios instance with auth interceptors and error handling
- **react-query-setup** — QueryClient configuration, provider setup, and query key factories
- **auth-context-setup** — AuthContext for session state, role-based routing, secure token storage
- **theming-config** — Theme object structure, dark mode setup, color tokens, typography
- **key-storage-handler** — Two-tier storage pattern (SecureStore for tokens, MMKV for app state)
- **ui-button-component** — Button component with variants, sizes, loading states, and icon support
- **ui-typography-component** — Heading and Paragraph wrappers with semantic sizing

---

## API (Data Fetching, Errors, Auth)

- **data-fetching-react-query** — Pattern for useQuery, useMutation, useInfiniteQuery with error handling
- **api-services** — Axios singleton, request interceptors, error extraction utilities
- **token-refresh** — Silent 401 refresh with queue pattern to prevent concurrent refresh storms
- **session-management** — Register/unregister callbacks for auth state changes, force logout flow

---

## Styling (Design System, Theming)

- **nativewind-tailwind-config** — Tailwind config for React Native, custom colors, fonts, responsive
- **theming-hooks** — useTheme (context consumer), useThemedStyles (memoized dynamic styles)

---

## Architecture (Code Organization)

- **feature-driven-folder-structure** — Module-per-feature pattern with hooks, components, utils, types inside

---

## Utils (Reusable Utilities)

- **image-handler** — pickImageHandler for iOS + Android, normalized asset format, multi-selection
- **file-downloader** — downloadImage with redirect handling, permission checks, MediaLibrary integration
- **file-sharing** — PDF generation and sharing (expo-print + expo-sharing), receipt example
- **toast-config** — react-native-toast-message setup with custom success/error/primary/info variants

---

## Hooks (Custom React Hooks)

- **use-notifications** — Push notification registration, listener setup, deep link routing on notification tap
- **use-pagination** — useInfiniteQuery wrapper for cursor-based pagination, flattens pages, handles refetch
- **use-timer** — Countdown timer with AppState resume handling, formatted output (mins:secs)

---

**Total:** 24 skills  
**Categories:** 6 (Scaffold, API, Styling, Architecture, Utils, Hooks)
