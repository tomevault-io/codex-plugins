# Macro Market Analyzer - Development Instructions

This project is a full-stack economic analysis dashboard built with Next.js 14 and Express.js.

## Project Structure

### Frontend (Next.js 14 App Router)
- TypeScript with strict mode
- Tailwind CSS for styling
- Recharts for data visualization
- Responsive design with dark mode support

### Backend (Express.js)
- RESTful API architecture
- Data fetching from Yahoo Finance and World Bank APIs
- CSV fallback for reliability
- Statistical analysis services
- In-memory caching (1-hour TTL)

## Key Features
1. Interactive data visualizations (line charts, scatter plots, heatmaps)
2. Statistical analysis (correlations, moving averages, volatility)
3. Auto-generated insights based on data patterns
4. Date range filtering and period comparison
5. CSV data export functionality

## Development Guidelines

### Code Style
- Use TypeScript for type safety
- Follow React best practices (hooks, functional components)
- Use async/await for asynchronous operations
- Implement proper error handling
- Add comments for complex logic

### Component Organization
- Keep components focused and reusable
- Use proper TypeScript interfaces
- Implement loading and error states
- Follow responsive design principles

### API Development
- Validate all inputs
- Use proper HTTP status codes
- Implement error handling middleware
- Cache expensive operations
- Log important events

### Data Handling
- Always filter by date for consistency
- Handle missing data gracefully
- Validate data formats
- Use proper number formatting

## Important Notes

### Disclaimer Requirements
- Prominent disclaimer on all pages
- Educational/research purpose only
- No investment advice language
- Past performance warnings

### Performance
- Charts are lazy loaded
- API responses cached for 1 hour
- CSV fallback for reliability
- Responsive images and components

### Testing Checklist
- [ ] All pages load without errors
- [ ] Charts render with data
- [ ] Date filtering works
- [ ] CSV export functions
- [ ] Dark mode toggles correctly
- [ ] Mobile responsiveness verified
- [ ] API endpoints respond correctly
- [ ] Error states display properly

## Common Tasks

### Adding New Data Source
1. Update dataService.js with new fetch function
2. Add CSV fallback file
3. Update types.ts with new interfaces
4. Create corresponding route and controller
5. Update frontend API client

### Adding New Visualization
1. Create chart component in components/charts/
2. Use Recharts library
3. Implement loading and error states
4. Make responsive
5. Add to appropriate page

### Modifying Analysis
1. Update analysisService.js
2. Add new statistical methods
3. Update types if needed
4. Create/update API endpoint
5. Update frontend to use new analysis

## Troubleshooting

### Data Not Loading
- Check backend is running (port 5000)
- Verify API_URL in frontend .env.local
- Check CSV files exist and are formatted correctly
- Review browser console for errors

### Build Errors
- Clear .next directory: `rm -rf .next`
- Delete node_modules: `rm -rf node_modules`
- Reinstall: `npm install`
- Check TypeScript errors: `npm run build`

### Chart Not Rendering
- Verify data format matches component props
- Check for undefined values
- Ensure ResponsiveContainer has proper dimensions
- Review browser console for errors

## Resources

- Next.js Docs: https://nextjs.org/docs
- Recharts Docs: https://recharts.org
- Express.js Docs: https://expressjs.com
- Tailwind CSS: https://tailwindcss.com

## Quick Commands

```bash
# Frontend
cd frontend
npm run dev      # Development
npm run build    # Production build
npm run lint     # Lint code

# Backend
cd backend
npm start        # Production
npm run dev      # Development with nodemon
```

## Environment Variables

### Frontend (.env.local)
```
NEXT_PUBLIC_API_URL=http://localhost:5000
```

### Backend (.env)
```
PORT=5000
NODE_ENV=development
CORS_ORIGIN=http://localhost:3000
ALPHA_VANTAGE_API_KEY=optional_key_here
```

## File Locations

- **Pages:** frontend/app/
- **Components:** frontend/components/
- **API Client:** frontend/lib/api.ts
- **Backend Routes:** backend/src/routes/
- **Data Services:** backend/src/services/
- **CSV Data:** backend/src/data/

---

For more details, see README.md, QUICKSTART.md, and API_DOCUMENTATION.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/divyashhree)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/divyashhree)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
