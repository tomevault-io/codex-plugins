<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# AgriMap PH - Copilot Instructions

This is a Next.js React web application for **AgriMap PH** - a real-time, AI-enhanced, crowdsourced agriculture pricing and demand-supply mapping system for the Philippines.

## Project Context

### Features
- Crowdsourced data input for agricultural products
- Real-time pricing and market condition tracking
- OpenStreetMap integration with heatmap overlays
- User type selection (Buyer, Farmer, Regular User)
- Weather API integration
- Firebase real-time database for data sync
- Basic AI recommendations for optimal locations

### Tech Stack
- **Frontend**: Next.js 15 with TypeScript and App Router
- **Styling**: Tailwind CSS
- **Mapping**: Leaflet.js with React-Leaflet
- **Backend**: Firebase Realtime Database
- **Icons**: Lucide React
- **Weather**: OpenWeatherMap API (planned)

### Key Components
- `MapView`: OpenStreetMap with overlays and heatmaps
- `InputForm`: User data entry for prices and conditions
- `UserTypeSelector`: Buttons for Buyer/Farmer/Regular User
- `ProductSelector`: Dropdown for agricultural products
- `DataFetcher`: Weather, location, and time data
- `HeatmapOverlay`: Visual heat logic for supply/demand
- `AIRecommender`: Location suggestions for users

### Code Guidelines
- Use TypeScript for all components
- Follow Next.js App Router patterns
- Use Tailwind CSS for styling
- Implement responsive, mobile-first design
- Use React hooks for state management
- Keep components modular and reusable
- Focus on data collection and visualization
- Ensure maps work offline with cached tiles when possible

### Data Structure
- Agricultural products with pricing
- Location data (GPS coordinates, barangay info)
- Market conditions (Normal, Panic Buying, Overstocked)
- Traffic status (Light, Moderate, Heavy)
- Weather data integration
- User type context for recommendations

### Philippines-Specific Considerations
- Use Philippine agricultural products in dropdowns
- Implement barangay-level location tracking
- Consider local market patterns and seasonality
- Use Filipino terms where appropriate in UI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qppd)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/qppd)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
