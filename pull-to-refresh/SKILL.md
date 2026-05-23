---
name: pull-to-refresh
description: Implement pull-to-refresh in Jetpack Compose with Material 3 APIs. Use when adding refresh gestures, refresh indicators, or syncing UI state with manual reload actions on scrollable screens.
---

# Expressive Pull to Refresh

**Description**: Implementing a modern, expressive pull-to-refresh mechanism in Jetpack Compose utilizes the experimental Material 3 `PullToRefreshBox` API. Follow this pattern to ensure smooth and standard pull-to-refresh behavior.

## Required Imports

Since these APIs are experimental, you must use the exact imports and opt-in to the `ExperimentalMaterial3Api`.

```kotlin
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.pulltorefresh.PullToRefreshBox
import androidx.compose.material3.pulltorefresh.PullToRefreshDefaults.LoadingIndicator
import androidx.compose.material3.pulltorefresh.rememberPullToRefreshState
```

## Implementation

1. **Opt-In to Experimental API**: Annotate your Composable with `@OptIn(ExperimentalMaterial3Api::class)`.
2. **State Management**: Use `rememberPullToRefreshState()` to maintain the internal mechanics of the refresh action.
3. **Trigger State**: Map your ViewModel's refresh or sync status to a boolean `isRefreshing`.
4. **Wrap your Scrollable Content**: Place `PullToRefreshBox` as the direct child of your `Scaffold`'s content lambda, and provide the `LoadingIndicator` with its required alignment.

### Example Code

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyScreen(
    uiState: MyUiState,
    onRefresh: () -> Unit
) {
    // 1. Remember the pull-to-refresh state
    val pullToRefreshState = rememberPullToRefreshState()

    // 2. Determine if currently refreshing from your state
    val isRefreshing = uiState.isSyncing

    Scaffold(
        topBar = { /* ... */ }
    ) { innerPadding ->

        // 3. Wrap your content in PullToRefreshBox
        PullToRefreshBox(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding),
            isRefreshing = isRefreshing,
            onRefresh = onRefresh,
            state = pullToRefreshState,
            indicator = {
                LoadingIndicator(
                    modifier = Modifier.align(Alignment.TopCenter),
                    state = pullToRefreshState,
                    isRefreshing = isRefreshing
                )
            }
        ) {
            // Your scrollable content goes here!
            LazyColumn(
                modifier = Modifier.fillMaxSize(),
            ) {
                // items...
            }
        }
    }
}
```

## Key Considerations

- Ensure the `PullToRefreshBox` occupies the maximum available size (`fillMaxSize()`) so pull gestures are caught anywhere on the screen.
- The `LoadingIndicator` must be aligned to `Alignment.TopCenter` within the indicator lambda.
- Your content inside the box MUST be scrollable (like a `LazyColumn` or `Column(Modifier.verticalScroll())`) for the pull gesture to be intercepted correctly when at the top.
