---
name: dependency-injection
description: Configure dependency injection in Android apps with Dagger Hilt. Use when wiring Application, Activity, ViewModel, Repository, module, or scoped dependency setup in a Jetpack Compose codebase.
---

# Dependency Injection with Hilt

**Description**: This guide provides instructions for implementing dependency injection in an Android app using Dagger Hilt.

## Application Setup

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

## Activity Setup

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                Navigation()
            }
        }
    }
}
```

## ViewModel with Hilt

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    // ViewModel implementation
}
```

## Repository Injection

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
    @Provides
    @Singleton
    fun provideUserRepository(
        userDao: UserDao
    ): UserRepository = UserRepositoryImpl(userDao)
}
```

## ViewModel Usage in Composables

Always use `hiltViewModel()` from the correct import:

```kotlin
import androidx.hilt.navigation.compose.hiltViewModel

@Composable
fun MyScreen() {
    val viewModel: MyViewModel = hiltViewModel()
    // Use viewModel
}
```
