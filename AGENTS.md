# Repository Guidance

Hugo Extended >= 0.168 + Go >= 1.27 aggregator site (theme via Hugo modules, no Node.js). README.md is the source of truth for setup and architecture.

This repo is the **CAI Field Manual Library** — the aggregated shelf that owns the landing (`why-this-approach`, `writing-the-field-manual`), the `data/home.yaml` bookshelf/cards/faq/cta, and three submodule mounts. It imports [`CAI-Field-Manual-Shell`](https://github.com/rh-aiservices-bu/CAI-Field-Manual-Shell) as a Hugo module for look-and-feel and rules — propose shell/chrome changes there, not here.

## Before you push: the publication gate (mandatory)

There is no test suite. The only correctness check is the production build:

```sh
hugo --cleanDestinationDir --gc --minify --environment production \
  --printPathWarnings 2>&1 | grep -i warn | grep -v "css.Sass: libsass"
```

No output = clean. Build failure or any other WARN = not done. Fix and re-run. Note the root 404 is expected: with a path in baseURL the dev server serves at `http://localhost:1313/CAI-Field-Manual-Library/`.

In CI the gate runs via `shell/scripts/publication-gate.sh` (see `.github/workflows/github-pages.yaml`); locally the raw pipeline above works without the sibling checkout.

## Contract check (optional, needs sibling clone)

After a production build, validate dual-consumption contracts against the sibling Shell checkout:

```sh
sh ../CAI-Field-Manual-Shell/scripts/check-contracts.sh
```

Requires `public/` from the production build and reads `docs_section` from this repo's `hugo.yaml`; checks that every variant axis/option and customize token used in content is declared in this repo's `hugo.yaml`.

## Content rules

- Library-owned content: `content/_index.md` (home cascade — `sidebar_expanded: true` site-wide + per-section `sidebar_root_for: self` on `target: {kind: section, path: /*}` — **do not drop** the per-section rule; it is what segregates each manual's sidebar from the others), `content/why-this-approach.md`, `content/writing-the-field-manual/**`, `data/home.yaml`.
- Mounted manuals are **read-only here**: `external/manual-hello-world` → `content/hello-world`, `external/manual-3-migration` → `content/rhoai-2-25-to-3-5`, `external/manual-maas` → `content/manual-maas`. Edit in their own repos, then bump the submodule (`git submodule update --remote external/manual-<name>` + commit).
- Variant axes: five declared in this repo's `hugo.yaml` `params.variants` (`version`/`mode` Library-owned; `deploy`/`env`/`kueue` exist for the mounted Migration manual, merged verbatim from that manual repo's config). Never delete or rename an axis; never delete a variant block in any manual.
- Customize: `user`/`cluster` declared in `params.customize` here; mounted manuals (hello-world, MaaS) use `{{< customize … >}}` tokens against them.
- `hugo.yaml` also declares the `external/manual-hello-world/static/visualizers` mount for the hello-world Flow page's embedded visualizer.

## Do not touch without re-diffing

Chrome files live in the Shell repo (pinned to oink v1.4.0). If you propose shell changes there, these files are pinned oink copies — each carries a provenance comment naming its one deliberate change. Any edit beyond that change requires re-diffing against `~/go/pkg/mod/github.com/pgsty/oink@v1.4.0/` there:

`layouts/docs/baseof.html`, `layouts/_partials/shell/sidebar-tree.html`, `layouts/_partials/pager-state.html`, `assets/js/action-registry.js`, `assets/js/page-actions.js`, `layouts/_partials/actions/context.html`, `layouts/_partials/actions/title-menu.html`. The Shell also ships a fixup `layouts/_partials/favicons.html`.

Local overrides — the only two in Library (both with `TODO(Shell)` headers, re-diff on module bumps):

- `layouts/_partials/hooks/head-end.html` — Shell's copy plus the bookshelf `<link>` (the bookshelf is Library-only; a generic landing-section hook in Shell would let this override be deleted).
- `assets/scss/_styles_project.scss` — Shell's copy plus the viz block for Hello-World's Flow visualizer (`TODO(Shell): move visualizer shortcode + styles to Shell`).
- `layouts/_shortcodes/visualizer.html` — kept in Library for the mounted hello-world Flow page; Shell has no counterpart yet.

Other Library-unique keeps with no Shell counterpart: `layouts/_partials/landing/sections/bookshelf.html` + `static/css/bookshelf.css` (driven by `data/home.yaml` `sections`/`bookshelf`).

Sanctioned free-to-edit here: `hugo.yaml` (site identity, variants, customize, mounts), `data/home.yaml`, `content/` (Library-owned pages + cascade), submodule pins.

## Updating the shell

Edit the Shell import's `version:` hash in `hugo.yaml` to the new commit (e.g. `git ls-remote https://github.com/rh-aiservices-bu/CAI-Field-Manual-Shell main | cut -c1-12`). `version:` accepts semver, branch names, or commit hashes (Hugo >= 0.150) — a commit hash is deliberate, commit-precise, no Shell tagging. Hugo resolves the pinned commit directly, so the build passes even with stale `go.mod`/`go.sum`; to keep them in sync after a bump run `go get github.com/rh-aiservices-bu/CAI-Field-Manual-Shell@<hash> && go mod tidy` (don't use `hugo mod get` — it rewrites `version:` to Shell `main`). Re-run the publication gate.

For testing unpushed Shell work, use a `hugo.work` workspace replacement (see README.md); `hugo.work` is gitignored, never commit it.

## Updating the manuals

Pins move automatically via the hourly `refresh-submodules.yaml` workflow. To pin manually:

```sh
git submodule update --remote external/manual-hello-world
git add external/manual-hello-world && git commit -m "chore: bump manual-hello-world"
```

Submodule repos own their content; bump only the pin here.

## Verification for JS changes

Interactive behavior (variant selector, customize tokens, search, page actions) lives in the Shell module's `assets/js/` + `assets/css/` + `layouts/_partials/` — verify in a real browser (`hugo server` → `http://localhost:1313/CAI-Field-Manual-Library/`), not just the build.

## One-time admin

- **GitHub Pages**: Settings -> Pages -> Source = **GitHub Actions**.
