# Tachly plugin for Claude Code

The Tachly plugin sends usage data from your local Claude Code sessions to a
Tachly server. There you and your team see Claude Code limits (5 hours and
7 days), tokens, cost, sessions, subagents, skills and MCP servers, and get
Coach tips on how you work.

What you need:

- Claude Code, in the terminal or in the Claude desktop app. Only Claude Code
  is supported today.
- The address of a Tachly server. Today that is a server your team runs itself
  (one Docker Compose stack). A hosted Tachly cloud (tachly.dev)
  is in early access.

## Install

In Claude Code:

```
/plugin marketplace add tachly/tachly-plugin
/plugin install tachly@tachly
```

## Connect

```
/tachly:login
```

Without an address it connects to the hosted Tachly at https://tachly.dev. If
you run your own server, pass its address: `/tachly:login <server>`.

The plugin prints a sign-in link (and a code), opens it in your browser and
waits until you confirm. The token it gets is stored on this machine; you do
not type a password into Claude Code.

What to expect next:

- From now on every Claude Code session is sent in the background. Nothing
  changes in how Claude Code works.
- To import your older sessions too, run `/tachly:sync` (the last 30 days by
  default).
- To collect your limits, run `/tachly:statusline on`.
- `/tachly:status` shows the server, who you are signed in as, the queue, the
  last successful send and the plugin version.

## Commands

| Command | What it does |
|---|---|
| `/tachly:login [server]` | Signs this machine in to a Tachly server through the browser. Without an address it uses the server still saved in the config (for example after the server rejected the token), and if there is none (a new install, or after `/tachly:logout`) the hosted https://tachly.dev. A self-hosted server is always given by its address. |
| `/tachly:status` | Shows the server, person, whether prompt text is collected, last send, whether limits are collected (and when they were last read from the terminal `claude`), telemetry and Coach state, the queue (with dead files and data waiting for other servers) and the plugin version. |
| `/tachly:sync [--days N \| --all]` | Imports older session history (default: the last 30 days). Sessions already on the server are skipped, so running it again is safe. |
| `/tachly:statusline on\|off` | `on` wraps your status line to collect the 5-hour and 7-day limits (your original status line keeps working); `off` restores it as it was. |
| `/tachly:telemetry on\|off` | `on` points Claude Code's OpenTelemetry export (metrics and events) at the server with a personal key; `off` restores your previous settings and revokes the key. No prompt text is sent. |
| `/tachly:init [name or slug] [--force]` | Links the current repository to a project and writes `.tachly.json` at its root (outside a git repository, in the current folder). Without a name it keeps the project already in `.tachly.json`, else it takes the folder name. It also sends the repository's git remote, and the server attaches it to the project. The server creates the project if it does not exist. `--force` replaces a file that points to another project or server, or that cannot be read. |
| `/tachly:coach on\|off\|mute <id>\|mute all` | Turns the Coach line at session start on or off, hides one tip, or turns all tips off. Mutes reach the server with the next send. |
| `/tachly:me` | Opens your Tachly dashboard in the browser, signed in by this device, through a one-time link (valid for about 60 seconds, works once). You confirm with the "Sign in" button in the browser; no password. The link is printed too. |
| `/tachly:logout` | Removes the token and forgets the server, turns plugin telemetry off and revokes its key. The queue is kept. |

`/tachly:init`, `/tachly:sync`, `/tachly:me` and `/tachly:telemetry on` need
you to be signed in.

`/tachly:statusline` and `/tachly:telemetry` change Claude Code's
`settings.json`; the change takes effect in a new Claude Code session.

## How it works

- **Hooks never use the network.** The plugin registers hooks for eight
  Claude Code events (session start and end, prompt, tool use, subagent stop,
  compaction, stop and stop failure). A hook only leaves a small note on disk
  (at session start, at most once an hour, also a record of the machine, the
  Claude account and the plugin version), makes sure the background sender is
  running and returns at once. It always
  exits 0 and never blocks Claude Code. While you are not signed in, hooks do
  nothing.
- **A background sender** reads the new lines of your session transcripts,
  keeps only the usage fields listed under [Privacy](#privacy), and sends them
  to the server you signed in to.
- **The queue survives being offline.** Data waits in a queue on disk until
  the server accepts it. Network errors and temporary server errors are
  retried with growing pauses, and the next hook wakes the sender again. When
  the server rejects the token (401), everything is kept. A file the server
  rejects as invalid (400, or 413 when it cannot be split further) is moved to
  `queue/dead/` and not sent again. Data queued for one server is only ever
  sent to that server. Queue files older than 14 days are dropped.
- **The status line wrapper is optional.** With `/tachly:statusline on` the
  plugin reads the limits that Claude Code passes to the status line, then
  runs your original status line and prints its output unchanged. The
  original command gets 2 seconds; if it takes longer it is stopped. Because
  the Claude desktop app has no status line, the sender also asks the terminal
  `claude` CLI for the limits, at most every 10 minutes while you work (no
  model call, no cost). This needs a terminal `claude` installed and signed in
  on the machine; the binary bundled with the desktop app is never used.
- **Telemetry is optional.** `/tachly:telemetry on` sets Claude Code's
  OpenTelemetry variables in `settings.json` and saves the previous values.
  It never sets the `OTEL_LOG_*` content switches.
- **Projects.** The server assigns each session to a project. The nearest
  `.tachly.json` from the session's working directory upwards wins; without
  one, the server matches the git remote, then the working directory path,
  against the project's patterns. `/tachly:init` writes this file at the
  repository root and links the repository's remote; commit the file so
  sessions from the whole team go to the same project.
- **Coach.** At the start of a session (and only then) a hook can show one
  Coach line. See [Privacy](#privacy) for how it is shown.
- **Data directory.** Everything the plugin keeps is in `~/.tachly`: config,
  queue, state, the log (`log/tachly.log`) and the status line wrapper. On
  macOS the token is in the login keychain; elsewhere it is the file
  `~/.tachly/token`.
- **Environment variables.** `TACHLY_HOME` moves the data directory.
  `TACHLY_NO_BROWSER=1` prints links instead of opening a browser.
  `TACHLY_TOKEN_BACKEND=file` stores the token in a file on macOS too.
  `TACHLY_CLAUDE_BIN` names the `claude` binary used to read limits.
  `CLAUDE_CONFIG_DIR` is respected for Claude Code's own files.

## Updating

`/tachly:status` tells you when a newer version is available. To update:

```
/plugin marketplace update tachly
```

then restart Claude Code.

## Upgrading from the first releases

The first releases were called `pulse` (`pulse@ai-pulse`). Install this plugin as above, restart Claude Code, then
remove the old one:

```
/plugin uninstall pulse@ai-pulse
/plugin marketplace remove ai-pulse
```

Nothing else is needed: the first run moves the data directory `~/.claude-pulse` to `~/.tachly`, so you stay signed
in, the queue is kept, and the status line and telemetry settings are picked up as they are. `.claude-pulse.json`
files in your repositories are still read. The older `PULSE_*` names of the environment variables still work.

While `pulse@ai-pulse` is still enabled, `/tachly:status` says so, and so does the start of a session while you are
signed in. With `TACHLY_HOME` or `PULSE_HOME` set, `~/.claude-pulse` is not moved, so the old plugin keeps its data
directory and keeps sending (from the same directory only when `PULSE_HOME` is set; it does not read `TACHLY_HOME`).
Remove `pulse@ai-pulse` first on such machines.

## Supported platforms

- macOS (Apple silicon and Intel)
- Linux x64 and arm64
- Windows x64. Windows needs Git for Windows (Git Bash); PowerShell-only
  setups are not supported.

On any other platform, or when the binary cannot be run (no exec permission,
or a noexec mount), the hooks stay silent and do nothing, and the commands
print a short message.

## Privacy

`/tachly:login` and `/tachly:status` say it plainly: all Claude Code activity
on this machine is collected, including prompt text and personal projects.

What the plugin collects:

- Usage of your Claude Code sessions: session and project identifiers, model
  names, token counts, timestamps and the names of tools used. For subagents
  and skills it also records the subagent type and the skill name.
- The text of your prompts, but only while prompt collection is on for you
  (it is on by default). Secrets in the text are redacted before it leaves
  your machine, and each prompt is cut to 20,000 characters.
- Your Claude account, read from the named fields of `oauthAccount` in
  `.claude.json`: account id, email address, organization id and name,
  billing type, seat tier, and the plan fields (organization type, rate limit
  tier, plan name and extra usage). Nothing else is read from that file.
- The name of your machine (hostname) and its operating system and CPU type,
  with every batch of data.
- The full path of the working directory of each session, its git branch,
  and the git remote address of your project, without credentials.
- Session metadata: the Claude Code version and how it was started, session
  titles and subagent names (redacted; titles that Claude writes itself only
  while prompt collection is on), the length of each prompt, the error
  messages Claude Code records, the names of MCP servers and tools and of
  slash commands, cost and time totals (cost in USD, durations, lines added
  and removed), and for each tool result its size, whether it succeeded and
  its command name.
- Optionally, your Claude Code limits (5 hours and 7 days), when you turn on
  `/tachly:statusline`.
- Optionally, Claude Code's OpenTelemetry metrics and events, when you turn on
  `/tachly:telemetry`.

What the plugin never collects:

- Claude's responses or its thinking.
- The input of tools (except the subagent type and the skill name).
- The content of tool results (only their size).
- Any field of the session records not described above. Besides the items
  listed, the plugin keeps a few technical fields of each record: record and
  message ids, the parent record and whether it belongs to a subagent, the
  requested and resolved model, the effort level, the stop reason, the
  per-model cost breakdown, the system message type and what triggered a
  compaction.
- Your Claude credentials or anything from the Claude keychain items.
- OpenTelemetry log content: telemetry never turns on `OTEL_LOG_*`. If you
  set one of those switches yourself, `/tachly:telemetry on` warns you and
  leaves it as is; Claude Code then exports that content to the server, and
  the server drops it without storing it.

How it handles your data:

- Hooks never use the network. A separate background process sends the data to
  the server you logged in to. A hook prints nothing except at the start of a
  session, and only while you are signed in. Then it can show a
  `systemMessage` that only you see (the model never does): a notice when the
  old plugin of the first releases is still enabled (see
  [Upgrading](#upgrading-from-the-first-releases)), and one Coach line. The
  Coach line is read from a small file the background process keeps
  (`state/coach.json`), is limited to one tip in any 24 hours and one weekly
  digest, and is on by default. `/tachly:coach off` turns it off;
  `/tachly:coach mute <id>` or `mute all` also tells the server.
- The access token is never printed and never passed on a command line. The
  data directory is private to you (directories 0700, token and backups 0600).
- Only http(s) links are opened: the sign-in link your server returns. `/tachly:me` opens or prints only a link from the configured server's origin; a link to any other address is refused.
- The `claude` process that reads the limits is started without API key or
  session variables, without hooks and without saving a session.

## Troubleshooting

**Nothing shows up on the server.** Run `/tachly:status`. If it says
`Not connected`, run `/tachly:login` (or `/tachly:login <server>` for your own
server). Hooks do nothing while you are signed out, so afterwards run
`/tachly:sync` to import the sessions you missed.

**The server was unreachable.** Nothing is lost: the data stays in the queue
(`/tachly:status` shows how many files) and is sent once the server answers.
Queue files older than 14 days are dropped. Details are in
`~/.tachly/log/tachly.log`.

**"The server rejected this device's token".** The server no longer accepts
this device (for example, it was revoked). The data stays in the queue; run
`/tachly:login <server>` again and it is sent automatically.

**macOS refuses to run the binary.** Check the plugin files with `xattr -l`
for `com.apple.quarantine`. A plugin installed with `/plugin` in the terminal
Claude Code comes from git and should have no quarantine attribute. A copy
taken from a download (a browser zip, for example) can have one, and the
ad-hoc signed build is then blocked. Whether an install through the Claude
desktop app sets it has not been checked yet. If the attribute is there,
install the plugin with `/plugin` in the terminal as shown above.

**Windows.** The plugin's launcher is a shell script that runs under Git Bash.
Install Git for Windows; PowerShell-only setups are not supported.

**Turning things off or uninstalling.** `/tachly:statusline off` restores your
status line, `/tachly:telemetry off` restores your telemetry settings and
`/tachly:coach off` silences Coach. To remove the plugin, run
`/tachly:statusline off` and `/tachly:logout` first (logout also turns
telemetry off), then:

```
/plugin uninstall tachly@tachly
/plugin marketplace remove tachly
```

The queue, the settings and the status line wrapper's copy of the binary
(`~/.tachly/bin/tachly`, `tachly.exe` on Windows, which
`/tachly:statusline off` does not delete) stay
in `~/.tachly`; delete that directory if you want them gone. If you skipped
`/tachly:logout`, the token also stays (in the macOS keychain, or in
`~/.tachly/token`). The `.tachly.json` files committed to your repositories
stay as they are.

## macOS signing

The macOS binary is ad-hoc signed until a Developer ID is configured: it has a
valid signature but no notarization, so `spctl -a -vv -t install` is expected to
reject it. A plugin installed through `/plugin` from git should get no
quarantine attribute, so macOS runs it normally. A download that carries a
quarantine attribute (a browser zip, for example) needs a Developer ID build.
Whether an install through the Claude desktop app carries one has not been
checked yet.

Every release ships `SHA256SUMS`. In a clone of this repository, check the
binaries with:

```
shasum -a 256 -c SHA256SUMS
```

## A closed client, and what protects you

The binaries are compiled and the source code is not published. That hides the
code but not the behavior: the strings, endpoints and redaction rules can be
read from the binary, and the traffic can be seen in any proxy. The real
protection is on the server: device authorization, limits and binding to a
person. This page is the plain description of what the plugin sends.

## License

Proprietary. Copyright (c) 2026 Ivan Bardak. All rights reserved. You may
install and run the unmodified binaries with a Tachly server. See
[LICENSE](LICENSE).
