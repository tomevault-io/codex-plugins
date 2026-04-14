<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Project Details: Chat UI React Application

This is a React-based chat UI application that implements a chat interface with the following features:
- JSX Layout with chat interface components (input field, send button, and chat window)
- State management for storing and displaying chat messages
- Functional components and React Hooks
- Ability to send messages and display them in a scrollable container

## Project Structure

- `/src/components/ChatUI.js`: Contains the main chat interface component
- `/src/components/ChatUI.css`: Contains styles for the chat interface
- `/src/App.js`: Main application component that renders the ChatUI component

## Naming Conventions

- React components should use PascalCase (e.g., ChatUI, MessageList)
- Functions and variables should use camelCase (e.g., handleSendMessage, newMessage)
- CSS classes should use kebab-case (e.g., chat-container, message-input)

## State Management

- We use React Hooks, specifically useState for managing state
- The message state should include messages with sender type ('user' or 'bot') and text content

## Component Structure Guidelines

- Keep components focused on a single responsibility
- Use functional components with hooks instead of class components
- Extract reusable UI elements into their own components when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adityakr1108) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
