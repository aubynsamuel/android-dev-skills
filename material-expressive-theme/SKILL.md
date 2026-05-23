---
name: material-expressive-theme
description: Set up Material Expressive theming for Android apps using Jetpack Compose. Use when configuring app themes, dynamic color, motion scheme, or system bar appearance with MaterialExpressiveTheme.
---

# App Theme Setup with Material Expressive

**Description**: When setting up the app's core theme, we use the experimental `MaterialExpressiveTheme` rather than the standard `MaterialTheme`. This provides modern, expressive animations and updated styling defaults.

## Key Implementation Details

### 1. Window Insets & System Bars

Always handle the system UI (status bar and navigation bar) appearance within the theme to ensure icons and text contrast correctly with the background in both light and dark modes. Use `WindowCompat.getInsetsController` inside a `SideEffect`.

```kotlin
val view = LocalView.current
if (!view.isInEditMode) {
    SideEffect {
        val window = (view.context as android.app.Activity).window
        WindowCompat.getInsetsController(window, view).apply {
            isAppearanceLightStatusBars = !darkTheme
            isAppearanceLightNavigationBars = !darkTheme
        }
    }
}
```

### 2. Material Expressive Theme & Motion Scheme

Wrap your application content using `MaterialExpressiveTheme`. To fully utilize the expressive design system, pass `MotionScheme.expressive()` to the `motionScheme` parameter.

```kotlin
import androidx.compose.material3.ExperimentalMaterial3ExpressiveApi
import androidx.compose.material3.MaterialExpressiveTheme
import androidx.compose.material3.MotionScheme

@OptIn(ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        // ... Determine color scheme
    }

    // 1. System Bar Appearance Setup
    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as android.app.Activity).window
            WindowCompat.getInsetsController(window, view).apply {
                isAppearanceLightStatusBars = !darkTheme
                isAppearanceLightNavigationBars = !darkTheme
            }
        }
    }

    // 2. Expressive Theme Setup
    MaterialExpressiveTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = AppShapes,
        motionScheme = MotionScheme.expressive(), // Add expressive motion!
        content = content
    )
}
```

## Why Expressive?

- **MotionScheme**: Using `MotionScheme.expressive()` enables non-linear, spatial animations that make transitions feel more natural and fluid compared to the standard Material 3 motion.
- **System Bars**: Managing `isAppearanceLightStatusBars` and `isAppearanceLightNavigationBars` directly in the theme ensures that edge-to-edge content always has legible system overlays.
