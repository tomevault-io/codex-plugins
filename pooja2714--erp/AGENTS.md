# EduERP Assistant - Development Guide

## Project Overview

EduERP Assistant is a modern, responsive ERP (Enterprise Resource Planning) web application frontend for educational institutions. Built with React 18, TypeScript, Material UI (MUI v5), and Framer Motion, it provides a production-ready dashboard with an integrated AI Chat Assistant.

## Technology Stack

- React 18 with TypeScript
- Material UI (MUI) v5 for component library
- Framer Motion for animations
- React Router v6 for navigation
- Vite 5 for build tooling
- Emotion for CSS-in-JS styling
- Axios for HTTP requests

## Project Structure

```
src/
├── components/
│   ├── chat/               # Chat interface components
│   ├── layout/             # Layout components (Header, Sidebar)
│   └── PlaceholderPage.tsx # Placeholder pages
├── context/                # Context providers (Theme, Chat)
├── theme/                  # Material UI theme configuration
├── types/                  # TypeScript type definitions
├── App.tsx                # Main app component with routing
├── main.tsx               # React entry point
└── index.css              # Global styles
```

## Key Features

### Chat Assistant
- Real-time messaging interface
- Message bubbles with timestamps
- Typing indicator animation
- Auto-scroll to latest messages
- Enter key submission support
- Emoji picker support (UI ready)

### Theme System
- Light and dark mode support
- Persistent theme preference
- Smooth theme transitions
- Material Design color palette

### Responsive Design
- Mobile-first approach
- Collapsible sidebar on mobile
- Optimized touch interactions
- Responsive typography and spacing

### Navigation
- Sidebar with collapsible menu on mobile
- Active route highlighting
- Smooth page transitions
- Multiple module pages (Dashboard, Students, Teachers, Attendance, Grades, Reports)

## Development Workflow

### Setting Up the Development Environment

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```

3. Build for production:
   ```bash
   npm run build
   ```

### Component Development Guidelines

- Use functional components with hooks
- Leverage Material UI components for consistency
- Apply Framer Motion for animations
- Use TypeScript for type safety
- Keep components modular and reusable

### Customization

#### Adding New Menu Items
Edit `src/components/layout/Sidebar.tsx`:
1. Add entry to `menuItems` array
2. Create corresponding page component
3. Add route in `src/App.tsx`

#### Changing Theme Colors
Edit `src/theme/theme.ts`:
- Primary color: `palette.primary.main`
- Secondary color: `palette.secondary.main`
- Dark mode colors: `palette.background.default` and `palette.background.paper`

#### Integrating Real Chat API
In `src/components/chat/ChatPage.tsx`:
1. Replace the mock setTimeout with actual API call
2. Use axios for HTTP requests
3. Handle loading and error states
4. Update TypeScript interfaces as needed

## Code Standards

- Use TypeScript for type safety
- Follow Material Design principles
- Keep components under 300 lines (split into smaller components)
- Use meaningful variable and function names
- Add JSDoc comments for complex logic
- Use CSS-in-JS with Material UI sx prop

## Performance Considerations

- Animations use GPU acceleration (transform, opacity)
- Scroll virtualization for large message lists (future enhancement)
- Lazy loading for route components (future enhancement)
- Efficient re-renders with React best practices
- Memoization for expensive components

## Browser Support

- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)

## Common Tasks

### Add a New Page
1. Create component in `src/components/`
2. Add route in `src/App.tsx`
3. Add menu item in `src/components/layout/Sidebar.tsx`

### Customize Colors
Edit palette settings in `src/theme/theme.ts`

### Change Animations
Modify motion configuration in component files (look for `<motion.div>` elements)

### Integrate API
Update context files in `src/context/` or use Axios directly in components

## Deployment

Build the production bundle:
```bash
npm run build
```

The `dist/` folder is ready for deployment on any static hosting service (Vercel, Netlify, GitHub Pages, etc.).

## Future Enhancements

- Real API integration
- Student management module
- Teacher management module
- Attendance tracking
- Grade management
- Advanced reporting
- Real-time notifications
- File management
- Multi-language support

## Troubleshooting

### Port Already in Use
Change the dev server port in `vite.config.ts`:
```typescript
server: {
  port: 5174,  // Change this
}
```

### Build Fails
1. Clear node_modules: `rm -rf node_modules`
2. Reinstall dependencies: `npm install`
3. Try building again: `npm run build`

### Theme Not Updating
Check localStorage for saved theme preference:
```javascript
localStorage.getItem('theme-mode')
```

## Resources

- [React Documentation](https://react.dev)
- [Material UI Documentation](https://mui.com)
- [Framer Motion Documentation](https://www.framer.com/motion)
- [React Router Documentation](https://reactrouter.com)
- [TypeScript Handbook](https://www.typescriptlang.org/docs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pooja2714)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Pooja2714)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
