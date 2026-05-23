---
name: modularity
description: File-organization rules for Android projects that enforce focused modules and one primary item per file. Use when creating new files, splitting large files, or reviewing codebase navigability and reuse boundaries.
---

# Modularity & File Organization

**Description**: This guide enforces a strict modularity principle: "One Item Per File". This ensures code remains organized, maintainable, and easy to navigate.

## The "One Item Per File" Rule

Each file should contain exactly **one** primary item (Composable, ViewModel, utility function, or data model).

**Exception**: Combine 2-3 items ONLY if they are:

- Extremely small (< 20 lines each)
- Tightly coupled and inseparable
- Not reusable elsewhere

## Examples

✅ **Good - Modular**

```text
presentation/
  components/
    UserCard.kt          // Contains only UserCard composable
    UserAvatar.kt        // Contains only UserAvatar composable
    StatusBadge.kt       // Contains only StatusBadge composable
```

❌ **Bad - Messy**

```text
presentation/
  components/
    UserComponents.kt    // Contains UserCard, UserAvatar, StatusBadge
```

## Import Best Practices

Always use specific imports, never fully-qualified names in code:

✅ **Good**

```kotlin
import androidx.compose.foundation.layout.Spacer
import androidx.compose.ui.Modifier

Spacer(modifier = Modifier.size(4.dp))
```

❌ **Bad**

```kotlin
androidx.compose.foundation.layout.Spacer(modifier = Modifier.size(4.dp))
```
