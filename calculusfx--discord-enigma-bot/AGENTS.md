# Discord Bot - Copilot Instructions

## Project Overview
A comprehensive Discord bot with the following features:
- Music playback from various platforms (YouTube, Spotify, SoundCloud)
- Voice channel join/leave TTS announcements
- AI-powered content moderation
- Link filtering for gambling/illegal content
- Image/video content analysis

## Tech Stack
- **Runtime**: Node.js 18+
- **Framework**: Discord.js v14
- **Music**: discord-player, @discord-player/extractor
- **TTS**: Google TTS / Discord TTS
- **AI/ML**: OpenAI API for content moderation
- **Database**: SQLite (for learning/storing moderation data)

## Project Structure
```
├── src/
│   ├── index.js          # Bot entry point
│   ├── config.js         # Configuration loader
│   ├── commands/         # Slash commands
│   │   ├── music/        # Music commands
│   │   └── moderation/   # Moderation commands
│   ├── events/           # Discord events
│   ├── services/         # Core services
│   │   ├── music/        # Music player service
│   │   ├── tts/          # Text-to-speech service
│   │   └── moderation/   # Content moderation service
│   └── utils/            # Utility functions
├── data/                 # SQLite database & learned data
├── .env.example          # Environment variables template
├── package.json
└── README.md
```

## Development Guidelines
- Use ES6+ features with ESM modules
- Follow Discord.js best practices
- Implement proper error handling
- Use environment variables for sensitive data
- Log important events for debugging

---
> Source: [CalculusFx/discord-enigma-bot](https://github.com/CalculusFx/discord-enigma-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
