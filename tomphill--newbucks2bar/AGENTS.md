# Bucks2Bar - Budget Tracker

## Project Overview

Single-page vanilla JavaScript budget tracker with real-time chart visualization. Uses Bootstrap 5 for UI and Chart.js for data visualization. No build system - runs directly in browser.

## Architecture

- **`index.html`**: Complete UI with Bootstrap tabs for Data entry and Chart visualization
- **`script.js`**: Chart initialization, real-time data synchronization, and export functionality
- **External dependencies**: Bootstrap 5.3.2 (CDN), Chart.js 4.4.0 (CDN)

## Key Patterns

### UI elements

All buttons must be a pink color.

### ID Convention for Month Inputs

All form inputs follow the pattern `income-{monthKey}` and `expense-{monthKey}` where monthKey is lowercase 3-letter abbreviation (e.g., `income-jan`, `expense-mar`). The `monthKeys` array in `script.js` maintains the canonical list.

### Real-time Chart Updates

Chart updates are event-driven via input listeners attached in `attachEventListeners()`. Any change to income/expense inputs triggers `updateChart()` which reads all 24 inputs and updates both datasets simultaneously.

### Chart Configuration

- Bar chart with two datasets: Income (green rgba(40, 167, 69)) and Expenses (red rgba(220, 53, 69))
- Y-axis uses `toLocaleString()` for currency formatting with $ prefix
- Chart instance stored in global `budgetChart` variable for updates and export

### Data Flow

1. User modifies input → `input` event fires
2. `updateChart()` iterates through all `monthKeys`
3. Reads values via `getElementById(`income-${month}`)` pattern
4. Updates `budgetChart.data.datasets[0/1].data` arrays
5. Calls `budgetChart.update()` to re-render

## Development Workflow

- **Local testing**: Open `index.html` directly in browser (no server needed)
- **No build process**: Changes to HTML/JS are immediately reflected on page reload
- **Dependencies**: All external libraries loaded via CDN - no package.json or npm

## Common Tasks

### Adding a New Month

1. Add month to `months` array (display name) and `monthKeys` array (lowercase key) in `script.js`
2. Add corresponding HTML input row in `index.html` following the existing pattern
3. Ensure inputs have classes `income-input`/`expense-input` for event binding

### Changing Chart Type

Modify `type: "bar"` in `initChart()` to another Chart.js type (line, pie, etc.). May require adjusting `options.scales` configuration.

### Export Functionality

`downloadChart()` uses Chart.js's `toBase64Image()` method to generate PNG. Downloads as `budget-chart.png`.

## Testing Notes

- Pre-filled with sample data (values in HTML inputs) for immediate visualization
- Chart initializes on `window.onload` with initial data from input `value` attributes

---
> Source: [tomphill/newbucks2bar](https://github.com/tomphill/newbucks2bar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
