Rendered site: https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/

# CAI Field Manual — Library

The CAI Field Manual Library is the aggregated shelf of CAI field manuals: one landing site that mounts the individual manual repos as books. It is a **content-only repo** — chrome (layouts, assets, static, scripts) comes from the [`CAI-Field-Manual-Shell`](https://github.com/rh-aiservices-bu/CAI-Field-Manual-Shell) Hugo module pinned in `hugo.yaml` `module.imports[].version`; manuals come from git submodules mounted via `hugo.yaml` `module.mounts`.

Built with [Hugo](https://gohugo.io/) and the [OINK](https://oink.pgsty.com/) theme (`github.com/pgsty/oink`, transitively via the Shell module).

## How to consume it

**Humans** — start at the library landing, then pick a book from the bookshelf:

- [Hello World](https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/hello-world/) — the blank field-manual skeleton with six bundles.
- [RHOAI 2.25 → 3.5 migration](https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/rhoai-2-25-to-3-5/) — in-place OpenShift AI 2.25.10+ to 3.5, one page per phase.
- [MaaS](https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/guide/) — model-as-a-service field manual.

Each book is reachable via the **bookshelf** landing section (book width banded by top-level heading count), the landing cards, and its own sidebar.

- The **variant selector** at the top of every manual page exposes five axes: *RHAIE version* (3.4 / 3.5 / 3.6), *Ops mode* (GitOps / Manual / Scripted), *Deployment type* (GitOps-managed / Not GitOps-managed), *Cluster network* (Connected / Disconnected), *Kueue end state* (Removed / Unmanaged) — each with *Show all*. A status line beneath the pills restates the current view as a sentence (e.g. *Showing RHAIE 3.6, GitOps steps — reading as guest on api.ocp.abcd.sandbox1234.opentlc.com*); choices persist in `localStorage` across pages, visits, and tabs.
- **Customize** fields (your name, cluster) substitute into every `customize` token across the manuals via the *Customize this!* flow.
- Search with `/` or the navbar search box (`offline_search`).
- Each section also renders a **print bundle** for offline/print reading.

**AI agents** — consume the markdown:

- Per-page markdown via the page action menu: *Copy markdown*, *View markdown*, *Open in Claude / ChatGPT*, *Open Gemini*.
- **Command palette** (navbar search or `/`): type `agent` and choose **"Feed it to my AI Agent"** — a copy-ready prompt for Claude Code or Codex that clones the repo, reads the README and code, reports capabilities, and asks what to do next.
- Whole site as [`llms.txt`](https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/llms.txt).
- Per-manual flattened bundles as `llms-full.txt` under `rhoai-2-25-to-3-5/`.
- The markdown, `llms.txt`, and print outputs carry every display option labeled and every customize token as its shipped default — only the interactive HTML filters content or substitutes values.

## Where things live

- `hugo.yaml` — site identity (`title`, `baseURL`, `github_repo`), the top-level `cascade:` book-segregation lever (`sidebar_expanded: true` site-wide + `sidebar_root_for: self` on every top-level section `path: /*`), the five variant axes + two customize fields (`user`, `cluster`), and the three submodule mounts (`hello-world`, `rhoai-2-25-to-3-5`, `guide`). Chrome defaults (`logo`, `offline_search`, `copyright`, `footer_center_info`, `ui` incl. `docs_section`) live in the Shell module's `hugo.yaml` and are deep-merged; site values win on conflict.
- `content/` — intentionally absent. The aggregator owns no pages; every page is mounted from a manual repo via `hugo.yaml:module.mounts`. The book-segregation lever (`sidebar_expanded: true` + `sidebar_root_for: self` on every top-level section `path: /*`) lives in `hugo.yaml`'s top-level `cascade:`, not in a `content/_index.md`.
- `data/home.yaml` — landing page composition: `hero`, `bookshelf` (three books, URLs `hello-world/`, `rhoai-2-25-to-3-5/`, `guide/`), `timeline`, `cards`, `faq`, `cta`. Every card URL must point into a mount; doctrine links (definition, conventions, walkthrough) point into `hello-world/what-is-a-field-manual/`.
- `external/` — three git submodules, each a manual repo (read-only here; edit in their own repos): `manual-hello-world` → `CAI-Field-Manual-Hello-World`, `manual-3-migration` → `CAI-Field-Manual-Migration-2-3`, `manual-maas` → `CAI-Field-Manual-MaaS`.
- `layouts/_partials/landing/sections/bookshelf.html` + `static/css/bookshelf.css` — the only chrome that stays Library-owned (the bookshelf landing section). `layouts/_partials/hooks/head-end.html` is Shell's copy plus the bookshelf `<link>` (tagged `TODO(Shell)`).
- Layouts, assets, static, and scripts otherwise: live in the Shell module (`CAI-Field-Manual-Shell`). See that repo's README for what it exports.

## Clone and render locally

Prerequisites:

- Hugo Extended ≥ 0.168 (`hugo version`)
- Go ≥ 1.27
- **Dart Sass on PATH** (`sass --version` must print Dart Sass; `brew install sass/sass/sass` or `npm i -g sass`) — the Shell module's SCSS pipeline uses `transpiler: dartsass`; builds fail before the gate's WARN filter without it.

```sh
git clone --recurse-submodules https://github.com/rh-aiservices-bu/CAI-Field-Manual-Library.git
cd CAI-Field-Manual-Library
hugo server
```

With a path in `baseURL` (`https://rh-aiservices-bu.github.io/CAI-Field-Manual-Library/`), the dev server serves at `http://localhost:1313/CAI-Field-Manual-Library/` — a 404 at the root is expected, not a broken build. Builds use the Shell commit pinned in `hugo.yaml` `module.imports[].version`, not Shell `main` — deliberate, commit-precise; the `hugo.work` workspace replacement below overrides it only for testing unpushed Shell work.

### Updating the manuals

Manual pins move automatically via the hourly [`hourly-publish.yaml`](.github/workflows/hourly-publish.yaml) workflow, which refreshes the pins, gates the aggregated build, and only then commits the pins and deploys; a failing gate leaves main and the live site untouched and emails the workflow creator. If a deploy ever fails after pins were pushed, re-run via `gh workflow run hourly-publish.yaml` (dispatch gates and deploys even without pin changes). To pin manually:

```sh
git submodule update --remote external/manual-hello-world  # or manual-3-migration, manual-maas
git add external/manual-hello-world && git commit -m "chore: bump manual-hello-world"
```

Submodule repos own their content — never edit mounted content in this repo.

### Updating the shell

Edit the Shell import's `version:` hash in `hugo.yaml` to the new commit (e.g. `git ls-remote https://github.com/rh-aiservices-bu/CAI-Field-Manual-Shell main | cut -c1-12`), then re-run the gate:

```sh
go get github.com/rh-aiservices-bu/CAI-Field-Manual-Shell@<hash> && go mod tidy
```

Hugo resolves the pinned commit directly, so the build passes even with stale `go.mod`/`go.sum`; the `go get` keeps them in sync. Don't use `hugo mod get` / `hugo mod tidy` — they rewrite the pin to Shell `main`.

To verify a Shell working-tree change locally (before it is pushed), add a workspace replacement — create `hugo.work` in this repo root containing:

```
go 1.27.3

use .

replace github.com/rh-aiservices-bu/CAI-Field-Manual-Shell => ../CAI-Field-Manual-Shell
```

Then run with the workspace:

```sh
HUGO_MODULE_WORKSPACE=hugo.work hugo server
```

Verify with `hugo config` / `hugo mod graph` that the replacement is active. `hugo.work` is gitignored — never commit it.

### Publication gate

One implementation, used locally and by CI (both the deploy build and PR checks run it). Locally the raw build is enough:

```sh
hugo --cleanDestinationDir --gc --minify --environment production --printPathWarnings 2>&1 | grep -i warn | grep -v "css.Sass: libsass"
```

No output = clean. The build must succeed and the only WARN allowed is the known oink libsass deprecation; anything else fails.

In CI the gate runs via the Shell checkout:

```sh
shell/scripts/publication-gate.sh --baseURL "https://example.invalid/docs/"
```

Optional contract check (`check-contracts.sh`) validates dual-consumption
promises against a site's own `content/`. The Library owns no `content/`
(deleted) so `public/` has no `content/**` counterpart and the check's
vacuous-pass guard (`0 manual pages matched → FAILED`) is expected.
Run it in the content repos instead (after their production build):

```sh
(cd ../CAI-Field-Manual-Hello-World && hugo --cleanDestinationDir --gc --minify --environment production && sh ../CAI-Field-Manual-Shell/scripts/check-contracts.sh)
```

Note: `--panicOnWarning` is intentionally not used. The pinned theme version still transpiles some SCSS with Hugo's LibSass path (deprecated in Hugo ≥ 0.153), which emits a WARNING; the gate asserts no unexpected WARNs beyond that one. The Shell module's `head-css.html` uses `transpiler: dartsass` so this warning is gone in builds via the Shell, but the filter stays for compatibility.

## One-time admin

- **GitHub Pages**: Settings → Pages → Source = **GitHub Actions**.

## Report issues, propose changes

- **Issues** — file a [GitHub Issue](https://github.com/rh-aiservices-bu/CAI-Field-Manual-Library/issues) with the page URL (or `content/` path) and what you hit.
- **Changes** — PRs to `main` are welcome. Library-owned infra lives in `hugo.yaml` (cascade, site config, mounts) and `data/home.yaml` (landing composition — every card URL must point into a mount); authored content lives **only** in manual repos — edit there, then bump the submodule. Run the publication gate before opening a PR. Merges to `main` deploy automatically via GitHub Actions.

## Disclaimers

- **Orientation, not a support statement.** Commands and versions reflect what worked during field work when the manuals were written; verify against the current official product documentation.
- **Not official product documentation.** Field notes from prototypes, maintained by project contributors.
- **Prototype.** Structure, pages, and URLs are under development and may change or move.
