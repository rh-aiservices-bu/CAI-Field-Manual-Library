# Backlog

Working list of pending work for this repo. New items go at the bottom of their priority table.

## Now

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | Fix stale `guide/` references: README.md:15 (`/guide/` link), README.md:34 (mount list), README.md:36 (bookshelf URL list), AGENTS.md:40 (`external/manual-maas → content/guide`) — actual mount is `content/maas`, site serves `/maas/` (verified: no `public/guide/`, `public/maas/` exists) | pending | missed by rename commit a2eac5f |
| 2 | README.md:47 prerequisite contradicts CI: recommends `npm i -g sass`, but npm `sass` is the package f76ec28 removed from CI (wrong for Hugo's dartsass embedded protocol); keep `brew install sass/sass/sass`, drop the npm variant or change to `npm i -g sass-embedded` | pending | also trim "Dart Sass on PATH" claim if dartsass embedded is the only consumer |
| 3 | AGENTS.md:3 "no Node.js" is stale: CI runs `npm install -g sass-embedded` (github-pages.yaml:68,120) — Node/npm are now build toolchain; update the header line | pending | README.md:7 says "no Node.js" nowhere but AGENTS.md:3 does |
| 4 | CI gate drifts from build: both github-pages.yaml jobs check out CAI-Field-Manual-Shell with NO `ref:` (lines 42-47, 100-105) so `shell/scripts/publication-gate.sh` runs from floating Shell main while the site builds from pinned `1f1471c74947`; pin the checkout `ref:` to the same hash as hugo.yaml `module.imports[].version` | pending | gate script at Shell main currently matches the documented contract |
| 6 | Dead landing data in data/home.yaml: `timeline:` (70-85), `cards:` (33-61), `faq:` (87-94) defined but `sections:` (line 2) renders only hero/bookshelf/cta since 444801c; delete blocks or restore them to `sections:`; cards/faq carry doctrine links that would need a new home if deleted | pending | decision: delete (dead weight) vs restore — reviewer recommends delete |
| 7 | README landing-composition description stale: README.md:36 lists `hero, bookshelf, timeline, cards, faq, cta`; actual active sections are `hero, bookshelf, cta` (data/home.yaml:2); also README.md:29 says llms-full.txt bundles exist only under `rhoai-2-25-to-3-5/` but `public/maas/llms-full.txt` also ships | pending | fix wording to match data/home.yaml sections and list both llms-full.txt locations |
| 8 | AGENTS.md oink baseline drift: rediff doctrine cites `oink@v1.4.0` (AGENTS.md:47,55) but go.mod resolves `github.com/pgsty/oink v1.5.1` via the Shell module graph; all Library pinned layouts verified clean against v1.4.0 — decide whether Shell's oink bump moves the re-diff baseline, then update AGENTS.md | pending | informational unless Shell bump was meant to move the baseline |
| 9 | Bookshelf item "MaaS" lacks `subtitle:` (data/home.yaml:29-31) while Hello World/Migration carry `subtitle: CAI` — renders without the imprint line; add `subtitle: CAI` for consistency | pending | cosmetic |
| 10 | `mode` variant axis is dead: declared in Library + hello-world hugo.yaml (options gitops/manual/scripted) but zero `{{< variant mode … >}}` usage across all three manuals' content — renders as a permanently-empty selector axis on the site; repo rule forbids deleting axes, so either author mode-tagged content or retire the axis via the documented removal procedure | pending | decision needed: populate vs retire |
| 11 | Bump external/manual-3-migration pin: 1 commit behind origin/main (`48cb300`, CI-only — removes omp.yml + AGENTS.md omp section, no content/config impact); `git submodule update --remote external/manual-3-migration` + commit | pending | harmless staleness, doc accuracy only |

## Next

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | Pin the sass-embedded host version in both workflows (`npm install -g sass-embedded@<version>`) — currently floats on npm latest; nothing in-repo constrains the dartsass host | pending | low urgency: unpinned but working |
| 2 | Add `/content/` to `.gitignore`: root `content/maas/` residue is untracked but unignored, and the contract-check transiently recreates `content/`; delete the empty dir with it (pairs with Next 5) | pending | one-line .gitignore change |
| 3 | Add `actions/cache` for `HUGO_CACHEDIR` (github-pages.yaml:28) or delete the dead env var — no step saves/restores it now | pending | minor CI speed |
| 4 | Pin GitHub Actions to commit SHAs instead of major tags (`checkout@v7`, `setup-go@v7`, `configure-pages@v6`, `peaceiris/actions-hugo@v3`, `upload-pages-artifact@v5`, `deploy-pages@v5`); peaceiris fetches Hugo at run time | pending | supply-chain hardening, optional |
| 5 | Delete stray empty untracked `content/maas/` dir at repo root (AGENTS.md says content/ is deleted; empty, not in git) | pending | housekeeping |
| 6 | In the MaaS submodule's own repo (read-only here): move Docs/backlog.md Now #1 to Done (end state already executed on pinned HEAD — repo renamed, baseURL/docs_section/content/maas done) and fix pre-rename `content/guide/` path references in Docs/upstream-sync-questions.md | pending | submodule-owned; edit there, then bump pin here |

## Later

| # | Item | Status | Notes |
|---|------|--------|-------|
| | | | |

## Done

| 1 | Repo review via three scout sub-agents (layouts provenance vs oink@v1.4.0 + Shell@1f1471c74947; submodule pins/variant axes; CI workflows vs README/AGENTS claims) + direct reads; findings logged as Now 1-11, Next 1-6 | 2026-11-19 | layout pins verified clean (no extra divergence); pins, mounts, token contracts, gitignore coverage verified accurate |
| 5 | refresh-submodules.yaml pushes pin bumps that never redeploy + never gate: bot push with GITHUB_TOKEN does not trigger github-pages.yaml (GitHub anti-recursion; no PAT in workflow) despite header comment claiming it does (refresh-submodules.yaml:3-6); and no publication gate runs before bumping, so a broken manual commit on origin/main lands as a pin unverified | 2026-11-20 | fixed by hourly-publish.yaml (hourly-publish-pipeline branch; live on merge) — gate runs before any pin lands; anti-recursion moot because the hourly run deploys its own build; refresh-submodules.yaml deleted |
| | | | |
