# Yogesh Portfolio - Development Guide

## Project Overview
A professional Next.js portfolio website showcasing web development skills, projects, experience, education, and contact information.

## Quick Start
```bash
cd yogesh-portfolio
npm install --legacy-peer-deps
npm run dev
# Visit http://localhost:3000
```

## Project Structure
- `src/app/` - Next.js App Router pages
- `src/components/` - Reusable React components
- `src/app/globals.css` - Global Tailwind styles
- Public pages: Home, Projects, Experience, Education, Skills, Contact

## Key Customization Points

### Personal Data Files to Edit:
1. **Home Page** - `src/app/page.tsx`
   - Name, bio, tagline
   - Quick stats
   - Social links

2. **Projects** - `src/app/projects/page.tsx`
   - Project list, descriptions, technologies, links

3. **Experience** - `src/app/experience/page.tsx`
   - Job positions, companies, achievements

4. **Education** - `src/app/education/page.tsx`
   - Degrees, institutions, certifications

5. **Skills** - `src/app/skills/page.tsx`
   - Technical skills with proficiency levels

6. **Contact** - `src/app/contact/page.tsx`
   - Contact info, email, phone
   - Form handling (currently logs to console)

### Components to Customize:
- `src/components/Navigation.tsx` - Logo and nav links
- `src/components/Footer.tsx` - Footer content
- `src/app/layout.tsx` - Metadata/SEO

## Design System
- **Colors**: Primary (#0f172a), Secondary (#1e293b), Accent (#3b82f6)
- **Typography**: Tailwind CSS utility classes
- **Icons**: Lucide React
- **Spacing**: Tailwind padding/margin utilities

## Styling Notes
- Global styles in `src/app/globals.css`
- Component styles use Tailwind CSS classes
- Card component with hover effects
- Responsive grid layouts
- Dark theme optimized for web dev portfolios

## Development Tips
- Hot reload enabled - changes reflect immediately
- Use TypeScript for type safety
- Add images to `public/` directory
- Test on multiple devices
- Keep content updated regularly

## Deployment
Ready to deploy on:
- Vercel (recommended - 1-click deployment)
- Netlify
- AWS Amplify
- Any Node.js hosting

## Future Enhancements
- Add testimonials section
- Add blog/articles page
- Implement email service for contact form
- Add dark/light mode toggle
- Add animations with Framer Motion
- Add search functionality
- Implement analytics

## Common Tasks

### Add a New Project
1. Edit `src/app/projects/page.tsx`
2. Find `const projects: Project[] = [`
3. Add new object to array with required fields

### Change Color Scheme
1. Edit `tailwind.config.ts`
2. Modify color values in `extend.colors`
3. Restart dev server

### Update Metadata
1. Edit `src/app/layout.tsx`
2. Modify metadata object
3. Update for SEO

### Add Images
1. Place images in `public/images/`
2. Reference as `/images/filename` in code

## Troubleshooting

### Dependencies Issue
```bash
npm install --legacy-peer-deps
```

### Port Already Used
```bash
npm run dev -- -p 3001
```

### Build Issues
```bash
rm -rf .next node_modules
npm install --legacy-peer-deps
npm run dev
```

## Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint

## Tech Stack
- Next.js 16+
- React 19+
- TypeScript 5+
- Tailwind CSS 3+
- Lucide React (icons)
- ESLint (linting)

## Performance
- Server-side rendering ready
- Optimized images
- Fast page loads with Next.js
- SEO optimized

## Accessibility
- Semantic HTML
- ARIA labels
- Keyboard navigation
- Color contrast compliance

## Resources
- [Next.js Docs](https://nextjs.org/docs)
- [Tailwind CSS](https://tailwindcss.com)
- [Lucide Icons](https://lucide.dev)
- [TypeScript](https://www.typescriptlang.org)

---

**Start customizing now and showcase your amazing work to the world!** 🚀

---
> Source: [YOGESHBOTCHA965/yogesh-portfolio](https://github.com/YOGESHBOTCHA965/yogesh-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
