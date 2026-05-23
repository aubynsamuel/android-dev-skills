---
name: navigation
description: Implement type-safe Android navigation with Navigation 3 and Kotlin serialization. Use when adding screens, routes, deep links, back stack handling, or compile-time-safe navigation flows in Jetpack Compose.
---

# Navigation 3 Implementation

## Navigation Dependencies Setup

```build.gradle.kts
plugins {
    alias(libs.plugins.kotlinx.serialization) // serialization plugin
}

dependencies {
    implementation(libs.navigation3.runtime)
    implementation(libs.navigation3.ui)
    implementation(libs.kotlinx.serialization.json) // requires kotlin serialization lib
}

```

```libs.toml
[versions]
kotlin = "2.3.21"
kotlinxSerialization = "1.11.0"
nav3 = "1.1.1"

[libraries]
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version ref = "kotlinxSerialization" }

# Navigation 3
navigation3-runtime = { group = "androidx.navigation3", name = "navigation3-runtime", version.ref = "nav3" }
navigation3-ui = { group = "androidx.navigation3", name = "navigation3-ui", version.ref = "nav3" }

[plugins]
kotlinx-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }

```

Use **Navigation 3** library for type-safe, compile-time verified routing.

### Route Definitions

Define routes as `@Serializable` classes/objects implementing `NavKey`:

```kotlin
// File: presentation/navigation/AppRoutes.kt
package com.yourapp.presentation.navigation

import androidx.navigation3.runtime.NavKey
import kotlinx.serialization.Serializable

sealed class AppRoutes : NavKey {
    @Serializable
    object Home : NavKey

    @Serializable
    data class UserProfile(val userId: String) : NavKey

    @Serializable
    data class ProductDetail(
        val productId: String,
        val categoryId: String? = null
    ) : NavKey

    @Serializable
    object Settings : NavKey
}
```

## Navigation Extensions

Create extension functions for common navigation patterns:

```kotlin
// File: presentation/navigation/NavigationExtensions.kt
package com.yourapp.presentation.navigation

import androidx.navigation3.runtime.NavBackStack
import androidx.navigation3.runtime.NavKey

/**
 * Navigate to a new screen.
 * If the target screen is already at the top of the stack, replace it instead of adding.
 *
 * @param targetScreen The destination route
 */
fun NavBackStack<NavKey>.navigate(targetScreen: NavKey) {
    if (this.last() != targetScreen) {
        this.add(targetScreen)
    } else {
        this[this.lastIndex] = targetScreen
    }
}

/**
 * Navigate to a new screen and remove the current screen from backstack.
 * Useful for replacing the current screen without adding to navigation history.
 *
 * Example: Login -> Home (remove Login from stack)
 *
 * @param targetScreen The destination route
 */
fun NavBackStack<NavKey>.navigateAndRemoveCurrent(targetScreen: NavKey) {
    this[this.lastIndex] = targetScreen
}

/**
 * Navigate to a screen and clear all other screens from the backstack.
 * Useful for "reset to home" or "logout" scenarios.
 *
 * Example: Any Screen -> Home (clear entire stack)
 *
 * @param targetScreen The destination route (typically your home/main screen)
 */
fun NavBackStack<NavKey>.navigateAndRemoveAllOther(targetScreen: NavKey) {
    this.add(targetScreen)
    this.removeAll { navKey -> navKey != targetScreen }
}

/**
 * Clear backstack up to (and including) the target screen.
 * Useful for "back to X" navigation patterns.
 *
 * Example: Screen A -> B -> C -> D, clearUpTo(B) results in: A -> B
 *
 * @param targetScreen The screen to navigate back to
 */
fun NavBackStack<NavKey>.clearUpTo(targetScreen: NavKey) {
    val targetIndex = this.indexOf(targetScreen)
    if (targetIndex != -1) {
        this.removeAll { navKey -> this.indexOf(navKey) > targetIndex }
    }
}

/**
 * Pop the backstack if there are multiple entries, otherwise stay on current screen.
 * Prevents accidentally exiting the app when on the root screen.
 *
 * Use this in "Back" button handlers.
 */
fun NavBackStack<NavKey>.popOrStay() {
    if (this.size > 1) {
        this.removeLastOrNull()
    }
}
```

## Navigation Setup

Create the main navigation composable:

```kotlin
// File: presentation/navigation/Navigation.kt
package com.yourapp.presentation.navigation

import androidx.compose.foundation.background
import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.retain.retain
import androidx.compose.ui.Modifier
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.navigation3.runtime.NavBackStack
import androidx.navigation3.runtime.NavKey
import androidx.navigation3.runtime.entryProvider
import androidx.navigation3.runtime.rememberNavBackStack
import androidx.navigation3.ui.NavDisplay

/**
 * Main navigation component for the application.
 * Uses Navigation 3 library with type-safe routes and Hilt integration.
 *
 * @param deepLinkData Optional data for handling deep links
 */
@Composable
fun Navigation(deepLinkData: String? = null) {
    // Remember backstack across recompositions
    val backStack = rememberNavBackStack(AppRoutes.Home)

    // Handle deep links
    LaunchedEffect(deepLinkData) {
        if (deepLinkData != null) {
            // Parse deep link and navigate
            backStack.navigate(AppRoutes.UserProfile(deepLinkData))
        }
    }

    NavDisplay(
        onBack = { backStack.removeLastOrNull() },
        backStack = backStack,
        modifier = Modifier.background(MaterialTheme.colorScheme.background),
        entryProvider = entryProvider {
            // Home screen
            entry<AppRoutes.Home> {
                val viewModel: HomeViewModel = hiltViewModel()
                HomeScreen(
                    backStack = backStack,
                    viewModel = viewModel
                )
            }

            // User profile screen with parameter
            entry<AppRoutes.UserProfile> {
                val viewModel: UserViewModel = hiltViewModel()
                UserProfileScreen(
                    userId = it.userId,
                    backStack = backStack,
                    viewModel = viewModel
                )
            }

            // Product detail with multiple parameters
            entry<AppRoutes.ProductDetail> {
                val viewModel: ProductViewModel = hiltViewModel()
                ProductDetailScreen(
                    productId = it.productId,
                    categoryId = it.categoryId,
                    backStack = backStack,
                    viewModel = viewModel
                )
            }

            // Settings screen
            entry<AppRoutes.Settings> {
                val viewModel: SettingsViewModel = hiltViewModel()
                SettingsScreen(
                    backStack = backStack,
                    viewModel = viewModel
                )
            }
        }
    )
}
```

## Navigation Usage in Screens

```kotlin
@Composable
fun HomeScreen(
    backStack: NavBackStack<NavKey>,
    viewModel: HomeViewModel
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Home") },
                actions = {
                    IconButton(onClick = { backStack.navigate(AppRoutes.Settings) }) {
                        Icon(Icons.Default.Settings, contentDescription = "Settings")
                    }
                }
            )
        }
    ) { padding ->
        LazyColumn(modifier = Modifier.padding(padding)) {
            items(viewModel.users) { user ->
                UserCard(
                    user = user,
                    onClick = { backStack.navigate(AppRoutes.UserProfile(user.id)) }
                )
            }
        }
    }
}
```
