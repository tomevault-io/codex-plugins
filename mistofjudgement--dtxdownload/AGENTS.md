# DTX Download Manager - AI Development Guide

## Project Overview
This is a TypeScript-based multi-platform application for downloading and managing DTX charts. The project consists of a **Node.js backend API server**, **vanilla TypeScript frontend GUI**, **Electron desktop app**, and **CLI tool** - all sharing common business logic and models.

## Architecture & Key Patterns

### Multi-Target Build System
```bash
# Backend API server
npm run server           # Build and start Express API
npm run server:dev       # Dev mode with watch

# Electron desktop app
npm run build:electron   # Build electron targets
npm run electron        # Start desktop app

# Frontend GUI 
npm run build:gui       # Build web GUI (in gui/ subfolder)

# CLI tool
npm run dtx             # CLI interface
npm run dtx:scrape      # Scrape charts
```

### Service Layer Architecture
- **Core services** in `src/core/`: Database, download, scraping
- **API layer** in `src/api/`: Express routes bridging GUI and core services  
- **GUI layer** in `gui/src/`: TypeScript managers pattern with event-driven architecture
- **Shared models** in `shared/`: TypeScript interfaces for API contracts

### Manager Pattern in Frontend (`gui/src/managers/`)
The GUI uses a **manager pattern** with event bus communication:
```typescript
// Main app orchestrates managers
class DTXDownloadManager {
  private chartManager: ChartManager;      // Chart data and filtering
  private selectionManager: SelectionManager; // Multi-select state
  private uiStateManager: UIStateManager;  // View modes, pagination, infinite scroll
}

// Managers communicate via EventBus
eventBus.on('charts-updated', (charts: Chart[]) => this.onChartsUpdated(charts));
eventBus.on('selection-changed', (selection: Set<string>) => this.onSelectionChanged(selection));
```

### TypeScript Path Mapping
Use `@shared/*` imports for shared models between frontend/backend:
```typescript
import { DownloadRequest, ChartQuery } from '@shared/models';
```

## Critical Development Workflows

### Database Integration
- **SQLite database** managed by `ChartDatabase` class
- **Automatic migrations** on startup (check `src/core/database/database.ts`)
- **Both backend and CLI** use same database instance
- Charts are scraped from external sources and stored locally

### Download Provider Detection
The system intelligently handles different download sources:
```typescript
// Downloads auto-detect provider from URL patterns
const provider = detectDownloadProvider(chart.downloadUrl);
// Google Drive folders → manual download instructions
// Google Drive files → automation attempts  
// Direct URLs → full automation
```

### Real-time Progress (SSE)
Backend provides Server-Sent Events for download progress:
```typescript
// Backend: res.write(`data: ${JSON.stringify(progressData)}\n\n`);
// Frontend: EventSource connection in APIClient
```

## Project-Specific Conventions

### Error Handling Strategy
- **Domain errors**: Custom error classes with descriptive messages
- **API responses**: Always include success/error status and message
- **Frontend**: Toast-style status updates via `updateStatus()` method
- **Downloads**: Graceful degradation (folder URLs → manual instructions)

### Testing Approach
```bash
npm run test:e2e        # End-to-end tests (real network calls)
npm run test:e2e:scraping # Database scraping workflow
npm run test:e2e:download # Download automation testing
```
- **E2E tests use real data sources** (ApprovedDTX website)
- **Network-aware tests** handle connectivity gracefully
- **Cleanup patterns** for temporary files and test databases

### File Organization Patterns
- **Smart ZIP extraction**: Detects existing folder structures vs creating new ones
- **DTXMania compatibility**: Organizes downloads for DTXMania import
- **Cross-platform paths**: All file operations use Node.js path utilities

## Integration Points

### Electron Integration
- `electron/main.ts` creates Express server internally
- `electron/preload.ts` exposes native APIs to renderer
- GUI runs in renderer process, communicates via IPC

### CLI and GUI Coordination
Both interfaces use the same core services:
```typescript
// CLI: src/cli.ts directly uses ScrapingService, DownloadService
// GUI: Communicates via API server (DTXApiServer)
```

### External Dependencies
- **Web scraping**: Cheerio for ApprovedDTX parsing
- **Downloads**: Custom Google Drive detection and automation
- **File processing**: extract-zip for automatic chart extraction
- **Database**: sqlite3 with custom migration system

## Development Tips

### Adding New Download Providers
1. Implement provider detection in `downloadProviderUtils.ts`
2. Add provider logic in download service
3. Update UI provider display in chart rendering

### Extending Chart Sources
1. Create new strategy in `src/scraping/strategies/`
2. Register strategy in `ScrapingService`
3. Add source configuration in API endpoints

### Working with the Manager Pattern
- **ChartManager**: Handles data, filtering, search
- **SelectionManager**: Multi-select state with bulk operations
- **UIStateManager**: View modes, pagination, infinite scroll
- Use EventBus for cross-manager communication

### Build Targets
```typescript
// Target-specific TypeScript configs:
// tsconfig.json (backend), gui/tsconfig.json (frontend), electron/tsconfig.json (electron)
```

The project emphasizes **progressive enhancement** (web → electron), **graceful degradation** (automation → manual), and **shared business logic** across multiple interfaces.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MistOfJudgement) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
