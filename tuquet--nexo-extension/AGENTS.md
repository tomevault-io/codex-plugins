# Copilot / AI Assistant Instructions for nexo-ext-react

This file is a short, action-oriented guide for AI coding assistants working in this repository. Focus on making small, well-tested changes, and prefer conservative edits that match the repo's existing patterns.

Keep changes scoped and verified with the project's lint/typecheck tooling. If you make edits that touch runtime logic (db operations, assets, stores), run the package's linter and type-check commands locally and report the results.

---

## Quick Start Facts

- **Monorepo**: pnpm workspaces + Turbo (`pnpm-workspace.yaml`, `turbo.json`)
- **Chrome Extension MV3**: Background service worker architecture with message-passing
- **Primary app**: `pages/new-tab/` - AI-powered movie script generator using Gemini API
- **Tech stack**: React 19, TypeScript 5.8, Vite 6, Tailwind CSS, shadcn/ui components
- **State management**: Zustand with persist middleware + IndexedDB via Dexie wrapper
- **AI Integration**: Google GenAI SDK (`@google/genai`) for Gemini, Imagen, Veo APIs
- **Styling**: Tailwind CSS with shared config in `packages/tailwindcss-config`
- **External integration**: CapCut API server (Python Flask) for video export workflow
- **Background refactoring**: New DI Container pattern with BaseHandler (see `chrome-extension/src/background/REFACTORING_COMPLETE.md`)

---

## User Journey & Workflow

```
1. Prompts (Templates) 📝
   └─ Create/edit templates with variables & system instructions
   └─ Preview & test prompts before generation

2. Scripts (Generation) 🎬  
   └─ Select template → Fill variables → Generate via API
   └─ Edit script (with validation & AI enhancement)
   └─ Import/export JSON with versioning

3. Assets (Media) 🎨
   └─ Generate images (Imagen), videos (Veo), audio (Vbee TTS)
   └─ Link assets to specific scenes
   └─ Download/export for production
```

---

Architecture & data flow highlights

**Background Service Worker Pattern (Critical)**:
- UI pages CANNOT directly call external APIs due to Chrome extension security model
- ALL API calls (Gemini, Vbee TTS) are proxied through `chrome-extension/src/background/`
- Message protocol: `pages/new-tab` → `chrome.runtime.sendMessage()` → `core/router.ts` → handler
- Type-safe messaging: see `chrome-extension/src/background/types/messages.ts` for all message types
- Frontend wrapper: `pages/new-tab/src/services/background-api.ts` provides typed methods

**NEW: Dependency Injection Architecture** (Refactored):
- `core/di-container.ts` - Singleton service registry
- `core/base-handler.ts` - Abstract handler with automatic error handling
- `core/router.ts` - Type-safe MessageRouter using handler registry (Open/Closed Principle)
- Services: `GeminiAIService`, `ChromeSettingsService`, `VbeeTTSService`, `PageOpenerService`, `AutomationService`, `ScriptService`
- All handlers extend `BaseHandler<TMessage, TResponse>` for consistent error handling
- To add new API: 1) Create handler extending BaseHandler, 2) Register in `background/index.ts`, 3) Add message type to `types/messages.ts`

**State & Data Flow**:
- **IndexedDB** (via Dexie): `db.scripts`, `db.images`, `db.videos`, `db.audios` for persistent storage
- **Zustand stores**: `useScriptsStore` (canonical script state), `usePreferencesStore` (UI settings), `useApiKey` (secure storage)
- **Store pattern**: Use `produce` from immer for immutable updates, persist critical state with `zustand/middleware`
- **Event-driven updates**: Dispatch `window.dispatchEvent(new CustomEvent('assets-changed'))` when assets change so gallery auto-refreshes
- **No prop-drilling**: Large state (scripts, assets) should live in stores, not component props

**Key architectural decisions**:
- Scripts use three-act structure with nested acts/scenes/dialogues (see `types.ts`)
- Each scene can have generated image/video/audio linked by ID (not embedded)
- Assets stored as Blobs in IndexedDB (previously base64, migrated for performance)

**CapCut Integration** (External Workflow):
- Python Flask server (`capcut-api/`) on `localhost:9001` handles video export
- Workflow: Create draft → Upload assets (multipart/form-data) → Add media tracks → Render → Download
- Extension sends Blobs via `pages/new-tab/src/services/capcut-api.ts` (NOT background proxy - direct HTTP)
- State managed by `use-capcut-store.ts` with export progress tracking
- Server must be running before export: `cd capcut-api && python capcut_server.py`
- See `CAPCUT_INTEGRATION_SUMMARY.md` and `GALLERY_UPLOAD_TESTING_GUIDE.md` for details

---

Developer workflows & useful commands

- **Install & workspace commands**: pnpm is the package manager. Use pnpm from repo root.
- **Lint/check** for a package (e.g. `new-tab`):
  ```powershell
  pnpm -w -C pages/new-tab lint
  pnpm -w -C pages/new-tab type-check
  ```

- **Run development mode**:
  ```powershell
  # Chrome (run as administrator on Windows to avoid symlink issues)
  pnpm dev
  
  # Firefox
  pnpm dev:firefox
  ```

- **Build for production**:
  ```powershell
  pnpm build              # Chrome
  pnpm build:firefox      # Firefox
  ```

- **CapCut server** (required for video export):
  ```powershell
  cd capcut-api
  python capcut_server.py
  ```

- **After editing stores/DB**: Run package lint & type-check immediately to catch parse errors.

---

UI Design System & Component Patterns

**shadcn/ui Component Library** (`packages/ui`):
- **Base**: Built on Radix UI primitives + Tailwind CSS + class-variance-authority (CVA)
- **Style**: "new-york" preset with zinc color scheme, CSS variables for theming
- **Theme system**: next-themes for light/dark mode, CSS variables in `global.css`
- **Component structure**: All UI components in `packages/ui/lib/components/ui/`
- **Import pattern**: `import { Button, Card, Dialog } from '@extension/ui';`

**Design Tokens** (CSS Variables):
```css
/* Light mode: --background, --foreground, --primary, --secondary, etc. */
/* Dark mode: Automatic via .dark class with different HSL values */
/* Custom scrollbar: Styled for both light/dark modes */
```

**Component Variants** (using CVA):
- Button: `default | destructive | outline | secondary | ghost | link`
- Sizes: `default | sm | lg | icon`
- Always use semantic variants (don't hardcode colors)
- Example: `<Button variant="destructive" size="sm">Delete</Button>`

**Styling Conventions**:
- **Prefer UI components** over custom styled divs
- **Composition pattern**: Card = CardHeader + CardTitle + CardDescription + CardContent + CardFooter + CardAction
- **Utility-first**: Use Tailwind utilities (`flex`, `gap-2`, `rounded-xl`) for layout
- **Semantic HTML**: Use proper elements (`<button>`, `<label>`, `<fieldset>`)
- **No inline styles**: Always use className with Tailwind utilities
- **Color consistency**: Use CSS variable colors (`text-muted-foreground`, `bg-card`, `border-border`)

**Component Usage Best Practices**:
```tsx
// ✅ Good: Use UI components with variants
import { Button, Card, CardHeader, CardTitle, CardContent } from '@extension/ui';

const MyComponent = () => (
  <Card>
    <CardHeader>
      <CardTitle>Script Title</CardTitle>
    </CardHeader>
    <CardContent>
      <Button variant="outline" size="sm">Edit</Button>
    </CardContent>
  </Card>
);

export { MyComponent };

// ❌ Bad: Custom styled divs with inline colors + inline exports
export const MyComponent = () => (
  <div style={{ backgroundColor: 'white', borderRadius: '8px' }}>
    <button className="bg-blue-500">Edit</button>
  </div>
);
```

**Accessibility**:
- All interactive elements use Radix UI for keyboard navigation, focus management, ARIA attributes
- Use `<Label>` from UI package for form fields
- Dialog/AlertDialog automatically trap focus and handle escape key

**Responsive Design**:
- Mobile-first approach with Tailwind breakpoints (`sm:`, `md:`, `lg:`)
- Container sizing controlled by `usePreferencesStore` (compact/comfortable/spacious)
- See `ContainerWrapper.tsx` for dynamic container width implementation

**Icon Library**:
- Primary: lucide-react (specified in components.json)
- Import: `import { Edit, Trash2, Download } from 'lucide-react';`
- Sizing: Icons auto-size to `size-4` within Button components via `[&_svg]:size-4`

---

## DualModeEditor Pattern (Generic Entity Editor)

**CRITICAL: Always use `DualModeEditor` for create/edit dialogs with JSON support.**

The codebase provides a generic `DualModeEditor<T>` component for any entity that needs:
- Visual form editor (primary UX)
- JSON editor mode (advanced/power users)
- Mode switching via button
- Type-safe validation

**Architecture**:
```
src/components/common/dual-mode-editor.tsx  ← Generic component
src/utils/{entity}-validation.ts             ← Validation logic
src/constants/{entity}-defaults.ts           ← Default templates
src/components/{entity}/{entity}-form.tsx    ← Specific implementation
src/components/{entity}/{entity}-editor.tsx  ← Visual form UI
```

**Usage Pattern** (see `components/prompts/prompt-form.tsx` as reference):

```tsx
// 1. Create validation utilities (utils/prompt-validation.ts)
export const validatePromptJSON = (jsonString: string): ValidationResult => {
  try {
    const parsed = JSON.parse(jsonString);
    // Validate required fields
    if (!parsed.title?.trim()) return { isValid: false, error: 'Title required' };
    if (!parsed.category) return { isValid: false, error: 'Category required' };
    return { isValid: true, data: parsed };
  } catch (error) {
    return { isValid: false, error: 'Invalid JSON format' };
  }
};

// 2. Create constants (constants/prompt-defaults.ts)
export const DEFAULT_PROMPT_TEMPLATE = {
  title: 'New Prompt',
  category: 'general',
  prompt: 'Your prompt template here...',
  // ... all fields with defaults
};

// 3. Use DualModeEditor in form component
import { DualModeEditor } from '@src/components/common/dual-mode-editor';
import { PromptEditor } from '@src/components/prompts';
import { DEFAULT_PROMPT_TEMPLATE } from '@src/constants/prompt-defaults';
import { validatePromptJSON } from '@src/utils/prompt-validation';

interface PromptFormData extends Record<string, unknown> {
  title: string;
  category: string;
  prompt: string;
  // ... other fields
}

const PromptForm = ({ open, onOpenChange, onSubmit, initialData, mode }) => (
  <DualModeEditor<PromptFormData>
    open={open}
    onOpenChange={onOpenChange}
    onSubmit={onSubmit}
    initialData={initialData as unknown as PromptFormData}
    defaultTemplate={DEFAULT_PROMPT_TEMPLATE as PromptFormData}
    mode={mode}
    validateJSON={validatePromptJSON}
    renderUIEditor={({ 
      open, 
      onOpenChange, 
      data, 
      onSave, 
      title, 
      description,
      onSwitchToJSON  // CRITICAL: Pass to visual editor for switch button
    }) => (
      <PromptEditor
        open={open}
        onOpenChange={onOpenChange}
        initialData={data as unknown as PromptRecord}
        onSave={onSave}
        title={title}
        description={description}
        onSwitchToJSON={onSwitchToJSON}  // CRITICAL: Enable switch to JSON
      />
    )}
    title={{
      create: 'Create New Prompt',
      edit: 'Edit Prompt',
      createJSON: 'Create New Prompt (JSON)',
      editJSON: 'Edit Prompt (JSON)',
    }}
    description={{
      ui: 'Use form fields below to configure. Switch to JSON mode for advanced editing.',
      json: 'Edit the JSON object above.',
    }}
    fieldsToExclude={['id', 'createdAt', 'updatedAt']}
    helpText={{
      requiredFields: 'Required fields: title, category, prompt.',
      validationRules: 'Valid categories: script-generation, image-generation, ...',
    }}
  />
);
```

**Visual Editor Component Pattern** (PromptEditor, ScriptEditor, etc.):

```tsx
interface PromptEditorProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  initialData: PromptRecord;
  onSave: (data: Partial<PromptRecord>) => void;
  title: string;
  description?: string;
  onSwitchToJSON?: () => void;  // CRITICAL: Required for mode switch button
}

const PromptEditor = ({ 
  open, 
  onOpenChange, 
  initialData, 
  onSave, 
  title, 
  description,
  onSwitchToJSON  // CRITICAL: Must accept and use
}) => {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle className="flex items-center justify-between">
            <span>{title}</span>
            {onSwitchToJSON && (  // CRITICAL: Show switch button
              <Button variant="ghost" size="sm" onClick={onSwitchToJSON} className="gap-2">
                <FileJson className="size-4" />
                Switch to JSON Editor
              </Button>
            )}
          </DialogTitle>
          {description && <DialogDescription>{description}</DialogDescription>}
        </DialogHeader>
        
        {/* Form fields here */}
        
        <DialogFooter>
          <Button onClick={handleSave}>Save</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
};
```

**Benefits**:
- **Code reuse**: ~150-200 lines saved per entity (240 → 85 lines for PromptForm)
- **Consistency**: Same UX across all entity editors
- **Type safety**: TypeScript generics ensure compile-time validation
- **Maintainability**: Bug fixes/features apply to ALL entity editors
- **Extensibility**: Add new entity editor in ~20 lines (validation + constants + form wrapper)

**When to Use**:
- ✅ ANY create/edit dialog for structured data (Prompts, Scripts, Settings, Characters, Templates)
- ✅ When users need both visual form AND raw JSON editing
- ✅ When data has complex nested structure (preprocessing, modelSettings, etc.)

**When NOT to Use**:
- ❌ Simple single-field inputs (use plain Dialog + Input)
- ❌ Pure text/markdown editors (use Textarea directly)
- ❌ Non-structured data (images, videos, binary data)

**Checklist for Adding New Entity Editor**:
1. ☑ Create `utils/{entity}-validation.ts` with `validate{Entity}JSON()` function
2. ☑ Create `constants/{entity}-defaults.ts` with `DEFAULT_{ENTITY}_TEMPLATE`
3. ☑ Create/update visual editor component with `onSwitchToJSON` prop
4. ☑ Add switch button in DialogTitle: `{onSwitchToJSON && <Button onClick={onSwitchToJSON}>Switch to JSON</Button>}`
5. ☑ Wrap in `DualModeEditor<{Entity}FormData>` with all required props
6. ☑ Ensure `{Entity}FormData extends Record<string, unknown>` for generic constraint
7. ☑ Test: Visual mode → JSON mode → Visual mode (data persistence)
8. ☑ Test: Validation errors in JSON mode
9. ☑ Test: Create new vs Edit existing

---

## Patterns & Conventions to Follow

**ESLint Rules (Strictly Enforced)**:
- **`import-x/exports-last`**: ALL exports MUST be at the end of file
  ```typescript
  // ✅ Good
  const helper = () => {};
  const Component = () => {};
  export { helper, Component }; // At end
  
  // ❌ Bad
  export const helper = () => {}; // Export inline
  const Component = () => {};
  ```
- **`import-x/first`**: Imports MUST be at top, before any code
- **`import-x/order`**: Alphabetical, groups: builtin → external → internal → parent → sibling → index → type
- **`import-x/newline-after-import`**: Blank line after import block required
- **`import-x/consistent-type-specifier-style`**: Use `import type` for type-only imports
- **`@typescript-eslint/consistent-type-imports`**: Type imports must use `import type`
- **`@typescript-eslint/consistent-type-exports`**: Type exports must use `export type`
- **`func-style`**: Use function expressions, not declarations (const fn = () => {})
- **`arrow-body-style`**: Omit braces when possible (const fn = () => value)
- **`prefer-const`**: Use const unless reassignment needed
- **`no-var`**: Never use var

**Code Patterns**:
- Small helper functions: utilities that mutate nested objects commonly use `setNestedValue(obj, path, value)` pattern (see `useScriptsStore.ts`). Keep these pure-ish and well-typed.
- Error messages and UI text sometimes use Vietnamese strings — preserve locale in user-visible strings unless intentionally changing UX.
- IndexedDB usage: use the `db` wrapper methods (`toArray`, `add`, `put`, `delete`, `where(...).delete`) and ensure to clear dependent assets on script delete.

---

## SOLID Principles in Practice

This codebase demonstrates strong adherence to SOLID for extensibility and maintainability. Follow these patterns when adding features:

### **S - Single Responsibility Principle**
Each module has ONE reason to change:

```typescript
// ✅ GOOD: Separated concerns
// background/router.ts - ONLY routing logic
const messageRoutes = { 
  GENERATE_SCRIPT: handleGenerateScript,
  GENERATE_IMAGE: handleGenerateSceneImage,
};

// gemini-api-handler.ts - ONLY API calls
export const handleGenerateScript = async (message) => { ... }

// settings-handler.ts - ONLY settings persistence
export const handleGetSettings = async () => { ... }

// ❌ BAD: Mixed responsibilities
const handleGenerateScriptAndSave = async (message) => {
  const script = await callAPI(message);      // API concern
  await chrome.storage.local.set({ script }); // Storage concern
  await navigateToScript(script.id);          // Navigation concern
};
```

**Real example:** `use-script-generation.ts` hook encapsulates generation logic but delegates to:
- `generateScript()` for API calls
- `addScript()` for store updates
- `navigate()` for routing
- `toast()` for notifications

### **O - Open/Closed Principle**
Open for extension, closed for modification:

```typescript
// ✅ GOOD: Extensible router pattern
// Adding new handler requires ZERO changes to router logic
const messageRoutes: { [key: string]: MessageHandler } = {
  GENERATE_SCRIPT: handleGenerateScript,
  GENERATE_IMAGE: handleGenerateSceneImage,
  // Add new: just register here, router.ts unchanged
  NEW_FEATURE: handleNewFeature,
};

// ❌ BAD: if/else chain requires modification
const router = (message) => {
  if (message.type === 'GENERATE_SCRIPT') return handleGenerateScript();
  if (message.type === 'GENERATE_IMAGE') return handleGenerateSceneImage();
  // Adding new feature requires modifying this function
};
```

**Real example:** Background router uses dictionary pattern - adding new API endpoints requires only:
1. Add handler function
2. Register in `messageRoutes` object
3. Zero changes to router logic

### **L - Liskov Substitution Principle**
Subtypes must be substitutable for base types:

```typescript
// ✅ GOOD: All handlers conform to MessageHandler signature
type MessageHandler = (
  message: BackgroundMessage,
  sender: chrome.runtime.MessageSender
) => Promise<BackgroundResponse>;

// Any handler can be substituted
const handleGenerateScript: MessageHandler = async (message, sender) => { ... }
const handleGenerateImage: MessageHandler = async (message, sender) => { ... }

// ❌ BAD: Inconsistent signatures break substitutability
const handleA = async (message) => { ... }              // Missing sender
const handleB = async (message, sender, extra) => { ... } // Extra param
```

**Real example:** All background handlers implement same interface - router doesn't care about implementation details, only interface contract.

### **I - Interface Segregation Principle**
Clients shouldn't depend on interfaces they don't use:

```typescript
// ✅ GOOD: Specific message types
interface GeminiGenerateScriptMessage {
  type: 'GENERATE_SCRIPT';
  payload: {
    prompt: string;
    language: 'en-US' | 'vi-VN';
    apiKey: string;
    modelName: string;
    temperature: number;
    // ... only fields needed for script generation
  };
}

interface GeminiGenerateImageMessage {
  type: 'GENERATE_SCENE_IMAGE';
  payload: {
    prompt: string;
    aspectRatio: AspectRatio;
    // ... only fields needed for image generation
  };
}

// ❌ BAD: Fat interface with unused fields
interface AllInOneMessage {
  type: string;
  prompt?: string;
  language?: string;
  apiKey?: string;
  aspectRatio?: string;
  temperature?: number;
  // ... image handler doesn't need temperature
  // ... script handler doesn't need aspectRatio
}
```

**Real example:** `chrome-extension/src/background/types/messages.ts` uses discriminated unions - each message type has ONLY the fields it needs.

### **D - Dependency Inversion Principle**
Depend on abstractions, not concretions:

```typescript
// ✅ GOOD: Hooks depend on abstractions
export const useScriptGeneration = () => {
  const { apiKey } = useApiKey();           // Abstract: any key provider
  const addScript = useScriptsStore(s => s.addScript); // Abstract: any store
  const navigate = useNavigate();            // Abstract: any router
  
  // Logic doesn't care HOW these are implemented
};

// ❌ BAD: Direct dependency on implementation
export const useScriptGeneration = () => {
  const apiKey = chrome.storage.local.get('apiKey'); // Concrete storage
  const addScript = (script) => {
    db.scripts.add(script);                 // Concrete database
  };
};
```

**Real example:** `background-api.ts` abstracts chrome message passing:
- UI components call `generateScript(params)` (abstraction)
- Implementation uses `chrome.runtime.sendMessage()` (concrete)
- Future: could swap to WebSocket without changing callers

### **When to Apply SOLID**

**DO apply when:**
- Adding new API handlers (use router pattern)
- Creating new hooks (separate concerns)
- Adding validation (create dedicated validators)
- Implementing new stores (single responsibility per store)

**DON'T over-engineer:**
- Simple utility functions (formatDate, clamp, etc.)
- Component-specific helpers (no reuse needed)
- One-off scripts or migrations

---

## Folder Structure & Organization

The `pages/new-tab/src/` directory follows a **feature-based organization** pattern aligned with SOLID Single Responsibility Principle. Each folder has ONE clear purpose.

### **Project Structure**

```
pages/new-tab/src/
├── components/
│   ├── common/                   # Shared UI components across features
│   │   └── app-setting-modal.tsx
│   │
│   ├── layout/                   # Layout-specific components
│   │   ├── container-wrapper.tsx
│   │   └── header.tsx
│   │
│   ├── gallery/                  # Asset gallery feature
│   │   └── [gallery components]
│   │
│   ├── prompts/                  # Prompt template management
│   │   └── [prompt components]
│   │
│   └── script/                   # Script management feature (MAIN FEATURE)
│       ├── forms/                # User input forms
│       │   ├── create-form.tsx
│       │   ├── manual-creation-form.tsx
│       │   └── template-selector.tsx
│       │
│       ├── modals/               # Modal dialogs
│       │   ├── image-generation.tsx
│       │   ├── json-import.tsx
│       │   ├── model-settings.tsx
│       │   └── tts-export.tsx
│       │
│       ├── display/              # Read-only display components
│       │   ├── asset-display.tsx
│       │   ├── content.tsx
│       │   ├── header.tsx
│       │   ├── loader.tsx
│       │   └── responsive-detail-layout.tsx
│       │
│       ├── cards/                # Card components (list items, scene items)
│       │   ├── scene.tsx
│       │   ├── scene-asset.tsx
│       │   ├── tts-asset.tsx
│       │   └── vbee-project-asset.tsx
│       │
│       ├── generation/           # AI generation forms
│       │   ├── ai-generation-tab.tsx
│       │   ├── generation-form.tsx
│       │   ├── json-import-tab.tsx
│       │   ├── plot-suggestions.tsx
│       │   └── variable-inputs.tsx
│       │
│       ├── actions/              # Action buttons & triggers
│       │   ├── action-button.tsx
│       │   └── create-button.tsx
│       │
│       ├── settings/             # Settings & configuration
│       │   ├── advanced-model-options.tsx
│       │   └── model-settings.tsx
│       │
│       └── ui/                   # Reusable UI elements (script-specific)
│           ├── audio-player.tsx
│           ├── editable-field.tsx
│           └── creatable-select.tsx
│
├── pages/                        # Route pages (React Router)
│   ├── gallery/                  # Asset gallery routes
│   ├── home/                     # Home page
│   ├── prompts/                  # Prompt management routes
│   └── script/                   # Script CRUD routes
│       ├── create.tsx            # Create script page
│       ├── detail.tsx            # Script detail page
│       ├── list.tsx              # Script list page
│       └── breakpoints.ts        # Responsive breakpoints
│
├── hooks/                        # Custom React hooks
│   ├── use-assets.ts             # Asset generation hooks
│   ├── use-script-generation.ts  # Script generation logic
│   └── [other hooks]
│
├── stores/                       # Zustand state management
│   ├── use-scripts-store.ts      # Canonical script state
│   ├── use-preferences-store.ts  # UI preferences
│   ├── use-api-key.ts            # API key storage
│   └── [other stores]
│
├── services/                     # API communication layer
│   ├── background-api.ts         # Background service worker proxy
│   ├── gemini-service.ts         # Gemini API wrapper
│   ├── vbee-service.ts           # Vbee TTS API wrapper
│   └── api-errors.ts             # Error handling
│
├── utils/                        # Utility functions
│   ├── dialogue-validator.ts     # TTS quality validation
│   └── prompt-builder.ts         # Template processing
│
├── constants/                    # Constants & configuration
│   ├── index.ts                  # Main constants (models, genres, etc.)
│   └── script-generation.ts      # Script generation specific
│
└── types/                        # TypeScript type definitions
    ├── index.ts                  # Main types (ScriptStory, Scene, etc.)
    └── script-generation.ts      # Generation-specific types
```

### **Organizational Principles**

1. **Feature-based grouping**: Script components are grouped by responsibility (forms, modals, display), not by file type
2. **SOLID Single Responsibility**: Each folder has ONE clear purpose
   - `forms/` = User input
   - `modals/` = Dialog overlays
   - `display/` = Read-only presentation
   - `cards/` = List item components
   - `actions/` = Action triggers
   - `settings/` = Configuration UI
   - `ui/` = Reusable UI primitives

3. **Scalability**: New components have obvious home
   - New form component? → `script/forms/`
   - New modal dialog? → `script/modals/`
   - New card type? → `script/cards/`

4. **Avoid deep nesting**: Maximum 2 levels under `components/` (feature → responsibility)

### **Where to Add New Components**

| Component Type | Location | Example |
|----------------|----------|---------|
| Script form | `components/script/forms/` | create-form.tsx |
| Modal dialog | `components/script/modals/` | image-generation.tsx |
| Read-only display | `components/script/display/` | content.tsx |
| List/card item | `components/script/cards/` | scene.tsx |
| Action button | `components/script/actions/` | action-button.tsx |
| Settings UI | `components/script/settings/` | model-settings.tsx |
| Reusable UI element | `components/script/ui/` | editable-field.tsx |
| AI generation feature | `components/script/generation/` | ai-generation-tab.tsx |
| Cross-feature component | `components/common/` | app-setting-modal.tsx |
| Route page | `pages/{feature}/` | pages/script/detail.tsx |

### **Import Path Guidelines**

- **Absolute imports** for cross-feature: `@src/components/script/forms/create-script-form`
- **Relative imports** within same feature: `../modals/image-generation-modal` (from script/display/)
- **Always follow ESLint import order**: builtin → external → internal → parent → sibling → index → type

### **Anti-Patterns to Avoid**

❌ **Flat structure with 25+ files** (old pattern before refactor)
❌ **Mixing concerns** (forms + modals + display in one folder)
❌ **Deep nesting** (components/script/forms/user-input/fields/)
❌ **Type-based folders** (components/containers/, components/presentational/)

---

## Files of interest (quick map)

**Chrome Extension Background (Critical for API calls)**:
- chrome-extension/src/background/
  - **Core Architecture** (NEW - DI Pattern):
    - index.ts — Service registration and handler initialization
    - core/di-container.ts — Singleton service registry
    - core/base-handler.ts — Abstract handler base class
    - core/router.ts — Type-safe MessageRouter (Open/Closed Principle)
  - **Services** (business logic):
    - services/gemini-ai-service.ts — Gemini/Imagen/Veo API calls
    - services/chrome-settings-service.ts — Settings persistence
    - services/vbee-tts-service.ts — Vbee TTS API
    - services/page-opener-service.ts — Tab/window management
    - services/automation-service.ts — UI automation
    - services/script-service.ts — Script operations
  - **Handlers** (message processing):
    - handlers/generate-script-handler.ts — Script generation
    - handlers/generate-image-handler.ts — Image generation
    - handlers/generate-video-handler.ts — Video generation
    - handlers/vbee-create-project-handler.ts — TTS project creation
    - handlers/settings-handlers.ts — Get/save settings
    - [12 other handlers] - See `handlers/` directory
  - **Types & Schemas**:
    - types/messages.ts — Type-safe message protocol (BackgroundMessage/BackgroundResponse unions)
    - schemas/script-schema.ts — JSON schemas using Type enum for structured generation
  - **Old files** (deprecated, see REFACTORING_COMPLETE.md):
    - ~~gemini-api-handler.ts~~ → Refactored to services/
    - ~~vbee-api-handler.ts~~ → Refactored to services/
    - ~~settings-handler.ts~~ → Refactored to services/

**Frontend (New Tab Page)**:
- pages/new-tab/src/
  - NewTab.tsx — React Router HashRouter + Routes (Prompts, Scripts, AssetGallery pages)
  - db.ts — Re-exports shared database from @extension/database package
  - services/background-api.ts — **Frontend wrapper for background communication** (sendMessage, unwrapResponse)
  - stores/use-scripts-store.ts — Canonical zustand store for scripts (init, reloadFromDB, add, save, delete)
  - stores/use-preferences-store.ts — UI preferences (theme, containerSize, compactMode)
  - stores/use-api-key.ts — API key storage with persist middleware
  - stores/use-model-settings.ts — AI model configuration (text, image, video, TTS models)
  - hooks/use-assets.ts — Asset generation hooks (image, video, audio)
  - utils/dialogue-validator.ts — Validates dialogue lines for TTS quality (no stage directions)
  - utils/prompt-builder.ts — Template processing, variable replacement, validation
  - **Generic Components** (Reusable patterns):
    - components/common/dual-mode-editor.tsx — **Generic dual-mode editor** (UI + JSON modes) with TypeScript generics
    - utils/prompt-validation.ts — Validation logic for prompts (extract pattern for other entities)
    - constants/prompt-defaults.ts — Default templates/constants (extract pattern for other entities)
  - components/prompts/ — **Reference implementation of DualModeEditor pattern**
    - prompt-form.tsx — Wrapper using DualModeEditor<PromptFormData> (85 lines vs 240 monolithic)
    - prompt-editor.tsx — Visual form editor with onSwitchToJSON prop
  - components/script/* — Script editing components: EditableField, SceneCard, CreationForm
  - pages/prompts/* — Prompt template management with variable support
  - pages/script/* — Script list and CRUD operations

**Options Page** (Settings UI):
- pages/options/src/
  - Options.tsx — Entry point with ThemeProvider + Toaster
  - components/layout/
    - OptionsLayout.tsx — Tab-based navigation (General, AI Models, Display, TTS, Advanced)
    - OptionsHeader.tsx — Header with logo + theme toggle
    - OptionsFooter.tsx — Simple footer
  - components/tabs/
    - GeneralTab.tsx — API keys (Google AI, Vbee), connection test, status badges
    - AIModelsTab.tsx — Model selection, temperature, topP sliders
    - DisplayTab.tsx — Theme, layout preferences, font scale, container size
    - TTSTab.tsx — Voice selection, speaking rate, pitch, volume
    - AdvancedTab.tsx — Data export/import, storage usage, cache management, reset

**Shared Packages**:
- packages/database — Centralized Dexie database singleton (scripts, prompts, images, videos, audios)
- packages/ui — Shared shadcn/ui components (Button, Dialog, Card, Alert, etc.)
- packages/shared — Common utilities and types
- packages/tailwindcss-config — Shared Tailwind configuration

---

## Common Pitfalls & Quick Fixes

- **API calls in UI pages will fail silently**: NEVER call external APIs directly from UI pages. Always use `pages/new-tab/src/services/background-api.ts` methods which proxy through background service worker.
- **Message protocol types**: When adding new API calls, update `chrome-extension/src/background/types/messages.ts` with new message/response types, add handler in appropriate file, register in `router.ts`.
- **DO NOT import deprecated handlers**: After refactoring, old files like `gemini-api-handler.ts`, `vbee-api-handler.ts`, `settings-handler.ts` are deprecated. Use new DI pattern: Create handler extending `BaseHandler`, register in `background/index.ts`. See `REFACTORING_COMPLETE.md`.
- **ALWAYS use DualModeEditor for entity editors**: When creating create/edit dialogs for structured data (Prompts, Scripts, Settings, etc.), use the generic `DualModeEditor<T>` component pattern. DO NOT write monolithic form components with duplicate JSON/UI logic. See `components/prompts/prompt-form.tsx` as reference implementation.
- **Missing switch button in visual editor**: Visual editor components MUST accept and use `onSwitchToJSON?: () => void` prop. Add switch button in DialogTitle: `{onSwitchToJSON && <Button onClick={onSwitchToJSON}><FileJson /> Switch to JSON Editor</Button>}`
- Malformed or duplicate files (esp. store files) cause TypeScript parse errors that cascade into many lint errors. If you see "Parsing error: ';' expected" inspect the file for duplicated content or stray backticks.
- After editing a store, re-run `pnpm -w -C pages/new-tab lint` — many consumers import the store and will fail to parse if the store doesn't compile.
- Remember to dispatch `assets-changed` after deleting assets or clearing DB so galleries update.
- **Schema definitions**: Use `Type` enum from `@google/genai` (not JSON.parse) for structured generation schemas in `background/schemas/`.
- **Zustand updates**: Always use `produce` from immer for nested state updates in stores to maintain immutability.

---

## Critical Performance Patterns

**Asset Generation Queue** (prevents quota exhaustion):
- Limit concurrent API calls to avoid rate limits
- User can spam "Generate Image" → queue system needed
- See `use-assets.ts` for generation hooks

**IndexedDB Best Practices**:
- Store assets as **Blobs**, not base64 (performance!)
- Use `db.scripts.toArray()` for list views
- Clear dependent assets on delete: `db.images.where({ scriptId }).delete()`
- Event-driven updates: `window.dispatchEvent(new CustomEvent('assets-changed'))`

**Store Reload Pattern**:
```typescript
// ✅ After bulk DB operations, reload store from DB
const reloadFromDB = useScriptsStore.getState().reloadFromDB;
await db.scripts.bulkAdd(items);
await reloadFromDB(); // Triggers re-render

// ❌ Don't rely on store state after direct DB mutation
await db.scripts.add(item);
// Store is now stale!
```

**Validation on Edit**:
- Dialogue lines: Check for stage directions `(action)` or `[note]`
- See `utils/dialogue-validator.ts` for TTS quality enforcement
- Warnings appear in `EditableField` component

---

## Known Improvement Opportunities

**Error Recovery** (Missing):
- No retry logic for API failures
- Long operations (video gen) can timeout without recovery
- Add exponential backoff wrapper for critical API calls

**Progress Tracking** (Missing):
- Video generation polls every 10s but shows no progress %
- User has no ETA or cancel option
- Implement progress events via message protocol

**Undo/Redo** (Missing):
- Script edits are permanent
- AI enhancements cannot be reverted
- History stack needed in `useScriptsStore`

**Export/Import** (Needs versioning):
- No schema version in exports
- Cannot handle breaking changes between versions
- Add migration system for legacy imports

---

## When to Ask for Clarification

- If the change touches cross-package public APIs (packages/*), ask before rearranging exports.
- If you need to modify UX text in Vietnamese, confirm if locale files or localization strategy should be updated.
- When adding new message types, confirm handler registration in `router.ts`

---

## If You Modify or Add Tests

- Run package-level type-check and lint. There is no single test runner configured that we can rely on; prefer small type-safe unit tests in the package where appropriate.

---

## Need More Info?

If anything above is unclear, point to the file you want to change and I'll fetch the minimal surrounding files and note exactly which commands to run to validate changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tuquet)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tuquet)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
