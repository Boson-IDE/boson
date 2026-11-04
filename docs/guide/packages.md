# Finding and adding packages

Open **Packages** in the activity bar to find dependencies for any project type in your workspace, the same way for every language.

## How it works

1. **Pick an ecosystem.** The row of chips lists every supported ecosystem. Those with projects in your workspace come first and show how many they found; Boson starts on the first one it detects.
2. **Pick the target project.** The list holds every folder with that ecosystem's manifest (for example each `Cargo.toml` or `.csproj`). Folders such as `node_modules`, `target`, `vendor`, `bin` and `obj` are skipped.
3. **Search.** Type at least two characters. Results show the latest version, the description, download counts where the registry reports them, and a ready-to-paste snippet.
4. **Install or copy.** Where Boson can run the package tool it shows **Install Package**; otherwise it shows **Copy snippet**. Installing always asks for confirmation first, and its output appears in the Output panel.

## Supported ecosystems

| Ecosystem | Registry | Search | What Install does |
|---|---|---|---|
| JavaScript / TypeScript | npm | keyword | `npm install --save` (or `--save-dev`) with lifecycle scripts disabled |
| Rust | crates.io | keyword | `cargo add` (or `cargo add --dev`) |
| Python | PyPI | exact name only | `uv add` or `poetry add` when the project has `uv.lock` or `poetry.lock`; otherwise copy `name==version` into `requirements.txt` or `pyproject.toml` |
| Go | Go module proxy | full module path only | `go get module@version` |
| .NET | NuGet | keyword | `dotnet add package` |
| PHP | Packagist | keyword | `composer require` with `--no-scripts --no-plugins` |
| Ruby | RubyGems | keyword | `bundle add --skip-install`, which only edits the `Gemfile` |
| Dart / Flutter | pub.dev | keyword | `dart pub add` (`flutter pub add` in Flutter projects) |
| Java (Maven) | Maven Central | keyword | snippet for `pom.xml` |
| Java / Kotlin (Gradle) | Maven Central | keyword | snippet for `build.gradle` or `build.gradle.kts` |

PyPI and Go publish no search endpoint, so those two look up the exact name you type and say so when nothing matches. Maven and Gradle have no command that adds a dependency, so Boson shows the snippet instead of editing your build file behind your back.

## Safety

- Only the Runtime talks to registries, using fixed registry addresses. Package names and versions are validated before they are used in a command, and a name that could be read as an option (for example one starting with `-`) is refused.
- Commands run without a shell, in the selected project folder only, and only after you confirm.
- Where the tool allows it, install scripts are disabled (npm `--ignore-scripts`, Composer `--no-scripts --no-plugins`, Bundler `--skip-install`). Cargo, Go, NuGet, Dart and Python installs can still fetch code that runs when you later build or run the project; review packages as you would anywhere else.
- Install needs the tool on the Runtime machine's `PATH` (`cargo`, `go`, `dotnet`, `composer`, `bundle`, `dart`, `uv` or `poetry`). If it is missing, the Output panel shows the error.
