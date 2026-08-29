# AI Coding Guidelines for Hotel Booking System

## Architecture Overview
This is an Angular 20 standalone components application for hotel booking. All data is hardcoded in components with no external APIs - the `BookingService` provides in-memory storage for bookings. Components communicate via router navigation and shared services.

## Key Patterns
- **Standalone Components**: All components use `standalone: true` with explicit `imports` arrays (e.g., `imports: [CommonModule, FormsModule, RouterModule]`)
- **Data Flow**: Components contain hardcoded hotel/room data; bookings flow through `BookingService.addBooking()` and persist in memory only
- **Routing**: Simple path-based routes in `app.routes.ts` (e.g., `'hotels'` → `HotelListComponent`)
- **Forms**: Use `FormsModule` with `[(ngModel)]` for two-way binding in filters/selections
- **Models**: Simple interfaces in `models/` directory - `Hotel`, `Room`, `Booking`

## Development Workflow
- **Start**: `npm start` (ng serve) runs on http://localhost:4200
- **Test**: `npm test` runs Karma/Jasmine tests
- **Build**: `npm run build` outputs to `dist/`
- **Format**: Prettier with single quotes, 100 char width, Angular HTML parser

## Component Structure
```
components/[name]/
  [name].component.ts    # Component class with @Component decorator
  [name].html           # Template with Angular directives
  [name].css            # Component styles
  [name].spec.ts        # Jasmine unit tests
```

## Common Patterns
- Filter hotels by location using computed `filteredHotels` getter
- Navigate with `[routerLink]="['/book']"` (note: route is 'book', not 'booking')
- Display availability with `[ngClass]="{ 'unavailable': !hotel.available }"`
- Inject services in constructor: `constructor(private bookingService: BookingService)`

## Code Style
- Use single quotes in TypeScript, double in HTML attributes
- Import interfaces from `../../models/` relative paths
- Test standalone components with `imports: [ComponentName]` in TestBed

## Known Issues
- RouterLink in hotel-list.html points to '/booking' but route is '/book'
- AdminPanelComponent is empty/placeholder</content>
<parameter name="filePath">c:\Users\joelb\hotel-booking-system\.github\copilot-instructions.md

---
> Source: [joelbijukainikat-git1/LnT-Project-Hotel-Management](https://github.com/joelbijukainikat-git1/LnT-Project-Hotel-Management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
