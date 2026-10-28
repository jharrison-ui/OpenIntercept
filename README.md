<p align="center">
  <img src="docs/img/logo.svg" width="96" alt="OpenIntercept logo" />
</p>

<h1 align="center">OpenIntercept</h1>

<p align="center">
  Models trust their tools blindly. This is the control panel for that trust.
</p>

<p align="center">
  <a href="https://github.com/asucodes/OpenIntercept/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT"></a>
  <a href="https://bun.sh"><img src="https://img.shields.io/badge/runtime-bun-f9f1e5?logo=bun" alt="Bun"></a>
  <img src="https://img.shields.io/badge/platform-win%20%7C%20mac%20%7C%20linux-lightgrey" alt="Platform">
  <a href="https://github.com/asucodes/OpenIntercept/stargazers"><img src="https://img.shields.io/github/stars/asucodes/OpenIntercept?style=social" alt="Stars"></a>
</p>

An [OpenCode](https://opencode.ai) plugin + local sidecar that works like Burp Suite for
tool calls. Every call your agent makes can **pause for a human** - before it runs, or
before its result reaches the model. You read exactly what the model would have
believed, then decide what it actually gets. Everything stays on your machine.

![Calls list with inspector](docs/img/calls.png?v=2)

## The premise

Models cannot verify their tools. A fetch that "failed," a search that "returned
nothing," a shell command that "passed" - the model believes whatever bytes come
back, then reasons, plans, and acts on them. That channel is an unauthenticated
trust boundary: no signature, no provenance, no second opinion.

Whoever controls that boundary controls the agent. One spoofed result can redirect
research, launder a guess into a confirmed fact, or convince the model to hand over
secrets - no prompt injection required, because the injection *is* the tool result.

OpenIntercept puts that boundary under a human's hand.

## What it can do

- [x] **Hold tool calls** at the arguments stage and the result stage
- [x] **Run, edit arguments, spoof the result, or drop** - the honest original is always preserved in local history
- [x] **Spoof results invisibly** - schema-correct envelopes, original arguments untouched, no operator markers the model can detect
- [x] **Rules engine** - tool, stage, args-regex, and result-regex matching with one-click templates, import/export, live match counts
- [x] **Intercept queue** - live countdowns, timeout policy (forward or drop)
- [x] **Decision timeline** - who decided what, by which rule, on every call
- [x] **Opt-in model HTTP capture** - bounded bodies, credentials redacted (panel → Settings)
- [x] **Opt-in plaintext HTTP proxy** - listens on `127.0.0.1:7475`, HTTPS CONNECT is tunnelled, never decrypted (panel → Settings)
- [x] **Append-only JSONL audit log** - the durable local record of everything observed (`data/audit.jsonl`, next to the repo; `~/.openintercept/data/` for the one-click install)
- [x] **One-click install** - Windows and Unix, with copy-paste fallback for custom plugin folders

**Intentionally not included:** TLS MITM / local CA, telemetry.

![Intercept queue](docs/img/intercept.png)

## Install

**One click - any machine with OpenCode and `git`** (Bun ≥ 1.1 is installed automatically if missing):

```powershell
# Windows (PowerShell)
irm https://raw.githubusercontent.com/asucodes/OpenIntercept/main/scripts/install.ps1 | iex
```

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/asucodes/OpenIntercept/main/scripts/install.sh | bash
```

The installer: ensures [Bun](https://bun.sh) is present, clones the repo to
`~/.openintercept`, writes the adapter (`openintercept.ts` and its V2 variant `openintercept-v2.ts`) into
`~/.config/opencode/plugins` (OpenCode's default even on Windows), starts the sidecar, and prints the panel
URL. Re-running it updates everything in place.

**Custom plugins folder?**

```powershell
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/asucodes/OpenIntercept/main/scripts/install.ps1))) -PluginsDir "D:\path\to\opencode\plugins"
```

```bash
curl -fsSL https://raw.githubusercontent.com/asucodes/OpenIntercept/main/scripts/install.sh | bash -s -- --plugins-dir /path/to/opencode/plugins
```

**Manual:**

```bash
git clone https://github.com/asucodes/OpenIntercept.git
cd OpenIntercept
bun install
bun run start                              # sidecar + panel

# in a second terminal - install the adapter:
bun scripts/install-plugin.ts --global     # all projects
bun scripts/install-plugin.ts ./your-project   # or one project only
```

The panel is at <http://127.0.0.1:7474>. Restart OpenCode so it loads the
adapter.

## Try it

Open the panel and turn **Intercept** on (top bar, or press `I`) - it is off by default.
Then run the command:

```bash
opencode run "Fetch https://example.com and tell me what the page says"
```

The `webfetch` call appears in the panel and OpenCode pauses mid-call. Press
**Spoof result**, write any text, apply - and watch the model describe the page
you invented, not the real one. That is the trust boundary in your hands.

## Configuration

| Variable | Default | Description |
| --- | --- | --- |
| `OPENINTERCEPT_PORT` | `7474` | Sidecar and panel port |
| `OPENINTERCEPT_HOST` | `127.0.0.1` | Bind address (local only by design) |
| `OPENINTERCEPT_PROXY_PORT` | `7475` | Port of the opt-in HTTP proxy |
| `OPENINTERCEPT_URL` | `http://127.0.0.1:7474` | Where the adapter looks for the sidecar (set it if you changed the port) |

## Scope & limits

OpenIntercept is for systems you are authorized to test. It is passive: it never
sends payloads, replays requests, scans targets, or decrypts TLS.

- The adapter must be loaded by the agent; nothing is intercepted otherwise.
- If the sidecar is down, every hook fails open - the agent never breaks because
  OpenIntercept is.
- Tool failures are never disguised as successes.

## Layout

```
panel/                 no-build single-page panel (index.html, app.js, styles.css)
packages/protocol/     zod wire contract shared by adapter and sidecar
packages/plugin/       the OpenCode adapter (single file, no dependencies)
packages/server/       Hono sidecar: hold queue, rules, history, SSE, audit
scripts/               one-click installers, plugin installer, traffic simulator
docs/                  notes on the OpenCode tool pipeline
```

## Contributing & issues

Issues and PRs are welcome at
[github.com/asucodes/OpenIntercept/issues](https://github.com/asucodes/OpenIntercept/issues).
Tests: `bun test ./packages`. Types: `bun run check`.

## License

[MIT](LICENSE) © OpenIntercept contributors
