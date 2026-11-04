# Workspace Runtime

The Workspace Runtime is a local Node.js service. It executes builds, tests and runs, hosts terminal sessions, serves workspace files, watches for changes, runs Git, discovers modules, validates code, and hosts language servers. The renderer and shared UI never start project processes themselves; only the Runtime does.

## Which setup do I need?

| You are using | Runtime setup |
|---|---|
| **Desktop app** | None. The Electron main process starts, health-checks, restarts and stops it for you. |
| **Browser app** | Run a Runtime on the machine that holds your projects (`npm run dev:runtime`) and connect to it in **Settings → Workspace Runtime**. |
| **Browser app, remote host** | Same, plus a token, an exact origin allow-list and HTTPS or an encrypted tunnel. |

## Desktop (automatic)

The Runtime runs as an Electron utility process on a free loopback port with a per-launch bearer token, and receives the chosen folder through an authenticated registration request. It is restarted if it exits unexpectedly and stopped when Electron quits. It never broadens access to your home directory.

Runtime API calls and event streams go through the preload IPC bridge and the Electron main process, which adds the bearer token. The renderer does not call the loopback Runtime directly, which avoids CORS and loopback-fetch restrictions. The bridge accepts only the managed `127.0.0.1` port and an explicit allow-list of authenticated task, terminal, workspace-file-change and language-server event routes. It does not proxy arbitrary event-stream URLs.

## Standalone (browser development and remote hosts)

```bash
npm run dev:runtime
```

By default the Runtime listens on `127.0.0.1:4178`, uses the current working directory as its workspace, and prints a generated bearer token at startup. In Boson open **Settings → Workspace Runtime**, enter the URL and token, and click **Test connection**.

**Test connection** checks that:

1. The unauthenticated health endpoint responds.
2. The bearer token is accepted by the authenticated status endpoint.
3. If you entered a Runtime-visible workspace path, that directory exists on the Runtime host.

### Choosing workspace roots

```bash
BOSON_WORKSPACE_ROOT=/path/to/project \
BOSON_ALLOWED_WORKSPACE_ROOTS=/path/to/project:/path/to/another/project \
npm run dev:runtime
```

Separate `BOSON_ALLOWED_WORKSPACE_ROOTS` entries with `;` on Windows and `:` on macOS and Linux. The default allow-list contains only `BOSON_WORKSPACE_ROOT` (or the working directory). A workspace must be registered before tasks, file operations or language servers can use it. Registration is automatic when you open a folder in Boson.

> **Allowed roots are enforced.** By default (`BOSON_ALLOW_DYNAMIC_ROOTS` unset) a client can open only folders inside `BOSON_WORKSPACE_ROOT` / `BOSON_ALLOWED_WORKSPACE_ROOTS`; `POST /v1/workspaces/register`, the folder browser and `POST /v1/workspaces/check` refuse everything else with `403 ROOT_NOT_ALLOWED` or `PATH_NOT_ALLOWED`. The desktop app starts its own Runtime with `BOSON_ALLOW_DYNAMIC_ROOTS=true` so you can open any folder you pick; folders such as your home directory, a drive root or a system location need an explicit confirmation. Roots are not a sandbox for code you run: build scripts and the terminal still execute as your account. See [Security](security.md).

### Exposing the Runtime to a hosted web app

```bash
BOSON_RUNTIME_HOST=0.0.0.0 \
BOSON_ALLOWED_ORIGINS=https://ide.example.test \
BOSON_RUNTIME_TOKEN=<long-random-secret> \
npm run dev:runtime
```

- Put the Runtime behind **HTTPS or an encrypted tunnel**. Browsers block insecure HTTP requests from HTTPS pages.
- Use **exact origins** in `BOSON_ALLOWED_ORIGINS`; never the development wildcards.
- `localhost` in the browser always means the browser's own computer, not the Runtime host.
- Requests from unlisted origins are rejected with `403 ORIGIN_NOT_ALLOWED`, so the UI can name the missing origin instead of a generic `Failed to fetch`.
- Behind a reverse proxy, set `BOSON_RUNTIME_PUBLIC_ORIGIN` to the Runtime's public URL. Forwarded headers are ignored unless you also set `BOSON_TRUST_PROXY_HEADERS=true`.
- The Runtime **refuses to start** on a non-loopback address without an explicit `BOSON_ALLOWED_ORIGINS` list (no `*`, no `null`), an `https://` `BOSON_RUNTIME_PUBLIC_ORIGIN`, and a token of at least 32 characters.
- Requests whose `Host` header is not loopback, the public origin's host or listed in `BOSON_ALLOWED_HOSTS` get `421 HOST_NOT_ALLOWED` (DNS-rebinding defence). Repeated bad tokens from one address are throttled with `429`.

## Configuration

`apps/runtime/.env.example` is a ready-made template.

### Core

| Variable | Default | Purpose |
|---|---|---|
| `BOSON_RUNTIME_HOST` | `127.0.0.1` | Interface to bind |
| `BOSON_RUNTIME_PORT` | `4178` | Port |
| `BOSON_RUNTIME_TOKEN` | random per start | Bearer token. Set a stable value for standalone or remote use |
| `BOSON_WORKSPACE_ROOT` | working directory | Default project root |
| `BOSON_ALLOWED_WORKSPACE_ROOTS` | the root above | Additional roots allowed for tasks and file access |
| `BOSON_ALLOW_DYNAMIC_ROOTS` | `false` | `true` lets a token holder register further folders at run time (set by the desktop app) |
| `BOSON_ALLOWED_ORIGINS` | loopback on any port | Comma-separated CORS allow-list. Required (exact origins) for a non-loopback bind |
| `BOSON_ALLOWED_HOSTS` | none | Extra accepted `Host` values, comma-separated |
| `BOSON_TOKEN_FILE`, `BOSON_PRINT_TOKEN` | `~/.boson/runtime-token`, `false` | A generated token is written to this file (mode 600) instead of the log; `BOSON_PRINT_TOKEN=true` prints it |
| `BOSON_INDEX_*`, `BOSON_MAX_OPEN_FILE_BYTES` | see [Indexing limits](../guide/workspaces.md#indexing-limits) | Workspace index and editor file-size limits |
| `BOSON_GRAPH_MAX_FILES`, `BOSON_GRAPH_MAX_FILE_BYTES`, `BOSON_GRAPH_MAX_MS` | 6000, 512 KiB, 20 s | Limits for the workspace graph analysis (see [Graphs](../guide/workspaces.md#graphs)) |
| `BOSON_CODE_GRAPH_MAX_NODES`, `_MAX_EDGES`, `_MAX_AST_NODES`, `_MAX_SOURCE_BYTES` | 4000, 12000, 1500000, 2 MiB | Limits for the per-file [code graph](../guide/workspaces.md#code-graph). Clamped to safe ranges |
| `BOSON_RUNTIME_TASK_TIMEOUT_MS` | `1800000` (30 min) | Task timeout, minimum 5000, capped at one hour |
| `BOSON_DOCS_ROOT` | `docs/` next to the repository (the desktop app sets it to its bundled `docs/`) | Directory served by the [documentation viewer](../guide/documentation-viewer.md) |
| `BOSON_TERMINAL_SHELL` | PowerShell/cmd on Windows; `$SHELL` or `/bin/bash` elsewhere | Shell for the terminal, e.g. `pwsh.exe`, `bash`, `zsh` |

### Reverse proxy and browser sessions

| Variable | Default | Purpose |
|---|---|---|
| `BOSON_RUNTIME_PUBLIC_ORIGIN` | derived from the request | The Runtime's public origin, used when rewriting browser-session URLs. The desktop app sets it for you |
| `BOSON_TRUST_PROXY_HEADERS` | off | Set `true` to trust `X-Forwarded-Proto` and `X-Forwarded-Host`. Leave off unless a proxy you control sets them |
| `BOSON_BROWSER_PROFILE_DIR` | `~/.boson/browser-profile` | Where browser-session cookies and sessions are stored. Falls back to `BOSON_RUNTIME_DATA_DIR` |
| `BOSON_RUNTIME_DATA_DIR` | none | Alternative storage directory for browser data |
| `BOSON_BROWSER_MAX_HEADER_BYTES` | `262144` | Browser upstream response-header budget (16 KiB to 256 KiB) |
| `BOSON_BROWSER_DIAGNOSTICS` | off | Set `true` to log upstream response status (no query strings or cookies) when a site loads only partially |
| `BOSON_RUNTIME_VERBOSE` | off | Set `true` (or start the Runtime with `--verbose` / `-v`) to print informational logs such as the default workspace, allowed roots and root grants. Warnings and errors always print. The desktop app accepts `--runtime-verbose` |

### Language servers

Each adapter has an optional `BOSON_LSP_<NAME>_COMMAND` variable that replaces its executable. The full list of variable names, defaults and extensions is in [Language support](../guide/language-support.md#supported-servers), for example:

```bash
BOSON_LSP_PYTHON_COMMAND=/opt/pyright/bin/pyright-langserver
BOSON_LSP_CPP_COMMAND=clangd-18      # the C / C++ adapter uses CPP
```

### Set by Boson for child processes

You normally don't set these. `BOSON_RUNTIME_MODULE_ROOT` is set by the desktop app so the packaged Runtime can load its dependencies from `app.asar`. Language server processes are started with `BOSON_LANGUAGE_SERVER=1`, and terminals with `BOSON_TERMINAL=1`. The Runtime removes its own bearer token and Electron management variables from the environment of **tasks and terminals**. Other helper processes, such as Git and language servers, currently inherit the Runtime's environment.

### Legacy names

`BOSON_AGENT_TOKEN`, `BOSON_AGENT_PORT`, `BOSON_AGENT_HOST` and `BOSON_TASK_TIMEOUT_MS` are still accepted, and `npm run dev:agent` remains an alias for `npm run dev:runtime`.

## HTTP API

All endpoints except those marked "No" require `Authorization: Bearer <token>`. Long-running output uses server-sent events (SSE).

### Health and connection

| Endpoint | Auth | Purpose |
|---|---|---|
| `GET /health` | No | Liveness, name and version |
| `GET /v1/workspace/check` (also `/v1/workspaces/check`) | No | Liveness probe that reports whether the request was authenticated, without exposing workspace data. Handy as a manual browser check |
| `POST /v1/workspace/check` (also `/v1/workspaces/check`) | Yes | Body `{ "root": "..." }`; confirms the directory exists on the Runtime host |
| `GET /v1/auth/check` | Yes | Validates the token |
| `GET /v1/status` | Yes | Version, Node version, platform, capabilities and limits |
| `POST /v1/shutdown` | Yes | Stops the Runtime (used by the desktop app) |

### Workspaces and files

| Endpoint | Purpose |
|---|---|
| `GET /v1/filesystem/roots`, `GET /v1/filesystem/browse?path=` | List drive/root entries and the subfolders of **any directory on the host**, for the in-app folder browser (names only, up to 400 folders, skipping dependency and build folders) |
| `POST /v1/workspaces/register` | Authorize a folder. Refused (`ROOT_NOT_ALLOWED`) outside the allowed roots unless dynamic roots are enabled; dangerous roots need `confirmDangerousRoot` |
| `GET /v1/workspaces/snapshot?root=` | Index a workspace: `files` (with content), `manifest` (metadata only: path, size, mtime), directories, assets, `truncated` and `truncatedReasons` |
| `GET /v1/workspaces/file` | Read one text file on demand (up to `BOSON_MAX_OPEN_FILE_BYTES`, default 8 MiB; larger returns `413`) |
| `GET /v1/workspaces/asset` | Read an asset (up to 24 MiB) |
| `POST /v1/workspaces/sync-file` | Write or upload one file (text, or base64 for assets) |
| `POST /v1/workspaces/directory`, `/move`, `/delete` | Create a folder, move or rename, and delete |
| `GET /v1/workspaces/events?workspaceRoot=` | SSE stream of file-change events (see [Live file watching](../guide/workspaces.md#live-file-watching)) |
| `POST /v1/workspaces/search` | Search file contents on disk: `{ workspaceRoot, query, caseSensitive?, wholeWord?, regex?, include?, exclude?, maxResults? }`. Returns matches per file with zero-based `line` and `column`. Skips `.git`, `node_modules`, build output, binaries, files over 2 MiB and symlinks; stops at 10 s or the result limit and says why |
| `POST /v1/workspaces/replace` | Replace matches on disk: the search body plus `replacement`, optional `paths` (restrict to these files) and `dryRun`. Literal mode inserts the text as written; `regex` mode expands `$1` and `$&`. Skips ignored folders, symlinks, binaries and files that change while it runs. Returns `files`, `replacements`, `skipped` |
| `GET /v1/workspaces/graph?workspaceRoot=` | Import relationships for the [workspace graph](../guide/workspaces.md#workspace-graph): per-file symbols and import counts, plus importer → imported `edges`. Includes `truncated` and `warnings` when a limit is reached |
| `GET /v1/workspaces/model` | The module graph |
| `GET /v1/workspaces/build-plan` | Dependency-ordered build plan (`409` if the graph has a cycle) |
| `POST /v1/workspaces/validate` | Built-in language validation |
| `GET /v1/workspaces/scaffold/options`, `POST /v1/workspaces/scaffold` | Project and module templates, and creating them |

### Tasks, terminals and tools

| Endpoint | Purpose |
|---|---|
| `POST /v1/tasks` | Start a build, test or run task |
| `GET /v1/tasks/<id>/events` | SSE for task status and output |
| `POST /v1/tasks/<id>/terminate` | Stop a task |
| `POST /v1/terminals` | Open a terminal session (up to 8 at once) |
| `GET /v1/terminals/<id>/events` | Terminal output stream |
| `POST /v1/terminals/<id>/input`, `/resize`, `/terminate` | Send input, resize, close |
| `GET /v1/tools`, `POST /v1/tools/call` | Bounded workspace tools: list, read, write and search files, and run an allow-listed executable |
| `GET /v1/toolchains` | Installed, allow-listed developer executables |

### Source control, packages and browser

| Endpoint | Purpose |
|---|---|
| `GET /v1/git/status`, `GET /v1/git/diff` | Repository status and diff |
| `GET /v1/git/file-at?path=&rev=` | A file's content at `HEAD`, `HEAD~n` or a commit hash, for comparisons |
| `GET /v1/git/history?path=&limit=` | The commits that changed a file (up to 100, following renames) |
| `GET /v1/git/blame?path=` | Per-line author, time and summary (up to 20,000 lines) |
| `POST /v1/git/<operation>` | `init`, `stage`, `unstage`, `commit`, `create-branch`, `switch-branch`, `add-remote`, `fetch`, `pull`, `push`, `publish` |
| `GET /v1/workspaces/trust`, `POST /v1/workspaces/trust` | Read or change the trust decision for a workspace (`action`: `trust` or `revoke`; `scope`: `folder` or `parent`). Untrusted folders get `403 WORKSPACE_UNTRUSTED` from tasks, terminals, debugging, language servers, package installs, `process.execute` and Git commit/fetch/pull/push/publish |
| `GET /v1/packages/ecosystems`, `GET /v1/packages/modules`, `GET /v1/packages/search`, `POST /v1/packages/install` | Package finder: the supported ecosystems, project discovery (`?ecosystem=all` or one id), registry search (`?ecosystem=`) and install. npm plus Rust, Python, Go, .NET, PHP, Ruby, Dart, Maven and Gradle |
| `POST /v1/browser/sessions` and session routes | Browser-tab sessions for the browser app (public URLs only) |

### Debugging

| Endpoint | Purpose |
|---|---|
| `POST /v1/debug/sessions` | Start a Node.js program (`program`, optional `args`, `breakpoints` as `{ path: [zero-based lines] }`, `stopOnEntry`). Only `.js`, `.mjs` and `.cjs` files inside the workspace; at most 4 sessions |
| `GET /v1/debug/sessions/<id>/events` | SSE: `output`, `paused` (reason, call frames, exception), `resumed`, `terminated` (exit code). Events since the start are replayed to a late subscriber |
| `POST /v1/debug/sessions/<id>/command` | `continue`, `next`, `stepIn`, `stepOut`, `pause`, `terminate`, `breakpoints` (`path`, `lines`), `scopes` (`frameId`), `variables` (`reference`), `evaluate` (`expression`, `frameId`) |
| `DELETE /v1/debug/sessions/<id>` | Stop the program and end the session |

The debuggee runs `node --inspect-brk=127.0.0.1:0` with the same scrubbed environment as other project processes; the inspector listens on the loopback interface only and is never exposed. Sessions end when the Runtime shuts down. The debuggee is the `node` on the Runtime host's `PATH`; the Runtime itself needs Node.js 22+ (built-in WebSocket client).

### Language servers

| Endpoint | Purpose |
|---|---|
| `GET /v1/lsp/servers` | Adapters and whether their executables are available |
| `POST /v1/lsp/sessions` | Start (or reuse) a server for a workspace and language |
| `GET /v1/lsp/sessions/<id>/events` | SSE for server messages, including diagnostics |
| `POST /v1/lsp/sessions/<id>/request`, `/notify` | Send an allow-listed LSP request or notification |
| `DELETE /v1/lsp/sessions/<id>` (or `POST …/close`) | Stop a session |

### Documentation

| Endpoint | Auth | Purpose |
|---|---|---|
| `GET /v1/docs` | Yes | List documentation files (`markdown`, `html` or `asset`) |
| `GET /v1/docs/file?path=` | Yes | Read one Markdown document (up to 2 MiB) |
| `POST /v1/docs/sessions` | Yes | Create a short-lived read-only session for web documents |
| `GET /docs/<session>/<path>` | Session | Serve an HTML document or asset (up to 8 MiB) with a restrictive Content Security Policy |

See [Documentation viewer](../guide/documentation-viewer.md#how-it-works) for the session model.

The status response lists capability flags such as `build`, `test`, `run`, `interactive-terminal`, `git-source-control`, `npm-package-install` and `runtime-browser-proxy`, plus either `pseudo-terminal` or `pipe-shell-compatibility`.

## Terminal

The terminal is a persistent, interactive OS shell rendered with xterm.js. It supports keyboard input, resizing, ANSI colors and interactive applications when the native `node-pty` module loads.

If the terminal reports **pipe compatibility**, the Runtime started a shell but could not load a `node-pty` binary matching the installed Electron version. Basic commands work, but job control, raw TTY mode and full-screen programs (`vim`, `top`, terminal debuggers) do not. Run `npm run rebuild:native`; see [Native PTY rebuild](../development/building.md#native-pty-rebuild).

## Security

The Runtime executes project build scripts, compiler processes, package installs and an interactive shell **with the permissions of the user running it**. Read [Security](security.md) before exposing it beyond your own machine. In short:

- **The token is full control.** Anyone holding it can run commands as you. There are no per-operation permissions yet.
- **Allowed roots are enforced** unless you enable `BOSON_ALLOW_DYNAMIC_ROOTS`, in which case a token holder can register more folders. They do not sandbox the commands you run.
- **Keep it on 127.0.0.1** unless you use HTTPS (or an encrypted tunnel), an exact origin list and a long random token.
- **Open only trusted projects.** Build scripts, package installs, Git hooks and some language servers run project-controlled code.
- **The browser tab stores cookies on disk.** The Runtime keeps browser-tab cookies and sessions under `~/.boson/browser-profile` (or `BOSON_BROWSER_PROFILE_DIR`) as an unencrypted file readable by your account, for up to 30 days of inactivity, with at most 40 sessions. Delete that directory to clear them.
- **The browser proxy loads public sites only.** It refuses `localhost`, `.local`/`.internal` names, private and reserved addresses, and URLs with embedded credentials.

For rules that apply when changing task execution, see [Contributing](../development/contributing.md).
