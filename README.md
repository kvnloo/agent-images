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

## Traces into AgentsView

Every agent run should end up in AgentsView. Two pieces:

- **`trace-export`** (in agent-base, `/usr/local/bin`): packs this machine's Claude Code
  (`~/.claude/projects`), Codex (`~/.codex/sessions`, `archived_sessions`) and Hermes
  (`$HERMES_HOME/state.db`, `profiles/*/state.db`, via the SQLite backup API) into
  `~/.trace-exports/trace-<host>-<utc>.tar.gz` with a `manifest.json` (host = `TRACE_HOST`, else the
  Boat sandbox id, else hostname, plus a sha256 for every file). Only session files are copied.
  Credential-like names (`.credentials*`, `auth.json`, `.env*`, `*token*`, `*secret*`, keys) are
  deleted, and the script fails closed if any remain. Any secret-named env value found inside a file
  is overwritten with `X`s. `lane` runs it on exit (`LANE_TRACE_EXPORT=0` to skip); run it by hand
  any time.
- **`tools/boat-trace-pull <sandbox-id>`** (run on the Mac Studio **before `boat stop`**): installs
  `trace-export` on the sandbox if it is missing, then runs it. Boat sandboxes run Boat's own Ubuntu
  image, not agent-base (`boat new` has no image option), so the script is shipped over `boat exec`.
  The tool copies the tarball home (`boat scp`, falling back to base64 chunks over `boat exec`),
  checks its sha256, and refuses anything over 5 GB. It unpacks into the Studio AgentsView data dir
  (`~/.local/state/agentsview/boat-traces/`) and registers those roots once in that dir's
  `config.toml` as Claude/Codex `homes` and Hermes `dirs`. Then it syncs and labels each session
  `boat=<id> boat_name=<name>`.
  Re-pulls are idempotent: files land at stable paths and sessions keep their native ids.
  Find them with `agentsview session list --label boat=<id> --include-one-shot`. AgentsView files
  headless `claude -p` and `codex exec` runs as child sessions, so search needs
  `--include-children` (and `--include-one-shot`) to show them.

Canonical store: groot's AgentsView (`/mnt/zer0models/sft-svlm/data/agentsview`) pulls the Studio's
data dir over ssh (`[[remote_hosts]] studio`). So Boat sessions imported on the Studio reach groot on
its next 30-minute `agentsview-sync` run, with no groot-side step.

```sh
boat exec <id> -- bash -lc 'claude -p "..."'     # or boat prompt / lane
tools/boat-trace-pull <id>                       # export -> copy -> import -> label
boat stop <id>
```

## The boat profile (unattended, no approval prompts)

Nobody is there to approve commands on a Boat box, so the first agent run installs
[`images/base/profiles/boat`](images/base/profiles/boat) into `$HERMES_HOME`:

- `SOUL.md`: the Hermes starter persona ([teknium1/hermes-starter-profile](https://github.com/teknium1/hermes-starter-profile), MIT),
  rewritten for unattended work: never wait for input, record assumptions, never claim unverified success,
  finish with a `REPORT.md`.
- `config.yaml`: `approvals.mode: off` plus `single_query_mode` / `cron_mode` / `unattended_mode: approve`, so
  no prompt can stall a run. No `clarify` tool. Memory and skill writes go straight through. Hermes hardline
  floors and the `approvals.deny` globs still block under yolo: force/mirror/delete pushes, pushes to
  `main`/`master`, `gh repo delete`, `rm -rf /` or `~`. Secret redaction stays on.

It only applies when an agent starts (`lane` runs `agent-setup`); using the image for builds or CI without
an agent changes nothing. `AGENT_PROFILE=none` keeps upstream Hermes defaults. The file is public: keep hosts,
accounts and personal paths out of it.

## Add an image

`images/<repo>/Dockerfile` starting `FROM ${BASE}` with `ARG BASE=ghcr.io/kvnloo/agent-base:latest`,
then add `<repo>` to the matrix in `.github/workflows/build.yml`.
