# Repository Guidance

Hugo Extended >= 0.168 + Go >= 1.27 aggregator site (theme via Hugo modules, no Node.js). README.md is the source of truth for setup and architecture.

This repo is the **CAI Field Manual Library** — the aggregated shelf that owns the landing (`data/home.yaml` bookshelf/cards/faq/cta), the `hugo.yaml` site config (including the `cascade:` book-segregation lever), and three submodule mounts. It owns **no authored content**: every page is mounted from a manual repo. It imports [`CAI-Field-Manual-Shell`](https://github.com/rh-aiservices-bu/CAI-Field-Manual-Shell) as a Hugo module for look-and-feel and rules — propose shell/chrome changes there, not here.

## Before you push: the publication gate (mandatory)

There is no test suite. The only correctness check is the production build:

```sh
hugo --cleanDestinationDir --gc --minify --environment production \
  --printPathWarnings 2>&1 | grep -i warn | grep -v "css.Sass: libsass"
```

No output = clean. Build failure or any other WARN = not done. Fix and re-run. Note the root 404 is expected: with a path in baseURL the dev server serves at `http://localhost:1313/CAI-Field-Manual-Library/`.

In CI the gate runs via `shell/scripts/publication-gate.sh` (see `.github/workflows/github-pages.yaml`); locally the raw pipeline above works without the sibling checkout.

## Contract check (manual repos only)

Validation of dual-consumption promises (variant/customize axes, status-gate
markers, markdown label lines) runs **against `content/`**. The Library owns
no `content/` — it aggregates via submodule mounts — so `content/` is
deleted and `public/` has no `content/**` counterpart for the check to
enumerate. The vacuous-pass guard at the end of the script
(`pages -eq 0 → fail=1`) fails in that case, which is expected.

Run the check in the content repos that own their pages:

```sh
(cd ../CAI-Field-Manual-Hello-World && hugo --cleanDestinationDir --gc --minify --environment production && sh ../CAI-Field-Manual-Shell/scripts/check-contracts.sh)
```

Requires `public/` from the production build and reads `docs_section` from that site's `hugo.yaml`; checks that every variant axis/option and customize token used in its `content/` is declared in its `hugo.yaml`. The build-time shortcode guards (`customize.html`/`variant.html: warnf`) still fire in manual repos' publication gates.

## Content rules

- Library-owned infra: `data/home.yaml` drives the landing composition, but every card URL must point into a mount. The book-segregation lever — `sidebar_expanded: true` site-wide + per-section `sidebar_root_for: self` on `target: {kind: section, path: /*}` — **do not drop** it; it is what segregates each manual's sidebar from the others — lives in `hugo.yaml`'s top-level `cascade:` and is the only aggregator-owned config that touches page rendering (the aggregator owns no `content/` at all: every page comes from a submodule mount).
- Mounted manuals are **read-only here**: `external/manual-hello-world` → `content/hello-world`, `external/manual-3-migration` → `content/rhoai-2-25-to-3-5`, `external/manual-maas` → `content/guide`. Edit in their own repos, then bump the submodule (`git submodule update --remote external/manual-<name>` + commit). Author doctrine (`why-this-approach`, `writing-the-field-manual`, conventions/examples/write-a-manual) lives in `CAI-Field-Manual-Hello-World` under `content/hello-world/what-is-a-field-manual/` — not here.
- Variant axes: five declared in this repo's `hugo.yaml` `params.variants` (`version`/`mode` Library-owned; `deploy`/`env`/`kueue` exist for the mounted Migration manual, merged verbatim from that manual repo's config). Never delete or rename an axis; never delete a variant block in any manual.
- Customize: `user`/`cluster` declared in `params.customize` here; mounted manuals (hello-world, MaaS) use `{{< customize … >}}` tokens against them.
- `hugo.yaml` also declares the `external/manual-hello-world/static/visualizers` mount for the hello-world Flow page's embedded visualizer.

## Do not touch without re-diffing

Chrome files live in the Shell repo (pinned to oink v1.4.0). If you propose shell changes there, these files are pinned oink copies — each carries a provenance comment naming its one deliberate change. Any edit beyond that change requires re-diffing against `~/go/pkg/mod/github.com/pgsty/oink@v1.4.0/` there:

`layouts/docs/baseof.html`, `layouts/_partials/shell/sidebar-tree.html`, `layouts/_partials/pager-state.html`, `assets/js/action-registry.js`, `assets/js/page-actions.js`, `layouts/_partials/actions/context.html`, `layouts/_partials/actions/title-menu.html`. The Shell also ships a fixup `layouts/_partials/favicons.html`.

Local overrides — each shadows Shell (with `TODO(Shell)` header, re-diff on module bumps):

- `layouts/_partials/hooks/head-end.html` — Shell's copy plus the bookshelf `<link>` (the bookshelf is Library-only; a generic landing-section hook in Shell would let this override be deleted).
- `layouts/_shortcodes/visualizer.html` — kept in Library for the mounted hello-world Flow page; visualizer chrome is inlined in the shortcode so Library does not shadow `assets/scss/_styles_project.scss` (Shell owns that file outright). Delete this shortcode once Shell ships its own.
- `layouts/_partials/shell/root-menu-roots.html` — pinned copy of `oink@v1.4.0` `layouts/_partials/shell/root-menu-roots.html`; Library-only change is explicit shelf order `Hello World → RHOAI 2.25 → 3.5 migration → MaaS` via `params.book_order` in `hugo.yaml` instead of `ByWeight` (front-matter `weight` cannot be overridden via `cascade:` — without this the weights `MaaS(0) → Migration(5) → Hello(10)` would invert the requested order). Re-diff against the oink source on bumps; delete the reordering (restore plain `ByWeight`) once Shell ships a param-driven or weight-aligned upstream ordering. Upstream fix alternative: align weights in the three manual repos' `_index.md` and re-pin submodules.

Other Library-unique keeps with no Shell counterpart: `layouts/_partials/landing/sections/bookshelf.html` + `static/css/bookshelf.css` (driven by `data/home.yaml` `sections`/`bookshelf`).

Sanctioned free-to-edit here: `hugo.yaml` (site identity, variants, customize, mounts, top-level `cascade:`), `data/home.yaml` (landing composition — every card URL must point into a mount), submodule pins.

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
