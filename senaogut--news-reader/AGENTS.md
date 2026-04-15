# Smart Reader - AI Coding Instructions

## Project Overview
A Flutter news aggregator with AI summarization. Currently uses **mock repositories** simulating all backend operations (web scraping, LLM API calls, database). Real implementations are planned but not yet built.

## Architecture Pattern: Simplified Clean Architecture

```
lib/
├── core/                    # Shared resources across features
│   ├── constants/          # App-wide constants
│   ├── services/           # Cross-cutting services (PromptService, StorageService)
│   ├── theme/              # AppTheme, AppColors, AppTextStyles
│   └── widgets/            # Reusable widgets
└── features/
    ├── news/               # News aggregation & summarization
    │   ├── data/
    │   │   ├── models/     # Plain Dart classes with copyWith
    │   │   └── repositories/ # Abstract interfaces + Mock implementations
    │   └── presentation/
    │       ├── providers/  # Riverpod state management
    │       ├── screens/    # Full-page widgets
    │       └── widgets/    # Feature-specific components
    └── history/            # Reading history tracking
```

### Key Architectural Decisions

**Repository Pattern**: All data access goes through abstract repository interfaces (`NewsSourceRepository`, `NewsScraperRepository`, `LlmRepository`, `HistoryRepository`). Current implementations are prefixed `Mock*` and return hardcoded data with delays to simulate network calls.

**Data Models**: Immutable classes with `copyWith` for state updates. Key models:
- `NewsSource` - Sources user can toggle (TechCrunch, BBC, etc.)
- `NewsArticle` - Individual scraped articles with `briefSummary` and `summary`
- `GroupedNews` - Similar articles grouped together from multiple sources
- `SummaryHistory` - Saved fetch sessions with timestamp

**No Domain Layer**: Business logic lives directly in Riverpod Notifiers to keep the architecture simple for this app's scope.

## State Management: Riverpod 3.0

### Provider Patterns Used

**1. AsyncNotifier for Async Data** (see `news_source_provider.dart`, `history_provider.dart`)
```dart
class NewsSourceNotifier extends AsyncNotifier<List<NewsSource>> {
  @override
  Future<List<NewsSource>> build() async {
    // Initial async load
  }
  
  Future<void> toggleSource(String id) {
    // Mutate state with ref.read(repositoryProvider)
    state = AsyncValue.data(updatedList);
  }
}
```

**2. Notifier for Sync State** (see `news_feed_provider.dart`)
```dart
class NewsFeedNotifier extends Notifier<NewsFeedState> {
  @override
  NewsFeedState build() => const NewsFeedState();
  
  Future<void> fetchNews(List<NewsSource> sources) {
    state = state.copyWith(isLoading: true);
    // ... async work ...
    state = NewsFeedState(...);
  }
}
```

**3. Provider for Dependency Injection**
```dart
final newsScraperRepositoryProvider = Provider<NewsScraperRepository>((ref) {
  return MockNewsScraperRepository(); // Swap for real implementation later
});
```

### State Access in Widgets
- Use `ConsumerWidget` or `ConsumerStatefulWidget`
- Read state: `ref.watch(newsSourceNotifierProvider)`
- Call methods: `ref.read(newsSourceNotifierProvider.notifier).toggleSource(id)`

## Critical Workflows

### Running the App
```bash
flutter pub get
flutter run
```

### Testing (Not Yet Implemented)
Testability is built-in via repository abstractions. To add tests:
1. Mock repositories are already separated - use them in tests
2. Widget tests: Wrap with `ProviderScope` and override providers
3. Unit tests: Test Notifier logic by providing mock repositories

### Adding a New Feature
1. Create feature folder under `lib/features/`
2. Add `data/models/` and `data/repositories/` (abstract + mock)
3. Add `presentation/providers/` with Riverpod notifiers
4. Add `presentation/screens/` and `presentation/widgets/`
5. Register providers in the feature's provider file

## UI/UX Conventions

### Theme System
All colors come from `core/theme/app_colors.dart`. Text styles from `core/theme/app_text_styles.dart`. Never hardcode colors or text styles - always reference these constants.

**Primary Colors**: Deep Blue (`#2563EB`), Purple accent (`#7C3AED`)

### Loading States
Use `NewsLoadingShimmer` widget for skeleton loading. All async operations must show loading UI via `isLoading` state flags.

### Navigation
Currently uses direct `Navigator.push`. No named routes or routing package. Screens are instantiated with required parameters:
```dart
Navigator.push(context, MaterialPageRoute(
  builder: (context) => ArticleDetailScreen(article: article),
));
```

### Screen Structure
`HomeScreen` uses bottom navigation with 3 tabs. FAB triggers news fetch. See `home_screen.dart:_autoFetchNews()` for initial load pattern - uses `WidgetsBinding.instance.addPostFrameCallback` to fetch after first frame.

## Integration Points

### Future LLM Integration
`LlmRepository` defines 3 methods:
- `generateBriefSummary(content)` - 1-2 sentences for cards
- `generateDetailedSummary(content)` - Full summary for detail view
- `generateCombinedSummary(articles)` - Multi-article synthesis

Prompts will be handled by `PromptService` (currently stubbed in `core/services/`).

### Future Web Scraping
`NewsScraperRepository.scrapeSource()` should use `html` package (already in dependencies). Currently returns 3 mock articles per source with 2-second delay.

### Future Database Integration
`HistoryRepository` should use `sqflite` (already in dependencies) for local persistence. Currently stores in-memory list.

## Common Pitfalls

1. **Don't bypass repositories** - Always access data through repository providers, never hardcode data in widgets
2. **Immutable state** - Use `copyWith` for updates, never mutate state directly
3. **AsyncValue handling** - Always handle `.loading`, `.error`, and `.data` cases when using `AsyncNotifier`
4. **Mock data delays** - When replacing mocks, remember to remove artificial `Future.delayed()` calls
5. **Google Fonts** - Already configured in `AppTheme`, using Epilogue font family

## Dependencies to Know

- **flutter_riverpod** (3.0.3): State management
- **dio** (5.4.3): HTTP client for future scraping
- **html** (0.15.6): HTML parsing for future scraping
- **sqflite** (2.4.2): Local database for future history persistence
- **flutter_secure_storage** (9.2.4): Secure storage (not yet used)
- **google_fonts** (6.2.1): Typography (Epilogue font)
- **cached_network_image** (3.4.1): Image caching

## Next Steps for Real Implementation

1. Replace `MockNewsScraperRepository` with real HTML scraping
2. Replace `MockLlmRepository` with OpenAI/Gemini API integration
3. Replace `MockHistoryRepository` with SQLite persistence
4. Implement `PromptService` for consistent LLM prompts
5. Add error handling and retry logic for network calls

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/senaogut) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
