# agent-images

Prebuilt containers for agent lanes (Boat sandboxes, local Docker): **Hermes Agent + Claude Code + the
DirectSDK provider**, plus one image per repo we work on with that repo's toolchain baked in.
Cataloged in [kvnloo/home](https://github.com/kvnloo/home) (`map/config.yaml`, area `agent-images`).

| image | for | adds |
|---|---|---|
| `ghcr.io/kvnloo/agent-base` | everything | `nousresearch/hermes-agent:stable`, Claude Code, DirectSDK plugin, gh, uv, rg, jq |
| `ghcr.io/kvnloo/agent-hermes-android` | kvnloo/hermes-android | Flutter 3.44.0, JDK 17, Android SDK 36 |
| `ghcr.io/kvnloo/agent-dash` | kvnloo/dash | Bun 1.4.3 |
| `ghcr.io/kvnloo/agent-glide` | kvnloo/glide | Python 3.14 via uv |
| `ghcr.io/kvnloo/agent-hermex` | kvnloo/hermex | Ruby + xcodeproj (Linux-side checks; Swift still needs macOS CI) |

Self-sustaining: `build.yml` rebuilds every Monday (new Hermes stable, Claude Code, plugin) and on push.
Public repo, so Actions minutes and GHCR pulls cost nothing. Toolchain versions follow each repo's CI file;
bump the `ARG` when that repo bumps.

## Run a lane

```sh
docker run --rm -e CLAUDE_CODE_OAUTH_TOKEN -v "$PWD/work:/work" -v "$PWD/data:/opt/data" \
  ghcr.io/kvnloo/agent-dash lane /work/PROMPT.md          # prints the answer; session id is in the prompt context
docker run ... ghcr.io/kvnloo/agent-dash lane /work/CONTINUE.md <session-id>   # resume
```

Auth is whatever Claude Code finds: `CLAUDE_CODE_OAUTH_TOKEN` (Boat injects it into every sandbox) or a
mounted `~/.claude`. Keep `/opt/data` on a volume so sessions survive container restarts.

## Add an image

`images/<repo>/Dockerfile` starting `FROM ${BASE}` with `ARG BASE=ghcr.io/kvnloo/agent-base:latest`,
then add `<repo>` to the matrix in `.github/workflows/build.yml`.
