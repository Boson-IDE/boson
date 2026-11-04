# Architecture

Boson is an npm-workspaces monorepo. One shared React UI runs inside two hosts (Electron and Next.js), and one Node.js service, the Workspace Runtime, performs everything that touches processes, project files on the host, Git or language tooling.

## Overview

```text
 ┌────────────────────────┐        ┌────────────────────────┐
 │  apps/desktop          │        │  apps/web              │
 │  Electron host         │        │  Next.js host          │
 │  preload IPC bridge    │        │  browser UI only       │
 └───────────┬────────────┘        └───────────┬────────────┘
             │           both render           │
             └──────────► packages/ui ◄────────┘
                    (shared React IDE shell)
                               │  HTTP + SSE, bearer token
                               ▼
              ┌────────────────────────────────────┐
              │  apps/runtime  (Workspace Runtime) │
              │  tasks · terminals · file access   │
              │  watcher · Git · packages          │
              │  workspace analysis · scaffolding  │
              │  language servers · browser proxy  │
              └────────────────────────────────────┘
```

On desktop the main process starts the Runtime and relays requests. In the browser the UI talks to a separately started Runtime directly, so the same features work on a remote host.

## Repository layout

```text
apps/
  desktop/    Electron app: main process, preload bridge, renderer entry
  runtime/    Workspace Runtime service and its tests
  web/        Next.js browser application
packages/
  contracts/  Shared TypeScript contracts: desktop bridge, workspace, task, plugin types
  themes/     Design tokens, IDE styles, Monaco themes, brand geometry
  ui/         Shared React IDE shell, editor, dialogs, layouts, translations
examples/     Example plugin manifest and embedding example
docs/         Guides (this folder)
```

### Inside the Runtime (`apps/runtime/src`)

| File | Responsibility |
|---|---|
| `index.js` | HTTP server, authentication, CORS, tasks, terminals, file and Git endpoints, packages, browser proxy, documentation server |
| `workspace-analysis.js` | Module discovery and dependency graph, built-in language validation, dependency-ordered build plans |
| `cmake-validation.js` | Project-aware CMake validation used by workspace analysis |
| `workspace-watcher.js` | Bounded, cross-platform directory watching with debounced change events |
| `project-scaffolding.js` | Workspace and module templates, and `boson.workspace.json` handling |
| `language-servers.js` | Language server adapters, process lifecycle, and the allow-listed LSP methods |

### Inside the shared UI (`packages/ui/src`)

| Area | Notes |
|---|---|
| `BosonShell.tsx` | The application shell and most state |
| `layouts/` | `BosonIDELayout`, the overall composition |
| `components/` | Code editor, editor group grid, dialogs, Markdown and asset previews, terminal, browser tab, project setup panel, CMake validator |
| `editorLayout.mjs`, `editorDropTarget.mjs`, `editorGroupTransfer.mjs` | Pure logic for the recursive editor grid, drop-edge hit testing and moving tabs between groups. Plain modules so they can be unit tested |
| `i18n.ts` | Dependency-free UI translations |
| `components/BosonDocumentation.tsx` | The in-app documentation viewer (Markdown reader and sandboxed web-document frame) |
| `components/BosonDialogProvider.tsx` | Shared asynchronous confirmation queue used instead of native dialogs |
| `editorLayoutPersistence.mjs`, `tabDropOrder.mjs`, `fileDragUrl.mjs`, `editorTokenization.mjs` | Versioned layout save/restore, tab drop ordering, file-drag URLs and tokenization helpers, kept as plain testable modules |
| `brand/` | `BosonMark` logo, splash, activity indicators |

## Packages

| Package | Responsibility |
|---|---|
| `@boson/ui` | Application shell, IDE layouts, Explorer, editor panels, toolbars, menus, dialogs, command palette, settings, workspace interface |
| `@boson/themes` | Brand colors, typography, theme definitions, CSS design tokens, editor themes, component appearance |
| `@boson/contracts` | Shared interfaces: workspace snapshots, modules and dependencies, diagnostics, build plans, scaffolding, task and Runtime events, the desktop bridge, and plugin manifests |

## Boundaries

- **The shared UI is runtime-agnostic.** It never imports Electron APIs, Node.js filesystem APIs or Next.js server-only modules. Platform features are supplied through interfaces by the host.
- **Desktop** exposes window, dialog and filesystem operations through the typed preload bridge. Runtime requests and event streams go through the main process, which attaches the bearer token and only forwards an explicit allow-list of routes.
- **Web** has no filesystem access of its own beyond imports. Real workspaces live on a Runtime host and are reached through the Runtime API.
- **The Runtime is the only component that starts project processes** or reads and writes workspace files on the host. It never runs language servers or build commands because of workspace contents alone.

## Documentation viewer

The docs in the repository's `docs/` folder are shipped with the app and read through the Runtime: the viewer lists files from `GET /v1/docs`, renders Markdown with the shared Markdown reader, and shows HTML documents in a sandboxed frame served from short-lived session URLs. See [Documentation viewer](../guide/documentation-viewer.md).

## Trust boundaries

The security-relevant boundaries are the Runtime token (full control of the host account), the sandboxed browser tab, the sandboxed documentation frame, and the Electron preload bridge. They are summarized in [Security](../reference/security.md).

## Localization

All UI copy goes through `t('English text')`, and `i18n.ts` holds translations for ten languages besides English. Untranslated text falls back to English. Source code, compiler output and tool messages are never translated. See [Contributing → Localization](contributing.md#localization).

## UI layout

The desktop composition is the visual reference: top toolbar, left activity bar, Explorer, a recursive grid of editor groups, bottom Output/Problems/Terminal panel, right Inspector and status bar. The browser keeps this composition on wide screens and adapts below it: the Inspector collapses on medium screens, the Explorer becomes a drawer on narrow screens, and the editor gets priority on small ones.

## Brand and visuals

| Name | Hex | Use |
|---|---|---|
| Ink | `#0C1017` | Primary dark surface |
| Field | `#2F5BFF` | Primary interaction and emphasis |
| Photon | `#C6F432` | Indicators, particles, active states |
| Paper | `#F4F6F8` | Light-theme surface |

Typography: **Space Grotesk** for the interface, **JetBrains Mono** for code, terminal and numbers (both SIL OFL 1.1; prefer bundled copies over runtime Google Fonts).

The `BosonMark` logo component is the single approved implementation and must be reused everywhere. Its geometry (`0 0 64 64` viewBox, particle at `cx=47, cy=32`) lives in `packages/themes/src/brand.ts` and is shared with the native desktop splash. Do not redesign the mark. See AGENTS.md (`AGENTS.md`).

Motion respects `prefers-reduced-motion`. Brand timings: a 3-second logo loop, a 1.1-second inline spinner rotation, and a 1.6-second indeterminate bar sweep.

## Roadmap direction

The long-term plan (Laravel control plane for hosted workspaces, development environments, debug adapters and an extension host) is described in AGENTS.md (`AGENTS.md`). Those parts are targets, not current features.
