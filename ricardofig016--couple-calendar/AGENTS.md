# Couple Calendar AI Instructions

## Big Picture

- Expo (React Native) app with expo-router. The backend is a Google Apps Script Web App used as a no-auth proxy to Google Calendar.
- All calendar data is pulled per selected calendar and merged client-side. See [context/event-context.tsx](context/event-context.tsx) and [hooks/use-calendars.ts](hooks/use-calendars.ts).

## Backend Integration

- Apps Script endpoints:
  - GET `?action=listCalendars` returns `{ ok, data: CalendarInfo[] }`.
  - GET `?action=getEvents&calendarId=X&daysBack=N&daysForward=M` returns `{ ok, data: CalendarEvent[] }`.
  - POST `{ action, id, calendarId, title, description, start, end }` for create/edit/delete.
- The app stores the Apps Script deployment ID (not the full URL). `useScriptUrl` builds `https://script.google.com/macros/s/<ID>/exec` and migrates stored full URLs by extracting the ID. See [hooks/use-script-url.ts](hooks/use-script-url.ts).
- Settings screen is the entry point for configuring the Deployment ID. See [app/(tabs)/settings.tsx](<app/(tabs)/settings.tsx>).

## State + Storage Patterns

- Calendar selections live in AsyncStorage: `couple_calendar_selected_cals`, `couple_calendar_primary_cal`. Use `loadStoredSelections()` in `useFocusEffect` to sync Settings changes on focus.
- Event range is configurable via Settings and stored in AsyncStorage: `couple_calendar_days_back`, `couple_calendar_days_forward`. `refreshEvents` reads storage each time to ensure latest range. See [hooks/use-event-range.ts](hooks/use-event-range.ts).
- After any create/edit/delete, call `refreshEvents(false)` for silent reload.

## Event Form Rules

- Form state is centralized in [hooks/use-event-form.ts](hooks/use-event-form.ts) and used by [app/(tabs)/index.tsx](<app/(tabs)/index.tsx>).
- Single-day uses `date`, `startTime`, `endTime`; multi-day uses `multiStartDate`, `multiEndDate` (date+time).
- Start/end are kept ordered: changing start or end adjusts the other if it would be before/after.
- All-day sets `00:00` to `23:59` (single) or `00:00` start to `23:59` end (multi).

## Presets + Rendering

- Presets resolve placeholders client-side before POST: `Preset.resolveTitle()` and `Preset.resolve()` in [utils/preset.ts](utils/preset.ts).
- Descriptions can contain `<b>`; UI strips HTML with `.replace(/<[^>]*>?/gm, "")` when rendering.

## UI Conventions

- Use `ThemedText` and `ThemedView` with `useThemeColor()` tokens (`text`, `background`, `icon`, `tint`, `danger`, `border`, `success`).
- Use `Fonts.rounded` for headings and `IconSymbol` for icons.

## Dev Workflow

- Run locally: `npx expo start` (choose web/iOS/Android).
- Lint: `npm run lint`.
- Android APK: `eas build --platform android --profile preview` with envs in [eas.json](eas.json).

---
> Source: [ricardofig016/couple-calendar](https://github.com/ricardofig016/couple-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
