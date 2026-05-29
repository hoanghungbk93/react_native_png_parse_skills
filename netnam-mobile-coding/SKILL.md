---
name: netnam-mobile-coding
description: Use when writing or fixing UI in the NetNam HRApp Expo/React-Native mobile app (apps/mobile) — building screens, shared components, list screens, touchable rows, theming, or shipping changes via OTA.
---

# NetNam Mobile Coding

## Overview
Conventions for the `apps/mobile` Expo Router + React Native app. Match surrounding code; colors come from one theme; ship via OTA.

## Touchables — the #1 gotcha
`<Pressable style={({pressed}) => ({...})}>` **silently drops layout/visual styles** (flexDirection, padding, backgroundColor, width, borderRadius) on this RN/Expo version. Symptom: icon+text stack vertically, no bg, no padding — looks "collapsed".
- **Use `TouchableOpacity` + `StyleSheet.create({...})`** for any touchable that needs layout/visual styling.
- Pressable callback-style is OK only for a single press-feedback property.
- Sweep with `grep -rn "style={({ pressed })" src` after touchable work.

## Styling & theme
- All colors in `src/theme.ts` → import `colors`. Add a token rather than hardcoding a repeated hex.
- `colors.bg` = pure white; `colors.surface` = mint-tint. App background = `ScreenBackground`/`GradientBackground`.
- Prefer `StyleSheet.create` over inline objects (perf + reliability).
- Beware white-on-white: a `colors.card` (white) card on a white bg needs a border/tint to stay visible.

## Shared components (reuse, don't reinvent)
- `AppHeader` (title/subtitle/showBack/`alignTitle`/right) — category & detail screens use `alignTitle="right"`.
- `BackButton` — lime circle + white chevron; pass `fallback`.
- `ScreenBackground` / `GradientBackground` — every screen wraps in this.
- `PaginatedList` — every list screen.

## List screens — avoid re-render
- Wrap each row in `memo(...)`.
- `useCallback` all handlers/renderItem; never pass inline `() => {}` or inline object styles into list rows.
- Lazy load via query `fetchNextPage`; `useMemo` the derived/filtered array.
```tsx
const NotifRow = memo(function NotifRow({ item, onPress }) { /* ... */ });
const handlePress = useCallback((it) => onPress(it), [onPress]);
const renderItem = useCallback(({ item }) => <NotifRow item={item} onPress={handlePress} />, [handlePress]);
```

## Architecture
`app/` = thin routing (one-line re-exports). `src/modules/{domain}/screens`, `src/components/{ui|layout|feedback|list|cards}`, `src/infra/`.

## Shipping (OTA)
- **Always `cd apps/mobile` first** — running `eas update` from repo root targets a stale `hrapp` EAS project instead of `netnam-hr`.
- `npx eas-cli update --branch production --message "..."` (runtime 0.2.0, Android+iOS). User kills+reopens app to receive.

## Common mistakes
- Sizing icons in fixed pt instead of a ratio of the container → see `analyzing-designs-for-code`.
- Editing a shared component without checking all call sites (`grep` usages first).
- Hardcoding hex instead of adding a theme token.
