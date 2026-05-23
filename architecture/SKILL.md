---
name: architecture
description: Clean Architecture guidance for Android apps using Jetpack Compose. Use when organizing layers, placing classes, defining boundaries between data, domain, and presentation, or reviewing maintainability and separation of concerns.
---

# Android Clean Architecture

**Description**: This guide provides instructions for implementing clean architecture in an Android app using Jetpack Compose. It enforces separation of concerns to ensure maintainability and testability.

## Layer Structure

- **Data Layer** (`com.yourapp.data`):

  - Database entities, DAOs (Room)
  - Repository implementations
  - Data sources (local/remote)
  - Services and Broadcast Receivers
  - Shared Preferences/DataStore

- **Domain Layer** (`com.yourapp.domain`) - _Optional_:

  - Pure Kotlin models (no Android dependencies)
  - Repository interfaces
  - Use cases/interactors (for complex business logic)
  - _Only create this layer if business logic justifies it_

- **Presentation Layer** (`com.yourapp.presentation`):
  - `theme/`: Material 3 design tokens (colors, typography, shapes)
  - `screens/`: Full-screen composables
  - `viewmodel/`: Hilt-injected ViewModels
  - `components/`: Reusable UI components
  - `navigation/`: Navigation configuration
  - `utils/`: Extension functions and helpers

## Key Principles

- **Domain models are pure Kotlin** - no Android framework dependencies
- **Presentation depends on Domain** - never the reverse
- **Data implements Domain interfaces** - dependency inversion
