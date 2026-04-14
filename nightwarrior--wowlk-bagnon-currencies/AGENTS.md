# Bagnon_Currencies - Copilot Instructions

## Project Overview

Bagnon_Currencies is a companion addon for Bagnon (WoW 3.3.5a / WotLK) that adds a currency tracker display to the bag interface. It shows player-owned currencies (e.g., Emblem of Frost) alongside the gold/silver/copper display at the bottom right of the bag window.

**Development workflow:** Make changes → User tests in-game → Feedback → Iterate

## WoW Version & API

- **Game Version:** WoW 3.3.5a (Wrath of the Lich King)
- **Interface Version:** 30300
- **Use WotLK APIs only** - no Retail or Classic Era APIs

### Key WotLK Currency APIs
- `GetCurrencyListSize()` - Returns total number of currency entries
- `GetCurrencyListInfo(index)` - Returns: name, isHeader, isExpanded, isUnused, isWatched, count, icon, itemID
- `ExpandCurrencyList(index, expand)` - Expand/collapse currency headers
- `GetCurrencyInfo(currencyID)` - Get info by currency ID
- Currency texture paths: Use icon path from `GetCurrencyListInfo()`

## Architecture

### Addon Structure (Match Bagnon Pattern)

```
Bagnon_Currencies/
├── .github/
│   └── copilot-instructions.md
├── Bagnon_Currencies.toc          # Addon metadata
├── main.lua                        # Initialization & integration
└── components/
    ├── currencyFrame.lua           # Main currency display component
    └── currencySelector.lua        # Popup selection window
```

**Follow Bagnon's component structure:**
- Each component is a class created using `Bagnon.Classy:New()`
- Components register with Bagnon's message system
- Use `BagnonFrameSettings` for persistence

### Integration Points

**Position in bag UI:** `> [currencyFrame] [moneyFrame]`
- Currency display sits between broker display (`>`) and money display (gold/silver/copper)
- Reference `moneyFrame.lua` and `brokerDisplay.lua` for positioning patterns
- Anchored to bottom right section of the bag frame

### Component: currencyFrame.lua

**Purpose:** Display selected currencies in the bag UI

**Behavior:**
- Default state (no currencies selected): Shows short placeholder text (e.g., "Currencies")
- Selected state: Shows `[amount][icon][gap][amount][icon]...` for each selected currency
- Click: Opens `currencySelector` popup directly above the frame
- Uses `CreateFrame()` with texture layers for icons
- Frame level management: Ensure popup appears above bag window

**Layout:**
- Horizontal layout with small gaps between currency entries
- Icons sized consistently with Bagnon's UI (~16-18px)
- Amount text uses `NumberFontNormalRight` font object
- Background should blend with Bagnon's frame styling

### Component: currencySelector.lua

**Purpose:** Popup window for selecting currencies to display

**Features:**
- Lists player's owned currencies from `GetCurrencyListSize()` / `GetCurrencyListInfo()`
- Checkbox per currency: `[ ] Emblem of Frost`
- Max 3 selections - grey out unchecked boxes when limit reached
- Hover tooltip on disabled checkbox: "Max currencies selected" or similar
- Save selections to SavedVariables per character

**Positioning:**
- Appears directly above `currencyFrame` when clicked
- Proper z-index/frame level to appear on top of bag window
- Click outside to close (use backdrop click detection)

### Persistence

Use character-specific SavedVariables (follow Bagnon pattern):
```lua
## SavedVariablesPerCharacter: BagnonCurrenciesSettings
```

Store:
- Selected currency IDs (array of up to 3 IDs)
- Window position if needed

### Message System Integration

Register with Bagnon's message system using:
```lua
self:RegisterMessage('FRAME_SHOW')
self:RegisterMessage('PLAYER_UPDATE')
-- etc.
```

Reference `moneyFrame.lua` for message registration patterns.

## Code Conventions

### Class System
```lua
local Bagnon = LibStub('AceAddon-3.0'):GetAddon('Bagnon')
local CurrencyFrame = Bagnon.Classy:New('Frame')
Bagnon.CurrencyFrame = CurrencyFrame
```

### Event Handling Pattern
```lua
function CurrencyFrame:OnShow()
    self:UpdateEverything()
end

function CurrencyFrame:UpdateEverything()
    self:UpdateEvents()
    self:UpdateValue()
end

function CurrencyFrame:UpdateEvents()
    self:UnregisterAllMessages()
    if self:IsVisible() then
        self:RegisterMessage('CURRENCY_DISPLAY_UPDATE')
    end
end
```

### Naming Conventions
- Frame objects: `CurrencyFrame`, `CurrencySelector`
- Methods: PascalCase (`UpdateValue`, `OnClick`)
- Variables: camelCase (`frameID`, `iconSize`)
- Constants: UPPER_SNAKE_CASE (`ICON_SIZE`, `MAX_CURRENCIES`)

### UI Styling
- Use Bagnon's existing backdrop style (see `frame.lua:24-30`)
- Font objects: `NumberFontNormalRight`, `GameFontNormal`, `GameFontHighlight`
- Color codes: Gold `|cffffd700`, Silver `|cffc7c7cf`, Copper `|cffeda55f`
- Frame levels: Parent + 3 for clickable overlays (see `moneyFrame.lua:49`)

## TOC File Structure

```lua
## Interface: 30300
## Title: Bagnon Currencies
## Author: [Your Name]
## Notes: Currency tracker for Bagnon
## RequiredDeps: Bagnon
## SavedVariablesPerCharacter: BagnonCurrenciesSettings
## Version: 1.0.0

main.lua
components\currencyFrame.lua
components\currencySelector.lua
```

## Testing Checklist

After making changes, user will verify:
- [ ] Currencies display in correct position (between `>` and gold display)
- [ ] Click opens selector popup above currency frame
- [ ] Popup appears on top of bag window (z-index correct)
- [ ] Checkboxes work and persist selections
- [ ] Max 3 currencies enforced (checkboxes grey out)
- [ ] Hover tooltip shows on disabled checkboxes
- [ ] Currency icons and amounts display correctly
- [ ] Settings persist across /reload and logout
- [ ] No lua errors in chat

## Common Pitfalls

1. **Z-index issues:** Use `:SetFrameLevel(parent:GetFrameLevel() + N)` to ensure proper layering
2. **Currency API quirks:** Some currencies may be headers or unused - filter with `isHeader == false and isUnused == false`
3. **Icon textures:** Use full path from `GetCurrencyListInfo()`, not item IDs
4. **Positioning:** Test with different bag window sizes - use relative positioning, not hardcoded coordinates
5. **Memory leaks:** Unregister events in `OnHide` - see `UpdateEvents()` pattern in moneyFrame.lua
6. **Addon loading:** Ensure `RequiredDeps: Bagnon` in TOC so Bagnon loads first

## Reference Files

- `Bagnon/components/moneyFrame.lua` - Similar display component with tooltips and positioning
- `Bagnon/components/brokerDisplay.lua` - Adjacent UI element with `< [icon] >` navigation
- `Bagnon/components/frame.lua` - Main bag frame and message system
- `Bagnon_Tooltips/` - Example companion addon structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NightWarrior) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
