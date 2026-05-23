---
name: flow-operators
description: Kotlin Flow operator patterns for Android ViewModels. Use when deriving reactive UI state with combine, flatMapLatest, filterNotNull, stateIn, debounce, or distinctUntilChanged instead of manual state synchronization.
---

# Reactive Flow Operators in ViewModels

**Description**: To build efficient and highly reactive architectures in ViewModels, heavily utilize Kotlin Flow operators to derive state. This prevents manually synchronizing variables and ensures the UI always represents the latest data source.

Here are the key operators and when to use them.

## 1. `filterNotNull` & `flatMapLatest`

When you need to observe database changes based on a reactive ID (like a dynamically changing `siteId`), use `filterNotNull` to ignore null states, followed by `flatMapLatest` to swap to the new Flow stream.

**Why `flatMapLatest` instead of `flatMapMerge`?**
When the `siteId` changes, `flatMapLatest` cancels the previous observation so you are only ever listening to the active site's data.

```kotlin
private val _currentSiteId = MutableStateFlow<String?>(null)

val sessions = _currentSiteId
    .filterNotNull() // Wait until siteId is not null
    .flatMapLatest { siteId ->
        // Cancel old observation, start observing new site's data
        repository.observeSessionsForDate(siteId, today())
    }
    .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())
```

## 2. `combine` & `distinctUntilChanged`

When your screen's UI State depends on multiple data streams (like user profile AND dashboard stats), use `combine` to merge them.

Use `distinctUntilChanged()` on the upstream flows to prevent `combine` from re-evaluating if the emitted object hasn't actually changed.

```kotlin
combine(
    repository.observeDashboard().distinctUntilChanged(),
    repository.observeChecklistOverview().distinctUntilChanged()
) { dashboard, overview ->
    // This lambda fires whenever EITHER flow emits a new distinct value.
    DashboardUiState(dashboard, overview)
}.stateIn(...)
```

## 3. `debounce` (For Search)

When observing user input (like a search query), use `debounce` before combining it with your data flow. This delays the emission until the user pauses typing, preventing rapid, expensive filtering operations.

```kotlin
private val _searchQuery = MutableStateFlow("")

val uiState = combine(
    dataFlow,
    _searchQuery.debounce(250) // Wait 250ms after user stops typing
) { data, query ->
    filterData(data, query)
}.stateIn(...)
```

## 4. `stateIn`

The `stateIn` operator converts a cold flow into a hot `StateFlow` that can be safely collected by the UI.

**Always use `SharingStarted.WhileSubscribed(5000)`** for UI state.
This keeps the flow active for 5 seconds after the UI goes to the background (or during configuration changes like rotations), avoiding unnecessary database re-queries.

```kotlin
val uiState: StateFlow<MyUiState> = combine( ... )
    .stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = MyUiState()
    )
```

## Summary Checklist

- Use **`flatMapLatest`** when the source of truth for an observation parameter changes.
- Use **`combine`** to merge multiple independent flows into a single `UiState`.
- Use **`distinctUntilChanged`** to stop unnecessary downstream processing.
- Use **`debounce`** on rapid user inputs like search fields.
- Use **`stateIn(..., WhileSubscribed(5000), ...)`** to expose the final flow to Compose securely.
