# AGENTS.md - Paper Minecraft Server

## Project Overview

Paper is a Minecraft server fork optimized for performance and extensibility. Target: Minecraft 1.21.8, Java 21.

### Module Structure
| Module | Purpose |
|---|---|
| `paper-api` | Bukkit/Spigot API + Paper extensions (plugin-facing API) |
| `paper-server` | Server implementation + Minecraft patches (NMS) |
| `paper-generator` | Code generation utilities |
| `test-plugin` | Test plugin for API verification (optional) |

## Build / Test Commands

### Prerequisites
- **JDK 21** (Gradle Toolchains auto-provisions if JRE 11+ available)
- **Git** (required — will not build without `.git` directory)

### Core Gradle Commands
```bash
# Build everything
./gradlew build

# Build specific module
./gradlew :paper-api:build
./gradlew :paper-server:build

# Run all tests
./gradlew test

# Run a single test class
./gradlew :paper-api:test --tests "org.bukkit.NamespacedKeyTest"
./gradlew :paper-server:test --tests "org.bukkit.craftbukkit.inventory.ItemStackTest"

# Run a single test method
./gradlew :paper-api:test --tests "org.bukkit.NamespacedKeyTest.testKeyCreation"

# Run test suite (paper-server uses suite-based tests)
./gradlew :paper-server:test --tests "org.bukkit.support.suite.NormalTestSuite"
```

### Running a Test Server
```bash
./gradlew runDevServer        # Dev mode (no jar assembly)
./gradlew runServer           # Mojang-mapped server jar
./gradlew runReobfServer      # Reobfuscated jar
./gradlew runBundler          # Mojang-mapped bundler jar
./gradlew runPaperclip        # Paperclip jar (distribution format)
```

### Patch Workflow (Modifying Minecraft)
```bash
./gradlew applyPatches        # Apply patches to paper-server/src/minecraft
./gradlew rebuildPatches      # Rebuild patch files from git commits
./gradlew fixupSourcePatches  # Fix per-file patches after edits
```

### Local Maven Install
```bash
./gradlew publishToMavenLocal  # Install API to local Maven for plugin testing
```

### CI Skip
Add `[ci skip]` to commit subject for doc-only changes.

## Code Style

### General
- **Language**: Java 21, Kotlin DSL for Gradle
- **Indentation**: 4 spaces (2 for YAML, `.tiny` uses tabs)
- **Line endings**: LF (CRLF for `.bat`)
- **Encoding**: UTF-8
- **Style**: Oracle/Java standard style (IDE default formatting)
- **Line length**: Flexible — go over 80 chars if readability isn't hurt
- **Match surrounding code style** when in doubt

### Naming Conventions
- Classes: `PascalCase`
- Methods/Fields: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Config fields: `camelCase` (auto-converted to `kebab-case` in config files)

### `var` Keyword
**Discouraged** — makes patch files harder to read and causes update confusion. Only use when lines would be excessively long with obvious generic types.

### Imports
- **In Vanilla classes**: Use fully qualified class names (FQN) instead of adding imports. Add import with `// Paper -` comment only if type is used many times.
- Import layout: `*, |, $*` (wildcards at top, separated)
- No import-on-demand threshold (`999999` — effectively disabled)
- No inner class imports

### Nullability Annotations
- **New classes**: Use `@NullMarked` from `org.jspecify.annotations` (non-null by default), mark nullable fields/params/returns with `@Nullable` from JSpecify.
- **Existing classes**: Keep using `@Nullable` / `@NotNull` from `org.jetbrains.annotations` (will be migrated later).

### Validation / Preconditions
Use `Preconditions` class (not `checkNotNull`):
```java
Preconditions.checkArgument(arg != null, "arg cannot be null");
Preconditions.checkState(this.state != null, "state cannot be null");
```

### Paper Change Markers (Vanilla modifications)
All Minecraft modifications must be marked:
```java
entity.doSomething(); // Paper - reason for change

// Paper start - descriptive label
// old code commented out
newCode();
// Paper end - descriptive label
```

### Final Variables
`final` on locals and parameters is auto-configured in IDE settings.

### Error Handling
- Use `Preconditions` for argument/state validation
- Throw `IllegalArgumentException` for invalid arguments (not `NullPointerException`)
- Use descriptive error messages with format placeholders: `"player %s must be online", player.getName()`

### Configuration
- Add settings to `GlobalConfiguration` (global) or `WorldConfiguration` (per-world) in `io.papermc.paper.configuration`
- Access: `GlobalConfiguration.get().misc.settingName` or `level.paperConfig().misc.settingName`

### Annotations
- `@DoNotUse` — marks API that should not be called (scanned by `scanJarForBadCalls`)
- `@GeneratedFrom` — marks auto-generated code

## Testing
- **Framework**: JUnit 5 (Jupiter) with `@Test`, parameterized tests
- **Mocking**: Mockito 5.14.1 (requires `-javaagent` on Java 21+)
- **Server tests**: Organized into test suites (`NormalTestSuite`, `LegacyTestSuite`, `SlowTestSuite`, `AllFeaturesTestSuite`)
- Tests use `@Suite` annotation for suite-based execution
- Compile tests with `-parameters` flag for better JUnit names
- `forkEvery = 1` for server tests (isolation)
- `Slow` tag excluded by default

## Key Directories
```
paper-api/src/main/java/          # API source code
paper-api/src/test/java/          # API tests
paper-api/src/generated/java/     # Auto-generated API code
paper-server/src/main/java/       # Server implementation
paper-server/src/test/java/       # Server tests
paper-server/src/minecraft/       # (gitignored) Deobfuscated Minecraft source + patches
paper-server/patches/             # Git patch files for Minecraft modifications
build-data/                       # Build configuration, access transformers
```

## Useful Notes
- `paper-server/src/minecraft` is **gitignored** — it's a special workspace created by `applyPatches`
- Patches live in `paper-server/patches/` as `.patch` files
- Access transformers: `build-data/paper.at`
- Gradle config: parallel builds, configuration cache, and caching enabled
- Memory: compile tasks bumped to 1GB; server defaults to 2GB heap
