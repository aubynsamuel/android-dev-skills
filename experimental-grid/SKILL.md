---
name: experimental-grid
description: Use Jetpack Compose experimental Grid APIs for structured layouts. Use when building dashboards, forms, or complex grid arrangements that would otherwise require deeply nested Rows and Columns.
---

# Experimental Grid Layout

**Description**: When laying out elements in complex or structured arrangements in Jetpack Compose, use the new experimental `Grid` layout instead of nesting `Row`s and `Column`s. This API provides flexible CSS Grid-like capabilities directly in Compose, allowing you to define versatile structures beyond just strict 2D layouts.

## Required Imports and Opt-Ins

The API requires opting into `ExperimentalGridApi` (and potentially `ExperimentalFlexBoxApi` depending on the exact version and usage).

```kotlin
import androidx.compose.foundation.layout.ExperimentalFlexBoxApi
import androidx.compose.foundation.layout.ExperimentalGridApi
import androidx.compose.foundation.layout.Grid
import androidx.compose.foundation.layout.GridTrackSize
```

## Grid Configuration

The `Grid` composable takes a `config` lambda where you define the tracks (columns and rows) and the gaps between them.

- **`column(weight/size)`**: Defines a column. You can pass a fraction (like `0.5f`) for equal weights, or use `GridTrackSize.Auto` / `GridTrackSize.Fixed(dp)`.
- **`row(size)`**: Defines a row.
- **`gap(dp)`**: Sets the spacing between grid items.

### Example Implementation

```kotlin
@OptIn(ExperimentalFlexBoxApi::class, ExperimentalGridApi::class)
@Composable
fun DashboardGrid() {
    Grid(
        config = {
            // Define 2 equal-width columns (0.5f each)
            repeat(2) {
                column(0.5f)
            }

            // Define 3 rows that size automatically based on content
            repeat(3) {
                row(GridTrackSize.Auto)
            }

            // Add spacing between grid items
            gap(8.dp)
        }
    ) {
        // Place exactly 6 items (2 columns * 3 rows) inside the block.
        // They will automatically flow into the defined grid slots.

        StatCard(label = "Units", value = "10")
        StatCard(label = "Items", value = "42")
        StatCard(label = "Activities", value = "5")
        StatCard(label = "Contractors", value = "3")
        StatCard(label = "Templates", value = "12")
        StatCard(label = "Workers", value = "100")
    }
}
```

## Best Practices

1. **Match Children to Tracks**: Ensure the number of composables you emit inside the `Grid` lambda aligns with the total cells defined by your track configuration.
2. **Opt-in Annotation**: Remember to add `@OptIn(ExperimentalFlexBoxApi::class, ExperimentalGridApi::class)` to the composable using this layout.
3. **Use for Static Grids**: This layout is ideal for dashboards, stat cards, and forms where the grid structure is fixed. For large, scrollable grids of dynamic items, stick to `LazyVerticalGrid`.
