# zeroclaw-patches

Patch queue on top of [zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw) releases. Not a fork: only `.patch` files plus the upstream version they apply to. CI checks out the upstream tag, applies the patches, builds the same `x86_64-unknown-linux-gnu` archive upstream ships (binaries + `web/dist`) and publishes it as a GitHub release.

The `zeroclaw` role in [homelab-infrastructure](https://github.com/Rutger505-Org/homelab-infrastructure) installs from these releases (`zeroclaw_version`, `zeroclaw_sha256`).

## Layout

- `release.env`: `ZEROCLAW_VERSION` (upstream tag without `v`) and `PATCH_RELEASE`. Release tag: `v<ZEROCLAW_VERSION>-p<PATCH_RELEASE>`.
- `patches/series`: patches in apply order.
- `patches/*.patch`: `git format-patch` output against the upstream tag.

## Patches

- `0001-gateway-ws-model-provider.patch`: `/ws/chat?model_provider=<family>.<alias>` sets the model for that conversation only, without touching config. Used by [someclaw](https://github.com/Rutger505/someclaw)'s model picker. Drop it once upstream has an equivalent.
- `0002-daemon-fast-shutdown.patch`: SIGTERM stops the daemon in milliseconds: no unbounded wait for running turns (a tool waiting on the restart hung it until systemd's SIGKILL), no 500 ms grace on the status writer, runtime shutdown with a timeout. Reload unchanged.
- `0003-gateway-ws-ask-user.patch`: `ask_user` works on `/ws/chat` (channel `wss`), free-form and with choices: gateway sends `{type:"ask_user", request_id, question, choices, timeout_secs}`, client answers `{type:"ask_user_response", request_id, answer}`. Timeout/detach = tool error. Used by someclaw's question card.
- `0004-gateway-ws-attach.patch`: a second socket on a session follows its running turn: `{type:"attach"}` → `{type:"attached", running, turn_id, replayed}`, the turn's frames so far (streamed deltas merged), then live up to `done`/`aborted`/`error`. Read-only (`ATTACHED_READ_ONLY`). Used by someclaw to show steps after its own restart mid-turn.

## Workflow

Commit straight to main: CI applies the patches, tests and builds, and publishes the release only when green (a failed build releases nothing, so the tag stays free). Then bump `zeroclaw_version` and `zeroclaw_sha256` in homelab-infrastructure.

New upstream version:

1. Set `ZEROCLAW_VERSION`, `PATCH_RELEASE=1`.
2. Patch doesn't apply: clone upstream at the tag, `git am patches/*.patch`, fix, `git format-patch -o patches/ v<version>`.
3. Push to main, check the build.

Changing a patch: edit it the same way, bump `PATCH_RELEASE` (main skips publishing when the tag already exists).

Building on the zeroclaw VM is off limits: a release build runs it out of memory.
