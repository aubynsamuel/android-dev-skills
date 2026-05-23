---
name: hilt-workmanager
description: Integrate WorkManager with Dagger Hilt in Android apps. Use when creating injected workers, configuring WorkerFactory, updating manifest startup settings, or wiring background jobs in a Hilt-based project.
---

# Hilt & WorkManager Setup

**Description**: When using WorkManager in an application that relies on Hilt for dependency injection, you must configure Hilt to properly inject dependencies into your Workers. This requires specific dependencies, disabling the default WorkManager initializer, and configuring the `Application` class.

## 1. Dependencies Setup (`gradle/libs.versions.toml`)

First, ensure you have the appropriate versions and libraries declared in your version catalog.

```toml
[versions]
work = "2.10.0"
hiltWork = "1.2.0"

[libraries]
androidx-work-runtime-ktx = { module = "androidx.work:work-runtime-ktx", version.ref = "work" }
hilt-work = { module = "androidx.hilt:hilt-work", version.ref = "hiltWork" }
hilt-work-compiler = { module = "androidx.hilt:hilt-compiler", version.ref = "hiltWork" }
```

## 2. App Module Dependencies (`app/build.gradle.kts`)

Add the WorkManager and Hilt Work dependencies to your app's `build.gradle.kts`. Note the use of `ksp` for the compiler.

```kotlin
dependencies {
    // WorkManager
    implementation(libs.androidx.work.runtime.ktx)

    // Hilt WorkManager integration
    implementation(libs.hilt.work)
    ksp(libs.hilt.work.compiler)
}
```

## 3. Disable Default Initializer (`AndroidManifest.xml`)

By default, WorkManager auto-initializes using AndroidX Startup. Because we need to pass a custom `WorkerFactory` created by Hilt, we must disable this default behavior by removing the initializer node in the Manifest.

```xml
<application ...>
    <provider
        android:name="androidx.startup.InitializationProvider"
        android:authorities="${applicationId}.androidx-startup"
        android:exported="false"
        tools:node="merge">
        <!-- Remove the default WorkManager initializer -->
        <meta-data
            android:name="androidx.work.WorkManagerInitializer"
            android:value="androidx.startup"
            tools:node="remove" />
    </provider>
</application>
```

## 4. Application Configuration (`ConstructlyApp.kt`)

Finally, your `Application` class must implement `Configuration.Provider` to tell WorkManager to use the `HiltWorkerFactory` for instantiating workers.

```kotlin
import android.app.Application
import androidx.hilt.work.HiltWorkerFactory
import androidx.work.Configuration
import dagger.hilt.android.HiltAndroidApp
import javax.inject.Inject

@HiltAndroidApp
class ConstructlyApp : Application(), Configuration.Provider {

    @Inject
    lateinit var workerFactory: HiltWorkerFactory

    override val workManagerConfiguration: Configuration
        get() = Configuration.Builder()
            .setWorkerFactory(workerFactory)
            .build()
}
```

## Worker Implementation

Once set up, you can successfully inject dependencies into your workers using `@HiltWorker` and `@AssistedInject`.

```kotlin
@HiltWorker
class SyncWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted workerParams: WorkerParameters,
    private val repository: MyRepository // Successfully injected!
) : CoroutineWorker(appContext, workerParams) {
    // ...
}
```
