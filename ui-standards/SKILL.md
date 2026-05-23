---
name: ui-standards
description: Visual and structural UI standards for Android Compose screens. Use when reviewing or building app UI for consistency, accessibility, icon usage, theme compliance, and avoidance of hardcoded visual shortcuts.
---

# UI Standards & Best Practices

**Description**: Strict visual and structural rules for maintaining a professional, accessible, and theme-compliant UI.

## The "No Emojis" Rule (CRITICAL)

**Never use emojis in UI text or as visual indicators.**

Use the `Icon` composable with Material Icons instead.

❌ **Bad - Using Emojis**

```kotlin
Text("🔥 Trending")
Text("📊 Dashboard")
Row {
    Text("👤")
    Text(userName)
}
```

✅ **Good - Using Icons**

```kotlin
Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    Icon(Icons.Default.Whatshot, contentDescription = null)
    Text("Trending")
}

Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    Icon(Icons.Default.Dashboard, contentDescription = null)
    Text("Dashboard")
}

Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    Icon(Icons.Default.Person, contentDescription = null)
    Text(userName)
}
```

## Theming & Colors

Never hardcode colors. Always use theme colors:

❌ **Bad**

```kotlin
Box(modifier = Modifier.background(Color(0xFF6200EE)))
Text("Hello", color = Color.Red)
```

✅ **Good**

```kotlin
Box(modifier = Modifier.background(MaterialTheme.colorScheme.primary))
Text("Hello", color = MaterialTheme.colorScheme.error)
```
