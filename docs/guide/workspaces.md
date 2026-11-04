# Using workspaces

This guide covers day-to-day use of Boson: opening a project, editing, splitting the editor, previews, tasks, Git and packages, and the limits that apply on desktop and in the browser.

- Creating projects and modules: [Projects, modules and build plans](projects-and-modules.md)
- Diagnostics and language servers: [Language support](language-support.md)

> **Open only projects you trust.** Building, running, installing packages, Git hooks and some language servers execute code from the project. See [Security](../reference/security.md#untrusted-projects).

## Opening a workspace

**File → Open Workspace…** (`Ctrl/Cmd+O`) opens the workspace dialog.

| Where you are | **Choose a folder** | **Import a folder / Import files** |
|---|---|---|
| **Desktop app** | Browse folders on your computer inside Boson, then **Open this folder**. Edits save straight to disk. | Temporary read-only snapshot |
| **Browser app** | Browse folders on the **Workspace Runtime host** (the machine where the Runtime runs), then **Open this folder**. This becomes a *remote workspace*. | If a remote workspace is open, files are **uploaded into it**. Otherwise a temporary read-only snapshot |

### Trust and Restricted Mode

The first time you open a folder through a Workspace Runtime, Boson asks whether you trust it. **Trust this folder** allows Build, Run, terminals, debugging, language servers, package installs and Git push, pull and commit. **Open in Restricted Mode** keeps reading, editing and searching and blocks those, and the header shows a **Restricted Mode** chip you can click to change your mind. Projects you create from a template are trusted automatically. Details: [Security](../reference/security.md#workspace-trust-and-restricted-mode).

### Remote workspaces (browser)

In the browser, a workspace lives on the Runtime host, not in your browser:

1. Connect to a Runtime in **Settings → Workspace Runtime** (see [Workspace Runtime](../reference/workspace-runtime.md)).
2. Choose **Choose a folder**. The browser starts in the current workspace folder, or the **Workspace Runtime-visible path** from Settings, or the host's filesystem roots.
3. Boson registers the folder with the Runtime, which must have it inside its allowed workspace roots, then indexes it.

Saving, creating, renaming, moving and deleting files all act directly on the Runtime host's folder, and build, test, run, terminal, Git, packages and diagnostics work on those same files. Changes made on the host by other tools appear live (see [Live file watching](#live-file-watching)).

**Importing into a remote workspace** uploads the selected text files, and images, audio, video and PDFs, into the open workspace, overwriting files at matching relative paths. Importing never deletes anything on the host. Use a dedicated folder if you don't want browser uploads mixed with other files.

### Read-only snapshots

A snapshot is a temporary copy held in the browser. You can read and edit in memory and download files individually, but a snapshot cannot build, test, run, use Git, or be validated. A banner reads **READ-ONLY SNAPSHOT**; choose **Open writable folder…** to switch to a real workspace.

## The workbench

The left activity bar switches between **Explorer**, **Search**, **Source Control**, **Packages**, **Toolchains** and **Settings**. The editor sits in the centre, with the Inspector on the right and the Output/Problems/Terminal panel below. Drag the Explorer, Inspector and bottom-panel dividers to resize them (double-click a divider to reset; the bottom divider also works with the keyboard). Sizes persist.

### Menus and shortcuts

| Action | Shortcut |
|---|---|
| Command palette (Boson commands plus the active editor's actions; type `>` to filter commands) | `Ctrl/Cmd+Shift+P` or `F1` |
| New file | `Ctrl+N` |
| Open workspace | `Ctrl+O` |
| Save / Save all | `Ctrl+S` / `Ctrl+Shift+S` |
| Undo / Redo | `Ctrl+Z` / `Ctrl+Y` |
| Toggle sidebar / Inspector | `Ctrl+B` / `Ctrl+Shift+B` |
| Find in file | `Ctrl+F` |
| Search in files | `Ctrl+Shift+F` |
| Toggle terminal | `Ctrl/Cmd` plus the backtick key |
| Run active file | `F5` |

The **Run** menu has Build Workspace, Run Tests, Run Active File and Stop Current Task. The **View** menu has the editor split commands. The **Help** menu opens [the documentation](documentation-viewer.md) and **About Boson**. A complete, searchable list is in the [shortcut reference](../web/shortcuts.html). On macOS use `Cmd` in place of `Ctrl`.

## Explorer

- Create files and folders from the toolbar (**New File**, **New Folder**) or a folder's right-click menu, name them inline, then confirm.
- Rename inline, use **Move to…**, or drag onto a folder or the workspace root.
- Right-click an entry for open, rename, move and delete.
- Right-click the workspace root and choose **Refresh** to re-index. Pending editor changes are saved first.
- Expanded folders are remembered per workspace.
- Files and folders show diagnostic badges. A folder shows a badge if anything inside it has a problem.

### Live file watching

For desktop and remote workspaces the Runtime watches the workspace and pushes change events to Boson, so files changed outside the IDE appear without a manual refresh. Watching is bounded: at most 512 directories (watching stops adding more past that), nine levels deep, changes debounced by about 90 ms, symlinks never followed, and dependency/build folders such as `node_modules`, `.git`, `dist`, `build`, `target`, `.venv` and `out` skipped. The Runtime allows at most 24 concurrent watch streams.

## Editor

### Tabs and groups

- A single click opens a file in a temporary preview tab; double-click or edit to pin it. Drag tabs to reorder them.
- **Split the editor** from the **View** menu or a group's header: *Split Editor Horizontally*, *Split Editor Vertically*, *Split Entire Layout to the Right*, *Split Entire Layout Below*, and *Close Editor Split*. A **Split scope** control in the group header chooses whether a new split applies to **This group** or the **Entire layout**.
- **Drag a tab or an Explorer file** onto the right edge of a group to open it side by side, or onto the bottom edge to stack it. Only the right and bottom edges are split zones; where they overlap, the nearer edge wins.
- Groups nest recursively and can be resized. The group header offers equal, favour-existing-group and favour-new-group ratios. Ratios stay between 15% and 85%.
- The layout holds up to **4 columns × 4 rows (16 groups)**.
- Closing the last tab in a group removes that group.
- Drop a dragged tab on the left or right half of another tab to insert it before or after that tab.
- Boson remembers each workspace's editor layout (the split tree and its proportions), up to 40 open tabs and the active file, and restores them the next time you open it.
- Markdown, SVG, asset and browser tabs all work in any group. Ctrl/Cmd+click on a link opens its target in the group where you clicked.

### Editing features

- Monaco-based editing with language modes and unsaved-change indicators.
- Quick Open, in-workspace text search and optional auto-save after a delay.
- Settings cover font, size, line height, tab width, indentation, word wrap, line numbers, minimap, ligatures, folding, rulers, cursor style, current-line highlight, Ctrl+wheel zoom, scroll beyond last line, bracket and indent guides, sticky scroll, smooth scrolling and format-on-paste/type. Each section has reset controls.
- **Settings** sections: Appearance, Custom Themes, Editor, Extensions, Language, Panels, Syntax Highlighting, Workbench, Workspace Runtime and Workspace.
- Panel typography and icon sizes are under **Settings → Panels**, in fixed pixels or responsive `rem`.
- Dark and light themes with customizable syntax colors; see [Custom themes](custom-themes.md). Plugins add commands, snippets and themes; see [Plugin manifests](plugins.md).
- Preferences are stored locally on the device.

### Interface language

**Settings → Language** sets the interface language: English, Spanish, Russian, French, German, Dutch, Portuguese (Brazil), Italian, Japanese, Chinese (Simplified) or Korean. The choice is saved on the device and shared by the desktop and browser versions. Your source code and tool output, such as compiler messages, stay in the language their source emits.

### Diagnostics

Problems come from the Workspace Runtime, not from the editor. It validates TypeScript/JavaScript, JSON/JSONC, Python and CMake, and hosts language servers for 38 languages. CMake validation is project-aware: it checks command structure, block matching, likely misspelled commands, and target, include and subdirectory references against the whole workspace, including your unsaved edits. Monaco keeps syntax highlighting but its own TypeScript/JavaScript diagnostics are switched off. Results appear as editor markers, in the **Problems** panel and as Explorer badges. See [Language support](language-support.md).

This is analysis, not a build. **Build/Test still run your installed toolchain.**

## Search

**Search** (`Ctrl/Cmd+Shift+F`) finds file names and file contents. With a Workspace Runtime (always the case in the desktop app) it searches the files on disk, so it covers the whole workspace, not only the files loaded in the editor. Unsaved changes in open editors are searched as well and take priority over the saved version.

- **Aa** matches case, **ab** matches whole words, **.\*** treats the query as a regular expression. An invalid or empty-matching pattern is explained instead of run.
- **Files to include / exclude** take comma-separated globs: `src`, `*.ts`, `src/**/*.test.ts`. A pattern without `/` matches a file or folder name anywhere.
- Click a result to open the file at that line. Searches skip `.git`, `node_modules`, build output, binary files, symlinks and files over 2 MiB, and stop after 500 results or 10 seconds, saying so.
- Without a Runtime, Search covers the files already loaded in the editor and says so.

### Replace in files

Click **↔** next to the search box to show **Replace with**. **Replace all** replaces in every listed file; the **↔** beside a file replaces in just that file. Boson first counts the matches and asks you to confirm, because the change is written to disk and is not part of the editor's undo history.

- In plain mode the replacement is inserted exactly as typed. With **.\*** on, `$1`, `$&` and named groups work.
- Files with unsaved changes in an editor are skipped (and counted in the message), so unsaved work is never overwritten. A file that changes on disk while the replace runs is skipped too.
- Open files without unsaved changes reload automatically after the replace.

## Graphs

Boson draws relationships as a **Blueprint-style graph**: nodes sit in titled, colored groups on a large canvas, and curved wires connect output pins to input pins. Flow runs left to right, so callers and dependents are on the left and the things they use are on the right. Tall groups wrap sideways, so a graph grows in both directions instead of becoming one long column or a cluster.

### Workspace graph

The **workspace graph** is a pinned tab at the start of every editor group's tab bar. It cannot be closed, and it is what you see when a workspace opens: Boson never opens a file on its own, so you choose where to start from the graph or the Explorer. Switching to a file tab and back keeps the folder you were looking at, and the tab is restored as selected after a restart if it was the one you left active.

- Each node is a folder or file in the current folder, placed in an inferred group such as **Entry points**, **Folders**, **Modules**, **Tests**, **Documentation**, **Configuration & tooling** or **Styles & assets**.
- Wires are import dependencies (importer → imported). Wires between files in different folders are rolled up onto the folder nodes, so you can see how areas of the project depend on each other.
- Source files list their top-level functions, classes and other symbols as rows with pins.
- **Click a folder** to open it as the next graph (use the breadcrumb to go back up). **Click a file** to open it in the editor. **Click a function** to open the file at that function.
- Hover a node to light up everything upstream (amber wires) and downstream (colored wires) of it.
- **Right-click** any node, a symbol row or empty canvas for quick actions (or focus a node and press the Menu key or Shift+F10). Files and folders offer Open, Copy path, New file/folder here, Reveal in Explorer, Rename, Move to… and Delete; symbols offer Go to symbol and Copy name; empty canvas offers Up one folder, Back, Forward, Refresh graph, Fit to view and Reset view. Every graph also has *Center on this node* and *Turn flow tracing on/off*. Use the arrow keys, Enter and Escape inside the menu.
- Without a Workspace Runtime the graph still shows the folder and file structure; connect a Runtime to add the dependency wires. **Refresh graph** re-reads the relationships.

The Runtime reads TypeScript and JavaScript through the TypeScript parser, and Python and C/C++ with a lighter line-based reader. It resolves relative imports, `index` files, `.js` → `.ts` mappings and packages in the same monorepo. Results are cached per file, so a refresh only re-reads files that changed. If a limit is reached the graph says it is partial.

| Limit | Default | Environment variable |
|---|---|---|
| Files analyzed | 6,000 | `BOSON_GRAPH_MAX_FILES` |
| Largest file analyzed | 512 KiB | `BOSON_GRAPH_MAX_FILE_BYTES` |
| Analysis time budget | 20 s | `BOSON_GRAPH_MAX_MS` |

### Code graph

**Preview Code Graph** in a source file's breadcrumb shows that file's structure and calls. Boson infers the groups for you: one group per class, namespace or enum with its members, **Local imports** and **Package imports**, **Types**, **Constants & variables**, a **Flow** group for each entry point (a function nothing else calls) with everything it calls, and **Shared helpers** used by more than one flow. Click a node to jump to it in the editor, or a group title to jump to the class or entry point.

A component or function with many nested functions (six or more, typical of a React component) is treated as a **Scope**: it gets its own group, and each nested function seeds its own flow instead of everything merging into one giant flow. Functions that are called from many places ("hubs") get faint wires so they don't bury the real structure.

**Trace flow** (toolbar) controls hover tracing. When it is on, hovering a node lights up everything upstream (amber) and downstream (colored) and dims the rest. When it is off, hovering does nothing and any highlight is cleared.

The graph is only called partial when symbols or connections were really left out, and the message says how many and which setting controls it. Defaults handle large files (thousands of symbols, up to 2 MiB); raise `BOSON_CODE_GRAPH_MAX_NODES` or `BOSON_CODE_GRAPH_MAX_EDGES` for more.

### Navigating a graph

- Scroll to zoom, drag empty space to pan, or click the minimap to jump. **Fit** shows the whole graph and **Reset** returns to a readable zoom.
- Zoomed out, nodes drop their details and then show only group titles, so a large graph stays readable and fast. Only what is on screen is drawn.
- Nodes are keyboard-focusable: Tab to a node and press Enter to open it.

### Back and forward (mouse thumb buttons)

The **Back** and **Forward** thumb buttons on your mouse work like a web browser's, in every view that has a history. A click goes to the innermost view under the pointer; if that view has nowhere to go, it passes to the next one outward.

| View | What Back / Forward do |
|---|---|
| Graph canvas | Undo or redo the last jump: Overview, Reset, or a click on the minimap (panning and zooming are not history) |
| Workspace graph | Move through the folders you visited; the **←** and **→** buttons in its toolbar do the same |
| Documentation | Move through the pages you opened, including links inside web pages; **←** and **→** are next to the search box |
| Browser tab | Move through the page history (on desktop, also while the pointer is over the page itself) |
| Editor | Return to previously opened files, the workspace graph tab and places you jumped to from a graph, then forward again |

Mouse buttons never reload or leave Boson itself. Page history inside a cross-origin website in the browser-only build cannot be reached by the mouse buttons; use the toolbar buttons there.

## Previews

- **Markdown:** use **Preview** on a source tab. Supports headings, emphasis, links, local and HTTP(S) images, code fences, tables, task lists and quotations, and preserves source line breaks. Links to `#headings` scroll within the preview; heading anchors are stable and unique, even for repeated headings.
- **SVG:** small SVG files open as editable XML. Use **Preview SVG** / **Edit SVG** in the breadcrumb to switch. Previews render in a sandboxed frame and are never injected as inline markup. Large SVGs are preview-only.
- **Assets:** images, PDF, audio and video have dedicated previews (up to 24 MiB each).

## Building, testing and running

**Build**, **Test** and **Run** submit a task to the Workspace Runtime and stream output into the task panel. A running task can be stopped from the UI. The task timeout is configurable, with a hard maximum of one hour.

| Detected from | Tasks |
|---|---|
| `package.json` scripts | npm, pnpm, Yarn, Bun |
| `Cargo.toml` | Cargo |
| Root `CMakeLists.txt` | CMake |
| `pom.xml`, Gradle files | Maven, Gradle (uses the wrapper if present) |
| Python, Go and .NET projects | Conventional commands |
| Single `.py`, `.js`, `.go` files | Run directly |
| Simple `.c` / `.cpp` files | Run directly, if a compiler is installed |

Task planning looks at conventional root-level manifests and scripts. You cannot yet edit commands or define custom tasks.

For multi-module workspaces, Boson also computes a dependency-ordered **build plan**; see [Projects, modules and build plans](projects-and-modules.md#build-plans).

The **Toolchains** view lists installed, allow-listed developer executables on the Runtime host and shows project hints separately. It is discovery only.

The Runtime progress indicator appears only while connecting, indexing, discovering toolchains or running a task, not simply because a Runtime is connected.

## Symbols, Outline and references

- **Go to Symbol in File** (`Ctrl/Cmd+Shift+O`) lists the classes, functions, methods and other declarations of the open file and jumps to the one you pick. The **Outline** section in the inspector lists the same symbols as a tree; click one to jump to it.
- Symbols come from the Runtime's code graph for TypeScript and JavaScript files, and from the language server (`textDocument/documentSymbol`) for other languages while that server runs. Without either, the Outline says so.
- **Find All References in Workspace (text)** (`Shift+Alt+F12`, or the editor's context menu) searches the whole workspace for the word under the cursor, whole word and case-sensitive, in the Search view. It is a text search, not a semantic one: it also finds unrelated identifiers with the same name, and it cannot tell a definition from a use. Monaco's own `Shift+F12` still gives semantic references inside open TypeScript and JavaScript files.

## Debugging (Node.js)

Open a `.js`, `.mjs` or `.cjs` file and press **F5** (or run **Start Debugging (Node.js)**). Boson starts `node` with the V8 inspector on the Runtime host, attaches to it, and opens the **Debug** panel.

- Click the gutter beside a line number, or press **F9**, to set a breakpoint. Breakpoints can be changed while the program runs.
- **Continue** (`F5`), **Step over** (`F10`), **Step into** (`F11`), **Step out** (`Shift+F11`), **Pause** and **Stop** (`Shift+F5`) are in the panel's toolbar.
- When paused, the panel shows the call stack, the variables of the selected frame (expand objects), and a console where you can evaluate expressions in the selected frame. Program output appears in the same console.
- The program runs with the Runtime's scrubbed environment, inside the workspace, with at most four debug sessions at a time. It starts the `node` found on the Runtime host's `PATH`, so Node.js must be installed there; the Runtime itself needs Node.js 22 or newer (the desktop app's Runtime already is).

Other languages need a debug adapter (the Debug Adapter Protocol) and are not available yet. Breakpoints are kept for the current window only; they are not saved between sessions.

## Source Control (Git)
## Source Control (Git)

The Source Control view talks to the Runtime, which runs `git` in the workspace (90-second timeout per operation). It supports:

- Status and diff for changed files
- Initialize a repository in a folder that is not one yet
- Stage (selected or all) and unstage
- Commit
- Create and switch branches
- Add a remote, fetch, pull, push and publish a branch
- **Compare Active File with Git History…** opens a side-by-side comparison of the open file with its last commit (HEAD) or with any of its last 50 commits.
- **Toggle Git Blame for Current Line** shows who last changed the line under the cursor, when, and the commit summary. Blame is shown for saved files only (the line numbers of a modified file would no longer match).

Git must be installed on the Runtime host. It needs a writable filesystem workspace.

## Packages (npm)

The Packages view finds npm modules in the workspace (every folder with a `package.json`, up to a scan limit), searches the npm registry and installs a package into a chosen module as a dependency or dev dependency. Installs run as a normal task and stream into the task panel; Boson asks for confirmation first because install scripts from a package and its dependencies can execute code. Only **npm** is supported for installation.

## Browser tab

Boson can open web pages in an editor tab. How it works depends on where you run it:

| | Desktop | Browser app |
|---|---|---|
| Mechanism | Native Electron `<webview>` (sandboxed, isolated from the app) | An iframe, with scripts enabled, loading pages through a Runtime browser session |
| Local servers (`localhost`) | Yes | **No**, the Runtime proxy refuses local and private hosts |
| Public sites | Yes | Yes (HTTP/HTTPS only, no embedded credentials) |

The address bar behaves like a browser's: host-like input gets `https://` and anything else becomes a web search. On desktop, mouse back/forward thumb buttons navigate the tab's history.

Browser sessions on the Runtime keep cookies on disk so logins survive restarts; see [Security](../reference/security.md#browser-data) for where and for how long.

## Terminal

The integrated terminal is a real operating-system shell. Click **+** in the terminal tab strip to open another independent shell (up to 8); each tab has its own Runtime session, and closing a tab ends its shell. Terminals keep running while the panel is collapsed or you look at Problems or Output. Shell selection and the pipe-mode fallback are in [Workspace Runtime → Terminal](../reference/workspace-runtime.md#terminal).

## Indexing limits

Large workspaces are indexed by **metadata first**. Opening a folder lists every readable text file (path, size and modification time) in the Explorer. Only a small, byte-budgeted subset also arrives with its content; the other files are read from the Runtime the first time you open them. Common dependency and build folders, symlinks and unsupported binary formats are skipped.

| Limit | Desktop folder and remote workspace | Environment variable |
|---|---|---|
| Files listed (metadata) | 100,000 | `BOSON_INDEX_MAX_FILES` |
| Files sent with content | 2,000 | `BOSON_INDEX_INLINE_FILES` |
| Largest file sent with content | 256 KiB | `BOSON_INDEX_INLINE_FILE_BYTES` |
| Total content sent with the index | 16 MiB | `BOSON_INDEX_INLINE_TOTAL_BYTES` |
| Largest file the editor opens on demand | 8 MiB | `BOSON_MAX_OPEN_FILE_BYTES` |
| Directory depth | 40 | `BOSON_INDEX_MAX_DEPTH` |
| Directories | 50,000 | `BOSON_INDEX_MAX_DIRECTORIES` |
| Entries visited | 400,000 | `BOSON_INDEX_MAX_ENTRIES` |
| Previewable assets | 5,000 (24 MiB each) | `BOSON_INDEX_MAX_ASSETS`, `BOSON_MAX_ASSET_BYTES` |

Values are clamped to safe ranges; the desktop app reads the same variables. A large file never marks the workspace as truncated: it is listed and opens on demand, up to the editor limit. Beyond that limit Boson tells you the file is too large to open in the editor. The workspace is marked truncated only when a hard cap (files, entries, depth, directories) is reached.

**Browser import** (a folder opened through the browser's file picker without a Runtime) still reads file contents up front and keeps its older limits: 250 files, 512 KiB per file, 16 MiB in total, depth 9. Connect a Runtime for large workspaces. Imports show progress and can be cancelled without losing the open workspace.

Files that are listed but not yet loaded are not searched by the in-editor features that work on loaded text (for example, open-buffer validation) until you open them; Runtime-side search and the code graph read from disk.

## Unsaved work, Zen mode and shortcuts

- **Hot exit.** Unsaved edits are backed up on this device a moment after you stop typing (up to 60 files, 3 MB in total). After a crash or an unexpected close, Boson offers to restore them when you open the same workspace again. Nothing is restored without asking, and a file you have already started editing is never overwritten. The backup is plain text in the browser's local storage on this device.
- **Zen mode** (`Ctrl/Cmd+K Z`, or **Toggle Zen Mode**) hides everything except the editor. Press `Escape` twice to leave it; your previous layout comes back.
- **Keyboard Shortcuts** (command palette) lists every command with its shortcut. Click a shortcut, press a new combination, and the command is rebound. Combinations already in use are refused, and plain letters cannot be assigned. Changes are stored on this device. Shortcuts for editor-internal Monaco commands are not listed.
- **Welcome** is shown when Boson starts until you turn off *Show this page when Boson starts*; **Show Welcome Page** reopens it. To turn it back on, use **Settings → Workbench → Startup → Show the Welcome page when Boson starts**.
## Getting help

Choose **Help → Documentation** (or run **Documentation** from the command palette) to read these guides without leaving Boson. See [Documentation viewer](documentation-viewer.md).

## Current limitations

- **Debugging:** Node.js (JavaScript) only, with no launch configuration file. Other languages need Debug Adapter Protocol adapters, which are not provided yet.
- **Multi-root workspaces:** a window opens one folder. Open another window for another folder.
- **Language servers:** diagnostics, completion, hover, go to definition and highlights are connected. Other LSP features are permitted by the Runtime but not yet exposed in the editor.
- **Tasks:** detection covers conventional root-level manifests; there is no custom task editor.
- **Source control:** common Git operations, file comparison with history and blame for the current line. Conflict resolution, rebasing and repository-wide history need the terminal.
- **Packages:** installation supports npm only.
- **Snapshots:** read-only; they cannot run tasks, use Git or be validated.
- **Terminal:** runs with the Runtime user's full permissions and is not a sandbox. Open only trusted workspaces.
