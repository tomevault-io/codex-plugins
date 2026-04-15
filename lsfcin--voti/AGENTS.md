<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Vôti - Political Affinity App

This is a Next.js web application called "Vôti" that helps Brazilian citizens discover their political affinity with federal deputies and senators based on actual congressional votes.

## Project Overview

- **Tech Stack**: Next.js 15, TypeScript, Tailwind CSS, React 19
- **Purpose**: Political transparency and civic engagement platform
- **Target**: Brazilian citizens interested in understanding their political alignment

## Key Features

1. **Data Collection**: Scrapes voting data from Brazilian Congress portals (Câmara dos Deputados and Senado Federal)
2. **AI Simplification**: Uses free LLM APIs to convert complex legal text into simple user-friendly questions
3. **Interactive Questionnaire**: Users answer simplified questions about real congressional votes
4. **Affinity Analysis**: Shows compatibility percentage with politicians and parties
5. **Detailed Results**: Visual representation of affinity with color-coded progress bars
6. **Politician Profiles**: Links to electoral and justice records

## Code Conventions

- Use TypeScript with strict type checking
- Follow React functional components with hooks
- Use Tailwind CSS for styling with custom color scheme for affinity levels
- API routes follow Next.js 13+ app directory structure
- Components are modular and reusable

## Color Scheme for Affinity

- Perfect (80-100%): Green (#059669)
- High (60-79%): Light Green (#10b981) 
- Medium (40-59%): Yellow/Orange (#f59e0b)
- Low (20-39%): Orange (#dc2626)
- None (0-19%): Red (#7f1d1d)

## Data Sources

- Câmara dos Deputados: www.camara.leg.br
- Senado Federal: www25.senado.leg.br
- Justice records and electoral information

## Development Notes

- Web scraping should be respectful and follow robots.txt
- AI text simplification uses Hugging Face free models as fallback
- All voting data should be verifiable and transparent
- Focus on user experience and democratic transparency

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lsfcin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
