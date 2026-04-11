<!-- Cooking AI Agent Development Instructions -->

# Cooking AI Agent - Development Guidelines

This is a Python-based AI agent application powered by GitHub Models for recipe search and ingredient extraction.

## Project Overview

- **Language:** Python 3.8+
- **Framework:** GitHub Models API (GPT-4o-mini)
- **Type:** Interactive Console Application
- **Features:** Recipe search, ingredient extraction, multi-turn conversation

## Setup Instructions

1. Install dependencies: `pip install -r requirements.txt`
2. Configure GitHub token in `.env` file
3. Run the application: `python src/cooking_agent.py`

## Key Files

- `src/cooking_agent.py` - Main agent implementation
- `requirements.txt` - Python dependencies
- `.env.example` - Environment configuration template
- `README.md` - User documentation

## Development Notes

- The agent uses GitHub Models API endpoint: `https://models.inference.ai.azure.com/chat/completions`
- Model: GPT-4o-mini
- Maintains conversation history for multi-turn interactions
- Supports three main modes: recipe search, ingredient extraction, and general chat

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinwiliamleo-dev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kevinwiliamleo-dev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
