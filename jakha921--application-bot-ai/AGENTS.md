# Ariza AI Bot - AI Coding Guidelines

## Project Overview

This is an AI-powered Telegram bot system for generating legal documents (заявления/ariza) for Uzbekistan citizens. The project has two implementations:

1. **Django Application** (`app/`) - Production-ready standalone bot ⭐
2. **n8n Workflow** (`docs/`) - Original prototype/reference implementation

## Architecture Components

### Primary: Django + aiogram Application (`app/`)
**Full-stack production system** with:
- **Django framework** - web framework, ORM, admin panel
- **aiogram 3.x** - modern async Telegram bot library
- **PostgreSQL** - persistent storage for users, conversations, documents
- **Redis** - FSM state storage and caching
- **AI services** - OpenAI (Whisper + GPT) or Google Gemini (configurable)
- **python-docx** - Word document generation
- **Docker** - containerized deployment

**Data flow**:
```
Telegram User → aiogram Bot → [Whisper/Gemini Transcription] 
    → [OpenAI/Gemini Dialogue] → Word Generator → PostgreSQL → User
                                      ↓
                              Django Admin Panel
```

### Alternative: n8n Workflow (`docs/`)
**Original prototype** with:
- **n8n workflows** orchestrate the entire process from Telegram messages to document delivery
- **Flask API service** handles Word document generation using python-docx
- **AI services**: OpenAI Whisper (voice→text), Anthropic Claude (intelligent dialogue)
- **Document pipeline**: Voice/Text → Transcription → AI Dialogue → Structured Document → Word File

**Data flow**:
```
Telegram User → n8n Webhook → [Whisper API | Claude API | Flask API] → Telegram Bot
```

## Critical Developer Patterns

### Django Application Patterns (`app/`)

#### 1. Django Project Structure
```
app/
├── config/          # Django settings, URLs, WSGI
├── core/            # Database models (User, Conversation, Message, Document)
├── bot/             # aiogram handlers, webhook, management commands
├── ai_services/     # OpenAI/Gemini provider abstraction
├── documents/       # Word generator (based on docs/Flask API)
├── manage.py
├── requirements.txt
└── docker-compose.yml
```

#### 2. AI Service Provider Pattern
```python
# Switchable AI providers via settings.AI_PROVIDER
ai_service = get_ai_service()  # Returns OpenAI or Gemini
transcription = ai_service.transcribe_audio(audio_file, language='uz')
response = ai_service.chat_completion(messages, system_prompt=ARIZA_SYSTEM_PROMPT)
```

#### 3. Bot Handler Pattern (aiogram)
```python
# FSM state management with Redis
@router.message(F.voice, ArizaStates.waiting_for_input)
async def handle_voice(message: Message, state: FSMContext):
    user = await get_or_create_user(message.from_user)
    conversation = await get_or_create_conversation(user)
    # Process with AI...
    await process_user_message(message, conversation, text, state)
```

#### 4. Database Access Pattern
```python
# Use sync_to_async for Django ORM in async handlers
@sync_to_async
def get_or_create_user(telegram_user):
    user, created = TelegramUser.objects.get_or_create(...)
    return user
```

#### 5. Document Generation Pattern
```python
# Same core logic as Flask API, adapted for Django
from documents.generator import generate_ariza_document
doc_bytes = generate_ariza_document(document_text)
await save_document(conversation, filename, doc_bytes, document_text)
```

#### 6. Deployment Modes
- **Polling mode** (development): `python manage.py runbot`
- **Webhook mode** (production): `python manage.py setwebhook` + gunicorn

### n8n Workflow Development Patterns (`docs/`)

#### 1. n8n Workflow Development
- All business logic lives in `docs/N8n Telegram Voice Workflow.json`
- Use the provided workflow as the canonical implementation
- JavaScript code nodes handle conversation state management and parsing
- Credentials must be configured: Telegram Bot Token, OpenAI API, Anthropic API

#### 2. Flask API Patterns (`docs/Flask API Server for Telegram Workflow.py`)
```python
# Document generation follows this specific pattern:
generator = ArizaDocumentGenerator()
parsed_data = generator.parse_ariza_text(document_text)
generator.generate_document(parsed_data)
file_stream = generator.save_to_bytes()
```

### 3. Document Structure Parsing
Documents follow Uzbek legal format:
- Header (right-aligned recipient info)
- Title: "А Р И З А" (center-aligned)
- Body (justified text with first-line indent)
- Appendix: "Илова:" section
- Footer: Date and signature line

### 4. Multi-Language Support
- Voice transcription: Uzbek (`uz`) with Cyrillic prompt for Whisper
- UI/System messages: Mixed Russian/Uzbek
- Legal content: Uzbek Cyrillic as required by law

## Essential Workflows

### Django Application Workflows

#### Quick Start (Docker)
```bash
cd app
cp .env.example .env  # Configure API keys
docker-compose up -d
docker-compose exec app python manage.py migrate
docker-compose exec app python manage.py createsuperuser
# Bot runs automatically in polling mode
```

#### Local Development
```bash
cd app
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Configure
python manage.py migrate
python manage.py runbot  # Start bot in polling mode
```

#### Management Commands
```bash
python manage.py runbot              # Start bot (polling)
python manage.py setwebhook --url    # Configure webhook
python manage.py createsuperuser     # Create admin user
```

#### Admin Panel
```
http://localhost:8000/admin/
- Manage users, conversations, messages
- View generated documents
- Access daily statistics
```

### n8n Workflow Setup (Legacy)
```bash
# Quick Docker-based setup
docker-compose up -d  # Runs n8n + Flask + Redis
# Configure credentials in n8n UI
# Import workflow JSON
# Test with voice/text messages
```

### Debugging Workflows

#### Django Application Debug
```bash
# View logs
docker-compose logs -f app

# Check bot status
docker-compose exec app python manage.py shell
>>> from bot.bot import bot
>>> import asyncio
>>> asyncio.run(bot.me())

# Test AI service
>>> from ai_services.providers import get_ai_service
>>> ai = get_ai_service()
>>> ai.chat_completion([{"role": "user", "content": "Test"}])
```

#### n8n Workflow Debug
```bash
# Test Flask API directly
curl -X POST http://localhost:5000/parse-ariza \
  -H "Content-Type: application/json" \
  -d '{"text": "document_text_here"}'
```

## File Locations & Entry Points

### Django Application (`app/`)
- **Main entry**: `app/manage.py`
- **Settings**: `app/config/settings.py`
- **Bot handlers**: `app/bot/handlers.py`
- **AI services**: `app/ai_services/providers.py`
- **Word generator**: `app/documents/generator.py`
- **Models**: `app/core/models.py`
- **Quick start**: `app/QUICKSTART.md`
- **Full docs**: `app/README.md`

### n8n Workflow (`docs/`)
- **Main workflow**: `docs/N8n Telegram Voice Workflow.json`
- **Flask service**: `docs/Flask API Server for Telegram Workflow.py`
- **Setup guide**: `docs/N8n Telegram Voice Workflow Guide.md`
- **Quick start**: `docs/N8n Telegram Voice Workflow Checklist.md`
- **Docker config**: `docs/N8n Telegram Voice Workflow Setup.txt`

When working on this project, always reference the comprehensive documentation in the `docs/` folder and test the complete pipeline from Telegram → n8n → AI services → document generation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakha921) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
