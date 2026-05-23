---
name: search-feature
description: Build responsive search UIs in Jetpack Compose with local TextFieldValue state and debounced ViewModel queries. Use when implementing search bars, filtering lists, or fixing cursor-jump behavior.
---

# Search Feature with TextFieldValue

**Description**: When implementing search functionalities with `OutlinedTextField` or similar components in Jetpack Compose, use a local `TextFieldValue` state to manage the text and cursor position smoothly, while synchronizing only the text string to the ViewModel. This prevents cursor jumping issues commonly caused by asynchronous state updates or debouncing.

## ViewModel Implementation

In your ViewModel, expose a string state for the query and a method to update it. Use `debounce` when combining state flows to prevent excessive filtering operations.

```kotlin
class MyViewModel : ViewModel() {
    private val _searchQuery = MutableStateFlow("")
    val searchQuery = _searchQuery.asStateFlow()

    @OptIn(FlowPreview::class)
    val uiState = combine(
        sourceFlow,
        searchQuery.debounce(250)
    ) { items, query ->
        // Filter your items here
        filterItems(items, query)
    }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), InitialState())

    fun updateSearchQuery(query: String) {
        _searchQuery.value = query
    }
}
```

## UI Implementation

In your Compose screen, use a local `TextFieldValue` initialized with the ViewModel's query and handle focus requesting.

```kotlin
@Composable
fun SearchScreen(
    searchQuery: String,
    onSearchQueryChanged: (String) -> Unit,
    onBack: () -> Unit
) {
    var toggleSearch by remember { mutableStateOf(searchQuery.isNotBlank()) }
    val focusRequester = remember { FocusRequester() }

    // Crucial: Use TextFieldValue to preserve cursor position
    var textFieldValue by remember {
        mutableStateOf(
            TextFieldValue(
                text = searchQuery,
                selection = TextRange(searchQuery.length)
            )
        )
    }

    // Handle back press to close search
    BackHandler(toggleSearch) {
        toggleSearch = false
    }

    // Auto-focus when search is opened
    LaunchedEffect(toggleSearch) {
        if (toggleSearch) {
            focusRequester.requestFocus()
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    if (toggleSearch) {
                        OutlinedTextField(
                            value = textFieldValue,
                            onValueChange = {
                                textFieldValue = it // Update local state with cursor
                                onSearchQueryChanged(it.text) // Send text to ViewModel
                            },
                            modifier = Modifier
                                .fillMaxWidth()
                                .focusRequester(focusRequester),
                            singleLine = true,
                            placeholder = { Text("Search...") },
                            shape = RoundedCornerShape(20.dp),
                            keyboardOptions = KeyboardOptions(capitalization = KeyboardCapitalization.Words),
                            colors = TextFieldDefaults.colors(
                                focusedContainerColor = Color.Transparent,
                                unfocusedContainerColor = Color.Transparent,
                                disabledContainerColor = Color.Transparent,
                                focusedIndicatorColor = Color.Transparent,
                                unfocusedIndicatorColor = Color.Transparent,
                            )
                        )
                    } else {
                        Text("My Title")
                    }
                },
                navigationIcon = {
                    IconButton(onClick = {
                        if (toggleSearch) {
                            toggleSearch = false
                        } else {
                            onBack()
                        }
                    }) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Back")
                    }
                },
                actions = {
                    if (toggleSearch) {
                        IconButton(onClick = {
                            onSearchQueryChanged("")
                            textFieldValue = textFieldValue.copy(text = "")
                        }) {
                            Icon(Icons.Default.Clear, contentDescription = "Clear")
                        }
                    } else {
                        IconButton(onClick = { toggleSearch = !toggleSearch }) {
                            Icon(Icons.Default.Search, contentDescription = "Search")
                        }
                    }
                }
            )
        }
    ) { padding ->
        // Content
    }
}
```

## Key Takeaways

1. **Local State for TextField**: Always hold a local `TextFieldValue` state in the Composable to manage the text and `TextRange` (cursor position).
2. **One-way Sync**: Update the local `TextFieldValue` on `onValueChange` and emit just the string (`it.text`) to the ViewModel.
3. **Debounce filtering**: Filter operations should happen in the ViewModel by combining the debounced query flow with your data flow.
4. **Focus Requester & BackHandler**: Provide a seamless UX by requesting focus when search opens, and use `BackHandler` to dismiss search mode on back press.
