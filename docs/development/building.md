# Building, testing and packaging

## Development commands

Run these from the repository root after `npm install`.

| Command | What it does |
|---|---|
| `npm run dev` | Starts the Electron desktop app (and its managed Runtime) |
| `npm run dev:web` | Starts the Next.js browser app |
| `npm run dev:runtime` | Starts a standalone Workspace Runtime (alias: `npm run dev:agent`) |
| `npm run dev:desktop:pty` | Rebuilds `node-pty` for Electron, then starts the desktop app |
| `npm run rebuild:native` | Rebuilds native modules for the installed Electron version |
| `npm run typecheck` | Type-checks every workspace that defines it |
| `npm run build` | Runs each workspace's build script |
| `npm run package:desktop` | Builds an installer for the current platform |
| `npm run package:desktop:dir` | Builds an unpacked app directory |

When the desktop app starts, a lightweight native splash appears while the main process launches the Runtime and waits for its health endpoint. It closes as soon as the IDE window is ready. The titlebar is draggable wherever there is no interactive control.

## Checks and tests

```bash
npm run typecheck
npm run build
npm test                                          # desktop/UI suite, then the Runtime suite
```

`npm test` runs `node --test "apps/desktop/test/*.test.mjs"` followed by `npm run test --workspace @boson/runtime`. Run either half on its own while iterating.

| Suite | Location | What it covers |
|---|---|---|
| Runtime | `apps/runtime/test` | The HTTP API, documentation server, task planning, browser workspaces, project scaffolding, workspace analysis, language servers, and packaged-app startup |
| Desktop / UI | `apps/desktop/test` | Editor-group layout, drop targets, tab order and file-drag URLs, layout persistence, editor tokenization, Markdown anchors, Explorer state, locale options, project setup, Runtime stream routing |

Notes:

- Run `npm install` first. Runtime tests start the real Runtime, which needs its dependencies (`acorn`, `node-pty`, `undici`, `postcss`); without them the browser-workspace test fails with a missing-dependency error.
- Many desktop tests are source-level regression checks that read `BosonShell.tsx`, the layout and the Runtime source. Expect them to need updates when those files are restructured.
- The desktop typecheck covers the Electron main/preload and renderer projects. The web workspace has a TypeScript check and a Next.js production build.
- The packaged-layout test (`apps/runtime/test/desktop-package-startup.test.js`) starts the Runtime from a copy of the files the installer ships, so it catches a Runtime file that is missing from the package.

Monaco is exposed from the `@boson/ui/editor` entrypoint and loaded lazily after the shell mounts. Importing the normal `@boson/ui` entrypoint does not evaluate browser-only code during server rendering. The browser stylesheet is standard CSS, so Next.js can process it without the desktop Vite/Tailwind plugin.

## Native PTY rebuild

`npm install` / `npm ci` installs JavaScript dependencies and the Electron distribution but does **not** rebuild `node-pty` for Electron. Until you do, the terminal runs in pipe compatibility mode.

```bash
npm run rebuild:native   # once, then: npm run dev
# or, in one step:
npm run dev:desktop:pty
```

Packaging commands run this rebuild automatically, and `node-pty` is unpacked from the application archive.

### Windows prerequisites

The rebuild needs the Visual Studio C++ toolset **and the Spectre-mitigated libraries** for your target architecture. Without them it fails with `MSB8040` ([Microsoft documentation](https://learn.microsoft.com/en-us/visualstudio/msbuild/errors/msb8040?view=vs-2022)).

1. Open **Visual Studio Installer → Modify → Individual components**.
2. Search for "Spectre" and install the libraries matching your MSVC toolset and architecture (for example x64 or x86).
3. Reopen the terminal, then run `npm run rebuild:native` again.

## Desktop packaging

```bash
npm run package:desktop        # installer(s) for the current platform
npm run package:desktop:dir    # unpacked directory, useful for testing
```

The build runs the Runtime packaging check (`verify:runtime-package`), the desktop typecheck and production bundle, rebuilds native dependencies, and invokes Electron Builder. Output goes to the repository-root `binaries/` directory, which is generated and ignored by Git.

| Platform | Artifacts |
|---|---|
| Windows | NSIS installer (`Boson-<version>-setup.exe`) and portable executable (`Boson-<version>-portable.exe`) |
| macOS | DMG and ZIP |
| Linux | AppImage, Snap and DEB |

Targets are configured in `apps/desktop/electron-builder.yml`. Per-platform scripts are also available in the desktop workspace: `build:win`, `build:mac`, `build:linux` and `build:unpack`.

### How the Runtime is packaged

The Runtime is **not** bundled into the app archive. Its source files are copied to `resources/runtime/` through `extraResources`, and the desktop main process launches `resources/runtime/index.js` as a utility process, passing `BOSON_RUNTIME_MODULE_ROOT` so it can load production dependencies from `app.asar`. The bundled documentation is copied the same way to `resources/docs/`, and the main process points the Runtime at it with `BOSON_DOCS_ROOT`.

Every file the Runtime imports must be listed under `extraResources` in `electron-builder.yml` and checked by `apps/desktop/scripts/verify-runtime-package.mjs`. Today that is `index.js`, `workspace-analysis.js`, `cmake-validation.js`, `workspace-watcher.js`, `project-scaffolding.js`, `language-servers.js` and `package.json`, plus the `docs/` directory.

> **When you add a Runtime source file** that `index.js` (or another Runtime module) imports, add it to `extraResources` *and* to the verify script's file list, and extend the packaged-layout test if needed. A missing file otherwise ships as a Runtime that cannot start (`ERR_MODULE_NOT_FOUND`).

> **Everything in docs/ ships to every user** and is browsable in **Help → Documentation**. Keep internal notes, handoff files and anything sensitive out of it.

## Before you ship a release

The packaging defect that stopped the Runtime from starting is fixed. These items still need attention before a public release. Treat the security-related ones as blockers; the project keeps a detailed internal checklist at the repository root, outside `docs/`.

**Security-related**

- **Placeholders.** `apps/desktop/package.json` has `author: example.com` and `homepage: https://electron-vite.org`; `electron-builder.yml` has the Linux maintainer `electronjs.org` and the auto-update URL `https://example.com/auto-updates` (also in `dev-app-update.yml`). Replace them.
- **Dependency advisories.** The lockfile records Electron `44.7.2` and Monaco's transitive DOMPurify `3.4.15`. Earlier audit output flagged both. Run `npm audit`, read the advisories for those exact versions, then update to patched releases and re-run all tests.
- **macOS entitlements.** `electron-builder.yml` references `build/entitlements.mac.plist`, but `apps/desktop/build/` does not exist. Add the file (with only the entitlements Electron needs) or remove the setting. Notarization is currently disabled.
- **Electron download mirror.** `electronDownload.mirror` points at `https://npmmirror.com/mirrors/electron/`. Prefer the official source with checksum verification.
- **Code signing and updates.** No signing is configured for Windows, and the update feed is a placeholder.

**General**

- **Version number.** Every package, including the desktop app, is at `0.1.0`, so installers are named `Boson-0.1.0-…`. Bump versions deliberately.
- **Documentation.** Open **Help → Documentation** in the packaged app and confirm the guides, the web page and the images all load.
- **Repository hygiene.** A compiled `packages/contracts/src/index.js` and `apps/web/tsconfig.tsbuildinfo` are checked in; remove them and extend `.gitignore`.
