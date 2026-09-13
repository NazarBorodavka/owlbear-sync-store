# owlbear-sync-store

A personal ZimaOS/CasaOS app store, kept separate from the official
community app store, so the [owlbear-sync](https://github.com/NazarBorodavka/owlbear-sync)
tracker can be installed and updated through the normal ZimaOS App Store UI
instead of manual `docker compose down && docker compose up` cycles.

## How it stays up to date

`owlbear-sync`'s CI (`.github/workflows/docker-publish.yml`) builds and pushes
a new image on every push to `master`, then fires a `repository_dispatch`
event at this repo. `update-app.yml` here receives it, bumps the pinned image
tag and version in `Apps/OwlbearTracker/docker-compose.yml`, and pushes to
`main`. That push triggers `release-store.yml`, which rebuilds the store
bundle (via `IceWhaleTech/build-appstore-action`) and publishes it to the
`gh-pages` branch. ZimaOS periodically re-checks the store URL and shows an
"Update available" prompt once the version differs from what's installed.

The image tag is deliberately never `latest`/`master` — CasaOS/ZimaOS detect
updates by comparing that exact string, and a floating tag never changes, so
it would silently stop prompting.

## Adding this store to ZimaOS

1. Once GitHub Pages is live for this repo (Settings → Pages, source:
   `gh-pages` branch — usually auto-configured the first time
   `release-store.yml` runs successfully), the store manifest is at:
   `https://nazarborodavka.github.io/owlbear-sync-store/store.json`
2. In ZimaOS: **App Store → the gear/Community-store icon → + → paste that
   URL → refresh.**
3. Install "Owlbear Token Tracker" from the store listing (not via a pasted
   raw compose file) — only store-installed apps get diffed against the
   store for update prompts.

## One-time setup this repo needs

- A GitHub Actions secret is **not** needed here — `update-app.yml` only
  pushes within this same repo, which the default `GITHUB_TOKEN` can already
  do with the `contents: write` permission declared in the workflow.
- The **owlbear-sync** repo needs a fine-grained PAT (scoped to just this
  repo, `Contents: Read and write`) saved as the `STORE_DISPATCH_TOKEN`
  secret, so its workflow can fire the cross-repo `repository_dispatch` —
  the default `GITHUB_TOKEN` in *that* repo's workflow can't reach across to
  a different repo.
