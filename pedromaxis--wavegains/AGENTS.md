<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Copilot Instructions - WaveGains App

This is a React TypeScript application for hypertrophy periodization training programs. The project uses:

- **Framework**: React 18 with TypeScript
- **Styling**: Tailwind CSS with custom design system
- **Icons**: Lucide React
- **Charts**: Recharts for data visualization
- **Build Tool**: Vite

## Key Design Principles

- **Evidence-based**: All periodization models should be based on scientific literature
- **User-focused**: Interface should be intuitive for different experience levels (beginner, intermediate, advanced)
- **Responsive**: Mobile-first design approach
- **Accessibility**: Follow WCAG guidelines
- **Performance**: Optimize for fast loading and smooth interactions

## Architecture Guidelines

- Use functional components with hooks
- Implement proper TypeScript types for all data structures
- Keep state management simple with useState/useEffect (no localStorage requirement)
- Create reusable components with proper prop interfaces
- Use custom hooks for complex logic

## Periodization Models to Implement

1. **Linear Traditional**: Gradual progression (volume ↓, intensity ↑)
2. **Undulating (DUP)**: Weekly or daily variation
3. **Block Periodization**: Accumulation → Transmutation → Realization phases
4. **Conjugate Method**: Simultaneous training of different qualities

## Code Style

- Use arrow functions for components
- Implement proper error handling
- Add loading states for user feedback
- Use semantic HTML elements
- Follow the established color scheme (primary: blue, secondary: green, dark theme support)

## File Organization

- Components in `/src/components/`
- Hooks in `/src/hooks/`
- Utils in `/src/utils/`
- Types in `/src/types/`
- Data/constants in `/src/data/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PedroMaxis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/PedroMaxis)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
