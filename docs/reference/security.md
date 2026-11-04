# Security

This page explains what Boson's Workspace Runtime can do, how to run it safely, and what to avoid. It describes how things behave today, including limitations that are still being worked on.

## The short version

- **The Runtime token is full control.** Anyone who has it can run commands as the account that started the Runtime.
- **Open only projects you trust, or leave them in Restricted Mode.** Building, running, installing packages, Git hooks and some language servers execute code that comes with the project; the Runtime refuses all of those in a folder you have not trusted.
- **Keep the Runtime on your own machine** (`127.0.0.1`) unless you use HTTPS or an encrypted tunnel, a long random token and an exact list of allowed origins.
- **Allowed workspace roots do not confine a token holder.** They are a convenience, not a security boundary.
- **The browser tab stores cookies on disk**, unencrypted, for up to 30 days.

## What the Runtime can do

The Runtime runs builds, tests and runs, a real interactive terminal, package installs, Git operations and language servers, and it reads and writes files on its host. It does this with the permissions of the user who started it. The bearer token is the only thing standing between a client and those abilities, and it currently grants all of them at once; there are no per-operation permissions or approvals yet.

## Choose how to run it

| Setup | Exposure | Notes |
|---|---|---|
| **Desktop app** | The Runtime listens on a random loopback port with a per-launch token that only the app knows | The safest default |
| **Browser app and a Runtime on the same machine** | Loopback only | You copy the token printed by the Runtime |
| **Browser app and a remote Runtime** | Reachable over a network | Needs HTTPS or a tunnel, an exact origin list and a strong token; see below |

## Protect the token

- Use the generated token, or set `BOSON_RUNTIME_TOKEN` to a long random value (at least 32 characters).
- Do not paste it into chats, tickets or shared documents, and do not commit it.
- Treat anything that holds it, including external automation and agents, as having a shell on the host.
- In the browser app the token is kept for the current tab only.
- Restart the Runtime to rotate a generated token.

## Allowed workspace roots

`BOSON_WORKSPACE_ROOT` and `BOSON_ALLOWED_WORKSPACE_ROOTS` are enforced by the Runtime: file, folder-browser, registration and tool requests outside them are refused (`ROOT_NOT_ALLOWED` / `PATH_NOT_ALLOWED`), including through symlinks. The desktop app runs its own local Runtime with `BOSON_ALLOW_DYNAMIC_ROOTS=true` so the folder you choose in the native picker can be opened; your home folder, drive roots and system locations require a native confirmation first. A standalone or remote Runtime keeps dynamic roots **off** unless you turn them on. These checks limit file access through the Runtime API; they do not sandbox build scripts, the terminal or Git hooks, which run as your account. Use a dedicated account, container or virtual machine for real isolation.

## Untrusted projects

Opening a project is not always safe. These actions run project-controlled code:

- **Build, Test and Run**, and anything you type in the terminal.
- **Package installs.** npm lifecycle scripts run during installation.
- **Git.** Hooks run on commit, and a repository's own configuration can name programs that Git runs.
- **Language servers.** Some run project code while indexing, such as `rust-analyzer` (build scripts and procedural macros), TypeScript plugins named in `tsconfig.json`, and JVM servers that import Gradle or Maven builds.

### Workspace trust and Restricted Mode

The Runtime keeps a list of **trusted folders** and enforces it itself, so it holds for every client (the app, a browser, a script). A folder is trusted when it, or a parent, is on the list. Anything else is in **Restricted Mode**.

In Restricted Mode you can read, edit and search files, use the built-in validators and module analysis, and run read-only Git (status, diff, branches, history, blame). The Runtime refuses everything that would run the project's own programs, with `403` and `code: "WORKSPACE_UNTRUSTED"`:

- Build, Test and Run tasks, and the terminal.
- Debugging.
- Language servers.
- Package installs.
- The `process.execute` tool.
- Git commit, fetch, pull, push and publish (hooks, credential helpers and configured transports).

How the decision is made:

- Opening a folder you have not decided about shows a dialog: **Trust this folder**, **Trust the parent folder** or **Open in Restricted Mode**. A **Restricted Mode** chip in the header and **Manage Workspace Trust** (Help menu and command palette) reopen it later.
- Decisions are stored in `trust.json` in the Runtime's data directory (`~/.boson`, or `BOSON_RUNTIME_DATA_DIR`; override with `BOSON_TRUST_FILE`), written with owner-only permissions. A corrupt or missing file trusts nothing.
- A drive root, your home folder and top-level folders such as `/home` or `C:\Users` are refused as too broad.
- Projects you create from a template in Boson are trusted automatically. Nothing is trusted because it was registered over HTTP, restored from an earlier session, or contains a particular file.
- Headless hosts can list folders to trust for one run in `BOSON_TRUSTED_WORKSPACE_ROOTS` (path-separator list). These cannot be revoked from the API.

Limits you should know:

- **Anyone with the Runtime token can change the trust list.** Trust protects you from what a *folder* does, not from a stolen token. Scoped tokens and approvals are still planned (SEC-02).
- Restricted Mode does not sandbox a folder you chose to trust. Trusted builds, terminals and installs run as your account.
- Even in Restricted Mode, Boson started its own helpers carefully: Git read commands run with a hardened configuration, and the Python syntax checker runs in isolated mode so a planted `ast.py` or `json.py` in the folder is never imported.

For code you do not trust at all, still prefer a virtual machine, a container or a dedicated account.

## Browser data

The browser tab behaves differently on desktop and in the browser app.

| | Desktop | Browser app |
|---|---|---|
| Mechanism | A native, sandboxed Electron webview | Pages loaded through the Runtime's browser proxy |
| Local servers | Allowed | Refused (public sites only) |
| Where logins are stored | The app's persistent browser profile | Cookies and sessions on the Runtime host |

For the proxy, the Runtime saves cookies and sessions as an **unencrypted** file under `~/.boson/browser-profile` (or `BOSON_BROWSER_PROFILE_DIR`). The folder and file are limited to your account on Linux and macOS; Windows relies on your user profile's permissions. Sessions expire after 30 days of inactivity, at most 40 are kept, and session cookies are saved along with persistent ones. There is no command to clear them yet: delete the folder on the Runtime host.

Practical advice: do not sign in to sensitive accounts through a Runtime on a shared or remote host, and delete the profile folder when you are finished.

The proxy refuses `localhost`, `.local` and `.internal` names, private and reserved addresses, and URLs with embedded credentials.

## Documentation viewer

Documents are read-only and come from the installed `docs/` folder. Web documents run in an isolated frame with a strict policy and never receive the token. See [Documentation viewer](../guide/documentation-viewer.md#how-it-works).

## Hosting the browser app

If you serve Boson to others, treat it like any web application that controls a shell:

- Serve it and the Runtime over HTTPS only.
- Set `BOSON_ALLOWED_ORIGINS` to the exact origin of the app. Never use the development wildcards.
- Use a long random token and rotate it.
- Do not serve the web app and the Runtime from the same origin.
- The web app sets a Content Security Policy and standard security headers (`X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, HSTS in production). Its CSP still allows inline scripts for Next.js bootstrap; set `NEXT_PUBLIC_BOSON_RUNTIME_ORIGINS` to your Runtime origin(s) so `connect-src` and `frame-src` allow them. Add your own stricter policy at the web server if you can.
- Keep the Runtime on a private network, or behind an authenticated tunnel.
- Run it as a dedicated, low-privilege account.

## Summary of known limitations

| Limitation | Meaning for you |
|---|---|
| One token with full access | Guard it like a password to the machine |
| Allowed roots limit the Runtime API, not the commands you run | Use an OS-level boundary for real isolation |
| No Restricted Mode for untrusted folders | Review before you build; use a VM or container |
| Browser cookies stored unencrypted | Delete the profile folder when done |
| Web CSP keeps `'unsafe-inline'` for scripts | Tighten at your web server if you can |
| Desktop renderer is not yet fully sandboxed (`sandbox: false`) | Planned; the main process now validates IPC senders, navigation and permissions |

Report security problems privately to the project maintainers rather than in a public issue.
