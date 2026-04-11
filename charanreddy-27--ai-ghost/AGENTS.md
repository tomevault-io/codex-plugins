<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# AI Ghost Mode - Digital Footprint Minimizer

## Project Overview
Privacy-focused AI tool that audits public social media data (Reddit, GitHub, Instagram) to identify privacy risks and suggest cleanup actions using ML-powered analysis.

## Architecture & Data Flow
**Dual-Service Architecture**: FastAPI backend (port 8000) + Streamlit frontend (port 8501)
- **Data Flow**: Scrapers → RiskAnalyzer → AnalysisResult → Frontend Dashboard
- **Core Components**: 
  - `src/scrapers/` - Platform data collectors (async/await patterns)
  - `src/analyzers/risk_analyzer.py` - AI risk detection engine
  - `src/frontend/dashboard.py` - Streamlit multi-page UI
  - `src/config.py` - Centralized configuration with `Config` class

## Key Implementation Patterns

### Configuration Management
- **Single Config Class**: `src/config.py` exports singleton `Config` class
- **Import Fix**: Use `sys.path.append(os.path.dirname(os.path.dirname(__file__)))` to resolve relative imports
- **Environment Variables**: All settings from `.env` with sensible defaults (see `.env.example`)

### Risk Analysis Architecture
- **Dataclass Pattern**: `RiskItem` and `AnalysisResult` for structured data
- **Model Loading**: AI models loaded once in `RiskAnalyzer.__init__()` with graceful fallback
- **Risk Scoring**: 0-10 scale based on severity weights (`{"low": 1, "medium": 3, "high": 5}`)
- **Cross-Platform Identity Linking**: String similarity using Levenshtein distance

### AI Model Integration
- **spaCy NER**: `en_core_web_sm` for person/location detection
- **HuggingFace Pipelines**: 
  - Sentiment: `cardiffnlp/twitter-roberta-base-sentiment-latest`
  - Toxicity: `unitary/toxic-bert`
- **Regex Patterns**: Email (`r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'`) and phone detection

### Async Data Collection
- **Dataclass Models**: `RedditPost`, `RedditComment`, `RedditProfile` for typed data
- **PRAW Integration**: Reddit API with proper error handling and rate limiting
- **Concurrent Analysis**: `asyncio.gather()` for parallel platform processing

## Developer Workflows

### Development Setup
```bash
# Environment setup (VS Code tasks available)
.venv/Scripts/python.exe -m spacy download en_core_web_sm
# Start services via tasks: "Start AI Ghost Mode - Backend API" & "Start AI Ghost Mode - Streamlit Dashboard"
```

### Key File Relationships
- **Scrapers**: Import `config.Config` for API credentials
- **Analyzers**: Accept scraper output, return `AnalysisResult` objects
- **Frontend**: Polls backend API, displays risk visualizations
- **Testing**: `pytest tests/` with mocked external API calls

## Privacy & Security Constraints
- **Local Processing Only**: No external data transmission
- **Public Data Only**: Respect platform ToS and rate limits
- **Confidence Scoring**: All AI predictions include confidence levels
- **Logging**: Use `loguru` with file rotation, never log personal data

## Project-Specific Conventions
- **Risk Types**: "Email Exposure", "Phone Number", "Toxic Content", "Username Similarity"
- **Severity Levels**: "low", "medium", "high" (lowercase strings)
- **Platform Context**: Include platform name in risk item context field
- **Async Methods**: All scraper methods are async, return typed dataclass lists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/charanreddy-27)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/charanreddy-27)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
