---
name: building-react-native-app-infra
description: Use when writing the non-UI layer of a React Native / Expo app — API client, request/response types for a REST/OpenAPI (e.g. FastAPI) backend, login and refresh tokens, TanStack Query hooks, creating orders or payments, camera/QR scanning and permissions, polling an order status, user-visible strings, environments (dev/staging/prod), EAS build/submit/OTA.
---

# Building React Native App Infra

## Overview
Companion to `building-react-native-app-ui`. Agents already know these libraries; what goes wrong is **guessing contracts and skipping required pieces when the task didn't mention them**. Every item below is required by default, not only when asked.

## Required defaults (one way per concern)

| Concern | Required |
|---|---|
| **API types** | Generated, never hand-written. `openapi-typescript` → `src/infra/api/schema.d.ts` (script `gen:api`, from the backend's `openapi.json` URL or a spec file in the repo) + `openapi-fetch` client typed with it. If no spec is reachable, STOP and ask for it — do not write "guessed" types with a TODO. If generated `paths` keys already include the router prefix (`/api/v1/...`), `baseUrl` is the origin only. |
| **Client & tokens** | One client in `src/infra/api/`. Refresh token in `expo-secure-store`, access token in memory. Single-flight refresh (all 401s await one promise). Log out only when refresh returns 4xx, never on network error. |
| **Data hooks** | TanStack Query hooks in `src/modules/{domain}/api.ts`. Mutations `retry: false`. Any POST that creates an order/payment/money movement sends an `Idempotency-Key` (one UUID per user attempt, reused on retry) — and note that the backend must honor it. |
| **Strings** | Every user-visible string through `t()` (`i18next` + `react-i18next` + `expo-localization`) from the first screen, even if only one language is requested. Default and `fallbackLng` = the product's primary market language (e.g. `vi` for a Vietnam app), not `en`. No literal text in JSX or `Alert`. |
| **Permissions** | Handle all three states: not yet asked → request; denied & `canAskAgain` → request; denied & `!canAskAgain` → button calling `Linking.openSettings()`. |
| **QR scanner** | `useRef` lock against repeated `onBarcodeScanned`; reset the lock on screen focus (`useFocusEffect`), not only via a button. |
| **Status polling** | `refetchInterval` returns `false` on terminal statuses; refetch when `AppState` becomes `active`; the server decides expiry (local countdown at 0 → refetch, don't assume). Navigate with `router.replace` so back can't reopen a stale payment. |
| **Environments** | `app.config.ts` keyed by `APP_VARIANT` (separate bundle id per env). API base URL chosen at runtime from `Application.applicationId`; `EXPO_PUBLIC_API_URL` only as a `__DEV__` override — otherwise an OTA bundled with the wrong env repoints production. |
| **Release** | `runtimeVersion: { policy: 'fingerprint' }`, one channel per env, `eas update` always with `--environment` and `APP_VARIANT` (npm scripts). Native change → new build, JS-only → OTA. |
| **Files** | Files under `app/` only default-export a screen. Shared components/helpers go in `src/components` / `src/modules`; never import from another route file. |

## Red flags — stop and fix
- `export type Battery = { ... }` written by hand for a backend response
- A string literal inside `<Text>` or `Alert.alert(...)`
- Permission screen with only a "request" button
- `useMutation` for an order/payment without an idempotency header
- `process.env.EXPO_PUBLIC_API_URL` used outside a `__DEV__` branch
- `export function Btn` inside `app/**`
