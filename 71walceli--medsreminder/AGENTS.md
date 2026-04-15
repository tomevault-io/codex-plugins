# Copilot Instructions for MedReminder

## Project Overview
This is a cross-platform medication reminder app built with **Expo + React Native** using **NativeWind** (Tailwind CSS) for styling. The app runs as a single-page application with screen-based navigation using state management.

## Architecture & Key Patterns

### State-Driven Navigation
- Single main component (`app/page.tsx`) manages all screen state via `currentScreen` 
- No React Navigation router - uses conditional rendering for screens: `"home" | "create" | "settings"`
- Screen components in `/components/screens/` receive props for data and callbacks

### React Native + Web Hybrid Approach
- Uses React Native components (`View`, `Text`) but runs in DOM mode (`"use dom"`)
- for now, we still use `lucide-react` icons instead of `lucide-react-native`
- NativeWind for styling, allowing Tailwind-like utility classes in React Native
- No need for `react-native-web` - everything runs in the browser
- This is a mobile-first design, but works in web browsers too
- For now, we don't want to use actual React Native components like `ScrollView` or `FlatList` - we use standard HTML elements like `div` and `ul` for layout. 
    - This is a temporary workaround until we create a fully functional mobile app MVP.
    - Therefore, any full React Native migration is not needed at this stage.

### UI Component System
- **shadcn/ui-style components** in `/components/ui/` built with:
  - Radix UI primitives for complex components
  - `class-variance-authority` (cva) for variant management
  - NativeWind classes for styling
- **Design tokens**: CSS custom properties in `styles/global.css` with automatic dark mode
- **Utility function**: Use `cn()` from `lib/utils.ts` to merge Tailwind classes

### Data Flow Pattern
```typescript
// Central state in page.tsx
const [alarms, setAlarms] = useState<Alarm[]>(initialAlarms)

// Screen components receive data + callbacks
<AlarmListScreen 
  alarms={alarms}
  onToggleAlarm={toggleAlarm}
  onDeleteAlarm={deleteAlarm}
  onEditAlarm={(alarm) => {
    setEditingAlarm(alarm)
    setCurrentScreen("create")
  }}
/>
```

### Native code: Android
- **Native APIs**: We will use native APIs for scheduling alarms, starting with Android
- Preferably, all native code should be written in Kotlin.

## Development Workflow

### Essential Commands
```bash
# Start development
npx expo start

# Platform-specific
npx expo start --web    # Browser preview
npx expo start --ios    # iOS simulator  
npx expo start --android # Android emulator

# Reset project structure
npm run reset-project
```

### Key File Structure
- `app/page.tsx` - Main application component with all state
- `components/screens/` - Full-screen components (AlarmListScreen, CreateEditAlarmScreen, etc.)
- `components/ui/` - Reusable UI components following shadcn/ui patterns
- `types/index.ts` - TypeScript interfaces (Alarm, AppSettings)
- `data/mockData.ts` - Initial data for development

## Component Development Patterns

### Screen Components
- Always accept props interface with data + callback functions
- Use TypeScript interfaces for all props
- Import icons from `lucide-react` (not `lucide-react-native`)

### UI Components
- Follow shadcn/ui pattern: variants with `cva`, forwardRef for DOM components
- Use `cn()` for className merging
- Support `asChild` pattern with Radix Slot when applicable

### Event Handling Convention
```tsx
// React pattern (preferred)
<Button onClick={() => action()}>

// Avoid React Native patterns, use web-compatible handlers
<Button onPress={() => action()}> // not used in this project
```

### Styling Approach
- **NativeWind classes** for layout/spacing: `flex`, `items-center`, `p-4`, `space-y-4`
- **CSS custom properties** for colors: `bg-primary`, `text-foreground`
- **Dark mode**: Automatic via `dark:` prefixes and CSS custom properties
- **Responsive**: Use `max-w-md mx-auto` pattern for mobile-first design

## Integration Points
### Mock Data Strategy
- All data lives in `data/mockData.ts` for rapid prototyping
- No backend integration yet - we'll use AsyncStorage or similar for persistence
- Alarm scheduling simulated with setTimeout (see demo trigger in page.tsx)

### Alarm Scheduling
- We are going to be implementing an AlarmService in the future to handle scheduling and notifications
    - It has be native to the platform, in orser to use the native alarm APIs.
    - We'll do it for Android first, however, iOS isn't going to be supported for a while.

### Theming System
- HSL-based design tokens in CSS custom properties
- Toggle dark mode via `settings.darkMode` state
- Conditional className: `${settings.darkMode ? "dark" : ""}`

## Common Tasks

### Adding New Screen
1. Create component in `components/screens/`
2. Add screen key to `currentScreen` union type
3. Add conditional render in `app/page.tsx`
4. Add navigation button in bottom nav

### Creating New UI Component
1. Use `components/ui/button.tsx` as template
2. Import `cn` utility and implement `cva` variants
3. Use React.forwardRef for DOM refs
4. Follow TypeScript interface pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/71walceli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
