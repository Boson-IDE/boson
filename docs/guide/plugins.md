# Plugin manifests

Boson plugins are imported from a single `.boson-plugin.json` file. Schema version 1 is a **declarative** contribution format: it does not evaluate plugin JavaScript, load native modules, or run code in the IDE renderer or the Workspace Runtime.

A complete example is in `examples/boson-plugin-example.json`.

## Install and manage

Open **Settings → Extensions → Install plugin manifest…** and choose a `.boson-plugin.json` file. Installed plugins can be enabled, disabled or uninstalled. The validated manifest and its enabled state are saved in Boson's local preferences.

## What a plugin can contribute

| Contribution | What it does | Limit |
|---|---|---|
| **Commands** | Insert bounded text at the editor selection, open an HTTPS URL, or show a notification | 100 per plugin |
| **Snippets** | Language-labelled templates in the command palette; selecting one inserts its body at the selection | 200 per plugin |
| **Themes** | Complete color palettes in the unified theme picker, shared by workbench, editor and terminal (same format as [custom themes](custom-themes.md)) | 24 per plugin |

Other rules: a manifest is limited to 1 MiB, IDs must be unique within each contribution type, and strings and colors are bounded and validated both on import and when restored.

## Why plugins are declarative today

Executable third-party extensions need a separately sandboxed extension host, capability permissions, lifecycle isolation and a trust and update model. Running imported JavaScript inside the privileged renderer or Runtime would expose workspace files and application capabilities to untrusted code. The manifest API therefore offers useful contributions without arbitrary code execution. An executable extension host is a separate security milestone, not something implied by importing a manifest.
