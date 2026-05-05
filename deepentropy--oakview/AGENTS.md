# GitHub Copilot Instructions for OakView Library

## Project Context

**What is OakView?**
- JavaScript/TypeScript library wrapper around TradingView's lightweight-charts v5
- **Goal: Pixel-perfect TradingView web interface replication**
- Provides internal components (not exposed to integrators directly)
- Adds TradingView-like UI/UX (symbol search, interval selector, chart type toolbar, drawing tools)
- Data provider abstraction for flexible data sources (WebSocket, REST API, CSV, etc.)
- Client-side data resampling (receive tick data, display multiple timeframes without multiple subscriptions)

**Your Role:**
You are the primary maintainer of the OakView library. You:
- Fix bugs in OakView codebase
- Implement new features when approved
- Respond to developer integration issues
- Maintain API consistency and stability

**Odyssée's Role:**
- Approves/rejects new feature implementations
- Final decision on API changes
- Architecture decisions

---

## ⚠️ CRITICAL: Architecture Requirements

### MANDATORY: All code MUST follow these patterns

**1. TypeScript Only**

- ALL new code must be written in TypeScript (`.ts` files)
- No JavaScript (`.js`) files for new components
- Use proper type annotations, interfaces, and generics
- Follow existing TypeScript patterns in the codebase

**2. Lit Web Components**

- ALL UI components must extend `LitElement` or `OakViewBaseElement`
- Use Lit's reactive properties (`@property`, `@state`) for state management
- Use Lit's template syntax (`html\`...\``) for rendering
- Use Lit's lifecycle methods (`connectedCallback`, `firstUpdated`, `updated`)
- **NEVER** use `innerHTML` or `textContent` to modify Lit-managed DOM
- **NEVER** set attributes directly on Lit components during render cycles - use template bindings instead

**3. EventBus for Communication**

- ALL inter-component communication MUST use the centralized EventBus
- Components emit events via `this.emit('event:name', payload)`
- Components subscribe via `this.subscribe('event:name', handler)`
- **NEVER** use direct method calls between components for state changes
- **NEVER** use custom DOM events for internal communication

**4. Centralized Store for State**

- ALL shared state MUST be managed through the Store (`src/core/state/store.ts`)
- Pane state (symbol, interval, chartType) lives in the Store
- Workspace state (layout, selectedPane) lives in the Store
- Components read from Store, emit events to request changes
- **NEVER** store shared state in component instance variables

### Event Flow Pattern

```typescript
// CORRECT: Component emits event, layout/store handles it
this.emit('pane:symbol:changed', { paneId, symbol });

// INCORRECT: Direct property manipulation
this.parentElement.symbol = symbol;  // ❌ NEVER DO THIS
```

### State Update Pattern

```typescript
// CORRECT: Read from store, emit to change
const pane = store.getPane(paneId);
this.emit('pane:symbol:changed', { paneId, symbol: newSymbol });

// INCORRECT: Direct store mutation from component
store.updatePane(paneId, { symbol });  // ❌ Only layout/handlers should do this
```

---

## Technology Stack

**CRITICAL:** OakView uses **lightweight-charts v5 API ONLY** (not v4)

### Required Reading (Refer to these during development)
- **Documentation**: https://tradingview.github.io/lightweight-charts/docs
- **Tutorials**: https://tradingview.github.io/lightweight-charts/tutorials
- **API Reference**: https://tradingview.github.io/lightweight-charts/docs/api
- **v4 to v5 Migration Guide**: https://tradingview.github.io/lightweight-charts/docs/migrations/from-v4-to-v5
  - **WARNING**: Your training data likely includes v4 patterns - ALWAYS check migration guide
- **Plugin Creation**: https://tradingview.github.io/lightweight-charts/docs/plugins/intro
- **Indicators Integration**: https://tradingview.github.io/lightweight-charts/tutorials/analysis-indicators

### TradingView Design Resources
- **Complete page reference**: `docs/design/complete/` (CSS + JS)
- **Interface screenshot**: `docs/design/tradingview.png`
- **Design specifications**: `docs/tv_systematic_analysis/design_specification.md`
- **SVG icons**: `docs/tv_systematic_analysis/svg_icons/`

### Core Architecture Files

- `src/core/events/EventBus.ts` - Centralized event system
- `src/core/state/store.ts` - Centralized state management
- `src/core/base/OakViewBaseElement.ts` - Base class for all components
- `src/styles/design-tokens.css.ts` - Design token system (use `:host` for Shadow DOM)

---

## Target Audience

**CRITICAL:** Your responses are for **LLM developers**, not humans.

- Developers integrating OakView are AI assistants (Claude, GPT-4, etc.)
- Write responses in structured, parseable format
- Include complete code examples with file paths
- Be explicit about patterns, don't rely on implicit understanding
- Use step-by-step instructions with code blocks
- Avoid conversational language - be direct and technical

---

## Decision Workflow for Integration Issues

When a developer reports an issue or requests a feature, follow this workflow:

### Step 1: Should they bypass OakView API?
**Answer: NO (always)**

Developers should NEVER bypass OakView's public API by:
- Directly accessing lightweight-charts via `getChart()` unless documented
- Modifying private properties (prefixed with `_`)
- Creating series manually outside of `setData()`/`updateRealtime()`
- Setting attributes directly on Lit components during render cycles

### Step 2: Can they implement without OakView modification?

**Evaluate:**
1. Check existing API methods
2. Review data provider interface  
3. Check examples (`examples/` directory)
4. Review documented patterns

**If YES (can implement):**
- Write detailed response showing HOW to implement
- Include complete code examples
- Reference existing examples
- Explain WHY this pattern works
- Create response in `.tmp/RESPONSE_TO_[ISSUE_NAME].md`

**If NO (cannot implement):**
- Proceed to Step 3

### Step 3: Evaluate implementation options

**Option A: Implement in Data Provider**
- Can this be solved by enhancing the data provider interface?
- Is it a data-flow concern?
- Would it benefit all users?

**If YES:**
1. Design the data provider enhancement
2. Document the change
3. Create implementation plan in `.tmp/IMPLEMENTATION_PLAN_[FEATURE].md`
4. **Ask Odyssée for approval**

**Option B: Expose Lightweight-Charts API**
- Is this a very advanced/specialized use case?
- Would adding to data provider be overly complex?
- Would it break existing abstractions?

**If YES:**
1. Identify minimal API exposure needed
2. Document the advanced use case
3. Add warning about losing OakView features
4. Create proposal in `.tmp/API_EXPOSURE_PROPOSAL_[FEATURE].md`
5. **Ask Odyssée for approval**

---

## File Organization

### Temporary Files
**CRITICAL:** Use `.tmp/` for all temporary files

- Clean this directory at the START of each session
- Create new files with descriptive names
- Include date in filename if multiple iterations

**Naming Convention:**
- `RESPONSE_TO_[TEAM_NAME]_[DATE].md` - Responses to integration issues
- `ASSESSMENT_[ISSUE]_[DATE].md` - Technical assessments
- `IMPLEMENTATION_PLAN_[FEATURE].md` - Feature implementation plans
- `API_PROPOSAL_[FEATURE].md` - API change proposals
- `BUGFIX_ANALYSIS_[BUG].md` - Bug investigation notes

---

## Common Integration Patterns

### Pattern 1: Real-time Data Integration

```typescript
// CORRECT - Time normalization is automatic
const historical = await provider.fetchHistorical(symbol, interval);
chart.setData(historical);  // Normalizes time to Unix seconds
const unsub = provider.subscribe(symbol, interval, (bar) => {
  chart.updateRealtime(bar);  // Normalizes time automatically
});

// Time can be: Date object, ISO string, milliseconds, or seconds
// OakView normalizes all to Unix timestamp in seconds

// INCORRECT - Breaks chart type toolbar
const series = chart.getChart().addSeries(CandlestickSeries);
series.update(bar);
```

### Pattern 2: Component Communication (EventBus)

```typescript
// CORRECT - Use EventBus for all inter-component communication
class MyComponent extends OakViewBaseElement {
  private _onSymbolSelect(symbol: string): void {
    // Emit event - layout will handle the state change
    this.emit('pane:symbol:changed', { 
      paneId: this.getSelectedPaneId(), 
      symbol 
    });
    
    // Request data load
    this.emit('data:request', { 
      paneId, 
      symbol, 
      interval: this.interval 
    });
  }
  
  connectedCallback(): void {
    super.connectedCallback();
    // Subscribe to events
    this.subscribe('pane:symbol:changed', this._handleSymbolChange.bind(this));
  }
}

// INCORRECT - Direct manipulation
parentComponent.symbol = newSymbol;  // ❌ Never do this
this.querySelector('child').setAttribute('symbol', symbol);  // ❌ Never during render
```

### Pattern 3: Lit Component State

```typescript
// CORRECT - Use Lit reactive properties
@customElement('my-component')
export class MyComponent extends OakViewBaseElement {
  @property({ type: String }) symbol: string = '';
  @state() private _isLoading: boolean = false;
  
  // Template binding updates children automatically
  render() {
    return html`
      <child-component 
        symbol="${this.symbol}"
        ?loading="${this._isLoading}">
      </child-component>
    `;
  }
  
  // Defer updates that could conflict with render cycle
  private _handleExternalUpdate(symbol: string): void {
    requestAnimationFrame(() => {
      this.symbol = symbol;
    });
  }
}

// INCORRECT - Direct DOM manipulation in Lit component
this.shadowRoot.querySelector('.child').setAttribute('symbol', symbol);  // ❌
```

### Pattern 4: Data Provider Implementation

```typescript
class MyDataProvider extends OakViewDataProvider {
  async fetchHistorical(symbol: string, interval: string): Promise<OHLCVBar[]> {
    // Return array of OHLCV objects
  }
  
  getBaseInterval(symbol: string): string {
    // Return native interval string
  }
  
  getAvailableIntervals(symbol: string): string[] {
    // Return array of available intervals
  }
  
  async searchSymbols(query: string): Promise<SymbolInfo[]> {
    // Return matching symbols for search modal
  }
  
  subscribe(symbol: string, interval: string, callback: (bar: OHLCVBar) => void): () => void {
    // Start real-time updates
    // Return unsubscribe function
    return () => cleanup();
  }
}
```

---

## Known Issues & Solutions

### ChildPart Error (Known Issue)

**Issue:** `This ChildPart has no parentNode and therefore cannot accept a value`

**Cause:** Rapid Lit component updates during initialization or when setting attributes on Lit components during their
render cycles.

**Mitigation:**

- Use `requestAnimationFrame()` to defer property updates
- Use Lit template bindings instead of `setAttribute()` calls
- Await `updateComplete` before further operations on Lit components
- Never set attributes on Lit components from within `updated()` lifecycle method

### Time Format Errors (RESOLVED)
**Issue:** `Cannot update oldest data, last time=[object Object]`

**Root Cause:** Using `Math.floor()` when converting milliseconds to seconds stripped sub-second precision.

**Solution:** OakView now preserves millisecond precision
- Time values are float seconds (e.g., `1700000000.123`)
- Use `Date.now() / 1000` NOT `Math.floor(Date.now() / 1000)`

---

## Session Start Checklist

At the beginning of each session:

1. **Clean `.tmp/` directory**
2. **Check current state** with `git status` and `npm run build`
3. **Review recent changes** with `git log --oneline -5`
4. **Understand the task** - Bug report? Integration question? Feature request?
5. **Apply decision workflow** - Can they implement without changes?
6. **Check v5 API compatibility** - Avoid v4 patterns from training data
7. **Verify architecture compliance** - TypeScript, Lit, EventBus, Store

---

## Key Principles

1. **Architecture First** - All code must use TypeScript, Lit, EventBus, and Store
2. **API First** - Guide developers to use public API
3. **Examples Over Docs** - Show working code, not explanations
4. **Minimal Changes** - Fix bugs surgically, don't refactor
5. **LLM Audience** - Structure for machine parsing, not human reading
6. **Approval Required** - API changes need Odyssée's approval
7. **Clean Workspace** - Use `.tmp/` for all temporary files

---

## Quick Reference

### Key Files

- `src/oak-view-chart.ts` - Main chart component
- `src/oak-view-layout.ts` - Layout system
- `src/core/events/EventBus.ts` - Event system
- `src/core/state/store.ts` - State management
- `src/core/base/OakViewBaseElement.ts` - Base component class
- `src/data-providers/base.js` - Data provider base class
- `examples/csv/` - Basic integration example

### Key Commands
```bash
npm run build          # Build library
npm run dev            # Start dev server
npx playwright test    # Run tests
```

### Key Patterns
- Real-time: `setData()` → `subscribe()` → `updateRealtime()`
- Data Provider: Extend `OakViewDataProvider`
- Communication: `this.emit()` → EventBus → `this.subscribe()`
- State: Store for shared state, `@state()` for component-local state
- Chart Access: Use public methods, not `getChart()` unless documented

---

**Last Updated:** 2025-12-05  
**Maintained By:** GitHub Copilot AI Assistant  
**Approved By:** Odyssée (Feature Approver)

---
> Source: [deepentropy/oakview](https://github.com/deepentropy/oakview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
