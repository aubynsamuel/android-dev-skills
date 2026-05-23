---
name: app-language-switching
description: Implement in-app language switching for Android apps with Jetpack Compose and AppCompat locale APIs. Use when adding per-app language selection, locale config, runtime locale updates, or multilingual settings screens.
---

# Android In-App Language Switching (Jetpack Compose)

**Description**: This guide provides instructions for implementing in-app language switching in an Android app using Jetpack Compose, ensuring compatibility across different Android versions without requiring a hard restart of the activity.

## 0. Prerequisites

Ensure these are present in app/build.gradle.kts (or the Groovy equivalent):

```kotlin
dependencies {
    implementation("androidx.appcompat:appcompat:1.7.0")   // AppCompatActivity + AppCompatDelegate
}
```

## 1. Manifest — Locale Config (Required for Android 13+)

Android 13 (API 33) requires an explicit list of supported locales in the manifest. Without
this, the system-level per-app language picker will not display the app's languages.

**`res/xml/locales_config.xml`** — create this file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<locale-config xmlns:android="http://schemas.android.com/apk/res/android">
    <locale android:name="en"/>
    <locale android:name="hi"/>
    <!-- Add every locale your app ships with -->
</locale-config>
```

**`AndroidManifest.xml`** — reference it on the `<application>` tag:

```xml
<application
    ...
    android:localeConfig="@xml/locales_config">
```

> Skip this step only if targeting API < 33 exclusively.

## 2. Prerequisites & Activity Setup

To support per-app language preferences via `AppCompatDelegate` on Android versions below API 33 (Android 13), the main activity hosting the Compose UI **must** inherit from `AppCompatActivity` rather than `ComponentActivity`.

**Update MainActivity inheritance:**

```kotlin
import androidx.appcompat.app.AppCompatActivity

@AndroidEntryPoint // (If using Hilt)
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            // Your App Theme and Navigation
        }
    }
}
```

> [!IMPORTANT]  
> If an app previously used `ComponentActivity`, changing to `AppCompatActivity` also requires ensuring that the app's base theme in `themes.xml` (if any are still defined for the splash/launch screen) inherits from a `Theme.AppCompat` derivative (like `Theme.AppCompat.DayNight.NoActionBar`).

## 3. Implementing the Language Switch Logic

The actual switching logic needs to handle API 33+ (using `LocaleManager`) and API < 33 (using `AppCompatDelegate`) differences seamlessly.

**Create a `setAppLanguage` utility function:**
This function changes the application locale dynamically across all supported API levels.

```kotlin
import android.app.LocaleManager
import android.content.Context
import android.os.Build
import android.os.LocaleList
import androidx.appcompat.app.AppCompatDelegate
import androidx.core.os.LocaleListCompat

fun setAppLanguage(context: Context, languageCode: String) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        // Android 13+ (API 33+)
        context.getSystemService(LocaleManager::class.java).applicationLocales =
            LocaleList.forLanguageTags(languageCode)
    } else {
        // Below Android 13
        val localeList = LocaleListCompat.forLanguageTags(languageCode)
        AppCompatDelegate.setApplicationLocales(localeList)
    }
}
```

## 4. Retrieving Current Language in Compose

To reflect the currently selected language in the UI (e.g., checking the correct radio button in settings), retrieve the current locale configuration dynamically within your Composable function. This value will automatically trigger a recomposition when the locale changes.

```kotlin
import androidx.compose.ui.platform.LocalConfiguration

@Composable
fun SettingsScreen() {
    val configuration = LocalConfiguration.current

    // Grabs the active language code, e.g., "en", "hi", "es"
    val currentLanguageCode = configuration.locales[0].language

    // ... UI to display language options ...
}
```

## 5. Building the UI Component

When building the UI component to switch languages, trigger the `setAppLanguage` function whenever a user taps an option.

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.selection.selectable
import androidx.compose.foundation.selection.selectableGroup
import androidx.compose.material3.RadioButton
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.semantics.Role
import androidx.compose.ui.unit.dp

data class LanguageOption(val displayName: String, val tag: String)

private val supportedLanguages = listOf(
    LanguageOption("English", "en"),
    LanguageOption("Hindi", "hi"),
    // Add more as needed
)

@Composable
fun LanguageSelector(currentLanguageCode: String) {
    val context = LocalContext.current

    Column(modifier = Modifier.selectableGroup()) {
        supportedLanguages.forEach { option ->
            LanguageRow(
                displayName = option.displayName,
                isSelected = currentLanguageCode == option.tag,
                onSelect = { setAppLanguage(context, option.tag) }
            )
        }
    }
}

@Composable
private fun LanguageRow(
    displayName: String,
    isSelected: Boolean,
    onSelect: () -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .selectable(
                selected = isSelected,
                onClick = onSelect,
                role = Role.RadioButton
            )
            .padding(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        RadioButton(selected = isSelected, onClick = null)
        Text(
            text = displayName,
            modifier = Modifier.padding(start = 16.dp)
        )
    }
}
```

## 6. Summary of Agent Flow

When explicitly asked to implement a language switch:

1. **Gradle** — confirm `appcompat` dependency is present.
2. **Manifest** — add `res/xml/locales_config.xml` listing all supported locales and reference it via `android:localeConfig` in `<application>`. (Required for Android 13+)
3. **Activity** — ensure `MainActivity` extends `AppCompatActivity`; update theme if needed.
4. Extract the current language via `LocalConfiguration.current.locales[0].language` inside the Compose hierarchy.
5. Supply a unified `setAppLanguage(context, languageCode)` wrapper that respects API versioning differences (`LocaleManager` vs `AppCompatDelegate`).
6. Apply standard structural interactions via RadioButtons or DropdownMenus invoking the wrapper.
