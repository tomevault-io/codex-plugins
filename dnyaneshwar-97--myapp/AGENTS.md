# GitHub Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview

This is a Flutter mobile application for managing government services with Firebase integration. The app includes both user and admin functionality.

## Key Technologies

- **Flutter/Dart**: Mobile app framework
- **Firebase Authentication**: Phone Auth (users) and Email/Password (admins)
- **Cloud Firestore**: Database for storing users, admins, services, and applications
- **Firebase Security Rules**: Access control and data validation
- **Cloud Functions**: Admin role management and backend logic

## Architecture Guidelines

### State Management
- Use Provider pattern for dependency injection
- StreamBuilder for real-time data updates from Firestore
- FutureBuilder for one-time data fetching

### Code Organization
- Models: Data classes with Firestore serialization methods
- Services: Business logic and Firebase interactions
- Screens: UI components organized by user type (user/admin)
- Widgets: Reusable UI components

### Firebase Integration
- Always check authentication state before data operations
- Use proper error handling for Firebase operations
- Implement offline capabilities where appropriate
- Follow Firebase security best practices

### UI/UX Guidelines
- Material Design components
- Responsive layouts for different screen sizes
- Loading states for async operations
- Proper error messaging and user feedback
- Accessibility considerations

## Security Considerations

- Validate user input on both client and server side
- Use Firebase Security Rules for data access control
- Implement proper authentication checks
- Never expose sensitive data in client code
- Use Cloud Functions for sensitive server-side operations

## Data Models

### User Flow
- Phone authentication → User profile creation → Service browsing → Application submission → Status tracking

### Admin Flow
- Email authentication → Dashboard overview → Service management → Application processing → Admin management

## Common Patterns

### Error Handling
```dart
try {
  // Firebase operation
} catch (e) {
  if (mounted) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Error: $e')),
    );
  }
}
```

### Loading States
```dart
bool _isLoading = false;

// In UI
child: _isLoading
  ? CircularProgressIndicator()
  : ElevatedButton(...)
```

### Firestore Streams
```dart
StreamBuilder<List<Model>>(
  stream: firestoreService.getCollection(),
  builder: (context, snapshot) {
    // Handle loading, error, and data states
  },
)
```

## Development Guidelines

- Use const constructors where possible
- Implement proper disposal of controllers and streams
- Follow Dart/Flutter naming conventions
- Add comprehensive error handling
- Use meaningful variable and function names
- Write self-documenting code with appropriate comments
- Test on both Android and iOS platforms

---
> Source: [Dnyaneshwar-97/MyApp](https://github.com/Dnyaneshwar-97/MyApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
