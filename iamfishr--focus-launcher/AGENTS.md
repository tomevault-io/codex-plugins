# Focus Launcher - Development Instructions

Focus Launcher is a minimalist Android launcher application built with Flutter. It's designed to provide a distraction-free phone experience by showing only the essential apps and features.

## Project Overview

- **Type**: Mobile application (Android launcher)
- **Framework**: Flutter
- **Architecture**: Provider pattern for state management
- **Target Platforms**: Android (primary), with potential for other platforms

## Key Features

- Minimalist home screen with clock and essential information
- Start menu for accessing installed applications
- App drawer for all installed applications
- Settings for customization (themes, wallpapers, etc.)
- Favorite apps selection for quick access

## Core Components

1. **Home Screen**: The main interface showing time, date, and providing access to the start menu
2. **Start Menu**: A menu providing access to commonly used apps
3. **App Drawer**: A screen showing all installed applications
4. **Settings**: Configuration options for the launcher
5. **Theme Management**: Light/dark theme support

## Project Structure

- **lib/**
  - **main.dart**: Entry point of the application
  - **app_launcher.dart**: Interface for launching apps
  - **theme_notifier.dart**: Theme state management
  - **theme.dart**: Theme definitions
  - **models/**: Data models
  - **screens/**: UI screens
  - **services/**: Business logic and services
  - **utils/**: Utility functions
  - **widgets/**: Reusable UI components

## Development Guidelines

### Coding Standards

1. **Architecture**: Follow the Provider pattern for state management
2. **Code Style**: Follow the official Dart style guide
3. **File Naming**: Use snake_case for file names
4. **Class Naming**: Use PascalCase for class names
5. **Variable/Method Naming**: Use camelCase for variables and methods

### UI/UX Guidelines

1. **Minimalism**: Keep the UI clean and distraction-free
2. **Performance**: Ensure smooth animations and transitions
3. **Accessibility**: Support different screen sizes and accessibility features
4. **Consistency**: Maintain consistent design language across the app

### Feature Implementation

When implementing new features:

1. **State Management**: Use Provider for state management
2. **Platform Services**: Use the InstalledApps package for app-related operations
3. **Settings Storage**: Use shared_preferences for storing user settings
4. **Image Handling**: Use image_picker for wallpaper selection

## Testing

1. **Unit Tests**: Write tests for all business logic
2. **Widget Tests**: Test individual widgets
3. **Integration Tests**: Test complete user flows

## Common Tasks

### Adding a New Setting

1. Add the setting key to a constants file
2. Create the UI component in the settings screen
3. Implement the storage logic using shared_preferences
4. Update the relevant parts of the app that use the setting

### Adding a New Theme

1. Define the theme in theme.dart
2. Add an option in the theme selector
3. Implement the theme switching logic in ThemeNotifier

### Working with Apps

1. Use the AppLauncher service for all app-related operations
2. Remember to handle permissions correctly
3. Test on different Android versions

## Dependencies

- **installed_apps**: For app management
- **image_picker**: For selecting images
- **intl**: For internationalization
- **path_provider**: For file system access
- **provider**: For state management
- **shared_preferences**: For storing user preferences

## Build and Deployment

### Debug Build

```bash
flutter run
```

### Release Build

```bash
flutter build apk --release
```

## Common Issues and Solutions

1. **App List Not Loading**: Check Android permissions
2. **Theme Not Applying**: Ensure ThemeNotifier is properly set up
3. **Wallpaper Issues**: Verify file path and permissions

## Contributing

1. Create a feature branch from main
2. Implement your changes
3. Write or update tests
4. Submit a pull request with a clear description of changes

## Roadmap

- Implement custom app categories
- Add widget support
- Improve theme customization options
- Add backup/restore functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IamFishR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
