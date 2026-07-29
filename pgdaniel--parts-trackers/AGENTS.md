# Copilot Instructions for Parts Tracker

## Architecture Overview

This is a **Rails 8.1** application using the modern Rails stack with SQLite for all storage needs (database, cache, job queue, and Action Cable). The project uses a **unique hybrid architecture**: each Rails view renders a **complete standalone SPA** with **ActionCable for real-time data transfer**.

Key architectural decisions:
- **One Rails route → One complete SPA**: Each major section (e.g., `/tools`, `/inventory`) is a full React application
- **React Router** handles all routing within each SPA (no page reloads)
- **Zustand** for state management within each SPA
- **ActionCable for data**: Use WebSocket channels instead of REST/JSON APIs for all client-server communication
- **Stimulus mounts React**: Each SPA is mounted via a Stimulus controller for clean lifecycle management
- **Solid Queue** for background jobs (runs in-process with Puma via `SOLID_QUEUE_IN_PUMA=true`)
- **Solid Cache** and **Solid Cable** for performance and real-time features (Solid Cable backs ActionCable)
- **Propshaft** for asset pipeline (not Sprockets)
- **esbuild** for JavaScript bundling (via jsbundling-rails)
- **SQLite multi-database setup**: separate databases for primary, cache, queue, and cable in production
- **Kamal** deployment configured for containerized deployment

## Development Workflows

### Starting the Application
```bash
bin/dev  # Runs Procfile.dev: Rails server + esbuild watch mode
```
This starts two processes: the Rails server with Ruby debugger enabled and JavaScript building in watch mode.

### Setup/Reset
```bash
bin/setup              # Initial setup: bundle, yarn, db:prepare
bin/setup --reset      # Full reset: drops and recreates database
```

### Running Tests
```bash
bin/rails test         # Run all tests (parallelized by default)
bin/rails test:system  # System tests with Selenium/Chrome headless
bin/ci                 # Full CI suite: tests + linting + security audits
```

### Code Quality & Security
```bash
bin/rubocop           # Ruby style (follows Rails Omakase styling)
bin/brakeman          # Security vulnerability scanner
bin/bundler-audit     # Check for vulnerable gem versions
```

## JavaScript/Frontend Conventions

Each Rails route renders a **complete standalone SPA** that manages all its own routing, state, and UI. Think of Rails as providing the entry point and data layer, while React takes over the entire user experience within that section.

### Structure
- **Complete SPAs** in `app/javascript/apps/` (e.g., `ToolsApp.jsx`, `InventoryApp.jsx`)
- **Stimulus controllers** in `app/javascript/controllers/` - mount React apps with lifecycle management
- **React components** in `app/javascript/components/` - organized by feature/domain
- **Zustand stores** in `app/javascript/stores/` - state management per app
- **ActionCable channels** in `app/javascript/channels/` for real-time data
- Entry point: `app/javascript/application.js`
- Build output: `app/assets/builds/application.js` (generated, don't edit)

### Complete SPA Pattern
A single Rails route (e.g., `/tools`) loads a full React application that handles all sub-routes internally:

```javascript
// app/javascript/apps/ToolsApp.jsx
import React from 'react'
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { useToolsStore } from '../stores/toolsStore'
import ToolsList from '../components/tools/ToolsList'
import ToolDetail from '../components/tools/ToolDetail'
import ToolEdit from '../components/tools/ToolEdit'

export default function ToolsApp({ channel }) {
  return (
    <BrowserRouter basename="/tools">
      <Routes>
        <Route path="/" element={<ToolsList channel={channel} />} />
        <Route path="/:id" element={<ToolDetail channel={channel} />} />
        <Route path="/:id/edit" element={<ToolEdit channel={channel} />} />
        <Route path="*" element={<Navigate to="/" replace />} />
      </Routes>
    </BrowserRouter>
  )
}
```

```javascript
// app/javascript/stores/toolsStore.js
import { create } from 'zustand'

export const useToolsStore = create((set) => ({
  tools: [],
  currentTool: null,
  setTools: (tools) => set({ tools }),
  setCurrentTool: (tool) => set({ currentTool: tool }),
  updateTool: (id, updates) => set((state) => ({
    tools: state.tools.map(t => t.id === id ? { ...t, ...updates } : t),
    currentTool: state.currentTool?.id === id ? { ...state.currentTool, ...updates } : state.currentTool
  }))
}))
```

```javascript
// app/javascript/controllers/tools_app_controller.js
import { Controller } from "@hotwired/stimulus"
import React from 'react'
import { createRoot } from 'react-dom/client'
import ToolsApp from '../apps/ToolsApp'
import consumer from '../channels/consumer'

export default class extends Controller {
  connect() {
    this.root = createRoot(this.element)
    this.setupChannel()
  }
  
  disconnect() {
    if (this.channel) {
      this.channel.unsubscribe()
    }
    if (this.root) {
      this.root.unmount()
    }
  }
  
  setupChannel() {
    this.channel = consumer.subscriptions.create("ToolsChannel", {
      connected: () => {
        console.log('Connected to ToolsChannel')
        this.render()
      },
      
      received: (data) => {
        // Channel communicates with Zustand store via callbacks or events
        console.log('Received:', data)
      }
    })
  }
  
  render() {
    this.root.render(
      <React.StrictMode>
        <ToolsApp channel={this.channel} />
      </React.StrictMode>
    )
  }
}
```

```erb
<%# app/views/tools/index.html.erb %>
<div data-controller="tools-app"></div>
```

```ruby
# config/routes.rb
Rails.application.routes.draw do
  # Single route - React Router handles everything inside
  get 'tools', to: 'tools#index'
  get 'tools/*path', to: 'tools#index'  # Catch all sub-routes for React Router
end
```

### ActionCable Data Pattern
Instead of REST APIs or Turbo Streams, use ActionCable channels for all data operations:

```javascript
// Example: app/javascript/channels/tools_channel.js
import consumer from "./consumer"
import { useToolsStore } from '../stores/toolsStore'

export function createToolsChannel() {
  return consumer.subscriptions.create("ToolsChannel", {
    received(data) {
      const store = useToolsStore.getState()
      
      if (data.action === 'tools_list') {
        store.setTools(data.tools)
      } else if (data.action === 'tool_updated') {
        store.updateTool(data.tool.id, data.tool)
      } else if (data.action === 'tool_created') {
        store.setTools([...store.tools, data.tool])
      }
    },
    
    fetchTools() {
      this.perform('fetch_tools')
    },
    
    createTool(toolData) {
      this.perform('create_tool', { tool: toolData })
    },
    
    updateTool(id, toolData) {
      this.perform('update_tool', { id, tool: toolData })
    }
  })
}
```

### Rails Channel Pattern
Corresponding server-side channel handles data operations:

```ruby
# app/channels/tools_channel.rb
class ToolsChannel < ApplicationCable::Channel
  def subscribed
    stream_from "tools"
  end
  
  def fetch_tools
    tools = Tool.all
    transmit(action: 'tools_list', tools: tools.as_json)
  end
  
  def create_tool(data)
    tool = Tool.create!(data['tool'])
    ActionCable.server.broadcast("tools", 
      action: 'tool_created', tool: tool.as_json)
  end
  
  def update_tool(data)
    tool = Tool.find(data['id'])
    if tool.update(data['tool'])
      ActionCable.server.broadcast("tools",
        action: 'tool_updated', tool: tool.as_json)
    end
  end
end
```

**Note**: This pattern uses Stimulus for lifecycle management (mounting/unmounting React) while React handles the UI and ActionCable handles data. Zustand stores manage app state and React Router handles navigation within each SPA.

## Background Jobs

Jobs inherit from `ApplicationJob` and are processed by Solid Queue. In development/single-server setups, jobs run inside the Puma process. For production scaling, disable `SOLID_QUEUE_IN_PUMA` and run dedicated job workers.

Recurring jobs are configured in `config/recurring.yml` (see `clear_solid_queue_finished_jobs` example).

## Database Patterns

- Uses SQLite with **separate database files** for different concerns in production
- Migrations live in `db/migrate/` (primary), `db/cache_migrate/`, `db/queue_migrate/`, `db/cable_migrate/`
- Schema files: `db/schema.rb`, `db/cache_schema.rb`, `db/queue_schema.rb`, `db/cable_schema.rb`
- Connection pooling configured per database in `config/database.yml`

## Deployment

Uses **Kamal** for zero-downtime Docker deployments:
```bash
bin/kamal deploy       # Deploy to servers in config/deploy.yml
bin/kamal app exec     # Run commands in deployed container
```

Configuration in `config/deploy.yml`:
- Server IPs defined under `servers.web`
- Registry at `localhost:5555` (update for your registry)
- `RAILS_MASTER_KEY` required for secrets (stored in `.kamal/secrets`)

## Testing Conventions

- Tests parallelize automatically (`parallelize(workers: :number_of_processors)`)
- System tests support **remote Selenium** for CI (see `CAPYBARA_SERVER_PORT` and `SELENIUM_HOST` env vars in `application_system_test_case.rb`)
- Fixtures in `test/fixtures/` are loaded for all tests
- CI validates: tests, style (RuboCop), security (Brakeman, bundler-audit), and seed data integrity

## Project-Specific Notes

- Application name: **Parts Trakcer** (note the typo in module name `PartsTrakcer` in `config/application.rb`)
- PWA support configured but disabled by default (see commented routes in `config/routes.rb` and manifest links in `application.html.erb`)
- Health check endpoint: `/up` for load balancers/monitoring
- No root route defined yet—add controllers/views as needed

---
> Source: [pgdaniel/parts_trackers](https://github.com/pgdaniel/parts_trackers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-27 -->
