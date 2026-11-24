# zeroclaw-patches

Patch queue on upstream ZeroClaw releases. Layout, patch list, upgrade steps: README.md.

## Rules
- **Main only**: commit + push, no PRs, branches or worktrees.
- **Build only in CI**, never on the zeroclaw VM: a release build OOMs it. CI (`build.yaml`, runs only on `release.env`, `patches/**`, the workflow) applies, tests, builds (~30-45 min); main publishes `v<ZEROCLAW_VERSION>-p<PATCH_RELEASE>` only when green and the tag doesn't exist yet.
- **Failed build** releases nothing: fix + push, no bump.
- **Edit a patch**: clone the upstream tag in /tmp, `git am patches/*.patch`, change, `git format-patch -o patches/ v<version>`, bump `PATCH_RELEASE` (existing tag = publish skipped).
- **New patch**: add to `patches/series` + README Patches list; its test names into the `Test patched code` filter in `build.yaml`.
- **After release**: PR in Rutger505-Org/homelab-infrastructure bumping `zeroclaw_version` + `zeroclaw_sha256` (role zeroclaw).
