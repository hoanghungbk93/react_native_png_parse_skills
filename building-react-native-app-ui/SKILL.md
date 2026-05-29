---
name: building-react-native-app-ui
description: Use when starting or building UI in any React Native / Expo app, especially a NEW project — setting up theme tokens, shared component primitives, list screens, performance, and design-accurate layout before/while writing screens.
---

# Building React Native App UI

## Overview
**Foundation first, screens second.** In a new RN/Expo app, the reason screens end up inconsistent is that people write screens before establishing tokens + primitives. Build the foundation, then every screen composes it.

## Build order (do NOT skip ahead to screens)
1. **Theme tokens** — one `theme.ts` exporting `colors`, `spacing`, `radius`, `font`. Single source of truth. Screens never hardcode a hex/number.
2. **Shared primitives** — build these BEFORE any screen, because every screen needs them:
   - `ScreenBackground` (app bg wrapper)
   - `AppHeader` (title/subtitle, optional back button, title alignment, right slot)
   - `Button` (variants: primary/outline/link)
   - `Card`
   - `PaginatedList` (the list wrapper used by every list screen)
3. **Screens** — compose primitives only. A screen file should contain almost no raw styling.

## Touchables — version gotcha (RN)
`<Pressable style={({pressed}) => ({...})}>` can **silently drop layout/visual styles** (flexDirection, padding, backgroundColor, width, borderRadius) on some RN/Expo versions. Symptom: children stack vertically, no bg/padding.
- Use **`TouchableOpacity` + `StyleSheet.create({...})`** for touchables needing layout.
- Callback-style only for a single press-feedback prop.

## List screens — performance by default
- `FlatList` (never `.map()` a long list).
- Row component wrapped in `memo`.
- `useCallback` for `renderItem` + every handler; `useMemo` for derived/filtered data.
- **No inline `() => {}` or inline `{}` style objects** passed to rows (breaks memo).
- Stable `keyExtractor`; `getItemLayout` when row height is fixed.
- Lazy load: `onEndReached` → `fetchNextPage` (paginate, don't load all).

## Styling
- `StyleSheet.create` over inline objects (perf + reliability).
- Add a token instead of repeating a hex.
- Avoid white-on-white: a white card on a white bg needs a border or tint.

## Architecture
- `app/` (or navigator) = thin routing, one-line re-exports.
- `src/modules/{domain}/screens`, `src/components/{ui|layout|feedback|list|cards}`, `src/infra/` (api, storage, push).

## Design fidelity
- Match design by **ratio**, not fixed pt: size elements relative to their container; account for safe-area insets. See `analyzing-designs-for-code` for pixel/ratio extraction.
- Verify on a real screenshot before claiming done.

## Common mistakes
- Writing screens before tokens/primitives exist → inconsistent UI, hard to re-skin.
- Hardcoding colors/sizes instead of tokens/ratios.
- `.map()` lists, inline callbacks in rows → jank + re-render.
- Pressable callback-style for layout → collapsed touchables.
