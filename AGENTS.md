# AGENTS.md
Guidance for coding agents operating in this repository.

## Repository summary
- Project: C++ Geode mod (`xdBot`, mod id `zilko.xdbot`).
- Build system: CMake + Geode SDK (`setup_geode_mod(...)`).
- Main code lives in `src/`:
  - `hacks/`, `ui/`, `renderer/`, `practice_fixes/`, `utils/`.
- Runtime assets live in `resources/` and are declared in `mod.json`.
- Key project files:
  - `CMakeLists.txt`
  - `mod.json`
  - `.github/workflows/multi-platform.yml`
  - `.github/workflows/android.yml`

## Environment requirements
- `GEODE_SDK` is required.
  - Build fails if undefined (`CMakeLists.txt` has a fatal check).
- C++ standard: C++20.
- Platform-gated code is common:
  - `GEODE_IS_WINDOWS`
  - `GEODE_IS_ANDROID`
- Preserve platform guards when editing shared logic.

## Build commands
Use out-of-source directories.

Standard local build:
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build -j
```

Release build:
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

Android-style directories (match `.gitignore` conventions):
```bash
cmake -S . -B build-android32 -DCMAKE_BUILD_TYPE=Release
cmake --build build-android32 -j

cmake -S . -B build-android64 -DCMAKE_BUILD_TYPE=Release
cmake --build build-android64 -j
```

CI parity notes:
- CI action: `geode-sdk/build-geode-mod@main`.
- Windows uses `RelWithDebInfo`.
- Android32/Android64 use `Release`.

## Lint and format commands
Current repo state:
- No `.clang-format` found.
- No `.editorconfig` found.
- No clang-tidy config or dedicated lint scripts found.

Recommended practice:
- Use compile warnings from normal builds as lint baseline.
- Keep diffs focused; avoid mass reformatting.
- If formatting manually, only format touched regions.

## Test commands
Current repo state:
- No committed test suite found.
- No `tests/` directory and no explicit CTest registration found.

If CTest tests are introduced, use:
```bash
# all tests
ctest --test-dir build --output-on-failure

# single test (regex)
ctest --test-dir build -R "<test-name-regex>" --output-on-failure

# single exact test
ctest --test-dir build -R "^<exact-test-name>$" --output-on-failure
```

Validation expectations for this repo today:
- Treat successful build as the primary automated validation.
- For behavior changes, run focused in-game checks.

## Cursor / Copilot rule files
Checked locations:
- `.cursor/rules/`
- `.cursorrules`
- `.github/copilot-instructions.md`

Result in this snapshot:
- No Cursor rule files found.
- No Copilot instruction file found.

If these files are added later, treat them as higher-priority instructions than this document.

## Code style guide (derived from existing code)

### Includes and dependencies
- Keep include ordering consistent with local file style.
- Common pattern: project headers -> Geode/framework headers -> STL headers.
- Keep platform-specific includes inside `#ifdef` blocks.
- Avoid adding heavy headers in central headers unless needed.

### Formatting
- Follow surrounding style in the file being edited.
- Existing style is mostly 4-space indentation with compact blocks.
- Prefer early returns over deep nesting.
- Do not do unrelated whitespace/style churn.

### Types and conversions
- Use explicit integer widths at boundaries (`int64_t`, etc.).
- Use `std::filesystem::path` for paths.
- Use `static_cast<...>` for explicit conversions.
- Keep code compatible with C++20.

### Naming conventions
- Types/classes: `PascalCase`.
- Functions/methods/variables: mostly `camelCase`.
- Keep existing identifier and setting-key names unless a rename is required.

### State management
- Runtime mutable state is centralized in `Global::get()`.
- Common pattern for repeated access: `auto& g = Global::get();`.
- Respect existing state machine values:
  - `state::none`
  - `state::recording`
  - `state::playing`
- After meaningful state changes, keep UI in sync:
  - `Interface::updateLabels()`
  - `Interface::updateButtons()`

### Error handling
- Prefer defensive null checks for game-layer pointers.
- For non-fatal filesystem ops, use `std::error_code`.
- Use `FLAlertLayer::create(...)->show()` for user-facing failures.
- Use `log::debug`, `log::warn`, `log::info` for diagnostics.
- Avoid throwing exceptions across Geode hook boundaries.

### Hooks and threading
- Preserve Geode hook signatures exactly (`$modify`, `$execute`).
- For UI updates from worker threads, dispatch to main thread:
  - `Loader::get()->queueInMainThread(...)`
- In renderer/audio code, preserve existing lock discipline.

### Platform compatibility
- Keep Windows-only APIs behind Windows guards.
- Avoid introducing Windows assumptions into Android paths.
- Validate both code paths when touching shared subsystems.

### Settings/resources conventions
- New user settings should be declared in `mod.json` with type/default/constraints.
- Saved-value keys are compatibility-sensitive; reuse exact key strings.
- Add new assets to `mod.json` resources when required at runtime.

## Scope guidance for agents
- Prefer surgical changes over broad refactors.
- Preserve macro behavior and file format compatibility.
- Follow existing patterns in `src/ui/`, `src/hacks/`, and `src/renderer/`.

## Pre-merge checklist
- Build succeeds in at least one relevant config.
- No platform-guard regressions introduced.
- No formatting-only churn in unrelated code.
- No new warnings/errors in modified areas.
- `mod.json` updated when settings or resources change.
