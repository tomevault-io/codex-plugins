# GitHub Copilot Instructions for Plotfolio

## Project Overview

Plotfolio is a modern property management application for Nigerian land plots, built with Next.js 16, TypeScript, MongoDB, and Leaflet maps. The application enables landowners to visualize, manage, and track their property portfolios with interactive maps and survey document integration.

## Code Style and Standards

### General Guidelines

- Use TypeScript for all new files with strict type checking
- Follow Next.js 16 App Router conventions
- Use functional components with React hooks
- Implement proper error handling and loading states
- Add meaningful comments for complex logic

### File Organization

- Place all API routes in `/src/app/api/`
- Place all components in `/src/components/` organized by feature
- Place all types in `/src/types/`
- Place all utilities in `/src/lib/`
- Place all database models in `/src/models/`
- **Place ALL documentation in `/docs/` directory** - NO markdown files in root except README.md

### Component Structure

```typescript
"use client"; // Only if needed for client components

import { ... } from "react";
import { ... } from "@/types/...";
import { ... } from "@/lib/...";

interface ComponentProps {
  // Props with detailed comments
}

export default function Component({ prop1, prop2 }: ComponentProps) {
  // State declarations
  // Effects
  // Event handlers
  // Render
}
```

### TypeScript Guidelines

- Use interfaces for object shapes
- Use enums for fixed sets of values
- Export types from `/src/types/property.ts`
- Avoid `any` type - use proper types or `unknown`
- Use optional chaining and nullish coalescing

## Architecture Patterns

### Database Layer

- Use Mongoose for MongoDB interactions
- Define schemas in `/src/models/`
- Use `PropertyService` class for business logic
- Always handle database errors gracefully

### API Routes

- Follow REST conventions (GET, POST, PUT, DELETE)
- Return proper HTTP status codes
- Use NextResponse for responses
- Validate input data before processing
- Connect to database at start of handler

### Component Patterns

- Use dynamic imports for map components (avoid SSR issues)
- Implement loading states for async operations
- Use controlled components for forms
- Lift state up when needed for sibling communication

## Feature-Specific Guidelines

### Property Management

- Properties must have coordinates, area, and status
- Support PropertyType enum: RESIDENTIAL, COMMERCIAL, etc.
- Support PropertyStatus enum: OWNED, FOR_SALE, DEVELOPMENT, etc.

### Boundary Registration

Three methods supported:

1. **Survey Document Upload** - AI extraction of boundaries
2. **Manual Drawing** - Interactive map-based drawing
3. **API Integration** - Programmatic boundary submission

### Map Components

- Use Leaflet with React Leaflet wrapper
- Always use dynamic imports with `ssr: false`
- Center on Abuja, Nigeria by default (9.0765°N, 7.4951°E)
- Support zoom levels 10-18 for property visualization

### Survey Data

- Store coordinates as `{ point: string, lat: number, lng: number }[]`
- Store boundaries as `{ from: string, to: string, distance: number }[]`
- Calculate area using Shoelace formula
- Support bearings and measurements

## MongoDB Integration

### Connection

- Use connection pooling via `/src/lib/mongoose.ts`
- Handle connection errors gracefully
- Check connection state before operations

### Schema Design

- Use Mongoose schemas with TypeScript types
- Include timestamps (createdAt, updatedAt)
- Add indexes for frequently queried fields
- Support embedded documents for related data

## Documentation Rules

### Critical Documentation Rule

**When a user provides a command with "(instructions)" in parentheses:**

1. Recognize this as a directive to codify the command
2. Add the rule/pattern to this copilot-instructions.md file
3. Place the rule in the appropriate section
4. Make it clear and actionable for future reference
5. Commit the change to the instructions file

Example:

```
User: "create a docs folder and make sure all mds always go there (instructions)"
→ Add rule: "Place ALL documentation in /docs/ directory"
```

### Documentation Standards

- **ALL markdown files MUST go in `/docs/` directory**
- Only exception: `README.md` in root
- Follow naming convention: UPPERCASE for main docs (README.md, CONTRIBUTING.md)
- Update `/docs/README.md` index when adding new documentation
- Reference docs from main README when user-facing

### Creating Documentation

When creating new documentation:

1. Create file in `/docs/` directory
2. Use clear, descriptive filename
3. Include proper sections and examples
4. Add to documentation index
5. Link from main README if applicable

## API Endpoints

### Properties

- `GET /api/properties` - List all properties
- `GET /api/properties/[id]` - Get single property
- `POST /api/properties` - Create property
- `PUT /api/properties/[id]` - Update property
- `DELETE /api/properties/[id]` - Delete property

### Survey Data

- `POST /api/properties/[id]/survey` - Add survey data to property

### Seeding

- `POST /api/seed` - Seed database with sample properties

## Environment Variables

Required in `.env.local`:

```bash
MONGODB_URI=mongodb+srv://...
MONGODB_DB=plotfolio
```

## Testing Considerations

- Test database connections before operations
- Validate coordinates are within reasonable ranges
- Ensure boundary points form valid polygons (3+ points)
- Test map rendering at different zoom levels
- Verify survey data extraction accuracy

## Nigerian Land Plot Specifics

### Typical Dimensions

- Residential plots: 600-1000 sqm
- Plot shapes: Usually rectangular or irregular polygons
- Common locations: Abuja (Maitama, Gwarinpa, Jahi, Kubwa, Wuye)

### Coordinate System

- Use decimal degrees (not DMS)
- Latitude range: ~6° to 14°N (Nigeria bounds)
- Longitude range: ~3° to 15°E (Nigeria bounds)

### Legal Documents

- Certificate of Occupancy (C of O)
- Survey plans with bearings and distances
- Deed of assignment
- Plot allocation papers

## Common Patterns

### Error Handling

```typescript
try {
	// Operation
} catch (error) {
	console.error("Descriptive message:", error);
	return NextResponse.json({ error: "User-friendly message" }, { status: 500 });
}
```

### Loading States

```typescript
const [loading, setLoading] = useState(true);
const [error, setError] = useState<string | null>(null);

useEffect(() => {
	const fetchData = async () => {
		try {
			setLoading(true);
			setError(null);
			// Fetch data
		} catch (err) {
			setError(err instanceof Error ? err.message : "An error occurred");
		} finally {
			setLoading(false);
		}
	};
	fetchData();
}, []);
```

### API Client

```typescript
import { PropertyAPI } from "@/lib/api";

const properties = await PropertyAPI.getAllProperties();
const property = await PropertyAPI.getProperty(id);
await PropertyAPI.updateProperty(id, updates);
```

## Performance Optimization

- Use dynamic imports for heavy components (maps, charts)
- Implement pagination for large property lists
- Use MongoDB indexes for frequent queries
- Cache static data when appropriate
- Minimize re-renders with proper memoization

## Security Considerations

- Validate all user input
- Sanitize file uploads
- Use environment variables for secrets
- Implement proper error messages (don't expose internals)
- Validate coordinates and boundaries before saving

## Deployment Notes

- Application runs on Next.js 16 with Turbopack
- MongoDB Atlas for production database
- Static assets in `/public`
- Environment variables required for production
- Build with `npm run build`

## Git Commit Convention

Use conventional commit messages:

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Code style changes
- `refactor:` - Code refactoring
- `test:` - Test additions/changes
- `chore:` - Maintenance tasks

## Command Codification Rule

**IMPORTANT: When user includes "(instructions)" in their command:**

This signals that the command should become a permanent rule/pattern in this file. When you see this:

1. Parse the command for the underlying rule/pattern
2. Determine the appropriate section of this file
3. Add the rule in clear, actionable language
4. Make it specific and easy to follow
5. Commit the updated instructions file

Example patterns:

- "create X folder and make sure Y always go there (instructions)" → Add file organization rule
- "always use Z when doing W (instructions)" → Add code pattern rule
- "validate X before Y (instructions)" → Add validation rule

The "(instructions)" marker means: **make this a permanent part of how the project works**.

---

Last updated: November 2, 2025

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stadier)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/stadier)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
