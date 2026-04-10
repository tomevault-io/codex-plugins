# Copilot Instructions for General Security Knowledge HOF 2026

## Repository Overview
This repository contains an interactive security essentials presentation for Full Sail University Hall of Fame 2026. The presentation is built using Reveal.js and covers essential security knowledge for technology professionals.

## Project Structure
- `index.html` - Main presentation file containing all slides and styling
- `README.md` - Comprehensive documentation about the presentation

## Technology Stack
- **Reveal.js 4.6.1** - Core presentation framework
- **Font Awesome 6.5.1** - Icons
- **Google Fonts** - Typography (Montserrat, Roboto Mono)
- **HTML/CSS/JavaScript** - No build process required

## Development Guidelines

### When Working with HTML
- All presentation content is in a single `index.html` file
- Follow the existing slide structure using `<section>` tags
- Maintain consistency with the existing Full Sail-inspired dark theme
- Use CSS variables defined in `:root` for colors
- Preserve the responsive design features

### When Adding New Slides
- Wrap new slides in `<section>` tags
- Use `class="fragment"` for progressive disclosure of content
- Follow the existing pattern for icons using Font Awesome
- Keep slide content concise and visually appealing
- Use the established color scheme and gradients

### Styling Guidelines
- Primary color: `#00d4ff` (cyan/blue)
- Secondary color: `#ff6b35` (orange for warnings)
- Accent color: `#7b2cbf` (purple)
- Background: `#0a0e27` (dark)
- Font: Montserrat (headings), Roboto Mono (code)
- Maintain high contrast for readability

### Content Guidelines
- Keep security content current and accurate
- Use real statistics and cite sources when possible
- Include practical, actionable advice
- Make technical concepts accessible to varied audiences
- Use visual elements (icons, boxes, gradients) for engagement

### Security Content Topics
The presentation covers:
1. Phishing attacks and recognition
2. Social engineering tactics
3. Malware and ransomware
4. Password security best practices
5. Two-factor authentication
6. Data privacy protection
7. Incident response procedures
8. Security culture development

### Testing Changes
- Open `index.html` in a modern browser (Chrome/Firefox/Safari)
- Test in fullscreen mode (press F)
- Verify all slides are accessible via navigation
- Check that fragments display correctly
- Test responsive design on different screen sizes
- Ensure no dependencies are broken (CDN links work)

### No Build Process
- This is a static HTML file with CDN dependencies
- No compilation, bundling, or build steps required
- Changes are immediately visible by refreshing the browser
- No package manager or dependencies to install

### Accessibility Considerations
- Maintain high contrast color scheme
- Keep fonts large and readable
- Ensure keyboard navigation works (arrow keys, ESC, F)
- Preserve semantic HTML structure
- Test with screen readers if adding complex content

## Best Practices
- Keep the presentation self-contained (single HTML file)
- Use CDN links for external dependencies
- Optimize images if adding any (keep file size small)
- Test offline functionality (CDN resources are cached)
- Maintain mobile responsiveness
- Document significant changes in README.md

## When Asked to Modify the Presentation
1. Always preserve the existing dark theme and Full Sail branding
2. Keep slide transitions and animations consistent
3. Follow the established visual hierarchy
4. Test changes in a browser before committing
5. Update README.md if adding new sections or features
6. Maintain backward compatibility with Reveal.js 4.6.1

## Common Tasks
- **Adding slides**: Insert new `<section>` within appropriate parent section
- **Changing colors**: Update CSS variables in `:root`
- **Adding icons**: Use Font Awesome classes (fa-*)
- **Progressive reveals**: Add `class="fragment"` to elements
- **Updating statistics**: Modify stat-box elements
- **Adding warnings**: Use warning-box or success-box classes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mlaplante)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mlaplante)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
