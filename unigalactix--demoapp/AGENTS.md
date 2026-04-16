<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Multi Tasks Android App Development Guidelines

## Project Overview
This is an Android application built with Kotlin for managing multiple types of tasks. The app follows MVVM architecture pattern and uses Room database for local storage.

## Architecture Components
- **Model**: Task data class with Room annotations
- **View**: Activities and Fragments with Material Design UI
- **ViewModel**: Manages UI-related data and business logic
- **Repository**: Abstracts data sources (Room database)
- **Database**: Room database with DAO for data operations

## Key Features
- Create, read, update, and delete tasks
- Categorize tasks (Work, Personal, Shopping, etc.)
- Set task priorities (Low, Medium, High, Urgent)
- Due dates and reminders
- Task completion tracking
- Material Design 3 UI components

## Coding Standards
- Use Kotlin for all Android code
- Follow Android Architecture Components best practices
- Implement proper separation of concerns
- Use LiveData for reactive UI updates
- Handle database operations asynchronously with coroutines
- Apply Material Design guidelines for UI/UX

## Dependencies Used
- Room for local database
- LiveData and ViewModel for architecture
- Material Design Components
- RecyclerView for list display
- Navigation Components
- WorkManager for background tasks

## Code Patterns
- Repository pattern for data access
- Observer pattern with LiveData
- ViewHolder pattern for RecyclerView
- Factory pattern for ViewModel creation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unigalactix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
