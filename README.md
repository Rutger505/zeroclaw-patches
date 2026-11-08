# zeroclaw-patches

Patch queue on top of [zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw) releases. Not a fork: only `.patch` files plus the upstream version they apply to. CI checks out the upstream tag, applies the patches, builds the same `x86_64-unknown-linux-gnu` archive upstream ships (binaries + `web/dist`) and publishes it as a GitHub release.

The `zeroclaw` role in [homelab-infrastructure](https://github.com/Rutger505-Org/homelab-infrastructure) installs from these releases (`zeroclaw_version`, `zeroclaw_sha256`).

## Layout

- `release.env`: `ZEROCLAW_VERSION` (upstream tag without `v`) and `PATCH_RELEASE`. Release tag: `v<ZEROCLAW_VERSION>-p<PATCH_RELEASE>`.
- `patches/series`: patches in apply order.
- `patches/*.patch`: `git format-patch` output against the upstream tag.

## Patches

- `0001-gateway-ws-model-provider.patch`: `/ws/chat?model_provider=<family>.<alias>` sets the model for that conversation only, without touching config. Used by [someclaw](https://github.com/Rutger505/someclaw)'s model picker. Drop it once upstream has an equivalent.

## Workflow

Changes go through a PR: CI applies the patches and builds, so a patch that no longer applies fails there. Merging publishes the release; then bump `zeroclaw_version` and `zeroclaw_sha256` in homelab-infrastructure.

New upstream version:

1. Set `ZEROCLAW_VERSION`, `PATCH_RELEASE=1`.
2. Patch doesn't apply: clone upstream at the tag, `git am patches/*.patch`, fix, `git format-patch -o patches/ v<version>`.
3. PR, green build, merge.

Changing a patch: edit it the same way, bump `PATCH_RELEASE`. A PR whose tag already exists fails on purpose.

Building on the zeroclaw VM is off limits: a release build runs it out of memory.
