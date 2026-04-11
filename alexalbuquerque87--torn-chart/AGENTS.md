# Torn TV Chart - AI Coding Instructions

## Project Overview
A professional candlestick charting application for Torn game stock market data using React 19, TypeScript, Vite, and the Lightweight Charts library. Single-component monolith architecture (~1650 lines in `App.tsx`).

## Architecture

### Single-Component Design
- **Everything lives in `src/App.tsx`**: Main `App` component contains state, nested `CandleChart` component, and `SettingsButton`
- **No component split**: Intentionally monolithic - avoid suggesting extraction unless explicitly requested
- **Co-located utilities**: Helper functions (`computeEma`, `computeBollingerBands`, etc.) are defined at file scope above components

### State Management
- Pure React hooks - no external state library
- Critical refs for chart APIs: `chartRef`, `seriesRef`, plus refs for each indicator series (e.g., `ema9Ref`, `rsiSeriesRef`)
- **Never trigger re-renders from chart API calls** - use refs to directly manipulate series without state changes
- Visibility toggles use boolean state (e.g., `visibleEmas.ema9`, `visibleBB`, `visibleVolume`)

### Lightweight Charts Integration
- Direct API usage via `createChart()` and type-safe series APIs (`ISeriesApi<'Line'>`, `ISeriesApi<'Candlestick'>`, etc.)
- Chart lifecycle: Create in first `useEffect`, store refs, clean up in return function
- Multiple price scales: Main (candles, EMAs, BB), 'volume', 'rsi' (also used for Stochastic)
- **Drawing tools**: Custom implementation using LineSeries for trendlines, horizontal lines, Fibonacci retracements, and ruler

## Data Flow

### API & Caching
- Data source: `https://tornsy.com/api/{ticker}?interval={apiInterval}`
- **localStorage cache with time-based expiration**: Different expiration per interval (2min for 1h, 30min for daily, etc.)
- Interval mapping: UI uses `'1h'|'12h'|'d1'|'w1'`, API expects `'h1'|'h12'|'d1'|'w1'`
- Type flow: `RawCandle[]` (API response) → parse to `Candle[]` (extends `CandlestickData` with volume)

### Technical Indicators
All indicators computed in-house (not from external libs):
- **EMA**: Single-pass with smoothing constant `k = 2/(period+1)`
- **Bollinger Bands**: 20-period SMA ± 2 standard deviations
- **RSI**: 14-period with smoothed gains/losses
- **Stochastic**: %K (12-period), %D (3-period SMA of %K)
- Indicators recalculate on every data change in single `useEffect`

### Pattern Detection
- **123 Pattern** (Low): Detects B.low < A.low && C.low > B.low
- **MR Pattern** (L50): Detects candle.low < 98.5% of EMA50
- Applied by setting `color: '#fbbf24'` on matching candles
- Mutually exclusive - only one pattern visible at a time

## Development Patterns

### Type Safety
- Extensive use of `lightweight-charts` types: import both value types (`CandlestickSeries`) and type-only names (`IChartApi`, `Time`, etc.)
- Custom type aliases for readability: `Interval`, `Candle`, `Drawing`, `DrawingTool`
- API calls typed with explicit response shapes: `ApiResponse = { data: RawCandle[] }`

### Event Handlers
- **Crosshair moves**: Update legend without triggering renders (use `setLegend` but refs for series data)
- **Chart clicks**: For drawing tools - extract time/price from `MouseEventParams`, handle two-click workflows
- **Keyboard shortcuts**: Arrow keys for navigation, Shift key for temporary ruler activation

### Styling Conventions
- Dark theme with specific colors: Background `#0b1120`, text `#e5e7eb`, borders `#1f2933`
- Indicator colors: EMA9 `#22c55e` (green), EMA20 `#a855f7` (purple), EMA50 `#facc15` (yellow), EMA200 `#38bdf8` (blue)
- Use existing color palette - avoid introducing new colors without reason

### Drawing Tools State
```typescript
type Drawing = DrawingLine | DrawingRuler | DrawingFibonacci
```
- Stored in `drawings` array with unique IDs
- `drawingInProgress` tracks first click (time1, price1)
- Clear all drawings when ticker/interval changes
- Fibonacci uses array of series (8 lines for different levels)

## Common Tasks

### Adding a New Indicator
1. Create ref: `const newIndicatorRef = useRef<ISeriesApi<'Line'> | null>(null)`
2. Add visibility state: `const [visibleNewIndicator, setVisibleNewIndicator] = useState(true)`
3. Create series in chart initialization `useEffect`
4. Compute data in data update `useEffect` (after line ~1013)
5. Add legend display with toggle in `chart-legend` div
6. Include in legend state type and crosshair handler

### Modifying Cache Behavior
- Cache keys: `ohlc_{ticker}_{interval}`
- Expiration times in `CACHE_EXPIRATION` object
- Cache clearing: `clearOldCache()` removes entries older than 2 hours
- Quota handling: Automatically clears old entries on `QuotaExceededError`

### Working with Chart Coordinates
- Price from Y: `series.coordinateToPrice(point.y)`
- Time from X: `chart.timeScale().coordinateToTime(point.x)`
- Use `getClickTime()` utility to extract time from `MouseEventParams`

## Build & Development

### Commands
```bash
npm run dev      # Vite dev server on http://localhost:5173
npm run build    # TypeScript check + production build
npm run lint     # ESLint with flat config
npm run preview  # Preview production build
```

### Project Structure
```
src/
  App.tsx         # Main application (1653 lines)
  App.css         # All styles
  main.tsx        # Entry point
  index.css       # Global styles
```

### Dependencies
- `lightweight-charts` v5 - TradingView charting engine
- React 19 with modern hooks
- No routing, no tests, no component library

## Gotchas

- **Don't split App.tsx** unless requested - the monolith is intentional for this project size
- **Preserve zoom/scroll**: Use `timeScale.getVisibleLogicalRange()` before updating data, restore after
- **Volume/RSI/Stochastic are mutually exclusive**: Toggling one off may need to disable others (see `toggleIndicator`)
- **Drawing tools extend time range**: Horizontal lines use `extendTimeRange()` to reach right edge
- **Pattern highlighting requires full data re-render**: Modify candle objects with color property
- **localStorage limits**: Cache implementation handles quota issues gracefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexalbuquerque87)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alexalbuquerque87)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
