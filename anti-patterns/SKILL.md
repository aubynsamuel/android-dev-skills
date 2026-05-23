---
name: anti-patterns
description: Common Jetpack Compose and Android architecture anti-patterns to avoid. Use when reviewing Android code, refactoring risky patterns, or checking Compose, ViewModel, Flow, coroutine, and state-management decisions for structural issues.
---

# Anti-Patterns to Avoid

**Description**: A collection of common Jetpack Compose and Android architecture anti-patterns that must be strictly avoided to maintain a clean, performant, and bug-free codebase.

## ❌ Avoid: Collecting Flows inside Composables directly

Using `.collectAsState()` without lifecycle awareness can cause flows to remain active even when the app is in the background, leading to resource leaks and crashes.

- **✅ Do instead**: Use `collectAsStateWithLifecycle()` from `androidx.lifecycle.compose`.

## ❌ Avoid: Mutable UI state scattered across ViewModel

Having multiple `MutableStateFlow` or `MutableState` properties exposed individually (e.g., `isLoading`, `errorMessage`, `dataList`) makes the UI state fragmented and prone to race conditions.

- **✅ Do instead**: Combine your data streams into a single, cohesive `UiState` data class and expose a single `StateFlow<UiState>`.

## ❌ Avoid: Launching coroutines in Composables unnecessarily

Using `rememberCoroutineScope { scope.launch { ... } }` for initial data fetching or side effects that should happen automatically on composition can lead to multiple unintended executions on recomposition.

- **✅ Do instead**: Use `LaunchedEffect` for side effects tied to the Composable lifecycle, or better yet, trigger initial data loading from the ViewModel.

## ❌ Avoid: Passing repositories or use cases into UI components

Passing heavy dependencies or DAOs directly into Composables breaks the architecture, makes previews impossible without heavy mocking, and tightly couples the UI to data sources.

- **✅ Do instead**: Pass data down as pure state (UI models) and handle actions via lambda callbacks (`onSave: () -> Unit`). Inject repositories only into ViewModels.

## ❌ Avoid: Using `GlobalScope`

Launching coroutines in `GlobalScope` prevents them from being cancelled when the component is destroyed, causing memory leaks.

- **✅ Do instead**: Always use `viewModelScope` inside ViewModels, or `lifecycleScope` / `LaunchedEffect` in UI components.

## ❌ Avoid: Nested `collect` calls

Calling `.collect { }` inside another `.collect { }` block leads to callback hell, race conditions, and unmanageable state synchronizations.

- **✅ Do instead**: Use declarative flow operators like `combine()`, `flatMapLatest()`, or `zip()` to merge and transform streams before collecting them once at the end.
