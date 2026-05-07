<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Agro Advisory Bot - WhatsApp Integration

This is a Node.js WhatsApp bot project for Nigerian farmers providing agricultural advisory services.

## Project Context

- **Technology Stack**: Node.js, Express, WhatsApp Web.js
- **Target Audience**: Nigerian farmers
- **Primary Features**: Weather data, crop prices, pest control, government schemes
- **Communication**: WhatsApp messaging platform

## Code Guidelines

### WhatsApp Integration
- Use whatsapp-web.js library for WhatsApp Web API
- Handle QR code authentication properly
- Implement message parsing and response logic
- Manage user sessions and subscriptions

### Data Sources
- Integrate with weather APIs (OpenWeatherMap)
- Connect to Nigerian agricultural commodity exchanges
- Fetch government scheme information
- Provide real-time market prices

### Agricultural Focus
- Use Nigerian crop varieties and local names
- Include region-specific pest control advice
- Reference Nigerian agricultural seasons
- Support local languages where applicable

### Error Handling
- Implement robust error handling for API failures
- Provide fallback data when external APIs are unavailable
- Log errors appropriately for debugging
- Graceful degradation of features

### User Experience
- Keep messages concise and farmer-friendly
- Use emojis and formatting for better readability
- Provide clear menu options and commands
- Support both English and simple Nigerian Pidgin

### Performance
- Implement rate limiting for API calls
- Cache frequently requested data
- Optimize message sending for multiple users
- Handle high concurrent user loads

## Nigerian Agricultural Context

- **Major Crops**: Rice, maize, yam, cassava, tomato, onion
- **Growing Seasons**: Wet season (April-October), Dry season (November-March)
- **Key Markets**: Mile 12 (Lagos), Bodija (Ibadan), Gbagi (Ibadan), Alaba (Lagos)
- **Government Programs**: Anchor Borrowers Programme, GES, FADAMA

## Code Patterns

- Use async/await for asynchronous operations
- Implement proper environment variable handling
- Follow Node.js best practices for file structure
- Use meaningful variable and function names
- Add comprehensive error logging

---
> Source: [pchukwuemeka424/agro-bot](https://github.com/pchukwuemeka424/agro-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
