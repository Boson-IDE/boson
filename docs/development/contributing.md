# Contributing

Thanks for working on Boson. Set up with the Quick start (`README.md`), then read [Architecture](architecture.md) for the package boundaries below.

## Where code belongs

| Change | Location |
|---|---|
| UI shared by desktop and web | `packages/ui` |
| Colors, tokens, IDE styling, Monaco themes | `packages/themes` |
| Typed interfaces shared across packages | `packages/contracts` |
| Electron-only file access, dialogs, window operations | `apps/desktop`, exposed through the preload bridge |
| Anything that starts a project process, reads host files, or runs Git or language tools | `apps/runtime` |

Never add shell execution to the renderer or the shared UI. Never import Electron, Node filesystem or Next.js server-only modules in `packages/ui`.

## Before opening a change

```bash
npm run typecheck
npm run build
npm test
```

Run `npm install` first. Report only the checks you actually ran. Many desktop tests read source files and assert on their contents, so restructuring `BosonShell.tsx` or the Runtime may require updating them. See [Building → Checks and tests](building.md#checks-and-tests).

## Workflows

### Extending task execution

- Preserve the workspace allow-list.
- Validate every relative path against the selected root.
- Avoid shell interpolation; pass arguments as arrays.
- Bound output size and execution time.
- Make supported toolchain behavior explicit.

### Adding or renaming a Runtime source file

The packaged desktop app copies Runtime files individually. In the same change:

1. Add the file to `extraResources` in `apps/desktop/electron-builder.yml`.
2. Add it to the file list in `apps/desktop/scripts/verify-runtime-package.mjs`.
3. Run `npm run build --workspace @boson/desktop` (it runs the verification).

A Runtime dependency must also be a production dependency of `@boson/desktop` and be loaded through `loadRuntimeDependency()`.

### Adding a language server adapter

Add an entry to `SERVER_DEFINITIONS` in `apps/runtime/src/language-servers.js` with its label, default executable, a `BOSON_LSP_<NAME>_COMMAND` override, arguments and file extensions. Then add the row to [Language support](../guide/language-support.md#supported-servers) and a test in `apps/runtime/test/language-servers.test.js`. Adapters must never download or install anything, and only allow-listed LSP methods may be forwarded.

### Adding a project or module template

Edit `apps/runtime/src/project-scaffolding.js` and keep it side-effect free: no spawned processes, exclusive file creation, paths validated against the workspace root, rollback on failure. Update the Project Setup panel, the translations and [Projects, modules and build plans](../guide/projects-and-modules.md), and add a case to `project-scaffolding.test.js`.

### Adding an API route

Require `authorized(request)`, resolve any workspace through `resolveWorkspaceRoot` (this enforces the allow-list), bound input and output sizes, and document the route in [Workspace Runtime → HTTP API](../reference/workspace-runtime.md#http-api). If the desktop renderer needs an event stream, add it to the explicit allow-list in `apps/desktop/src/main/runtime-stream-routes.mjs`.

### Writing documentation

User and developer documentation lives in `docs/` and is shown inside Boson (**Help → Documentation**). Read [Documentation viewer](../guide/documentation-viewer.md) first; the rules that matter most:

- Link only to other files **inside docs/**. The viewer cannot open `README.md`, `AGENTS.md` or source files, so name those in plain code text instead of linking.
- Keep security-sensitive material, handoff notes and anything internal **out of docs/**: the whole folder is installed on every user's machine.
- Web documents (`.html`) must be self-contained: no remote scripts, styles, fonts or images.
- Update the docs in the same change as the behavior they describe, and re-read them in the viewer.

### Security-sensitive changes

Anything that touches authentication, workspace roots, file or process access, the browser proxy, the documentation server or the Electron shell must keep the Runtime as the enforcement point (a UI confirmation is not authorization). Check the project's internal security checklist (kept at the repository root, outside `docs/`) for open issues in that area before you start.

## UI rules

- Keep keyboard accessibility and visible focus states.
- Support reduced motion.
- Use the shared dark/light theme tokens.
- Do not use `window.prompt`, `alert` or `confirm`; use the shared dialog system.
- Reuse `BosonMark`; do not create a second logo implementation.

## Localization

Boson's UI is translated through `packages/ui/src/i18n.ts`, which has no dependencies.

- Wrap every user-visible string in `t('English text')`. The English text is the key, and untranslated strings fall back to English.
- Add translations for all ten non-English locales (`es`, `ru`, `fr`, `de`, `nl`, `pt-BR`, `it`, `ja`, `zh-CN`, `ko`). Keep `{placeholders}` intact.
- Do not translate user source code, file contents or tool output such as compiler messages.
- To add a locale, extend `BosonLocale` and `BOSON_LOCALE_OPTIONS`, add its tables, and update the locale test.

For larger efforts and the roadmap, see AGENTS.md (`AGENTS.md`).
