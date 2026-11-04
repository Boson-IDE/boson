# Documentation viewer

Boson includes a built-in viewer for its own documentation. It shows Markdown guides and interactive web pages in one dialog, from anywhere in the app. This page explains how to use it and how to write documents for it.

## Open it

- Choose **Help → Documentation** from the menu bar.
- Open the command palette (`Ctrl/Cmd+Shift+P` or `F1`) and run **Documentation**.
- In the compact **More** menu, choose **Documentation**.

The viewer opens as a dialog over your work. It does not close when you click outside it; use its close button. Your editor, tabs and tasks are untouched.

## What you see

- **A list of documents** on the left, in folders, with a **Markdown** or **Web** badge. The count at the bottom shows how many documents were found.
- **A search box** above the list. It filters by file name and folder, not by the text inside documents.
- **The reader** on the right. The start page is chosen automatically: `index.md`, otherwise `readme.md`, otherwise the first Markdown file, otherwise the first web document.

The list is sorted by path, with folders and files mixed alphabetically, so folder names such as `development`, `guide` and `reference` appear in that order, with `index.md` between `guide` and `reference`.

## Reading Markdown

Markdown documents are shown in Boson's Markdown reader.

- Links to other documents (`other.md`, `../guide/setup.md#heading`) open in the viewer. Links to a heading (`#heading`) scroll within the page.
- Links to websites (`https://…`) open in a Boson browser tab in your workspace.
- Images and other files beside a document load by relative path.
- Code blocks, tables, task lists and block quotes are supported.

## Using web documents

A document written in HTML, CSS and JavaScript opens in an isolated frame. It can have its own layout, diagrams, calculators or search, like the [shortcut finder](../web/shortcuts.html). Links from a web document to other documents open them in the viewer, so navigation feels the same as in Markdown. The [web documentation example](../web/index.html) shows the pieces.

## How it works

1. The viewer asks the Workspace Runtime for the list of documents (`GET /v1/docs`).
2. Markdown is fetched file by file (`GET /v1/docs/file`) and rendered by the viewer.
3. For web documents the viewer asks for a **session** (`POST /v1/docs/sessions`). The Runtime returns a short-lived, unguessable path such as `/docs/<session>/`. The frame loads documents and their assets from that path. The Runtime token is never given to a web document.
4. A session expires 15 minutes after its last use (and never lives longer than 8 hours), and at most 32 sessions exist at once. If you see "Documentation session expired", reopen the viewer.
5. Web documents run in a sandboxed frame with no access to Boson. They may use their own external script and style files, but inline scripts are blocked: the Runtime's policy only allows scripts it injected itself (a navigation helper that lets links between documents move the viewer). Do not add inline `<script>` blocks or `onclick` handlers to web documents.

Because documents are read through the Runtime, the viewer needs a connected Runtime. The desktop app connects automatically. In the browser, connect one under **Settings → Workspace Runtime** first.

## Where the documents live

Documentation is the `docs/` folder of the repository. The desktop installer copies it next to the app, and the Runtime serves it. A standalone Runtime reads `docs/` from the repository, or the folder named by the `BOSON_DOCS_ROOT` environment variable.

> Everything in `docs/` is installed on every user's machine and is visible in the viewer. Do not put internal notes, handoff files, credentials or anything sensitive in it.

## Writing documents

### Files, folders and names

- Use **Markdown** (`.md`; `.mdx` files are shown as plain Markdown) for guides and **HTML** (`.html`, `.htm`) for pages that need layout or interaction. Keep straightforward guides in Markdown.
- Choose descriptive file names. The list shows paths, and search matches only the path.
- Group documents in folders (`guide/`, `reference/`, `development/`). Folders can be up to 12 levels deep, but one level is usually enough.
- Name the start page `index.md`.
- Hidden files and folders (names starting with `.`) and symbolic links are never listed or served.
- Start every Markdown file with a single `#` title.

### Supported file types

| Kind | Extensions |
|---|---|
| Markdown | `.md`, `.mdx` |
| Web documents | `.html`, `.htm` |
| Web assets | `.css`, `.js`, `.mjs`, `.json`, `.svg` |
| Images | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.avif`, `.ico` |
| Fonts and other | `.woff`, `.woff2`, `.ttf`, `.otf`, `.pdf` |

### Limits

| Limit | Value |
|---|---|
| Markdown document size | 2 MiB |
| Asset or web document size | 8 MiB |
| Files listed | 3,000 |
| Folder depth | 12 |
| File or folder name length | 255 characters |

### Markdown that renders well

The reader supports headings, paragraphs, bold, italic, strikethrough, inline code, links, images, fenced code blocks, tables, task lists, block quotes and horizontal rules. It is intentionally simple, so avoid the following:

- **Nested lists.** Indentation is ignored, so nested items flatten into one list. Use separate lists or a table.
- **Raw HTML** in Markdown. It is shown as text. Use a web document instead.
- **Nested inline formatting**, such as code or links inside bold text. Put them next to each other instead.
- **Multi-level block quotes** and footnotes.

Source line breaks inside a paragraph are kept, so write each paragraph as one line.

### Linking

- Link to other documents with relative paths, for example `[Security](../reference/security.md)`.
- **Link only to files inside docs/.** Links to the repository's `README.md`, source files or configuration files cannot be opened; mention them in plain code text, such as `apps/runtime/.env.example`.
- Link to headings with `#heading-text` (lowercase, words joined by hyphens). Repeated headings get unique suffixes.
- Put images beside the document or in an `images/` folder and use a relative path.

### Rules for web documents

Web documents run under a strict policy so that documentation can never read your files or control Boson:

- They are shown in a frame that allows scripts, forms and popups, but is isolated from Boson and from the Runtime. It has no access to your workspace and no Runtime token.
- They can load only their **own sibling files**, plus `data:` and `blob:` resources. Remote scripts, styles, fonts and images are blocked, so bundle what you need.
- Use ordinary relative URLs for stylesheets, scripts, images and fonts: `<link rel="stylesheet" href="./docs.css">`.
- To go to another document, use a normal link such as `<a href="../index.md">Back to the index</a>`. Boson intercepts it and opens the document in the viewer.
- Support light and dark themes with `prefers-color-scheme`, and design for a dialog that can be narrow.
- Keep each page self-contained and small. Do not store secrets in a web document.

### Try it

1. Add or edit a document under `docs/`.
2. Close and reopen **Help → Documentation** so the list is refreshed.
3. Check the page in both a wide and a narrow window.

When you are done, run the tests (`npm test`) and, before a release, open the packaged app's viewer to confirm everything shipped. See [Building](../development/building.md#how-the-runtime-is-packaged).


## Browsing the list

The list on the left works like the Explorer. Folders such as **guide** or **reference** are collapsed rows with a count; click one to see only the documents inside it, and click again to fold it. The folder that contains the open document expands on its own. Typing in the search box shows only matching documents together with their folders.

## Building an HTML documentation site

`npm run docs:site` converts every Markdown file under `docs/` into a standalone website in `docs-site/`: a folder sidebar, full-text search (press `/`), an on-page contents list, previous/next links, copy buttons on code blocks and light and dark themes. Open `docs-site/index.html` in a browser or publish the folder on any static host. Pass your own folders with `node scripts/build-docs-site.mjs <docsDir> <outDir>`. Links between Markdown files are rewritten to the generated pages, and the site loads no remote resources.
