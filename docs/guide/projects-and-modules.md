# Projects, modules and build plans

Boson treats a workspace as a set of **modules**, each with its own language and build system, rather than assuming one toolchain for the whole repository. This guide covers creating projects and modules, the optional `boson.workspace.json` file, and how Boson works out build order.

## Creating projects and modules

Projects you create from a template with **New Project…** are trusted automatically (see [Security](../reference/security.md#workspace-trust-and-restricted-mode)).

Open **File → New Project…** or **File → New Module…**. Both open the Project Setup dialog, which needs a filesystem workspace connected to a Workspace Runtime.

> Boson writes configuration and starter files only. It never installs dependencies and never runs build commands as part of setup.

### Initialize a workspace

**New Project…** configures the open folder. Pick a name and a template:

| Template | Intended for |
|---|---|
| Generic | Metadata only, no build system |
| JavaScript / TypeScript monorepo (npm) | npm workspaces with a shared `tsconfig.base.json` |
| JavaScript / TypeScript monorepo (pnpm) | `pnpm-workspace.yaml` plus a shared `tsconfig.base.json` |
| C / C++ (CMake) | A root CMake project |
| Rust (Cargo) | A Cargo workspace |
| Python (uv) | A `pyproject.toml` workspace |
| Kotlin / JVM (Gradle) | Gradle multi-project (`settings.gradle.kts`, `build.gradle.kts`) |
| Java (Maven) | Maven multi-module (`pom.xml`) |
| Go | A Go workspace |
| Swift Package Manager | A `Package.swift` workspace |

Initialization never overwrites an existing file. Files that already exist are left alone, and `boson.workspace.json` is merged rather than replaced (invalid JSON in it stops the operation with an error). If any write fails, the files written so far are removed.

### Add a module

**New Module…** creates a starter module and connects it to the workspace's build.

| Module template | Native build chain |
|---|---|
| TypeScript, JavaScript | npm (or pnpm when the workspace declares pnpm) |
| Python | uv |
| Rust | Cargo |
| C, C++ | CMake |
| Java | Maven |
| C# | .NET |
| Go | Go workspace |
| Kotlin | Gradle |
| Swift | Swift Package Manager |

You choose a name, a folder (default `packages/<name>`) and a **build chain**:

- **Auto** picks the module's native toolchain. A Rust module in an npm workspace gets a Cargo module rather than being forced into npm.
- **Metadata** records the module in `boson.workspace.json` without changing a build file.
- A specific chain must be compatible with the template, otherwise Boson explains which chains are allowed.

Besides the module's own starter files, Boson registers the module in the root build file for the chosen chain: root `package.json` workspaces, `pnpm-workspace.yaml`, `Cargo.toml`, `CMakeLists.txt`, `pom.xml`, Gradle settings, `go.work` or `pyproject.toml`. The module's folder must not already exist, must stay inside the workspace, and may only use letters, numbers, dots, underscores, hyphens and `@` scopes. On failure, the module folder and any root edits are rolled back.

## `boson.workspace.json`

An optional file at the workspace root that records Boson's view of the project.

```json
{
  "schemaVersion": 1,
  "name": "my-workspace",
  "modules": [
    {
      "id": "packages/engine",
      "path": "packages/engine",
      "name": "engine",
      "template": "cpp",
      "language": "cpp",
      "buildSystem": "cmake"
    }
  ],
  "buildChains": ["cmake"]
}
```

Boson also discovers modules on its own from manifests such as `package.json`, `Cargo.toml`, `CMakeLists.txt`, `pom.xml`, Gradle files, `go.work`, `pyproject.toml`, .NET projects, `Package.swift`, `composer.json`, Ruby bundles and Zig builds, so existing projects work without this file.

## The module graph

The Runtime builds a graph of modules and the dependencies between them. An edge can come from a declared dependency (including dev, peer and optional dependencies in npm manifests), workspace membership, a CMake subdirectory, or an explicit module relationship. This graph drives the workspace overview and the build plan.

## Build plans

For a filesystem workspace, Boson shows a **dependency-aware build plan**: modules in an order where dependencies come before the modules that use them, with the command planned for each.

- When the root build file already orchestrates all children (for example a Maven aggregator or an npm workspace root), its own aggregate step is omitted so children are not built twice, and a note says so.
- Modules Boson cannot plan are listed as skipped.
- If the graph contains a **cycle**, the plan is refused with an error that lists the cycle. Break the cycle before building.

The plan is a preview and a source of truth for ordering. Build, Test and Run tasks still use the task planner described in [Using workspaces → Building, testing and running](workspaces.md#building-testing-and-running).

## Safety

Scaffolding and analysis are designed not to execute project code: scaffolding writes files with exclusive creation, validates every path against the workspace root, and does not run build scripts. Language servers are never started from workspace contents (see [Language support](language-support.md)).
