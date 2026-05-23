---
name: screen-overloads
description: Compose screen architecture that separates stateful wrappers from stateless UI functions. Use when building full-screen composables, previews, UI tests, or ViewModel-backed screens in Android apps.
---

# Screen Composable Overloads

**Description**: To ensure that your screens are highly testable and compatible with Jetpack Compose Previews, always implement a two-tier "overload" architecture for screen Composables where applicable.

## The Pattern

For every screen, create two Composable functions with the same name:

1. **The Stateful Wrapper**: Accepts the `ViewModel` (or acquires it via `hiltViewModel()`) and Navigation dependencies (e.g., `NavBackStack`). It collects state flows and maps navigation events.
2. **The Stateless UI**: Accepts only pure data states (UI state classes, strings, booleans) and lambda callbacks for actions. This function actually draws the UI.

### Why do this?

- **Previews**: You can easily build `@Preview` composables by calling the Stateless UI function with mock data. You don't have to mock a ViewModel or a Navigation BackStack.
- **Testing**: UI tests can verify the Stateless Composable by passing test data and asserting that the callbacks are triggered when buttons are clicked.
- **Separation of Concerns**: It cleanly separates business/navigation logic from the layout code.

## Example Implementation

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.navigation3.runtime.NavBackStack
import androidx.navigation3.runtime.NavKey

// 1. Stateful Wrapper
// Handles ViewModel injection, State collection, and Navigation.
@Composable
fun WorkersScreen(
    sessionUiState: SessionUiState.Ready,
    backStack: NavBackStack<NavKey>
) {
    val workersViewModel = hiltViewModel<WorkersViewModel>()
    val workersUiState by workersViewModel.uiState.collectAsStateWithLifecycle()
    val searchQuery by workersViewModel.searchQuery.collectAsStateWithLifecycle()

    // Delegates directly to the Stateless UI
    WorkersScreen(
        uiState = workersUiState,
        searchQuery = searchQuery,
        onSearchQueryChanged = workersViewModel::updateSearchQuery,
        onWorkerClick = { workerId ->
            backStack.navigate(AppRoutes.WorkerDetail(workerId))
        },
        onBack = { backStack.popOrStay() }
    )
}

// 2. Stateless UI
// Pure UI representation. Contains NO ViewModels, NO Navigation stacks.
@Composable
private fun WorkersScreen(
    uiState: WorkersUiState,
    searchQuery: String,
    onSearchQueryChanged: (String) -> Unit,
    onWorkerClick: (String) -> Unit,
    onBack: () -> Unit
) {
    Scaffold(
        topBar = { /* ... */ }
    ) { padding ->
        // Layout...
    }
}
```

### Best Practices

- Consider making the Stateless UI function `private` if it is only ever used within the same file (or leave it public if you are writing isolated UI component tests).
- Pass individual pieces of state or a consolidated `UiState` data class depending on what the UI actually needs.
- Pass explicitly named lambda callbacks (e.g., `onWorkerClick: (String) -> Unit`) rather than a generic action dispatcher to keep the UI's contract clear and decoupled from the ViewModel's inner workings.
