# AGENTS.md - Agent Guidelines for Llama Async Proxy Dashboard

This document provides comprehensive guidelines for AI coding assistants working on the **Llama Async Proxy Dashboard** project. Following these guidelines ensures consistent, maintainable, and high-quality code contributions.

## 1. Introduction

### 1.1 Purpose of This Document

This document serves as the authoritative reference for all AI agents contributing to the Llama Async Proxy Dashboard codebase. It establishes coding standards, architectural patterns, communication protocols, and best practices that must be followed to maintain project integrity and developer productivity. By adhering to these guidelines, AI assistants can produce code that seamlessly integrates with existing patterns, passes all tests, and meets the project's quality standards.

The guidelines contained herein are not optional recommendations but strict requirements that enforce the project's architectural decisions. Deviation from these patterns creates technical debt, introduces bugs, and makes the codebase harder to maintain. When in doubt about whether a pattern or approach is acceptable, refer to this document before implementing any solution.

### 1.2 Target Audience

This documentation is specifically designed for AI coding assistants, including Claude Code, GitHub Copilot, and other LLM-based development tools. Human developers should also use this as a reference, but the guidelines are written with AI agent capabilities and limitations in mind. The document assumes AI agents have access to the codebase, can read and write files, execute commands, and understand the project's technology stack.

AI agents working on this project should treat these guidelines as hard constraints. Unlike human developers who might use judgment to determine when exceptions are appropriate, AI agents should follow these rules strictly unless explicitly told otherwise by a human. When uncertain about whether a particular approach complies with these guidelines, the conservative approach is to follow the documented pattern.

### 1.3 What Makes This Project Unique

The Llama Async Proxy Dashboard is a vanilla JavaScript application that deliberately avoids modern frontend frameworks like React, Vue, or Angular. This architectural decision prioritizes simplicity, reduced bundle sizes, and straightforward debugging over developer convenience or ecosystem popularity. The project uses an event-driven architecture with Socket.IO for all communication between client and server, eliminating REST API complexity and enabling real-time updates.

The project also employs a server-owned state architecture where the backend maintains the authoritative state and broadcasts changes to all connected clients. This pattern differs from typical client-side state management approaches and requires careful attention to state synchronization and event handling. The frontend components are purely reactive, receiving state updates and updating the DOM directly without virtual DOM reconciliation or component re-rendering.

Another unique aspect is the project's integration with llama.cpp's router mode, which allows a single server process to manage multiple models with automatic loading, unloading, and LRU-based eviction. Understanding this integration is crucial for implementing features related to model management, metrics collection, and configuration handling.

### 1.4 Critical Rules Summary

The following rules are non-negotiable and must be followed at all times:

- **NEVER use React, Vue, Angular, or any frontend framework** - The project uses pure vanilla JavaScript
- **NEVER use TypeScript** - This is a pure JavaScript project
- **NEVER use npm** - Always use pnpm for package management
- **NEVER write outside the project directory** - All file operations must remain within the project
- **Keep files under 200 lines** - Split large files into smaller, focused modules
- **Single responsibility per module** - Each component/class should do one thing well
- **No memory leaks** - Always clean up subscriptions and event listeners
- **Use Socket.IO for all communication** - No REST API endpoints
- **Direct DOM updates** - No virtual DOM or re-rendering patterns
- **Always show user feedback** - Use notifications for all user actions

## 2. Project Overview

### 2.1 What is Llama Async Proxy Dashboard?

The Llama Async Proxy Dashboard is a web-based management interface for llama.cpp models running in router mode. Llama.cpp is a high-performance inference engine for large language models, and its router mode enables a single server process to manage multiple models simultaneously. The dashboard provides a unified interface for discovering models, loading and unloading them, monitoring performance metrics, and configuring system settings.

The application operates as a single-page application (SPA) where all frontend logic runs in the browser using vanilla JavaScript. The backend is built with Node.js and Express, providing Socket.IO endpoints for real-time bidirectional communication. All state is maintained on the server, and clients receive broadcasts whenever relevant state changes occur. This architecture ensures that all connected clients see a consistent view of the system state without requiring polling or manual refresh.

The dashboard is designed for production deployment where reliability and performance are critical. It includes comprehensive error handling, graceful degradation for missing data, and real-time feedback for all user actions. The interface is built with accessibility and usability in mind, providing clear visual feedback for all interactions and maintaining responsiveness even during heavy model operations.

### 2.2 Key Technologies Used

The project uses a carefully selected technology stack optimized for simplicity, performance, and maintainability:

- **Runtime**: Node.js version 18 or higher is required for all operations
- **Server Framework**: Express 5.x provides the HTTP server infrastructure
- **Real-time Communication**: Socket.IO 4.x enables bidirectional event-based communication
- **Database**: better-sqlite3 with SQLite provides synchronous, high-performance local storage
- **Frontend**: Vanilla JavaScript with no frameworks or bundlers
- **Model Parsing**: @huggingface/gguf for reading GGUF model metadata
- **Testing**: Jest 30.x for comprehensive unit testing
- **Code Quality**: ESLint 9.x for linting and Prettier 3.x for formatting
- **Package Management**: pnpm for all dependency operations

### 2.3 Architecture Summary

The application follows a layered architecture with clear separation of concerns:

1. **Server Layer** (Express + Socket.IO): Handles HTTP requests and WebSocket connections, manages application state, and coordinates with llama.cpp processes
2. **Database Layer** (SQLite): Persists configuration, logs, and model metadata using a repository pattern
3. **Handler Layer**: Modular Socket.IO event handlers organized by domain (models, metrics, logs, configuration)
4. **Frontend Layer** (Vanilla JS): Event-driven UI with direct DOM manipulation, no virtual DOM

The frontend uses a Component-Controller pattern where each page has a controller that manages state subscriptions and a component that handles rendering and user interactions. Components extend a base Component class that provides event delegation, DOM manipulation helpers, and lifecycle management.

### 2.4 Communication Pattern (Socket.IO Only)

All communication between client and server occurs exclusively through Socket.IO events. There are no REST API endpoints. This design choice simplifies the architecture, enables real-time updates, and ensures that all clients receive the same state information simultaneously.

The communication pattern follows a request-response model where clients send events with a payload and receive responses with a standardized format. The server also broadcasts events to all connected clients when state changes occur. This broadcast mechanism is the primary way the frontend receives updates about model status changes, new log entries, and metric updates.

All Socket.IO responses follow a consistent format:

```javascript
{
  success: boolean,      // true if request succeeded, false otherwise
  data?: any,            // response payload when successful
  error?: {              // error details when unsuccessful
    message: string,
    code?: string
  },
  timestamp: string      // ISO timestamp of the response
}
```

## 3. Technology Stack Quick Reference

### 3.1 Core Runtime and Server

The server-side of the application is built on Node.js, providing a consistent JavaScript runtime across development and production environments. Node.js version 18 or higher is required due to modern ECMAScript features and improved performance characteristics used throughout the codebase.

Express 5.x serves as the HTTP server framework, handling static file serving and providing the foundation for Socket.IO integration. Express 5 was chosen for its improved performance, better error handling, and reduced middleware overhead compared to version 4. Routes are minimal since most functionality is handled through Socket.IO events.

Socket.IO 4.x provides the real-time communication layer. The library handles WebSocket connections, fallback to HTTP long-polling, automatic reconnection, and room-based message routing. All Socket.IO functionality is encapsulated in a dedicated client module, ensuring consistent usage patterns across the frontend.

### 3.2 Database and Storage

better-sqlite3 provides synchronous SQLite access with better performance than asynchronous alternatives. The synchronous API aligns with the event-driven architecture, avoiding callback complexity and ensuring consistent state management. SQLite was chosen for its reliability, zero-configuration requirements, and excellent performance for read-heavy workloads typical of dashboard applications.

The database schema is defined in code using SQL statements that execute on application startup. A repository pattern abstracts database operations, making the data layer testable and maintaining separation between data access and business logic. Each repository handles a specific domain, such as models, logs, configuration, or metrics.

### 3.3 Frontend Technology

The frontend uses pure vanilla JavaScript without any frameworks, transpilation, or bundling. This approach maximizes performance, simplifies debugging, and reduces the learning curve for new contributors. ES modules are used for code organization, with script tags in index.html loading modules in a specific order to ensure proper dependency resolution.

The Component base class provides the foundation for all UI elements. It includes helpers for creating virtual DOM-like structures using the h() method, event delegation for consistent event handling, and direct DOM manipulation methods for updates. Components follow a lifecycle pattern with constructor, render, bindEvents, onMount, and destroy phases.

### 3.4 Development and Quality Tools

Jest 30.x serves as the testing framework with comprehensive coverage requirements. Tests are organized alongside source files with the `.test.js` suffix, and coverage reports must maintain at least 98% statement coverage. The testing approach focuses on behavior verification rather than implementation details, ensuring tests remain stable as code evolves.

ESLint 9.x enforces code quality rules defined in the project configuration. The linter runs as part of the continuous integration pipeline and must pass before any code is merged. Prettier 3.x handles code formatting, and the project is configured to format on save in supported editors.

pnpm serves as the exclusive package manager, chosen for its efficient disk usage, fast installs, and strict dependency resolution. All documentation and scripts reference pnpm commands, and npm should never be used for any operations.

## 4. Architecture Patterns

### 4.1 Server-Owned State Architecture

The application follows a server-owned state pattern where the backend maintains the authoritative state for all application data. This includes the list of available models, their current status (loaded, loading, unloaded, error), system metrics, configuration settings, and application logs. Clients never modify state directly; instead, they send requests through Socket.IO events, and the server updates state and broadcasts changes.

This architecture provides several benefits: all clients see a consistent view of the system state, state management is centralized and predictable, and the server can enforce validation and business rules before accepting changes. The downside is that the server must handle all state transitions and broadcast updates promptly to maintain responsiveness.

Frontend components subscribe to state changes through the stateManager, which maintains a registry of subscribers and broadcasts updates when the server sends new data. Components update their internal state and then directly manipulate the DOM to reflect changes. This pattern avoids the complexity of virtual DOM diffing while still providing reactive updates.

### 4.2 Socket.IO-Only Communication

All client-server communication uses Socket.IO events exclusively. There are no REST API endpoints, and traditional HTTP requests are only used for serving static files. This design simplifies the architecture by using a single communication protocol for all interactions and enables real-time bidirectional communication.

Event names follow a consistent naming convention using colon separators to indicate namespace and action. For example, `models:list` requests a list of all models, `models:load` requests loading a specific model, and `models:status` is a broadcast event indicating model status changes. Prefixes indicate the domain (models, metrics, logs, configuration), and suffixes indicate the action (list, load, start, stop, status).

The client initiates most interactions by sending a request event and waiting for a response. The server processes the request, performs necessary operations, and returns a response in the standardized format. For state changes that affect all clients, the server broadcasts an event without waiting for a specific request.

### 4.3 Event-Driven Updates

The frontend uses an event-driven programming model where components react to state changes and user events rather than actively polling for updates. StateManager maintains the central registry of state values and their subscribers. When state changes, either through a direct update or a Socket.IO broadcast, all subscribed components receive notifications.

Components register subscriptions in their onMount lifecycle method and clean them up in destroy. This pattern ensures that components only receive updates while they are active and that memory leaks are prevented by properly unsubscribing when components are destroyed. The subscription pattern is simple function-based, with each subscriber receiving the new value when state changes.

User interactions are handled through event delegation, where components attach a single event listener to their root element and delegate handling to child elements based on data attributes. This pattern reduces the number of event listeners and ensures that dynamically added elements are handled correctly without additional binding.

### 4.4 Repository Pattern for Database

The database layer uses a repository pattern that abstracts SQL operations behind JavaScript methods. Each repository is responsible for a specific domain and exposes methods for CRUD operations, queries, and aggregations. Repositories are instantiated with a database connection and can participate in transactions when needed.

Repository files are organized by domain in the server/db directory. The models-repository.js handles model metadata, the logs-repository.js manages application logs, and the configuration-repository.js stores and retrieves system configuration. New repositories can be added by creating a new file and importing it into the main index.js file.

Transactions are supported through the db.transaction() method, allowing multiple operations to be grouped with atomic commit or rollback. This is particularly important for operations that modify multiple tables or that must succeed completely or not at all.

### 4.5 Component-Based UI with Vanilla JS

The frontend uses a component-based architecture where each UI element is implemented as a class extending the Component base class. Components encapsulate their state, rendering logic, event handlers, and cleanup procedures. This pattern provides structure and organization while avoiding the complexity of framework-based components.

Components follow a lifecycle pattern with distinct phases: construction, rendering, event binding, mounting, and destruction. The constructor initializes properties, render() produces HTML or a Component instance, bindEvents() attaches event handlers, onMount() sets up subscriptions and timers, and destroy() performs cleanup. This separation of concerns makes components easier to understand and test.

Components render using the h() method, which creates virtual DOM-like structures that are converted to actual DOM elements. The h() method accepts a tag name or Component class, an attributes object, and an array of children. This approach provides a declarative way to describe component structure while maintaining the performance benefits of vanilla JavaScript.

## 5. Critical Project Rules

The following rules are absolute requirements for all code contributions:

### 5.1 Framework and Language Restrictions

**NEVER use React, Vue, Angular, Svelte, or any other frontend framework.** This project is built with pure vanilla JavaScript, and introducing any framework would violate the architectural decisions that make this project unique. If a contributor suggests using a framework, politely decline and explain the project's architectural constraints.

**NEVER use TypeScript.** This project uses pure JavaScript to simplify the build process, reduce tooling complexity, and make the codebase accessible to contributors familiar with any JavaScript environment. TypeScript adds compilation steps, type definitions, and potential compatibility issues that are not acceptable for this project.

**NEVER use npm.** The pnpm package manager must be used for all dependency operations. This includes installing packages (pnpm add), removing packages (pnpm remove), updating packages (pnpm update), and installing all dependencies (pnpm install). Using npm creates duplicate node_modules directories and may introduce version conflicts.

### 5.2 File and Code Organization

**NEVER write outside the project directory.** All file operations must occur within /home/bamer/nextjs-llama-async-proxy. Writing files outside this directory is prohibited and will result in command failures. This rule ensures that the project remains self-contained and portable.

**Keep files under 200 lines.** Large files are harder to read, understand, and maintain. When a file approaches 200 lines, consider splitting it into smaller, focused modules. This rule encourages single-responsibility design and makes navigation easier for developers.

**Single responsibility per module.** Each class, function, and module should have one clear purpose. A models-repository.js should only handle model database operations, not configuration or logging. If a module handles multiple concerns, split it into separate files.

### 5.3 Memory and Resource Management

**No memory leaks.** Always clean up subscriptions, event listeners, timers, and other resources when components are destroyed. Subscribe in onMount() and unsubscribe in destroy(). Remove event listeners when components are removed. Clear intervals and timeouts when they are no longer needed.

**No memory leaks in server code.** Server-side handlers should also clean up resources properly. Database connections should be returned to pools, file handles should be closed, and child processes should be terminated when no longer needed.

### 5.4 User Experience Requirements

**Always show user feedback.** Every user action should produce visible feedback, whether through notifications, loading indicators, or UI state changes. Users should never wonder whether their action was processed or what the current state is.

**Always catch and display errors.** Wrap all async operations in try-catch blocks and display meaningful error messages to users. Errors should never result in silent failures or unhandled rejections. Use showNotification() with the 'error' type for error feedback.

**Graceful degradation for missing data.** Components should handle missing, null, or undefined data gracefully. Show loading states while data is being fetched, display empty states when no data is available, and handle errors without breaking the entire application.

## 6. Code Style Guidelines

### 6.1 General Formatting Rules

All code in this project follows specific formatting rules enforced by Prettier:

- **Double quotes only**: Use "double quotes" for all string literals. Single quotes are not permitted under any circumstances.
- **Always use semicolons**: Terminate all statements with semicolons. Do not rely on automatic semicolon insertion.
- **2-space indentation**: Use two spaces for indentation. Do not use tabs.
- **Trailing commas**: Include trailing commas in multi-line objects and arrays. This improves diff readability and git history.
- **Max line width**: Keep lines under 100 characters. Break long lines at logical points.
- **Object curly spacing**: Always include spaces inside curly braces: `{ key: value }`
- **Array bracket spacing**: Never include spaces inside square brackets: `[1, 2, 3]`

### 6.2 Prettier Configuration

The project uses the following Prettier configuration in `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

Run `pnpm format` before committing to ensure code follows these rules. Configure your editor to format on save using the project's Prettier configuration.

### 6.3 ESLint Configuration

ESLint enforces code quality rules beyond formatting. The configuration requires:

- No unused variables
- No undefined variables
- Consistent use of const and let
- No console statements (except console.error for errors, console.warn for warnings)
- Consistent spacing and indentation
- Proper error handling

Run `pnpm lint` to check for violations. Auto-fix issues with `pnpm lint:fix`.

## 7. Naming Conventions

Consistent naming makes code more readable and maintainable:

### 7.1 Classes and Components

Use PascalCase for class names and component constructors:

```javascript
class DashboardController extends Component {}
class ModelsPage extends Component {}
class SocketClient {}
class StateManager {}
```

### 7.2 Functions and Variables

Use camelCase for function names and variable names:

```javascript
function getModels() {}
function formatBytes(bytes) {}
const isLoading = true;
const modelList = [];
```

### 7.3 Constants

Use UPPER_SNAKE_CASE for constant values that never change:

```javascript
const DEFAULT_CONFIG = { timeout: 5000 };
const API_BASE_URL = "/api";
const MAX_MODELS = 4;
const SUPPORTED_EXTENSIONS = [".gguf", ".safetensors"];
```

### 7.4 Private Members

Prefix private class members with an underscore:

```javascript
class MyComponent extends Component {
  _handleClick(event) {
    this._internalState = "updated";
  }
}
```

### 7.5 File Names

Match file names to their primary export:

```javascript
// layout.js - exports Layout class
// models-repository.js - exports ModelsRepository class
// state-manager.js - exports StateManager class
```

### 7.6 Event Names

Use colon-separated naming for Socket.IO events:

```javascript
// Request events (client to server)
"models:list"
"models:load"
"models:unload"
"configuration:get"

// Broadcast events (server to client)
"models:status"
"metrics:update"
"logs:entry"
```

## 8. File Organization

### 8.1 Directory Structure

The project follows a specific directory structure that separates concerns and maintains organization:

```
/home/bamer/nextjs-llama-async-proxy/
├── server.js                    # Main server entry point
├── package.json                 # Dependencies and scripts
├── pnpm-workspace.yaml          # pnpm workspace configuration
│
├── server/                      # Backend code
│   ├── index.js                 # Server initialization
│   ├── db/                      # Database layer
│   │   ├── index.js             # DB orchestrator and exports
│   │   ├── schema.js            # Table definitions
│   │   ├── models-repository.js # Model data access
│   │   ├── logs-repository.js   # Log data access
│   │   └── config-repository.js # Configuration data access
│   │
│   ├── handlers/                # Socket.IO event handlers
│   │   ├── index.js             # Handler registration
│   │   ├── models-handler.js    # Model-related events
│   │   ├── metrics-handler.js   # Metrics events
│   │   ├── logs-handler.js      # Log events
│   │   └── config-handler.js    # Configuration events
│   │
│   └── gguf/                    # GGUF parsing utilities
│       └── metadata.js          # GGUF metadata extraction
│
├── public/                      # Static frontend files
│   ├── index.html               # SPA entry point
│   ├── favicon.ico              # Application icon
│   ├── css/                     # Stylesheets
│   │   ├── main.css             # Core styles and variables
│   │   └── components.css       # Component-specific styles
│   │
│   └── js/                      # Frontend JavaScript
│       ├── app.js               # Application initialization
│       ├── core/                # Framework code
│       │   ├── component.js     # Base Component class
│       │   ├── router.js        # History API router
│       │   └── state.js         # State manager
│       │
│       ├── services/            # Service modules
│       │   └── socket.js        # Socket.IO client wrapper
│       │
│       ├── pages/               # Page controllers
│       │   ├── dashboard.js     # Dashboard page
│       │   ├── models.js        # Models management
│       │   ├── monitoring.js    # Monitoring page
│       │   ├── logs.js          # Logs viewer
│       │   ├── configuration.js # Configuration page
│       │   └── settings.js      # Settings page
│       │
│       └── components/          # Reusable UI components
│           └── layout/          # Layout components
│               └── layout.js    # Navigation and layout
│
├── data/                        # SQLite database storage
│   └── llama-dashboard.db       # SQLite database file
│
├── __tests__/                   # Test files
│   ├── server/                  # Server-side tests
│   └── utils/                   # Utility tests
│
├── docs/                        # Documentation
│   ├── README.md                # User documentation
│   ├── ARCHITECTURE.md          # Technical architecture
│   └── AGENTS.md                # This file
│
└── coverage/                    # Test coverage reports
```

### 8.2 Module Loading Order

Frontend modules are loaded in a specific order in index.html to ensure proper dependency resolution:

1. Core Framework: component.js, router.js, state.js
2. Services: socket.js
3. Pages: dashboard.js, models.js, monitoring.js, logs.js, configuration.js, settings.js
4. Components: layout/layout.js
5. Main App: app.js

This order ensures that base classes are available before components that depend on them, and that services are initialized before pages that use them.

## 9. Component Pattern (Event-Driven)

### 9.1 Component Base Class

All UI components extend the Component base class, which provides common functionality for rendering, event handling, and lifecycle management:

```javascript
class Component {
  constructor(props = {}) {
    this.props = props;
    this.$el = null;
    this._eventListeners = [];
    this._subscriptions = [];
  }

  /**
   * Create a virtual DOM element
   */
  static h(tag, attrs, ...children) {
    // Implementation in component.js
  }

  /**
   * Render the component - must be implemented by subclasses
   */
  render() {
    throw new Error("Component must implement render()");
  }

  /**
   * Bind event handlers - override in subclasses
   */
  bindEvents() {}

  /**
   * Called after component is mounted to DOM
   */
  onMount() {}

  /**
   * Cleanup - must be called when component is destroyed
   */
  destroy() {
    this._eventListeners.forEach(([el, type, handler]) => {
      el.removeEventListener(type, handler);
    });
    this._eventListeners = [];
    this._subscriptions.forEach((unsub) => unsub());
    this._subscriptions = [];
  }

  /**
   * Query a single element within the component
   */
  $(selector) {
    return this.$el?.querySelector(selector);
  }

  /**
   * Query all elements within the component
   */
  $$(selector) {
    return Array.from(this.$el?.querySelectorAll(selector) || []);
  }

  /**
   * Add event listener with delegation
   */
  on(type, selector, handler) {
    const wrappedHandler = (e) => {
      const target = e.target.closest(selector);
      if (target) {
        handler.call(this, e, target);
      }
    };
    this.$el.addEventListener(type, wrappedHandler);
    this._eventListeners.push([this.$el, type, wrappedHandler]);
  }

  /**
   * Set text content
   */
  setText(selector, text) {
    const el = this.$(selector);
    if (el) el.textContent = text;
  }

  /**
   * Set HTML content
   */
  setHTML(selector, html) {
    const el = this.$(selector);
    if (el) el.innerHTML = html;
  }

  /**
   * Add/remove/toggle classes
   */
  addClass(selector, className) {
    this.$(selector)?.classList.add(className);
  }

  removeClass(selector, className) {
    this.$(selector)?.classList.remove(className);
  }

  toggleClass(selector, className, force) {
    const el = this.$(selector);
    if (el) {
      if (typeof force === "boolean") {
        el.classList.toggle(className, force);
      } else {
        el.classList.toggle(className);
      }
    }
  }

  /**
   * Show/hide elements
   */
  show(selector) {
    this.$(selector)?.classList.remove("hidden");
  }

  hide(selector) {
    this.$(selector)?.classList.add("hidden");
  }
}
```

### 9.2 Complete Component Example

Here is a complete example of a component following all conventions:

```javascript
class ModelsTable extends Component {
  constructor(props) {
    super(props);
    this.models = props.models || [];
    this.selectedModel = null;
    this.loading = false;
  }

  render() {
    return Component.h("div", { className: "models-table-container" }, [
      Component.h("h2", {}, "Available Models"),
      Component.h("div", { className: "table-controls" }, [
        Component.h("button", {
          className: "btn btn-primary",
          "data-action": "refresh",
        }, "Refresh"),
      ]),
      Component.h("table", { className: "models-table" }, [
        Component.h("thead", {}, Component.h("tr", {},
          Component.h("th", {}, "Name"),
          Component.h("th", {}, "Status"),
          Component.h("th", {}, "Size"),
          Component.h("th", {}, "Actions")
        )),
        Component.h("tbody", {}, this._renderRows()),
      ]),
    ]);
  }

  _renderRows() {
    if (this.loading) {
      return Component.h("tr", {},
        Component.h("td", { colSpan: 4, className: "text-center" },
          "Loading models..."
        )
      );
    }

    if (this.models.length === 0) {
      return Component.h("tr", {},
        Component.h("td", { colSpan: 4, className: "text-center" },
          "No models found"
        )
      );
    }

    return this.models.map((model) =>
      Component.h("tr", {
        className: model.status === "loaded" ? "is-loaded" : "",
        "data-model-id": model.id,
      },
        Component.h("td", {}, model.name),
        Component.h("td", {},
          Component.h("span", {
            className: `status-badge status-${model.status}`,
          }, model.status)
        ),
        Component.h("td", {}, this._formatSize(model.size)),
        Component.h("td", {},
          Component.h("button", {
            className: "btn btn-sm",
            "data-action": "toggle",
            "data-model-id": model.id,
          }, model.status === "loaded" ? "Unload" : "Load")
        )
      )
    );
  }

  _formatSize(bytes) {
    if (bytes < 1024) return bytes + " B";
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + " KB";
    if (bytes < 1024 * 1024 * 1024) return (bytes / (1024 * 1024)).toFixed(1) + " MB";
    return (bytes / (1024 * 1024 * 1024)).toFixed(1) + " GB";
  }

  bindEvents() {
    this.on("click", "[data-action=refresh]", () => {
      this._handleRefresh();
    });

    this.on("click", "[data-action=toggle]", (e, target) => {
      const modelId = target.dataset.modelId;
      this._handleToggle(modelId);
    });
  }

  _handleRefresh() {
    this.loading = true;
    this._updateUI();
    this.trigger("refresh");
  }

  _handleToggle(modelId) {
    const model = this.models.find((m) => m.id === modelId);
    if (model) {
      this.trigger("toggle", { model, action: model.status === "loaded" ? "unload" : "load" });
    }
  }

  _updateUI() {
    const tbody = this.$("tbody");
    if (tbody) {
      tbody.innerHTML = "";
      const rows = this._renderRows();
      rows.forEach((row) => {
        const el = Component.h("div", {}, row);
        tbody.appendChild(el.firstElementChild);
      });
    }
  }

  onMount() {
    this._subscriptions.push(
      stateManager.subscribe("models", (models) => {
        this.models = models;
        this.loading = false;
        this._updateUI();
      })
    );
  }

  destroy() {
    super.destroy();
  }
}
```

### 9.3 Component Lifecycle

Components follow a specific lifecycle:

1. **Constructor**: Initialize properties, set initial state
2. **render()**: Return the component's HTML structure
3. **bindEvents()**: Attach event handlers
4. **onMount()**: Called after component is added to DOM, setup subscriptions
5. **destroy()**: Cleanup all resources

The app.js handles mounting components and calling lifecycle methods:

```javascript
// In app.js
const container = document.getElementById("app");
const page = router.getCurrentPage();
const component = page.render();
container.innerHTML = "";
container.appendChild(component.$el);
if (component.onMount) {
  component.onMount();
}
```

## 10. State Management

### 10.1 StateManager API

The global stateManager provides a simple API for state management:

```javascript
// Subscribe to state changes
const unsubscribe = stateManager.subscribe(key, callback);

// Update state (only server should call this in production)
stateManager.set(key, value);

// Get current state
const value = stateManager.get(key);

// Get all state
const allState = stateManager.getState();

// Make API request via Socket.IO
const response = await stateManager.request(event, payload);

// Unsubscribe when done
unsubscribe();
```

### 10.2 State Structure

The state object contains all application state:

```javascript
{
  models: [],        // Array of model objects
  currentModel: null, // Currently selected model
  metrics: {},       // System metrics
  logs: [],          // Recent log entries
  config: {},        // Application configuration
  connection: {      // Socket connection status
    status: "connected" | "disconnected" | "connecting"
  }
}
```

### 10.3 Subscription Pattern

Components subscribe to state changes in onMount() and clean up in destroy():

```javascript
class DashboardPage extends Component {
  constructor(props) {
    super(props);
    this.metrics = stateManager.get("metrics") || {};
  }

  onMount() {
    this._subscriptions = [
      stateManager.subscribe("metrics", (metrics) => {
        this.metrics = metrics;
        this._updateCharts();
      }),
      stateManager.subscribe("models", (models) => {
        this._updateModelCards(models);
      }),
    ];
  }

  destroy() {
    this._subscriptions.forEach((unsub) => unsub());
    this._subscriptions = [];
  }

  _updateCharts() {
    // Direct DOM updates based on new metrics
    const cpuElement = this.$("#cpu-chart");
    if (cpuElement) {
      cpuElement.style.width = this.metrics.cpu + "%";
    }
  }
}
```

## 11. Socket.IO Patterns

### 11.1 Socket Client

The socketClient wraps Socket.IO functionality:

```javascript
// Check connection status
socketClient.isConnected; // true or false

// Listen for broadcast events
socketClient.on("models:status", (data) => {
  stateManager.set("models", data.models);
});

socketClient.on("metrics:update", (data) => {
  stateManager.set("metrics", data.metrics);
});

socketClient.on("logs:entry", (data) => {
  const logs = stateManager.get("logs") || [];
  stateManager.set("logs", [data.entry, ...logs].slice(0, 100));
});

// Send requests (prefer stateManager.request)
socketClient.emit("models:list", {}, (response) => {
  if (response.success) {
    stateManager.set("models", response.data);
  } else {
    showNotification("Failed to load models: " + response.error.message, "error");
  }
});
```

### 11.2 Using stateManager.request

For cleaner code, use stateManager.request which handles the Socket.IO emit and response:

```javascript
// Get all models
const response = await stateManager.request("models:list", {});
if (response.success) {
  stateManager.set("models", response.data);
}

// Load a model
const loadResponse = await stateManager.request("models:load", { modelId: "llama-7b" });
if (loadResponse.success) {
  showNotification("Model loaded successfully", "success");
} else {
  showNotification("Failed to load model: " + loadResponse.error.message, "error");
}

// Get configuration
const configResponse = await stateManager.request("configuration:get", {});
if (configResponse.success) {
  stateManager.set("config", configResponse.data);
}
```

### 11.3 Server-Side Event Handling

Server handlers follow a consistent pattern:

```javascript
// In server/handlers/models-handler.js
function handleModelsList(socket, req) {
  const models = modelsRepository.getAll();
  socket.emit("models:list:response", {
    success: true,
    data: models,
    timestamp: new Date().toISOString(),
  });
}

function handleModelsLoad(socket, req) {
  const { modelId } = req;

  try {
    const result = llamaRouter.loadModel(modelId);
    socket.emit("models:load:response", {
      success: true,
      data: result,
      timestamp: new Date().toISOString(),
    });
    // Broadcast to all clients
    socket.broadcast.emit("models:status", {
      models: modelsRepository.getAll(),
    });
  } catch (error) {
    socket.emit("models:load:response", {
      success: false,
      error: {
        message: error.message,
        code: "LOAD_FAILED",
      },
      timestamp: new Date().toISOString(),
    });
  }
}
```

## 12. Error Handling

### 12.1 Frontend Error Handling

All async operations must be wrapped in try-catch blocks:

```javascript
async _handleStartModel(modelId) {
  try {
    this.loading = true;
    this._updateUI();

    const response = await stateManager.request("models:start", { modelId });

    if (response.success) {
      showNotification("Model started successfully", "success");
    } else {
      showNotification("Failed to start model: " + response.error.message, "error");
    }
  } catch (error) {
    console.error("[Models] Error starting model:", error);
    showNotification("Unexpected error: " + error.message, "error");
  } finally {
    this.loading = false;
    this._updateUI();
  }
}
```

### 12.2 Notification Types

Use the appropriate notification type for different situations:

```javascript
showNotification("Operation completed", "success");  // Green - positive outcomes
showNotification("Check your input", "warning");     // Yellow - cautionary messages
showNotification("Operation failed", "error");       // Red - errors and failures
showNotification("Information message", "info");     // Blue - neutral information
```

### 12.3 Server Error Handling

Server handlers should catch errors and emit appropriate responses:

```javascript
function handleModelsScan(socket, req) {
  try {
    console.log("[DEBUG] models:scan request", { requestId: req.requestId });

    const modelsDir = path.join(process.cwd(), "models");
    const files = fs.readdirSync(modelsDir);

    const models = files
      .filter((file) => /\.(gguf|safetensors)$/i.test(file))
      .map((file) => ({
        name: file,
        path: path.join(modelsDir, file),
        size: fs.statSync(path.join(modelsDir, file)).size,
      }));

    socket.emit("models:scan:response", {
      success: true,
      data: { models, count: models.length },
      timestamp: new Date().toISOString(),
    });
  } catch (error) {
    console.error("[DEBUG] Scan error:", error);
    socket.emit("models:scan:response", {
      success: false,
      error: {
        message: "Failed to scan models directory: " + error.message,
        code: "SCAN_ERROR",
      },
      timestamp: new Date().toISOString(),
    });
  }
}
```

## 13. Adding New Features

### 13.1 Adding a New Page

To add a new page to the dashboard:

1. Create the controller in `public/js/pages/`:

```javascript
// public/js/pages/newfeature.js
class NewFeatureController {
  constructor(options = {}) {
    this.router = options.router || window.router;
    this.unsubscribers = [];
    this.component = null;
  }

  init() {
    this.unsubscribers.push(
      stateManager.subscribe("data", (data) => {
        if (this.component) {
          this.component.updateData(data);
        }
      })
    );
  }

  willUnmount() {
    this.unsubscribers.forEach((unsub) => unsub());
    if (this.component) {
      this.component.destroy();
    }
  }

  render() {
    this.component = new NewFeaturePage({
      data: stateManager.get("data") || [],
    });
    this.init();
    return this.component.render();
  }

  didMount() {
    if (this.component && this.component.didMount) {
      this.component.didMount();
    }
  }
}

class NewFeaturePage extends Component {
  constructor(props) {
    super(props);
    this.data = props.data || [];
  }

  render() {
    return Component.h("div", { className: "new-feature-page" }, [
      Component.h("h1", {}, "New Feature"),
      this._renderContent(),
    ]);
  }

  _renderContent() {
    if (this.data.length === 0) {
      return Component.h("div", { className: "empty-state" }, "No data available");
    }
    return Component.h("div", { className: "data-list" },
      this.data.map((item) =>
        Component.h("div", { className: "data-item" }, item.name)
      )
    );
  }

  updateData(data) {
    this.data = data;
    this._updateUI();
  }

  _updateUI() {
    const content = this.$(".data-list");
    if (content) {
      content.innerHTML = this._renderContent().children.map(c => {
        const wrapper = Component.h("div", {}, c);
        return wrapper.innerHTML;
      }).join("");
    }
  }

  bindEvents() {
    // Add event handlers here
  }

  onMount() {
    // Setup subscriptions
  }

  destroy() {
    super.destroy();
  }
}

window.NewFeatureController = NewFeatureController;
```

2. Register the route in `public/js/app.js`:

```javascript
// In the routes section
router.register("/newfeature", () => new NewFeatureController({}));
```

3. Add navigation link in `public/js/components/layout/layout.js`:

```javascript
// In the navigation menu
Component.h("a", {
  href: "#/newfeature",
  className: location.pathname === "/newfeature" ? "active" : "",
}, "New Feature")
```

### 13.2 Adding a New Socket.IO Event

To add a new Socket.IO event:

1. Add the handler in `server/handlers/`:

```javascript
// server/handlers/newfeature-handler.js
function handleNewFeatureAction(socket, req) {
  const { action, payload } = req;

  try {
    console.log("[DEBUG] newfeature:action request", { action, payload });

    // Process the action
    const result = processAction(action, payload);

    socket.emit("newfeature:action:response", {
      success: true,
      data: result,
      timestamp: new Date().toISOString(),
    });

    // Broadcast to other clients if needed
    socket.broadcast.emit("newfeature:update", {
      action,
      result,
    });
  } catch (error) {
    console.error("[DEBUG] newfeature:action error:", error);
    socket.emit("newfeature:action:response", {
      success: false,
      error: {
        message: error.message,
        code: "ACTION_FAILED",
      },
      timestamp: new Date().toISOString(),
    });
  }
}

module.exports = { handleNewFeatureAction };
```

2. Export from `server/handlers/index.js`:

```javascript
const { handleNewFeatureAction } = require("./newfeature-handler");

module.exports = {
  registerHandlers: (socket) => {
    socket.on("newfeature:action", (req) => handleNewFeatureAction(socket, req));
    // ... other handlers
  },
};
```

3. Add frontend method in `public/js/core/state/`:

```javascript
// In StateManager class or as standalone function
async function newFeatureAction(action, payload) {
  return new Promise((resolve, reject) => {
    socketClient.emit("newfeature:action", { action, payload }, (response) => {
      if (response.success) {
        resolve(response);
      } else {
        reject(new Error(response.error?.message || "Unknown error"));
      }
    });
  });
}

// Make available globally
window.newFeatureAction = newFeatureAction;
```

### 13.3 Adding a New Repository

To add a new database repository:

1. Create the repository file:

```javascript
// server/db/newfeature-repository.js
const db = require("./index");

class NewFeatureRepository {
  constructor(database) {
    this.db = database;
  }

  getAll() {
    return this.db.prepare("SELECT * FROM new_feature ORDER BY created_at DESC").all();
  }

  getById(id) {
    return this.db.prepare("SELECT * FROM new_feature WHERE id = ?").get(id);
  }

  create(data) {
    const stmt = this.db.prepare(`
      INSERT INTO new_feature (name, value, created_at)
      VALUES (?, ?, ?)
    `);
    return stmt.run(data.name, data.value, new Date().toISOString());
  }

  update(id, data) {
    const stmt = this.db.prepare(`
      UPDATE new_feature SET name = ?, value = ? WHERE id = ?
    `);
    return stmt.run(data.name, data.value, id);
  }

  delete(id) {
    return this.db.prepare("DELETE FROM new_feature WHERE id = ?").run(id);
  }
}

module.exports = NewFeatureRepository;
```

2. Import and instantiate in `server/db/index.js`:

```javascript
const NewFeatureRepository = require("./newfeature-repository");

const db = new Database(path.join(__dirname, "../../data/llama-dashboard.db"));
const modelsRepository = new ModelsRepository(db);
const logsRepository = new LogsRepository(db);
const configRepository = new ConfigRepository(db);
const newFeatureRepository = new NewFeatureRepository(db);

module.exports = {
  db,
  modelsRepository,
  logsRepository,
  configRepository,
  newFeatureRepository,
};
```

## 14. Debug Logging Standards

### 14.1 Backend Debug Logging

All server-side debug output must use the `[DEBUG]` prefix:

```javascript
// Lifecycle events
console.log("[DEBUG] Server starting on port", port);
console.log("[DEBUG] Socket connection established", { socketId: socket.id });

// Request handling
console.log("[DEBUG] Event received:", { event: "models:list", data: req });
console.log("[DEBUG] Request details:", { requestId, payload });

// Operation results
console.log("[DEBUG] Scan result:", { found: files.length, path: modelsDir });
console.log("[DEBUG] Model loaded:", { modelId, memory: result.memory });
console.log("[DEBUG] Database query:", { operation: "select", table: "models" });

// Error handling
console.error("[DEBUG] Error details:", error.message);
console.error("[DEBUG] Stack trace:", error.stack);
```

### 14.2 Frontend Debug Logging

Frontend debug logging follows the same pattern:

```javascript
// Controllers
console.log("[DEBUG] NewFeatureController created");
console.log("[DEBUG] NewFeatureController init");
console.log("[DEBUG] NewFeatureController willUnmount");

// State operations
console.log("[DEBUG] State changed:", { key: "models", value: models.length });
console.log("[DEBUG] API request:", { event: "models:list", data: {} });
console.log("[DEBUG] API response:", { success: true, data: models });

// Component lifecycle
console.log("[DEBUG] NewFeaturePage created, props:", { dataLength: data.length });
console.log("[DEBUG] Button clicked:", { action: "refresh", modelId });

// Error handling
console.error("[DEBUG] Error:", { message: error.message, stack: error.stack });
```

### 14.3 Debug Log Format

Use consistent JSON formatting for complex data:

```javascript
// Good - JSON format with braces
console.log("[DEBUG] Model status changed:", {
  modelId: "llama-7b",
  oldStatus: "unloaded",
  newStatus: "loading",
  timestamp: Date.now(),
});

// Good - Simple values
console.log("[DEBUG] Models count:", models.length);

// Avoid - String concatenation
console.log("[DEBUG] Model status: " + status + " for " + modelId);
```

## 15. Testing Requirements

### 15.1 Test Coverage Goals

The project requires high test coverage:

- **Statement coverage**: 98% minimum
- **Branch coverage**: 95% minimum
- **Function coverage**: 98% minimum
- **Line coverage**: 98% minimum

All new code must include corresponding tests. Tests should verify behavior, not implementation details, ensuring tests remain stable as code evolves.

### 15.2 Test Organization

Tests are organized alongside source files:

```
__tests__/
├── server/
│   ├── db/
│   │   ├── models-repository.test.js
│   │   └── logs-repository.test.js
│   └── handlers/
│       └── models-handler.test.js
└── utils/
    ├── validation.test.js
    └── formatting.test.js
```

### 15.3 Running Tests

```bash
# Run all tests
pnpm test

# Run with coverage report
pnpm test:coverage

# Run in watch mode
pnpm test:watch

# Run specific test file
pnpm test -- __tests__/server/db/models-repository.test.js
```

### 15.4 Test Example

```javascript
// __tests__/utils/formatting.test.js
const { formatBytes } = require("../../src/utils/formatting");

describe("formatBytes", () => {
  describe("basic units", () => {
    test("formats bytes correctly", () => {
      expect(formatBytes(512)).toBe("512 B");
    });

    test("formats kilobytes correctly", () => {
      expect(formatBytes(1024)).toBe("1.00 KB");
      expect(formatBytes(1536)).toBe("1.50 KB");
    });

    test("formats megabytes correctly", () => {
      expect(formatBytes(1024 * 1024)).toBe("1.00 MB");
      expect(formatBytes(5 * 1024 * 1024)).toBe("5.00 MB");
    });

    test("formats gigabytes correctly", () => {
      expect(formatBytes(1024 * 1024 * 1024)).toBe("1.00 GB");
      expect(formatBytes(2.5 * 1024 * 1024 * 1024)).toBe("2.50 GB");
    });
  });

  describe("edge cases", () => {
    test("handles zero bytes", () => {
      expect(formatBytes(0)).toBe("0 B");
    });

    test("handles very large values", () => {
      expect(formatBytes(1024 * 1024 * 1024 * 1024)).toBe("1.00 TB");
    });
  });
});
```

## 16. Forbidden Patterns

The following patterns are explicitly forbidden and will result in code review rejection:

### 16.1 Framework and Language

- **NO React, Vue, Angular, Svelte, or any frontend framework**
  - This is a vanilla JavaScript project. Using frameworks violates the core architectural decision.

- **NO TypeScript**
  - This project uses pure JavaScript. TypeScript compilation adds complexity.

- **NO JSX, TSX, or template literals for HTML**
  - Use Component.h() for creating elements.

- **NO npm - use pnpm exclusively**
  - Using npm creates nested node_modules and version conflicts.

### 16.2 Architecture Patterns

- **NO REST API endpoints**
  - All communication must use Socket.IO events.

- **NO virtual DOM or reconciliation**
  - Use direct DOM manipulation through Component methods.

- **NO setState or state re-rendering patterns**
  - Update component properties directly and use DOM methods.

- **NO React-like hooks (useEffect, useState)**
  - Use the Component lifecycle methods (onMount, destroy).

### 16.3 Code Quality

- **NO memory leaks**
  - Always clean up subscriptions, listeners, and timers.

- **NO unhandled promises**
  - Always attach .catch() or use try/catch with async/await.

- **NO console.log in production code**
  - Use debug logs with `[DEBUG]` prefix.

- **NO synchronous XMLHttpRequest**
  - Use Socket.IO or fetch for all network requests.

## 17. Common Mistakes to Avoid

### 17.1 Package Manager Mistakes

Using npm instead of pnpm is the most common mistake:

```javascript
// WRONG - using npm
npm install express

// CORRECT - using pnpm
pnpm add express

// WRONG - npm run start
npm run start

// CORRECT - pnpm start
pnpm start
```

### 17.2 Component Pattern Mistakes

Using setState or re-rendering instead of direct DOM updates:

```javascript
// WRONG - React-like pattern
this.setState({ loading: true });
this.setState({ data: newData });

// CORRECT - Event-driven pattern
this.loading = true;
this.$btn.disabled = true;
this.$btn.textContent = "Loading...";
```

### 17.3 Event Handler Mistakes

Forgetting event delegation or improper cleanup:

```javascript
// WRONG - Adding listeners to each element
connectedElements.forEach((el) => {
  el.addEventListener("click", this._handleClick.bind(this));
});

// CORRECT - Event delegation on root element
this.on("click", "[data-action]", (e, target) => {
  const action = target.dataset.action;
  this._handleAction(action);
});
```

### 17.4 Memory Leak Mistakes

Forgetting to clean up subscriptions:

```javascript
// WRONG - No cleanup
onMount() {
  stateManager.subscribe("models", (models) => {
    this.models = models;
  });
}

// CORRECT - Store and cleanup
onMount() {
  this._subscriptions = [
    stateManager.subscribe("models", (models) => {
      this.models = models;
    }),
  ];
}

destroy() {
  this._subscriptions.forEach((unsub) => unsub());
  this._subscriptions = [];
}
```

### 17.5 Import/Export Mistakes

Using incorrect file names or export patterns:

```javascript
// WRONG - file name doesn't match export
// layout.js exports "Navbar" class

// CORRECT - file name matches export
// navbar.js exports Navbar class
```

## 18. Quick Reference

### 18.1 Essential Commands

```bash
# Development
pnpm start              # Start production server
pnpm dev                # Start with file watching
pnpm watch              # Watch mode for development

# Package Management (ALWAYS use pnpm)
pnpm add <package>              # Add dependency
pnpm add -D <package>           # Add dev dependency
pnpm remove <package>           # Remove dependency
pnpm install                   # Install all dependencies
pnpm update                    # Update all dependencies

# Testing
pnpm test                      # Run all tests
pnpm test:watch                # Watch mode
pnpm test:coverage             # Coverage report

# Quality
pnpm lint                      # Run ESLint
pnpm lint:fix                  # Auto-fix issues
pnpm format                    # Format with Prettier
pnpm format:check              # Check formatting

# Database
pnpm db:export                 # Export database backup
pnpm db:reset                  # Reset database
```

### 18.2 Socket.IO Event Quick Reference

```javascript
// Request events (client -> server)
stateManager.request("models:list", {});           // Get all models
stateManager.request("models:load", { modelId });  // Load a model
stateManager.request("models:unload", { modelId }); // Unload a model
stateManager.request("models:start", { modelId });  // Start generation
stateManager.request("models:stop", { modelId });   // Stop generation
stateManager.request("configuration:get", {});      // Get config
stateManager.request("configuration:set", config);  // Update config
stateManager.request("logs:get", { limit: 100 });   // Get logs

// Broadcast events (server -> client)
socketClient.on("models:status", (data) => {});     // Model status change
socketClient.on("metrics:update", (data) => {});    // New metrics
socketClient.on("logs:entry", (data) => {});        // New log entry
```

### 18.3 Component Methods Quick Reference

```javascript
// Creating elements
Component.h("div", { className: "container" }, "Content");
Component.h("button", { onClick: handler }, "Click");

// Querying
this.$(".selector");              // Single element
this.$$(".selector");             // All elements

// Updating content
this.setText(".label", "Text");
this.setHTML(".container", "<div>");

// Classes
this.addClass(".item", "active");
this.removeClass(".item", "hidden");
this.toggleClass(".item", "selected", true);

// Visibility
this.show(".element");
this.hide(".element");

// Events
this.on("click", "[data-action]", handler);
```

### 18.4 StateManager Quick Reference

```javascript
// Subscribe
const unsub = stateManager.subscribe("key", (value) => {
  console.log(value);
});

// Get/Set
const value = stateManager.get("key");
stateManager.set("key", newValue);

// Request
const response = await stateManager.request("event", payload);

// Cleanup
unsub();
```

## 19. Related Documentation

For additional information, consult these documents:

- **[docs/README.md](docs/README.md)** - User guide and feature documentation
- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Detailed technical architecture
- **llama.cpp documentation** - Router mode and CLI options
- **Socket.IO documentation** - Event handling and rooms

---

**Remember**: This is a Vanilla JavaScript project using Socket.IO for communication and SQLite for persistence. Always use pnpm, never use npm. Keep files under 200 lines. Follow the established patterns for components, state management, and error handling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bamer)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/bamer)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
