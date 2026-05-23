---
name: window-background-night-theme
description: Add a dark-mode XML theme override for Android apps by mirroring the active app theme into res/values-night/themes.xml and adding android:windowBackground with @color/black. Use when an app shows a white launch or window background in dark mode because the night theme does not override windowBackground.
---

# Window Background Night Theme

When an Android app uses XML themes and does not provide a `values-night/themes.xml` override, the app can keep a light window background in dark mode. Fix this by mirroring the active theme into a night resource file and adding a black window background override.

## Goal

Ensure dark mode uses a dark window background without changing the rest of the app theme.

## Workflow

1. Find the app theme actually used by the application or launcher activity.
2. Locate the source theme definition, usually in `res/values/themes.xml`.
3. Create `res/values-night/themes.xml` if it does not exist.
4. Copy the same theme definition into the night file without removing or altering existing items.
5. Add this item inside the copied theme:

```xml
<item name="android:windowBackground">@color/black</item>
```

## Rules

- Preserve the existing theme parent exactly.
- Preserve all existing `<item>` entries exactly as they are.
- Add only the `android:windowBackground` override unless the user explicitly asks for more theme changes.
- Use the same theme name the app already uses.
- Do not assume a specific app name. Example names should be neutral, such as `Theme.SampleApp`.
- Apply this to Android apps generally, not only Jetpack Compose apps.

## Example

If the app uses a theme like this in `res/values/themes.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.SampleApp" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">@color/blue</item>
    </style>
</resources>
```

Create or update `res/values-night/themes.xml` like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.SampleApp" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">@color/blue</item>
        <item name="android:windowBackground">@color/black</item>
    </style>
</resources>
```

## Verification

- Confirm the night theme file uses the same theme name as the active app theme.
- Confirm `android:windowBackground` is present in the night theme.
- Confirm the app now shows a dark background in dark mode instead of a white background.
