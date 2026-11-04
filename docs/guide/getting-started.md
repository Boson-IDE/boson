# Getting started

A ten-minute tour: run Boson, open a project, edit, split the editor, build and run it, and find help.

## 1. Run Boson

**Requirements:** Node.js 20 or newer (22 LTS recommended), npm 10 or newer, and the toolchain for the projects you want to build (for example a C/C++ compiler and CMake, Python, Go, a JDK and Maven, or .NET).

From the repository root:

```bash
npm install
npm run dev
```

The desktop app starts its own Workspace Runtime, so there is nothing else to launch.

To use Boson in the browser instead:

```bash
npm run dev:web        # the app, at http://localhost:3000
npm run dev:runtime    # in another terminal: the Runtime that holds your projects
```

## 2. Connect a Runtime (browser only)

The browser app needs to know where your Runtime is. Open **Settings → Workspace Runtime**, enter the URL and token that the Runtime printed when it started (by default `http://127.0.0.1:4178`), and choose **Test connection**. See [Workspace Runtime](../reference/workspace-runtime.md) for remote hosts and HTTPS.

> The token gives full control of the account that runs the Runtime. Keep it private and read [Security](../reference/security.md) before exposing a Runtime beyond your own machine.

## 3. Open a project

Choose **File → Open Workspace…** (`Ctrl/Cmd+O`), browse to a folder and choose **Open this folder**. On desktop you browse your own computer; in the browser you browse the Runtime host.

Starting from scratch? **File → New Project…** sets up a workspace from a template, and **File → New Module…** adds a module to it. See [Projects, modules and build plans](projects-and-modules.md).

> Open only projects you trust. Building, running, installing packages and some language servers execute code from the project. See [Security](../reference/security.md#untrusted-projects).

## 4. Find your way around

| Area | What it does |
|---|---|
| **Activity bar** (left edge) | Switch between Explorer, Search, Source Control, Packages, Toolchains and Settings |
| **Explorer** | Browse files; create, rename, move and delete from the toolbar or right-click menu |
| **Editor** | Tabs and editor groups; a single click opens a preview tab, a double-click keeps it |
| **Bottom panel** | Output, Problems and the Terminal (`Ctrl/Cmd` plus the backtick key toggles the terminal) |
| **Inspector** (right) | Workspace overview and details (`Ctrl/Cmd+Shift+B` toggles it) |
| **Status bar** | Runtime connection and task progress |

## 5. Edit and save

Open a file from the Explorer or with **Quick Open**, edit it, and save with `Ctrl/Cmd+S` (**Save All** is `Ctrl/Cmd+Shift+S`). Unsaved tabs show an indicator. Search across the workspace with `Ctrl/Cmd+Shift+F`, and run any command from the palette with `Ctrl/Cmd+Shift+P` or `F1`.

## 6. Split the editor

Drag a tab, or a file from the Explorer, to the **right edge** of an editor to open it side by side, or to the **bottom edge** to stack it. Splits nest, up to four columns by four rows, and you can drag the dividers to resize. Boson remembers the layout and your open tabs for each workspace. More in [Using workspaces](workspaces.md#tabs-and-groups).

## 7. Build, test and run

Use the **Run** menu: **Build Workspace**, **Run Tests**, **Run Active File** (`F5`) and **Stop Current Task**. Output streams into the bottom panel. For multi-module projects Boson also works out a dependency-ordered build plan. See [Building, testing and running](workspaces.md#building-testing-and-running).

## 8. Check problems

Diagnostics appear as markers in the editor, in the **Problems** panel and as badges in the Explorer. They come from built-in validators and from language servers you have installed. See [Language support](language-support.md).

## 9. Use Git and packages

The **Source Control** view handles status, diff, staging, commits, branches and remotes. The **Packages** view finds npm modules and installs packages. See [Source Control](workspaces.md#source-control-git) and [Packages](workspaces.md#packages-npm).

## 10. Get help and make it yours

- **Help → Documentation** opens this guide at any time.
- The [shortcut finder](../web/shortcuts.html) lists every shortcut.
- **Settings → Language** changes the interface language (11 are available), and **Settings → Appearance** and **Custom Themes** change how Boson looks. See [Custom themes](custom-themes.md) and [Plugins](plugins.md).
- Stuck? [Troubleshooting](troubleshooting.md) lists common symptoms and fixes.
