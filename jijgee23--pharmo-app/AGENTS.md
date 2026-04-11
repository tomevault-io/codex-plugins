# Copilot Instructions for pharmo_app

## Overview
This project is a Flutter application designed for managing deliveries and related services. It integrates various functionalities such as battery monitoring, connectivity checks, and Firebase services for notifications and data management.

## Architecture
- **Main Components**: The application is structured around several key services and models:
  - **Services**: `BatteryService`, `LogService`, and `FirebaseApi` handle specific functionalities like battery state monitoring, logging, and Firebase integration.
  - **Models**: The `Delivery` model encapsulates delivery-related data, including delivery man details, zones, and orders.
  - **Providers**: The application uses the Provider package for state management, with various providers like `AuthProvider`, `BasketProvider`, and `HomeProvider` managing different aspects of the app's state.

## Developer Workflows
- **Building the Project**: Use `flutter build` to compile the application for different platforms.
- **Testing**: Run tests using `flutter test` to ensure code quality and functionality.
- **Debugging**: Utilize Flutter's debugging tools in your IDE or run `flutter run --debug` for debugging sessions.

## Project-Specific Conventions
- **Naming Conventions**: Follow camelCase for variable names and PascalCase for class names.
- **File Structure**: Organize files by functionality, with separate directories for models, services, and controllers.

## Integration Points
- **Firebase**: The app integrates with Firebase for messaging and notifications. Ensure Firebase is properly configured in `firebase_options.dart`.
- **Battery and Connectivity**: Use `BatteryService` and `ConnectivityPlus` for monitoring device state and connectivity.

## Communication Patterns
- **State Management**: The app uses the Provider package for state management, allowing components to reactively update based on state changes.
- **Event Handling**: Use event classes like `AppEvent` to manage application-wide events and state transitions.

## Examples
- **Using BatteryService**: To monitor battery state, subscribe to the `batteryLevelStream` in your widget.
- **Logging**: Use `LogService` to log important events and errors throughout the application.

## Conclusion
These instructions should help AI agents understand the structure and workflows of the `pharmo_app` codebase, enabling them to assist effectively in development tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jijgee23)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Jijgee23)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
