# Tachly plugin for Claude Code

The Tachly plugin sends usage data from your local Claude Code sessions to a
Tachly server, so a team can see how Claude Code is used.

## Install

In Claude Code:

```
/plugin marketplace add tachly/tachly-plugin
/plugin install tachly@tachly
```

Then connect to your server:

```
/tachly:login <server>
```

Other commands: `/tachly:status`, `/tachly:init`, `/tachly:statusline`,
`/tachly:telemetry`, `/tachly:sync` (import older session history),
`/tachly:me` (open your dashboard in the browser) and `/tachly:logout`.

## Upgrading from the first releases

The first releases were called `pulse` (`pulse@ai-pulse`). Install this plugin as above, restart Claude Code, then
remove the old one:

```
/plugin uninstall pulse@ai-pulse
/plugin marketplace remove ai-pulse
```

Nothing else is needed: the first run moves the data directory `~/.claude-pulse` to `~/.tachly`, so you stay signed
in, the queue is kept, and the status line and telemetry settings are picked up as they are. `.claude-pulse.json`
files in your repositories are still read.

## Supported platforms

- macOS (Apple silicon and Intel)
- Linux x64 and arm64
- Windows x64. Windows needs Git for Windows (Git Bash); PowerShell-only
  setups are not supported.

On any other platform, or when the binary cannot be run (no exec permission,
or a noexec mount), the hooks stay silent and do nothing, and the commands
print a short message.

## Privacy

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

What the plugin never collects:

- Claude's responses or its thinking.
- The input of tools (except the subagent type and the skill name).
- The content of tool results (only their size).
- Any other field of the session records.
- Your Claude credentials or anything from the Claude keychain items.
- OpenTelemetry log events: telemetry never turns on `OTEL_LOG_*`.

How it handles your data:

- Hooks never use the network. A separate background process sends the data to
  the server you logged in to. A hook prints nothing, with one exception: at the
  start of a session (and only then) it can show you one Coach line, a
  `systemMessage` that only you see (the model never does). It is read from a
  small file the background process keeps (`state/coach.json`), is limited to
  one tip a day and one weekly digest, and is on by default. `/tachly:coach off`
  turns it off; `/tachly:coach mute <id>` or `mute all` also tells the server.
- The access token is never printed and never passed on a command line. The
  data directory is private to you (directories 0700, token and backups 0600).
- Only http(s) links are opened: the sign-in link your server returns. `/tachly:me` opens or prints only a link from the configured server's origin; a link to any other address is refused.
- The `claude` process that reads the limits is started without API key or
  session variables, without hooks and without saving a session.

## macOS signing

The macOS binary is ad-hoc signed until a Developer ID is configured: it has a
valid signature but no notarization, so `spctl -a -vv -t install` is expected to
reject it. A plugin installed through `/plugin` from git gets no quarantine
attribute, so macOS runs it normally. A download that carries a quarantine
attribute (a browser zip, for example) needs a Developer ID build.

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
