# Troubleshooting

Find your symptom, then follow the fix. Sections are grouped by area.

## Contents

- [Connecting to a Runtime](#connecting-to-a-runtime)
- [Opening and syncing workspaces](#opening-and-syncing-workspaces)
- [Editor and diagnostics](#editor-and-diagnostics)
- [Terminal and native builds](#terminal-and-native-builds)
- [Browser tab](#browser-tab)
- [Git, packages and build plans](#git-packages-and-build-plans)
- [Packaged desktop app](#packaged-desktop-app)
- [Debugging](#debugging)
- [Performance and background CPU](#performance-and-background-cpu)
- [Discord Rich Presence](#discord-rich-presence)
- [Documentation viewer](#documentation-viewer)

## Connecting to a Runtime

### "Failed to fetch" or `403 ORIGIN_NOT_ALLOWED`

The Runtime does not list the browser app's origin. Add the exact origin to `BOSON_ALLOWED_ORIGINS` (for example `https://ide.example.test`) and restart the Runtime. Development defaults already allow `localhost`, `127.0.0.1` and `[::1]` on any port.

Also check that:

- The Runtime URL is reachable **from the browser's machine**. `localhost` there means the browser's own computer.
- If Boson is served over HTTPS, the Runtime is too (or reached through an encrypted tunnel). Browsers block insecure requests from secure pages.
- Opening `<Runtime base URL>/v1/workspace/check` in the browser shows a JSON response.

### Test connection reports an invalid token

Use the token the Runtime printed at startup, or set a stable one with `BOSON_RUNTIME_TOKEN`. A random token is generated on every start when none is set.

### Behind a reverse proxy, browser-tab pages point at the wrong host

Set `BOSON_RUNTIME_PUBLIC_ORIGIN` to the Runtime's public URL. Boson ignores `X-Forwarded-*` headers unless `BOSON_TRUST_PROXY_HEADERS=true`.

## Opening and syncing workspaces

### "Choose a folder" shows the Runtime host's folders, not mine

That is intended in the browser app: workspaces live on the Runtime host. To work with files from your own computer, run the desktop app, or use **Import a folder / Import files** to upload them into a workspace that is already open on the Runtime host.

### The folder browser says to configure a connection first

Browsing needs a working Runtime connection. Enter the URL and token in **Settings → Workspace Runtime** and run **Test connection** first.

### "Workspace root is outside the Workspace Runtime allow-list"

The folder is not one of the Runtime's allowed roots. Opening it from Boson normally registers it automatically. If you see this error anyway, add the folder, or a parent, to `BOSON_ALLOWED_WORKSPACE_ROOTS` (`;` separated on Windows, `:` elsewhere) and restart the Runtime.

### Build/Test/Run, Git or validation is unavailable

You are probably in a **read-only snapshot** (a banner says so). Snapshots cannot run tasks, use Git or be validated. Choose **Open writable folder…** to open a real workspace.

### Files I imported are missing or old on the Runtime host

Imports upload into the open remote workspace and overwrite files at the same relative path. Check that you imported *after* opening the workspace, and that nothing was skipped by the [indexing limits](workspaces.md#indexing-limits). Imports never delete host files.

### A workspace opens with files missing

Indexing skips dependency and build folders such as `node_modules`. Every readable text file is listed and opens on demand (see [Indexing limits](workspaces.md#indexing-limits)); the workspace is flagged as truncated only when a hard cap is reached, and the output panel names which. Raise the `BOSON_INDEX_*` limits on the Runtime, or open the specific module you need. A file over 8 MiB is listed but cannot be opened in the editor (`BOSON_MAX_OPEN_FILE_BYTES`).

### Changes made outside Boson don't appear

Live watching covers at most 512 directories and skips dependency/build folders. In a very large workspace, right-click the workspace root and choose **Refresh**.

## Editor and diagnostics

### A source file shows no language diagnostics

Boson uses built-in validators for TypeScript/JavaScript, Python, JSON/JSONC and CMake, and Runtime-host language servers for other languages.

1. Check **Problems** for a language-server start error.
2. Confirm the server executable is installed on the **Runtime host** and on its `PATH`. Boson only starts servers it can find there. You can query what the Runtime sees at `GET /v1/lsp/servers`.
3. For a non-default name or location, set the matching `BOSON_LSP_<NAME>_COMMAND` variable and restart the Runtime (names are listed in [Language support](language-support.md#supported-servers); the C / C++ adapter is `BOSON_LSP_CPP_COMMAND`).
4. Make sure the project's SDKs and dependencies are installed, because semantic diagnostics depend on them.
5. Run **Validate workspace** for the built-in checks.

Boson never installs language servers for you.

### Problems show "results are out of date"

The workspace changed since the last validation. Run **Validate workspace** again.

### CMake diagnostics disagree with the real build

The editor check is a lightweight parser, not CMake. Use Build/Test, which run the installed CMake.

### SVG appears blank

Open the file with **Edit SVG** and check the source for errors. Very large SVG files are preview-only.

### The editor won't split further

The layout holds at most 4 columns × 4 rows (16 groups). Close a split first.

## Terminal and native builds

### The terminal says "pipe compatibility"

`node-pty` has not been built for your Electron version. Basic commands work, but full-screen programs and job control do not. Run `npm run rebuild:native` once (or `npm run dev:desktop:pty`). On Windows you also need the Spectre-mitigated MSVC libraries; see [Native PTY rebuild](../development/building.md#native-pty-rebuild).

### `npm run rebuild:native` fails with MSB8040 on Windows

Install the Spectre-mitigated libraries for your MSVC toolset and architecture via Visual Studio Installer → Modify → Individual components, then rerun the rebuild.

## Browser tab

### `localhost` pages won't open in the browser app

The browser app loads pages through the Runtime's proxy, which deliberately refuses `localhost`, `.local`/`.internal` names and private network addresses. Use the desktop app, whose tab uses a native webview that can open local servers, or open the page in a separate browser window.

### A website loads only partially

Large API responses can exceed the header budget. Raise `BOSON_BROWSER_MAX_HEADER_BYTES` (up to 262144), and temporarily set `BOSON_BROWSER_DIAGNOSTICS=true` to log upstream response status without query strings or cookies.

### I want to clear saved logins from the browser tab

Delete the browser profile directory on the Runtime host: `~/.boson/browser-profile`, or `BOSON_BROWSER_PROFILE_DIR` if you set it. There is no in-app command for this yet. See [Security → Browser data](../reference/security.md#browser-data).

## Git, packages and build plans

### Source Control says the folder is not a Git repository

Initialize the repository from the Source Control view, or run `git init` in the terminal. Git must be installed on the Runtime host.

### The build plan reports a dependency cycle

Boson refuses to order modules that depend on each other in a loop. The error lists the cycle. Break it in the manifests that declare the dependencies, then refresh.

### Installing a package fails or isn't offered

Installation supports npm only, into a module that has a `package.json`. Make sure `npm` is available to the Runtime.

## Packaged desktop app

### The packaged app starts but the Runtime is offline

Check the Runtime's startup log for `ERR_MODULE_NOT_FOUND`. Every Runtime source file must be listed under `extraResources` in `electron-builder.yml` and checked by `verify-runtime-package.mjs`. See [Building → How the Runtime is packaged](../development/building.md#how-the-runtime-is-packaged).

## Debugging

### "Could not start node" or "This Node.js version has no built-in WebSocket client"

The debuggee is started with the `node` executable on the **Runtime host's** `PATH`, so Node.js must be installed there. The Runtime process also needs Node.js 22 or newer, because it attaches with Node's built-in WebSocket client; the desktop app's Runtime already qualifies, a standalone Runtime may not.

### Breakpoints are shown but the program never stops on them

Breakpoints bind to the file by its path inside the workspace. A program that runs transpiled output (for example `dist/index.js` from TypeScript) does not stop on breakpoints in the `.ts` source, because source maps are not applied yet. Debug the JavaScript file that actually runs.

### "A debug session is already running" or "allows 4 debug sessions"

Stop the current session in the Debug panel (`Shift+F5`). The Runtime keeps at most four at a time.

## Performance and background CPU

### Boson uses CPU while minimized or idle

Run **Show Process Metrics (CPU and memory)** from the command palette (desktop app). It lists every Boson process with its CPU and memory in the Output panel, so you can see which one is busy: the renderer (the editor UI), the GPU process, the Workspace Runtime (a utility process), or a browser-tab guest. Run it twice a few seconds apart; the first reading covers the time since the last sample.

- A busy **Tab** or **webview** process is a browser tab (a playing video or a heavy page). Close it or switch away.
- A busy **Workspace Runtime** usually means a language server, a watcher on a very large folder, or a running task. Large generated folders such as `node_modules`, `.git` and `dist` are skipped by the watchers.
- Decorative animations pause automatically while the window is hidden. Language servers shut down after 10 idle minutes.

### Boson appears as "Electron" in the task manager

Packaged builds are named **Boson**, with the Boson icon (executable `boson`). A development run uses the stock Electron executable, so Windows labels the process group "Electron". `npm run dev` therefore starts through `apps/desktop/scripts/dev-branded.mjs`, which on Windows makes a one-time copy of Electron under `node_modules/.cache/boson-dev`, stamps it with Boson's name and icon, and runs the dev server with it. It needs the `rcedit` dev dependency (run `npm install`); if anything fails it prints why and starts normally. Use `npm run dev:plain` to skip it. Delete `node_modules/.cache/boson-dev` to rebuild the copy.

### The portable or installed app shows the Electron icon

The icon comes from `apps/desktop/resources/icon.png` (set as `icon` for Windows, macOS and Linux in `electron-builder.yml`). Rebuild the package after changing it. Windows caches icons: if an old `.exe` still shows the old one, rename the file or clear the icon cache.

### The Update button

On the desktop app, Boson looks at the latest GitHub release a few seconds after start and every six hours. The **Update** button appears only when that release is newer than the installed version; opening it shows the release page. With no repository configured, no network, or no newer release, nothing is shown. The repository is the `repository` field of `apps/desktop/package.json` (set to `Boson-IDE/boson`), the `BOSON_UPDATE_REPOSITORY` environment variable, or **Settings → Workbench → Updates**. Only stable (not draft or pre-release) releases count.

### Build, Run or the terminal say "Restricted Mode"

The folder is not trusted, so the Runtime refuses to run its programs (`WORKSPACE_UNTRUSTED`). Click the **Restricted Mode** chip in the header, or use **Help → Manage Workspace Trust**, and choose **Trust this folder** if you know where the code came from. Language servers also stay off until then, so a file can show no diagnostics in Restricted Mode. Trust is stored by the Runtime, so it follows the folder, not the window.

### Send Feedback

**Help → Send Feedback** (also in the command palette) opens the **new issue** page of the same GitHub repository the Update button uses, with the Boson version, platform and Electron version already filled in. Nothing is sent from the app itself: you review and submit the issue on GitHub. If no repository is configured (see above) Boson says so instead of opening a page. Please check the existing issues first, and never paste tokens or private code into a report.

### The window opens in the wrong place or size

Boson remembers the window's position, size and maximized state in `window-state.json` in the app's user-data folder, and restores them on the next start. The restored size is the un-maximized one, so un-maximizing later returns to where you left it. If a monitor was unplugged, a window that would be off-screen is placed by the system instead and keeps its size. To reset it, close Boson and delete `window-state.json`.

## Discord Rich Presence

Discord lists "Playing ..." for programs on a list that Discord maintains, so Boson cannot add itself. Rich Presence is the supported route and is **off by default**:

1. At [discord.com/developers/applications](https://discord.com/developers/applications) create an application named **Boson** (that name is what your profile shows). Optionally upload an art asset called `boson` under Rich Presence.
2. Copy the **Application ID**.
3. In Boson open **Settings → Workbench → Discord Rich Presence**, turn it on and paste the ID. Choose whether the workspace and file names are shared.

Also check in Discord itself: **User Settings → Activity Privacy → Share your detected activities with others** (or *Display current activity as a status message*) must be on, otherwise Discord hides Rich Presence. Boson only appears once an Application ID is entered; without one there is nothing for Discord to show. The status line under the setting says which step failed: *Connecting…*, *Discord refused this Application ID* (the ID is wrong), or *Discord is not running or not reachable* (Discord is closed, or it is the Flatpak/Snap build, whose socket Boson also tries).

Discord must be running on the same computer. If it is not, Boson retries every 30 seconds. Nothing is sent to anyone but your local Discord client.

## Documentation viewer

### "Connect a Workspace Runtime to read Boson documentation"

The viewer reads documentation through the Runtime. In the browser app, connect one first in **Settings → Workspace Runtime** (URL and token, then **Test connection**). The desktop app connects automatically.

### "The Boson documentation directory is unavailable" or an empty list

The Runtime cannot find its `docs/` directory. For a standalone Runtime set `BOSON_DOCS_ROOT` to the folder. In a packaged desktop app this means the installer is missing its `docs` resources; see [Building](../development/building.md#how-the-runtime-is-packaged).

### A web document says "Documentation session expired"

Sessions last 15 minutes after their last use. Close and reopen **Help → Documentation**.

### A web document's script or style is blocked

Web documents run under a strict Content Security Policy: they can load only their own sibling files and `data:`/`blob:` resources, never remote scripts, styles or fonts. See [Authoring documents](documentation-viewer.md#rules-for-web-documents).
