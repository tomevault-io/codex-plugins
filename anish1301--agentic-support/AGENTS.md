<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Richpanel AI Agent POC - Copilot Instructions

## Project Overview
This is a Proof of Concept (POC) for Richpanel's AI customer support platform, showcasing autonomous AI agents that can handle customer inquiries and perform real actions like order cancellations, tracking, and returns.

## Technology Stack
- **Frontend**: Vue.js 3 with Composition API
- **State Management**: Vuex 4
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **AI Integration**: Gemini API (simulated for POC)

## Key Features
1. **AI Chat Interface**: Natural language processing for customer support
2. **Order Management**: Cancel, track, and return orders autonomously
3. **Real-time State Updates**: Live order status changes
4. **Mock Data**: Realistic dummy data to simulate real operations

## Code Guidelines

### Vue Components
- Use Vue 3 Composition API where appropriate
- Follow single-responsibility principle for components
- Use props for data down, events for actions up
- Implement proper error handling

### Vuex Store Structure
- Modular store with separate modules for: chat, orders, ui, agent
- Use namespaced modules
- Actions should handle async operations
- Mutations should be synchronous and pure

### Styling Guidelines
- Use Tailwind CSS utility classes
- Custom component classes defined in style.css
- Responsive design (mobile-first approach)
- Consistent color scheme using primary/gray colors

### AI Agent Logic
- Simulate realistic AI processing delays
- Extract intents and entities from user messages
- Provide contextual responses based on order data
- Handle edge cases gracefully

## Component Structure
```
src/
├── components/
│   ├── chat/          # Chat interface components
│   ├── orders/        # Order management components
│   └── ui/           # Reusable UI components
├── store/
│   └── modules/      # Vuex store modules
└── assets/           # Static assets
```

## Mock Data Features
- Realistic order data with proper statuses
- Customer information
- Product details with images
- Tracking numbers and delivery dates
- Action history logging

## Best Practices
- Always validate user inputs
- Provide loading states for async operations
- Show success/error notifications
- Maintain audit trail of all actions
- Use semantic HTML and ARIA labels for accessibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anish1301)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/anish1301)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
