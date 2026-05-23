# Constructly Android Skills

A public multi-skill repository for Android coding agents working with Jetpack Compose, Hilt, Room, Navigation 3, and related Android architecture patterns.

Each folder in this repository is an installable skill. Install one or more skills by pointing the Codex skill installer at a specific folder path in the GitHub repo.

## Repository Layout

- `anti-patterns`: Review Android and Compose code for risky patterns.
- `app-language-switching`: Add per-app language switching with modern Android locale APIs.
- `architecture`: Apply clean Android architecture boundaries.
- `dependency-injection`: Wire Android dependencies with Hilt.
- `dto-entity-alignment`: Align remote DTOs with local persistence models.
- `experimental-grid`: Use Compose Grid APIs for structured layouts.
- `flow-operators`: Derive Android UI state with Kotlin Flow.
- `hilt-workmanager`: Integrate Hilt with WorkManager.
- `material-expressive-theme`: Configure Material Expressive theming.
- `modularity`: Keep files small, focused, and navigable.
- `navigation`: Implement type-safe Navigation 3 flows.
- `pull-to-refresh`: Add Material 3 pull-to-refresh behavior.
- `room-setup`: Configure Room with schema exports and migrations.
- `screen-overloads`: Split screens into stateful and stateless layers.
- `search-feature`: Build debounced search without cursor-jump issues.
- `textfield-best-practices`: Improve TextField behavior and IME handling.
- `ui-standards`: Enforce UI consistency and theme-safe patterns.

## Installation

Install a single skill from this repository by targeting its folder path:

```bash
install-skill-from-github.py --repo <your-github-user>/<your-repo> --path architecture
```

Install multiple skills in one command:

```bash
install-skill-from-github.py --repo <your-github-user>/<your-repo> --path architecture --path dependency-injection --path navigation
```

If you prefer the GitHub URL form:

```bash
install-skill-from-github.py --url https://github.com/<your-github-user>/<your-repo>/tree/main/architecture
```

## Publishing Notes

- Keep each skill folder self-contained with `SKILL.md` and optional `agents/openai.yaml`.
- Put repo-level documentation in this `README.md`, not inside skill folders.
- Choose and add a license before publishing publicly.
