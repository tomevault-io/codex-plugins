# AuraTracker - Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
AuraTracker è una webapp per tracciare "l'aura" dei giocatori in diverse partite. Ogni partita ha un codice univoco e classifiche separate.

## Tech Stack
- **Frontend**: Next.js 14+ con App Router, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes 
- **Database**: Supabase (PostgreSQL)
- **Deployment**: Vercel
- **State Management**: React Context/useState
- **UI Components**: Custom components con Tailwind

## Architecture Guidelines
- Usa App Router per la struttura delle pagine
- API Routes in `src/app/api/` per logica backend
- Database schema con Supabase per partite, giocatori, azioni
- Responsive design mobile-first
- Componenti riutilizzabili in `src/components/`

## Database Schema
- `games`: id, code, name, created_at, updated_at
- `players`: id, game_id, name, avatar, aura_points, created_at
- `actions`: id, game_id, player_id, action_type, points, description, created_at

## UI/UX Guidelines
- Design pulito ispirato allo screenshot fornito
- Cards per giocatori con avatar, nome e punti aura
- Navigazione con tab (Home, Giocatori, Reset, Elimina Partita)
- Status online/offline
- Form per aggiungere giocatori
- Storico azioni in tempo reale
- Colori: background neutro, accenti colorati per status e azioni

## Code Style
- Usa TypeScript strict mode
- Componenti funzionali con hooks
- Naming conventions: PascalCase per componenti, camelCase per variabili
- Gestione errori con try-catch
- Validazione input con zod o simili

---
> Source: [braeddy/aura-tracker](https://github.com/braeddy/aura-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
