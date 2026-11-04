# Language support and diagnostics

Boson turns problems from several sources into one diagnostic model. Monaco squiggles, the **Problems** panel and the Explorer's file and folder badges all read the same normalized results, so it does not matter which tool found the problem.

## Two sources of diagnostics

| Source | What it is | Needs installing? |
|---|---|---|
| **Built-in validators** | Read-only checks run by the Workspace Runtime: TypeScript/JavaScript projects, Python syntax, JSON/JSONC and CMake | No |
| **Language servers (LSP)** | Standard language servers started on the Runtime host for the file you are editing | Yes, on the **Runtime host** |

Monaco keeps syntax highlighting, but its own standalone TypeScript/JavaScript diagnostics and the former editor-side CMake check are switched off. Project-aware diagnostics for TypeScript/JavaScript, JSON/JSONC, Python and CMake come from the Runtime, including your unsaved edits.

## Validating a workspace

Use **Validate workspace** to run the built-in validators across the whole workspace. The Problems panel opens with the results. A single file is re-validated as you work, following whichever editor group has focus. Results are marked out of date when the workspace changes.

Validation requires a filesystem workspace (desktop folder, or a remote workspace on a Runtime). It is not available for temporary import snapshots.

Built-in validation is bounded: up to 12,000 workspace entries, nine directory levels, 1,500 source files, 32 TypeScript/JavaScript projects (`tsconfig.json` / `jsconfig.json`) per pass, 300 JSON files and 1,000 reported diagnostics. When a limit is hit the result says so.

## Language servers

When an installed server matches the focused file, Boson finds it on the Runtime host's `PATH`, starts it, and keeps its view of the active document in sync as you change focus. Boson only starts a server whose executable is actually present.

**Important details**

- The executable must be installed on the **Workspace Runtime host**, not on the machine running your browser. For the desktop app these are the same machine.
- Boson never downloads or installs a language server, and a workspace file cannot choose which executable runs. It only starts servers that you installed, when a file in their language opens.
- **Some servers run project code while indexing**, for example `rust-analyzer` (build scripts and procedural macros), TypeScript plugins named in `tsconfig.json`, and JVM servers that import Gradle or Maven builds. Open only projects you trust. In Restricted Mode (a folder you have not trusted) language servers are not started at all. See [Security](../reference/security.md#untrusted-projects).
- Servers still need the project's SDKs and dependencies to produce full semantic diagnostics.
- Workspace-wide coverage depends on what each server indexes. Boson collects diagnostics for opened documents plus whatever extra files the server reports.
- At most eight language server sessions run at once.

### What Boson uses from a server

The editor currently uses **diagnostics, completion, hover, go to definition and occurrence highlighting**. The Runtime also permits additional standard requests (references, rename, formatting, signature help, code actions, semantic tokens and others) so they can be adopted in the UI later. Requests and notifications outside Boson's allow-list are rejected.

### Supported servers

38 adapters are defined. Install the server you want and, if it is not on `PATH` under its default name, point Boson at it with the override variable before starting the Runtime (see [Workspace Runtime](../reference/workspace-runtime.md#language-servers)).

| Language | Default executable | Override variable | Extensions |
|---|---|---|---|
| TypeScript / JavaScript | `typescript-language-server` | `BOSON_LSP_TYPESCRIPT_COMMAND` | `.ts` `.tsx` `.js` `.jsx` `.mts` `.cts` `.mjs` `.cjs` |
| Python | `pyright-langserver` | `BOSON_LSP_PYTHON_COMMAND` | `.py` `.pyi` |
| Rust | `rust-analyzer` | `BOSON_LSP_RUST_COMMAND` | `.rs` |
| Go | `gopls` | `BOSON_LSP_GO_COMMAND` | `.go` |
| C / C++ | `clangd` | `BOSON_LSP_CPP_COMMAND` | `.c` `.h` `.cc` `.cpp` `.cxx` `.hpp` `.hh` `.hxx` `.mm` |
| Java | `jdtls` | `BOSON_LSP_JAVA_COMMAND` | `.java` |
| C# | `csharp-ls` | `BOSON_LSP_CSHARP_COMMAND` | `.cs` `.csx` |
| PHP | `intelephense` | `BOSON_LSP_PHP_COMMAND` | `.php` `.phtml` |
| Ruby | `solargraph` | `BOSON_LSP_RUBY_COMMAND` | `.rb` `.rake` `.gemspec` |
| Kotlin | `kotlin-language-server` | `BOSON_LSP_KOTLIN_COMMAND` | `.kt` `.kts` |
| Swift | `sourcekit-lsp` | `BOSON_LSP_SWIFT_COMMAND` | `.swift` |
| Dart | `dart` | `BOSON_LSP_DART_COMMAND` | `.dart` |
| Lua | `lua-language-server` | `BOSON_LSP_LUA_COMMAND` | `.lua` |
| Shell | `bash-language-server` | `BOSON_LSP_SHELL_COMMAND` | `.sh` `.bash` `.zsh` |
| HTML | `vscode-html-language-server` | `BOSON_LSP_HTML_COMMAND` | `.html` `.htm` `.xhtml` |
| CSS / SCSS / Less | `vscode-css-language-server` | `BOSON_LSP_CSS_COMMAND` | `.css` `.scss` `.less` |
| JSON / JSONC | `vscode-json-language-server` | `BOSON_LSP_JSON_COMMAND` | `.json` `.jsonc` |
| YAML | `yaml-language-server` | `BOSON_LSP_YAML_COMMAND` | `.yaml` `.yml` |
| Terraform | `terraform-ls` | `BOSON_LSP_TERRAFORM_COMMAND` | `.tf` `.tfvars` |
| SQL | `sqls` | `BOSON_LSP_SQL_COMMAND` | `.sql` |
| Dockerfile | `docker-langserver` | `BOSON_LSP_DOCKER_COMMAND` | `.dockerfile` |
| CMake | `cmake-language-server` | `BOSON_LSP_CMAKE_COMMAND` | `.cmake` |
| LaTeX | `texlab` | `BOSON_LSP_LATEX_COMMAND` | `.tex` `.bib` |
| Haskell | `haskell-language-server-wrapper` | `BOSON_LSP_HASKELL_COMMAND` | `.hs` `.lhs` |
| OCaml | `ocamllsp` | `BOSON_LSP_OCAML_COMMAND` | `.ml` `.mli` |
| Zig | `zls` | `BOSON_LSP_ZIG_COMMAND` | `.zig` |
| Perl | `perl-language-server` | `BOSON_LSP_PERL_COMMAND` | `.pl` `.pm` `.t` |
| R | `R` | `BOSON_LSP_R_COMMAND` | `.r` `.rhistory` |
| Scala | `metals` | `BOSON_LSP_SCALA_COMMAND` | `.scala` `.sc` |
| Elixir | `elixir-ls` | `BOSON_LSP_ELIXIR_COMMAND` | `.ex` `.exs` |
| Vue | `vue-language-server` | `BOSON_LSP_VUE_COMMAND` | `.vue` |
| Svelte | `svelteserver` | `BOSON_LSP_SVELTE_COMMAND` | `.svelte` |
| GraphQL | `graphql-lsp` | `BOSON_LSP_GRAPHQL_COMMAND` | `.graphql` `.gql` |
| F# | `fsautocomplete` | `BOSON_LSP_FSHARP_COMMAND` | `.fs` `.fsi` `.fsx` |
| Fortran | `fortls` | `BOSON_LSP_FORTRAN_COMMAND` | `.f` `.for` `.f90` `.f95` `.f03` `.f08` |
| Clojure | `clojure-lsp` | `BOSON_LSP_CLOJURE_COMMAND` | `.clj` `.cljs` `.cljc` `.edn` |
| Erlang | `erlang_ls` | `BOSON_LSP_ERLANG_COMMAND` | `.erl` `.hrl` |
| Julia | `julia` | `BOSON_LSP_JULIA_COMMAND` | `.jl` |

Note that the C / C++ adapter uses `BOSON_LSP_CPP_COMMAND`. For example, to use a specific Java server:

```bash
BOSON_LSP_JAVA_COMMAND=/opt/jdtls/bin/jdtls npm run dev:runtime
```

The desktop app starts its Runtime itself, so set such variables in the environment you launch Boson from.

## Related

- [Troubleshooting → A source file shows no language diagnostics](troubleshooting.md#a-source-file-shows-no-language-diagnostics)
- [Workspace Runtime → HTTP API](../reference/workspace-runtime.md#http-api) for the `/v1/lsp/*` endpoints
