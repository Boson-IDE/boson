# Changelog

## 0.1.0

First release.

### Editor and workspace
- Monaco-based editor with split groups, tabs, language servers and Runtime diagnostics.
- Explorer, search, packages, Git source control, integrated documentation and a browser preview.
- Opening a workspace no longer opens a file on its own; the workspace graph is a pinned first tab in every editor group.

### Graph menus, Welcome and Explorer
- Graph views have right-click context menus (also Menu key / Shift+F10) with quick actions for files, folders, symbols and the canvas.
- **Settings → Workbench → Startup** turns the Welcome page on start-up back on or off, and can open it immediately.
- Files inside a folder are indented past the folder's icon, so they no longer line up with files outside it.

### Menus and Discord
- **Settings** now appears once, under **File**, instead of in File, View and Help.
- Discord Rich Presence reports *Connecting*, *refused Application ID* and *not reachable* separately, and the status updates by itself once Discord answers (it previously stayed on "not reachable").

### Security
- **Workspace trust and Restricted Mode.** Folders you have not trusted open in Restricted Mode: the Runtime itself refuses to run builds, tasks, terminals, debugging, language servers, package installs, process tools and Git commit/fetch/pull/push there (`403 WORKSPACE_UNTRUSTED`), while reading, editing, search and read-only Git keep working. Boson asks the first time you open a folder; **Help → Manage Workspace Trust** and the header chip change the decision later. Projects created from a template are trusted automatically.
- Fixed: the Python syntax checker ran with the workspace as its import path, so a file named `ast.py` or `json.py` in a folder could run when the folder was validated. It now runs in isolated mode.

### Window and feedback
- The window remembers its position, size and maximized state between runs. A saved position that is no longer on any connected display is dropped instead of stranding the window.
- **Help → Send Feedback** opens the new-issue page of the project's GitHub repository with the version and platform prefilled.

### Packages
- The Package Explorer now works for every project type in the workspace, not just npm: Rust (crates.io), Python (PyPI), Go, .NET (NuGet), PHP (Packagist), Ruby (RubyGems), Dart/Flutter (pub.dev), Java (Maven Central for Maven and Gradle). Pick an ecosystem, search, then install or copy the snippet. See [Finding and adding packages](docs/guide/packages.md).

### Documentation
- The documentation viewer's list is now a folder tree like the Explorer: click a folder such as **guide** to see only the documents inside it. Folders come first, `index` pages lead each folder, the folder holding the open document expands itself, and searching shows only matching documents with their folders.
- `npm run docs:site` builds a standalone HTML documentation site into `docs-site/` (folder navigation, search, on-page contents, previous/next, light and dark themes). It needs no dependencies and no network.

### Search
- Search covers every file on disk through the Runtime (`POST /v1/workspaces/search`), with match case, whole word, regular expressions, include/exclude globs, line-level results that open at the match, and honest limits. Unsaved editor changes are searched too.

### Performance
- Typing no longer redoes workspace-sized work: the explorer's folder tree and directory list are indexed once per structural change (about 35 ms down to about 1 ms per keystroke in a modeled 6,000-file workspace), the saved-file check looks only at dirty files, and search runs on a deferred copy of the buffers.
- Editor models of files that are no longer open are disposed, so memory follows open tabs instead of everything ever visited.
- Chromium spell checking is off, and idle language servers shut down after 10 minutes instead of 30.

### Debugging
- Debug Node.js programs (`.js`, `.mjs`, `.cjs`): breakpoints in the gutter (F9), step over/into/out, pause, call stack, variables, expression evaluation and a debug console, through the V8 inspector (`/v1/debug/*`). Other languages need Debug Adapter Protocol adapters and are not available yet.

### Navigation and Git
- Go to Symbol in File (`Ctrl/Cmd+Shift+O`) and an Outline in the inspector, from the code graph (TypeScript, JavaScript) or a running language server.
- Find All References in Workspace (text-based, whole word) in the Search view.
- Compare the open file with HEAD or any of its last 50 commits, and Git blame for the current line (`/v1/git/file-at`, `/history`, `/blame`).

### Workbench
- Search can replace across the workspace, with a count and confirmation first.
- Hot exit: unsaved edits are backed up locally and offered for restore after a crash.
- Zen mode, a Welcome page and a Keyboard Shortcuts editor (rebind any command; conflicts are refused).
- Optional Discord Rich Presence (off by default). Packaged builds are named Boson with the Boson icon in the task manager, and `npm run dev` on Windows now uses a branded copy of Electron.
- **Show Process Metrics** lists CPU and memory per Boson process.

### Updates
- The header's save button and Settings' "Save all files" are gone (Ctrl/Cmd+S, Save All and Auto Save remain). An **Update** button appears only when the configured GitHub repository has a newer stable release.
- The Boson icon is set explicitly for Windows, macOS and Linux packages.

### Graphs
- Blueprint-style **workspace graph** (folders, files, imports) and per-file **code graph** (classes, flows, scopes, calls) with inferred, titled groups.
- Large components are split into a **Scope** and one flow per nested function; heavily shared functions get faint wires.
- **Trace flow** lights up the upstream and downstream chain of the hovered node; turning it off disables hover tracing.
- Viewport culling with three levels of detail, even sampling when a view is capped, and sub-pixel edges dropped when zoomed out.
- Mouse thumb buttons (Back and Forward) navigate history in the editor, graphs, documentation and browser preview.

### Terminal
- Any number of independent terminals (up to 8), each with its own Runtime shell. Terminals keep running while the panel is collapsed or another panel is shown.

### Workspace Runtime
- Informational logging is opt-in: `--verbose` or `BOSON_RUNTIME_VERBOSE=true` (desktop: `--runtime-verbose`).
- Code-graph limits raised and configurable (`BOSON_CODE_GRAPH_MAX_NODES`, `_EDGES`, `_AST_NODES`, `_SOURCE_BYTES`); workspace-graph limits (`BOSON_GRAPH_*`).
- Token written to a mode-600 file instead of the console.
