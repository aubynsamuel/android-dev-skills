---
name: room-setup
description: Set up Room databases for Android apps with Hilt and auto migrations. Use when configuring Room, schema exports, DAO access, database builders, or migration-safe local persistence.
---

# Room Database Setup with Hilt & AutoMigrations

**Description**: When setting up a Room database, standardizing the configuration ensures that schema migrations, dependency injection, and DAO access are clean and predictable. This guide covers how to set up Room with Hilt, schema exports, and automatic migrations.

## 1. Schema Export Configuration (`build.gradle.kts`)

To utilize Room's AutoMigration feature, you MUST enable schema exports. Configure the KSP plugin in your app's `build.gradle.kts` to define where the JSON schema files will be saved.

```kotlin
ksp {
    arg("room.schemaLocation", "$projectDir/schemas")
}
```

_Note: Make sure to commit the generated `schemas/` folder to version control!_

## 2. Database Class & AutoMigrations

Your core database class should extend `RoomDatabase`. It must define the entities, enable `exportSchema = true`, and declare `autoMigrations` for version bumps.

Also, expose a companion object builder to instantiate the database using `Room.databaseBuilder`.

```kotlin
import android.content.Context
import androidx.room.AutoMigration
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [
        // List your entities here...
        UserEntity::class,
        ProfileEntity::class
    ],
    version = 2,
    exportSchema = true, // REQUIRED for AutoMigrations
    autoMigrations = [
        // Define your version jumps here
        AutoMigration(from = 1, to = 2)
    ]
)
abstract class AppDatabase : RoomDatabase() {

    // Abstract DAO definitions
    abstract fun userDao(): UserDao
    abstract fun profileDao(): ProfileDao

    companion object {
        fun getDatabase(context: Context): AppDatabase {
            return Room.databaseBuilder(
                context,
                AppDatabase::class.java,
                "app_database"
            )
                .addMigrations() // Add manual migrations here if AutoMigration isn't enough
                .build()
        }
    }
}
```

## 3. Dependency Injection (`DatabaseModule.kt`)

Use Hilt to provide the singleton instance of your database and individual DAOs. This decouples your Repositories from database instantiation logic.

```kotlin
import android.content.Context
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    // 1. Provide the main Database instance
    @Provides
    @Singleton
    fun provideAppDatabase(
        @ApplicationContext context: Context
    ): AppDatabase {
        return AppDatabase.getDatabase(context)
    }

    // 2. Provide individual DAOs
    @Provides
    @Singleton
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }

    @Provides
    @Singleton
    fun provideProfileDao(database: AppDatabase): ProfileDao {
        return database.profileDao()
    }
}
```

## Summary Checklist

- [ ] Export schema path configured in `ksp` block.
- [ ] `@Database` annotation has `exportSchema = true` and configured `autoMigrations`.
- [ ] `Companion object` provides a `Room.databaseBuilder` factory.
- [ ] `DatabaseModule` provides the `@Singleton` database instance and all DAO interfaces using Hilt.
