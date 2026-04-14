# GitHub Copilot Instructions

## Project Overview
**Grok Chat** - A modern AI chat application featuring:
- **Frontend**: Angular 19 with TypeScript, Tailwind CSS, standalone components
- **Backend**: Node.js/Express API proxy for X.AI's Grok-4-fast-reasoning model
- **Key Features**: Real-time chat, A/B testing, response evaluation, conversation management

## Architecture

### Frontend (`grok-chat/`)
- **Framework**: Angular 19 with standalone components (no modules)
- **Styling**: Tailwind CSS + SCSS with glass morphism effects
- **State Management**: Component-based with services
- **API Communication**: HttpClient service for backend integration

### Backend (`backend/`)
- **Framework**: Express.js
- **Purpose**: API proxy to X.AI Grok API
- **Endpoints**: `/api/chat`, `/api/evaluate`, `/api/health`
- **Security**: CORS enabled, API key in environment variables

## Coding Standards

### TypeScript/Angular (Frontend)
- Use **standalone components** (no NgModules)
- Follow Angular style guide for component structure
- Use **signals** for reactive state when appropriate
- Type everything - leverage TypeScript's type safety
- Component naming: `*.component.ts` → `app.ts` (simplified)
- Use dependency injection for services
- Keep components focused - extract complex logic to services

### Node.js/Express (Backend)
- Use async/await for asynchronous operations
- Implement proper error handling with try-catch
- Return appropriate HTTP status codes
- Validate incoming requests
- Keep routes simple - extract business logic to separate functions
- Use environment variables for configuration

### Code Style
- **Indentation**: 2 spaces
- **Quotes**: Single quotes for TypeScript/JavaScript
- **Semicolons**: Required
- **Naming Conventions**:
  - Variables/functions: camelCase
  - Classes/Interfaces: PascalCase
  - Constants: UPPER_SNAKE_CASE
  - Files: kebab-case

## Grok API Integration

### Important Notes
- **Model**: Always use `grok-4-fast-reasoning`
- **Temperature Range**: 0-2 (not 0-1 like some models)
- **System Prompts**: Always include as first message
- **API Endpoint**: `https://api.x.ai/v1/chat/completions`
- **Authentication**: Bearer token via `XAI_API_KEY`

### Temperature Guidelines
- **0.3**: Focused, deterministic responses
- **0.7**: Balanced creativity and consistency (default)
- **1.0**: More creative and varied
- **1.5+**: Highly creative, experimental

## Feature Implementation Guidelines

### When Adding New Features
1. Update both frontend service and backend endpoint
2. Add proper TypeScript interfaces for new data structures
3. Implement error handling on both layers
4. Update UI to reflect new functionality
5. Consider responsive design for mobile
6. Add user feedback (loading states, error messages)

### UI/UX Standards
- Maintain glass morphism aesthetic (backdrop-blur, gradients)
- Use emoji consistently in UI labels
- Smooth animations (150-300ms transitions)
- Purple/pink/blue color scheme
- Mobile-first responsive design
- Accessibility: proper ARIA labels, keyboard navigation

## Error Handling

### Frontend
```typescript
// Always handle HTTP errors gracefully
this.chatService.sendMessage(messages, systemPrompt, temperature)
  .subscribe({
    next: (response) => { /* handle success */ },
    error: (error) => {
      console.error('Error:', error);
      // Show user-friendly message
      this.errorMessage = 'Failed to send message. Please try again.';
    }
  });
```

### Backend
```javascript
// Always wrap API calls in try-catch
try {
  const response = await axios.post(GROK_API_URL, data, config);
  res.json(response.data);
} catch (error) {
  console.error('Error:', error.response?.data || error.message);
  res.status(error.response?.status || 500).json({ 
    error: error.response?.data?.error?.message || 'Friendly error message' 
  });
}
```

## Testing
- Write unit tests for services and utility functions
- Test API endpoints with various inputs
- Test error scenarios
- Ensure responsive design works on different screen sizes
- Test with different Grok API responses

## Security Considerations
- **Never** commit API keys or secrets
- Use environment variables for sensitive data
- Validate and sanitize user inputs
- Implement rate limiting for production
- Configure CORS properly for production (don't use `*`)
- Add request size limits

## Deployment

### Pre-Deployment Checklist
- [ ] Update API URLs from localhost to production
- [ ] Configure environment variables
- [ ] Build frontend for production (`ng build --configuration production`)
- [ ] Test in production-like environment
- [ ] Set up proper CORS policies
- [ ] Enable HTTPS
- [ ] Set up monitoring/logging

### Recommended Platforms
- **Frontend**: Vercel, Netlify, GitHub Pages
- **Backend**: Vercel, Railway, Render, Heroku
- **Full Stack**: Docker + any cloud provider

## Environment Variables

### Backend Required
```
XAI_API_KEY=your_api_key_here
PORT=3000
NODE_ENV=production
ALLOWED_ORIGINS=https://your-frontend-url.com
```

### Frontend (environment files)
```typescript
export const environment = {
  production: true,
  apiUrl: 'https://your-backend-url.com/api'
};
```

## Dependencies Management
- Keep Angular and dependencies up to date
- Review security advisories regularly (`npm audit`)
- Pin major versions in package.json
- Test thoroughly after dependency updates
- Document any breaking changes

## Git Workflow
- **Branches**: Feature branches from `main`
- **Commits**: Clear, descriptive messages with emoji prefixes
  - ✨ New feature
  - 🐛 Bug fix
  - 📝 Documentation
  - 🎨 UI/styling
  - ♻️ Refactoring
  - 🚀 Deployment/performance
- **PRs**: Required for main branch, include description and testing notes

## Additional Notes
- This is a showcase project demonstrating Grok API integration
- Prioritize user experience and visual polish
- Performance matters - optimize bundle size and API calls
- Keep the UI fun and engaging with emojis and animations
- Document any new features in the README

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Phazzie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
