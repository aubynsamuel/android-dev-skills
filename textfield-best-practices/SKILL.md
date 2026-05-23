---
name: textfield-best-practices
description: Best practices for Jetpack Compose text fields in Android apps. Use when building forms or search inputs and you need guidance on IME padding, keyboard options, focus movement, and accessible input behavior.
---

# TextField Best Practices

**Description**: When building forms or search inputs using TextFields in Jetpack Compose, follow these practical rules to ensure a seamless and accessible user experience.

## 1. Window Insets & IME Padding

**Always** ensure the parent or top-level scrolling container has `Modifier.imePadding()` applied. This ensures that when the on-screen keyboard appears, the UI automatically scrolls or adjusts so that the focused TextField is not obscured.

```kotlin
Scaffold(
    modifier = Modifier.imePadding(),
    // ...
) { padding ->
    LazyColumn(modifier = Modifier.padding(padding)) {
        // ...
    }
}
```

## 2. Keyboard Options & Capitalization

**Always** apply appropriate `KeyboardOptions` to your TextFields. Specify the `capitalization` rules based on the expected input (e.g., `KeyboardCapitalization.Words` for names, `KeyboardCapitalization.Sentences` for descriptions).

## 3. IME Actions & Focus Navigation (Forms)

For forms with multiple inputs, configure the `imeAction` to guide the user:

- Use `ImeAction.Next` for all fields except the last one.
- Use `ImeAction.Done` for the final field to complete the form.
- Use `keyboardActions` to handle the focus movement explicitly.

```kotlin
@Composable
fun UserForm() {
    val focusManager = LocalFocusManager.current
    var firstName by remember { mutableStateOf("") }
    var lastName by remember { mutableStateOf("") }

    Column {
        OutlinedTextField(
            value = firstName,
            onValueChange = { firstName = it },
            label = { Text("First Name") },
            keyboardOptions = KeyboardOptions(
                capitalization = KeyboardCapitalization.Words,
                imeAction = ImeAction.Next
            ),
            keyboardActions = KeyboardActions(
                onNext = { focusManager.moveFocus(FocusDirection.Down) }
            )
        )

        OutlinedTextField(
            value = lastName,
            onValueChange = { lastName = it },
            label = { Text("Last Name") },
            keyboardOptions = KeyboardOptions(
                capitalization = KeyboardCapitalization.Words,
                imeAction = ImeAction.Done
            ),
            keyboardActions = KeyboardActions(
                onDone = {
                    focusManager.clearFocus()
                    // Submit form
                }
            )
        )
    }
}
```

## 4. Explicit Search Inputs

When a TextField is used for searching, use `ImeAction.Search` to show the correct icon on the keyboard (a magnifying glass). Use `keyboardActions` with `onSearch` to trigger the search event and clear focus to hide the keyboard.

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    onSearchTriggered: () -> Unit
) {
    val focusManager = LocalFocusManager.current

    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text("Search...") },
        singleLine = true,
        keyboardOptions = KeyboardOptions(
            capitalization = KeyboardCapitalization.Sentences,
            imeAction = ImeAction.Search
        ),
        keyboardActions = KeyboardActions(
            onSearch = {
                focusManager.clearFocus() // Hide keyboard
                onSearchTriggered() // Execute the explicit search action
            }
        )
    )
}
```
