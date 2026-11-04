# Boson

Boson is a cross-platform development environment for multi-language projects. It combines a code editor, project explorer, terminal, Git, package manager, language diagnostics and task output in one workspace, and runs as an **Electron desktop app** or in the **browser**, using the same shared React UI.

Boson is organized around the workspace rather than a single language or build system. A workspace can contain independent modules and toolchains. The editor explores and changes source files; the **Workspace Runtime** is the process boundary that builds, tests and runs trusted projects, watches files, runs Git and hosts language servers.

> **Status:** active development (version 0.1.0). Node.js debugging works; debug adapters for other languages, custom tasks and hosted multi-user workspaces are not available yet. Read [Security](docs/reference/security.md) before exposing a Runtime beyond your own machine.

## Read the docs inside Boson

The documentation ships with the app. Choose **Help → Documentation** (or run **Documentation** from the command palette, `Ctrl/Cmd+Shift+P`) to browse the guides, the reference and an interactive shortcut finder without leaving your work. See [Documentation viewer](docs/guide/documentation-viewer.md).

## Quick start

**Requirements:** Node.js 20+ (22 LTS recommended), npm 10+, and the toolchain for any project you want to build (for example a C/C++ compiler and CMake, Python, Go, a JDK and Maven, or .NET).

```bash
npm install        # once, from the repository root (npm workspaces)
npm run dev        # desktop app
```

The desktop app starts and manages its own Workspace Runtime, so there is nothing else to launch. Use **File → Open Workspace…**, pick a folder, and start working.

**Browser version:**

```bash
npm run dev:web        # app at http://localhost:3000
npm run dev:runtime    # in another terminal: the Runtime that holds your projects
```

Then connect them in **Settings → Workspace Runtime** (the URL is printed by the Runtime; a generated token is written to `~/.boson/runtime-token`, or set `BOSON_RUNTIME_TOKEN` yourself) and use **Choose a folder** to open a folder on the Runtime host. See [Getting started](docs/guide/getting-started.md) and [Workspace Runtime](docs/reference/workspace-runtime.md).

## Features

| Area | Highlights |
|---|---|
| **Editing** | Monaco editor; recursive side-by-side and stacked editor groups (up to 4 × 4) with drag-to-split; layout and tabs remembered per workspace; Quick Open, workspace search and command palette; auto-save; extensive settings |
| **Projects and modules** | Create workspaces and modules (npm, pnpm, Cargo, CMake, Maven, Gradle, Go, Swift, Python, .NET); module graph and dependency-ordered build plans |
| **Diagnostics** | Runtime-owned validation for TypeScript/JavaScript, Python, JSON and CMake, plus 38 language server adapters; one Problems panel, editor markers and Explorer badges |
| **Build, test, run** | Detects npm/pnpm/Yarn/Bun, Cargo, CMake, Maven, Gradle, Python, Go and .NET; streams output; tasks can be stopped |
| **Terminal** | Real interactive shell (`node-pty` + xterm.js) with a pipe-mode fallback |
| **Source control** | Git status, diff, stage, commit, branches, remotes, fetch, pull and push |
| **Packages** | Find npm modules, search the registry and install packages |
| **Remote workspaces** | The browser app opens and edits folders on a Runtime host, with live file watching |
| **Previews** | Markdown, images, SVG (sandboxed), PDF, audio and video, plus an in-app browser tab |
| **Built-in documentation** | Markdown guides and interactive web pages in a dialog, from anywhere in the app |
| **Customization** | 11 interface languages; custom themes; declarative plugins (themes, snippets, commands) |

## Documentation

All guides live in [`docs/`](docs/index.md) and are readable in the app:

| Guide | What it covers |
|---|---|
| [Getting started](docs/guide/getting-started.md) | A ten-minute tour |
| [Using workspaces](docs/guide/workspaces.md) | The workbench, editor groups, previews, tasks, Git, packages, limits |
| [Projects, modules and build plans](docs/guide/projects-and-modules.md) | New Project / New Module, `boson.workspace.json`, build order |
| [Language support](docs/guide/language-support.md) | Diagnostics, validators, language server list and setup |
| [Custom themes](docs/guide/custom-themes.md) · [Plugins](docs/guide/plugins.md) | Theme and plugin formats |
| [Finding and adding packages](docs/guide/packages.md) | The Package Explorer for every supported ecosystem |
| [Documentation viewer](docs/guide/documentation-viewer.md) | How the viewer works and how to write documents for it |
| [Troubleshooting](docs/guide/troubleshooting.md) | Symptoms and fixes |
| [Workspace Runtime](docs/reference/workspace-runtime.md) | Setup, configuration, HTTP API |
| [Security](docs/reference/security.md) | What the Runtime can do and how to run it safely |
| [Architecture](docs/development/architecture.md) | Layout and package boundaries |
| [Building, testing and packaging](docs/development/building.md) | Commands, tests, installers, release checklist |
| [Contributing](docs/development/contributing.md) | Rules and workflows for changing the code |
| [AGENTS.md](AGENTS.md) | Roadmap and handoff notes for AI coding agents |

App notes: [`apps/desktop`](apps/desktop/README.md) · [`apps/web`](apps/web/README.md) · [`apps/runtime`](apps/runtime/README.md).

## Repository layout

```text
apps/
  desktop/    Electron desktop application
  runtime/    Workspace Runtime service
  web/        Next.js browser application
packages/
  contracts/  Shared contracts: desktop bridge, workspace, task and plugin types
  themes/     Design tokens, IDE styles and Monaco themes
  ui/         Shared React IDE shell and editor components
docs/         User and developer documentation (shipped with the app)
examples/     Example plugin manifest and embedding example
```

## Security in one paragraph

The Workspace Runtime runs build scripts, compilers, package installs and an interactive shell **with the permissions of the user who started it**, and its token grants all of that. Folders you have not trusted open in Restricted Mode, where the Runtime refuses to run their code (Boson asks the first time you open one). Open only projects you trust, keep the Runtime on localhost unless you have set up HTTPS (or a tunnel), a long random token and an exact origin allow-list. Details are in [Security](docs/reference/security.md).
