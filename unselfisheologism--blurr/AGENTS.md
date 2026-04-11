# Copilot Instructions for Blurr Android App

## Project Overview
Blurr is an Android application that functions as an AI assistant with voice control capabilities. It allows users to interact with their devices using natural language commands.

## Technology Stack
- **Language**: Kotlin
- **Framework**: Android SDK
- **Architecture**: MVVM with coroutines for asynchronous operations
- **AI Services**: Google Gemini API for natural language processing
- **Cloud Services**: Puter.js for authentication, storage, and key-value database
- **Voice Recognition**: Android's built-in speech recognition
- **UI Framework**: Jetpack Compose and traditional XML layouts

## Key Components
1. **Voice Interaction**: Users can activate the assistant using voice commands or a floating button
2. **Task Automation**: The app can perform various tasks like opening apps, sending messages, setting alarms, etc.
3. **Screen Analysis**: Uses accessibility services to understand the current screen context
4. **Memory System**: Stores user preferences and interaction history
5. **Authentication**: Uses Puter.js for user authentication

## Development Environment
- Android Studio Ladybug or newer
- Kotlin 1.9+
- Android SDK 34 (Android 14)
- Minimum SDK 24 (Android 7.0)

### Required SDKs and Tools
- Android SDK Platform 34
- Android SDK Build-Tools 34.0.0
- Android SDK Command-line Tools
- Android Emulator (optional, for testing)
- JDK 17

### API Keys and Configuration
API keys are managed through local.properties file in the project root:
```properties
sdk.dir=/path/to/your/android/sdk
GEMINI_API_KEYS=your_comma_separated_gemini_keys
TAVILY_API=your_tavily_api_key
MEM0_API=your_mem0_api_key
```

### Build Commands
```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Install debug APK to connected device
./gradlew installDebug

# Run unit tests
./gradlew test

# Run instrumented tests
./gradlew connectedAndroidTest
```

## Architecture Patterns
- **MVVM**: Model-View-ViewModel pattern for separation of concerns
- **Repository Pattern**: Abstracts data sources
- **Dependency Injection**: Manual DI for better testability
- **Coroutines**: For asynchronous operations and threading

## Code Structure
```
app/src/main/java/com/blurr/voice/
├── agents/                 # AI agents for specific tasks
├── api/                    # API clients for external services
├── data/                   # Data models and repositories
├── ui/                     # UI components and activities
├── utilities/              # Helper classes and utilities
├── services/               # Background services
└── managers/               # Manager classes for specific functionalities
```

## Important Classes and Files
- `ConversationalAgentService.kt`: Main service handling voice interactions
- `MainActivity.kt`: Entry point of the application
- `LoginActivity.kt`: Handles user authentication
- `PuterManager.kt`: Manages Puter.js integration
- `TTSManager.kt`: Text-to-speech functionality
- `SpeechCoordinator.kt`: Coordinates speech recognition and synthesis

## Testing
- Unit tests are located in `src/test/`
- Instrumentation tests are in `src/androidTest/`
- Use JUnit 4 for unit tests
- Use Espresso for UI tests

## Debugging Tips
Monitor logs using Logcat with these tags:
```bash
adb logcat | grep GeminiApi
```

### Required API Services
- Gemini API: https://makersuite.google.com/app/apikey
- Tavily: https://tavily.com/
- Mem0: https://mem0.ai/

### File Locations
- Main source: `app/src/main/java/com/blurr/voice/`
- Tests: `app/src/test/java/com/blurr/voice/`
- Resources: `app/src/main/res/
- Build config: `app/build.gradle.kts`
- Dependencies: `gradle/libs.versions.toml` (AGP version 8.9.2 - NEVER change to 8.5.2)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unselfisheologism)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Unselfisheologism)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
